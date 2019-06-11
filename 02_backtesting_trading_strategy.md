
# Backtesting A Trading Strategy Part 2

## How To Retrieve S&P Constituents Historical Data Using Python

## Introduction
Backtesting is a tool to measure the performance of a trading strategy using historical data. The backtesting process consists of three parts: 1. determining the universe of securities where we will invest in (e.g. equity or fixed income? US or emerging markets?); 2. gathering historical data for the universe of securities; and 3. implementing a trading strategy using the historical data collected.  

In the previous article, I illustrated the first step in the backtesting process of determining the universe of stocks, namely the S&P 500, S&P MidCap 400 and S&P SmallCap 600 indices. In this article, I will discuss the second step of the backtesting process of collecting historical data for each constituent of the universe of stocks. 

## Retrieving S&P Constituents Historical Data
### Step By Step
1. Load the S&P tickers which were gathered from the previous article. 
2. Collect the S&P constituents' 5-year historical data using Python package pandas-datareader from the Investors Exchange (IEX). 

You can find the code below on https://github.com/DinodC/backtesting-trading-strategy.

Import packages


```python
import pandas as pd
from pandas import Series, DataFrame
import pickle
import pandas_datareader.data as web
```

### S&P Constituents Tickers
In this section, we load the lists pickled from the last article.

Set an id for each index


```python
id = ['sp500', 'sp400', 'sp600']
```

Create a dictionary to map each id to a tickers file


```python
input_file = {'sp500': 'sp500_barchart.pickle',
              'sp400': 'sp400_barchart.pickle', 
              'sp600': 'sp600_barchart.pickle'} 
```

Define a dictionary to map each id to a tickers list


```python
sp500_tickers = []
sp400_tickers = []
sp600_tickers = []
sp_tickers = {'sp500': sp500_tickers,
              'sp400': sp400_tickers,
              'sp600': sp600_tickers}
```

Fill the tickers lists


```python
for i in input_file:
    with open(input_file[i], 'rb') as f:
        
        # Update tickers list        
        sp_tickers[i] = pickle.load(f)

        # Sort tickers list
        sp_tickers[i].sort()
        
    f.close()
```

### S&P Constituents Historical Data 

Define dictionary of historical data


```python
sp500_data = pd.DataFrame()
sp400_data = pd.DataFrame()
sp600_data = pd.DataFrame()
sp_data = {'sp500': sp500_data,
           'sp400': sp400_data,
           'sp600': sp600_data}
```

Set the start and date of the historical data


```python
start_date = '2015-01-01'
end_date = '2020-01-01'
```

Set the source Investors Exchange(IEX) to be used


```python
source = 'iex'
```

Create a dictionary to map each id to an output file


```python
output_file = {'sp500': 'sp500_data.pickle',
               'sp400': 'sp400_data.pickle',
               'sp600': 'sp600_data.pickle'}
```

Retrieve historical data for each constituent of each S&P index


```python
for i in output_file:
    
    # Retrieve historical data 
    # Note that we set number of tickers to < 100 because DataReader gives error when number of tickers > 100
    data1 = web.DataReader(sp_tickers[i][:98], source, start_date, end_date)
    data2 = web.DataReader(sp_tickers[i][98:198], source, start_date, end_date)
    data3 = web.DataReader(sp_tickers[i][198:298], source, start_date, end_date)
    data4 = web.DataReader(sp_tickers[i][298:398], source, start_date, end_date)
    data5 = web.DataReader(sp_tickers[i][398:498], source, start_date, end_date)
    if i == 'sp400':
        # Concatenate historical data
        sp_data[i] = pd.concat([data1, data2, data3, data4, data5], axis=1, sort=True)
    if i == 'sp500':
        data6 = web.DataReader(sp_tickers[i][498:], source, start_date, end_date)
        # Concatenate historical data        
        sp_data[i] = pd.concat([data1, data2, data3, data4, data5, data6], axis=1, sort=True)
    elif i == 'sp600':
        data6 = web.DataReader(sp_tickers[i][498:598], source, start_date, end_date)
        data7 = web.DataReader(sp_tickers[i][598:], source, start_date, end_date)
        # Concatenate historical data
        sp_data[i] = pd.concat([data1, data2, data3, data4, data5, data6, data7], axis=1, sort=True)        
    else:
        pass
            
    # Convert index to datetime
    sp_data[i].index = pd.to_datetime(sp_data[i].index)
    
    # Save historical data to file
    with open(output_file[i], 'wb') as f:
        pickle.dump(sp_data[i], f)
    f.close()
```

## Constituents Close Prices

### S&P 500 Index

Look at the dimensions of our DataFrame


```python
sp_data['sp500'].close.shape
```




    (1116, 505)



Check the first rows


