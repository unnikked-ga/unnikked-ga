---
layout:     post
title:      "How to build a Brainfuck interpreter"
subtitle:   "An esoteric programming language that does not lead to headaches"
date:       2014-12-24 15:51:58
author:     "Nicola Malizia"
tags: ["interpreter", "java"]


twitter-card: true
twitter-image: "data/how-to-build-a-brainfuck-interpreter.png"

open-graph: true
open-graph-image: "data/how-to-build-a-brainfuck-interpreter.png"
---

I've always wondered when I started to approach programming when I was younger how a programming language could drive a computer.

No matter if the language is Java, C, Python, PHP or other, I've always been fascinated the way a simple text file could produce the desired output almost magically - it's like transferring your thoughts.  

Some programming languages are easy to be used, some of them are a sort of natural language. There are others, instead, that despite the fact having the same power as a high level language they are very difficult to be used in practice, one of them is Brainfuck. 

Brainfuck it was designed by Urban Müller in 1993 with the aim to be a programming language suitable for a Turing Machine using the smallest possible compiler. 

This language comprehend of only eight elementary instruction, two of them are user for basic I/O. 

<table>
<thead>
<tr>
  <th>Character</th>
  <th>Meaning</th>
</tr>
</thead>
<tbody><tr>
  <td><code>&gt;</code></td>
  <td>increment the data pointer (to point to the next cell to the right).</td>
</tr>
<tr>
  <td><code>&lt;</code></td>
  <td>decrement the data pointer (to point to the next cell to the left).</td>
</tr>
<tr>
  <td><code>+</code></td>
  <td>increment (increase by one) the byte at the data pointer.</td>
</tr>
<tr>
  <td><code>-</code></td>
  <td>decrement (decrease by one) the byte at the data pointer.</td>
</tr>
<tr>
  <td><code>.</code></td>
  <td>output the byte at the data pointer.</td>
</tr>
<tr>
  <td><code>,</code></td>
  <td>accept one byte of input, storing its value in the byte at the data pointer.</td>
</tr>
<tr>
  <td><code>[</code></td>
  <td>if the byte at the data pointer is zero, then instead of moving the instruction pointer forward to the next command, jump it forward to the command after the matching ] command.</td>
