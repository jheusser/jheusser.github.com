---
layout: post
title: Order Flow Toxicity of the Bitcoin April Crash
---

{{ page.title }}
================

<p class="meta">25 August 2013</p>

# Introduction to Order Flow Toxicity and PIN

Information based market making models aim to explain the bid ask spread due to asymmetric information -- some traders are more informed than others about the true value of the traded asset. Consequently, a market maker who continuously offers bid and ask quotes has to have a buffer (the spread) which protects his quotes from being adversely selected by better informed traders.

An influential paper [1] describes such a model by using the concept of informed and uninformed traders. I will briefly describe the concepts introduced in that paper. 

A perfectly informed trader is one that only buys when an asset is undervalued and only sells when an asset is overvalued. Trading against informed traders is a bad thing since any execution from an informed trader means you received a bad price. This is especially important when you trade with limit orders all day long at a high frequency, such as market makers. Order flow is said to be toxic (from a liquidity provider perspective) if the people you trade against have a better idea than you about the fundamental price. Thus, it is not surprising that a branch of market making research is concerned with estimating the level of informed trading [information based mm papers].

The paper further describes a way to model bid and ask prices based on the arrival of "good" and "bad" information. This leads to the sequential trade model of market making, which derives break even conditions for a liquidity provider given arrival rates which are explained in the next paragraph.

The authors assume uninformed traders always buy and sell no matter whether good or bad news arrived. Informed traders only buy on good news and sell on bad news. 
Information events are independent and happen with probability \\(\alpha\\). Good news happen with probability \\(1-\delta\\), bad news with \\(\delta\\).
Trades come from both informed or uninformed traders and both traders are assumed to be determined by an independent Poisson process. Uninformed traders arrive with rate \\(\epsilon\\); the arrival rate of informed traders is \\(\mu\\).

Thus on a good news day, the arrival rate is \\(\epsilon + \mu\\) as both types of traders trade. On bad news days the arrival rate for buy trades is \\(\epsilon\\) only and for sell trades it is again \\(\epsilon + \mu\\). And so on. 

The authors eventually derive the break even condition provided by the model. This condition contains a factor which can be interpreted as the probability of informed trading occuring. This factor is the following

$$PIN = \frac{\alpha \mu}{\alpha \mu + 2 \epsilon}$$

which can be explained as the proportion of orders originating from informed traders (\\(\alpha \mu\\)) of the total order flow (\\(\alpha \mu + 2 \epsilon\\)).

The various arrival rates above have to be estimated from order arrival data. This turns out to be tricky since none of the parameters can directly be observed in the data, also the assumption on the distribution of arrivals is questionable.


# VPIN

Recently, Easley, de Prado, and OHara [2] have suggested an adapted metric which is easier to estimate and more useful in practice called Volume Synchronized Probability of Informed Trading (VPIN). 


