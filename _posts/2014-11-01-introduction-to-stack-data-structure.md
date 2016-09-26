---
layout:     post
title:      "An introduction to the stack data structure"
subtitle:   ""
date:       2014-11-01 15:00:00
author:     "Nicola Malizia"
tags: ["data structures", "java"]


twitter-card: true
twitter-image: "https://unnikked.ga/data/the-stack-data-structure-2.png"

open-graph: true
open-graph-image: "https://unnikked.ga/data/the-stack-data-structure-2.png"

math: true
flowchart: true
---

We use stacks inadvertently every day, either using computer and in daily routines. Instead of defining directly the data structure we will see a first example on where a stack is used.

Whenever we write a program, we are used to an expression like `c = a + c`, we also know what it does: the $+$ operator in the middle takes two operands and add them together.

We could also use operators $+$, $-$, $\times$ and so on.  Notice that the operator comes in between the operands, and because it is inserted and inscribed inside the two operands we reference to this as **infix notation**.

We are very familiar with this notation. Is the notation we were thought at the High School and we use it every day.  

It perfectly possible to write like that $+$ $a$  $b$ using a **prefix** plus, that it is the **prefix notation**.

If you pay attention to this notation you probably notice that you are already familiar with it, just think to a method that add two numbers, we would like to describe it as:

```python
def add(a, b):
	return a + b
```

And we would like to read it as _"Add together $a$ and $b$"_. Very self explanatory huh?

What happens if you write the operator after the operands, like $a$ $b$ $+$? This is the **postfix notation**

