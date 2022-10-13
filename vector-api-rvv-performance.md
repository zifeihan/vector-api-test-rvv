### [RVV] Vector API impact on the number of instructions

##### The Java test cases are as follows:

```
import jdk.incubator.vector.IntVector;
import jdk.incubator.vector.VectorSpecies;

public class AddTest {
    static final VectorSpecies<Integer> SPECIES =
        IntVector.SPECIES_256;

    static final int SIZE = 1024;
    static int[] a = new int[SIZE];
    static int[] b = new int[SIZE];
    static int[] c = new int[SIZE];

    static {
        for (int i = 0; i < SIZE; i++) {
            a[i] = 1;
            b[i] = 2;
        }
    }

    static void workload() {
        for (int i = 0; i < a.length; i += SPECIES.length()) {
            IntVector av = IntVector.fromArray(SPECIES, a, i);
            IntVector bv = IntVector.fromArray(SPECIES, b, i);
            av.add(bv).intoArray(c,i);
        }
    }

    public static void main(String args[]) {
        for (int i = 0; i < 30_0000; i++) {
            workload();
        }
    }
}
```



##### No Vector API started/not implemented, some of the C2 logs are as follows:

```
;; B3: #	out( B4 B4 ) &lt;- in( B2 )  Freq: 0.899982
  0x000000400cfc8f24:   addw	t2,a0,zero
  0x000000400cfc8f28:   bltu	s0,a0,0x000000400cfc8f2c
 ;; B4: #	out( B38 B5 ) &lt;- in( B3 B3 )  Freq: 0.899982
  0x000000400cfc8f2c:   bltu	s0,a0,0x000000400cfc92fc
 ;; B5: #	out( B38 B6 ) &lt;- in( B4 )  Freq: 0.899981
  0x000000400cfc8f30:   slli	t2,t2,0x2
  0x000000400cfc8f34:   addi	t2,t2,23
  0x000000400cfc8f38:   ld	s0,264(s7)
  0x000000400cfc8f3c:   andi	t3,t2,-8
  0x000000400cfc8f40:   ld	t4,280(s7)
  0x000000400cfc8f44:   add	t3,s0,t3
  0x000000400cfc8f48:   bgeu	t3,t4,0x000000400cfc92fc
 ;; B6: #	out( B7 ) &lt;- in( B5 )  Freq: 0.899891
  0x000000400cfc8f4c:   sd	t3,264(s7)
  0x000000400cfc8f50:   addiw	t3,zero,1
  0x000000400cfc8f54:   sd	t3,0(s0) # 0x0000000000040000
  0x000000400cfc8f58:   lui	t3,0x41                     ;   {metadata({type array int})}
  0x000000400cfc8f5c:   addiw	t3,t3,-568
  0x000000400cfc8f60:   slli	t3,t3,0x20
  0x000000400cfc8f64:   srli	t3,t3,0x20
  0x000000400cfc8f68:   sw	t3,8(s0)
  0x000000400cfc8f6c:   srli	t2,t2,0x3
  0x000000400cfc8f70:   addi	t3,s0,16
  0x000000400cfc8f74:   addi	t4,t2,-2
  0x000000400cfc8f78:   sw	a0,12(s0)
 ;; zero_words {
  0x000000400cfc8f7c:   addiw	t0,zero,8
  0x000000400cfc8f80:   bltu	t4,t0,0x000000400cfc8f88
  0x000000400cfc8f84:   jal	ra,0x000000400cfc94c4       ;   {runtime_call StubRoutines (2)}
  0x000000400cfc8f88:   andi	t0,t4,4
  0x000000400cfc8f8c:   beqz	t0,0x000000400cfc8fb0
  0x000000400cfc8f90:   sd	zero,0(t3) # 0x0000000000041000
  0x000000400cfc8f94:   addi	t3,t3,8
  0x000000400cfc8f98:   sd	zero,0(t3)
  0x000000400cfc8f9c:   addi	t3,t3,8
  0x000000400cfc8fa0:   sd	zero,0(t3)
  0x000000400cfc8fa4:   addi	t3,t3,8
  0x000000400cfc8fa8:   sd	zero,0(t3)
  0x000000400cfc8fac:   addi	t3,t3,8
  0x000000400cfc8fb0:   andi	t0,t4,2
  0x000000400cfc8fb4:   beqz	t0,0x000000400cfc8fc8
  0x000000400cfc8fb8:   sd	zero,0(t3)
  0x000000400cfc8fbc:   addi	t3,t3,8
  0x000000400cfc8fc0:   sd	zero,0(t3)
  0x000000400cfc8fc4:   addi	t3,t3,8
  0x000000400cfc8fc8:   andi	t0,t4,1
  0x000000400cfc8fcc:   beqz	t0,0x000000400cfc8fd4
  0x000000400cfc8fd0:   sd	zero,0(t3)                  ;*newarray {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@15 (line 226)
 ;; } zero_words
 ;; B7: #	out( B59 B8 ) &lt;- in( B39 B6 )  Freq: 0.899982
  0x000000400cfc8fd4:   ld	a1,0(sp)
 ;; 0xffffffffffffffff
  0x000000400cfc8fd8:   lui	t1,0x0
  0x000000400cfc8fdc:   addi	t1,t1,-1
  0x000000400cfc8fe0:   slli	t1,t1,0xb
  0x000000400cfc8fe4:   addi	t1,t1,2047
  0x000000400cfc8fe8:   slli	t1,t1,0x6
  0x000000400cfc8fec:   addi	t1,t1,63
  0x000000400cfc8ff0:   jal	ra,0x000000400cfc94dc       ; ImmutableOopMap {fp=Oop [0]=Oop [8]=Oop [16]=Oop [24]=Oop }
                                                            ;*invokevirtual vec {reexecute=0 rethrow=0 return_oop=1}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@20 (line 227)
                                                            ;   {virtual_call}
 ;; B8: #	out( B50 B9 ) &lt;- in( B7 )  Freq: 0.899964
  0x000000400cfc8ff4:   sd	a0,32(sp)
  0x000000400cfc8ff8:   ld	t4,8(sp)
  0x000000400cfc8ffc:   lwu	t3,8(t4)                    ; implicit exception: dispatches to 0x000000400cfc93e4
 ;; B9: #	out( B42 B10 ) &lt;- in( B8 )  Freq: 0.899963
  0x000000400cfc9000:   addiw	t2,zero,1
  0x000000400cfc9004:   slli	t2,t2,0x23
  0x000000400cfc9008:   add	t2,t2,t3
  0x000000400cfc900c:   ld	t2,96(t2)
 ;; 0x8000969a0
  0x000000400cfc9010:   lui	t3,0x40                     ;   {metadata(&apos;jdk/incubator/vector/IntVector&apos;)}
  0x000000400cfc9014:   addi	t3,t3,4 # 0x0000000000040004
  0x000000400cfc9018:   slli	t3,t3,0xb
  0x000000400cfc901c:   addi	t3,t3,1446
  0x000000400cfc9020:   slli	t3,t3,0x6
  0x000000400cfc9024:   addi	t3,t3,32
  0x000000400cfc9028:   bne	t2,t3,0x000000400cfc934c
 ;; B10: #	out( B60 B11 ) &lt;- in( B9 )  Freq: 0.899962
  0x000000400cfc902c:   ld	t2,24(sp)
  0x000000400cfc9030:   lwu	t2,8(t2)
  0x000000400cfc9034:   addiw	t3,zero,1
  0x000000400cfc9038:   slli	t3,t3,0x23
  0x000000400cfc903c:   add	t2,t3,t2
  0x000000400cfc9040:   mv	a1,t4
  0x000000400cfc9044:   ld	t2,88(t2)                   ;*checkcast {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@26 (line 228)
  0x000000400cfc9048:   sd	t2,8(sp)
 ;; 0xffffffffffffffff
  0x000000400cfc904c:   lui	t1,0x0
  0x000000400cfc9050:   addi	t1,t1,-1
  0x000000400cfc9054:   slli	t1,t1,0xb
  0x000000400cfc9058:   addi	t1,t1,2047
  0x000000400cfc905c:   slli	t1,t1,0x6
  0x000000400cfc9060:   addi	t1,t1,63
  0x000000400cfc9064:   jal	ra,0x000000400cfc94f4       ; ImmutableOopMap {fp=Oop [0]=Oop [16]=Oop [24]=Oop [32]=Oop }
                                                            ;*invokevirtual vec {reexecute=0 rethrow=0 return_oop=1}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@29 (line 228)
                                                            ;   {virtual_call}
 ;; B11: #	out( B43 B12 ) &lt;- in( B10 )  Freq: 0.899944
 ;; 0x80009d300
  0x000000400cfc9068:   lui	t3,0x40                     ;   {metadata(&apos;jdk/incubator/vector/AbstractMask&apos;)}
  0x000000400cfc906c:   addi	t3,t3,4 # 0x0000000000040004
  0x000000400cfc9070:   slli	t3,t3,0xb
  0x000000400cfc9074:   addi	t3,t3,1868
  0x000000400cfc9078:   slli	t3,t3,0x6
  0x000000400cfc907c:   mv	t3,t3
  0x000000400cfc9080:   ld	t2,8(sp)
  0x000000400cfc9084:   bne	t2,t3,0x000000400cfc9364
 ;; B12: #	out( N621 ) &lt;- in( B11 )  Freq: 0.899943
  0x000000400cfc9088:   ld	t3,16(sp)
  0x000000400cfc908c:   sd	t3,8(sp)
  0x000000400cfc9090:   ld	t2,24(sp)
  0x000000400cfc9094:   ld	t3,32(sp)                   ;*checkcast {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@35 (line 229)
  0x000000400cfc9098:   addiw	a1,zero,-83
  0x000000400cfc909c:   sd	t3,16(sp)
  0x000000400cfc90a0:   sd	a0,24(sp)
  0x000000400cfc90a4:   sd	t2,32(sp)
  0x000000400cfc90a8:   jal	ra,0x000000400cfc950c       ; ImmutableOopMap {fp=Oop [0]=Oop [8]=Oop [16]=Oop [24]=Oop [32]=Oop }
                                                            ;*invokevirtual getBits {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@38 (line 229)
                                                            ;   {runtime_call UncommonTrapBlob}
 ;; uncommon trap returned which should never happen
  0x000000400cfc90ac:   csrw	time,zero
  0x000000400cfc90b0:   sw	s0,72(s1)
  0x000000400cfc90b2:   slli	s0,s0,0x2
  0x000000400cfc90b4:   addi	s0,sp,4
  0x000000400cfc90b6:   unimp
 ;; B13: #	out( B57 B14 ) &lt;- in( B1 )  Freq: 0.1
 ;; 0xffffffffffffffff
  0x000000400cfc90b8:   lui	t1,0x0
  0x000000400cfc90bc:   addi	t1,t1,-1
  0x000000400cfc90c0:   slli	t1,t1,0xb
  0x000000400cfc90c4:   addi	t1,t1,2047
  0x000000400cfc90c8:   slli	t1,t1,0x6
  0x000000400cfc90cc:   addi	t1,t1,63
  0x000000400cfc90d0:   jal	ra,0x000000400cfc9524       ; ImmutableOopMap {[0]=Oop [8]=Oop [16]=Oop }
                                                            ;*invokevirtual length {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@1 (line 204)
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@7 (line 224)
                                                            ;   {virtual_call}
 ;; B14: #	out( B15 B15 ) &lt;- in( B13 )  Freq: 0.099998
  0x000000400cfc90d4:   addw	t2,a0,zero
  0x000000400cfc90d8:   mv	t5,a0
  0x000000400cfc90dc:   bltu	s0,a0,0x000000400cfc90e0
 ;; B15: #	out( B40 B16 ) &lt;- in( B14 B14 )  Freq: 0.099998
  0x000000400cfc90e0:   bltu	s0,a0,0x000000400cfc9324
 ;; B16: #	out( B40 B17 ) &lt;- in( B15 )  Freq: 0.0999979
  0x000000400cfc90e4:   slli	t2,t2,0x2
  0x000000400cfc90e8:   addi	t2,t2,23
  0x000000400cfc90ec:   ld	a0,264(s7)
  0x000000400cfc90f0:   andi	t3,t2,-8
  0x000000400cfc90f4:   ld	t4,280(s7)
  0x000000400cfc90f8:   add	t3,a0,t3
  0x000000400cfc90fc:   bgeu	t3,t4,0x000000400cfc9324
 ;; B17: #	out( B18 ) &lt;- in( B16 )  Freq: 0.0999879
  0x000000400cfc9100:   sd	t3,264(s7)
  0x000000400cfc9104:   addiw	t3,zero,1
  0x000000400cfc9108:   lui	t4,0x41                     ;   {metadata({type array int})}
  0x000000400cfc910c:   addiw	t4,t4,-568
  0x000000400cfc9110:   slli	t4,t4,0x20
  0x000000400cfc9114:   srli	t4,t4,0x20
  0x000000400cfc9118:   sd	t3,0(a0)
  0x000000400cfc911c:   sw	t4,8(a0)
  0x000000400cfc9120:   srli	t2,t2,0x3
  0x000000400cfc9124:   addi	t3,a0,16
  0x000000400cfc9128:   addi	t4,t2,-2
  0x000000400cfc912c:   sw	t5,12(a0)
 ;; zero_words {
  0x000000400cfc9130:   addiw	t0,zero,8
  0x000000400cfc9134:   bltu	t4,t0,0x000000400cfc913c
  0x000000400cfc9138:   jal	ra,0x000000400cfc953c       ;   {runtime_call StubRoutines (2)}
  0x000000400cfc913c:   andi	t0,t4,4
  0x000000400cfc9140:   beqz	t0,0x000000400cfc9164
  0x000000400cfc9144:   sd	zero,0(t3)
  0x000000400cfc9148:   addi	t3,t3,8
  0x000000400cfc914c:   sd	zero,0(t3)
  0x000000400cfc9150:   addi	t3,t3,8
  0x000000400cfc9154:   sd	zero,0(t3)
  0x000000400cfc9158:   addi	t3,t3,8
  0x000000400cfc915c:   sd	zero,0(t3)
  0x000000400cfc9160:   addi	t3,t3,8
  0x000000400cfc9164:   andi	t0,t4,2
  0x000000400cfc9168:   beqz	t0,0x000000400cfc917c
  0x000000400cfc916c:   sd	zero,0(t3)
  0x000000400cfc9170:   addi	t3,t3,8
  0x000000400cfc9174:   sd	zero,0(t3)
  0x000000400cfc9178:   addi	t3,t3,8
  0x000000400cfc917c:   andi	t0,t4,1
  0x000000400cfc9180:   beqz	t0,0x000000400cfc9188
  0x000000400cfc9184:   sd	zero,0(t3)
 ;; } zero_words
  0x000000400cfc9188:   mv	s0,t5
 ;; B18: #	out( B56 B19 ) &lt;- in( B41 B17 )  Freq: 0.099998
  0x000000400cfc918c:   fence	ow,ow                       ;*newarray {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@4 (line 204)
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@7 (line 224)
  0x000000400cfc9190:   sd	a0,24(sp)
  0x000000400cfc9194:   ld	a1,0(sp)
 ;; 0xffffffffffffffff
  0x000000400cfc9198:   lui	t1,0x0
  0x000000400cfc919c:   addi	t1,t1,-1
  0x000000400cfc91a0:   slli	t1,t1,0xb
  0x000000400cfc91a4:   addi	t1,t1,2047
  0x000000400cfc91a8:   slli	t1,t1,0x6
  0x000000400cfc91ac:   addi	t1,t1,63
  0x000000400cfc91b0:   jal	ra,0x000000400cfc9554       ; ImmutableOopMap {[0]=Oop [8]=Oop [16]=Oop [24]=Oop }
                                                            ;*invokevirtual vec {reexecute=0 rethrow=0 return_oop=1}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@8 (line 205)
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@7 (line 224)
                                                            ;   {virtual_call}
 ;; B19: #	out( B51 B20 ) &lt;- in( B18 )  Freq: 0.099996
  0x000000400cfc91b4:   sd	a0,32(sp)
  0x000000400cfc91b8:   ld	t4,8(sp)
  0x000000400cfc91bc:   lwu	t2,8(t4) # 0x0000000000041008; implicit exception: dispatches to 0x000000400cfc9400
 ;; B20: #	out( B44 B21 ) &lt;- in( B19 )  Freq: 0.0999959
  0x000000400cfc91c0:   addiw	t3,zero,1
  0x000000400cfc91c4:   slli	t3,t3,0x23
  0x000000400cfc91c8:   add	t2,t3,t2
  0x000000400cfc91cc:   ld	t2,96(t2)
 ;; 0x8000969a0
  0x000000400cfc91d0:   lui	t3,0x40                     ;   {metadata(&apos;jdk/incubator/vector/IntVector&apos;)}
  0x000000400cfc91d4:   addi	t3,t3,4 # 0x0000000000040004
  0x000000400cfc91d8:   slli	t3,t3,0xb
  0x000000400cfc91dc:   addi	t3,t3,1446
  0x000000400cfc91e0:   slli	t3,t3,0x6
  0x000000400cfc91e4:   addi	t3,t3,32
  0x000000400cfc91e8:   bne	t2,t3,0x000000400cfc937c
 ;; B21: #	out( B55 B22 ) &lt;- in( B20 )  Freq: 0.0999958
  0x000000400cfc91ec:   mv	a1,t4
 ;; 0xffffffffffffffff
  0x000000400cfc91f0:   lui	t1,0x0
  0x000000400cfc91f4:   addi	t1,t1,-1
  0x000000400cfc91f8:   slli	t1,t1,0xb
  0x000000400cfc91fc:   addi	t1,t1,2047
  0x000000400cfc9200:   slli	t1,t1,0x6
  0x000000400cfc9204:   addi	t1,t1,63
  0x000000400cfc9208:   jal	ra,0x000000400cfc956c       ; ImmutableOopMap {[0]=Oop [16]=Oop [24]=Oop [32]=Oop }
                                                            ;*invokevirtual vec {reexecute=0 rethrow=0 return_oop=1}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@17 (line 206)
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@7 (line 224)
                                                            ;   {virtual_call}
 ;; B22: #	out( B36 B23 ) &lt;- in( B21 )  Freq: 0.0999938
  0x000000400cfc920c:   blez	s0,0x000000400cfc92bc       ;*if_icmpge {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@29 (line 207)
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@7 (line 224)
 ;; B23: #	out( B46 B24 ) &lt;- in( B22 )  Freq: 0.0499969
  0x000000400cfc9210:   ld	t5,32(sp)
  0x000000400cfc9214:   lwu	t2,12(t5)                   ; implicit exception: dispatches to 0x000000400cfc9398
                                                            ;*iaload {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@42 (line 208)
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@7 (line 224)
 ;; B24: #	out( B46 B25 ) &lt;- in( B23 )  Freq: 0.0499969
  0x000000400cfc9218:   addiw	t4,s0,-1
  0x000000400cfc921c:   beqz	t2,0x000000400cfc9398
 ;; B25: #	out( B46 B26 ) &lt;- in( B24 )  Freq: 0.0499968
  0x000000400cfc9220:   bgeu	t4,t2,0x000000400cfc9398
 ;; B26: #	out( B46 B27 ) &lt;- in( B25 )  Freq: 0.0499968
  0x000000400cfc9224:   mv	t6,a0
  0x000000400cfc9228:   lwu	t2,12(a0)                   ; implicit exception: dispatches to 0x000000400cfc9398
                                                            ;*iaload {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@47 (line 208)
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@7 (line 224)
 ;; B27: #	out( B46 B28 ) &lt;- in( B26 )  Freq: 0.0499967
  0x000000400cfc922c:   beqz	t2,0x000000400cfc9398
 ;; B28: #	out( B46 B29 ) &lt;- in( B27 )  Freq: 0.0499967
  0x000000400cfc9230:   bgeu	t4,t2,0x000000400cfc9398
 ;; B29: #	out( B45 B30 ) &lt;- in( B28 )  Freq: 0.0499966
  0x000000400cfc9234:   ld	a0,16(sp)
  0x000000400cfc9238:   lwu	t3,8(a0)                    ; implicit exception: dispatches to 0x000000400cfc9394
 ;; B30: #	out( B49 B31 ) &lt;- in( B29 )  Freq: 0.0499966
  0x000000400cfc923c:   lui	t2,0xaf                     ;   {metadata(&apos;jdk/incubator/vector/IntVector$$Lambda$42+0x00000008000ae960&apos;)}
  0x000000400cfc9240:   addiw	t2,t2,-1696
  0x000000400cfc9244:   slli	t2,t2,0x20
  0x000000400cfc9248:   srli	t2,t2,0x20
  0x000000400cfc924c:   bne	t3,t2,0x000000400cfc93dc
 ;; B31: #	out( B47 B32 ) &lt;- in( B30 )  Freq: 0.0499965
  0x000000400cfc9250:   beqz	s0,0x000000400cfc93cc
 ;; B32: #	out( B48 B33 ) &lt;- in( B31 )  Freq: 0.0499965
  0x000000400cfc9254:   bgeu	t4,s0,0x000000400cfc93d4
 ;; B33: #	out( B34 ) &lt;- in( B32 )  Freq: 0.0499964
  0x000000400cfc9258:   mv	t2,a0                       ;*invokeinterface apply {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@48 (line 208)
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@7 (line 224)
  0x000000400cfc925c:   sext.w	t3,zero
  0x000000400cfc9260:   sw	s0,12(sp)
  0x000000400cfc9264:   sd	t5,16(sp)
  0x000000400cfc9268:   sd	t6,32(sp)
  0x000000400cfc926c:   sd	t2,40(sp)                   ;*aload_3 {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@32 (line 208)
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@7 (line 224)
 ;; B34: #	out( B61 B35 ) &lt;- in( B33 B35 ) Loop( B34-B35 inner ) Freq: 0.499924
  0x000000400cfc9270:   addw	t2,t3,zero
  0x000000400cfc9274:   sw	t3,8(sp)
  0x000000400cfc9278:   slli	t2,t2,0x2                   ;*iaload {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@47 (line 208)
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@7 (line 224)
  0x000000400cfc927c:   ld	t3,16(sp)
  0x000000400cfc9280:   add	t3,t3,t2
  0x000000400cfc9284:   ld	t4,32(sp)
  0x000000400cfc9288:   add	t4,t4,t2
  0x000000400cfc928c:   lw	a3,16(t3)                   ;*iaload {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@42 (line 208)
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@7 (line 224)
  0x000000400cfc9290:   lw	a4,16(t4)                   ;*iaload {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@47 (line 208)
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@7 (line 224)
  0x000000400cfc9294:   ld	t3,24(sp)
  0x000000400cfc9298:   add	s0,t3,t2
  0x000000400cfc929c:   ld	a1,40(sp)
  0x000000400cfc92a0:   lw	a2,8(sp)
  0x000000400cfc92a4:   jal	ra,0x000000400cfc9584       ; ImmutableOopMap {[0]=Oop [16]=Oop [24]=Oop fp=Derived_oop_[24] [32]=Oop [40]=Oop }
                                                            ;*invokeinterface apply {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@48 (line 208)
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@7 (line 224)
                                                            ;   {optimized virtual_call}
 ;; B35: #	out( B34 B36 ) &lt;- in( B34 )  Freq: 0.499914
  0x000000400cfc92a8:   lw	t3,8(sp)
  0x000000400cfc92ac:   addiw	t3,t3,1                     ;*iinc {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@54 (line 207)
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@7 (line 224)
  0x000000400cfc92b0:   sw	a0,16(s0)                   ;*iastore {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@53 (line 208)
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@7 (line 224)
  0x000000400cfc92b4:   lw	t2,12(sp)
  0x000000400cfc92b8:   blt	t3,t2,0x000000400cfc9270    ;*if_icmpge {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@29 (line 207)
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@7 (line 224)
 ;; B36: #	out( B54 B37 ) &lt;- in( B35 B22 )  Freq: 0.0999883
  0x000000400cfc92bc:   ld	a1,0(sp)
  0x000000400cfc92c0:   ld	a2,24(sp)
 ;; 0xffffffffffffffff
  0x000000400cfc92c4:   lui	t1,0x0
  0x000000400cfc92c8:   addi	t1,t1,-1
  0x000000400cfc92cc:   slli	t1,t1,0xb
  0x000000400cfc92d0:   addi	t1,t1,2047
  0x000000400cfc92d4:   slli	t1,t1,0x6
  0x000000400cfc92d8:   addi	t1,t1,63
  0x000000400cfc92dc:   jal	ra,0x000000400cfc959c       ; ImmutableOopMap {}
                                                            ;*invokevirtual vectorFactory {reexecute=0 rethrow=0 return_oop=1}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@62 (line 210)
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@7 (line 224)
                                                            ;   {virtual_call}
 ;; B37: #	out( N621 ) &lt;- in( B36 )  Freq: 0.0999863
  0x000000400cfc92e0:   ld	s0,64(sp)
  0x000000400cfc92e4:   ld	ra,72(sp)
  0x000000400cfc92e8:   addi	sp,sp,80
  0x000000400cfc92ec:   ld	t0,952(s7)                  ;   {poll_return}
  0x000000400cfc92f0:   bgeu	t0,sp,0x000000400cfc92f8
  0x000000400cfc92f4:   j	0x000000400cfc946c
  0x000000400cfc92f8:   ret
 ;; B38: #	out( B53 B39 ) &lt;- in( B4 B5 )  Freq: 9.0925e-05
 ;; 0x800040dc8
  0x000000400cfc92fc:   lui	a1,0x40                     ;   {metadata({type array int})}
  0x000000400cfc9300:   addi	a1,a1,2 # 0x0000000000040002
  0x000000400cfc9304:   slli	a1,a1,0xb
  0x000000400cfc9308:   addi	a1,a1,55
  0x000000400cfc930c:   slli	a1,a1,0x6
  0x000000400cfc9310:   addi	a1,a1,8
  0x000000400cfc9314:   mv	a2,a0
  0x000000400cfc9318:   jal	ra,0x000000400cfc95b4       ; ImmutableOopMap {[0]=Oop [8]=Oop [16]=Oop [24]=Oop }
                                                            ;*newarray {reexecute=0 rethrow=0 return_oop=1}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@15 (line 226)
                                                            ;   {runtime_call _new_array_Java}
 ;; B39: #	out( B7 ) &lt;- in( B38 )  Freq: 9.09232e-05
  0x000000400cfc931c:   mv	s0,a0
  0x000000400cfc9320:   j	0x000000400cfc8fd4
 ;; B40: #	out( B52 B41 ) &lt;- in( B15 B16 )  Freq: 1.01028e-05
  0x000000400cfc9324:   mv	s0,t5
 ;; 0x800040dc8
  0x000000400cfc9328:   lui	a1,0x40                     ;   {metadata({type array int})}
  0x000000400cfc932c:   addi	a1,a1,2 # 0x0000000000040002
  0x000000400cfc9330:   slli	a1,a1,0xb
  0x000000400cfc9334:   addi	a1,a1,55
  0x000000400cfc9338:   slli	a1,a1,0x6
  0x000000400cfc933c:   addi	a1,a1,8
  0x000000400cfc9340:   mv	a2,t5
  0x000000400cfc9344:   jal	ra,0x000000400cfc95cc       ; ImmutableOopMap {[0]=Oop [8]=Oop [16]=Oop }
                                                            ;*newarray {reexecute=0 rethrow=0 return_oop=1}
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@4 (line 204)
                                                            ; - jdk.incubator.vector.IntVector::bOpTemplate@7 (line 224)
                                                            ;   {runtime_call _new_array_Java}
```



