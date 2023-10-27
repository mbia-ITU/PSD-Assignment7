# Exercises 8

## 8.1
 
### 8.1.i

We have lexed, parsed, compiled and run ex11.

```{}
1 5 8 6 3 7 2 4
1 6 8 3 7 4 2 5
...
8 3 1 6 2 5 7 4
8 4 1 3 6 2 7 5

Ran 0.096 seconds
```

### 8.1.ii

```{}
> compile "ex3";;
val it: Machine.instr list =
    [LDARGS;                    // add the the commandline args to the stack
    CALL (1, "L1");             // starts the function
    STOP;                       // halts the program when it finishes.
    Label "L1";
        INCSP 1;                // int i;, to declare the local variable
        GETBP; CSTI 1; ADD;     // get the address of i, the i part of: i=0;
        CSTI 0;                 // places 0 on the stack, the 0 part of: i=0;
        STI;                    // assigns 0 to i, to finish i=0;
        INCSP -1;               // clears the stack, no C-code
        GOTO "L3";              // jumps to the while loop
    Label "L2";
        GETBP; CSTI 1; ADD;     //find addr of i, to do print i;
        LDI;                    //load i onto the stack 
        PRINTI;                 //print i;
        INCSP -1;               //remove the copy of i, at the top of the stack
        GETBP; CSTI 1; ADD;     //get the addr of the first i, to do i=i+1;
        GETBP; CSTI 1; ADD;     //get the addr of i, to do i=i+1;
        LDI;                    //load the second i onto the stack
        CSTI 1; ADD;            //add 1 to the element on the top of the stack, to do i+1
        STI;                    //store the result on the stack
        INCSP -1;               //removed the result from STI
        INCSP 0;                //zero local vars were created, and therefore it removes zero elements from the stack         
    Label "L3";                 
        GETBP; CSTI 1; ADD;     // gets location of i, the 1'th local var, the i part of i < n
        LDI;                    // loads i onto the stack
        GETBP; CSTI 0; ADD;     // gets location of n, the 0'th local var, the n part of i < n
        LDI;                    // loads n onto the stack
        LT;                     // compares the top two elements of the stack, the < part of i < n
        IFNZRO "L2";            // jumps to L2, if lt was not zero
        INCSP -1;               // shrinks the stack, to removes the result of the comparison from the stack
        RET 0
    ]
```

```{}
java Machinetrace ex3.out 4 > ex3.trace

[ ]{0: LDARGS}
[ 4 ]{1: CALL 1 5}
[ 4 -999 4 ]{5: INCSP 1}
[ 4 -999 4 0 ]{7: GETBP}
[ 4 -999 4 0 2 ]{8: CSTI 1}
...
[ 4 -999 4 4 0 ]{54: IFNZRO 18}
[ 4 -999 4 4 ]{56: INCSP -1}
[ 4 -999 4 ]{58: RET 0}
[ 4 ]{4: STOP}
```

The above trace, is taken directly from the ex3.trace file.
It is not annotated, because the TA's have expressed that the bytecode is a sufficient answer.

```{}
...MicroC>java Machine ex3.out 10
0 1 2 3 4 5 6 7 8 9
Ran 0.013 seconds
```

```{}
> compile "ex5";;
val it: Machine.instr list =
    [LDARGS;                    //load the commandline args to the stack
    CALL (1, "L1");             //jump to the L1 label, the content of main, and it has 1 param
    STOP;                       //halt when the function is done
    Label "L1";                 
        INCSP 1;                //allocate space on the stack, for int r; 
        GETBP; CSTI 1; ADD;     //get addr of r, to do r = n; 
        GETBP; CSTI 0; ADD;     //get addr of n, r = n;
        LDI;                    //get the content of n, onto the stack
        STI;                    //place n at the location of r, to do r = n;
        INCSP -1;               //remove the returned value from STI
        INCSP 1;                //allocate space on the stack for the new r, as in int r;
        GETBP; CSTI 0; ADD;     //get the address of n
        LDI;                    //place the content of n onto the stack, to prepare for square(n, &r);
        GETBP; CSTI 2; ADD;     //get the location of the new r
        CALL (2, "L2");         //do square(n, &r); where n and &r are on the stack. 2 is the number of params
        INCSP -1;               //remove the return value from square
        GETBP; CSTI 2; ADD;     //get the location of r
        LDI;                    //get value of r
        PRINTI;                 //print r, as it is on top of the stack
        INCSP -1;               //forget r, that print left there
        INCSP -1;               //forget r, as we exit the block
        GETBP; CSTI 1; ADD;     //gets the loc of the first r
        LDI;                    //loads the content of r to the stack
        PRINTI;                 //print r;
        INCSP -1;               //forget r, that print left there
        INCSP -1;               //forget r, as we exit the block
        RET 0;                  //returns, and removes 0 params from the stack
    Label "L2"; 
        GETBP; CSTI 1; ADD; LDI; //loads the value of *rp, in *rp = i * i;
        GETBP; CSTI 0; ADD; LDI; //loads the value of the first i, in *rp = i * i;
        GETBP; CSTI 0; ADD; LDI; //loads the value of the second i, in *rp = i * i;
        MUL;                     //multiplies the top two elements of the stack, here i*i
        STI;                     //place the result of the multiplication at 
        INCSP -1;                //removes the top of the stack, here the return from STI
        INCSP 0;                 //removes zero elements from the stack, since there were none allocated in the block
        RET 1                    //removes 1 element from the stack, then returns. it removes 1 element, because the function has 2 params (2-1=1)
    ]
```

