---
layout:     post
title:      "How to build a simple assembler"
subtitle:   "define your own instruction set"
author:     "Nicola Malizia"
tags: ["java", "compiler"]

permalink: "/:title"

twitter-card: true

open-graph: true
---

An assemble, as you may already know, is a program that translates a program written in the assebly language to the machine code of the target computer. 

Assemblers are far more simpler to design respect to a compiled or interpreted high-level language. 

Before to start to design the assembly language and the assembler we need to define the target architecture. 

I used as a reference [this](http://users.ece.cmu.edu/~koopman/stack_computers/sec3_2.html) architecture for a stack based machine, just to have a consisten base to work on. 

I also wrote about stack machines and the interpretation of an RPN language, if you are interested in check out [here](https://unnikked.ga/how-to-build-simple-stack-virtual-machine). 

<h2>Instruction Set</h2>

The instruction set defines what your computer can do, based on this schematic

<p align="center"><img class="img-responsive" src="http://users.ece.cmu.edu/~koopman/stack_computers/fig3_1.gif"></p>

I have translated it programmatically into a Java Class. 

The translation is easy since each component can be modeled within the language itself:

 - **Data Stack** translates into a `Stack<Integer>`.
 - **Return Stack** translates as well into a `Stack<Integer>`.
 - **ALU**'s operations are directly mapped to the basic operations of Java `+ - * / %`. 
 - **Program Counter (PC)** is modeled by an `int` variable.
 - **I/0** operations are described by `System.out` and `System.in` classes.
 - **Control Logic** consist on the use of the `if` statement. 
 - **RAM** the program memory is modeled by an array of `int[]`.

Those are the basic components of a computer design, such schematic is called the **[data path](http://en.wikipedia.org/wiki/Datapath)** and it is one of the stage of defining a computer architecture. 

Those components define the operational part of a computer. Alone they cannot do anything, they need to be controlled in some way, it is needed do design also a control part. 

In real design the control part are modeled in two ways:

- **hardwired** i.e. defining a finite state machine that controls each components
- **micro-programmed** i.e. the use of a **ROM** that encodes each micro-instruction of the **Instruction Set**. 

Even if I discussed first the operational part and then the control part you may want to first decide the instruction that your computer must implements. 

The instruction set is the specification of your machine, thus it influence the design of both control and operational part.

To define an instruction set it is usually used the RTL (Register Transfer Language) which is not a programming language but a formal syntax that helps you define each instruction. 

In our case the RTL is modeled by the statements of the Java language, so after this long introduction we will see the instruction set. 

By using some of new mechanics of Java 8 I first of all defined a functional interface that models a `MicroInstruction`:

```java
private interface MicroInstruction {
	public void execute();
}
```

Then by using an array of `MicroInstruction[]` I built the ROM of this architecture. 

For example to define the `STORE` micro-instruction: 
```java
/* Store N1 at location ADDR in program memory */
rom[STORE] = () -> {
	int ADDR = dataStack.pop();
	int N1 = dataStack.pop();
	ram[ADDR] = N1;
};
```
You should be able to see that we use the Data Stack as means for operations. 

The `LOAD` micro-instruction is modeled as well: 

```java
/* Fetch the value at location ADDR in program memory,
   returning N1 */
rom[LOAD] = () -> {
	int ADDR = dataStack.pop();
	int N1 = ram[ADDR];
	dataStack.push(N1);
};
```

To this specification I added a new instruction called `INT`, this instruction implements a basic interrupt system in order to perform I/O operations. 

```java
/* Treat the value in the next program cell as an integer
   constant, and use it as an index for the interrupt
   vector */
rom[INT] = () -> {
	int N1 = ram[ip++];
	interrupts[N1].execute();
};
```

The interrupt vector it is just another array of `MicroInstruction[]`:

```java
interrupts[RNUM] = () -> dataStack.push(new Scanner(System.in).nextInt());

interrupts[PNUM] = () -> System.out.print(dataStack.pop());
```
I just don't want to fill this blog post with each instruction as we need to focus on the assembler, if you want to check out the complete instruction set take a look to the repository.

<h2>Building the assembler</h2>

Now that we have an instruction set to work on let's build the assembler. 