The mathematics logician [Jan ?ukasiewicz](http://en.wikipedia.org/wiki/Jan_%C5%81ukasiewicz) invented the **prefix notation** also known as **Polish Notation**.

Since the **postfix notation** looks like a reverse **prefix notation** we will also reference to it with the name of **Reverse Polish Notation** or **RPN**.

This notation saves the compiler or the interpreter of a lot of effort in actually executing the expression you write down.

As an example if we consider this simple statement in `C` like pseudo-code :

```c
int c = a + b;
```

This would translate (not actually in this way, it is just a rough idea) in assembly for an [accumulator machine](http://en.wikipedia.org/wiki/Accumulator_%28computing%29):

```asm
LOAD a
ADD b
MOVE c
```

Into the accumulator we would have the result of the operation.

If we consider instead a [stack based machine](http://en.wikipedia.org/wiki/Stack_machine) the assembly for the code would looks like:

```asm
PUSH a
PUSH b
ADD
MOVE c
```

If we consider instead a [register based machine](http://en.wikipedia.org/wiki/Register_machine):

```asm
LOAD R1 a
LOAD R2 b
ADD R1 R2 R3
MOVE R3 R4 ;assuming in R4 there is c address memory
```

All the code resembles the **RPN** notation if you pay attention.

We first need to gets somewhere the operands (we know _a priori_ how many operands the operator takes) and then apply the operator.  

For ever complicated things it has to convert it into Reverse Polish Notation first, explicitly or implicitly, in order to decide what code to generate.

Let's consider this expression: $$a+b \times c$$ We know that the operator $\times$ must be applied first than $+$, if we want to force the addition first we must apply parenthesis $$(a+b)\times c$$

In reverse polish notation you have not to parenthesize in order to force a specific operator because

$$a+b \times c = abc\times+$$

$$(a+b)\times c = ab+c*$$

We will cover later how to convert programmatically and infix expression to a postfix expression (Spoiler Alert! Stacks are too involved).

Here it comes the use of the stack. How we now evaluate a RPN expression ? Let's see this flow chart algorithm:

<div id="diagram">
st=>start: tokens
e=>end
push=>operation: Push
pop=>operation: Pop two operands
apply=>subroutine: evaluate
operand=>condition: Is a operand?
more=>condition: more tokens?

st->more
operand(no, right)->pop
pop(right)->apply->push
more(yes)->operand(yes)->push->more
more(no)->e
</div>

We consider and expression like $a$ $b$ $+$ $c$ $\times$ as a stream of tokens. So for each token we first ask to ourselves if such token is an operand or an operator. If it is an operand we push it on top of the stack and repeat the process. If it is an operator we pop $n$ operands (we know _a priori_ the _arity_ of the operator) from the stack, combine them (evaluate) and push the result on top the stack, we repeat the process for each token. If we consume all the tokens we exit. And now on the top of the stack there will be the result of the expression.

From this first example we learned that a stack should support two basic operations: **push** and **pop**.

There is also a third operation that a stack supports: **top**.

The policy which an "item" enters and leaves this data structure is known as LIFO (Last In First Out). To convince you of this, let's think about your books, you usually put each book one on top of each other on your desk, the last you put (push) is the first you take (pop), if you just want have a look what is the first book on top of the stack (top) you have just to have a look at the stack, so make sure to put your favorite book on top of the stack if you are looking for the willingness to study :).  

We refer to the set of this three basic operation as the *Abstract Data Type* of the stack. Why abstract? Because there are many different ways to implement a stack.

<blockquote> an abstract data type (ADT) is a mathematical model for a certain class of data structures that have similar behavior (Wikipedia)</blockquote>

Using a object oriented language as Java, we could represent the ADT for a stack as:

```java
public interface Stack<E> {
	/**
	* Push an element on top of the stack
	*/
	public void push(E elem);
	/**
	* Removes the first element on top
	* of the stack
	*/
	public E pop();
	/**
	* Returns, without removing, the first
	* element on top of the stack
	*/
	public E top();
}
```

There are two common ways to implement a stack: array based and linked list based.

We will see these two implementations, focusing only to the three methods of the interface, to not make heavy this article.

If you search on the web there are a lot of implementations, plus every programming language should have already an implementation on it's basic library, as an example the [Stack](http://docs.oracle.com/javase/8/docs/api/java/util/Stack.html) class of Java.

##Array implementation
You probably know what is an array: a contiguous allocated space in the memory accessible through an index.

The idea is to use a prefixed sized array and pushing and popping items into it, we do not really push and pop like we put or take a book on top of a stack. We rather use an index to mark the top of the stack.

```java
public class StackArray<E> implements Stack<E> {
	private int top;
	private E[] stack;

	public StackArray(int length) {
		if(length <= 0)
			throw new StackException("Negative size");
		this.stack = (E[]) new Object[length];
	}

	public void push(E elem) {
		if(top == stack.length)
			throw new StackException("Stack Full");
		stack[top++] = elem;
	}

	public E pop() {
		if(top == 0)
			throw new StackException("Stack Empty");
		return stack[--top];
	}

	public E top() {
		if(top == 0)
			throw new StackException("Stack Empty");
		return stack[top - 1];
	}
}
```

##Linked list implementation

A stack can be implemented through the concept of a linked list and, unlikely array based, can grow indefinitely.

The idea is to use a node called `top` that represents the top of the stack. We will push and pop values from that node.

A representation of a push:

<p align="center"><img src="https://unnikked.ga/data/stack-push.png" class="img-thumbnail"></p>

A representation of a pop:

<p align="center"><img src="https://unnikked.ga/data/stack-pop.png" class="img-thumbnail"></p>

```java
public class LinkedStack<E> implements Stack<E> {
	private static class Node<N> {
		N value;
		Node<N> next;
	}

	private Node<E> top;

	public void push(E elem) {
		Node<E> node = new Node<E>();
		node.value = elem;
		node.next = top;
		top = node;
	}

	public E pop() {
		if(top == null)
			throw new StackException("Stack Empty");
		E elem = top.value;
		top = top.next;
		return elem;
	}

	public E top() {
		if(top == null)
			throw new StackException("Stack Empty");
		return top.value;
	}
}
```

We will cover in future more usage of the stack.
