
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
start_date = '2014-01-01'
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




    (1258, 505)



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
      <th>2014-06-11</th>
      <td>40.1199</td>
      <td>40.2868</td>
      <td>125.0694</td>
      <td>86.0249</td>
      <td>45.0769</td>
      <td>66.4223</td>
      <td>22.92</td>
      <td>35.9633</td>
      <td>74.9719</td>
      <td>67.46</td>
      <td>...</td>
      <td>25.6691</td>
      <td>40.9150</td>
      <td>84.1321</td>
      <td>46.6073</td>
      <td>28.8084</td>
      <td>35.4555</td>
      <td>51.7899</td>
      <td>101.5068</td>
      <td>28.0164</td>
      <td>30.9798</td>
    </tr>
    <tr>
      <th>2014-06-12</th>
      <td>39.7726</td>
      <td>38.2958</td>
      <td>123.2252</td>
      <td>84.5860</td>
      <td>44.6031</td>
      <td>65.9975</td>
      <td>23.03</td>
      <td>35.7925</td>
      <td>74.5018</td>
      <td>66.56</td>
      <td>...</td>
      <td>25.8547</td>
      <td>41.1019</td>
      <td>83.8928</td>
      <td>46.4230</td>
      <td>28.5372</td>
      <td>35.2780</td>
      <td>51.2950</td>
      <td>101.0367</td>
      <td>27.7535</td>
      <td>30.8448</td>
    </tr>
    <tr>
      <th>2014-06-13</th>
      <td>39.8407</td>
      <td>38.4672</td>
      <td>123.7407</td>
      <td>83.6603</td>
      <td>45.0187</td>
      <td>66.2930</td>
      <td>22.95</td>
      <td>35.7745</td>
      <td>74.7820</td>
      <td>66.82</td>
      <td>...</td>
      <td>25.8884</td>
      <td>41.6091</td>
      <td>84.7098</td>
      <td>46.3744</td>
      <td>28.4920</td>
      <td>36.0532</td>
      <td>51.5945</td>
      <td>101.2861</td>
      <td>27.8192</td>
      <td>30.9702</td>
    </tr>
    <tr>
      <th>2014-06-16</th>
      <td>39.7181</td>
      <td>39.1150</td>
      <td>124.0086</td>
      <td>84.5035</td>
      <td>44.8857</td>
      <td>66.0529</td>
      <td>23.27</td>
      <td>35.8734</td>
      <td>74.8634</td>
      <td>67.62</td>
      <td>...</td>
      <td>26.0740</td>
      <td>41.9028</td>
      <td>84.9326</td>
      <td>46.4424</td>
      <td>28.3791</td>
      <td>35.9318</td>
      <td>51.5099</td>
      <td>100.7105</td>
      <td>27.4060</td>
      <td>30.9798</td>
    </tr>
    <tr>
      <th>2014-06-17</th>
      <td>40.0927</td>
      <td>39.8867</td>
      <td>124.9411</td>
      <td>84.3935</td>
      <td>45.1351</td>
      <td>66.2561</td>
      <td>23.43</td>
      <td>35.8375</td>
      <td>74.5832</td>
      <td>67.54</td>
      <td>...</td>
      <td>26.0910</td>
      <td>42.2409</td>
      <td>84.5200</td>
      <td>46.2677</td>
      <td>28.8310</td>
      <td>36.0346</td>
      <td>51.7639</td>
      <td>100.4707</td>
      <td>27.9601</td>
      <td>31.6355</td>
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
      <th>2014-06-11</th>
      <td>40.1199</td>
      <td>40.2868</td>
      <td>125.0694</td>
      <td>86.0249</td>
      <td>45.0769</td>
    </tr>
    <tr>
      <th>2014-06-12</th>
      <td>39.7726</td>
      <td>38.2958</td>
      <td>123.2252</td>
      <td>84.5860</td>
      <td>44.6031</td>
    </tr>
    <tr>
      <th>2014-06-13</th>
      <td>39.8407</td>
      <td>38.4672</td>
      <td>123.7407</td>
      <td>83.6603</td>
      <td>45.0187</td>
    </tr>
    <tr>
      <th>2014-06-16</th>
      <td>39.7181</td>
      <td>39.1150</td>
      <td>124.0086</td>
      <td>84.5035</td>
      <td>44.8857</td>
    </tr>
    <tr>
      <th>2014-06-17</th>
      <td>40.0927</td>
      <td>39.8867</td>
      <td>124.9411</td>
      <td>84.3935</td>
      <td>45.1351</td>
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
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>...</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>52.010924</td>
      <td>41.207727</td>
      <td>145.418612</td>
      <td>135.094399</td>
      <td>66.653293</td>
      <td>84.553883</td>
      <td>162.741109</td>
      <td>49.113396</td>
      <td>118.473403</td>
      <td>142.243243</td>
      <td>...</td>
      <td>39.351443</td>
      <td>58.850843</td>
      <td>75.442714</td>
      <td>53.174206</td>
      <td>26.877544</td>
      <td>51.296575</td>
      <td>66.602743</td>
      <td>111.697803</td>
      <td>36.704361</td>
      <td>59.798250</td>
    </tr>
    <tr>
      <th>std</th>
      <td>13.866577</td>
      <td>6.366236</td>
      <td>24.128339</td>
      <td>38.127616</td>
      <td>17.721243</td>
      <td>9.483389</td>
      <td>116.577717</td>
      <td>12.653128</td>
      <td>31.189929</td>
      <td>70.413470</td>
      <td>...</td>
      <td>8.322974</td>
      <td>22.240068</td>
      <td>4.717698</td>
      <td>7.833835</td>
      <td>3.222176</td>
      <td>16.349757</td>
      <td>15.997628</td>
      <td>9.929764</td>
      <td>10.736900</td>
      <td>20.223068</td>
    </tr>
    <tr>
      <th>min</th>
      <td>32.258600</td>
      <td>24.539800</td>
      <td>79.168700</td>
      <td>82.743800</td>
      <td>42.066600</td>
      <td>65.718100</td>
      <td>22.220000</td>
      <td>33.935700</td>
      <td>68.852300</td>
      <td>60.880000</td>
      <td>...</td>
      <td>25.247700</td>
      <td>32.502600</td>
      <td>58.967500</td>
      <td>34.178400</td>
      <td>18.532600</td>
      <td>28.874500</td>
      <td>44.181800</td>
      <td>89.236100</td>
      <td>18.885300</td>
      <td>30.652000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>39.393500</td>
      <td>36.586125</td>
      <td>132.519725</td>
      <td>103.622650</td>
      <td>53.170800</td>
      <td>77.738125</td>
      <td>80.662500</td>
      <td>39.405975</td>
      <td>92.616475</td>
      <td>82.092500</td>
      <td>...</td>
      <td>31.485800</td>
      <td>42.102950</td>
      <td>72.534125</td>
      <td>48.731800</td>
      <td>24.215550</td>
      <td>35.033175</td>
      <td>53.060925</td>
      <td>103.343675</td>
      <td>26.826025</td>
      <td>44.749325</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>46.097600</td>
      <td>40.843300</td>
      <td>150.457700</td>
      <td>119.473800</td>
      <td>58.470800</td>
      <td>83.840300</td>
      <td>118.550000</td>
      <td>43.040600</td>
      <td>112.097050</td>
      <td>107.945000</td>
      <td>...</td>
      <td>39.055650</td>
      <td>52.368850</td>
      <td>75.817000</td>
      <td>54.521200</td>
      <td>26.573400</td>
      <td>48.085300</td>
      <td>61.564200</td>
      <td>112.761500</td>
      <td>38.198900</td>
      <td>51.333600</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>65.591850</td>
      <td>46.119900</td>
      <td>161.899875</td>
      <td>167.841200</td>
      <td>82.894300</td>
      <td>89.847025</td>
      <td>261.935000</td>
      <td>58.171250</td>
      <td>149.591750</td>
      <td>212.247500</td>
      <td>...</td>
      <td>45.444750</td>
      <td>68.789375</td>
      <td>78.491000</td>
      <td>59.650550</td>
      <td>29.650500</td>
      <td>67.341800</td>
      <td>80.202225</td>
      <td>119.131625</td>
      <td>46.431025</td>
      <td>80.809150</td>
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
      <td>86.137400</td>
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
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>52.010924</td>
      <td>41.207727</td>
      <td>145.418612</td>
      <td>135.094399</td>
      <td>66.653293</td>
    </tr>
    <tr>
      <th>std</th>
      <td>13.866577</td>
      <td>6.366236</td>
      <td>24.128339</td>
      <td>38.127616</td>
      <td>17.721243</td>
    </tr>
    <tr>
      <th>min</th>
      <td>32.258600</td>
      <td>24.539800</td>
      <td>79.168700</td>
      <td>82.743800</td>
      <td>42.066600</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>39.393500</td>
      <td>36.586125</td>
      <td>132.519725</td>
      <td>103.622650</td>
      <td>53.170800</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>46.097600</td>
      <td>40.843300</td>
      <td>150.457700</td>
      <td>119.473800</td>
      <td>58.470800</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>65.591850</td>
      <td>46.119900</td>
      <td>161.899875</td>
      <td>167.841200</td>
      <td>82.894300</td>
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




    (1297, 400)



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
      <th>2014-05-07</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2014-05-08</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2014-05-09</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2014-05-12</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2014-05-13</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
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
      <th>2014-05-07</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2014-05-08</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2014-05-09</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2014-05-12</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2014-05-13</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
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
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>655.00000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>...</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>35.344696</td>
      <td>39.074002</td>
      <td>48.806781</td>
      <td>22.656060</td>
      <td>32.409610</td>
      <td>51.41140</td>
      <td>15.791684</td>
      <td>76.790031</td>
      <td>55.704826</td>
      <td>59.509322</td>
      <td>...</td>
      <td>29.638305</td>
      <td>29.788041</td>
      <td>63.744381</td>
      <td>31.766971</td>
      <td>37.016177</td>
      <td>24.529339</td>
      <td>54.093362</td>
      <td>538.956165</td>
      <td>40.046097</td>
      <td>106.230811</td>
    </tr>
    <tr>
      <th>std</th>
      <td>9.825081</td>
      <td>4.946453</td>
      <td>13.903160</td>
      <td>4.065761</td>
      <td>3.178291</td>
      <td>20.70094</td>
      <td>3.997170</td>
      <td>20.795256</td>
      <td>9.501151</td>
      <td>13.608161</td>
      <td>...</td>
      <td>4.889501</td>
      <td>23.460312</td>
      <td>16.066755</td>
      <td>25.666425</td>
      <td>7.006602</td>
      <td>8.798477</td>
      <td>25.570709</td>
      <td>71.983064</td>
      <td>13.133896</td>
      <td>42.282714</td>
    </tr>
    <tr>
      <th>min</th>
      <td>20.118600</td>
      <td>27.793900</td>
      <td>24.750000</td>
      <td>13.617500</td>
      <td>23.150000</td>
      <td>12.57000</td>
      <td>8.872200</td>
      <td>46.029500</td>
      <td>40.217700</td>
      <td>37.847600</td>
      <td>...</td>
      <td>20.625100</td>
      <td>3.780000</td>
      <td>38.906800</td>
      <td>9.233700</td>
      <td>25.207900</td>
      <td>6.455100</td>
      <td>19.560000</td>
      <td>405.789900</td>
      <td>15.230000</td>
      <td>46.930000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>25.957675</td>
      <td>34.718300</td>
      <td>38.355000</td>
      <td>19.470000</td>
      <td>30.370000</td>
      <td>31.95000</td>
      <td>12.927300</td>
      <td>58.436225</td>
      <td>46.935500</td>
      <td>45.647675</td>
      <td>...</td>
      <td>24.419950</td>
      <td>12.060000</td>
      <td>48.823450</td>
      <td>15.752725</td>
      <td>31.123075</td>
      <td>18.589200</td>
      <td>34.322500</td>
      <td>473.883600</td>
      <td>31.690000</td>
      <td>74.915000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>34.792300</td>
      <td>39.727900</td>
      <td>46.525000</td>
      <td>22.275000</td>
      <td>32.465000</td>
      <td>57.04690</td>
      <td>14.696200</td>
      <td>75.144950</td>
      <td>54.295400</td>
      <td>58.636750</td>
      <td>...</td>
      <td>29.886900</td>
      <td>20.890000</td>
      <td>66.107300</td>
      <td>19.371300</td>
      <td>34.954450</td>
      <td>23.753650</td>
      <td>46.485000</td>
      <td>545.605200</td>
      <td>38.850000</td>
      <td>91.865000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>43.259125</td>
      <td>43.422450</td>
      <td>59.717500</td>
      <td>24.240000</td>
      <td>34.537500</td>
      <td>66.54120</td>
      <td>18.327025</td>
      <td>96.480175</td>
      <td>63.707075</td>
      <td>73.222850</td>
      <td>...</td>
      <td>33.454425</td>
      <td>44.240000</td>
      <td>75.237300</td>
      <td>36.257025</td>
      <td>42.975150</td>
      <td>32.198000</td>
      <td>64.747500</td>
      <td>601.316050</td>
      <td>45.397500</td>
      <td>136.300000</td>
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
      <td>84.960000</td>
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
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>35.344696</td>
      <td>39.074002</td>
      <td>48.806781</td>
      <td>22.656060</td>
      <td>32.409610</td>
    </tr>
    <tr>
      <th>std</th>
      <td>9.825081</td>
      <td>4.946453</td>
      <td>13.903160</td>
      <td>4.065761</td>
      <td>3.178291</td>
    </tr>
    <tr>
      <th>min</th>
      <td>20.118600</td>
      <td>27.793900</td>
      <td>24.750000</td>
      <td>13.617500</td>
      <td>23.150000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>25.957675</td>
      <td>34.718300</td>
      <td>38.355000</td>
      <td>19.470000</td>
      <td>30.370000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>34.792300</td>
      <td>39.727900</td>
      <td>46.525000</td>
      <td>22.275000</td>
      <td>32.465000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>43.259125</td>
      <td>43.422450</td>
      <td>59.717500</td>
      <td>24.240000</td>
      <td>34.537500</td>
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




    (1259, 601)



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
  </thead>
  <tbody>
    <tr>
      <th>2014-06-10</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2014-06-11</th>
      <td>22.33</td>
      <td>20.7695</td>
      <td>30.1070</td>
      <td>38.07</td>
      <td>13.91</td>
      <td>20.9537</td>
      <td>64.62</td>
      <td>24.3261</td>
      <td>NaN</td>
      <td>7.68</td>
      <td>...</td>
      <td>10.8245</td>
      <td>21.2765</td>
      <td>76.44</td>
      <td>9.3183</td>
      <td>57.5835</td>
      <td>24.8378</td>
      <td>NaN</td>
      <td>19.4619</td>
      <td>23.5042</td>
      <td>28.99</td>
    </tr>
    <tr>
      <th>2014-06-12</th>
      <td>22.30</td>
      <td>20.5904</td>
      <td>29.9566</td>
      <td>37.05</td>
      <td>14.03</td>
      <td>20.7716</td>
      <td>62.44</td>
      <td>24.1543</td>
      <td>NaN</td>
      <td>7.60</td>
      <td>...</td>
      <td>10.7626</td>
      <td>21.3093</td>
      <td>76.34</td>
      <td>9.2923</td>
      <td>57.2527</td>
      <td>24.3284</td>
      <td>NaN</td>
      <td>19.2547</td>
      <td>23.0351</td>
      <td>28.27</td>
    </tr>
    <tr>
      <th>2014-06-13</th>
      <td>21.86</td>
      <td>20.3792</td>
      <td>29.9654</td>
      <td>37.29</td>
      <td>13.87</td>
      <td>20.6183</td>
      <td>62.28</td>
      <td>23.9282</td>
      <td>NaN</td>
      <td>7.64</td>
      <td>...</td>
      <td>10.7795</td>
      <td>21.1865</td>
      <td>76.81</td>
      <td>9.3053</td>
      <td>57.3094</td>
      <td>24.8095</td>
      <td>NaN</td>
      <td>19.5742</td>
      <td>23.2012</td>
      <td>28.15</td>
    </tr>
    <tr>
      <th>2014-06-16</th>
      <td>22.71</td>
      <td>20.5712</td>
      <td>30.0273</td>
      <td>36.95</td>
      <td>13.78</td>
      <td>20.3980</td>
      <td>63.14</td>
      <td>23.7745</td>
      <td>NaN</td>
      <td>7.76</td>
      <td>...</td>
      <td>10.8864</td>
      <td>20.8754</td>
      <td>78.29</td>
      <td>9.2727</td>
      <td>57.4985</td>
      <td>24.6302</td>
      <td>NaN</td>
      <td>19.5828</td>
      <td>22.8885</td>
      <td>28.21</td>
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
  </thead>
  <tbody>
    <tr>
      <th>2014-06-10</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2014-06-11</th>
      <td>22.33</td>
      <td>20.7695</td>
      <td>30.1070</td>
      <td>38.07</td>
      <td>13.91</td>
    </tr>
    <tr>
      <th>2014-06-12</th>
      <td>22.30</td>
      <td>20.5904</td>
      <td>29.9566</td>
      <td>37.05</td>
      <td>14.03</td>
    </tr>
    <tr>
      <th>2014-06-13</th>
      <td>21.86</td>
      <td>20.3792</td>
      <td>29.9654</td>
      <td>37.29</td>
      <td>13.87</td>
    </tr>
    <tr>
      <th>2014-06-16</th>
      <td>22.71</td>
      <td>20.5712</td>
      <td>30.0273</td>
      <td>36.95</td>
      <td>13.78</td>
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
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>152.000000</td>
      <td>1258.000000</td>
      <td>...</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1094.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>26.523188</td>
      <td>29.595272</td>
      <td>37.383077</td>
      <td>49.431387</td>
      <td>31.595862</td>
      <td>36.468555</td>
      <td>67.475747</td>
      <td>32.757237</td>
      <td>30.553658</td>
      <td>16.014777</td>
      <td>...</td>
      <td>7.052403</td>
      <td>26.183249</td>
      <td>75.539273</td>
      <td>10.755358</td>
      <td>63.426474</td>
      <td>26.717355</td>
      <td>17.466643</td>
      <td>26.235295</td>
      <td>18.697389</td>
      <td>22.802699</td>
    </tr>
    <tr>
      <th>std</th>
      <td>17.116913</td>
      <td>7.763096</td>
      <td>3.595458</td>
      <td>10.156758</td>
      <td>15.917380</td>
      <td>10.508064</td>
      <td>10.126593</td>
      <td>5.585728</td>
      <td>3.460363</td>
      <td>6.525089</td>
      <td>...</td>
      <td>1.793529</td>
      <td>3.155567</td>
      <td>28.283319</td>
      <td>1.387724</td>
      <td>10.670268</td>
      <td>5.752016</td>
      <td>3.218554</td>
      <td>6.633022</td>
      <td>4.463565</td>
      <td>6.799962</td>
    </tr>
    <tr>
      <th>min</th>
      <td>8.380000</td>
      <td>16.287400</td>
      <td>29.352300</td>
      <td>31.400000</td>
      <td>10.500000</td>
      <td>20.120100</td>
      <td>45.070000</td>
      <td>22.347500</td>
      <td>20.930200</td>
      <td>6.840000</td>
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
      <td>14.810000</td>
      <td>22.305875</td>
      <td>35.384150</td>
      <td>40.970000</td>
      <td>22.285000</td>
      <td>26.449975</td>
      <td>59.167500</td>
      <td>28.549550</td>
      <td>28.943125</td>
      <td>10.600000</td>
      <td>...</td>
      <td>5.664050</td>
      <td>23.158525</td>
      <td>50.137500</td>
      <td>9.794800</td>
      <td>54.413025</td>
      <td>23.000950</td>
      <td>14.932475</td>
      <td>21.145625</td>
      <td>16.119775</td>
      <td>17.500000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>19.845000</td>
      <td>30.684600</td>
      <td>37.508550</td>
      <td>49.150000</td>
      <td>25.175000</td>
      <td>35.079700</td>
      <td>67.750000</td>
      <td>31.390800</td>
      <td>30.315400</td>
      <td>13.925000</td>
      <td>...</td>
      <td>6.576000</td>
      <td>26.576000</td>
      <td>76.820000</td>
      <td>10.795250</td>
      <td>61.023300</td>
      <td>26.738850</td>
      <td>17.782900</td>
      <td>26.661550</td>
      <td>19.138750</td>
      <td>21.850000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>33.927500</td>
      <td>35.473400</td>
      <td>39.254350</td>
      <td>56.542500</td>
      <td>38.340000</td>
      <td>45.666850</td>
      <td>74.352500</td>
      <td>37.597450</td>
      <td>32.236275</td>
      <td>20.950000</td>
      <td>...</td>
      <td>8.228400</td>
      <td>29.164850</td>
      <td>101.162500</td>
      <td>11.707950</td>
      <td>72.812975</td>
      <td>30.402625</td>
      <td>19.650875</td>
      <td>30.475175</td>
      <td>22.118050</td>
      <td>26.700000</td>
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
      <td>11.148200</td>
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
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>26.523188</td>
      <td>29.595272</td>
      <td>37.383077</td>
      <td>49.431387</td>
      <td>31.595862</td>
    </tr>
    <tr>
      <th>std</th>
      <td>17.116913</td>
      <td>7.763096</td>
      <td>3.595458</td>
      <td>10.156758</td>
      <td>15.917380</td>
    </tr>
    <tr>
      <th>min</th>
      <td>8.380000</td>
      <td>16.287400</td>
      <td>29.352300</td>
      <td>31.400000</td>
      <td>10.500000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>14.810000</td>
      <td>22.305875</td>
      <td>35.384150</td>
      <td>40.970000</td>
      <td>22.285000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>19.845000</td>
      <td>30.684600</td>
      <td>37.508550</td>
      <td>49.150000</td>
      <td>25.175000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>33.927500</td>
      <td>35.473400</td>
      <td>39.254350</td>
      <td>56.542500</td>
      <td>38.340000</td>
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
