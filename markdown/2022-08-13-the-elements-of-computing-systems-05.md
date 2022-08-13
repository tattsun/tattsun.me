---
title: コンピュータシステムの理論と実装　第５章　コンピュータアーキテクチャ
---

最近、[コンピュータシステムの理論と実装](https://www.oreilly.co.jp/books/9784873117126/)を読みながら手を動かしている。

ブログの試運用も兼ねて読書ログを記録しておく。

![Imgur](https://i.imgur.com/4eUxFS6.png)

## ソースコード

一応以下のソースでテストは通ったが、CPUの比較周りのコードがかなり煩雑になってしまった…。

### Memory.hdl

```hdl
CHIP Memory {
    IN in[16], load, address[15];
    OUT out[16];

    PARTS:
    DMux(in=load, sel=address[14], a=loadram, b=loadio);
    DMux(in=loadio, sel=address[13], a=loadscreen);

    RAM16K(in=in, load=loadram, address=address[0..13], out=outram);
    Screen(in=in, load=loadscreen, address=address[0..12], out=outscreen);
    Keyboard(out=outkeyboard);

    Mux16(a=outscreen, b=outkeyboard, sel=address[13], out=outio);
    Mux16(a=outram, b=outio, sel=address[14], out=out);
}
```

### CPU.hdl

```hdl
CHIP CPU {

    IN  inM[16],         // M value input  (M = contents of RAM[A])
        instruction[16], // Instruction for execution
        reset;           // Signals whether to re-start the current
                         // program (reset==1) or continue executing
                         // the current program (reset==0).

    OUT outM[16],        // M value output
        writeM,          // Write to M? 
        addressM[15],    // Address in data memory (of M)
        pc[15];          // address of next instruction

    PARTS:
    Mux16(a=instruction, b=outALU, sel=instruction[15], out=inA);
    Not(in=instruction[15], out=instA);

    Or(a=instA, b=instruction[5], out=saveA);
    ARegister(in=inA, load=saveA, out=outA, out[0..14]=addressM);

    Mux16(a=outA, b=inM, sel=instruction[12], out=outAorM);

    And(a=instruction[4], b=instruction[15], out=loadD);
    DRegister(in=outALU, load=loadD, out=outD);

    ALU(x=outD, y=outAorM, zx=instruction[11], nx=instruction[10], zy=instruction[9], ny=instruction[8], f=instruction[7], no=instruction[6], out=outM, out=outALU, zr=zr, ng=ng);

    And(a=instruction[15], b=instruction[3], out=writeM);

    // Compare
    Not(in=zr, out=notzr);
    Not(in=ng, out=notng);

    And(a=notzr, b=notng, out=jgt);
    And(a=zr, b=true, out=jeq);
    Or(a=zr, b=notng, out=jge);
    And(a=notzr, b=ng, out=jlt);
    And(a=notzr, b=true, out=jne);
    Or(a=zr, b=ng, out=jle);


    DMux8Way(in=true, sel=instruction[0..2], a=Inojmp, b=Ijgt, c=Ijeq, d=Ijge, e=Ijlt, f=Ijne, g=Ijle, h=Ijmp);

    And(a=jgt, b=Ijgt, out=Rjgt);
    And(a=jeq, b=Ijeq, out=Rjeq);
    And(a=jge, b=Ijge, out=Rjge);
    And(a=jlt, b=Ijlt, out=Rjlt);
    And(a=jne, b=Ijne, out=Rjne);
    And(a=jle, b=Ijle, out=Rjle);
    
    Or(a=Rjgt, b=Rjeq, out=w1);
    Or(a=w1, b=Rjge, out=w2);
    Or(a=w2, b=Rjlt, out=w3);
    Or(a=w3, b=Rjne, out=w4);
    Or(a=w4, b=Rjle, out=w5);
    Or(a=w5, b=Rjgt, out=w6);
    Or(a=w6, b=Ijmp, out=w7);
    And(a=w7, b=instruction[15], out=loadPC);
    Not(in=loadPC, out=notLoadPC);
    
    Or(a=Inojmp, b=instA, out=x1);
    Or(a=x1, b=notLoadPC, out=Rnojmp);

    PC(in=outA, load=loadPC, inc=Rnojmp, reset=reset, out[0..14]=pc);
}
```

### Computer.hdl

```hdl
CHIP Computer {

    IN reset;

    PARTS:
    
    ROM32K(address=pc, out=instruction);
    CPU(inM=inM, instruction=instruction, reset=reset, outM=outM, writeM=writeM, addressM=addressM, pc=pc);
    Memory(in=outM, load=writeM, address=addressM, out=inM);
}
```