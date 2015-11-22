---
layout:     post
title:      "Subleq - a one instruction set computer"
subtitle:   "A digest about OISCs abstract machine architecture"
date:       2015-11-22 13:42:00
author:     "Nicola Malizia"
tags: ["logisim", "interpreter", "compiler", "computer architecture"]

header-img: "data/ewk6igr.png"

twitter-card: true
twitter-image: "/data/social/subleq-a-one-instruction-set-computer.jpg"

open-graph: true
open-graph-image: "/data/social/subleq-a-one-instruction-set-computer.jpg"
---

Here again talking about the inner aspects of computer architectures. In some previous blog post I've already exposed, basically, what is a computer architecture and the instruction set while describing a [simple stack machine](/how-to-build-simple-stack-virtual-machine/) and a [brainfuck interpreter](/how-to-build-a-brainfuck-interpreter/). 

### Some definitions

I write there some definitions about some terms for make easy to understand this blog post. 

> A *computer architecture* is a set of rules and methods that describe the functionality, organization and implementation of computer systems. Some definitions of architecture define it as describing the capabilities and programming model of a computer but not a particular implementation. In other descriptions computer architecture involves instruction set architecture design, microarchitecture design, logic design, and implementation. ([1](https://en.wikipedia.org/wiki/Computer_architecture))

> An *instruction set*, or instruction set architecture (ISA), is the part of the computer architecture related to programming, including the native data types, instructions, registers, addressing modes, memory architecture, interrupt and exception handling, and external I/O. An ISA includes a specification of the set of opcodes (machine language), and the native commands implemented by a particular processor. ([2](https://en.wikipedia.org/wiki/Instruction_set))

For example the simple stack machine have more or less 15 instructions and the brainfuck interpreter (you can think the interpreter as an abstract machine) count 8 instructions. 

There is a class of abstract machines that have only 1 instruction, this class is called `OISC` (one instruction set computer) or also `URISC`(ultimate reduced instruction set computer). 

There are 4 types of instruction: 

- <u>[Subtract and branch if not equal to zero](https://en.wikipedia.org/wiki/One_instruction_set_computer#Subtract_and_branch_if_non_zero)</u>
- <u>[Subtract and branch if less than or equal to zero](https://en.wikipedia.org/wiki/One_instruction_set_computer#Subtract_and_branch_if_less_than_or_equal_to_zero)</u>
- <u>[Subtract and branch if negative](https://en.wikipedia.org/wiki/One_instruction_set_computer#Subtract_and_branch_if_negative)</u>
- <u>[Reverse subtract and skip if borrow](https://en.wikipedia.org/wiki/One_instruction_set_computer#Reverse_subtract_and_skip_if_borrow)</u>
- <u>[Transport triggered architecture](https://en.wikipedia.org/wiki/One_instruction_set_computer#Transport_triggered_architecture)</u>

Each instruction listed can be an [universal computer](https://en.wikipedia.org/wiki/Universal_computer) in the [turing machine](https://en.wikipedia.org/wiki/Turing_machine) model.

For the moment we will focus on the _Subtract and branch if less than or equal to zero_ or `subleq` instruction.

## Subtract and branch if less than or equal to zero

The subleq instruction subtracts the contents at address `a` from the contents at address `b`, stores the result at address `b`, and then, if the result is *not positive*, transfers control to address `c` (if the result is positive, execution proceeds to the next instruction in sequence).

Is trivial to implement such abstract machine, here for example a subleq interpreter made in Java (by me :P). 

```java
public class Subleq {
	private int programCounter, a, b, c;

	public void run(int[] memory) {
		while (programCounter >= 0) {
			a = memory[programCounter];
			b = memory[programCounter + 1];
			c = memory[programCounter + 2];

			if (a < 0 || b < 0) {
				programCounter = -1;
			} else {
				memory[b] = memory[b] - memory[a];

				if (memory[b] > 0) {
					programCounter += 3;
				} else {
					programCounter = c;
				}
			}
		}
	}
}
```

The variable `programCounter` should be familiar on what it represents. The memory is represented by an array of `int[]`. 

From the single `subleq` instruction, other complex instruction can be derived, for example an `ADD` instruction is [3](https://en.wikipedia.org/wiki/One_instruction_set_computer#Synthesized_instructions): 

```
ADD a, b == subleq a, Z
            subleq Z, b
            subleq Z, Z
```

> The first instruction subtracts the content at location a from the content at location Z (which is 0) and stores the result (which is the negative of the content at a) in location Z. The second instruction subtracts this result from b, storing in b this difference (which is now the sum of the contents originally at a and b); the third instruction restores the value 0 to Z.

I've assembled by and as example this program: 

```java
int[] memory = {
	10, 9, 3,
	9, 11, 6,
	9, 9, 9,
	0, 2, 5,
};
```

The program performs `2 + 5`. 

## A simple abstract machine to learn complex thigs

We could all agree that this architecture is simple but yet hard to work with. But yet it's very powerfull because you can learn new things. 

- Do you want to understand how an assembler can be made for this architecture ? Oleg Mazonka <u>[made one](http://mazonka.com/subleq/)</u>. 
- Do you want to understand how a compiler can be made ? Also Oleg Mazonka created <u>[Higher Subleq](http://mazonka.com/subleq/hsq.html)</u> a C-ish language to subleq compiler.
- Do you want to see how the real hardware for this architecture can be made? <u>[David A Roberts](https://davidar.io/)</u> <u>[implemented it](https://github.com/davidar/subleq)</u> using Logisim 

![Subleq Logisim](/data/subleq-logisim.jpg)

## Conclusions

I don't feel to say more but I hope I inspired you to investigate further on this particular computer architecture. 