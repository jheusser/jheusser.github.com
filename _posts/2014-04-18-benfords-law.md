---
layout: post
title: Using Benfords Law to identify fake Bitcoin exchange transactions
---

{{ page.title }}
================

{{:toc}}

<p class="meta">18 April 2014</p>

# Fake volumes

Bitcoin exchanges are booming -- on a weekly basis new exchanges appear out of nowhere, some already with considerable volume (for new exchanges). Clearly, every trader wants to trade on the most liquid exchange, as that exchange most likely has the best prices and fastest executions. Therefore there's a big incentive for new exchanges to attract liquidity somehow, either through a good fee structure or just by spending more money on marketing. For less honest exchanges there's another option: simply faking transaction data.

Recently, one of the bigger Chinese Bitcoin exchanges got <a href="http://www.coindesk.com/chinese-bitcoin-exchange-okcoin-accused-faking-trading-data/">accused of faking transaction data</a> in order to appear more attractive than they really were. 

Could you systematically find evidence for such fraud without having to have access to the internal accounting of such exchanges?

# Benford's Law

There's a very unintuitive observation when it comes to a lot of types of data: count the occurrences of the leading digit of a couple of numbers from the same source. How frequently will each digit 1 .. 9 appear? We humans usually assume the leading digits are equally likely. However that is often not the case at all. The digit 1 is much more likely than the digit 2, the digit 2 is more likely than 3, and so on down to 9. Why is that?! There are a lot of explanations for that, which go under <a href="https://en.wikipedia.org/wiki/Benford's_law">Benford's Law</a>.

Not all data sources follow Benford's law but often when the data is the outcome of an exponential growth process, like financial data, then the law applies. Imagine you pick a number 1 .. 9 equally likely, let's start with 1. Now doubling a number with a leading 1 will lead to a number with a leading 2 or 3 equally likely. However when the first digit is one of 5,6,7,8,9 then the next leading digit has to be 1. This should give you an intuition for why this phenomena occurs. A great explanation of the law can be found in this <a href="http://plus.maths.org/content/looking-out-number-one">maths.org article</a>.

# Detecting fraud

As often with fraud detection, it boils down to humans being terrible at coming up with random numbers. Sometimes you can catch fraudulent activities by the opposite idea: people trying to be _too_ random because they do not understand the underlying data generating process. This is exactly why Benford's Law can be effective: when data is expected to follow the law but it does not at all then something is fishy.

What do we expect to see if we apply this to Bitcoin exchange data?

I wrote <a href="https://github.com/jheusser/benford">a few of lines</a> of Python (inspired by <a href="http://www.johndcook.com/blog/2011/10/19/benfords-law-and-scipy/">this article</a>) which applies the law to a couple of days of <a href="http://www.btc-e.com">BTC-E</a>  non-zero price returns. It's very nice to see how well Benford's expected distribution almost exactly matches the trade data. 

<img src ="/images/btce_benford.png" alt="Benfords Law on Bitcoin returns" align="center" title="Benfords Law on Bitcoin returns" style="max-height: 700px; max-width: 700px;"></img>

We can see how the leading digit 1 appears about 30% of the time and all other leading digits decay nicely, almost exactly matching as expected by Benford's Law.

Please feel free to run the same script against your favorite exchange data! <a href="https://github.com/jheusser/benford">Get the code here</a>.

