---
layout: post
title: Local Order 
---

{{ page.title }}
================

<p class="meta">16 June 2011</p>

Quite some time ago I implemented the analysis described in [1] in R. It was long and unreadable -- I reimplemented it in Python in [46 lines](https://github.com/jheusser/local-order/blob/master/localorder.py). The function <i>local_uncertainty</i> calculates the next state uncertainty after a measured trajectory ngram. An ngram describes the quantised stock movements of the last n days, e.g. for 5 days. The local uncertainty h_5 then describes the uncertainty of the next symbol.

The example usage below describes the analysis from the paper. 

{% highlight python %}
import localorder
from pandas import Series

def part(x):
    if x < -0.0025:
        return 0
    elif x > 0.0034:
        return 2
    else:
        return 1

# get returns series and quantise
ret = Series(log(dji['DJI.Close'])).diff(1)
dji_binary = map(part, ret)

# local uncertainty of the 6th symbol when 5 symbols have been seen
order = localorder.local_uncertainty(dji_binary, 5, 3)

{% endhighlight %}

Looking at the local uncertainty time series shows a similar plot than in the paper. The uncertainty is mostly very close to one, with occiasional dips below 90%.

<img src ="/images/uncertainty.png" alt="Local Uncertainty" align="left" width="90%" title="Local Uncertainty" class="img"</img>


References
----------

[1] Lutz Molgedey and Werner Ebeling, <i>Local Order, Entropy and Predictability of Financial Time Series</i>,Eur. Phys. J. B 15, 733–737 
