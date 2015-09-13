---
layout:     post
title:      "How to build a simple stack machine"
subtitle:   "Stacks, stacks everywhere"
date:       2015-03-25 19:00:00
author:     "Nicola Malizia"
tags: ["interpreter", "java"]

permalink: "/:title"

twitter-card: true
twitter-image: "data/stack-machines.png"

open-graph: true
open-graph-image: "data/stack-machines.png"
---

I [introduced](introduction-to-stack-data-structure) lately the stack data structure on my blog. Stack machines are computers that uses a stack to perform the evaluation of a *postfix* expression.

While describing the [Shunting Yard Algorithm](the-shunting-yard-algorithm) I've also provided a simple stack machine that evaluates an arithmetic expression. 

In this blog post I'm going to develop further the concept of the reverse polish notation evaluator providing a more powerful stack machine. 

In general every computer architecture has it's own instruction set, which is the set of operations that the machine performs. 

The instructions usually are the basic arithmetic operations, I/O, jumps, subroutine, aka functions calls. 

## Architecture of a computer
The architecture of a computer is the way how inner logic components, logic circuits, are connected together in order to manifest specifics behaviors, in our case the implementation of the instruction set.

There are two main way of organize a computer architecture:

 - **Von Neumann**: in which the program data and code are stored in the same memory.
 - **Harvard**: in which the program data and code are stored in different memory.

Independently of the type of the architecture we can classify three main type of machines:

- **Accumulator**: are the simplest form of computers, the machine uses an intermediate register, called accumulator, to performs it's operations. 
- **Stack**: stack machines uses a stack in order to performs operations. 
- **Register**: as the name suggests it uses registers to performs operations. 

There are also **hybrid** machines which usually combines stacks and register to achieve faster performance. 

After this brief and synthetic discussion I'm going to describe a stack machine based on the Von Neumann architecture. 

## The instruction set
What does our stack machine should be able to do for us ? This is the first question that a designer of a computer should ask to himself before to start to build it.

That's why the instruction set is the definition of your machine, it defines how the logic circuits are interconnected and how they have to behave when an instruction has to be executed. 

I'm not describing at this level of detail here the machine, I rather uses the construct of the Java programming language to mimic those inner circuits. 

Without further ado I choose to describe a stack machine that it is capable to do basic arithmetic, simple I/O, basic control flow and subroutine calls. 

## Assembly
We are gonna use the Reverse Polish Notation as the "grammar" of our stack machine, it makes our lives simple as it requires very minimal parsing, almost none.

# Building the Virtual Machine
 The basic component of our machine are essentially: 

- **Instruction pointer**: it is a register that contains the next cell memory that stores an instruction or a data, integers in our case.
- **ALU**: the circuit that performs the basic arithmetic operations, we'll use the usual operators of Java. 
- **RAM**: our RAM will be the parsed input string, for example `1 2 +` becomes an array `["1", "2", "+"]`of strings. 
- **Stack**: it is used to evaluate the program.
- **Call stack**: it is used to performs call to subroutines properly storing the **Stack Frame**.

To keep things simple, since we are describing the machine in a very abstract way, we will use some additional data structures to "expand" the RAM. 

## Evaluation
Seeing that we are using the RPN notation, the evaluation of the code is straightforward. 

- for each token of the input string
	- if the token is a number 
		- push on to the stack
	- if the token is an instruction
		- pops many times from the stack as the arity of the function and execute the instruction   

Here the translation of the algorithm described: 

```java
public void evaluate() {
	String[] tokens = // input parsed 
	String token; // current token
	while (ip < tokens.length) {
		token = tokens[ip++];
		if (token.matches("[0-9]([0-9])*")) { // is and Integer
			stack.push(Integer.valueOf(token));
		} else {
			// execute the instruction
		}
	}
}
```

## Instruction pointer
It's a register that holds the next address cell memory of the RAM. The main memory is an indexed array that starts from 0 till the last address of the memory, just how arrays works in Java the RAM is the almost the same. 