```python
sp_data['sp500'].close.head()
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
      <th>Symbols</th>
      <th>A</th>
      <th>AAL</th>
      <th>AAP</th>
      <th>AAPL</th>
      <th>ABBV</th>
      <th>ABC</th>
      <th>ABMD</th>
      <th>ABT</th>
      <th>ACN</th>
      <th>ADBE</th>
      <th>...</th>
      <th>XEL</th>
      <th>XLNX</th>
      <th>XOM</th>
      <th>XRAY</th>
      <th>XRX</th>
      <th>XYL</th>
      <th>YUM</th>
      <th>ZBH</th>
      <th>ZION</th>
      <th>ZTS</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2015-01-02</th>
      <td>38.9022</td>
      <td>51.6157</td>
      <td>157.4188</td>
      <td>101.1385</td>
      <td>55.5960</td>
      <td>84.0786</td>
      <td>37.31</td>
      <td>40.8010</td>
      <td>81.3950</td>
      <td>72.340</td>
      <td>...</td>
      <td>31.3286</td>
      <td>39.3276</td>
      <td>77.6966</td>
      <td>50.5866</td>
      <td>31.5092</td>
      <td>35.8182</td>
      <td>47.6149</td>
      <td>108.6834</td>
      <td>26.6444</td>
      <td>41.9300</td>
    </tr>
    <tr>
      <th>2015-01-05</th>
      <td>38.1733</td>
      <td>51.5822</td>
      <td>155.3438</td>
      <td>98.2893</td>
      <td>54.5497</td>
      <td>83.3629</td>
      <td>37.07</td>
      <td>40.8101</td>
      <td>80.0207</td>
      <td>71.980</td>
      <td>...</td>
      <td>30.9730</td>
      <td>38.6015</td>
      <td>75.5707</td>
      <td>50.2359</td>
      <td>30.8218</td>
      <td>33.5890</td>
      <td>46.6475</td>
      <td>112.7377</td>
      <td>25.6460</td>
      <td>41.6783</td>
    </tr>
    <tr>
      <th>2015-01-06</th>
      <td>37.5786</td>
      <td>50.7828</td>
      <td>155.2346</td>
      <td>98.2985</td>
      <td>54.2797</td>
      <td>83.8183</td>
      <td>36.13</td>
      <td>40.3466</td>
      <td>79.4435</td>
      <td>70.530</td>
      <td>...</td>
      <td>31.1378</td>
      <td>38.0468</td>
      <td>75.1689</td>
      <td>49.6125</td>
      <td>30.4093</td>
      <td>33.3915</td>
      <td>46.0749</td>
      <td>111.7820</td>
      <td>24.6665</td>
      <td>41.2717</td>
    </tr>
    <tr>
      <th>2015-01-07</th>
      <td>38.0773</td>
      <td>50.7540</td>
      <td>158.5704</td>
      <td>99.6769</td>
      <td>56.4735</td>
      <td>85.4914</td>
      <td>37.28</td>
      <td>40.6738</td>
      <td>81.1110</td>
      <td>71.110</td>
      <td>...</td>
      <td>31.4066</td>
      <td>38.0603</td>
      <td>75.9306</td>
      <td>50.8984</td>
      <td>30.7988</td>
      <td>33.6548</td>
      <td>47.6017</td>
      <td>114.5621</td>
      <td>24.8973</td>
      <td>42.1236</td>
    </tr>
    <tr>
      <th>2015-01-08</th>
      <td>39.2187</td>
      <td>51.3764</td>
      <td>159.9603</td>
      <td>103.5067</td>
      <td>57.0642</td>
      <td>85.6865</td>
      <td>38.96</td>
      <td>41.5098</td>
      <td>82.3478</td>
      <td>72.915</td>
      <td>...</td>
      <td>31.7709</td>
      <td>38.9082</td>
      <td>77.1944</td>
      <td>52.1550</td>
      <td>31.6467</td>
      <td>33.9088</td>
      <td>48.4310</td>
      <td>115.7784</td>
      <td>25.2504</td>
      <td>42.7723</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 505 columns</p>
</div>




```python
sp_data['sp500'].close[['A', 'AAL', 'AAP', 'AAPL', 'ABBV']].head()
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
      <th>Symbols</th>
      <th>A</th>
      <th>AAL</th>
      <th>AAP</th>
      <th>AAPL</th>
      <th>ABBV</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2015-01-02</th>
      <td>38.9022</td>
      <td>51.6157</td>
      <td>157.4188</td>
      <td>101.1385</td>
      <td>55.5960</td>
    </tr>
    <tr>
      <th>2015-01-05</th>
      <td>38.1733</td>
      <td>51.5822</td>
      <td>155.3438</td>
      <td>98.2893</td>
      <td>54.5497</td>
    </tr>
    <tr>
      <th>2015-01-06</th>
      <td>37.5786</td>
      <td>50.7828</td>
      <td>155.2346</td>
      <td>98.2985</td>
      <td>54.2797</td>
    </tr>
    <tr>
      <th>2015-01-07</th>
      <td>38.0773</td>
      <td>50.7540</td>
      <td>158.5704</td>
      <td>99.6769</td>
      <td>56.4735</td>
    </tr>
    <tr>
      <th>2015-01-08</th>
      <td>39.2187</td>
      <td>51.3764</td>
      <td>159.9603</td>
      <td>103.5067</td>
      <td>57.0642</td>
    </tr>
  </tbody>
</table>
</div>



Check the end rows


```python
sp_data['sp500'].close.tail()
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
      <th>Symbols</th>
      <th>A</th>
      <th>AAL</th>
      <th>AAP</th>
      <th>AAPL</th>
      <th>ABBV</th>
      <th>ABC</th>
      <th>ABMD</th>
      <th>ABT</th>
      <th>ACN</th>
      <th>ADBE</th>
      <th>...</th>
      <th>XEL</th>
      <th>XLNX</th>
      <th>XOM</th>
      <th>XRAY</th>
      <th>XRX</th>
      <th>XYL</th>
      <th>YUM</th>
      <th>ZBH</th>
      <th>ZION</th>
      <th>ZTS</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2019-06-04</th>
      <td>67.95</td>
      <td>29.12</td>
      <td>154.61</td>
      <td>179.64</td>
      <td>76.75</td>
      <td>82.21</td>
      <td>267.06</td>
      <td>77.46</td>
      <td>177.97</td>
      <td>268.71</td>
      <td>...</td>
      <td>57.78</td>
      <td>106.90</td>
      <td>73.59</td>
      <td>54.40</td>
      <td>33.30</td>
      <td>77.40</td>
      <td>106.97</td>
      <td>117.41</td>
      <td>44.40</td>
      <td>108.12</td>
    </tr>
    <tr>
      <th>2019-06-05</th>
      <td>68.35</td>
      <td>30.36</td>
      <td>154.61</td>
      <td>182.54</td>
      <td>77.06</td>
      <td>81.65</td>
      <td>268.80</td>
      <td>78.69</td>
      <td>179.56</td>
      <td>272.86</td>
      <td>...</td>
      <td>59.32</td>
      <td>105.60</td>
      <td>72.98</td>
      <td>55.38</td>
      <td>33.42</td>
      <td>78.88</td>
      <td>107.29</td>
      <td>118.54</td>
      <td>44.18</td>
      <td>108.50</td>
    </tr>
    <tr>
      <th>2019-06-06</th>
      <td>69.16</td>
      <td>30.38</td>
      <td>154.90</td>
      <td>185.22</td>
      <td>77.07</td>
      <td>81.75</td>
      <td>269.19</td>
      <td>80.09</td>
      <td>180.40</td>
      <td>274.80</td>
      <td>...</td>
      <td>59.80</td>
      <td>106.01</td>
      <td>74.31</td>
      <td>55.63</td>
      <td>34.03</td>
      <td>79.15</td>
      <td>108.42</td>
      <td>120.31</td>
      <td>44.24</td>
      <td>108.89</td>
    </tr>
    <tr>
      <th>2019-06-07</th>
      <td>69.52</td>
      <td>30.92</td>
      <td>155.35</td>
      <td>190.15</td>
      <td>77.43</td>
      <td>83.48</td>
      <td>267.87</td>
      <td>80.74</td>
      <td>182.92</td>
      <td>278.16</td>
      <td>...</td>
      <td>59.43</td>
      <td>107.49</td>
      <td>74.58</td>
      <td>55.94</td>
      <td>34.16</td>
      <td>79.56</td>
      <td>109.07</td>
      <td>120.73</td>
      <td>43.64</td>
      <td>110.06</td>
    </tr>
    <tr>
      <th>2019-06-10</th>
      <td>70.29</td>
      <td>30.76</td>
      <td>153.52</td>
      <td>192.58</td>
      <td>76.95</td>
      <td>84.77</td>
      <td>272.43</td>
      <td>81.27</td>
      <td>184.44</td>
      <td>280.34</td>
      <td>...</td>
      <td>59.26</td>
      <td>110.88</td>
      <td>74.91</td>
      <td>57.10</td>
      <td>34.69</td>
      <td>80.38</td>
      <td>108.65</td>
      <td>121.71</td>
      <td>43.84</td>
      <td>110.22</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 505 columns</p>
</div>




```python
sp_data['sp500'].close[['A', 'AAL', 'AAP', 'AAPL', 'ABBV']].tail()
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
      <th>Symbols</th>
      <th>A</th>
      <th>AAL</th>
      <th>AAP</th>
      <th>AAPL</th>
      <th>ABBV</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2019-06-04</th>
      <td>67.95</td>
      <td>29.12</td>
      <td>154.61</td>
      <td>179.64</td>
      <td>76.75</td>
    </tr>
    <tr>
      <th>2019-06-05</th>
      <td>68.35</td>
      <td>30.36</td>
      <td>154.61</td>
      <td>182.54</td>
      <td>77.06</td>
    </tr>
    <tr>
      <th>2019-06-06</th>
      <td>69.16</td>
      <td>30.38</td>
      <td>154.90</td>
      <td>185.22</td>
      <td>77.07</td>
    </tr>
    <tr>
      <th>2019-06-07</th>
      <td>69.52</td>
      <td>30.92</td>
      <td>155.35</td>
      <td>190.15</td>
      <td>77.43</td>
    </tr>
    <tr>
      <th>2019-06-10</th>
      <td>70.29</td>
      <td>30.76</td>
      <td>153.52</td>
      <td>192.58</td>
      <td>76.95</td>
    </tr>
  </tbody>
