---
layout: post
title: Proof of concept code for SAT-based bitcoin mining
---

{{ page.title }}
================

{{:toc}}

<p class="meta">30 December 2013</p>

# Introduction

Earlier this year I [published some research](http://jheusser.github.io/2013/02/03/satcoin.html) which describes an alternative algorithm to solve the Bitcoin mining problem. In this blog post I'm going to release the example SAT-based Bitcoin miner which was used as the basis for that research and benchmarks. It consists of a C implementation which serves as input to the [CBMC](http://www.cprover.org/cbmc/) model checker. The SAT-based model checker then performs the nonce search by trying to find a counter-example to the hypothesis that no nonce exists in the given block. This leads to either a counter-example which contains a nonce in its trace, or a proof that no valid nonce exists.

Keep in mind, this is research code which is unoptimised and not production ready. It is not meant as a direct competitor to the current brute force mining. However, it does demonstrate a fully working alternative algorithm to find valid nonces.

# Usage 

The code can be found in this [github repo](https://github.com/jheusser/satcoin). It consists of one file _[satcoin.c](https://github.com/jheusser/satcoin/blob/master/satcoin.c)_ which contains a custom, CBMC-friendly SHA-256 implementation (i.e. fixed-sized loops, no heap allocation, etc) and the assumptions and assertions necessary to define the SAT-based nonce search.

The code contains the [Genesis block](https://blockexplorer.com/block/000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f) header by default (see the variable `input_block`) and assumes a search over only two different nonces for demonstration purposes. An example run with default settings should look similar to:
{% highlight python %}
$ cbmc --slice-formula --32 -DCBMC satcoin.c
size of program expression: 4867 assignments
slicing removed 528 assignments
Generated 1 VCC(s), 1 remaining after simplification
Passing problem to propositional reduction
Running propositional reduction
Solving with MiniSAT2 with simplifier
264289 variables, 893304 clauses
...
Violated property:
  file satcoin.c line 165 function verifyhash
  assertion
  flag == 1

VERIFICATION FAILED
{% endhighlight %}
Here, CBMC prints out a counter-example which contains and demonstrates the existence of a valid nonce for the Genesis block. The nonce can be found in the counter-example trace as described in the [original article](http://jheusser.github.io/2013/02/03/satcoin.html).

# Implementation Details

The file _satcoin.c_ can be compiled by GCC to ensure that the correct hash is actually being computed by the code for the given `input_block`. To run it with CBMC in "verification mode" just add the _-DCBMC_ flag to it. 

The global variable `input_block` contains 20 32-bit integers of the first 640 bits of the block header which should be solved. The last integer is the nonce which can be filled with zeros or arbitrary numbers.

The actual SHA-256 code is being computed in the function `verifyhash` which contains CBMC pre- and postconditions defining the SAT-based bitcoin mining algorithm. The precondition defines a pointer into the nonce field which is first set to a non-deterministic value. Depending on your experiment, the nonce search can be restricted to a range of values. 
{% highlight c %}
  unsigned int *u_nonce = ((unsigned int *)block+16+3);
  *u_nonce = nondet_uint();

   __CPROVER_assume(*u_nonce > 497822587 && *u_nonce < 497822589); // 1 nonces only
{% endhighlight %}

The postcondition assumes the leading zeros of a valid hash and then asserts that the correct bit is non-zero. 

{% highlight c %}
  __CPROVER_assume(
     (unsigned char)(state[7] & 0xff) == 0x00 &&
     (unsigned char)((state[7]>>8) & 0xff)  == 0x00 &&
     (unsigned char)((state[7]>>16) & 0xff) == 0x00); 

  int flag = 0;
  if((unsigned char)((state[7] >> 24) & 0xff) != 0x00) {
          flag = 1;
  }

  assert(flag == 1);
{% endhighlight %}

The pre- and postcondition together effectively specify the structure of a hash that satisfies the Bitcoin mining protocol. If such a hash exists then the valid nonce can be found in the counter-example trace provided by the model checker.

For more information check the [original article](http://jheusser.github.io/2013/02/03/satcoin.html), or have a look at the [code](https://github.com/jheusser/satcoin).