In order to represent it we will use a simple `int` variable: 

```java
int ip = 0;
```

## Arithmetic

Implementing basic arithmetic instructions is very straight forward, we know that every operator needs two operand. 

```java
if (token.equalsIgnoreCase("+")) {
	// second operand 
	s = stack.pop();
	// first operand 
	f = stack.pop(); 
	/*
	 * push the result 
	 * of the executed 
	 * instruction on
	 * top of the stack
	 */
	stack.push(f + s); 
} else if (/* op */) {
	...
} else if ...
```

And so on for others arithmetic instructions. 

## Control Flow

In order to implement basic instructions for control flow we need to define a pseudo-instruction for defining labels.

Labels are just mnemonics to an address of the memory, it is way more easier to write program using mnemonics instead of hard coding the addresses.

We define the pseudo instruction `label(name)` that define a "checkpoint" in our code so we can later refer to it via a jump instruction or a call instruction. 

We need to keep track of such labels so we use an `HashMap` as a support. 

Moreover since this is a pseudo-instruction we need first to scan the program and store in the HashMap the location of the labels. 

```java
pubic void evaluate() {
...
	while (ip < tokens.length) {
		token = tokens[ip]; // current token
		// check if it is a label definition
		if (token.matches("^label\\((.+)\\)$")) {
			// extract the label name
			String label = token.subSequence(
					token.indexOf('(') + 1,
					token.indexOf(')')
			).toString();
			// "register" the label and the point
			// encountered during scanning
			labels.put(label, ip);
		}
		ip++;
	}
...
}
```

It is wise to check if a label is already defined in order to prevent subtle bugs. 

That's not all, while executing the code we need to ignore label definitions so we just add:

```java
...
} else if (token.matches("^label\\((.+)\\)$")) {
	// pseudo instruction
} else {
...
```

Now we can implement our control flow statement: 

```java
// unconditional jump
} else if (token.matches("^jmp\\((.+)\\)$")) {
	String label = token.subSequence(
		token.indexOf('(') + 1,
		token.indexOf(')')
	).toString();
	ip = labels.get(label);
// jump if the popped value is zero
} else if (token.matches("^jz\\((.+)\\)$")) {
	String label = token.subSequence(
		token.indexOf('(') + 1,
		token.indexOf(')')
	).toString();
	if (stack.pop() == 0) {
		ip = labels.get(label);
	}
}
```

As you may have noticed the `jz` instruction consumes a value from the stack, dropping it. For this reason we introduce the instruction `dup` that duplicates the top of the stack if necessary during the execution of a program. 

```java
...
} else if (token.equalsIgnoreCase("dup")) {
	stack.push(stack.peek());
}
...
```

You may also choose that a conditional jump loop will not consume the top of the stack, this is up to you. 

As an example this code will never halts:

```bash
1 #push 1 on top of the stack
label(loop) # defines the label loop
	dup # duplicates the top of the stack
	jnz(loop)
```

Since the top of the stack is always `1` the loop will never exit, it is the same as writing in C:

```c
while(1) {
	// nothing
}
```

## I/O

I/O is useful to let the machine speak to the world. In our case we only provide two instruction `.` to print the ASCII character and `.num` to print a integer.

```java
...
} else if (token.equalsIgnoreCase(".")) { 
	// print the ASCII character
	System.out.print((char) stack.pop().byteValue());
} else if (token.equalsIgnoreCase(".num")) {
	// print int
	System.out.print(stack.pop());
}
...
```

In that way we can also output strings, for example: 

```bash
104 . 101 . 108 . 108 . 111 . 44 . 32 . 119 . 111 . 114 . 108 . 100 . 10 .
```

outputs:

```
hello, world
```

## Variables
Everybody probably agree that being able to use mnemonics to indicate variables are far more easy than remembering the address of the location of the variable. 