</table>
</div>



Descriptive stats


```python
sp_data['sp500'].close.describe()
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
      <th>Symbols</th>
      <th>A</th>
      <th>AAL</th>
      <th>AAP</th>
      <th>AAPL</th>
      <th>ABBV</th>
      <th>ABC</th>
      <th>ABMD</th>
      <th>ABT</th>
      <th>ACN</th>
      <th>ADBE</th>
      <th>...</th>
      <th>XEL</th>
      <th>XLNX</th>
      <th>XOM</th>
      <th>XRAY</th>
      <th>XRX</th>
      <th>XYL</th>
      <th>YUM</th>
      <th>ZBH</th>
      <th>ZION</th>
      <th>ZTS</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>...</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>53.682820</td>
      <td>41.423051</td>
      <td>146.493998</td>
      <td>140.290845</td>
      <td>68.852242</td>
      <td>85.875797</td>
      <td>179.890578</td>
      <td>50.444850</td>
      <td>124.099296</td>
      <td>151.371232</td>
      <td>...</td>
      <td>40.836168</td>
      <td>61.341104</td>
      <td>74.788485</td>
      <td>53.898061</td>
      <td>26.462871</td>
      <td>53.403838</td>
      <td>68.914116</td>
      <td>113.058047</td>
      <td>37.937309</td>
      <td>62.868532</td>
    </tr>
    <tr>
      <th>std</th>
      <td>13.849508</td>
      <td>6.496939</td>
      <td>25.131046</td>
      <td>37.315585</td>
      <td>17.563078</td>
      <td>8.968281</td>
      <td>112.738751</td>
      <td>12.825554</td>
      <td>28.542652</td>
      <td>69.639202</td>
      <td>...</td>
      <td>7.633397</td>
      <td>22.403880</td>
      <td>4.468653</td>
      <td>7.954859</td>
      <td>3.160050</td>
      <td>16.177875</td>
      <td>15.501008</td>
      <td>9.558257</td>
      <td>10.788513</td>
      <td>19.372341</td>
    </tr>
    <tr>
      <th>min</th>
      <td>32.258600</td>
      <td>24.539800</td>
      <td>79.168700</td>
      <td>85.976800</td>
      <td>42.066600</td>
      <td>65.718100</td>
      <td>36.130000</td>
      <td>33.935700</td>
      <td>76.988100</td>
      <td>69.990000</td>
      <td>...</td>
      <td>28.223200</td>
      <td>34.795000</td>
      <td>58.967500</td>
      <td>34.178400</td>
      <td>18.532600</td>
      <td>28.874500</td>
      <td>44.181800</td>
      <td>89.236100</td>
      <td>18.885300</td>
      <td>38.438700</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>40.231325</td>
      <td>36.699875</td>
      <td>137.239175</td>
      <td>107.503100</td>
      <td>54.802525</td>
      <td>79.292050</td>
      <td>94.167500</td>
      <td>40.571050</td>
      <td>100.310000</td>
      <td>91.392500</td>
      <td>...</td>
      <td>35.321450</td>
      <td>43.897275</td>
      <td>72.061175</td>
      <td>49.581750</td>
      <td>23.883800</td>
      <td>35.900400</td>
      <td>56.202700</td>
      <td>106.314000</td>
      <td>26.980875</td>
      <td>46.284250</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>51.786950</td>
      <td>41.052250</td>
      <td>152.678750</td>
      <td>135.893350</td>
      <td>60.182250</td>
      <td>84.772950</td>
      <td>125.575000</td>
      <td>44.888600</td>
      <td>116.497350</td>
      <td>126.590000</td>
      <td>...</td>
      <td>41.031700</td>
      <td>56.992350</td>
      <td>75.256750</td>
      <td>55.800150</td>
      <td>26.286500</td>
      <td>50.008100</td>
      <td>63.047300</td>
      <td>113.959300</td>
      <td>40.436250</td>
      <td>53.209200</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>66.235725</td>
      <td>46.425700</td>
      <td>163.304600</td>
      <td>170.222875</td>
      <td>85.766250</td>
      <td>90.887375</td>
      <td>276.560000</td>
      <td>60.211775</td>
      <td>152.661150</td>
      <td>224.590000</td>
      <td>...</td>
      <td>46.205200</td>
      <td>70.171850</td>
      <td>77.872600</td>
      <td>60.087700</td>
      <td>28.750200</td>
      <td>68.681950</td>
      <td>81.002350</td>
      <td>120.219250</td>
      <td>47.769625</td>
      <td>83.072500</td>
    </tr>
    <tr>
      <th>max</th>
      <td>81.940000</td>
      <td>57.586600</td>
      <td>199.159900</td>
      <td>229.392000</td>
      <td>116.445400</td>
      <td>107.649700</td>
      <td>449.750000</td>
      <td>81.270000</td>
      <td>184.440000</td>
      <td>289.250000</td>
      <td>...</td>
      <td>59.800000</td>
      <td>139.263300</td>
      <td>83.828700</td>
      <td>67.795300</td>
      <td>35.000000</td>
      <td>83.549000</td>
      <td>109.070000</td>
      <td>130.912800</td>
      <td>57.139500</td>
      <td>110.220000</td>
    </tr>
  </tbody>
</table>
<p>8 rows × 505 columns</p>
</div>




```python
sp_data['sp500'].close[['A', 'AAL', 'AAP', 'AAPL', 'ABBV']].describe()
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
      <th>Symbols</th>
      <th>A</th>
      <th>AAL</th>
      <th>AAP</th>
      <th>AAPL</th>
      <th>ABBV</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>53.682820</td>
      <td>41.423051</td>
      <td>146.493998</td>
      <td>140.290845</td>
      <td>68.852242</td>
    </tr>
    <tr>
      <th>std</th>
      <td>13.849508</td>
      <td>6.496939</td>
      <td>25.131046</td>
      <td>37.315585</td>
      <td>17.563078</td>
    </tr>
    <tr>
      <th>min</th>
      <td>32.258600</td>
      <td>24.539800</td>
      <td>79.168700</td>
      <td>85.976800</td>
      <td>42.066600</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>40.231325</td>
      <td>36.699875</td>
      <td>137.239175</td>
      <td>107.503100</td>
      <td>54.802525</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>51.786950</td>
      <td>41.052250</td>
      <td>152.678750</td>
      <td>135.893350</td>
      <td>60.182250</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>66.235725</td>
      <td>46.425700</td>
      <td>163.304600</td>
      <td>170.222875</td>
      <td>85.766250</td>
    </tr>
    <tr>
      <th>max</th>
      <td>81.940000</td>
      <td>57.586600</td>
      <td>199.159900</td>
      <td>229.392000</td>
      <td>116.445400</td>
    </tr>
  </tbody>