##### After using -XX:+UseRVV to turn on RVV, some of the C2 logs are as follows:

```
 ;; B10: #	out( B45 B11 ) &lt;- in( B9 )  Freq: 4.99783
  0x000000400cfd54c0:   addw	t3,t2,zero                  ;*i2l {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::arrayAddress@4 (line 3691)
                                                            ; - jdk.incubator.vector.IntVector::fromArray0Template@20 (line 3444)
                                                            ; - jdk.incubator.vector.Int256Vector::fromArray0@3 (line 859)
                                                            ; - jdk.incubator.vector.IntVector::fromArray@24 (line 2955)
                                                            ; - AddTest::workload@17 (line 23)
  0x000000400cfd54c4:   sw	t2,8(sp)                    ;*synchronization entry
                                                            ; - jdk.incubator.vector.IntVector::intoArray@-1 (line 3226)
                                                            ; - AddTest::workload@41 (line 25)
  0x000000400cfd54c8:   slli	t5,t3,0x2                   ;*lshl {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::arrayAddress@8 (line 3691)
                                                            ; - jdk.incubator.vector.IntVector::fromArray0Template@20 (line 3444)
                                                            ; - jdk.incubator.vector.Int256Vector::fromArray0@3 (line 859)
                                                            ; - jdk.incubator.vector.IntVector::fromArray@24 (line 2955)
                                                            ; - AddTest::workload@17 (line 23)
  0x000000400cfd54cc:   sd	t3,16(sp)
  0x000000400cfd54d0:   add	t2,t4,t5
  0x000000400cfd54d4:   sd	t5,24(sp)
  0x000000400cfd54d8:   addi	t2,t2,16                    ;*synchronization entry
                                                            ; - jdk.incubator.vector.IntVector::intoArray@-1 (line 3226)
                                                            ; - AddTest::workload@41 (line 25)
  0x000000400cfd54dc:   0x10072d7
  0x000000400cfd54e0:   0x203e087                           ;*invokestatic load {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::fromArray0Template@32 (line 3442)
                                                            ; - jdk.incubator.vector.Int256Vector::fromArray0@3 (line 859)
                                                            ; - jdk.incubator.vector.IntVector::fromArray@24 (line 2955)
                                                            ; - AddTest::workload@17 (line 23)
  0x000000400cfd54e4:   addi	t0,sp,32
  0x000000400cfd54e8:   0x28280a7                           ;*synchronization entry
                                                            ; - jdk.incubator.vector.IntVector::intoArray@-1 (line 3226)
                                                            ; - AddTest::workload@41 (line 25)
 ;; 0x408c06a5f0
  0x000000400cfd54ec:   lui	t2,0x22                     ;   {oop(a &apos;java/lang/Class&apos;{0x00000004338cdc70} = &apos;AddTest&apos;)}
  0x000000400cfd54f0:   addi	t2,t2,-1594 # 0x00000000000219c6
  0x000000400cfd54f4:   slli	t2,t2,0xb
  0x000000400cfd54f8:   addi	t2,t2,881
  0x000000400cfd54fc:   slli	t2,t2,0x6
  0x000000400cfd5500:   addi	t2,t2,48
  0x000000400cfd5504:   lwu	t3,120(t2)
  0x000000400cfd5508:   slli	s0,t3,0x3                   ;*getstatic b {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - AddTest::workload@24 (line 24)
 ;; 0x408c06a5f8
  0x000000400cfd550c:   lui	a1,0x22                     ;   {oop(a &apos;jdk/incubator/vector/IntVector$IntSpecies&apos;{0x00000004338db090})}
  0x000000400cfd5510:   addi	a1,a1,-1594 # 0x00000000000219c6
  0x000000400cfd5514:   slli	a1,a1,0xb
  0x000000400cfd5518:   addi	a1,a1,1730
  0x000000400cfd551c:   slli	a1,a1,0x6
  0x000000400cfd5520:   addi	a1,a1,16
  0x000000400cfd5524:   jal	ra,0x000000400cfd592c       ; ImmutableOopMap {fp=Oop }
                                                            ;*invokeinterface length {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::fromArray@2 (line 2953)
                                                            ; - AddTest::workload@28 (line 24)
                                                            ;   {optimized virtual_call}
 ;; B11: #	out( B37 B12 ) &lt;- in( B10 )  Freq: 4.99773
  0x000000400cfd5528:   mv	t4,s0
  0x000000400cfd552c:   lwu	t3,12(s0)                   ; implicit exception: dispatches to 0x000000400cfd584c
 ;; B12: #	out( B34 B13 ) &lt;- in( B11 )  Freq: 4.99772
  0x000000400cfd5530:   ld	t6,24(sp)
  0x000000400cfd5534:   add	t2,s0,t6
  0x000000400cfd5538:   subw	t3,t3,a0
  0x000000400cfd553c:   addi	t2,t2,16                    ;*synchronization entry
                                                            ; - jdk.incubator.vector.IntVector::intoArray@-1 (line 3226)
                                                            ; - AddTest::workload@41 (line 25)
  0x000000400cfd5540:   addiw	s0,t3,1                     ;*isub {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.VectorIntrinsics::checkFromIndexSize@42 (line 57)
                                                            ; - jdk.incubator.vector.IntVector::fromArray@9 (line 2953)
                                                            ; - AddTest::workload@28 (line 24)
  0x000000400cfd5544:   bltz	s0,0x000000400cfd57fc
 ;; B13: #	out( B31 B14 ) &lt;- in( B12 )  Freq: 4.99772
  0x000000400cfd5548:   lw	t5,8(sp)
  0x000000400cfd554c:   bgeu	t5,s0,0x000000400cfd57a8    ;*synchronization entry
                                                            ; - jdk.incubator.vector.IntVector::intoArray@-1 (line 3226)
                                                            ; - AddTest::workload@41 (line 25)
 ;; B14: #	out( B26 B15 ) &lt;- in( B13 )  Freq: 4.99771
  0x000000400cfd5550:   0x10072d7
  0x000000400cfd5554:   0x203e207                           ;*invokestatic load {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::fromArray0Template@32 (line 3442)
                                                            ; - jdk.incubator.vector.Int256Vector::fromArray0@3 (line 859)
                                                            ; - jdk.incubator.vector.IntVector::fromArray@24 (line 2955)
                                                            ; - AddTest::workload@28 (line 24)
  0x000000400cfd5558:   ld	a0,264(s7)
  0x000000400cfd555c:   ld	t2,280(s7)
  0x000000400cfd5560:   addi	t3,a0,48
  0x000000400cfd5564:   bgeu	t3,t2,0x000000400cfd56fc
 ;; B15: #	out( B16 ) &lt;- in( B14 )  Freq: 4.99721
  0x000000400cfd5568:   sd	t3,264(s7)
  0x000000400cfd556c:   addiw	t2,zero,1
  0x000000400cfd5570:   sd	t2,0(a0)
  0x000000400cfd5574:   lui	t3,0x41                     ;   {metadata({type array int})}
  0x000000400cfd5578:   addiw	t3,t3,-568
  0x000000400cfd557c:   slli	t3,t3,0x20
  0x000000400cfd5580:   srli	t3,t3,0x20
  0x000000400cfd5584:   sw	t3,8(a0)
  0x000000400cfd5588:   addiw	t2,zero,8
  0x000000400cfd558c:   sw	t2,12(a0)
  0x000000400cfd5590:   addi	t3,a0,16
  0x000000400cfd5594:   addiw	t4,zero,4
  0x000000400cfd5598:   0x1aef2d7
  0x000000400cfd559c:   0x2e000057
  0x000000400cfd55a0:   0x1aef2d7
  0x000000400cfd55a4:   0x20e7027
  0x000000400cfd55a8:   sub	t4,t4,t0
  0x000000400cfd55ac:   slli	t0,t0,0x3
  0x000000400cfd55b0:   add	t3,t3,t0
  0x000000400cfd55b4:   bnez	t4,0x000000400cfd55a0
 ;; B16: #	out( B28 B17 ) &lt;- in( B27 B15 )  Freq: 4.99771
  0x000000400cfd55b8:   fence	ow,ow
  0x000000400cfd55bc:   addi	t0,sp,32
  0x000000400cfd55c0:   0x2828087                           ;*synchronization entry
                                                            ; - jdk.incubator.vector.IntVector::intoArray@-1 (line 3226)
                                                            ; - AddTest::workload@41 (line 25)
  0x000000400cfd55c4:   0x10072d7
  0x000000400cfd55c8:   0x2120157                           ;*invokestatic binaryOp {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::lanewiseTemplate@154 (line 784)
                                                            ; - jdk.incubator.vector.Int256Vector::lanewise@3 (line 285)
                                                            ; - jdk.incubator.vector.Int256Vector::lanewise@3 (line 41)
                                                            ; - jdk.incubator.vector.IntVector::add@5 (line 1352)
                                                            ; - AddTest::workload@34 (line 25)
  0x000000400cfd55cc:   addi	t2,a0,16
  0x000000400cfd55d0:   0x10072d7
  0x000000400cfd55d4:   0x203e127
  0x000000400cfd55d8:   ld	a6,264(s7)
  0x000000400cfd55dc:   ld	t2,280(s7)
  0x000000400cfd55e0:   addi	t3,a6,16
  0x000000400cfd55e4:   bgeu	t3,t2,0x000000400cfd573c
```

