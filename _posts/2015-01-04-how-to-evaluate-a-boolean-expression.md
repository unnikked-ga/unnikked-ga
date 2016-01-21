---
layout:     post
title:      "How To Build A Boolean Expression Evaluator"
subtitle:   "Using a recursive descent parser and the interpreter pattern"
date:       2015-01-04 11:58:11
author:     "Nicola Malizia"
tags: ["interpreter", "java"]

twitter-card: true
twitter-image: "data/a-boolean-expression-evaluator.png"

open-graph: true
open-graph-image: "data/a-boolean-expression-evaluator.png"
---

Previously we saw the [Dijkstra Shunting Yard Algorithm](the-shunting-yard-algorithm) that helped us to convert an in-fix arithmetic expression into a post-fix one and then evaluated it. 

In this post we will use different tecniques to parse and evaluate a boolean expression. 

We refer to a boolean expression described by this [EBNF](http://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_Form) grammar: 

```
<expression>::=<term>{<or><term>}
<term>::=<factor>{<and><factor>}
<factor>::=<constant>|<not><factor>|(<expression>)
<constant>::= false|true
<or>::='|'
<and>::='&'
<not>::='!'
```

<blockquote>EBNF (*Extended Backus–Naur Form*) is a notation to express formally a [context-free grammar](http://en.wikipedia.org/wiki/Context-free_grammar).</blockquote>

In our case we describe a boolean expression as: 

- An `<expression>` composed by a `<term>` and eventually a repetition of `<or>` and `<term>`. 
- A `<term>` is composed by a `<factor>` and eventually repetition of `<and>` and `<factor>`. 
- A `<factor>` is composed by a `<constant>` or a `<not><factor>` or an `<expression>` (delimited by parenthesis).
- A `<constant>` can be `false` or `true` and lastly we have the symbolic definition for `<or>`, `<and>` and `<not>`.

This grammar describe also the operator precedence for a boolean expression.

<blockquote>
	<p>A context-free grammar (CFG) is a formal grammar in which every production rule is of the form `V -> w` where V is a single nonterminal symbol, and w is a string of terminals and/or nonterminals (w can be empty). A formal grammar is considered "context free" when its production rules can be applied regardless of the context of a nonterminal. No matter which symbols surround it, the single nonterminal on the left hand side can always be replaced by the right hand side.</p>
	<footer>From <cite title="Wikipedia">Wikipedia</cite></footer>
</blockquote>

This article aim to provide a point of start for beginners, like me, that are approaching to the study of contex-free-grammars, how to build (a very simple) interpreter and so on. 

I will show in this article a programmatically method that help us to parse, using a [Recursive Descent Parser](http://en.wikipedia.org/wiki/Recursive_descent_parser), a boolean expression and let us build an [Abstract Syntax Tree](http://en.wikipedia.org/wiki/Abstract_syntax_tree) for the given grammar and then, using the [Intepreter Pattern](http://en.wikipedia.org/wiki/Interpreter_pattern), interpret it. 

<blockquote>
	<p>A recursive descent parser is a kind of top-down parser built from a set of mutually recursive procedures (or a non-recursive equivalent) where each such procedure usually implements one of the production rules of the grammar.</p>
	<footer>From <cite title="Wikipedia">Wikipedia</cite></footer>
</blockquote>

From the theory we know that a Recursive Descent Parser is applicable to a family of **LL(k)** grammars.

That means that It parses the input from Left to right, performing [Leftmost derivation](http://en.wikipedia.org/wiki/Context-free_grammar#Derivations_and_syntax_trees) of the sentence if it uses `k` tokens of lookahead when parsing a sentence and it does not use backtracking: in our case the grammar is **LL(1)**.

From now on we are going to build our boolean expression evaluator following this scheme: 

<p align="center"><img class="img-responsive" src="/data/lexer-parser-interpreter.png" alt="lexer-parser-interpreter"></p>

## Building the Lexer
A [Lexer](http://en.wikipedia.org/wiki/Lexical_analysis) is a component that performs the Lexical Analysis of a source code program, it tokenize the input and provide a stream of tokens that will be consumed by the parser. 

Since the grammar is simple also the lexer is trivial to build:

```java
public class Lexer {
	private StreamTokenizer input;

	private int symbol = NONE;
	public static final int EOL = -3;
	public static final int EOF = -2;
	public static final int INVALID = -1;

	public static final int NONE = 0;

	public static final int OR = 1;
	public static final int AND = 2;
	public static final int NOT = 3;

	public static final int TRUE = 4;
	public static final int FALSE = 5;

	public static final int LEFT = 6;
	public static final int RIGHT = 7;

	public static final String TRUE_LITERAL = "true";
	public static final String FALSE_LITERAL = "false";

	public Lexer(InputStream in) {
		Reader r = new BufferedReader(new InputStreamReader(in));
		input = new StreamTokenizer(r);

		input.resetSyntax();
		input.wordChars('a', 'z');
		input.wordChars('A', 'Z');
		input.whitespaceChars('\u0000', ' ');
		input.whitespaceChars('\n', '\t');

		input.ordinaryChar('(');
		input.ordinaryChar(')');
		input.ordinaryChar('&');
		input.ordinaryChar('|');
		input.ordinaryChar('!');
	}

	public int nextSymbol() {
		try {
			switch (input.nextToken()) {
				case StreamTokenizer.TT_EOL:
					symbol = EOL;
					break;
				case StreamTokenizer.TT_EOF:
					symbol = EOF;
					break;
				case StreamTokenizer.TT_WORD: {
					if (input.sval.equalsIgnoreCase(TRUE_LITERAL)) symbol = TRUE;
					else if (input.sval.equalsIgnoreCase(FALSE_LITERAL)) symbol = FALSE;
					break;
				}
				case '(':
					symbol = LEFT;
					break;
				case ')':
					symbol = RIGHT;
					break;
				case '&':
					symbol = AND;
					break;
				case '|':
					symbol = OR;
					break;
				case '!':
					symbol = NOT;
					break;
				default:
					symbol = INVALID;
			}
		} catch (IOException e) {
			symbol = EOF;
		}

		return symbol;
	}
}
```

The `StreamTokenizer` class is [provided](http://docs.oracle.com/javase/7/docs/api/java/io/StreamTokenizer.html) by default by the Java API and this class help us to define the structure of each token in order to identify it during the scanning of the source code. 

The are defined some class constants that help us to identify each type of token, so we can lately use it during the parsing. 

Real lexers can be more complex, for example we can define some "helper" methods for the parser such as `lookAhead` to read tokens from inputs without consuming it (performing syntactic checkings) or a `getSval` method to retrieve the string value of a specific token (example a variable name).

The `Lexer` class will be used by the `Recursive Descent Parser` class that in turn produces the Abstract Syntax Tree of the grammar that we'll interpret using the Interpreter Pattern. 

## Building the parser
We build the parser using the recursive descent parser techinque. Each `NonTerminal` symbol is a method in the parser class and the code reflect closely the notation form of the grammar. 

```java
public class RecursiveDescentParser {
	private Lexer lexer;
	private int symbol;
	private BooleanExpression root;

	private final True t = new True();
	private final False f = new False();

	public RecursiveDescentParser(Lexer lexer) {
		this.lexer = lexer;
	}

	public BooleanExpression build() {
		expression();
		return root;
	}

	private void expression() {
		term();
		while (symbol == Lexer.OR) {
			Or or = new Or();
			or.setLeft(root);
			term();
			or.setRight(root);
			root = or;
		}
	}

	private void term() {
		factor();
		while (symbol == Lexer.AND) {
			And and = new And();
			and.setLeft(root);
			factor();
			and.setRight(root);
			root = and;
		}
	}

	private void factor() {
		symbol = lexer.nextSymbol();
		if (symbol == Lexer.TRUE) {
			root = t;
			symbol = lexer.nextSymbol();
		} else if (symbol == Lexer.FALSE) {
			root = f;
			symbol = lexer.nextSymbol();
		} else if (symbol == Lexer.NOT) {
			Not not = new Not();
			factor();
			not.setChild(root);
			root = not;
		} else if (symbol == Lexer.LEFT) {
			expression();
			symbol = lexer.nextSymbol(); // we don't care about ')'
		} else {
			throw new RuntimeException("Expression Malformed");
		}
	}
}
```

From the point of view of [Design Patterns](http://en.wikipedia.org/wiki/Software_design_pattern) principles we can identify into the `RecursiveDescentParser` class the *Builder* and *Director* component of the [Builder Pattern](http://en.wikipedia.org/wiki/Builder_pattern).

Since `True` and `False` are constants and they will not modify their state during the interpretation of the expression, I've also applied implicitly the [Flyweight Pattern](http://en.wikipedia.org/wiki/Flyweight_pattern) by declaring the class `True` and `False` final and using their object refereces during the building of the AST (look at `factor()` method).

## Building the interpeter
The `RecursiveDescentParser` class returns an instance of `BooleanExpression` class that represent the root of the AST of the given boolean expression.

The AST follow the interpreter pattern principle and can be schematized as the image:

<p align="center"><img class="img-responsive" src="/data/interpreter-pattern.png" alt="Interpreter Pattern"></p>

`BooleanExpression` is an interface that export the method `interpret()`:

```java
public interface BooleanExpression {
	public boolean interpret();
}
```

Each `NonTerminal` and `Terminal` class must implement such interface. Just for the sake to do it and for educational purpose I defined also the two mentioned abstract class:
 
```java
public abstract class NonTerminal implements BooleanExpression {
	protected BooleanExpression left, right;

	public void setLeft(BooleanExpression left) {
		this.left = left;
	}

	public void setRight(BooleanExpression right) {
		this.right = right;
	}
}
```

```java
public abstract class Terminal implements BooleanExpression{
	protected boolean value;

	public Terminal(boolean value) {
		this.value = value;
	}

	public String toString() {
		return String.format("%s", value);
	}
}
```

A non terminal symbol like `Or` will assume this shape:

```java
public class Or extends NonTerminal {
	public boolean interpret() {
		return left.interpret() || right.interpret();
	}

	public String toString() {
		return String.format("(%s | %s)", left, right);
	}
}
```

Notice how the `interpret()` method is implemented, we call recursively from left to right the `interpret()` method on each child node of the AST, the intepretation of the AST from a root node is automatically done by visiting the tree [in-order](http://en.wikipedia.org/wiki/Tree_traversal#In-order_.28symmetric.29) (You could also apply the [Visitor Pattern](http://en.wikipedia.org/wiki/Visitor_pattern)).

A terminal symbol like `True` can be represented as: 

```java
public class True extends Terminal {
	public True() {
		super(true);
	}

	public boolean interpret() {
		return value;
	}
}
```

On a terminal symbol the recursion ends and thus the evaluation begins.

## Example of parsing and evaluating an expression

Let's say we give to the program this boolean expression: 

```
true & ((true | false) & !(true & false))
```

The first thing we have to do is to tokenize the input string, it would be produced a stream of tokens like the follow:

```java
Token[true], line 1 -> 4
Token['&'], line 1 -> 2
Token['('], line 1 -> 6
Token['('], line 1 -> 6
Token[true], line 1 -> 4
Token['|'], line 1 -> 1
Token[false], line 1 -> 5
Token[')'], line 1 -> 7
Token['&'], line 1 -> 2
Token['!'], line 1 -> 3
Token['('], line 1 -> 6
Token[true], line 1 -> 4
Token['&'], line 1 -> 2
Token[false], line 1 -> 5
Token[')'], line 1 -> 7
Token[')'], line 1 -> 7
```

The the parser builds an Abstract Syntaxt Tree as follow: 

<p align="center"><img class="img-responsive" src="/data/ast-boolean-expression.png" alt="Abstract Syntax Tree For The Boolean Expression"></p>

Or according to the `toString()` method would look like: 

```
(true & ((true | false) & !(true & false)))
```

Calling the `interpret()` method on the root of the AST corresponds to an [in-order traversal](http://en.wikipedia.org/wiki/Tree_traversal#In-order_.28symmetric.29) of the tree. 

Here it is an example of esecution on this AST:

```java
Calling And#0
Calling True
Returning from True true
Calling And#1
Calling Or#0
Calling True
Returning from True true
Calling False
Returning from False false
Return from Or#0 true
Calling Not#0
Calling And#2
Calling True
Returning from True true
Calling False
Returning from False false
Return from And#2 false
Return from Not#0 true
Return from And#1 true
Return from And#0 true
```

## How to use this project
Clone the repository with: 

```
git clone https://github.com/unnikked/BooleanExpressionEvaluator.git
```

Compile:

```
javac src/tk/unnikked.booleanevaluator/*.java src/tk/unnikked/booleanevaluator/*/*.java src/tk/unnikked/booleanevaluator/ast/*/*.java
```

And execute it with (remember to `cd src`): 

```
java tk/unnikked/booleanevaluator/BooleanEvaluator
```

You can also pipe in a file using `-f` directive

```
cat yourfile | java tk/unnikked/booleanevaluator/BooleanEvaluator -f
```

Here an execution example:

```
$ cat test | java tk/unnikked/booleanevaluator/BooleanEvaluator -f
(true & ((true | false) & !(true & false)))
AST: (true & ((true | false) & !(true & false)))
RES: true
```

You can execute `BooleanEvaluator` by providing a text file from your shell or interactively.

# Conclusions
This might not be the fanciest and the most efficient implementation of an interpreter, there are many other tecniques that will let you to achieve far better results, but it's a good point of start to play with context-free-grammars.