</table>
</div>



### S&P MidCap 400 Index

Look at the dimensions of our DataFrame


```python
sp_data['sp400'].close.shape
```




    (1128, 400)



Check the first rows


```python
sp_data['sp400'].close.head()
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
      <th>Symbols</th>
      <th>AAN</th>
      <th>ACC</th>
      <th>ACHC</th>
      <th>ACIW</th>
      <th>ACM</th>
      <th>ADNT</th>
      <th>AEO</th>
      <th>AFG</th>
      <th>AGCO</th>
      <th>ALE</th>
      <th>...</th>
      <th>WTR</th>
      <th>WW</th>
      <th>WWD</th>
      <th>WWE</th>
      <th>WYND</th>
      <th>X</th>
      <th>XPO</th>
      <th>Y</th>
      <th>YELP</th>
      <th>ZBRA</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2015-01-02</th>
      <td>30.2075</td>
      <td>35.2031</td>
      <td>59.98</td>
      <td>19.71</td>
      <td>30.35</td>
      <td>NaN</td>
      <td>12.0755</td>
      <td>50.6126</td>
      <td>43.1325</td>
      <td>47.6957</td>
      <td>...</td>
      <td>24.0506</td>
      <td>21.53</td>
      <td>46.9623</td>
      <td>11.0529</td>
      <td>34.2053</td>
      <td>25.4410</td>
      <td>40.63</td>
      <td>454.3187</td>
      <td>55.15</td>
      <td>77.43</td>
    </tr>
    <tr>
      <th>2015-01-05</th>
      <td>30.0891</td>
      <td>35.4122</td>
      <td>59.12</td>
      <td>19.26</td>
      <td>29.02</td>
      <td>NaN</td>
      <td>12.2162</td>
      <td>49.9490</td>
      <td>41.1493</td>
      <td>46.9383</td>
      <td>...</td>
      <td>23.5846</td>
      <td>21.26</td>
      <td>45.9472</td>
      <td>10.8508</td>
      <td>33.5884</td>
      <td>24.2546</td>
      <td>39.37</td>
      <td>444.5068</td>
      <td>52.53</td>
      <td>76.34</td>
    </tr>
    <tr>
      <th>2015-01-06</th>
      <td>28.9644</td>
      <td>35.7803</td>
      <td>58.19</td>
      <td>18.95</td>
      <td>28.73</td>
      <td>NaN</td>
      <td>12.2601</td>
      <td>49.6634</td>
      <td>40.9577</td>
      <td>46.8953</td>
      <td>...</td>
      <td>23.6115</td>
      <td>19.74</td>
      <td>45.6186</td>
      <td>10.8784</td>
      <td>33.1878</td>
      <td>23.5178</td>
      <td>37.98</td>
      <td>441.8915</td>
      <td>52.44</td>
      <td>75.79</td>
    </tr>
    <tr>
      <th>2015-01-07</th>
      <td>29.8424</td>
      <td>35.8388</td>
      <td>60.63</td>
      <td>19.03</td>
      <td>29.32</td>
      <td>NaN</td>
      <td>12.8577</td>
      <td>49.9910</td>
      <td>40.9290</td>
      <td>47.7817</td>
      <td>...</td>
      <td>23.8266</td>
      <td>20.35</td>
      <td>46.0439</td>
      <td>10.4282</td>
      <td>33.8087</td>
      <td>23.5752</td>
      <td>38.14</td>
      <td>443.5531</td>
      <td>52.21</td>
      <td>77.72</td>
    </tr>
    <tr>
      <th>2015-01-08</th>
      <td>30.3258</td>
      <td>36.0479</td>
      <td>61.76</td>
      <td>19.01</td>
      <td>30.23</td>
      <td>NaN</td>
      <td>12.2513</td>
      <td>50.8982</td>
      <td>41.8008</td>
      <td>48.7198</td>
      <td>...</td>
      <td>23.9789</td>
      <td>20.32</td>
      <td>45.8119</td>
      <td>10.3087</td>
      <td>34.6899</td>
      <td>24.0919</td>
      <td>38.35</td>
      <td>447.3186</td>
      <td>53.83</td>
      <td>79.38</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 400 columns</p>
</div>




```python
sp_data['sp400'].close[['AAN', 'ACC', 'ACHC', 'ACIW', 'ACM']].head()
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
      <th>Symbols</th>
      <th>AAN</th>
      <th>ACC</th>
      <th>ACHC</th>
      <th>ACIW</th>
      <th>ACM</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2015-01-02</th>
      <td>30.2075</td>
      <td>35.2031</td>
      <td>59.98</td>
      <td>19.71</td>
      <td>30.35</td>
    </tr>
    <tr>
      <th>2015-01-05</th>
      <td>30.0891</td>
      <td>35.4122</td>
      <td>59.12</td>
      <td>19.26</td>
      <td>29.02</td>
    </tr>
    <tr>
      <th>2015-01-06</th>
      <td>28.9644</td>
      <td>35.7803</td>
      <td>58.19</td>
      <td>18.95</td>
      <td>28.73</td>
    </tr>
    <tr>
      <th>2015-01-07</th>
      <td>29.8424</td>
      <td>35.8388</td>
      <td>60.63</td>
      <td>19.03</td>
      <td>29.32</td>
    </tr>
    <tr>
      <th>2015-01-08</th>
      <td>30.3258</td>
      <td>36.0479</td>
      <td>61.76</td>
      <td>19.01</td>
      <td>30.23</td>
    </tr>
  </tbody>
</table>
</div>



Check the end rows


