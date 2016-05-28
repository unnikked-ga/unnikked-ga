---
layout:     post
title:      "How to execute a Markov algorithm"
subtitle:   "string rewriting system with the power of Turing completeness"
date:       2015-05-19 11:34:00
author:     "Nicola Malizia"
tags: ["algorithms", "php"]

twitter-card: true
twitter-image: "https://unnikked.ga/data/markov-algorithm.png"

open-graph: true
open-graph-image: "https://unnikked.ga/data/markov-algorithm.png"
---

A Markov algorithm is a [string rewriting](http://en.wikipedia.org/wiki/Semi-Thue_system) system that uses grammar-like rules to operate on strings of symbols. Markov algorithms have been shown to be [Turing-complete](http://en.wikipedia.org/wiki/Turing_complete), which means that they are suitable as a general model of computation and can represent any mathematical expression from its simple notation.<sup>[*](http://en.wikipedia.org/wiki/Markov_algorithm)</sup>

String rewriting system is a rewriting system over strings from a usually finite alphabet.<sup>[*](http://en.wikipedia.org/wiki/Semi-Thue_system)</sup>

<h2>The algorithm</h2>

The Rules is a sequence of pair of strings, usually presented in the form of `pattern -> replacement`. Each rule may be either ordinary or terminating.

Given an input string:

1. Check the Rules in order from top to bottom to see whether any of the patterns can be found in the input string.
2. If none is found, the algorithm stops.
3. If one (or more) is found, use the first of them to replace the leftmost occurrence of matched text in the input string with its replacement.
4. If the rule just applied was a terminating one, the algorithm stops.
5. Go to step 1.

Note that after each rule application the search starts over from the first rule.

<h2>The implementation</h2>
Implementing this algorithm in PHP is simple, the use of [`preg_match`](http://php.net/manual/en/function.preg-match.php) and [`preg_replace`](http://php.net/manual/en/function.preg-replace.php) will help us. So let's take a look to the implementation:

```php
<?php
while(true) {
    echo $input . "\n";
    $run = false;
    foreach ($rules as $rule => $replacement) {
        if (preg_match($rule, $input)) {
            $input = preg_replace($rule, $replacement, $input, 1);   
            if (isset($replacement{0}) && $replacement{0} === ".") {
                $run = false;
            } else {
                $run = true;
            }           
            break;
        }
    }

    if(!$run) {
        break;
    }
}
```

We need to iterate on the `$input` over and over again until no rules can be applied.

Since we need to apply the first matches of a rule at the leftmost occurrence we limit the `preg_match` function at `1` and we break the for each loop.

At the end of all replacements `$run` will be `false`, hence we `break` also the endless loop.

<h2>Some ruleset</h2>

I found on Internet some interesting rules that I would like to share with you and helped me test my implementation.  

<h3>Binary to unary</h3>
These rules converts a binary representation of a number into it's unary counterparts.

For example `101` (`5`) becomes `|||||`

```php
<?php
$rules = [
    "/\|0/" => "0||",
    "/1/" => "0|",
    "/0/" => ""
];

$input = "1111";
```
Output:
```
1111
0|111
0|0|11
00|||11
00|||0|1
00||0|||1
00|0|||||1
000|||||||1
000|||||||0|
000||||||0|||
000|||||0|||||
000||||0|||||||
000|||0|||||||||
000||0|||||||||||
000|0|||||||||||||
0000|||||||||||||||
000|||||||||||||||
00|||||||||||||||
0|||||||||||||||
|||||||||||||||
```

<h2>Binary Adder</h2>
This rule can handle parenthesized expression with binary numbers. For example `((111+1)+((1+10)+100))` will produce `1111`<sup>[*](http://utilitymill.com/utility/markov_rewriter)</sup>.

```php
<?php
$rules = [
    '/yy/' => 'ERROR! (Fix your parenthesis.)  Dump: ',
    '/0#/'=> '1',
    '/1#/'=> '#1',
    '/x#/'=> 'x1',
    '/0c0a/'=> 'a0',
    '/0c1a/'=> 'a1',
    '/1c0a/'=> 'a1',
    '/1c1a/'=> '#a0',
    '/0ya/'=> 'c0a',
    '/1ya/'=> 'c1a',
    '/0y0/'=> '00y',
    '/1y0/'=> '01y',
    '/0y1/'=> '10y',
    '/1y1/'=> '11y',
    '/1y\+x/'=> '+x1y',
    '/0y\+x/'=> '+x0y',
    '/1x/'=> 'x1',
    '/0x/'=> 'x0',
    '/b0/'=> '0b',
    '/b1/'=> '1b',
    '/b\)/'=> '',
    '/\(\+x1a/'=> '1b',
    '/\(\+x0a/'=> '0b',
    '/\(\+xc1a/'=> '1b',
    '/\(\+xc0a/'=> '0b',
    '/\+x/'=> 'y+x',
    '/a\)/'=> '',
    '/\)/'=> 'xa)',
];

$input = "((111+1)+((1+10)+100))";
```
Output:

```
((111+1)+((1+10)+100))
((111+1xa)+((1+10)+100))
((111+x1a)+((1+10)+100))
((111y+x1a)+((1+10)+100))
((11+x1y1a)+((1+10)+100))
((11+x11ya)+((1+10)+100))
((11+x1c1a)+((1+10)+100))
((11+x#a0)+((1+10)+100))
((11+x1a0)+((1+10)+100))
((11y+x1a0)+((1+10)+100))
((1+x1y1a0)+((1+10)+100))
((1+x11ya0)+((1+10)+100))
((1+x1c1a0)+((1+10)+100))
((1+x#a00)+((1+10)+100))
((1+x1a00)+((1+10)+100))
((1y+x1a00)+((1+10)+100))
((+x1y1a00)+((1+10)+100))
((+x11ya00)+((1+10)+100))
((+x1c1a00)+((1+10)+100))
((+x#a000)+((1+10)+100))
((+x1a000)+((1+10)+100))
(1b000)+((1+10)+100))
(10b00)+((1+10)+100))
(100b0)+((1+10)+100))
(1000b)+((1+10)+100))
(1000+((1+10)+100))
(1000+((1+10xa)+100))
(1000+((1+1x0a)+100))
(1000+((1+x10a)+100))
(1000+((1y+x10a)+100))
(1000+((+x1y10a)+100))
(1000+((+x11y0a)+100))
(1000+((+x101ya)+100))
(1000+((+x10c1a)+100))
(1000+((+x1a1)+100))
(1000+(1b1)+100))
(1000+(11b)+100))
(1000+(11+100))
(1000+(11+100xa))
(1000+(11+10x0a))
(1000+(11+1x00a))
(1000+(11+x100a))
(1000+(11y+x100a))
(1000+(1+x1y100a))
(1000+(1+x11y00a))
(1000+(1+x101y0a))
(1000+(1+x1001ya))
(1000+(1+x100c1a))
(1000+(1+x10a1))
(1000+(1y+x10a1))
(1000+(+x1y10a1))
(1000+(+x11y0a1))
(1000+(+x101ya1))
(1000+(+x10c1a1))
(1000+(+x1a11))
(1000+1b11))
(1000+11b1))
(1000+111b))
(1000+111)
(1000+111xa)
(1000+11x1a)
(1000+1x11a)
(1000+x111a)
(1000y+x111a)
(100+x0y111a)
(100+x10y11a)
(100+x110y1a)
(100+x1110ya)
(100+x111c0a)
(100+x11a1)
(100y+x11a1)
(10+x0y11a1)
(10+x10y1a1)
(10+x110ya1)
(10+x11c0a1)
(10+x1a11)
(10y+x1a11)
(1+x0y1a11)
(1+x10ya11)
(1+x1c0a11)
(1+xa111)
(1y+xa111)
(+x1ya111)
(+xc1a111)
1b111)
11b11)
111b1)
1111b)
1111
```

<h2>Turing Machine</h2>
A simple [Turing machine](http://en.wikipedia.org/wiki/Turing_machine), implementing a three-state [busy beaver](http://en.wikipedia.org/wiki/Busy_beaver).

The tape consists of `0`s and `1`s, the states are `A`, `B`, `C` and `H` (for Halt), and the head position is indicated by writing the state letter before the character where the head is. All parts of the initial tape the machine operates on have to be given in the input.

This rule demonstrates that the Markov algorithm is Turing-complete.

```php
<?php
$rules = [
    # Turing machine: three-state busy beaver
    #
    # state A, symbol 0 => write 1, move right, new state B
    "/A0/" => "1B",
    # state A, symbol 1 => write 1, move left, new state C
    "/0A1/" => "C01",
    "/1A1/" => "C11",
    # state B, symbol 0 => write 1, move left, new state A
    "/0B0/" => "A01",
    "/1B0/" => "A11",
    # state B, symbol 1 => write 1, move right, new state B
    "/B1/" => "1B",
    # state C, symbol 0 => write 1, move left, new state B
    "/0C0/" => "B01",
    "/1C0/" => "B11",
    # state C, symbol 1 => write 1, move left, halt
    "/0C1/" => "H01",
    "/1C1/" => "H11"
];

$input = "000000A000000";
```
Output:

```
000000A000000
0000001B00000
000000A110000
00000C0110000
0000B01110000
000A011110000
0001B11110000
00011B1110000
000111B110000
0001111B10000
00011111B0000
0001111A11000
000111C111000
00011H1111000
```
