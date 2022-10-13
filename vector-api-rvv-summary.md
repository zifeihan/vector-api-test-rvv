**[vectorIntrinsics] Vector API for RISC-V**

## Summary

The implementation of vector nodes plays an important role in the implementation of the Vector-API. In the current RISC-V backend implementation of the OpenJDK, some vector nodes have been implemented using the RISC-V V extensions, e.g. `LoadVector,StoreVector,AddVB` and so on. With these vector node implementations, the C2 compiler is able to handle some specific vector computations faster and with better performance. However, the current vector node implementations are still lacking compared to AARCH64's SVE/NEON and X86's avx512, for example: `Op_LoadVectorGather,Op_StoreVectorScatter,AndReductionV` and so on. 

Therefore, we currently want to make more vector node implementations based on RISC-V V extensions for the RISC-V backend of OpenJDK first.

## Status

According to our understanding, the C2 vector node of the RISC-V V extension currently exists to allow the program to use more of the RISC-V V extension during runtime, thus reducing the number of assembly instructions (using a single instruction, multiple data mode), thus allowing for faster execution of the program. Currently, the Vector API works fine on the OpenJDK RISC-V platform, but because some vector nodes are missing, the Vector API C2 mode uses the normal C2 nodes for the unimplemented C2 vector nodes, so that the lack of vectorized nodes does not cause the Vector API to be used in the OpenJDK RISC-V platform. API is not available on the OpenJDK RISC-V platform due to the lack of vectorized nodes.

https://github.com/openjdk/jdk/blob/master/test/jdk/jdk/incubator/vector/Int256VectorTests.java#ANDReduceInt256VectorTests This test performs AndReduce operations on a set of data. By printing the C2 execution log of the method, we can see that the method also performs C2 compilation, but it is implemented using normal C2 nodes and does not use the RISC-V V extensions.

## Example

The following implementation of AndReduce for the Vector API uses the RISC-V V extension, which provides 32 vector registers and an instruction set to manipulate them. These instruction sets enable vectorization operations similar to AARCH64's SVE, where the RISC-V V extension instruction set precedes operations on vector register data, Some RISC-V V extended instruction sets operate on registers that can contain scalar (normal) registers, for example `vop.vx vd, vs2, rs1, vm # integer vector-scalar vd[i] = vs2[i] op x[rs1]` . For the case where more RISC-V V extension instructions operate on vector registers, the data needs to be loaded into the vector registers first, and then the RISC-V V extension instruction set operates on the vector registers. The Vector API's AndReduce is similar to the existing AddReduce in that it loads data from memory/scalar registers into vector registers, then operates on the vector registers, and finally moves the data to the scalar registers. Since the loading and storage of vector data has already been implemented (src/hotspot/cpu/riscv/riscv_v.ad), we refer to `AddReductionVI` and implement `AndReductionV`, the main implementation node of AndReduce for the Vector API.

```
instruct reduce_andI(iRegINoSp dst, iRegIorL2I src1, vReg src2, vReg tmp) %{
  predicate(n->in(2)->bottom_type()->is_vect()->element_basic_type() == T_INT);
  match(Set dst (AndReductionV src1 src2));
  effect(TEMP tmp);
  ins_cost(VEC_COST);
  format %{ "vmv.s.x $tmp, $src1\t#@reduce_andI\n\t"
            "vredand.vs $tmp, $src2, $tmp\n\t"
            "vmv.x.s  $dst, $tmp" %}
  ins_encode %{
    __ vsetvli(t0, x0, Assembler::e32);
    __ vmv_s_x(as_VectorRegister($tmp$$reg), $src1$$Register);
    __ vredand_vs(as_VectorRegister($tmp$$reg), as_VectorRegister($src2$$reg),
                  as_VectorRegister($tmp$$reg));
    __ vmv_x_s($dst$$Register, as_VectorRegister($tmp$$reg));
  %}
  ins_pipe(pipe_slow);
%}
```

The `T_INT` data type is implemented here, and the implementation is given in a different node for `T_BYTE, T_SHORT, T_LONG`. After implementation, the compilation log of the https://github.com/openjdk/jdk/blob/master/test/jdk/jdk/incubator/vector/Int256VectorTests.java#ANDReduceInt256VectorTests method is printed, and RISC-V is enabled. After implementation, the compilation log of the  method is printed, and the RISC-V V extension is enabled, so that the execution of the method matches the new AndReductionV node.

```
27c     B21: #	out( B25 B22 ) &lt;- in( B20 )  Freq: 32.4376
27c     # castII of R8, #@castII
27c     addw  R7, R8, zr	#@convI2L_reg_reg
280     slli  R29, R7, (#2 &amp; 0x3f)	#@lShiftL_reg_imm
284     spill [sp, #24] -&gt; R7	# spill size = 64
288     add R7, R7, R29	# ptr, #@addP_reg_reg
28c     addi  R7, R7, #16	# ptr, #@addP_reg_imm
290     vle V2, [R7]	#@loadV
298     ....
2c0     vmv.s.x V1, R7	#@reduce_andI
	vredand.vs V1, V2, V1
	vmv.x.s  R28, V1
```

## Test tips

1. After implementing each vector node, write test cases for that node, perform rigorous functional testing, and perform complete testing of the vector in jtreg.
2. Print the JAVA test case method using the vector node, and analyze the compilation log to confirm that the optimization of the C2 Vector Node is occurring correctly.
3. Since no physical machine capable of executing RISC-V V extensions has been found, the above tests were performed with the RISC-V V extensions v1.0 enabled in QEMU.

## Performance Test

Continue using https://github.com/openjdk/jdk/blob/master/test/jdk/jdk/incubator/vector/Int256VectorTests.java#ANDReduceInt256VectorTests to Test the performance before and after implementing the RISC-V V extensions added.

Benchmark ADDReduceInt256VectorTests, ANDReduceInt256VectorTests, ORReduceInt256VectorTests, XORReduceInt256VectorTests, negInt256VectorTests and NEGInt256VectorTests under `test/jdk/jdk/incubator/vector` are tested. The sum of execution time shows ~50.7% reduction on average.

## Goals and roadmap

Considering code safety and testing, we plan to implement the Vector API step by step according to the C2 Vector Node types required by the Vector API. For example, we will separate `AndReductionV, OrReductionV, XorReductionV` into one class, `VectorCastB2X, VectorCastS2X, VectorCastD2X` into one class, and so on, and then we will submit PRs upstream according to the C2 Vector Node type. In order to keep the code safe, we will implement the simple vector nodes first, from simple to hard, and avoid modifying other public code in the process for the time being. These are our goals and plans, and we welcome suggestions and corrections from the community.