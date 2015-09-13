---
layout:     post
title:      "Compiling a boolean expression"
subtitle:   "because interpreting it was not enough"
date:       2015-05-24 19:21:00
author:     "Nicola Malizia"
tags: ["java", "compiler"]

twitter-card: true
twitter-image: "data/compiling-a-boolean-expression.png"

open-graph: true
open-graph-image: "data/compiling-a-boolean-expression.png"
---

Recently I have posted an article about [evaluating a boolean expression](how-to-evaluate-a-boolean-expression) using the Interpreter pattern. 

I've also written about how to implement a [simple stack machine](how-to-build-simple-stack-virtual-machine). 

In this blog post I want to combine those two article and to show you how to [compile](http://en.wikipedia.org/wiki/Compiler) a boolean expression for a [stack based virtual machine](http://en.wikipedia.org/wiki/Stack_machine). 

The task itself it is not a big deal but I hope that I can show you how powerful yet a [stack machine](http://en.wikipedia.org/wiki/Stack_machine) can be: a simple concept, a simple implementation, a great power!

## Instruction Set

This tiny virtual machine has a small instruction set just to support a boolean expression. 

- `PUSH` push a value onto the stack
- `AND` performs AND logic operation by popping two operands from the stack
- `OR` performs OR logic operation by popping two operands from the stack
- `NOT` performs NOT logic operation by popping one operand from the stack

The Virtual Machine handle `1` as `true` and `0` as `false`.  

```java
public class OpCode {
	public static final int PUSH = 0;
	public static final int AND = 1;
	public static final int OR = 2;
	public static final int NOT = 3;
}

public class Bool {
	public static final int FALSE = 0;
	public static final int TRUE = 1;
}
```

The generated code from the "compiler" is an array of integer `Integer[]`, that way the generated code looks like how a real compiler compiles a programming language. 

Here it is the virtual machine implemented: 

```java
public final class BooleanVM {
	private Stack<Integer> stack;

	public BooleanVM() {
		this.stack = new Stack<>();
	}

	public boolean start(Integer[] code) {
		System.out.println("CODE: " + Arrays.toString(code));
		int opcode;
		int ip = 0;
		while (ip < code.length) {
			opcode = code[ip++];
			switch (opcode) {
				case OpCode.PUSH : stack.push(code[ip++]); break;
				case OpCode.AND: {
					int b = stack.pop();
					stack.push(stack.pop() & b);
					break;
				}
				case OpCode.OR: {
					int b = stack.pop();
					stack.push(stack.pop() | b);
					break;
				}
				case OpCode.NOT: stack.push(~stack.pop() & 0x01); break;
				default: throw new RuntimeException("Invalid opcode " + code[ip -1]);
			}
		}
		return stack.pop() == Bool.TRUE;
	}
}
```

The code inside `boolean start(...)` describe the `fetch-decode-execute` cycle that every real CPU performs. 

The `fetch` stage is performed by `opcode = code[ip++];` followed by a `decode` stage performed by the `switch` statement and the actual `execution` is performed by the body of the `case` statement. 

Once the compile expression is evaluated the result is stored on top of the stack, returning the top of the stack we retrieve the result of the evaluated expression. 

Since this virtual machine uses Integer the logic operation must be done using bit-wise operations (this architecture does not know what a boolean is). 

`AND` and `OR` operation are done by using `&` and `|` bit-wise operators respectively. 

Since we are using Java `Integer`s a `NOT` operation on `0` can lead to some unexpected results. 

Java Integers are 32 bit wide in the `JVM`, by performing `~0`we will not have `1` in returns but instead `-1`. 

0 in binary is `00...00` (32 zeroes), applying the `~` operator we obtain `11...11` (32 ones) which is `-1` (Java uses [two's complement](http://en.wikipedia.org/wiki/Two%27s_complement)). 

By applying `& 0x01`to the result we are sure that we will retrieve only the last bit of the `Integer`.

## Compiling

What means [compiling](http://en.wikipedia.org/wiki/Compiler) ? Basically (very very basically) it means to translate a well defined way to express operations into another well defined way to express operations. 

As you may noticed a computer architecture, in general, has no notion what a `boolean` is, what a `while` statemens is, what a `class` is and so on. 

A compiler fill this gaps by translating a high level language to a low level language, which is the instruction set. 

In our case the "high level" language is a boolean expression and the "low level" language is the instruction set described above. 

I started from utilizing again the classes coded for the previous article about [evaluating a boolean expression](how-to-evaluate-a-boolean-expression). 

### Compiling by visiting

A common technique to implement a compiler is to [visit](http://en.wikipedia.org/wiki/Visitor_pattern) the [Abstract Syntax Tree](http://en.wikipedia.org/wiki/Abstract_syntax_tree) and generate the machine level code (binary code or byte code or whatever you want). 

The process of parsing and building the AST is the same used in the [other article](how-to-evaluate-a-boolean-expression). 

```java
public interface BooleanVisitor<T> {
	T visit(And a);

	T visit(Or o);

	T visit(Not n);

	T visit(True t);

	T visit(False f);
}
```

This interface is generic because in this way you can traverse and performs any operation on the generated AST. 

Indeed according to Wikipedia: 

> [..] the [visitor design pattern](http://en.wikipedia.org/wiki/Visitor_pattern) is a way of separating an algorithm from an object structure on which it operates. [...]

I had to modify the `BooleanExpression` interface accordingly: 

```java
public interface BooleanExpression {
	<T> T accept(BooleanVisitor<T> visitor);
}
```

Now the [Interpreter Pattern](http://en.wikipedia.org/wiki/Interpreter_pattern) is used to apply the Visitor recursively on the AST. 

### Actual compiling
Usually in order to generate code for a stack based machine it is sufficient to visit in post-order the AST. 

```java
public class BooleanCompiler implements BooleanVisitor<Void>{
	private List<Integer> code;

	public BooleanCompiler() {
		this.code = new LinkedList<>();
	}

	@Override
	public Void visit(And a) {
		visit(a.getLeft());
		visit(a.getRight());
		code.add(OpCode.AND);
		return null;
	}

	@Override
	public Void visit(Or o) {
		visit(o.getLeft());
		visit(o.getRight());
		code.add(OpCode.OR);
		return null;
	}

	@Override
	public Void visit(Not n) {
		visit(n.getLeft());
		code.add(OpCode.NOT);
		return null;
	}

	@Override
	public Void visit(True t) {
		code.add(OpCode.PUSH);
		code.add(Bool.TRUE);
		return null;
	}

	@Override
	public Void visit(False f) {
		code.add(OpCode.PUSH);
		code.add(Bool.FALSE);
		return null;
	}

	private Void visit(BooleanExpression b) {
		b.accept(this);
		return null;
	}

	public Integer[] getCode() {
		return code.toArray(new Integer[code.size()]);
	}
}
```

Since we only need to traverse the tree data structure and `emit` the proper opcode the return type of the visitor is `Void`. 

If we wanted to evaluate the expression instead of compiling it (like [the famous article](how-to-evaluate-a-boolean-expression)) we could have done instead. 

```java 
public class BooleanEvaluator implements BooleanVisitor<Boolean>{
	@Override
	public Boolean visit(And a) {
		return visit(a.getLeft()) && visit(a.getRight());
	}

	@Override
	public Boolean visit(Or o) {
		return visit(o.getLeft()) || visit(o.getRight());
	}

	@Override
	public Boolean visit(Not n) {
		return !visit(n.getLeft());
	}

	@Override
	public Boolean visit(True t) {
		return true;
	}

	@Override
	public Boolean visit(False f) {
		return false;
	}

	private Boolean visit(BooleanExpression b) {
		return b.accept(this);
	}
}
```
Note that the evaluation is still performed in a in-order traversal as described in the last article about boolean expression. 

Generating the Assembler code for the VM (parsing of this code is not supported in this project) is also trivial: 

```java
public class ToAssembly implements BooleanVisitor<Void> {
	private StringBuilder sb = new StringBuilder();

	@Override
	public Void visit(And a) {
		visit(a.getLeft());
		visit(a.getRight());
		sb.append("AND").append("\n");
		return null;
	}

	@Override
	public Void visit(Or o) {
		visit(o.getLeft());
		visit(o.getRight());
		sb.append("OR").append("\n");
		return null;
	}

	@Override
	public Void visit(Not n) {
		visit(n.getLeft());
		sb.append("NOT").append("\n");
		return null;
	}

	@Override
	public Void visit(True t) {
		sb.append("PUSH")
				.append(' ')
				.append("TRUE")
				.append("\n");
		return null;
	}

	@Override
	public Void visit(False f) {
		sb.append("PUSH")
				.append(' ')
				.append("FALSE")
				.append("\n");
		return null;
	}

	private Void visit(BooleanExpression b) {
		b.accept(this);
		return null;
	}

	public String getASM() {
		return sb.toString();
	}
}
```

As an example let's consider this boolean expression

```
(true & ((true | false) & !(true & false)))
```

First it is generated the Abstract Syntax Tree:

<p align="center"><img class="img-responsive" src="https://unnikked.ga/data/ast-boolean-expression.png"></p>

And then we can apply in any order one of the defined Visitor.

```
PUSH TRUE
PUSH TRUE
PUSH FALSE
OR
PUSH TRUE
PUSH FALSE
AND
NOT
AND
AND
```

If you want to check out the code it is hosted on [GitHub](https://github.com/unnikked-ga/booleancompliler). 