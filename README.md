# Ranking S&P 500 stocks with free data

Every Friday, a LightGBM model ranks the stocks of the S&P 500 by how much they should beat their sector over the
next week. It only sees daily prices and volumes from Yahoo Finance, and only the stocks that were in the index on
that day. I wanted to know whether a model like this finds real information, how much of it is new, how long it
lasts and what is left once you pay to trade.

The full method, with every definition and assumption, is in [model_and_strategy.pdf](model_and_strategy.pdf).

## Results

Each year is predicted by a model fitted on earlier years only. Over 2013-2022, the years on which I chose the
settings, the weekly rank correlation between predictions and realised returns, called the IC, averages 0.0161
(Newey-West t = 4.06). Over 2023-2026, which played no part in building the model, it is 0.0135 (t = 1.89).

Most of this is short-term reversal, an effect every quant fund already trades. Once sector, beta, size, 5- and
21-day reversal, momentum and volatility are taken out, the IC drops to 0.0081 (t = 3.23) over 2013-2022 and 0.0050
(t = 1.14) over 2023-2026. Beyond known effects, nothing significant is left since 2020. What remains comes almost
entirely from 2013-2019, the years in which the data miss the most stocks, so it may well be survivorship bias
rather than information.

The information also fades fast. It loses half of its strength in three to four trading days.

A beta-hedged, dollar-neutral portfolio built on the ranking has a Sharpe ratio of 0.42 before costs over 2013-2022
(t = 1.39), which is not significant. It breaks even at about 6 basis points per trade, so it only makes sense at
very low costs and next to other signals.

The 2023-2026 period was opened once, for a test written in advance. Other signals had already been tested on the
same years, so the bar was raised to t = 2.67. The ranking kept its sign there but did not pass.

## Files

The PDF explains the model, the portfolio and every number. The four CSV files in `results` hold the numbers behind
it.

- `ic_weekly_lightgbm.csv` gives the raw and pure IC of every week from 2013 to October 2026, dated by the day of
  the prediction
- `ic_2013_2022.csv` compares the mean weekly IC of the reversal rule, the ridge regression and LightGBM over
  2013-2022, 2013-2019 and 2020-2022
- `decay.csv` shows how the IC of a prediction falls from the first day after it to weeks 5 to 8
- `portfolios_2013_2022.csv` gives the Sharpe ratio, turnover and break-even cost of the two portfolios at each cost

## Forward test

The model was refitted on every week from 2008 to October 2026 and then frozen. Starting on 30 October 2026, it
ranks the current members of the index every weekend, after the Friday close and before Monday's open. The IC of
each week is known about two weeks later.

There is a single test, fixed in advance. On 26 October 2029, the model is said to carry information if the mean
weekly raw IC has a Newey-West t of at least 1.645. The raw IC still contains short-term reversal, so the pure IC
will be reported next to it.

The frozen model and its feature code are not in this repository, but their SHA-256 hashes below were set before
the first live week. If the files are published later, anyone can check that they never changed.

```
2c1cd9d84ce24af3bcc16c16ad89ca9d205fb40b540836e9709a977acbe1c033  model.txt
74f8194975e892317bcf5a4d23791fa49901aa5f78005218025e84cf2e934dfd  xs_features.py
```

## Limits

Yahoo has no prices for most companies that left the index. Only 71% of the members have a price in 2013, against
92% in 2022, and the missing ones are mostly companies that did badly. That flatters any strategy that buys recent
losers. The portfolio also ranks stocks across sectors, so it takes sector bets the model was never trained to
predict. Costs are a flat number of basis points with no model of market impact, and the results only cover large
US stocks over a one-week horizon.