</tr>
<tr>
  <td><code>]</code></td>
  <td>if the byte at the data pointer is nonzero, then instead of moving the instruction pointer forward to the next command, jump it back to the command after the matching <code>[</code> command.</td>
</tr>
</tbody></table>

From the specification of the language, an interpreter for it pretty straightforward. This code should be self explanatory. 

```java
import java.util.*;
public class Brainfuck {
    private Scanner sc = new Scanner(System.in);
    private final int LENGTH = 65535;
    private byte[] mem = new byte[LENGTH];
    private int dataPointer;
 
    public void interpret(String code) {
        int l = 0;
        for(int i = 0; i < code.length(); i++) {
            if(code.charAt(i) == '>') {
                dataPointer = (dataPointer == LENGTH-1) ? 0 : dataPointer + 1;
            } else if(code.charAt(i) == '<') {
                dataPointer = (dataPointer == 0) ? LENGTH-1 : dataPointer - 1;
            } else if(code.charAt(i) == '+') {
                mem[dataPointer]++;
            } else if(code.charAt(i) == '-') {
                mem[dataPointer]--;
            } else if(code.charAt(i) == '.') {
                System.out.print((char) mem[dataPointer]);
            } else if(code.charAt(i) == ',') {
                mem[dataPointer] = (byte) sc.next().charAt(0);
            } else if(code.charAt(i) == '[') {
                if(mem[dataPointer] == 0) {
                    i++;
                    while(l > 0 || code.charAt(i) != ']') {
                        if(code.charAt(i) == '[') l++;
                        if(code.charAt(i) == ']') l--;
                        i++;
                    }
                }
            } else if(code.charAt(i) == ']') {
                if(mem[dataPointer] != 0) {
                    i--;
                    while(l > 0 || code.charAt(i) != '[') {
                        if(code.charAt(i) == ']') l++;
                        if(code.charAt(i) == '[') l--;
                        i--;
                    }
                    i--;
                }
            }
        }
    }
 
    public static void main(String[] args) {
        new Brainfuck().interpret(args[0]);
    }
}
```

This source code assumes that the input Brainfuck source code is syntactically correct . The memory is represented by an array of byte as the specification. The object variable `dataPointer` saves the address of the pointed memory cell (handled by the instructions `+` `-` `>` `<` )

Particular attention must be done for branching instructions `[` and `]`.

Every time we encounter `[` we need to perform a "jump" (i.e we enter in a loop) so we need to find the matching `]`, you can think the variable `l` the level of nested loops.

## From Brainfuck to Java
Also translating a Brainfuck source code to a Java source code is pretty straightforward.

Each Brainfuck instruction can be directly translated in an instruction of a high level language such Java following this schema: 

<table>
<thead>
<tr>
  <th>Brainfuck</th>
  <th>Java</th>
</tr>
</thead>
<tbody><tr>
  <td><code>&gt;</code></td>
  <td><code>ptr++;</code></td>
</tr>
<tr>
  <td><code>&lt;</code></td>
  <td><code>ptr–;</code></td>
</tr>
<tr>
  <td><code>+</code></td>
  <td><code>mem[ptr]++;</code></td>
</tr>
<tr>
  <td><code>-</code></td>
  <td><code>mem[ptr]–;</code></td>
</tr>
<tr>
  <td><code>.</code></td>
  <td><code>System.out.print((char) mem[ptr]);</code></td>
</tr>
<tr>
  <td><code>,</code></td>
  <td><code>mem[ptr] = (byte) in.next().charAt(0);</code></td>
</tr>
<tr>
  <td><code>[</code></td>
  <td><code>while(mem[ptr] != 0) {</code></td>
</tr>
<tr>
  <td><code>]</code></td>
  <td><code>}</code></td>
</tr>
</tbody></table>

Also in this case the code is simple: 

```java
public class BrainfuckToJava {
	private StringBuilder source;
	private int ident;
	
	public BrainfuckToJava(String code) {
		source = new StringBuilder();
		source.append("import java.util.Scanner;\n");
		source.append("public class BFConverted {\n");
		source.append("\tprivate static int ptr;\n");
		source.append("\tprivate static byte[] mem = new byte[65535];\n");
		source.append("\tprivate static Scanner in = new Scanner(System.in);\n");
		source.append("\tpublic static void main(String[] args) {\n");
		convert(code, source);
		source.append("\t}\n");
		source.append("}\n");
		System.out.println(source.toString());
	}
	
	private void convert(String code, StringBuilder source) {
		for(int i = 0; i < code.length(); i++) {
			for(int t = 0; t < ident; t++) source.append('\t');
			switch(code.charAt(i)) {
				case '>': source.append("\t\tptr++;\n"); break;
				case '<': source.append("\t\tptr--;\n"); break;
				case '+': source.append("\t\tmem[ptr]++;\n"); break;
				case '-': source.append("\t\tmem[ptr]--;\n"); break;
				case '.': source.append("\t\tSystem.out.print((char) mem[ptr]);\n"); break;
				case ',': source.append("\t\tmem[ptr] = (byte) in.next().charAt(0);\n"); break;
				case '[': source.append("\t\twhile(mem[ptr] != 0) {\n"); ident++; break;
				case ']': source.append("\t\t}\n"); break;
				default: continue;
			}
			if(i+1 < code.length() && code.charAt(i+1) == ']') ident--;
		}
	}
	
	public static void main(String[] args) {
		new BrainfuckToJava(args[0]);
	}
}
```

Please note that the Java Virtual Machine does not allow methods with [segment code that are more than 64KB](http://stackoverflow.com/questions/2407912/code-too-large-compilation-error-in-java/2408005#2408005), therefore there may be some Brainfuck programs that cannot be executed in a single Java method. 

### Test the interpeter
If you want to test this code you may download the source code from [here](https://gist.github.com/unnikked/cfad836abd9e4619a1b1). If you are on a Linux machine after compiled the code you can execute it by typing: 

```
java Brainfuck "$(cat source.bf)"
```

Where `source.bf` is a valid Brainfuck program.

## Conclusions

If you want to go deep about Branfuck I suggest to read this [doc](https://docs.google.com/document/d/1M51AYmDR1Q9UBsoTrGysvuzar2_Hx69Hz14tsQXWV6M/edit). It contains also external resources, sample programs etc. 
