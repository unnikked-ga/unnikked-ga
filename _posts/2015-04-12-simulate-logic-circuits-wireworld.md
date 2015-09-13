---
layout:     post
title:      "Simulate logic circuits using Wireworld cellular automaton"
subtitle:   "you can even do 3D circuits"
date:       2015-04-12 14:20:00
author:     "Nicola Malizia"
tags: ["cellular automata", "python"]

twitter-card: true
twitter-image: "data/wireworld.png"

open-graph: true
open-graph-image: "data/wireworld.png"
---

Cellular automata never stop amusing me. From simple rules you can build complex things. 

I introduced in one of my last blog post [elementary cellular automaton](how-to-simulate-elementary-cellular-automaton-on-logisim) implemented in logisim. We discovered that the **Rule 110** is proved to be [Turing Complete](http://en.wikipedia.org/wiki/Turing_completeness). 

A cellular automaton consists of a regular grid of cells, each in one of a finite number of states, such as on and off in contrast to a coupled map lattice. The grid can be in any finite number of dimensions. For each cell, a set of cells called its neighborhood is defined relative to the specified cell. An initial state time t=0 is selected by assigning a state for each cell. A new generation is created advancing t by 1, according to some fixed rule generally, a mathematical function that determines the new state of each cell in terms of the current state of the cell and the states of the cells in its neighborhood. Typically, the rule for updating the state of cells is the same for each cell and does not change over time, and is applied to the whole grid simultaneously.

In this blog post I want to show you another interesting cellular automaton: **Wireworld**. 

Wireworld is a cellular automaton first proposed by Brian Silverman in 1987. Wireworld is particularly suited to simulating electronic logic elements, or "gates", and, despite the simplicity of the rules and it is Turing-complete.

## Rules

A Wireworld cell can be in one of four different states:

- Empty
- Electron head 
- Electron tail
- Conductor 

At each ticks cells behave as follows:

- `Empty -> Empty`
- `Electron head -> Electron tail`
- `Electron tail -> Conductor`
- `Conductor -> Electron head` if exactly one or two of the neighbouring cells are electron heads, or remains `Conductor` otherwise.

Wireworld uses what is called the [Moore neighborhood](http://en.wikipedia.org/wiki/Moore_neighborhood), which means that in the rules above, neighboring means one cell away (range value of one) in any direction, both orthogonal and diagonal as we saw in the article regarding [elementary cellular automaton](how-to-simulate-elementary-cellular-automaton-on-logisim) . 

Those simple rules are capable of expressing the behavior of the basic logic gates.

<p align="center"><img class="img-responsive" src="http://mathworld.wolfram.com/images/gifs/wireworl.gif"></p>

I've implemented a Java application that simulates this cellular automaton. However I'm not going to explain here how I managed to code it, since it is very simple to implement, check out the various implementations on [Rosetta Code](http://rosettacode.org/wiki/Wireworld). 

I invite you do check our my project [here](https://github.com/unnikked/Wireworld), feel free to use it. 

Using my application I recreated the **Rule 90** elementary cellular automaton.

<p align="center"><img class="img-responsive" src="https://unnikked.ga/data/wireworld-rule90.png"></p>

You can import it and check it out on my application, just take a look into the `examples` folder.

Wireworld it is not good only for simulating elementary cellular automata, David Moore and Mark Owen managed to [implement](http://www.quinapalus.com/wi-index.html) an entire computer in Wireworld. 

<p align="center"><img class="img-responsive" src="https://unnikked.ga/data/wireworld-computer.gif"></p>

Pretty impressive for such simple rules. 

## Wireworld 3D

I want to show your how I managed to create a 3D version of Wireworld. I used as base the implementation of a [rudimental](https://github.com/fogleman/Minecraft) Minecraft environment in python by [Michael Fogleman](https://github.com/fogleman).

Here it is my [modified](https://github.com/unnikked-ga/Minecraft) version. 

The first concept that I had to develop in 3D dimension was the Moore neighborhood. It is simple to think it as a `3x3x3` cube. I imagined it as this picture. 

<p align="center"><img class="img-responsive" src="https://unnikked.ga/data/moore-neighborhood-3d.png"></p>

Once figured out it the implementation was really, really straightforward:

```python
def wireworld(self, position, texture):
        x, y, z = position

        texture = self.world[position]

        if texture is CONDUCTOR:
            count = 0
            for dx in [-1, 0, 1]:
                for dy in [-1, 0, 1]:
                    for dz in [-1, 0, 1]:
                        key = (x + dx, y + dy, z + dz)
                        if key in self.world:
                            if self.world[key] is ELECTRON_HEAD:
                                count += 1
            if 1 <= count <= 2 and count != 0:
                self._enqueue(self.add_block, (x, y, z), ELECTRON_HEAD)
            return
                
        if texture is ELECTRON_HEAD:
            self._enqueue(self.add_block, (x, y, z), ELECTRON_TAIL)
            return
                        
        if texture is ELECTRON_TAIL:
            self._enqueue(self.add_block, (x, y, z), CONDUCTOR)
            return
```

<p align="center"><img class="img-responsive" src="https://unnikked.ga/data/wireworld-3d.png"></p>

If you want to check out the full commit click [here](https://github.com/unnikked-ga/Minecraft/commit/7a28e09fd1d51c427adb54ae0d8c85d75a76fcf3). 