```{}
...MicroC>java Machine ex5.out 42
1764 42
Ran 0.009 seconds
```

```{}
[ ]{0: LDARGS}
[ 42 ]{1: CALL 1 5}
[ 4 -999 42 ]{5: INCSP 1}
[ 4 -999 42 0 ]{7: GETBP}
[ 4 -999 42 0 2 ]{8: CSTI 1}
...
42 [ 4 -999 42 42 42 ]{51: INCSP -1}
[ 4 -999 42 42 ]{53: INCSP -1}
[ 4 -999 42 ]{55: RET 0}
[ 42 ]{4: STOP}
```

The above trace, is taken directly from the ex5.trace file.
It is not annotated, because the TA's have expressed that the bytecode is a sufficient answer.

## 8.3

PREINC and PREDEC added to CLex.fsl, and CPar.fsl
PreInc and PreDec added to absyn.fs
In comp.fs, in function cExpr, added PreInc and PreDec cases to the pattern match.

We tested the new compiler, with the following code.

```{}
void main(int n){
    print n;
    ++n;
    print n;
}
```

Bytecode generated from the program

```{}
> compileToFile (fromFile "Exer.8.3.c") "Exer.8.3.out";;
val it: Machine.instr list =
  [LDARGS; CALL (1, "L1"); STOP; Label "L1"; GETBP; CSTI 0; ADD; LDI; PRINTI;
   INCSP -1; GETBP; CSTI 0; ADD; DUP; LDI; CSTI 1; ADD; STI; INCSP -1; GETBP;
   CSTI 0; ADD; LDI; PRINTI; INCSP -1; INCSP 0; RET 0]
```

The result of running the program with the java machine

```{}
...>java machine Exer.8.3.out 42
42
43
```

## 8.4

### 8.4.ex8

```{}
> compile "ex8";;
val it: Machine.instr list =
  [LDARGS; 
  CALL (0, "L1"); 
  STOP; 
  Label "L1"; 
    INCSP 1; 
    GETBP; CSTI 0; ADD;
    CSTI 20000000; 
    STI; 
    INCSP -1; 
    GOTO "L3"; 
  Label "L2"; 
    GETBP; CSTI 0; ADD;
    GETBP; CSTI 0; ADD; LDI;
    CSTI 1; 
    SUB;
    STI;
    INCSP -1;
    INCSP 0;
  Label "L3";
    GETBP; CSTI 0; ADD; LDI; 
    IFNZRO "L2"; 
    INCSP -1; 
    RET -1
  ]
```

Prog1: `0 20000000 16 7 0 1 2 9 18 4 25`

The following is the text based compilation of Prog1.

```{}
CSTI 20000000;
GOTO 7;
CSTI 1;
SUB;
DUP;
IFNZERO;
DIV;
STOP;
```

The program compiled from ex8, is slower than Prog1.
This is the case because, it does more instructions to get to the same result.
For instance, the operation `GETBP; CSTI 0; ADD;`, evaluates to `0` in every case, because there is only one activation frame in this program.
Another slowdown, is the allocation and deallocation of stack space.
This can be avoided by knowing the location of elements and changing them directly, rather than using STI instructions.

### 8.4.ex13

```{}
> compile "ex13";;
val it: Machine.instr list =
    [LDARGS; CALL (1, "L1"); STOP; 
    Label "L1"; 
        INCSP 1; 
        GETBP; CSTI 1; ADD;
        CSTI 1889; 
        STI; 
        INCSP -1; 
        GOTO "L3"; 
    Label "L2"; 
        GETBP; CSTI 1; ADD; 
        GETBP; CSTI 1; ADD; LDI; 
        CSTI 1; ADD; 
        STI; 
        INCSP -1; 
        GETBP; CSTI 1; ADD; LDI;
        CSTI 4; 
        MOD; 
        CSTI 0; 
        EQ; 
        IFZERO "L7"; 
        GETBP; CSTI 1; ADD; LDI; 
        CSTI 100;
        MOD; 
        CSTI 0; 
        EQ; 
        NOT; 
        IFNZRO "L9"; 
        GETBP; CSTI 1; ADD;  LDI; 
        CSTI 400; 
        MOD;
        CSTI 0; 
        EQ; 
        GOTO "L8"; 
    Label "L9"; 
        CSTI 1; 
    Label "L8"; 
        GOTO "L6";
    Label "L7"; 
        CSTI 0; 
    Label "L6"; 
        IFZERO "L4"; 
        GETBP; CSTI 1; ADD; LDI;
        PRINTI; 
        INCSP -1; 
        GOTO "L5"; 
    Label "L4"; 
        INCSP 0; 
    Label "L5"; 
        INCSP 0;
    Label "L3"; 
        GETBP; CSTI 1; ADD; LDI; 
        GETBP; CSTI 0; ADD; LDI; 
        LT;
        IFNZRO "L2"; 
        INCSP -1; 
        RET 0
    ]
```

We see that while loops and conditionals are similar, and use jumps, to move around within the code.

## 8.5

## 8.6