With vlen = 256, vle V1, [R7] can load as much as 256/8 data into the vector registers at once when the data type is int/32 bits (consistent with SPECIES.length() in the java code above).



#### Statistics on the number of instructions compiled for the AddTest#workload method

Loop test without RVV enabled: 820 instructions for the outer loop, 30 instructions for the inner loop

RVV-enabled loop test: single loop with approximately 450 instructions

| Total data amount | Loop test without RVV enabled                                | Loop test with RVV enabled                                   |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1024              | Need outer loop 1024/8 = 128 times, need inner loop 8 times, need instruction number 128 * 820 + 8 * 30 = 104960 + 240 = 105200 | Need to loop 1024/8 = 128 times, need 128 * 450 =57600 instructions |
| 2056              | Needs 2048/8 = 256 outer loops, 8 more inner loops, 256 * 820 + 8 * 30 = 209920 + 240 = 210160 | The number of cycles required is 2048/8 = 256, and the number of instructions required is 256 * 450 =115200 |

The number of loops here is based on the total amount of data/SPECIES, and the SPECIES variable is defined as follows: ` static final VectorSpecies<Integer> SPECIES = IntVector.SPECIES_256;` , so if under C2, which does not implement the Vector API, the default processing logic is called in If the default processing logic is called, there is an inner loop during `av.add(bv)` to iterate through each element, but under C2, which implements the Vector API, the default processing logic is not called and there is no memory loop operation.

```
    static void workload() {
        for (int i = 0; i < a.length; i += SPECIES.length()) {
            IntVector av = IntVector.fromArray(SPECIES, a, i);
            IntVector bv = IntVector.fromArray(SPECIES, b, i);
            av.add(bv).intoArray(c,i);
        }
    }
```