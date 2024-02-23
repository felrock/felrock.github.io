---
layout: post
title:  "Visualize precision with harmonic convergence"
date:   2024-02-23 12:00:00 +0100
categories: script
---

## Visualize precision with harmonic convergence

This came about while I was looking at different implementations of half-precision(float16) in C++.
Wonder, what's the difference between bfloat16 and half-precision floats? Why do they prefer to use
bfloat16 in some ML projects?

Some background, harmonic convergence is this sequence `Hs = 1/2 + 1/3 + 1/4 + 1/5 ...`. The
sequence reaches a convergence when `1/x == 0`.

So I made a simple C++ implementation of the harmonic convergence, and ran it with the 4 different
floating point implementations and sizes.
```bash
git clone https://github.com/felrock/harmonic-converge-cpp
cd harmonic-converge-cpp && g++ main.cpp half.hpp bfloat16.h
./a.out
```

The program will take a very long time to finish, since converging a double takes a really really
long time.

But the result were,
```
native float = [value: 15.4037, steps: 2097153]
half precision float = [value: 5.74609, steps: 258]
bfloat16 = [value: 5.0625, steps: 66]
```

bfloat16 converges really quickly. Compared to the other two implementations of float16. And for
machine learning projects, when using back propagation, really small deltas will equal 0. This might
make it easier for the models to handle noisy data better, and sort of discard really small changes.
That's my speculation about the whole thing at least.


bfloat16 fast convergence and the fact that it takes double several days to finish on a off-the-shelf
computer was the most interesting take-away from this experiment.


[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