```python
sp_data['sp400'].close.tail()
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
      <th>Symbols</th>
      <th>AAN</th>
      <th>ACC</th>
      <th>ACHC</th>
      <th>ACIW</th>
      <th>ACM</th>
      <th>ADNT</th>
      <th>AEO</th>
      <th>AFG</th>
      <th>AGCO</th>
      <th>ALE</th>
      <th>...</th>
      <th>WTR</th>
      <th>WW</th>
      <th>WWD</th>
      <th>WWE</th>
      <th>WYND</th>
      <th>X</th>
      <th>XPO</th>
      <th>Y</th>
      <th>YELP</th>
      <th>ZBRA</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2019-06-04</th>
      <td>54.88</td>
      <td>46.17</td>
      <td>33.72</td>
      <td>32.10</td>
      <td>33.17</td>
      <td>18.92</td>
      <td>18.52</td>
      <td>101.46</td>
      <td>69.42</td>
      <td>82.76</td>
      <td>...</td>
      <td>39.82</td>
      <td>18.90</td>
      <td>111.80</td>
      <td>73.10</td>
      <td>41.11</td>
      <td>13.29</td>
      <td>55.28</td>
      <td>683.58</td>
      <td>31.79</td>
      <td>177.08</td>
    </tr>
    <tr>
      <th>2019-06-05</th>
      <td>54.97</td>
      <td>46.97</td>
      <td>33.30</td>
      <td>32.26</td>
      <td>33.29</td>
      <td>19.39</td>
      <td>18.54</td>
      <td>101.80</td>
      <td>69.96</td>
      <td>84.89</td>
      <td>...</td>
      <td>40.63</td>
      <td>19.04</td>
      <td>113.86</td>
      <td>73.68</td>
      <td>41.08</td>
      <td>13.09</td>
      <td>53.93</td>
      <td>693.19</td>
      <td>31.69</td>
      <td>181.59</td>
    </tr>
    <tr>
      <th>2019-06-06</th>
      <td>54.72</td>
      <td>47.02</td>
      <td>33.20</td>
      <td>32.40</td>
      <td>33.18</td>
      <td>19.75</td>
      <td>17.58</td>
      <td>101.53</td>
      <td>69.16</td>
      <td>85.26</td>
      <td>...</td>
      <td>40.88</td>
      <td>18.46</td>
      <td>114.32</td>
      <td>73.48</td>
      <td>41.01</td>
      <td>13.21</td>
      <td>53.32</td>
      <td>692.20</td>
      <td>31.57</td>
      <td>181.76</td>
    </tr>
    <tr>
      <th>2019-06-07</th>
      <td>55.72</td>
      <td>47.17</td>
      <td>33.91</td>
      <td>32.34</td>
      <td>33.47</td>
      <td>20.39</td>
      <td>17.50</td>
      <td>100.88</td>
      <td>69.75</td>
      <td>84.99</td>
      <td>...</td>
      <td>40.84</td>
      <td>18.95</td>
      <td>113.60</td>
      <td>73.28</td>
      <td>41.78</td>
      <td>13.58</td>
      <td>53.78</td>
      <td>690.74</td>
      <td>31.48</td>
      <td>188.37</td>
    </tr>
    <tr>
      <th>2019-06-10</th>
      <td>59.23</td>
      <td>46.81</td>
      <td>33.81</td>
      <td>32.75</td>
      <td>33.45</td>
      <td>20.79</td>
      <td>16.76</td>
      <td>100.83</td>
      <td>70.11</td>
      <td>84.70</td>
      <td>...</td>
      <td>40.63</td>
      <td>19.40</td>
      <td>113.63</td>
      <td>71.51</td>
      <td>42.55</td>
      <td>13.80</td>
      <td>55.55</td>
      <td>684.74</td>
      <td>31.95</td>
      <td>194.51</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 400 columns</p>
</div>




```python
sp_data['sp400'].close[['AAN', 'ACC', 'ACHC', 'ACIW', 'ACM']].tail()
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
      <th>Symbols</th>
      <th>AAN</th>
      <th>ACC</th>
      <th>ACHC</th>
      <th>ACIW</th>
      <th>ACM</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2019-06-04</th>
      <td>54.88</td>
      <td>46.17</td>
      <td>33.72</td>
      <td>32.10</td>
      <td>33.17</td>
    </tr>
    <tr>
      <th>2019-06-05</th>
      <td>54.97</td>
      <td>46.97</td>
      <td>33.30</td>
      <td>32.26</td>
      <td>33.29</td>
    </tr>
    <tr>
      <th>2019-06-06</th>
      <td>54.72</td>
      <td>47.02</td>
      <td>33.20</td>
      <td>32.40</td>
      <td>33.18</td>
    </tr>
    <tr>
      <th>2019-06-07</th>
      <td>55.72</td>
      <td>47.17</td>
      <td>33.91</td>
      <td>32.34</td>
      <td>33.47</td>
    </tr>
    <tr>
      <th>2019-06-10</th>
      <td>59.23</td>
      <td>46.81</td>
      <td>33.81</td>
      <td>32.75</td>
      <td>33.45</td>
    </tr>
  </tbody>
</table>
</div>



Descriptive stats


