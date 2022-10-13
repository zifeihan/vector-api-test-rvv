### [SVE] Vector API impact on the number of instructions

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
;; B3: #	out( B4 ) &lt;- in( B2 )  Freq: 0.499999
  0x000000550d88525c:   mov	w12, wzr                    ;*getstatic SPECIES {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - AddTest::workload@10 (line 23)
 ;; B4: #	out( B45 B5 ) &lt;- in( B3 B29 ) Loop( B4-B29 inner ) Freq: 4.99567
  0x000000550d885260:   str	w12, [sp, #8]               ;*invokestatic store {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::intoArray@43 (line 3228)
                                                            ; - AddTest::workload@41 (line 25)
 ;; 0x557806AC68
  0x000000550d885264:   mov	x1, #0x2710                	// #10000
                                                            ;   {oop(a &apos;jdk/incubator/vector/IntVector$IntSpecies&apos;{0x00000000c2e12710})}
  0x000000550d885268:   movk	x1, #0xc2e1, lsl #16
  0x000000550d88526c:   movk	x1, #0x0, lsl #32
  0x000000550d885270:   bl	0x000000550d8856f8          ; ImmutableOopMap {rfp=NarrowOop }
                                                            ;*invokeinterface length {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::fromArray@2 (line 2953)
                                                            ; - AddTest::workload@17 (line 23)
                                                            ;   {optimized virtual_call}
 ;; B5: #	out( B51 B6 ) &lt;- in( B4 )  Freq: 4.99557
  0x000000550d885274:   mov	w13, w29
  0x000000550d885278:   ldr	w10, [x29, #12]             ; implicit exception: dispatches to 0x000000550d885628
 ;; B6: #	out( B35 B7 ) &lt;- in( B5 )  Freq: 4.99557
  0x000000550d88527c:   sub	w11, w10, w0                ;*invokestatic store {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::intoArray@43 (line 3228)
                                                            ; - AddTest::workload@41 (line 25)
  0x000000550d885280:   add	w29, w11, #0x1              ;*isub {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.VectorIntrinsics::checkFromIndexSize@42 (line 57)
                                                            ; - jdk.incubator.vector.IntVector::fromArray@9 (line 2953)
                                                            ; - AddTest::workload@17 (line 23)
  0x000000550d885284:   tbnz	w29, #31, 0x000000550d885560
 ;; B7: #	out( B31 B8 ) &lt;- in( B6 )  Freq: 4.99556
  0x000000550d885288:   ldr	w10, [sp, #8]
  0x000000550d88528c:   cmp	w10, w29
  0x000000550d885290:   b.cs	0x000000550d885504  // b.hs, b.nlast
 ;; B8: #	out( B44 B9 ) &lt;- in( B7 )  Freq: 4.99556
  0x000000550d885294:   mov	w29, w10                    ;*invokestatic checkIndex {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - java.util.Objects::checkIndex@3 (line 385)
                                                            ; - jdk.incubator.vector.VectorIntrinsics::checkFromIndexSize@43 (line 57)
                                                            ; - jdk.incubator.vector.IntVector::fromArray@9 (line 2953)
                                                            ; - AddTest::workload@17 (line 23)
  0x000000550d885298:   sxtw	x11, w10                    ;*i2l {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::arrayAddress@4 (line 3691)
                                                            ; - jdk.incubator.vector.IntVector::fromArray0Template@20 (line 3444)
                                                            ; - jdk.incubator.vector.Int256Vector::fromArray0@3 (line 859)
                                                            ; - jdk.incubator.vector.IntVector::fromArray@24 (line 2955)
                                                            ; - AddTest::workload@17 (line 23)
  0x000000550d88529c:   str	w10, [sp, #8]
  0x000000550d8852a0:   lsl	x10, x11, #2                ;*invokestatic store {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::intoArray@43 (line 3228)
                                                            ; - AddTest::workload@41 (line 25)
  0x000000550d8852a4:   add	x10, x10, #0x10             ;*ladd {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::arrayAddress@9 (line 3691)
                                                            ; - jdk.incubator.vector.IntVector::fromArray0Template@20 (line 3444)
                                                            ; - jdk.incubator.vector.Int256Vector::fromArray0@3 (line 859)
                                                            ; - jdk.incubator.vector.IntVector::fromArray@24 (line 2955)
                                                            ; - AddTest::workload@17 (line 23)
  0x000000550d8852a8:   str	x10, [sp, #24]
  0x000000550d8852ac:   mov	x4, x13
  0x000000550d8852b0:   mov	x5, x10
 ;; 0x557806ACB8
  0x000000550d8852b4:   mov	x10, #0x50f0                	// #20720
                                                            ;   {oop(a &apos;jdk/incubator/vector/IntVector$$Lambda$37+0x00000001000e3390&apos;{0x00000000c2e350f0})}
  0x000000550d8852b8:   movk	x10, #0xc2e3, lsl #16
  0x000000550d8852bc:   movk	x10, #0x0, lsl #32
 ;; 0x557806AC88
  0x000000550d8852c0:   mov	x1, #0x2758                	// #10072
                                                            ;   {oop(a &apos;java/lang/Class&apos;{0x00000000c2e12758} = &apos;jdk/incubator/vector/Int256Vector&apos;)}
  0x000000550d8852c4:   movk	x1, #0xc2e1, lsl #16
  0x000000550d8852c8:   movk	x1, #0x0, lsl #32
  0x000000550d8852cc:   str	x11, [sp, #16]
 ;; 0x557806AC98
  0x000000550d8852d0:   mov	x2, #0x980                 	// #2432
                                                            ;   {oop(a &apos;java/lang/Class&apos;{0x00000000c6000980} = int)}
  0x000000550d8852d4:   movk	x2, #0xc600, lsl #16
  0x000000550d8852d8:   movk	x2, #0x0, lsl #32
 ;; 0x8
  0x000000550d8852dc:   orr	w3, wzr, #0x8
  0x000000550d8852e0:   mov	x6, x4
  0x000000550d8852e4:   mov	x7, x11
 ;; 0x557806AC68
  0x000000550d8852e8:   mov	x0, #0x2710                	// #10000
                                                            ;   {oop(a &apos;jdk/incubator/vector/IntVector$IntSpecies&apos;{0x00000000c2e12710})}
  0x000000550d8852ec:   movk	x0, #0xc2e1, lsl #16
  0x000000550d8852f0:   movk	x0, #0x0, lsl #32
  0x000000550d8852f4:   str	x10, [sp]
  0x000000550d8852f8:   bl	0x000000550d885708          ; ImmutableOopMap {}
                                                            ;*invokestatic load {reexecute=0 rethrow=0 return_oop=1}
                                                            ; - jdk.incubator.vector.IntVector::fromArray0Template@32 (line 3442)
                                                            ; - jdk.incubator.vector.Int256Vector::fromArray0@3 (line 859)
                                                            ; - jdk.incubator.vector.IntVector::fromArray@24 (line 2955)
                                                            ; - AddTest::workload@17 (line 23)
                                                            ;   {static_call}
 ;; B9: #	out( B52 B10 ) &lt;- in( B8 )  Freq: 4.99546
  0x000000550d8852fc:   ldr	w11, [x0, #8]               ; implicit exception: dispatches to 0x000000550d88563c
 ;; B10: #	out( B36 B11 ) &lt;- in( B9 )  Freq: 4.99545
  0x000000550d885300:   lsl	x10, x11, #3
  0x000000550d885304:   ldr	x10, [x10, #96]
 ;; 0x1000D3E78
  0x000000550d885308:   mov	x11, #0x3e78                	// #15992
                                                            ;   {metadata(&apos;jdk/incubator/vector/IntVector&apos;)}
  0x000000550d88530c:   movk	x11, #0xd, lsl #16
  0x000000550d885310:   movk	x11, #0x1, lsl #32
  0x000000550d885314:   cmp	x10, x11
  0x000000550d885318:   b.ne	0x000000550d885578  // b.any;*checkcast {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::fromArray0Template@35 (line 3442)
                                                            ; - jdk.incubator.vector.Int256Vector::fromArray0@3 (line 859)
                                                            ; - jdk.incubator.vector.IntVector::fromArray@24 (line 2955)
                                                            ; - AddTest::workload@17 (line 23)
 ;; B11: #	out( B43 B12 ) &lt;- in( B10 )  Freq: 4.99545
 ;; 0x557806AC60
  0x000000550d88531c:   mov	x10, #0x2fd0                	// #12240
                                                            ;   {oop(a &apos;java/lang/Class&apos;{0x00000000c2e02fd0} = &apos;AddTest&apos;)}
  0x000000550d885320:   movk	x10, #0xc2e0, lsl #16
  0x000000550d885324:   movk	x10, #0x0, lsl #32
  0x000000550d885328:   str	x0, [sp, #32]               ;*invokestatic store {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::intoArray@43 (line 3228)
                                                            ; - AddTest::workload@41 (line 25)
  0x000000550d88532c:   ldr	w29, [x10, #120]            ;*getstatic b {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - AddTest::workload@24 (line 24)
 ;; 0x557806AC68
  0x000000550d885330:   mov	x1, #0x2710                	// #10000
                                                            ;   {oop(a &apos;jdk/incubator/vector/IntVector$IntSpecies&apos;{0x00000000c2e12710})}
  0x000000550d885334:   movk	x1, #0xc2e1, lsl #16
  0x000000550d885338:   movk	x1, #0x0, lsl #32
  0x000000550d88533c:   bl	0x000000550d885718          ; ImmutableOopMap {rfp=NarrowOop [32]=Oop }
                                                            ;*invokeinterface length {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::fromArray@2 (line 2953)
                                                            ; - AddTest::workload@28 (line 24)
                                                            ;   {optimized virtual_call}
 ;; B12: #	out( B53 B13 ) &lt;- in( B11 )  Freq: 4.99535
  0x000000550d885340:   mov	w13, w29
  0x000000550d885344:   ldr	w10, [x29, #12]             ; implicit exception: dispatches to 0x000000550d885650
 ;; B13: #	out( B37 B14 ) &lt;- in( B12 )  Freq: 4.99534
  0x000000550d885348:   sub	w11, w10, w0                ;*invokestatic store {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::intoArray@43 (line 3228)
                                                            ; - AddTest::workload@41 (line 25)
  0x000000550d88534c:   add	w29, w11, #0x1              ;*isub {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.VectorIntrinsics::checkFromIndexSize@42 (line 57)
                                                            ; - jdk.incubator.vector.IntVector::fromArray@9 (line 2953)
                                                            ; - AddTest::workload@28 (line 24)
  0x000000550d885350:   tbnz	w29, #31, 0x000000550d885590
 ;; B14: #	out( B32 B15 ) &lt;- in( B13 )  Freq: 4.99534
  0x000000550d885354:   ldr	w10, [sp, #8]
  0x000000550d885358:   cmp	w10, w29
  0x000000550d88535c:   b.cs	0x000000550d88551c  // b.hs, b.nlast
                                                            ;*invokestatic store {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::intoArray@43 (line 3228)
                                                            ; - AddTest::workload@41 (line 25)
 ;; B15: #	out( B42 B16 ) &lt;- in( B14 )  Freq: 4.99533
  0x000000550d885360:   mov	x4, x13                     ;*getstatic b {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - AddTest::workload@24 (line 24)
 ;; 0x557806ACB8
  0x000000550d885364:   mov	x10, #0x50f0                	// #20720
                                                            ;   {oop(a &apos;jdk/incubator/vector/IntVector$$Lambda$37+0x00000001000e3390&apos;{0x00000000c2e350f0})}
  0x000000550d885368:   movk	x10, #0xc2e3, lsl #16
  0x000000550d88536c:   movk	x10, #0x0, lsl #32
 ;; 0x557806AC88
  0x000000550d885370:   mov	x1, #0x2758                	// #10072
                                                            ;   {oop(a &apos;java/lang/Class&apos;{0x00000000c2e12758} = &apos;jdk/incubator/vector/Int256Vector&apos;)}
  0x000000550d885374:   movk	x1, #0xc2e1, lsl #16
  0x000000550d885378:   movk	x1, #0x0, lsl #32
 ;; 0x557806AC98
  0x000000550d88537c:   mov	x2, #0x980                 	// #2432
                                                            ;   {oop(a &apos;java/lang/Class&apos;{0x00000000c6000980} = int)}
  0x000000550d885380:   movk	x2, #0xc600, lsl #16
  0x000000550d885384:   movk	x2, #0x0, lsl #32
 ;; 0x8
  0x000000550d885388:   orr	w3, wzr, #0x8
  0x000000550d88538c:   ldr	x5, [sp, #24]
  0x000000550d885390:   mov	x6, x4
  0x000000550d885394:   ldr	x7, [sp, #16]
 ;; 0x557806AC68
  0x000000550d885398:   mov	x0, #0x2710                	// #10000
                                                            ;   {oop(a &apos;jdk/incubator/vector/IntVector$IntSpecies&apos;{0x00000000c2e12710})}
  0x000000550d88539c:   movk	x0, #0xc2e1, lsl #16
  0x000000550d8853a0:   movk	x0, #0x0, lsl #32
  0x000000550d8853a4:   str	x10, [sp]
  0x000000550d8853a8:   bl	0x000000550d885728          ; ImmutableOopMap {[32]=Oop }
                                                            ;*invokestatic load {reexecute=0 rethrow=0 return_oop=1}
                                                            ; - jdk.incubator.vector.IntVector::fromArray0Template@32 (line 3442)
                                                            ; - jdk.incubator.vector.Int256Vector::fromArray0@3 (line 859)
                                                            ; - jdk.incubator.vector.IntVector::fromArray@24 (line 2955)
                                                            ; - AddTest::workload@28 (line 24)
                                                            ;   {static_call}
 ;; B16: #	out( B54 B17 ) &lt;- in( B15 )  Freq: 4.99523
  0x000000550d8853ac:   ldr	w11, [x0, #8]               ; implicit exception: dispatches to 0x000000550d885664
 ;; B17: #	out( B38 B18 ) &lt;- in( B16 )  Freq: 4.99523
  0x000000550d8853b0:   lsl	x10, x11, #3
  0x000000550d8853b4:   ldr	x10, [x10, #96]
 ;; 0x1000D3E78
  0x000000550d8853b8:   mov	x11, #0x3e78                	// #15992
                                                            ;   {metadata(&apos;jdk/incubator/vector/IntVector&apos;)}
  0x000000550d8853bc:   movk	x11, #0xd, lsl #16
  0x000000550d8853c0:   movk	x11, #0x1, lsl #32
  0x000000550d8853c4:   cmp	x10, x11
  0x000000550d8853c8:   b.ne	0x000000550d8855a8  // b.any
 ;; B18: #	out( B41 B19 ) &lt;- in( B17 )  Freq: 4.99522
  0x000000550d8853cc:   mov	x3, x0                      ;*checkcast {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::fromArray0Template@35 (line 3442)
                                                            ; - jdk.incubator.vector.Int256Vector::fromArray0@3 (line 859)
                                                            ; - jdk.incubator.vector.IntVector::fromArray@24 (line 2955)
                                                            ; - AddTest::workload@28 (line 24)
  0x000000550d8853d0:   ldr	x1, [sp, #32]               ;*invokestatic store {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::intoArray@43 (line 3228)
                                                            ; - AddTest::workload@41 (line 25)
 ;; 0x557806ACC8
  0x000000550d8853d4:   mov	x2, #0x8c80                	// #35968
                                                            ;   {oop(a &apos;jdk/incubator/vector/VectorOperators$AssociativeImpl&apos;{0x00000000c2eb8c80})}
  0x000000550d8853d8:   movk	x2, #0xc2eb, lsl #16
  0x000000550d8853dc:   movk	x2, #0x0, lsl #32
 ;; 0xFFFFFFFFFFFF
  0x000000550d8853e0:   mov	x9, #0xffff                	// #65535
  0x000000550d8853e4:   movk	x9, #0xffff, lsl #16
  0x000000550d8853e8:   movk	x9, #0xffff, lsl #32
  0x000000550d8853ec:   bl	0x000000550d885738          ; ImmutableOopMap {}
                                                            ;*invokevirtual lanewise {reexecute=0 rethrow=0 return_oop=1}
                                                            ; - jdk.incubator.vector.IntVector::add@5 (line 1352)
                                                            ; - AddTest::workload@34 (line 25)
                                                            ;   {virtual_call}
 ;; B19: #	out( B33 B20 ) &lt;- in( B18 )  Freq: 4.99512
 ;; 0x557806AC60
  0x000000550d8853f0:   mov	x10, #0x2fd0                	// #12240
                                                            ;   {oop(a &apos;java/lang/Class&apos;{0x00000000c2e02fd0} = &apos;AddTest&apos;)}
  0x000000550d8853f4:   movk	x10, #0xc2e0, lsl #16
  0x000000550d8853f8:   movk	x10, #0x0, lsl #32
  0x000000550d8853fc:   ldr	w29, [x10, #124]            ;*getstatic c {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - AddTest::workload@37 (line 25)
  0x000000550d885400:   cbz	x0, 0x000000550d885534
 ;; B20: #	out( B46 B21 ) &lt;- in( B19 )  Freq: 4.99512
  0x000000550d885404:   str	x0, [sp, #32]
  0x000000550d885408:   mov	x1, x0
 ;; 0xFFFFFFFFFFFF
  0x000000550d88540c:   mov	x9, #0xffff                	// #65535
  0x000000550d885410:   movk	x9, #0xffff, lsl #16
  0x000000550d885414:   movk	x9, #0xffff, lsl #32
  0x000000550d885418:   bl	0x000000550d885748          ; ImmutableOopMap {rfp=NarrowOop [32]=Oop }
                                                            ;*invokevirtual length {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::intoArray@2 (line 3226)
                                                            ; - AddTest::workload@41 (line 25)
                                                            ;   {virtual_call}
 ;; B21: #	out( B55 B22 ) &lt;- in( B20 )  Freq: 4.99502
  0x000000550d88541c:   mov	w12, w29
  0x000000550d885420:   ldr	w10, [x29, #12]             ; implicit exception: dispatches to 0x000000550d885678
 ;; B22: #	out( B39 B23 ) &lt;- in( B21 )  Freq: 4.99501
  0x000000550d885424:   sub	w11, w10, w0                ;*invokestatic store {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::intoArray@43 (line 3228)
                                                            ; - AddTest::workload@41 (line 25)
  0x000000550d885428:   add	w29, w11, #0x1              ;*isub {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.VectorIntrinsics::checkFromIndexSize@42 (line 57)
                                                            ; - jdk.incubator.vector.IntVector::intoArray@7 (line 3226)
                                                            ; - AddTest::workload@41 (line 25)
  0x000000550d88542c:   tbnz	w29, #31, 0x000000550d8855c0
 ;; B23: #	out( B34 B24 ) &lt;- in( B22 )  Freq: 4.99501
  0x000000550d885430:   ldr	w11, [sp, #8]
  0x000000550d885434:   cmp	w11, w29
  0x000000550d885438:   b.cs	0x000000550d885548  // b.hs, b.nlast
                                                            ;*invokestatic store {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::intoArray@43 (line 3228)
                                                            ; - AddTest::workload@41 (line 25)
 ;; B24: #	out( B47 B25 ) &lt;- in( B23 )  Freq: 4.995
  0x000000550d88543c:   mov	x10, x12                    ;*getstatic c {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - AddTest::workload@37 (line 25)
  0x000000550d885440:   mov	w29, w11
  0x000000550d885444:   str	x10, [sp, #8]
  0x000000550d885448:   ldr	x1, [sp, #32]
 ;; 0xFFFFFFFFFFFF
  0x000000550d88544c:   mov	x9, #0xffff                	// #65535
  0x000000550d885450:   movk	x9, #0xffff, lsl #16
  0x000000550d885454:   movk	x9, #0xffff, lsl #32
  0x000000550d885458:   bl	0x000000550d885758          ; ImmutableOopMap {[8]=Oop [32]=Oop }
                                                            ;*invokevirtual vspecies {reexecute=0 rethrow=0 return_oop=1}
                                                            ; - jdk.incubator.vector.IntVector::intoArray@12 (line 3227)
                                                            ; - AddTest::workload@41 (line 25)
                                                            ;   {virtual_call}
 ;; B25: #	out( B56 B26 ) &lt;- in( B24 )  Freq: 4.9949
  0x000000550d88545c:   ldr	w11, [x0, #44]              ; implicit exception: dispatches to 0x000000550d88568c
                                                            ;*invokestatic store {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::intoArray@43 (line 3228)
                                                            ; - AddTest::workload@41 (line 25)
 ;; B26: #	out( B48 B27 ) &lt;- in( B25 )  Freq: 4.9949
  0x000000550d885460:   ldr	w3, [x0, #12]               ;*getfield laneCount {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.AbstractSpecies::laneCount@1 (line 126)
                                                            ; - jdk.incubator.vector.IntVector::intoArray@25 (line 3229)
                                                            ; - AddTest::workload@41 (line 25)
 ;; 0x557806ACD8
  0x000000550d885464:   mov	x10, #0xc690                	// #50832
                                                            ;   {oop(a &apos;jdk/incubator/vector/IntVector$$Lambda$42+0x00000001000eccc8&apos;{0x00000000c2ddc690})}
  0x000000550d885468:   movk	x10, #0xc2dd, lsl #16
  0x000000550d88546c:   movk	x10, #0x0, lsl #32
  0x000000550d885470:   ldr	x4, [sp, #8]                ;*invokestatic store {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::intoArray@43 (line 3228)
                                                            ; - AddTest::workload@41 (line 25)
  0x000000550d885474:   lsr	x1, x11, #0                 ;*getfield vectorType {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector$IntSpecies::vectorType@1 (line 3833)
                                                            ; - jdk.incubator.vector.IntVector::intoArray@17 (line 3229)
                                                            ; - AddTest::workload@41 (line 25)
 ;; 0x557806AC98
  0x000000550d885478:   mov	x2, #0x980                 	// #2432
                                                            ;   {oop(a &apos;java/lang/Class&apos;{0x00000000c6000980} = int)}
  0x000000550d88547c:   movk	x2, #0xc600, lsl #16
  0x000000550d885480:   movk	x2, #0x0, lsl #32
 ;; merged ldr pair
  0x000000550d885484:   ldp	x5, x6, [sp, #24]
  0x000000550d885488:   mov	x7, x4
  0x000000550d88548c:   ldr	x0, [sp, #16]
  0x000000550d885490:   str	x10, [sp]
  0x000000550d885494:   bl	0x000000550d885768          ; ImmutableOopMap {}
                                                            ;*invokestatic store {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::intoArray@43 (line 3228)
                                                            ; - AddTest::workload@41 (line 25)
                                                            ;   {static_call}
 ;; B27: #	out( B40 B28 ) &lt;- in( B26 )  Freq: 4.9948
 ;; 0x557806AC68
  0x000000550d885498:   mov	x1, #0x2710                	// #10000
                                                            ;   {oop(a &apos;jdk/incubator/vector/IntVector$IntSpecies&apos;{0x00000000c2e12710})}
  0x000000550d88549c:   movk	x1, #0xc2e1, lsl #16
  0x000000550d8854a0:   movk	x1, #0x0, lsl #32
  0x000000550d8854a4:   bl	0x000000550d885778          ; ImmutableOopMap {}
                                                            ;*invokeinterface length {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - AddTest::workload@48 (line 22)
                                                            ;   {optimized virtual_call}
 ;; B28: #	out( B57 B29 ) &lt;- in( B27 )  Freq: 4.9947
  0x000000550d8854a8:   ldr	x10, [x28, #960]            ;*invokestatic store {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::intoArray@43 (line 3228)
                                                            ; - AddTest::workload@41 (line 25)
  0x000000550d8854ac:   add	w12, w0, w29                ; ImmutableOopMap {}
                                                            ;*goto {reexecute=1 rethrow=0 return_oop=0}
                                                            ; - (reexecute) AddTest::workload@55 (line 22)
  0x000000550d8854b0:   ldr	wzr, [x10]                  ;   {poll}
  0x000000550d8854b4:   ldrb	w8, [x28, #984]
  0x000000550d8854b8:   cbz	x8, 0x000000550d8854d0
 ;; 0x5503982790
  0x000000550d8854bc:   mov	x8, #0x2790                	// #10128
                                                            ;   {external_word}
  0x000000550d8854c0:   movk	x8, #0x398, lsl #16
  0x000000550d8854c4:   movk	x8, #0x55, lsl #32
  0x000000550d8854c8:   mov	x0, x28
  0x000000550d8854cc:   blr	x8                          ;*invokestatic store {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::intoArray@43 (line 3228)
                                                            ; - AddTest::workload@41 (line 25)
 ;; 0x557806AC60
  0x000000550d8854d0:   mov	x10, #0x2fd0                	// #12240
                                                            ;   {oop(a &apos;java/lang/Class&apos;{0x00000000c2e02fd0} = &apos;AddTest&apos;)}
  0x000000550d8854d4:   movk	x10, #0xc2e0, lsl #16
  0x000000550d8854d8:   movk	x10, #0x0, lsl #32
  0x000000550d8854dc:   ldr	w29, [x10, #116]            ;*getstatic a {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - AddTest::workload@3 (line 22)
  0x000000550d8854e0:   ldr	w10, [x29, #12]             ; implicit exception: dispatches to 0x000000550d8856a0
 ;; B29: #	out( B4 B30 ) &lt;- in( B28 )  Freq: 4.99469
  0x000000550d8854e4:   cmp	w12, w10                    ;*invokestatic store {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::intoArray@43 (line 3228)
                                                            ; - AddTest::workload@41 (line 25)
  0x000000550d8854e8:   b.lt	0x000000550d885260  // b.tstop;*if_icmpge {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - AddTest::workload@7 (line 22)
 ;; B30: #	out( N1 ) &lt;- in( B29 B2 )  Freq: 0.999469
  0x000000550d8854ec:   ldp	x29, x30, [sp, #64]
  0x000000550d8854f0:   add	sp, sp, #0x50
  0x000000550d8854f4:   ldr	x8, [x28, #952]             ;   {poll_return}
  0x000000550d8854f8:   cmp	sp, x8
  0x000000550d8854fc:   b.hi	0x000000550d8856c8  // b.pmore
  0x000000550d885500:   ret
 ;; B31: #	out( N1 ) &lt;- in( B7 )  Freq: 5.0619e-06
 ;; 0xFFFFFFE4
  0x000000550d885504:   mov	w1, #0xffffffe4            	// #-28
  0x000000550d885508:   str	w13, [sp, #12]
  0x000000550d88550c:   bl	0x000000550d885808          ; ImmutableOopMap {[12]=NarrowOop }
                                                            ;*invokestatic checkIndex {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - java.util.Objects::checkIndex@3 (line 385)
                                                            ; - jdk.incubator.vector.VectorIntrinsics::checkFromIndexSize@43 (line 57)
                                                            ; - jdk.incubator.vector.IntVector::fromArray@9 (line 2953)
                                                            ; - AddTest::workload@17 (line 23)
                                                            ;   {runtime_call UncommonTrapBlob}
 ;; uncommon trap returned which should never happen
  0x000000550d885510:   dcps1	#0xdeae
  0x000000550d885514:   .inst	0x0467bf68 ; undefined
  0x000000550d885518:   udf	#85
 ;; B32: #	out( N1 ) &lt;- in( B14 )  Freq: 5.06167e-06
  0x000000550d88551c:   str	w13, [sp, #12]
 ;; 0xFFFFFFE4
  0x000000550d885520:   mov	w1, #0xffffffe4            	// #-28
  0x000000550d885524:   bl	0x000000550d885808          ; ImmutableOopMap {[12]=NarrowOop [32]=Oop }
                                                            ;*invokestatic checkIndex {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - java.util.Objects::checkIndex@3 (line 385)
                                                            ; - jdk.incubator.vector.VectorIntrinsics::checkFromIndexSize@43 (line 57)
                                                            ; - jdk.incubator.vector.IntVector::fromArray@9 (line 2953)
                                                            ; - AddTest::workload@28 (line 24)
                                                            ;   {runtime_call UncommonTrapBlob}
 ;; uncommon trap returned which should never happen
  0x000000550d885528:   dcps1	#0xdeae
  0x000000550d88552c:   .inst	0x0467bf68 ; undefined
  0x000000550d885530:   udf	#85
 ;; B33: #	out( N1 ) &lt;- in( B19 )  Freq: 5.06145e-06
 ;; 0xFFFFFFF6
  0x000000550d885534:   mov	w1, #0xfffffff6            	// #-10
  0x000000550d885538:   bl	0x000000550d885808          ; ImmutableOopMap {rfp=NarrowOop }
                                                            ;*invokevirtual intoArray {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - AddTest::workload@41 (line 25)
                                                            ;   {runtime_call UncommonTrapBlob}
 ;; uncommon trap returned which should never happen
  0x000000550d88553c:   dcps1	#0xdeae
  0x000000550d885540:   .inst	0x0467bf68 ; undefined
  0x000000550d885544:   udf	#85
 ;; B34: #	out( N1 ) &lt;- in( B23 )  Freq: 5.06134e-06
  0x000000550d885548:   str	w12, [sp, #12]
 ;; 0xFFFFFFE4
  0x000000550d88554c:   mov	w1, #0xffffffe4            	// #-28
  0x000000550d885550:   bl	0x000000550d885808          ; ImmutableOopMap {[12]=NarrowOop [32]=Oop }
                                                            ;*invokestatic checkIndex {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - java.util.Objects::checkIndex@3 (line 385)
                                                            ; - jdk.incubator.vector.VectorIntrinsics::checkFromIndexSize@43 (line 57)
                                                            ; - jdk.incubator.vector.IntVector::intoArray@7 (line 3226)
                                                            ; - AddTest::workload@41 (line 25)
                                                            ;   {runtime_call UncommonTrapBlob}
 ;; uncommon trap returned which should never happen
  0x000000550d885554:   dcps1	#0xdeae
  0x000000550d885558:   .inst	0x0467bf68 ; undefined
  0x000000550d88555c:   udf	#85
 ;; B35: #	out( N1 ) &lt;- in( B6 )  Freq: 4.99557e-06
 ;; 0xFFFFFFCC
  0x000000550d885560:   mov	w1, #0xffffffcc            	// #-52
  0x000000550d885564:   str	w13, [sp, #12]
  0x000000550d885568:   bl	0x000000550d885808          ; ImmutableOopMap {[12]=NarrowOop }
                                                            ;*invokestatic checkIndex {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - java.util.Objects::checkIndex@3 (line 385)
                                                            ; - jdk.incubator.vector.VectorIntrinsics::checkFromIndexSize@43 (line 57)
                                                            ; - jdk.incubator.vector.IntVector::fromArray@9 (line 2953)
                                                            ; - AddTest::workload@17 (line 23)
                                                            ;   {runtime_call UncommonTrapBlob}
 ;; uncommon trap returned which should never happen
  0x000000550d88556c:   dcps1	#0xdeae
  0x000000550d885570:   .inst	0x0467bf68 ; undefined
  0x000000550d885574:   udf	#85
 ;; B36: #	out( N1 ) &lt;- in( B10 )  Freq: 4.99545e-06
  0x000000550d885578:   mov	x29, x0
 ;; 0xFFFFFFDE
  0x000000550d88557c:   mov	w1, #0xffffffde            	// #-34
  0x000000550d885580:   bl	0x000000550d885808          ; ImmutableOopMap {rfp=Oop }
                                                            ;*checkcast {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::fromArray0Template@35 (line 3442)
                                                            ; - jdk.incubator.vector.Int256Vector::fromArray0@3 (line 859)
                                                            ; - jdk.incubator.vector.IntVector::fromArray@24 (line 2955)
                                                            ; - AddTest::workload@17 (line 23)
                                                            ;   {runtime_call UncommonTrapBlob}
 ;; uncommon trap returned which should never happen
  0x000000550d885584:   dcps1	#0xdeae
  0x000000550d885588:   .inst	0x0467bf68 ; undefined
  0x000000550d88558c:   udf	#85
 ;; B37: #	out( N1 ) &lt;- in( B13 )  Freq: 4.99534e-06
  0x000000550d885590:   str	w13, [sp, #12]
 ;; 0xFFFFFFCC
  0x000000550d885594:   mov	w1, #0xffffffcc            	// #-52
  0x000000550d885598:   bl	0x000000550d885808          ; ImmutableOopMap {[12]=NarrowOop [32]=Oop }
                                                            ;*invokestatic checkIndex {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - java.util.Objects::checkIndex@3 (line 385)
                                                            ; - jdk.incubator.vector.VectorIntrinsics::checkFromIndexSize@43 (line 57)
                                                            ; - jdk.incubator.vector.IntVector::fromArray@9 (line 2953)
                                                            ; - AddTest::workload@28 (line 24)
                                                            ;   {runtime_call UncommonTrapBlob}
 ;; uncommon trap returned which should never happen
  0x000000550d88559c:   dcps1	#0xdeae
  0x000000550d8855a0:   .inst	0x0467bf68 ; undefined
  0x000000550d8855a4:   udf	#85
 ;; B38: #	out( N1 ) &lt;- in( B17 )  Freq: 4.99523e-06
  0x000000550d8855a8:   mov	x29, x0
 ;; 0xFFFFFFDE
  0x000000550d8855ac:   mov	w1, #0xffffffde            	// #-34
  0x000000550d8855b0:   bl	0x000000550d885808          ; ImmutableOopMap {rfp=Oop }
                                                            ;*checkcast {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::fromArray0Template@35 (line 3442)
                                                            ; - jdk.incubator.vector.Int256Vector::fromArray0@3 (line 859)
                                                            ; - jdk.incubator.vector.IntVector::fromArray@24 (line 2955)
                                                            ; - AddTest::workload@28 (line 24)
                                                            ;   {runtime_call UncommonTrapBlob}
 ;; uncommon trap returned which should never happen
  0x000000550d8855b4:   dcps1	#0xdeae
  0x000000550d8855b8:   .inst	0x0467bf68 ; undefined
  0x000000550d8855bc:   udf	#85
 ;; B39: #	out( N1 ) &lt;- in( B22 )  Freq: 4.99501e-06
  0x000000550d8855c0:   str	w12, [sp, #12]
 ;; 0xFFFFFFCC
  0x000000550d8855c4:   mov	w1, #0xffffffcc            	// #-52
  0x000000550d8855c8:   bl	0x000000550d885808          ; ImmutableOopMap {[12]=NarrowOop [32]=Oop }
                                                            ;*invokestatic checkIndex {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - java.util.Objects::checkIndex@3 (line 385)
                                                            ; - jdk.incubator.vector.VectorIntrinsics::checkFromIndexSize@43 (line 57)
                                                            ; - jdk.incubator.vector.IntVector::intoArray@7 (line 3226)
                                                            ; - AddTest::workload@41 (line 25)
                                                            ;   {runtime_call UncommonTrapBlob}
 ;; uncommon trap returned which should never happen
  0x000000550d8855cc:   dcps1	#0xdeae
  0x000000550d8855d0:   .inst	0x0467bf68 ; undefined
  0x000000550d8855d4:   udf	#85                         ;*invokeinterface length {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - AddTest::workload@48 (line 22)
 ;; B40: #	out( B50 ) &lt;- in( B27 )  Freq: 4.9948e-05
  0x000000550d8855d8:   mov	x1, x0
  0x000000550d8855dc:   b	0x000000550d885614          ;*invokevirtual lanewise {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::add@5 (line 1352)
                                                            ; - AddTest::workload@34 (line 25)
```



##### After using -XX:UseSVE=2 to turn on SVE, some of the C2 logs are as follows:

```
;; B10: #	out( B32 B11 ) &lt;- in( B9 )  Freq: 4.99772
  0x000000550d889104:   sub	w10, w11, w0                ;*synchronization entry
                                                            ; - jdk.incubator.vector.IntVector::intoArray@-1 (line 3226)
                                                            ; - AddTest::workload@41 (line 25)
  0x000000550d889108:   mov	x11, x29                    ;*getstatic b {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - AddTest::workload@24 (line 24)
  0x000000550d88910c:   ldr	x15, [sp, #24]
  0x000000550d889110:   add	x12, x11, x15               ;*synchronization entry
                                                            ; - jdk.incubator.vector.IntVector::intoArray@-1 (line 3226)
                                                            ; - AddTest::workload@41 (line 25)
  0x000000550d889114:   add	w29, w10, #0x1              ;*isub {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.VectorIntrinsics::checkFromIndexSize@42 (line 57)
                                                            ; - jdk.incubator.vector.IntVector::fromArray@9 (line 2953)
                                                            ; - AddTest::workload@28 (line 24)
  0x000000550d889118:   tbnz	w29, #31, 0x000000550d8893e0
 ;; B11: #	out( B29 B12 ) &lt;- in( B10 )  Freq: 4.99772
  0x000000550d88911c:   ldr	w13, [sp, #8]
  0x000000550d889120:   cmp	w13, w29
  0x000000550d889124:   b.cs	0x000000550d88938c  // b.hs, b.nlast
 ;; B12: #	out( B24 B13 ) &lt;- in( B11 )  Freq: 4.99771
  0x000000550d889128:   add	x10, x12, #0x10
  0x000000550d88912c:   ldr	p0, [sp, #12, mul vl]       ;*synchronization entry
                                                            ; - jdk.incubator.vector.IntVector::intoArray@-1 (line 3226)
                                                            ; - AddTest::workload@41 (line 25)
  0x000000550d889130:   ld1w	{z17.s}, p0/z, [x10]        ;*invokestatic load {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::fromArray0Template@32 (line 3442)
                                                            ; - jdk.incubator.vector.Int256Vector::fromArray0@3 (line 859)
                                                            ; - jdk.incubator.vector.IntVector::fromArray@24 (line 2955)
                                                            ; - AddTest::workload@28 (line 24)
  0x000000550d889134:   ldr	x14, [x28, #264]
  0x000000550d889138:   ldr	x10, [x28, #280]
  0x000000550d88913c:   add	x11, x14, #0x30
  0x000000550d889140:   cmp	x11, x10
  0x000000550d889144:   b.cs	0x000000550d8892dc  // b.hs, b.nlast
 ;; B13: #	out( B14 ) &lt;- in( B12 )  Freq: 4.99721
  0x000000550d889148:   str	x11, [x28, #264]
 ;; 0x1
  0x000000550d88914c:   orr	x10, xzr, #0x1
  0x000000550d889150:   str	x10, [x14]
  0x000000550d889154:   prfm	pstl1keep, [x11, #96]
  0x000000550d889158:   mov	x12, #0x20000000            	// #536870912
                                                            ;   {metadata({type array int})}
  0x000000550d88915c:   movk	x12, #0x81b9
  0x000000550d889160:   str	w12, [x14, #8]
  0x000000550d889164:   prfm	pstl1keep, [x11, #128]
 ;; 0x8
  0x000000550d889168:   orr	w10, wzr, #0x8
  0x000000550d88916c:   str	w10, [x14, #12]
  0x000000550d889170:   prfm	pstl1keep, [x11, #160]
  0x000000550d889174:   add	x10, x14, #0x10
 ;; zero_words (count = 4) {
  0x000000550d889178:   stp	xzr, xzr, [x10]
  0x000000550d88917c:   stp	xzr, xzr, [x10, #16]
 ;; } zero_words
 ;; B14: #	out( B26 B15 ) &lt;- in( B25 B13 )  Freq: 4.99771
  0x000000550d889180:   dmb	ishst
  0x000000550d889184:   add	x9, sp, #0x20
  0x000000550d889188:   ldr	z16, [x9]                   ;*synchronization entry
                                                            ; - jdk.incubator.vector.IntVector::intoArray@-1 (line 3226)
                                                            ; - AddTest::workload@41 (line 25)
  0x000000550d88918c:   add	z18.s, z16.s, z17.s         ;*invokestatic binaryOp {reexecute=0 rethrow=0 return_oop=0}
                                                            ; - jdk.incubator.vector.IntVector::lanewiseTemplate@154 (line 784)
                                                            ; - jdk.incubator.vector.Int256Vector::lanewise@3 (line 285)
                                                            ; - jdk.incubator.vector.Int256Vector::lanewise@3 (line 41)
                                                            ; - jdk.incubator.vector.IntVector::add@5 (line 1352)
                                                            ; - AddTest::workload@34 (line 25)
  0x000000550d889190:   add	x10, x14, #0x10
  0x000000550d889194:   st1w	{z18.s}, p0, [x10]
  0x000000550d889198:   ldr	x6, [x28, #264]
  0x000000550d88919c:   ldr	x10, [x28, #280]
  0x000000550d8891a0:   add	x11, x6, #0x10
  0x000000550d8891a4:   cmp	x11, x10
  0x000000550d8891a8:   b.cs	0x000000550d889320  // b.hs, b.nlast
 ;; B15: #	out( B16 ) &lt;- in( B14 )  Freq: 4.99721
  0x000000550d8891ac:   str	x11, [x28, #264]
 ;; 0x1
  0x000000550d8891b0:   orr	x10, xzr, #0x1
  0x000000550d8891b4:   str	x10, [x6]
  0x000000550d8891b8:   mov	x10, #0x20010000            	// #536936448
                                                            ;   {metadata(&apos;jdk/incubator/vector/Int256Vector&apos;)}
  0x000000550d8891bc:   movk	x10, #0xb8cd
  0x000000550d8891c0:   str	w10, [x6, #8]
  0x000000550d8891c4:   prfm	pstl1keep, [x11, #96]
 ;; B16: #	out( B36 B17 ) &lt;- in( B27 B15 )  Freq: 4.99771
  0x000000550d8891c8:   cbnz	x14, 0x000000550d8891d8
 ;; null oop passed to encode_heap_oop_not_null2
  0x000000550d8891cc:   dcps1	#0xdeae
  0x000000550d8891d0:   sqinch	x16, w16, vl3, mul #16
  0x000000550d8891d4:   udf	#85
  0x000000550d8891d8:   mov	x11, x14
  0x000000550d8891dc:   str	w11, [x6, #12]
  0x000000550d8891e0:   dmb	ishst
 ;; 0x5578072C70
  0x000000550d8891e4:   mov	x10, #0x31a8                	// #12712
                                                            ;   {oop(a &apos;java/lang/Class&apos;{0x00000000c2e031a8} = &apos;AddTest&apos;)}
  0x000000550d8891e8:   movk	x10, #0xc2e0, lsl #16
  0x000000550d8891ec:   movk	x10, #0x0, lsl #32
  0x000000550d8891f0:   ldr	w12, [x10, #124]
  0x000000550d8891f4:   ldr	w10, [x12, #12]             ; implicit exception: dispatches to 0x000000550d889444
                                                            ;*synchronization entry
                                                            ; - jdk.incubator.vector.IntVector::intoArray@-1 (line 3226)
                                                            ; - AddTest::workload@41 (line 25)
```

With sve256=on, loadV_masked V17, P0, [R10] can load as much as 256/8 data into the vector registers at once when the data type is int/32 bits (consistent with SPECIES.length() in the java code above).



#### Statistics on the number of instructions compiled for the AddTest#workload method

Loop test without SVE enabled: 740 instructions for the outer loop, 30 instructions for the inner loop

SVE-enabled loop test: single loop with approximately 430 instructions

| Total data amount | Loop test without SVE enabled                                | Loop test with SVE enabled                                   |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1024              | Need outer loop 1024/8 = 128 times, need inner loop 8 times, need instruction number 128 * 740 + 8 * 30 = 94720 + 240 = 94960 | Need to loop 1024/8 = 128 times, need 128 * 430 = 55040 instructions |
| 2056              | Needs 2048/8 = 256 outer loops, 8 more inner loops, 256 * 740 + 8 * 30 = 189440 + 240 = 189680 | The number of cycles required is 2048/8 = 256, and the number of instructions required is 256 * 430 = 110080 |

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