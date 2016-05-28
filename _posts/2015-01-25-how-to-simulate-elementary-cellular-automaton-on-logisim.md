---
layout:     post
title:      "How to simulate elementary cellular automaton on Logisim"
subtitle:   ""
date:       2015-01-25 18:23:00
author:     "Nicola Malizia"
tags: ["cellular automata", "logisim"]

twitter-card: true
twitter-image: "https://unnikked.ga/data/elementary-cellular-automaton.png"

open-graph: true
open-graph-image: "https://unnikked.ga/data/elementary-cellular-automaton.png"
---

A cellular automaton is a discrete model studied in computability theory, mathematics, physics, complexity science, theoretical biology and microstructure modeling.

> A cellular automaton consists of a regular grid of cells, each in one of a finite number of states, such as on and off (in contrast to a coupled map lattice). The grid can be in any finite number of dimensions. For each cell, a set of cells called its neighborhood is defined relative to the specified cell. An initial state (time t = 0) is selected by assigning a state for each cell. A new generation is created (advancing t by 1), according to some fixed rule (generally, a mathematical function) that determines the new state of each cell in terms of the current state of the cell and the states of the cells in its neighborhood. Typically, the rule for updating the state of cells is the same for each cell and does not change over time, and is applied to the whole grid simultaneously. <sup>[<a href="http://en.wikipedia.org/wiki/Cellular_automaton" target="_blank">1</a>]</sup>

There are two main ways to define the concept of neighborhood in two dimensional CA (can be extended for other dimensions) :

<p align="center"><img src="https://unnikked.ga/data/ca-moore.png" class="img-responsive"><a href="http://en.wikipedia.org/wiki/Moore_neighborhood">Moore Neighborhood</a></p>

That consists to count as neighbour the frame around the blue cell.

<p align="center"><img src="https://unnikked.ga/data/ca-von-neumann.png" class="img-responsive"><a href="http://en.wikipedia.org/wiki/Von_Neumann_neighborhood">Von Neumann Neighborhood</a></p>

That consists the red cells around the blue one and eventually (in some applications) also the pink one.

This model fascinated me for the property that simple rules can lead to complex behavior such the <a href="http://en.wikipedia.org/wiki/Conway%27s_Game_of_Life" title="Conway's Game of Life Cellular Automaton" target="_blank">Conway's Game of Life</a> which was proved to be <a href="http://en.wikipedia.org/wiki/Turing_completeness" title="Turing Completeness on Wikipedia" alt="_blank">Turing Complete</a>.

In this blog post I'm going to describe how I simulated the most simple, but yet powerful, class of cellular automaton which is the <a href="http://en.wikipedia.org/wiki/Elementary_cellular_automaton" title="Elementary Cellular Automaton" target="_blank">elementary cellular automaton</a>.

<h2>Elementary cellular automaton</h2>

An elementary cellular automaton is a one-dimensional cellular automaton where there are two possible states (labeled 0 and 1) and the rule to determine the state of a cell in the next generation depends only on the current state of the cell and its two immediate neighbors.

There are 8 = 2<sup>3</sup> possible configurations for a cell and its two immediate neighbors. The rule defining the cellular automaton must specify the resulting state for each of these possibilities so there are 256 = 2<sup>2<sup>3</sup></sup> possible elementary cellular automata.

The mathematician <a href="http://en.wikipedia.org/wiki/Stephen_Wolfram" target="_blank">Stephen Wolfram</a> studied such mathematical model thoroughly in his book <i>A New Kind of Science</i> and proposed the so called <a href="http://en.wikipedia.org/wiki/Wolfram_code" target="_blank">Wolfram Code</a> to describe this particular class of cellular automaton.

Since there are already tons of programs that shows how to simulate such class of cellular automaton (check <a href="http://rosettacode.org/wiki/Elementary_cellular_automaton" title="Elementary cellular automaton on Rosetta Code" target="_blank">here</a> on Rosetta Code) I've decided to challenge myself and I tried to implement the complete Wolfram Code via an electronic circuit using <a href="http://www.cburch.com/logisim/" title="Elementary cellular automaton in logisim" target="_blank">Logisim</a>.

<blockquote>Logisim is an educational tool for designing and simulating digital logic circuits. With its simple toolbar interface and simulation of circuits as you build them, it is simple enough to facilitate learning the most basic concepts related to logic circuits. With the capacity to build larger circuits from smaller subcircuits, and to draw bundles of wires with a single mouse drag, Logisim can be used (and is used) to design and simulate entire CPUs for educational purposes.
</blockquote>

I'm sure that my implementation could be optimized in order to use less logic gates but unfortunately I'm not an electronic guy and I managed to solve this task using the boolean algebra that helps us to describe the logic of a circuit, I'm open to see how can this task can be solved in a clever way.

Before to continue I have to explain how a rule is applied on a one dimensional grid.

Basically at each step of the simulation we inspect each cell (call it a bit) and its two neighbors and apply the rule according a table like that:

<table class="table">
<thead>
<tr>
  <th align="center">111</th>
  <th align="center">110</th>
  <th align="center">101</th>
  <th align="center">100</th>
  <th align="center">011</th>
  <th align="center">010</th>
  <th align="center">001</th>
  <th align="center">000</th>
  <th align="center"><strong>current pattern</strong></th>