```python
sp_data['sp400'].close.describe()
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
      <th>Symbols</th>
      <th>AAN</th>
      <th>ACC</th>
      <th>ACHC</th>
      <th>ACIW</th>
      <th>ACM</th>
      <th>ADNT</th>
      <th>AEO</th>
      <th>AFG</th>
      <th>AGCO</th>
      <th>ALE</th>
      <th>...</th>
      <th>WTR</th>
      <th>WW</th>
      <th>WWD</th>
      <th>WWE</th>
      <th>WYND</th>
      <th>X</th>
      <th>XPO</th>
      <th>Y</th>
      <th>YELP</th>
      <th>ZBRA</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>655.00000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>...</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>36.350373</td>
      <td>39.939185</td>
      <td>48.294091</td>
      <td>23.224933</td>
      <td>32.301595</td>
      <td>51.41140</td>
      <td>16.391984</td>
      <td>80.359913</td>
      <td>56.983298</td>
      <td>61.714817</td>
      <td>...</td>
      <td>30.566799</td>
      <td>30.454978</td>
      <td>65.714888</td>
      <td>34.319395</td>
      <td>37.721795</td>
      <td>23.601209</td>
      <td>56.632679</td>
      <td>553.035098</td>
      <td>36.543020</td>
      <td>110.150184</td>
    </tr>
    <tr>
      <th>std</th>
      <td>9.912823</td>
      <td>4.558736</td>
      <td>14.509896</td>
      <td>3.915154</td>
      <td>3.229840</td>
      <td>20.70094</td>
      <td>3.824099</td>
      <td>19.347078</td>
      <td>9.236321</td>
      <td>12.842531</td>
      <td>...</td>
      <td>4.380471</td>
      <td>24.809183</td>
      <td>16.007057</td>
      <td>26.167644</td>
      <td>7.118456</td>
      <td>8.720881</td>
      <td>26.018152</td>
      <td>63.682948</td>
      <td>8.607855</td>
      <td>43.313545</td>
    </tr>
    <tr>
      <th>min</th>
      <td>20.118600</td>
      <td>27.793900</td>
      <td>24.750000</td>
      <td>16.230000</td>
      <td>23.150000</td>
      <td>12.57000</td>
      <td>10.118800</td>
      <td>48.962000</td>
      <td>40.929000</td>
      <td>40.072600</td>
      <td>...</td>
      <td>22.220300</td>
      <td>3.780000</td>
      <td>38.906800</td>
      <td>9.233700</td>
      <td>25.207900</td>
      <td>6.455100</td>
      <td>19.560000</td>
      <td>433.190600</td>
      <td>15.230000</td>
      <td>46.930000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>27.644000</td>
      <td>36.077475</td>
      <td>37.447500</td>
      <td>20.147500</td>
      <td>30.187500</td>
      <td>31.95000</td>
      <td>13.619175</td>
      <td>61.302600</td>
      <td>48.388325</td>
      <td>48.854200</td>
      <td>...</td>
      <td>27.656025</td>
      <td>11.477500</td>
      <td>51.121250</td>
      <td>16.842825</td>
      <td>31.480175</td>
      <td>17.791000</td>
      <td>35.992500</td>
      <td>488.626100</td>
      <td>30.380000</td>
      <td>77.810000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>36.299650</td>
      <td>40.569000</td>
      <td>44.080000</td>
      <td>22.765000</td>
      <td>32.415000</td>
      <td>57.04690</td>
      <td>15.141550</td>
      <td>84.654250</td>
      <td>57.851800</td>
      <td>63.026700</td>
      <td>...</td>
      <td>31.104450</td>
      <td>19.085000</td>
      <td>67.425700</td>
      <td>20.155750</td>
      <td>36.271400</td>
      <td>22.852250</td>
      <td>49.290000</td>
      <td>566.443300</td>
      <td>37.235000</td>
      <td>100.645000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>44.065400</td>
      <td>43.776175</td>
      <td>59.352500</td>
      <td>24.852500</td>
      <td>34.440000</td>
      <td>66.54120</td>
      <td>19.135775</td>
      <td>97.804025</td>
      <td>65.280100</td>
      <td>73.766650</td>
      <td>...</td>
      <td>33.845575</td>
      <td>45.990000</td>
      <td>76.265750</td>
      <td>39.665300</td>
      <td>43.550625</td>
      <td>30.006425</td>
      <td>69.072500</td>
      <td>605.189700</td>
      <td>43.542500</td>
      <td>142.165000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>59.230000</td>
      <td>48.353300</td>
      <td>82.970000</td>
      <td>35.520000</td>
      <td>40.130000</td>
      <td>84.04680</td>
      <td>28.421700</td>
      <td>113.252900</td>
      <td>74.760300</td>
      <td>85.260000</td>
      <td>...</td>
      <td>40.880000</td>
      <td>103.090000</td>
      <td>114.320000</td>
      <td>99.250000</td>
      <td>54.884400</td>
      <td>45.545800</td>
      <td>114.540000</td>
      <td>693.190000</td>
      <td>57.470000</td>
      <td>235.440000</td>
    </tr>
  </tbody>
</table>
<p>8 rows × 400 columns</p>
</div>




```python
sp_data['sp400'].close[['AAN', 'ACC', 'ACHC', 'ACIW', 'ACM']].describe()
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
      <th>Symbols</th>
      <th>AAN</th>
      <th>ACC</th>
      <th>ACHC</th>
      <th>ACIW</th>
      <th>ACM</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>36.350373</td>
      <td>39.939185</td>
      <td>48.294091</td>
      <td>23.224933</td>
      <td>32.301595</td>
    </tr>
    <tr>
      <th>std</th>
      <td>9.912823</td>
      <td>4.558736</td>
      <td>14.509896</td>
      <td>3.915154</td>
      <td>3.229840</td>
    </tr>
    <tr>
      <th>min</th>
      <td>20.118600</td>
      <td>27.793900</td>
      <td>24.750000</td>
      <td>16.230000</td>
      <td>23.150000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>27.644000</td>
      <td>36.077475</td>
      <td>37.447500</td>
      <td>20.147500</td>
      <td>30.187500</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>36.299650</td>
      <td>40.569000</td>
      <td>44.080000</td>
      <td>22.765000</td>
      <td>32.415000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>44.065400</td>
      <td>43.776175</td>
      <td>59.352500</td>
      <td>24.852500</td>
      <td>34.440000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>59.230000</td>
      <td>48.353300</td>
      <td>82.970000</td>
      <td>35.520000</td>
      <td>40.130000</td>
    </tr>
  </tbody>
</table>
</div>



## S&P SmallCap 600 Index

Look at the dimensions of our DataFrame


```python
sp_data['sp600'].close.shape
```




    (1116, 601)



Check the first rows


```python
sp_data['sp600'].close.head()
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
      <th>Symbols</th>
      <th>AAOI</th>
      <th>AAON</th>
      <th>AAT</th>
      <th>AAWW</th>
      <th>AAXN</th>
      <th>ABCB</th>
      <th>ABG</th>
      <th>ABM</th>
      <th>ACA</th>
      <th>ACLS</th>
      <th>...</th>
      <th>WPG</th>
      <th>WRE</th>
      <th>WRLD</th>
      <th>WSR</th>
      <th>WTS</th>
      <th>WWW</th>
      <th>XHR</th>
      <th>XPER</th>
      <th>ZEUS</th>
      <th>ZUMZ</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2015-01-02</th>
      <td>10.79</td>
      <td>21.1378</td>
      <td>36.1497</td>
      <td>48.42</td>
      <td>26.51</td>
      <td>24.3685</td>
      <td>76.09</td>
      <td>26.1460</td>
      <td>NaN</td>
      <td>10.20</td>
      <td>...</td>
      <td>10.0733</td>
      <td>23.4646</td>
      <td>77.72</td>
      <td>10.3510</td>
      <td>59.7693</td>
      <td>27.5666</td>
      <td>NaN</td>
      <td>30.8228</td>
      <td>17.9012</td>
      <td>38.21</td>
    </tr>
    <tr>
      <th>2015-01-05</th>
      <td>10.65</td>
      <td>20.4197</td>
      <td>36.5616</td>
      <td>46.65</td>
      <td>25.69</td>
      <td>24.3492</td>
      <td>74.00</td>
      <td>26.1920</td>
      <td>NaN</td>
      <td>10.00</td>
      <td>...</td>
      <td>10.1716</td>
      <td>23.6822</td>
      <td>78.14</td>
      <td>10.4395</td>
      <td>58.2492</td>
      <td>27.3007</td>
      <td>NaN</td>
      <td>29.9799</td>
      <td>16.2561</td>
      <td>38.94</td>
    </tr>
    <tr>
      <th>2015-01-06</th>
      <td>10.25</td>
      <td>20.0968</td>
      <td>36.8481</td>
      <td>45.66</td>
      <td>25.57</td>
      <td>23.8478</td>
      <td>72.69</td>
      <td>26.3208</td>
      <td>NaN</td>
      <td>9.60</td>
      <td>...</td>
      <td>10.1195</td>
      <td>23.4144</td>
      <td>77.59</td>
      <td>10.4327</td>
      <td>57.1471</td>
      <td>26.6455</td>
      <td>NaN</td>
      <td>28.7633</td>
      <td>15.5804</td>
      <td>38.46</td>
    </tr>
    <tr>
      <th>2015-01-07</th>
      <td>9.85</td>
      <td>20.2799</td>
      <td>37.5466</td>
      <td>46.48</td>
      <td>25.98</td>
      <td>23.6163</td>
      <td>75.01</td>
      <td>26.7532</td>
      <td>NaN</td>
      <td>9.72</td>
      <td>...</td>
      <td>10.4145</td>
      <td>23.6738</td>
      <td>77.45</td>
      <td>10.4667</td>
      <td>56.7195</td>
      <td>26.9208</td>
      <td>NaN</td>
      <td>28.6764</td>
      <td>14.5325</td>
      <td>40.28</td>
    </tr>
    <tr>
      <th>2015-01-08</th>
      <td>9.96</td>
      <td>20.7812</td>
      <td>37.7704</td>
      <td>48.21</td>
      <td>26.63</td>
      <td>23.8188</td>
      <td>75.50</td>
      <td>27.0844</td>
      <td>NaN</td>
      <td>9.92</td>
      <td>...</td>
      <td>10.4376</td>
      <td>23.6655</td>
      <td>78.74</td>
      <td>10.3987</td>
      <td>57.3086</td>
      <td>27.6140</td>
      <td>NaN</td>
      <td>29.5194</td>
      <td>15.6881</td>
      <td>41.40</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 601 columns</p>
</div>




```python
sp_data['sp600'].close[['AAOI', 'AAON', 'AAT', 'AAWW', 'AAXN']].head()
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
      <th>Symbols</th>
      <th>AAOI</th>
      <th>AAON</th>
      <th>AAT</th>
      <th>AAWW</th>
      <th>AAXN</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2015-01-02</th>
      <td>10.79</td>
      <td>21.1378</td>
      <td>36.1497</td>
      <td>48.42</td>
      <td>26.51</td>
    </tr>
    <tr>
      <th>2015-01-05</th>
      <td>10.65</td>
      <td>20.4197</td>
      <td>36.5616</td>
      <td>46.65</td>
      <td>25.69</td>
    </tr>
    <tr>
      <th>2015-01-06</th>
      <td>10.25</td>
      <td>20.0968</td>
      <td>36.8481</td>
      <td>45.66</td>
      <td>25.57</td>
    </tr>
    <tr>
      <th>2015-01-07</th>
      <td>9.85</td>
      <td>20.2799</td>
      <td>37.5466</td>
      <td>46.48</td>
      <td>25.98</td>
    </tr>
    <tr>
      <th>2015-01-08</th>
      <td>9.96</td>
      <td>20.7812</td>
      <td>37.7704</td>
      <td>48.21</td>
      <td>26.63</td>
    </tr>
  </tbody>
