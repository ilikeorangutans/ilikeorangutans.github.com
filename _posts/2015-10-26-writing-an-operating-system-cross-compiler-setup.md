---
layout: post
title: "Writing an Operating System - Environment Setup"
description: "Been toying with writing my own operating system kernel. Here are some notes on setting up the gcc cross compiler."
category: 
tags: [Kernel, C, GCC, Binutils, make, osdev, cross compile]
---
{% include JB/setup %}

I've been reading [The little book about OS development](http://littleosbook.github.io/) and [wiki.osdev.org](http://wiki.osdev.org/) and took some notes along the line. Here's what I wrote on environment setup. 

# Environment setup
You'll need a cross compile toolchain consisting of [GNU Binutils](https://www.gnu.org/software/binutils/) and [gcc](https://gcc.gnu.org/). The [osdev wiki](http://wiki.osdev.org/GCC_Cross-Compiler) has a great page on setting up a cross compilation toolchain. 

It took me a few times because I didn't read the instructions properly. It is important to unpack the sources for binutils and gcc and have separate build directories, `gcc-4.2` and `gcc-build` for example. Then, for building gcc, make sure the newly built binutils are on your path. 

And finally, when building gcc, make sure you run `make all-gcc`, `make all-target-libgcc`, `make install-gcc`, and `make install-target-libgcc` instead of just `make all` like I did. 

Also, running make with `-j 4` or any value larger than 1 greatly sped up the build on my machine. Multicore for the win!

Once you've built both binutils and gcc you're ready to start building your kernel!