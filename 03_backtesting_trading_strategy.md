
# Backtesting A Trading Strategy Part 3

## How To Backtest A Mean-Reverting Trading Strategy Using Python

## Introduction
Backtesting is a tool to measure the performance of a trading strategy using historical data. The backtesting process consists of three parts: 1. determining the universe of securities where we will invest in (e.g. equity or fixed income? US or emerging markets?); 2. gathering historical data for the universe of securities; and 3. implementing a trading strategy using the historical data collected.  

In the previous articles, I illustrated the first two steps in the backtesting process of [determining the universe of stocks](https://medium.com/@dinodecastro/backtesting-a-trading-strategy-part-1-11229d380fb9), and [collecting historical data for each constituent](https://medium.com/@dinodecastro/backtesting-a-trading-strategy-part-2-2b4bedd3712d). In this article, I will illustrate the last step of the process: implementing a mean-reverting trading strategy.

## A Mean-Reverting Trading Strategy
We implement a mean-reverting trading strategy based on [Khandani and Lo](http://web.mit.edu/Alo/www/Papers/august07.pdf). The idea is to buy the previous day's "losers", and sell the previous day's "winners". Stocks which underperform the market average are classified as "losers"; while stocks which outperform the market average are classified as "winners".

For each stock $i$, we calculate the weight $w_{i, t}$ at time $t$

$$ w_{i, t} = - \frac{1}{N} \left( r_{i, t-1} - r_{Market, t-1} \right). $$

where $N$ is the total number of stocks in the investment universe. Market return at time $t-1$ is calculated as

$$ r_{Market, t-1} = \frac{1}{N} \sum_{i=1}^{N} r_{i, t-1}, $$

and return of stock $i$ at time $t-1$ is calculated as

$$ r_{i, t-1} = \frac{S_{i, t-1} - S_{i, t-2}}{S_{i, t-2}}, $$

where $S_{i, t-1}$ is the spot price of stock $i$ at time $t-1$.

## Backtesting A Trading Strategy
### Step By Step
1. Load S&P indices historical data
2. Implement a mean-reverting trading strategy
3. Calculate the trading strategy's performance using the Sharpe ratio

You can find the code on https://github.com/DinodC/backtesting-trading-strategy.

Import packages


```python
import numpy as np
```


```python
import pandas as pd
```


```python
from pandas import Series, DataFrame
```


```python
import pickle
```


```python
import matplotlib.pyplot as plt
```


```python
%matplotlib inline
```

### Load S&P Historical Data

Set an id for each index


```python
id = ['sp500', 'sp400', 'sp600']
```

Create a dictionary to map each id to a tickers file


```python
input_file = {'sp500': 'sp500_data.pickle',
              'sp400': 'sp400_data.pickle',
              'sp600': 'sp600_data.pickle'}
```

Create a dictionary to map each id to a S&P historical data


```python
sp500_data = pd.DataFrame()
sp400_data = pd.DataFrame()
sp600_data = pd.DataFrame()
sp_data = {'sp500': sp500_data,
           'sp400': sp400_data,
           'sp600': sp600_data}
```

Retrieve S&P historical data


```python
for i in id:
    # Load data
    with open(input_file[i], 'rb') as f:
        sp_data[i] = pickle.load(f)
    f.close()

    # Select close prices
    sp_data[i] = sp_data[i].close
```

### Implement A Mean-Reverting Trading Strategy

Create a dictionary to map each id to a S&P returns data


```python
sp500_returns = DataFrame()
sp400_returns = DataFrame()
sp600_returns = DataFrame()
sp_returns = {'sp500': sp500_returns,
              'sp400': sp400_returns,
              'sp600': sp600_returns}
```

Create a dictionary to map each id to a S&P market returns data


```python
sp500_market_returns = Series()
sp400_market_returns = Series()
sp600_market_returns = Series()
sp_market_returns = {'sp500': sp500_market_returns,
                     'sp400': sp400_market_returns,
                     'sp600': sp600_market_returns}
```

Create a dictionary to map each id to a trading strategy weighting


```python
sp500_weights = DataFrame()
sp400_weights = DataFrame()
sp600_weights = DataFrame()
sp_weights = {'sp500': sp500_weights,
              'sp400': sp400_weights,
              'sp600': sp600_weights}
```

Create a dictionary to map each id to a trading strategy pnl per stock


```python
sp500_pnl = DataFrame()
sp400_pnl = DataFrame()
sp600_pnl = DataFrame()
sp_pnl = {'sp500': sp500_pnl,
          'sp400': sp400_pnl,
          'sp600': sp600_pnl}
```

Create a dictionary to map each id to a trading strategy pnl


```python
sp500_pnl_net = Series()
sp400_pnl_net = Series()
sp600_pnl_net = Series()
sp_pnl_net = {'sp500': sp500_pnl_net,
              'sp400': sp400_pnl_net,
              'sp600': sp600_pnl_net}
```

Implement the mean-reverting trading strategy on stock universes: S&P 500, S&P MidCap 400, and S&P SmallCap indices


```python
for i in id:
    # Calculate the returns
    sp_returns[i] = sp_data[i].pct_change()

    # Calculate the equally weighted market returns
    sp_market_returns[i] = sp_returns[i].mean(axis='columns')

    # Calculate the weights of each stock
    sp_weights[i] = - (sp_returns[i].sub(sp_market_returns[i], axis='index')).div(sp_data[i].count(axis='columns'), axis='index')

    # Adjust the weights to 0 if price or return is NaN
    sp_weights[i][sp_data[i].isna() | sp_data[i].shift(periods=1).isna()] = 0

    # Calculate the daily pnl
    # Idea is to buy yesterday's losers, and sell yesterday's winners
    sp_pnl[i] = (sp_weights[i].shift(periods=1)).mul(sp_returns[i], axis='index')
    sp_pnl_net[i] = sp_pnl[i].sum(axis='columns')
```

### Calculate The Performance Metrics Of A Trading Strategy
In this section, I will provide a brief review of the Sharpe ratio which is a popular measure of a trading strategy's performance. The Sharpe ratio is a special case of the more generic Information ratio (IR). Consequently, I will present the IR first, and followed by the illustration on the Sharpe ratio.

#### Information Ratio
The Information ratio measures the excess returns of a trading strategy over a benchmark, such as the S&P 500 index. The IR scales the excess returns with the standard deviation to measure the consistency of the trading strategy. We calculate the Information ratio using the formula

$$ IR = \frac{r_{Strategy} - r_{Benchmark}}{\sigma_{Strategy}}. $$

In practice, we usually calculate the annualized IR as follows:
1. Calculate the average and standard deviation of daily returns
2. Annualize the two metrics
3. Compute the annualized return of the benchmark

#### Sharpe Ratio
The Sharpe ratio is a special case of the Information ratio where the benchmark is set to the risk-free rate. It allows for decomposition of a trading strategy's profit and losses into risky and risk-free parts. The Sharpe ratio is popular because it facilitates comparison of different trading strategies using different benchmarks.

#### How To Calculate The Sharpe Ratio
The Sharpe ratio calculation depends on the trading strategy deployed.

##### Long Only Strategies
For long-only trading strategies, we calculate the Sharpe ratio as

$$ Sharpe = \frac{r_{Long} - r_{Risk-free}}{\sigma_{Long}},  $$

where the $r_{Risk-free}$ is usually obtained from the treasury yield curve.

##### Long And Short Strategies
For long and short strategies holding equal amount of capital on both positions, also known as "dollar-neutral", we have a simpler formula for calculating the Sharpe ratio

$$ Sharpe = \frac{r_{Long-Short}}{\sigma_{Long-Short}}. $$

The $r_{Risk-free}$ disappears in the equation because the cash received from the short positions earns the same risk-free rate.

#### Performance Metrics Of A Mean-Reverting Strategy

Divide the periods of observation


```python
period = ['2014-01-01', '2015-01-01', '2016-01-01', '2017-01-01', '2018-01-01', '2019-01-01']
```

Create a dictionary to map each id to a trading performance


```python
sp500_performance = pd.DataFrame()
sp400_performance = pd.DataFrame()
sp600_performance = pd.DataFrame()
sp_performance = {'sp500': sp500_performance,
                  'sp400': sp400_performance,
                  'sp600': sp600_performance}
```

Calculate the trading strategy's annualized Sharpe ratio, average daily returns and standard deviation


```python
for i in id:
    # Initialize performance measures
    avg_returns = []
    std_returns = []
    sharpe = []

    # Calculate performance measures
    for j in range(len(period) - 1):
        # Period of observation
        start = period[j]
        end = period[j + 1]

        # Calculate average daily returns
        avg_returns.append(sp_pnl_net[i][start:end].mean())

        # Calculate standard deviation of daily returns
        std_returns.append(sp_pnl_net[i][start:end].std())

        # Calculate Sharpe ratio
        sharpe.append(np.sqrt(252) * avg_returns[j] / std_returns[j])

    # Update performance measures DataFrame
    sp_performance[i] = pd.DataFrame({'Avg Daily Returns': avg_returns,
                                      'Std Daily Returns': std_returns,
                                      'Sharpe Ratio': sharpe},
                                    index=['2014', '2015', '2016', '2017', '2018'])
```

#### Comparison Of The Mean-Reverting Strategy Using Different Stock Universes

##### Average Of Daily Returns


```python
sp_avg_returns = pd.DataFrame({'S&P 500': (sp_performance['sp500']['Avg Daily Returns'] * 100).round(4).astype('str') + '%',
                               'S&P 400': (sp_performance['sp400']['Avg Daily Returns'] * 100).round(4).astype('str') + '%',
                               'S&P 600': (sp_performance['sp600']['Avg Daily Returns'] * 100).round(4).astype('str') + '%'})
```


```python
sp_avg_returns
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>S&amp;P 500</th>
      <th>S&amp;P 400</th>
      <th>S&amp;P 600</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2014</th>
      <td>0.0007%</td>
      <td>0.001%</td>
      <td>0.0012%</td>
    </tr>
    <tr>
      <th>2015</th>
      <td>-0.0%</td>
      <td>-0.0007%</td>
      <td>0.0004%</td>
    </tr>
    <tr>
      <th>2016</th>
      <td>-0.0001%</td>
      <td>-0.0004%</td>
      <td>0.001%</td>
    </tr>
    <tr>
      <th>2017</th>
      <td>0.0008%</td>
      <td>0.0009%</td>
      <td>0.0016%</td>
    </tr>
    <tr>
      <th>2018</th>
      <td>0.0004%</td>
      <td>0.0004%</td>
      <td>0.0004%</td>
    </tr>
  </tbody>
</table>
</div>



##### Standard Deviation Of Daily Returns


```python
sp_std_returns = pd.DataFrame({'S&P 500': (sp_performance['sp500']['Std Daily Returns'] * 100).round(4).astype('str') + '%',
                               'S&P 400': (sp_performance['sp400']['Std Daily Returns'] * 100).round(4).astype('str') + '%',
                               'S&P 600': (sp_performance['sp600']['Std Daily Returns'] * 100).round(4).astype('str') + '%',})
```


```python
sp_std_returns
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>S&amp;P 500</th>
      <th>S&amp;P 400</th>
      <th>S&amp;P 600</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2014</th>
      <td>0.0049%</td>
      <td>0.0095%</td>
      <td>0.0123%</td>
    </tr>
    <tr>
      <th>2015</th>
      <td>0.0029%</td>
      <td>0.0075%</td>
      <td>0.0071%</td>
    </tr>
    <tr>
      <th>2016</th>
      <td>0.0068%</td>
      <td>0.0111%</td>
      <td>0.012%</td>
    </tr>
    <tr>
      <th>2017</th>
      <td>0.0084%</td>
      <td>0.0112%</td>
      <td>0.0106%</td>
    </tr>
    <tr>
      <th>2018</th>
      <td>0.0037%</td>
      <td>0.004%</td>
      <td>0.0049%</td>
    </tr>
  </tbody>
</table>
</div>



##### Sharpe Ratio


```python
sp_sharpe = pd.DataFrame({'S&P 500': sp_performance['sp500']['Sharpe Ratio'],
                          'S&P 400': sp_performance['sp400']['Sharpe Ratio'],
                          'S&P 600': sp_performance['sp600']['Sharpe Ratio']})
```


```python
sp_sharpe
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>S&amp;P 500</th>
      <th>S&amp;P 400</th>
      <th>S&amp;P 600</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2014</th>
      <td>2.363955</td>
      <td>1.612314</td>
      <td>1.492651</td>
    </tr>
    <tr>
      <th>2015</th>
      <td>-0.136814</td>
      <td>-1.462997</td>
      <td>0.896777</td>
    </tr>
    <tr>
      <th>2016</th>
      <td>-0.319982</td>
      <td>-0.571094</td>
      <td>1.291112</td>
    </tr>
    <tr>
      <th>2017</th>
      <td>1.535338</td>
      <td>1.259514</td>
      <td>2.358388</td>
    </tr>
    <tr>
      <th>2018</th>
      <td>1.813055</td>
      <td>1.471451</td>
      <td>1.326466</td>
    </tr>
  </tbody>
</table>
</div>



The mean-reverting strategy's Sharpe ratio is highest on the S&P 600 index composed of small-cap stocks. The inverse relationship between profitability and market-capitalization suggests that inefficiencies are more abundant on small-cap stocks. However, be mindful that small-capitalization stocks are less liquid instruments and have higher transactions costs attached when trading them.

##### Profit And Losses

Plot the cumulative pnl of the mean-reverting strategy by investment universe


```python
sp_pnl_net['sp500'].cumsum().plot()
sp_pnl_net['sp400'].cumsum().plot()
sp_pnl_net['sp600'].cumsum().plot()
plt.legend(id)
```




    <matplotlib.legend.Legend at 0x1173fa6a0>




![png](output_55_1.png)


### Improvements To The Backtesting Process
The Backtesting process measures the performance of our trading strategy using historical data. We hope that the performance of a trading strategy in the past will reproduce into the future. Unfortunately, such guarantees do not exist.

Our focus should center on rendering the backtesting process as close to reality as possible by including transaction costs, and correcting for the following:
1. Data-snooping bias is the application of an overfitted model in the trading strategy.
2. Look-ahead bias is the use of future data, or data not yet available, in the investment strategy.
3. Survivorship bias is the absence of stocks in the investment universe belonging to companies who went bankrupt, merged or acquired.

## Conclusion
In this article, we implemented a mean-reverting trading strategy and backtested it on our universe of stocks - the S&P 500, S&P MidCap 400 and S&P SmallCap 600 indices. The mean-reverting trading strategy performed best on the S&P 600 index which is composed of small-capitalization stocks. In the next articles, I will illustrate improvements to the backtesting process by including transaction costs, and correcting for potential biasses.