The most important aspect of VPIN is that it is calculated in _volume-time_ instead of wall-clock- or trade-time. In volume-time the time axis consists of equal sizes volume buckets -- i.e. a number of trades are aggregated together which make up a certain fixed amount of volume. As trades [arrive in clusters](http://jheusser.github.io/2013/09/08/hawkes.html) and some trades are more influential than others [7] the authors argue that it makes sense to compare periods of equal information (i.e. volume) exchanged.

They show how VPIN is a good estimate of PIN with expected volume imbalance between sell and buy classified trades in a given bucket being

$$E[|V_S-V_B|]\approx\alpha\mu$$ 

and expected total number of trades is

$$E[V_B+V_S]=\alpha\mu+2\epsilon$$

As the second expection above is constant across the whole trading period (since every volume bucket is the same size) this leads to the following, simple VPIN toxicity measure where the sum is over n volume buckets

$$VPIN=\frac{\alpha \mu}{\alpha \mu + 2 \epsilon}\approx\frac{\sum{E[|V_S-V_B|]}}{nV}$$

To compute VPIN you have to specify the volume size V and an averaging period n. The authors give the example that if V is chosen to be one fiftieth of the daily volume and n=50 then this corresponds to finding a daily VPIN.

# Flash Crash and VPIN as crash metric

In another paper [3] the authors argue that VPIN is a good leading indicator of volatility, as demonstrated by their analysis of [Flash Crash](https://en.wikipedia.org/wiki/2010_Flash_Crash) data which shows growing order flow toxicity during the hours leading up to the crash.

A figure from the paper shows the VPIN measure and E-mini SP500 ticker during the hours of the flash crash.

<p>
<img src ="/images/flash_crash.png" alt="VPIN during Flash Crash" align="center" title="Flash Crash" style="max-height: 700px; max-width: 700px;"></img>
</p>

We see how VPIN gradually increases leading up to the event. It peaks after the event around 0.8 and then slowly decreases. The paper concludes by suggesting VPIN as a real-time indicator of order flow toxicity which could be used by market makers and exchanges alike as a warning of impending market turmoil.

These are quite strong claims and naturally lead to a bit of a dispute by people trying to show how they can't reproduce the volatility leading characteristics of VPIN [4, 5].

I do not take either side of this argument, however it is still interesting to apply VPIN to Bitcoin exchange data. Bitcoin is known to be extremely volatile with [frequent crashes](http://www.forbes.com/sites/timothylee/2013/04/11/an-illustrated-history-of-bitcoin-crashes/); being able to monitor order flow toxicity and react to upcoming volatility would be useful to any type of trader.

So let us have a look at computing VPIN on trade data recorded from MtGox.


# Calculating VPIN using Bitcoin historical data

We examine the time period of 15 April to 3 May. The beginning of the period is a few days after the market crashed from its all time high of around $250 down to around $70. Thus this time frame is interesting as it contains a lot of uncertainty and volatility from the aftermath of the biggest panic selling in the history of bitcoin. The data consists millisecond resolution of almost 470'000 trades of a total volume of 2.5 million BTC.

We use the algorithm described in [6] to calculate VPIN. The only three parameters to decide on are the fixed size of volume buckets, the time bar resolution, and the averaging period. Once the parameters are chosen, time bars and volume bars can be calculated. 

VPIN calculation core is the Order Imbalance (OI). This is the absolute difference of sell and buy volume within one volume bucket. This is computed by converting 1 minute time bars into unit (1BTC) volume transactions. Thus if the median 1 minute volume is 20BTC, then this is transformed into 20 individual transactions of 1BTC, all receiving the trade sign assigned to the given minute. The unit volume transaction series is then grouped into equal sizes volume buckets. OI can be trivially calculated from that.
VPIN is then simply the moving average of OI over a number of volume buckets.

We consider volume buckets of the size of 500 BTC, which corresponds to about one fifteenth of MtGox daily trading volume. These are relatively large buckets compared to the financial VPIN papers, but due to the extreme dispersion of order sizes (median is 0.5BTC while maximum is 3300BTC) and the 1 minute trade sign aggregation to small buckets would result in some buckets containing many small orders while many consecutive buckets being filled by a single large trade.

We also use 1 minute time bars and assign the most common trade sign within that minute as sign. In contrary to financial data, MtGox provides trade signs for every trade, thus we do not have to use trade classification heuristics.

The calculation necessary to recreate the graphs below and the data itself is available on github. I use the excellent Pandas python package which makes alignment, resampling, and grouping of timeseries much easier. One word of warning, I am sure the whole code could be re-written in one or two elegant Pandas groupby (perhaps a VolumeGrouper?) statement.

# Results

The trades over the given period show the initial bottom of around $50, a recovery to $160 at the end of a month and a second, fast depreciation at the beginning of May.

<p>
<img src ="/images/trades_1min.png" alt="Translation" align="center" title="Trades" style="max-height: 700px; max-width: 700px;"></img>
</p>
<p>

The volume over the same period clearly shows the clustered nature of trades.

<p>
<img src ="/images/volume_1min.png" alt="Translation" align="center" title="Volume" style="max-height: 700px; max-width: 700px;"></img>
</p>
<p> 

The next graph shows the time required to fill a volume bucket of 500BTC (on the y-axis). We can see that the duration of a volume bucket varies a lot, from a few seconds during the most active times to a maximum of 155 minutes. Especially right after the second crash on 17 April most buckets are below 5 minutes -- most likely made up of traders who are either getting rid of their inventory or who are 'buying the dip'.

<p>
<img src ="/images/bucket_durations.png" alt="Translation" align="center" title="Durations" style="max-height: 700px; max-width: 700px;"></img>
</p>
<p> 


Also of interest is the (normalised) price distribution of the volume time returns versus the normal chronological time, as shown in [2]. 
The volume-time distribution is closer to an i.i.d normal distribution than the chronological time, which could be used as argument for using volume-time when analysing high frequency trade data.

<p>
<img src ="/images/price_changes.png" alt="Price Distribution" align="center" title="Price Distribution" style="max-height: 700px; max-width: 700px;"></img>
</p>
<p> 

Finally, a plot of VPIN and the price series over the whole period.

<p>
<img src ="/images/vpin.png" alt="VPIN" align="center" title="VPIN" style="max-height: 700px; max-width: 700px;"></img>
</p>
<p> 

# References

[1] D. Easley, N. M. Kiefer, M. O'Hara, J. B. Paperman: Liquidity, Information, and Infrequently Traded Stocks [pdf].

[2] D. Easley, M. M. Lopez de Prado, M. O'Hara: Flow Toxicity and Volatility in a High Frequency World [pdf].

[3] D. Easley, M. M. Lopez de Prado, M. O'Hara: The Microstructure of the 'Flash Crash' [pdf].

[4] T. G. Andersen, and O. Bondarenko: VPIN and the Flash Crash [pdf].

[5] http://insight.kellogg.northwestern.edu/article/the_trouble_with_vpin

[6] D. Abad, and J. Yague: From PIN to VPIN: An introduction to order flow toxicity [pdf].

[7] A. Cartea, S. Jaimungal, and J. Ricci: Buy Low Sell High: A High Frequency Trading Perspective [ssrn](http://papers.ssrn.com/sol3/papers.cfm?abstract_id=1964781).
