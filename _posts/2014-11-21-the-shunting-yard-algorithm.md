---
layout:     post
title:      "The Shunting Yard Algorithm"
subtitle:   ""
date:       2014-11-21 15:00:00
author:     "Nicola Malizia"
tags: ["algorithms", "java"]


twitter-card: true
twitter-image: "data/the-shunting-yard-algorithm.png"

open-graph: true
open-graph-image: "data/the-shunting-yard-algorithm.png"
---

In the last article I introduced the stack data structure. I've also provided a flow chart of a RPN expression evaluation. 

In this article we will see how to convert an infix notation into a postfix notation programmatically using the Djkstra's Shunting Yard algorithm.

Just as a recap, we are used to write math expression like `4+5* (5+2)`.

Using the Shunting Yard algorithm we will be able to convert this infix notation into the post fix notation `4 5 5 2 + * +`.

The Wikipedia page described very well this algorithm in a sort of pseudo-code so we will base on it, for who wants to go deepen you can read the [original paper](http://www.cs.utexas.edu/~EWD/MCReps/MR35.PDF) of Dijkstra. 

## The algorithm

- While there are tokens to be read:
	- Read a token.
	- If the token is a number, then add it to the output queue.
	- If the token is a function token, then push it onto the stack.
	- If the token is a function argument separator (e.g., a comma):
		- Until the token at the top of the stack is a left parenthesis, pop operators off the stack onto the output queue. If no left parentheses are encountered, either the separator was misplaced or parentheses were mismatched.
	- If the token is an operator, `o_1`, then:
		- while there is an operator token, `o_2`, at the top of the stack, and either `o_1` is left-associative and its precedence is *less than or equal* to that of `o_2`, or `o_1` if right associative, and has precedence *less than* that of `o_2`, then pop `o_2` off the stack, onto the output queue;
		- push `o_1` onto the stack.
	- If the token is a left parenthesis, then push it onto the stack.
	- If the token is a right parenthesis:
		- Until the token at the top of the stack is a left parenthesis, pop operators off the stack onto the output queue.
		- Pop the left parenthesis from the stack, but not onto the output queue.
		- If the token at the top of the stack is a function token, pop it onto the output queue.
		- If the stack runs out without finding a left parenthesis, then there are mismatched parentheses.
- When there are no more tokens to read:
	- While there are still operator tokens in the stack:
		- If the operator token on the top of the stack is a parenthesis, then there are mismatched parentheses.
		- Pop the operator onto the output queue.
- Exit.

A simple execution of this algorithm we can see through this image 

<p align="center"><img class="img-thumbnail" src="http://upload.wikimedia.org/wikipedia/commons/thumb/2/24/Shunting_yard.svg/628px-Shunting_yard.svg.png"></p>
*(Image courtesy of Wikipedia)*

## Implementation

I've implemented this algorithm in Java, and I added also a Lexer and a simple RPN evaluator to actually evaluate the parsed expression. 

The Lexer class do all the dirty work about lexing the given input string. 

As a example the string `2+3.2*(3-3)+sin(3)-ack(4,3)` will be lexed as:

```
5 2
1 +
5 3.2
3 *
7 (
5 3
2 -
5 3
8 )
1 +
13 sin
7 (
5 3
8 )
2 -
13 ack
7 (
5 4
11 ,
5 3
8 )
```

According to an inner table of symbols. The method `nextSymbol` of the class `Lexer` identifies the type of symbol analyzed through input. 

```java
public int nextSymbol() {
		try {
			switch (input.nextToken()) {
				case StreamTokenizer.TT_EOL:
					symbol = EOL; break;
				case StreamTokenizer.TT_EOF:
					symbol = EOF; break;
				case StreamTokenizer.TT_WORD: {
					if(input.sval.matches(CONST)) symbol = CONSTANT;
					else if(input.sval.matches(FUNC)) symbol = FUNCTION;
					else symbol = INVALID;
					break;
				}
				default:
					symbol = (mnemonics.get(Character.toString((char)input.ttype)) != null) ? mnemonics.get(Character.toString((char)input.ttype)) : NONE; break;
			}
		}catch (IOException e) {
			symbol = EOF;
		}
		return symbol;
	}
```

Since I'm a lazy programmer I created an HashMap (Actually a custom made BijectiveMap) on which I stored the pair `<operator, id>`

```java
public final DoubleHashMap<String, Integer> mnemonics = new DoubleHashMap<String, Integer>() \{\{
		put("+", PLUS);
		put("-", MINUS);
		put("*", MULTIPLY);
		put("/", DIVIDE);
		put("^", POWER);
		put("(", LEFT);
		put(")", RIGHT);
		put(",", COMMA);
	\}\};
```

The `ShuntingYardAlgorithm` class is pretty straightforward, it is just the implementation of the pseudo-code stated above. 

```java
public String evaluate() {
	    int id; String token;
	    while((id = lexer.nextSymbol()) != Lexer.EOF ) {
		    token = lexer.stringfy(id);
	        if(isNumber(token)) {
                output.append(token).append(' ');
            }

	        if(isFunction(token)) {
				stack.push(token);
	        }

	        if(isFunctionSeparator(token)) {
				while(stack.peek().charAt(0) != '(') {
					output.append(stack.pop()).append(' ');
				}
	        }

            if(isOperator(token)) {
                while (!stack.isEmpty() && isOperator(stack.peek()) && isHigerPrec(token)) {
                    output.append(stack.pop()).append(' ');
                }
                stack.push(token);
            }

            if(token.charAt(0) == '(') {
                stack.push(token);
            }

            if(token.charAt(0) == ')') {
                try{
                    while(!stack.isEmpty() && stack.peek().charAt(0) != '(') {
                        output.append(stack.pop()).append(' ');
                    }
                } catch (Exception e) {
                    throw new RuntimeException("Mismatched parenthesis");
                }
		       if(stack.isEmpty())
                    throw new RuntimeException("Mismatched parenthesis");
               stack.pop();
            }
        }

        while(!stack.isEmpty()) {
            if(stack.peek().charAt(0) == ')' || stack.peek().charAt(0) == '(') {
                throw new RuntimeException("Mismatched parenthesis");
            }
            output.append(stack.pop()).append(' ');
        }

        return output.toString();
    }
```

Last but not least, the `ReversePolishEvaluator` class which evaluate the "bytecode" of the parsed input string. 

```java
public Double evaluate() {
	    String code[] = bytecode.split(" ");
        String instruction;
	    double firstOperand, secondOperand;
	    int instructionPointer = 0;
	    while (instructionPointer < code.length) {
            instruction = code[instructionPointer++];
		    if(isNumber(instruction)) {
                stack.push(instruction);
            } else {
			    if (instruction.equals("^")) {
				    secondOperand = cast(stack.pop());
				    firstOperand = cast(stack.pop());
				    stack.push("" + Math.pow(firstOperand, secondOperand));
			    } else if (instruction.equals("/")) {
				    secondOperand = cast(stack.pop());
				    firstOperand = cast(stack.pop());
				    stack.push("" + (firstOperand / secondOperand));
			    } else if (instruction.equals("*")) {
				    secondOperand = cast(stack.pop());
				    firstOperand = cast(stack.pop());
				    stack.push("" + (firstOperand * secondOperand));
			    } else if (instruction.equals("+")) {
				    secondOperand = cast(stack.pop());
				    firstOperand = cast(stack.pop());
				    stack.push("" + (firstOperand + secondOperand));
			    } else if (instruction.equals("-")) {
				    secondOperand = cast(stack.pop());
				    if(stack.peek() != null && isNumber(stack.peek())) { // unary minus
				    	firstOperand = cast(stack.pop());
				    	stack.push("" + (firstOperand - secondOperand));
				    } else {
				    	stack.push("" + (-secondOperand));
				    }
			    } else if (instruction.equals("sin")) {
		            firstOperand = cast(stack.pop());
		            stack.push("" + Math.sin(firstOperand));
	            } else if (instruction.equals("cos")) {
		            firstOperand = cast(stack.pop());
		            stack.push("" + Math.cos(firstOperand));
	            } else if (instruction.equals("atan")) {
		            firstOperand = cast(stack.pop());
		            stack.push("" + Math.atan(firstOperand));
	            } else if (instruction.equals("ln")) {
		            firstOperand = cast(stack.pop());
		            stack.push("" + Math.log(firstOperand));
	            } else if (instruction.equals("exp")) {
		            firstOperand = cast(stack.pop());
		            stack.push("" + Math.exp(firstOperand));
	            } else if (instruction.equals("ack")) {
	            	secondOperand = cast(stack.pop());
		            firstOperand = cast(stack.pop());
		            stack.push("" + ackermann(firstOperand, secondOperand));
	            }else {
		            throw new RuntimeException("Unknown instruction: " + instruction);
	            }
            }
        }
        return cast(stack.pop());
    }
```

Since the parser algorithm is not capable of determine if the minus `-` operator is unary or binary you have may noticed that I added the control to determine if it is an unary or binary operator. 

```java
secondOperand = cast(stack.pop());
if(stack.peek() != null && isNumber(stack.peek())) { // unary minus
 	firstOperand = cast(stack.pop());
   	stack.push("" + (firstOperand - secondOperand));
} else {
   	stack.push("" + (-secondOperand));
}
```

For testing purpose I've added 6 builtin function: `sin`, `cos`, `atan`, `ln`, `exp`, `ack`. 

For example if you want to compute the `pi` number you could write as input:

```
4*atan(1)
```

Or if you want to compute the [Ackermann](http://en.wikipedia.org/wiki/Ackermann_function) function of 4 and 3 (don't do that :P):

```
ack(4,3)
```

If you want to combine more than two builin function I recommend to parentheses them:

```
(sin(0.71))^2+(cos(0.71))^2
```

If you want to checkout the code and test it to yourself you can also clone the github repository. 

Clone the repository with: 

```
git clone https://github.com/unnikked/ExpressionEvaluator.git
```

Compile:

```
javac src/ga/unnikked/expressionevaluator/*.java src/ga/unnikked/expressionevaluator/*/*.java
```

And execute it with (remember to `cd src`): 

```
java ga/unnikked/expressionevaluator/Main
```

You can also pipe in a file using `-f` directive

```
cat yourfile | java ga/unnikked/expressionevaluator/Main -f
```

Here an execution example:

```
$ cat test | java ga.unnikked.expressionevaluator.Main -f
4*atan(1)
4 1 atan * 
3.141592653589793
```