</table>
</div>



Check the end rows


```python
sp_data['sp600'].close.tail()
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
      <th>Symbols</th>
      <th>AAOI</th>
      <th>AAON</th>
      <th>AAT</th>
      <th>AAWW</th>
      <th>AAXN</th>
      <th>ABCB</th>
      <th>ABG</th>
      <th>ABM</th>
      <th>ACA</th>
      <th>ACLS</th>
      <th>...</th>
      <th>WPG</th>
      <th>WRE</th>
      <th>WRLD</th>
      <th>WSR</th>
      <th>WTS</th>
      <th>WWW</th>
      <th>XHR</th>
      <th>XPER</th>
      <th>ZEUS</th>
      <th>ZUMZ</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2019-06-04</th>
      <td>9.19</td>
      <td>47.80</td>
      <td>45.11</td>
      <td>39.70</td>
      <td>67.63</td>
      <td>37.25</td>
      <td>79.67</td>
      <td>37.60</td>
      <td>35.52</td>
      <td>15.30</td>
      <td>...</td>
      <td>4.08</td>
      <td>27.03</td>
      <td>137.79</td>
      <td>12.54</td>
      <td>85.75</td>
      <td>28.42</td>
      <td>20.95</td>
      <td>21.32</td>
      <td>13.16</td>
      <td>20.65</td>
    </tr>
    <tr>
      <th>2019-06-05</th>
      <td>9.23</td>
      <td>48.08</td>
      <td>46.39</td>
      <td>38.62</td>
      <td>68.00</td>
      <td>37.03</td>
      <td>77.80</td>
      <td>37.28</td>
      <td>36.20</td>
      <td>14.95</td>
      <td>...</td>
      <td>4.05</td>
      <td>27.46</td>
      <td>137.16</td>
      <td>12.70</td>
      <td>85.79</td>
      <td>28.25</td>
      <td>21.00</td>
      <td>20.70</td>
      <td>12.86</td>
      <td>19.94</td>
    </tr>
    <tr>
      <th>2019-06-06</th>
      <td>9.30</td>
      <td>48.18</td>
      <td>46.39</td>
      <td>37.99</td>
      <td>67.97</td>
      <td>36.90</td>
      <td>76.95</td>
      <td>39.78</td>
      <td>37.06</td>
      <td>14.89</td>
      <td>...</td>
      <td>3.99</td>
      <td>27.52</td>
      <td>136.06</td>
      <td>12.67</td>
      <td>86.42</td>
      <td>28.32</td>
      <td>20.85</td>
      <td>20.69</td>
      <td>13.01</td>
      <td>18.67</td>
    </tr>
    <tr>
      <th>2019-06-07</th>
      <td>9.50</td>
      <td>48.85</td>
      <td>46.61</td>
      <td>39.88</td>
      <td>69.18</td>
      <td>36.58</td>
      <td>77.95</td>
      <td>39.36</td>
      <td>38.34</td>
      <td>15.03</td>
      <td>...</td>
      <td>4.05</td>
      <td>27.47</td>
      <td>136.72</td>
      <td>12.68</td>
      <td>87.35</td>
      <td>28.50</td>
      <td>21.01</td>
      <td>20.93</td>
      <td>12.98</td>
      <td>21.65</td>
    </tr>
    <tr>
      <th>2019-06-10</th>
      <td>9.60</td>
      <td>48.64</td>
      <td>46.51</td>
      <td>40.30</td>
      <td>71.91</td>
      <td>37.20</td>
      <td>78.88</td>
      <td>39.47</td>
      <td>38.69</td>
      <td>15.74</td>
      <td>...</td>
      <td>4.21</td>
      <td>27.29</td>
      <td>142.45</td>
      <td>12.50</td>
      <td>87.97</td>
      <td>28.39</td>
      <td>21.12</td>
      <td>20.57</td>
      <td>13.19</td>
      <td>21.46</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 601 columns</p>
</div>




```python
sp_data['sp600'].close[['AAOI', 'AAON', 'AAT', 'AAWW', 'AAXN']].tail()
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
      <th>Symbols</th>
      <th>AAOI</th>
      <th>AAON</th>
      <th>AAT</th>
      <th>AAWW</th>
      <th>AAXN</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2019-06-04</th>
      <td>9.19</td>
      <td>47.80</td>
      <td>45.11</td>
      <td>39.70</td>
      <td>67.63</td>
    </tr>
    <tr>
      <th>2019-06-05</th>
      <td>9.23</td>
      <td>48.08</td>
      <td>46.39</td>
      <td>38.62</td>
      <td>68.00</td>
    </tr>
    <tr>
      <th>2019-06-06</th>
      <td>9.30</td>
      <td>48.18</td>
      <td>46.39</td>
      <td>37.99</td>
      <td>67.97</td>
    </tr>
    <tr>
      <th>2019-06-07</th>
      <td>9.50</td>
      <td>48.85</td>
      <td>46.61</td>
      <td>39.88</td>
      <td>69.18</td>
    </tr>
    <tr>
      <th>2019-06-10</th>
      <td>9.60</td>
      <td>48.64</td>
      <td>46.51</td>
      <td>40.30</td>
      <td>71.91</td>
    </tr>
  </tbody>