</tr>
</thead>
<tbody><tr>
  <td align="center">0</td>
  <td align="center">1</td>
  <td align="center">1</td>
  <td align="center">0</td>
  <td align="center">1</td>
  <td align="center">1</td>
  <td align="center">1</td>
  <td align="center">0</td>
  <td align="center"><strong>new state</strong></td>
</tr>
</tbody></table>

In this case we are describing the rule 110 because is 01101110<sub>2</sub>.

If you take a look at the table at the second row you see that the base 2 of the number rule determines the next cell state according to the tree cells in input.

Also the first row is the numeration from 0 to 7 that represents  8 = 2<sup>3</sup> possible configurations for a cell and its two immediate neighbors as show before.

So it naturally comes into my mind to a <a href="http://en.wikipedia.org/wiki/Multiplexer" title="Multiplexer for mapping rule for elementary cellular automaton electronic circuit" target="_blank">multiplexer</a> for mapping all the Wolfram Code using only one basic circuit.

<p align="center"><img src="https://unnikked.ga/data/ca-rulemapping.png" class="img-responsive"></p>

Where the control bits are the current cell with his two neighbors which determines the new state cell.

Since it is only for one bit of the current state I managed to create a 32 bit module in order to be able to see how the input string evolves during several cycles.

<p align="center"><img src="https://unnikked.ga/data/rule-mapping.png" alt="8 bit module" class="img-responsive"></p>

This is the 8 Bit Module. (I would have made only a single module and combining them to have more order into the schematic)

<p align="center"><img src="https://unnikked.ga/data/rule-mapping-16-bit.png" alt="16 bit module" class="img-responsive"></p>

This is the 16 Bit Module.

<p align="center"><img src="https://unnikked.ga/data/rule-mapping-32-bit.png" alt="32 bit module" class="img-responsive"></p>

This is the 32 Bit Module.

In order to display the time-space diagram of a given rule I connected 16 of this modules to compute 16 steps.

<p align="center"><img src="https://unnikked.ga/data/16-step-32-bit.png" alt="Elementary Cellular Automaton in Logisim" class="img-responsive"></p>

And the main circuit wired into a led display provided by the standard components of Logisim:

<p align="center"><img src="https://unnikked.ga/data/elementary-cellular-automaton-logisim.png" alt="Elementary Cellular Automaton in Logisim" class="img-responsive"></p>

You can see it in action <a href="https://www.youtube.com/watch?v=3cRRnjiVbIQ" title="Elementary Cellular Automaton in Logisim" target="_blank">here</a> on my YouTube channel.

You can also download the source code of the schematics <a href="https://github.com/unnikked/LogisimWorkspace/blob/master/cellular%20automata/elementary%20cellular%20automata.circ" target="_blank">here</a>.

## Some remarkably rules
There are some rules that produces some interesting behavior.

**Rule 30** shows enough <a href="http://www.cs.indiana.edu/~dgerman/2005midwestNKSconference/dgelbm.pdf" title="Is Rule 30 Random?" target="_blank">randomness</a> that can be used as a <a href="http://en.wikipedia.org/wiki/Pseudorandom_number_generator" target="_blank">pseudorandom number generator</a>.

Here a <a href="http://rosettacode.org/wiki/Elementary_cellular_automaton/Random_Number_Generator#C" title="Rule 30 Random Number Generator" target="_blank">C</a> code snippet that shows this behavior.

```c
#include <stdio.h>
#include <limits.h>

typedef unsigned long long ull;
#define N (sizeof(ull) * CHAR_BIT)
#define B(x) (1ULL << (x))

void evolve(ull state, int rule)
{
	int i, p, q, b;

	for (p = 0; p < 10; p++) {
		for (b = 0, q = 8; q--; ) {
			ull st = state;
			b |= (st&1) << q;

			for (state = i = 0; i < N; i++)
				if (rule & B(7 & (st>>(i-1) | st<<(N+1-i))))
					state |= B(i);
		}
		printf(" %d", b);
	}
	putchar('\n');
	return;
}

int main(void)
{
	evolve(1, 30);
	return 0;
}
```

```
220 197 147 174 117 97 149 171 100 151
```

**Rule 90** generates graphically the <a href="http://en.wikipedia.org/wiki/Sierpinski_triangle" title="Sierpinski triangle" target="_blank">Sierpinski triangle</a>.

<p align="center"><img src="https://unnikked.ga/data/r090-pulse-wide.png" alt="Rule 90 Sierpinski triangle" class="img-responsive"></p>

**Rule 110** was <a href="http://www.complex-systems.com/pdf/15-1-1.pdf" title="Turing Completeness of Rule 110" target="_blank">demonstrated</a> to be Turing Complete.

**Rule 184** notable for solving the <a href="http://en.wikipedia.org/wiki/Majority_problem_(cellular_automaton)" target="_blank">majority problem</a> as well as for its ability to simultaneously describe several, seemingly quite different, <a href="http://en.wikipedia.org/wiki/Particle_system" target="_blank">particle systems</a>.

# Conclusion
While I'm not an expert in this field In my opinion cellular automata are a powerful mathematical model to describe complex system using only simple rules. And makes me wonder if our universe can also be described by a set of "easy" rules. Are our intelligence designed by purpose or it is only the results of the interaction of simple rules ?