Variables can be threat as labels, indeed they are only a reference to a location into memory, so we are gonna to define two instructions that will help us define and use variables. 

We use another `HashMap` to store it. 

```java
} else if (token.matches("^!var\\((.+)\\)$")) {
	String name = token.subSequence(
		token.indexOf('(') + 1,
		token.indexOf(')')
	).toString();
	variables.put(name, stack.pop());
} else if (token.matches("^var\\((.+)\\)$")) {
	String name = token.subSequence(
		token.indexOf('(') + 1,
		token.indexOf(')')
	).toString();
	// better check if it is already defined
	stack.push(variables.get(name));
} 
```

We can write an iterative version of the Fibonacci sequence using the instruction set that we built so far:

```bash
# define vars
0 !var(i)
0 !var(p)
1 !var(n)
0 !var(tmp)

# output i prev
var(i) .num 32 .
var(p) .num 10 .

# output i n
var(i) .num 32 .
var(n) .num 10 .

label(next)

# tmp = n + p
# p = n
# n = tmp
var(p) var(n) + !var(tmp)
var(n) !var(p)
var(tmp) !var(n)

# output i n
var(i) .num 32 .
var(n) .num 10 .

# i++
var(i) 1 + !var(i)

jmp(next)
```

## Subroutines
In order to perform subroutine calls we need to introduce another stack, the call stack. 

This particular stack is useful to save and the restore the context used by a subroutine and the return address of the routine itself. 

A class `StackFrame` will help us to define a stack frame. 

```java
private class StackFrame {
	public int ip; // return address
	public HashMap<String, Integer> variables;

	public StackFrame(int ip, HashMap<String, Integer> variables) {
		this.ip = ip;
		this.variables = variables;
	}
}
```

We are now able to implement the `call` and `ret` instruction:

```java
} else if (token.matches("^call\\((.+)\\)$")) {
	String label = token.subSequence(
		token.indexOf('(') + 1,
		token.indexOf(')')
	).toString();
	// make sure you copy the HashMap to avoid aliasing
	calls.push(new StackFrame(ip, new HashMap<String, Integer>(variables)));
	ip = labels.get(label);
} else if (token.equalsIgnoreCase("ret")) {
	StackFrame frame = calls.pop();
	ip = frame.ip;
	variables = frame.variables;
}
```

It is important to save the `variables` hash table so each subroutine will have it's own context. 

This aspect is called *lexical scoping* and I invite you to read the [Wikipedia](http://en.wikipedia.org/wiki/Scope_%28computer_science%29) page about it. 

```bash
jmp(start)

# if n == 0 || n < 2 ret 1
# ret n * f(n-1)

label(fac)
	dup jz(ret)
	dup 2 -
		jlez(ret_1)
# recurse
	dup 1 - call(fac) * ret

label(ret_1)
	ret

label(ret)
	drop
	1 ret

label(start)
	3 call(fac)
	.num
# print \n
	10 .
```

## Conclusions
That's it, we built a simple stack machine starting from a simple notation, the code may look unfamiliar but it's the same as you write a normal asm code. 

A big thanks to [Igor](https://github.com/igorw) that inspired me to wrote this blog post, I've used some of his example here and reproduced his [PHP implementation](https://igor.io/2013/08/28/stack-machines-fundamentals.html) so most of the credit goes to him. 

I really recommend you to check out his [blog](https://igor.io/archive.html) for amazing stuff!

In the past I've also tried to implement a simple accumulator machine, If you want to check it out click [here](https://gist.github.com/unnikked/e6c258094bd45332e01a).

The source code of my implementation is [here](https://gist.github.com/unnikked/2c91bd9ca0bcceae7088), mess with it and let me know how you implemented it. I know that my implementation is basic and rough, I just wanted to keep things simple in order to show the key concepts. 

To end this blog post I suggest to check out the Wikipedia article about [stack machines](http://en.wikipedia.org/wiki/Stack_machine).