</table>
</div>



Descriptive stats


```python
sp_data['sp600'].close.describe()
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
      <th>Symbols</th>
      <th>AAOI</th>
      <th>AAON</th>
      <th>AAT</th>
      <th>AAWW</th>
      <th>AAXN</th>
      <th>ABCB</th>
      <th>ABG</th>
      <th>ABM</th>
      <th>ACA</th>
      <th>ACLS</th>
      <th>...</th>
      <th>WPG</th>
      <th>WRE</th>
      <th>WRLD</th>
      <th>WSR</th>
      <th>WTS</th>
      <th>WWW</th>
      <th>XHR</th>
      <th>XPER</th>
      <th>ZEUS</th>
      <th>ZUMZ</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>152.000000</td>
      <td>1116.000000</td>
      <td>...</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1094.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>27.743244</td>
      <td>30.918709</td>
      <td>38.058995</td>
      <td>50.943889</td>
      <td>33.501694</td>
      <td>38.263235</td>
      <td>67.166470</td>
      <td>33.834044</td>
      <td>30.553658</td>
      <td>17.025923</td>
      <td>...</td>
      <td>6.638630</td>
      <td>26.699762</td>
      <td>75.499521</td>
      <td>10.873052</td>
      <td>64.134756</td>
      <td>26.901846</td>
      <td>17.466643</td>
      <td>26.470261</td>
      <td>18.461373</td>
      <td>21.713634</td>
    </tr>
    <tr>
      <th>std</th>
      <td>17.739201</td>
      <td>7.217437</td>
      <td>3.162768</td>
      <td>9.630046</td>
      <td>15.846801</td>
      <td>9.777167</td>
      <td>10.615101</td>
      <td>4.977216</td>
      <td>3.460363</td>
      <td>6.232940</td>
      <td>...</td>
      <td>1.439664</td>
      <td>2.965398</td>
      <td>29.980650</td>
      <td>1.427787</td>
      <td>11.103416</td>
      <td>6.054634</td>
      <td>3.218554</td>
      <td>6.868666</td>
      <td>4.585663</td>
      <td>6.338225</td>
    </tr>
    <tr>
      <th>min</th>
      <td>8.380000</td>
      <td>18.515200</td>
      <td>30.623200</td>
      <td>33.370000</td>
      <td>14.500000</td>
      <td>21.938400</td>
      <td>45.070000</td>
      <td>24.986400</td>
      <td>20.930200</td>
      <td>8.960000</td>
      <td>...</td>
      <td>3.990000</td>
      <td>20.496900</td>
      <td>26.700000</td>
      <td>7.337200</td>
      <td>43.206400</td>
      <td>14.741000</td>
      <td>10.508700</td>
      <td>11.940500</td>
      <td>8.155900</td>
      <td>11.450000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>14.930000</td>
      <td>23.764775</td>
      <td>36.253125</td>
      <td>42.410000</td>
      <td>23.020000</td>
      <td>29.015975</td>
      <td>58.387500</td>
      <td>29.762450</td>
      <td>28.943125</td>
      <td>11.270000</td>
      <td>...</td>
      <td>5.551425</td>
      <td>23.872600</td>
      <td>48.457500</td>
      <td>9.865475</td>
      <td>53.693475</td>
      <td>22.452275</td>
      <td>14.932475</td>
      <td>21.132900</td>
      <td>15.543625</td>
      <td>17.127500</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>20.410000</td>
      <td>32.401300</td>
      <td>37.858000</td>
      <td>50.895000</td>
      <td>26.100000</td>
      <td>37.445250</td>
      <td>67.020000</td>
      <td>33.295050</td>
      <td>30.315400</td>
      <td>15.950000</td>
      <td>...</td>
      <td>6.430150</td>
      <td>27.307200</td>
      <td>77.015000</td>
      <td>10.956150</td>
      <td>62.285450</td>
      <td>27.334650</td>
      <td>17.782900</td>
      <td>27.297250</td>
      <td>18.884700</td>
      <td>20.697500</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>37.910000</td>
      <td>35.719525</td>
      <td>39.674850</td>
      <td>57.950000</td>
      <td>42.792500</td>
      <td>46.383700</td>
      <td>74.500000</td>
      <td>38.071300</td>
      <td>32.236275</td>
      <td>21.362500</td>
      <td>...</td>
      <td>7.361450</td>
      <td>29.385200</td>
      <td>104.135000</td>
      <td>11.843275</td>
      <td>74.294575</td>
      <td>31.442500</td>
      <td>19.650875</td>
      <td>30.892075</td>
      <td>21.879050</td>
      <td>24.750000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>99.610000</td>
      <td>51.631400</td>
      <td>46.770000</td>
      <td>74.000000</td>
      <td>74.890000</td>
      <td>57.403000</td>
      <td>95.540000</td>
      <td>43.220900</td>
      <td>38.690000</td>
      <td>36.625000</td>
      <td>...</td>
      <td>10.596600</td>
      <td>31.602900</td>
      <td>142.450000</td>
      <td>14.108200</td>
      <td>87.970000</td>
      <td>39.469500</td>
      <td>24.438200</td>
      <td>41.762300</td>
      <td>30.738900</td>
      <td>41.400000</td>
    </tr>
  </tbody>
</table>
<p>8 rows × 601 columns</p>
</div>




```python
sp_data['sp600'].close[['AAOI', 'AAON', 'AAT', 'AAWW', 'AAXN']].describe()
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
      <th>Symbols</th>
      <th>AAOI</th>
      <th>AAON</th>
      <th>AAT</th>
      <th>AAWW</th>
      <th>AAXN</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
      <td>1116.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>27.743244</td>
      <td>30.918709</td>
      <td>38.058995</td>
      <td>50.943889</td>
      <td>33.501694</td>
    </tr>
    <tr>
      <th>std</th>
      <td>17.739201</td>
      <td>7.217437</td>
      <td>3.162768</td>
      <td>9.630046</td>
      <td>15.846801</td>
    </tr>
    <tr>
      <th>min</th>
      <td>8.380000</td>
      <td>18.515200</td>
      <td>30.623200</td>
      <td>33.370000</td>
      <td>14.500000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>14.930000</td>
      <td>23.764775</td>
      <td>36.253125</td>
      <td>42.410000</td>
      <td>23.020000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>20.410000</td>
      <td>32.401300</td>
      <td>37.858000</td>
      <td>50.895000</td>
      <td>26.100000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>37.910000</td>
      <td>35.719525</td>
      <td>39.674850</td>
      <td>57.950000</td>
      <td>42.792500</td>
    </tr>
    <tr>
      <th>max</th>
      <td>99.610000</td>
      <td>51.631400</td>
      <td>46.770000</td>
      <td>74.000000</td>
      <td>74.890000</td>
    </tr>
  </tbody>
</table>
</div>



## Summary
In this article, we retrieved historical data for every constituent in our universe of stocks - the S&P 500, S&P MidCap 400 and S&P SmallCap 600 indices. The 5-year historical data is relatively straightforward to obtain, and is provided for free by the Investors Exchange. In the next article, we implement a simple trading strategy, and backtest it using the historical data collected.
