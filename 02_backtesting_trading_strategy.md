
# Backtesting a Trading Strategy Part 2

## Or How to Retrieve S&P Constituents Historical Data Using Python Package pandas-datareader

## Introduction
Backtesting is a tool we can use to measure the performance of a trading strategy using historical data. The backtesting process can be broken down in three parts: first, we need to decide the universe of securities where we invest in (e.g. equity or fixed income? US or emerging markets?); second, we need to retrieve historical data for the universe of securities; and last part is where we implement our trading strategy using the historical data collected. In the previous article, we completed the first step in the backtesting process of defining the universe of stocks, namely the S&P 500, S&P MidCap 400 and S&P SmallCap 600 indices. In this article, we proceed to the second step of the backtesting process, that is, we retrieve historical data for each constituent on every index. 

Below we load the tickers of S&P 500, S&P 400 and S&P 600 indices retrieved from the previous article. After, we use Python package [pandas-datareader](https://pandas-datareader.readthedocs.io) which allows us to collect historical market data from providers such as the [Investor Exchange (IEX)](https://iextrading.com), [Morningstar](https://www.morningstar.com), or [Robinhood](https://robinhood.com). We choose IEX as our data source since it provides 5 year historical data.

## Retrieve Historical Data

You can find the code below on https://github.com/DinodC/backtesting-mean-reverting-strategy.

Import packages


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

Set the source [Investors Exchange (IEX)](https://iextrading.com) to be used


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

Eyeball


```python
sp_data['sp500'].close.shape
```




    (1258, 505)




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
      <th>2014-05-09</th>
      <td>37.7227</td>
      <td>36.4477</td>
      <td>123.5028</td>
      <td>76.9614</td>
      <td>43.4394</td>
      <td>60.6682</td>
      <td>21.59</td>
      <td>35.0552</td>
      <td>71.1754</td>
      <td>59.59</td>
      <td>...</td>
      <td>25.9390</td>
      <td>41.3795</td>
      <td>85.0965</td>
      <td>44.7737</td>
      <td>26.7522</td>
      <td>34.7281</td>
      <td>49.9143</td>
      <td>95.9139</td>
      <td>27.4130</td>
      <td>29.4467</td>
    </tr>
    <tr>
      <th>2014-05-12</th>
      <td>38.4174</td>
      <td>37.5241</td>
      <td>124.1869</td>
      <td>77.9193</td>
      <td>43.5308</td>
      <td>61.6766</td>
      <td>21.92</td>
      <td>35.3070</td>
      <td>71.5821</td>
      <td>60.70</td>
      <td>...</td>
      <td>25.7028</td>
      <td>42.1741</td>
      <td>85.3302</td>
      <td>45.0550</td>
      <td>27.1364</td>
      <td>35.5124</td>
      <td>49.7835</td>
      <td>96.7773</td>
      <td>27.9511</td>
      <td>29.7071</td>
    </tr>
    <tr>
      <th>2014-05-13</th>
      <td>38.7034</td>
      <td>37.4479</td>
      <td>122.3625</td>
      <td>78.0415</td>
      <td>43.3895</td>
      <td>61.8431</td>
      <td>21.55</td>
      <td>35.6576</td>
      <td>72.0251</td>
      <td>60.81</td>
      <td>...</td>
      <td>25.7872</td>
      <td>41.2278</td>
      <td>85.4387</td>
      <td>45.1908</td>
      <td>27.3171</td>
      <td>35.4471</td>
      <td>50.0059</td>
      <td>97.1610</td>
      <td>27.5924</td>
      <td>29.6299</td>
    </tr>
    <tr>
      <th>2014-05-14</th>
      <td>38.0360</td>
      <td>37.0002</td>
      <td>122.0948</td>
      <td>78.0560</td>
      <td>43.9464</td>
      <td>62.2594</td>
      <td>21.55</td>
      <td>35.9004</td>
      <td>71.2748</td>
      <td>60.88</td>
      <td>...</td>
      <td>26.0234</td>
      <td>40.9064</td>
      <td>85.3803</td>
      <td>44.7834</td>
      <td>27.0686</td>
      <td>35.2603</td>
      <td>49.6070</td>
      <td>96.8253</td>
      <td>26.8278</td>
      <td>29.5431</td>
    </tr>
    <tr>
      <th>2014-05-15</th>
      <td>37.1098</td>
      <td>36.3810</td>
      <td>123.1756</td>
      <td>77.3922</td>
      <td>43.7968</td>
      <td>62.6632</td>
      <td>21.20</td>
      <td>35.2800</td>
      <td>70.8861</td>
      <td>60.20</td>
      <td>...</td>
      <td>25.7956</td>
      <td>40.5850</td>
      <td>84.1199</td>
      <td>44.5893</td>
      <td>26.7974</td>
      <td>34.7654</td>
      <td>48.9596</td>
      <td>96.0770</td>
      <td>26.6862</td>
      <td>29.3696</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 505 columns</p>
</div>




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
      <th>2019-05-02</th>
      <td>78.20</td>
      <td>33.8523</td>
      <td>166.78</td>
      <td>209.15</td>
      <td>78.47</td>
      <td>77.06</td>
      <td>264.77</td>
      <td>78.78</td>
      <td>179.39</td>
      <td>279.64</td>
      <td>...</td>
      <td>55.92</td>
      <td>118.90</td>
      <td>77.29</td>
      <td>51.50</td>
      <td>32.95</td>
      <td>79.54</td>
      <td>101.74</td>
      <td>123.21</td>
      <td>49.48</td>
      <td>103.15</td>
    </tr>
    <tr>
      <th>2019-05-03</th>
      <td>79.29</td>
      <td>34.6899</td>
      <td>163.27</td>
      <td>211.75</td>
      <td>78.71</td>
      <td>79.14</td>
      <td>271.75</td>
      <td>78.69</td>
      <td>176.98</td>
      <td>285.58</td>
      <td>...</td>
      <td>56.58</td>
      <td>119.02</td>
      <td>77.47</td>
      <td>55.05</td>
      <td>32.95</td>
      <td>82.29</td>
      <td>102.72</td>
      <td>124.31</td>
      <td>50.05</td>
      <td>103.75</td>
    </tr>
    <tr>
      <th>2019-05-06</th>
      <td>79.35</td>
      <td>34.6500</td>
      <td>161.99</td>
      <td>208.48</td>
      <td>79.26</td>
      <td>78.31</td>
      <td>268.95</td>
      <td>79.07</td>
      <td>176.26</td>
      <td>283.66</td>
      <td>...</td>
      <td>56.51</td>
      <td>118.81</td>
      <td>77.13</td>
      <td>54.73</td>
      <td>32.65</td>
      <td>80.08</td>
      <td>102.41</td>
      <td>125.55</td>
      <td>49.68</td>
      <td>103.33</td>
    </tr>
    <tr>
      <th>2019-05-07</th>
      <td>76.67</td>
      <td>33.9100</td>
      <td>160.66</td>
      <td>202.86</td>
      <td>77.95</td>
      <td>77.46</td>
      <td>261.98</td>
      <td>76.91</td>
      <td>173.94</td>
      <td>277.07</td>
      <td>...</td>
      <td>56.58</td>
      <td>117.81</td>
      <td>76.72</td>
      <td>54.83</td>
      <td>32.67</td>
      <td>79.17</td>
      <td>101.47</td>
      <td>123.39</td>
      <td>48.71</td>
      <td>101.37</td>
    </tr>
    <tr>
      <th>2019-05-08</th>
      <td>76.61</td>
      <td>33.7500</td>
      <td>158.62</td>
      <td>202.90</td>
      <td>77.99</td>
      <td>78.72</td>
      <td>260.27</td>
      <td>76.22</td>
      <td>173.82</td>
      <td>276.77</td>
      <td>...</td>
      <td>55.88</td>
      <td>117.69</td>
      <td>76.84</td>
      <td>54.95</td>
      <td>32.09</td>
      <td>79.13</td>
      <td>100.49</td>
      <td>122.56</td>
      <td>48.19</td>
      <td>101.86</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 505 columns</p>
</div>




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
      <td>51.467024</td>
      <td>41.343656</td>
      <td>144.824652</td>
      <td>133.794584</td>
      <td>66.064139</td>
      <td>84.739733</td>
      <td>158.525886</td>
      <td>48.392377</td>
      <td>116.636740</td>
      <td>138.544952</td>
      <td>...</td>
      <td>38.786547</td>
      <td>57.890999</td>
      <td>76.467798</td>
      <td>53.016833</td>
      <td>26.795565</td>
      <td>50.742943</td>
      <td>65.957073</td>
      <td>111.383727</td>
      <td>36.627797</td>
      <td>58.504963</td>
    </tr>
    <tr>
      <th>std</th>
      <td>13.768604</td>
      <td>6.216657</td>
      <td>24.250751</td>
      <td>38.325308</td>
      <td>17.881638</td>
      <td>9.845917</td>
      <td>117.226499</td>
      <td>12.213183</td>
      <td>30.722033</td>
      <td>68.878061</td>
      <td>...</td>
      <td>8.124638</td>
      <td>21.478398</td>
      <td>4.883311</td>
      <td>7.892281</td>
      <td>3.139932</td>
      <td>16.191694</td>
      <td>15.459884</td>
      <td>10.036876</td>
      <td>10.820941</td>
      <td>19.725507</td>
    </tr>
    <tr>
      <th>min</th>
      <td>32.258600</td>
      <td>24.539800</td>
      <td>79.168700</td>
      <td>76.961400</td>
      <td>42.066600</td>
      <td>60.668200</td>
      <td>20.880000</td>
      <td>33.935700</td>
      <td>68.852300</td>
      <td>59.590000</td>
      <td>...</td>
      <td>25.247700</td>
      <td>32.609200</td>
      <td>59.643400</td>
      <td>34.178400</td>
      <td>18.532600</td>
      <td>28.968100</td>
      <td>44.366900</td>
      <td>89.236100</td>
      <td>19.008100</td>
      <td>29.196000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>39.221225</td>
      <td>36.834425</td>
      <td>130.553250</td>
      <td>103.314650</td>
      <td>52.521875</td>
      <td>77.600350</td>
      <td>75.840000</td>
      <td>39.159175</td>
      <td>90.555525</td>
      <td>80.567500</td>
      <td>...</td>
      <td>31.174375</td>
      <td>41.414825</td>
      <td>73.431000</td>
      <td>46.986675</td>
      <td>24.215550</td>
      <td>35.094975</td>
      <td>52.731125</td>
      <td>102.304000</td>
      <td>26.912550</td>
      <td>44.327550</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>45.631150</td>
      <td>40.848200</td>
      <td>149.187400</td>
      <td>118.695800</td>
      <td>58.031100</td>
      <td>84.243350</td>
      <td>116.970000</td>
      <td>42.809100</td>
      <td>111.583650</td>
      <td>105.580000</td>
      <td>...</td>
      <td>38.622750</td>
      <td>51.057700</td>
      <td>76.828250</td>
      <td>53.996800</td>
      <td>26.573400</td>
      <td>47.844950</td>
      <td>61.595700</td>
      <td>112.414150</td>
      <td>31.156350</td>
      <td>50.342300</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>65.193450</td>
      <td>46.119900</td>
      <td>161.834300</td>
      <td>166.081025</td>
      <td>82.894300</td>
      <td>90.312650</td>
      <td>236.567500</td>
      <td>57.500400</td>
      <td>147.791425</td>
      <td>197.712500</td>
      <td>...</td>
      <td>45.139600</td>
      <td>68.123175</td>
      <td>79.625850</td>
      <td>59.650550</td>
      <td>29.473700</td>
      <td>66.772550</td>
      <td>79.238550</td>
      <td>118.954900</td>
      <td>46.685425</td>
      <td>76.949925</td>
    </tr>
    <tr>
      <th>max</th>
      <td>81.940000</td>
      <td>57.586600</td>
      <td>199.159900</td>
      <td>230.275400</td>
      <td>116.445400</td>
      <td>108.207500</td>
      <td>449.750000</td>
      <td>79.733700</td>
      <td>182.670000</td>
      <td>289.250000</td>
      <td>...</td>
      <td>57.360000</td>
      <td>139.720000</td>
      <td>87.124800</td>
      <td>67.795300</td>
      <td>35.000000</td>
      <td>83.820000</td>
      <td>104.390000</td>
      <td>130.912800</td>
      <td>57.511000</td>
      <td>103.750000</td>
    </tr>
  </tbody>
</table>
<p>8 rows × 505 columns</p>
</div>



### S&P MidCap 400 Index

Eyeball


```python
sp_data['sp400'].close.shape
```




    (1275, 400)




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
      <td>31.6652</td>
      <td>31.9756</td>
      <td>42.21</td>
      <td>13.4225</td>
      <td>32.07</td>
      <td>NaN</td>
      <td>9.8177</td>
      <td>48.8031</td>
      <td>52.6060</td>
      <td>42.2125</td>
      <td>...</td>
      <td>22.0474</td>
      <td>21.94</td>
      <td>43.5655</td>
      <td>15.3995</td>
      <td>28.3556</td>
      <td>23.8807</td>
      <td>23.75</td>
      <td>407.6382</td>
      <td>54.22</td>
      <td>74.03</td>
    </tr>
    <tr>
      <th>2014-05-12</th>
      <td>32.7774</td>
      <td>31.9510</td>
      <td>43.03</td>
      <td>13.9825</td>
      <td>32.60</td>
      <td>NaN</td>
      <td>9.9030</td>
      <td>49.1525</td>
      <td>52.5488</td>
      <td>42.1031</td>
      <td>...</td>
      <td>22.1535</td>
      <td>22.06</td>
      <td>44.9818</td>
      <td>16.2560</td>
      <td>28.7706</td>
      <td>24.7804</td>
      <td>24.59</td>
      <td>412.2492</td>
      <td>56.60</td>
      <td>74.23</td>
    </tr>
    <tr>
      <th>2014-05-13</th>
      <td>32.2164</td>
      <td>31.6063</td>
      <td>43.06</td>
      <td>13.8000</td>
      <td>32.02</td>
      <td>NaN</td>
      <td>10.0738</td>
      <td>49.0277</td>
      <td>52.5870</td>
      <td>41.9076</td>
      <td>...</td>
      <td>22.0474</td>
      <td>21.88</td>
      <td>44.5868</td>
      <td>16.4166</td>
      <td>28.7152</td>
      <td>24.8187</td>
      <td>23.79</td>
      <td>412.9473</td>
      <td>55.53</td>
      <td>73.41</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 400 columns</p>
</div>




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
      <th>2019-05-02</th>
      <td>56.94</td>
      <td>47.15</td>
      <td>32.26</td>
      <td>34.98</td>
      <td>33.11</td>
      <td>23.90</td>
      <td>24.04</td>
      <td>102.49</td>
      <td>74.93</td>
      <td>80.30</td>
      <td>...</td>
      <td>38.39</td>
      <td>20.31</td>
      <td>108.52</td>
      <td>85.60</td>
      <td>43.71</td>
      <td>14.39</td>
      <td>64.10</td>
      <td>655.30</td>
      <td>40.19</td>
      <td>205.94</td>
    </tr>
    <tr>
      <th>2019-05-03</th>
      <td>58.75</td>
      <td>47.31</td>
      <td>33.42</td>
      <td>35.42</td>
      <td>33.62</td>
      <td>24.89</td>
      <td>24.12</td>
      <td>103.51</td>
      <td>74.82</td>
      <td>82.05</td>
      <td>...</td>
      <td>38.09</td>
      <td>22.96</td>
      <td>112.34</td>
      <td>85.80</td>
      <td>44.63</td>
      <td>16.88</td>
      <td>64.45</td>
      <td>662.82</td>
      <td>40.98</td>
      <td>206.46</td>
    </tr>
    <tr>
      <th>2019-05-06</th>
      <td>59.21</td>
      <td>47.11</td>
      <td>33.82</td>
      <td>35.44</td>
      <td>33.68</td>
      <td>24.35</td>
      <td>23.52</td>
      <td>104.22</td>
      <td>74.15</td>
      <td>82.99</td>
      <td>...</td>
      <td>37.80</td>
      <td>23.14</td>
      <td>111.50</td>
      <td>87.16</td>
      <td>44.58</td>
      <td>16.63</td>
      <td>65.48</td>
      <td>666.31</td>
      <td>40.73</td>
      <td>206.36</td>
    </tr>
    <tr>
      <th>2019-05-07</th>
      <td>58.28</td>
      <td>46.25</td>
      <td>32.60</td>
      <td>34.40</td>
      <td>33.11</td>
      <td>21.97</td>
      <td>22.71</td>
      <td>102.42</td>
      <td>73.11</td>
      <td>82.20</td>
      <td>...</td>
      <td>37.88</td>
      <td>22.54</td>
      <td>109.12</td>
      <td>84.93</td>
      <td>43.92</td>
      <td>16.41</td>
      <td>62.51</td>
      <td>652.75</td>
      <td>39.91</td>
      <td>200.82</td>
    </tr>
    <tr>
      <th>2019-05-08</th>
      <td>57.57</td>
      <td>45.91</td>
      <td>32.56</td>
      <td>34.12</td>
      <td>33.74</td>
      <td>22.03</td>
      <td>22.92</td>
      <td>102.01</td>
      <td>73.00</td>
      <td>80.82</td>
      <td>...</td>
      <td>37.32</td>
      <td>22.41</td>
      <td>108.25</td>
      <td>84.28</td>
      <td>43.69</td>
      <td>15.40</td>
      <td>62.68</td>
      <td>663.53</td>
      <td>40.57</td>
      <td>200.05</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 400 columns</p>
</div>




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
      <td>633.000000</td>
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
      <td>34.951951</td>
      <td>39.217452</td>
      <td>48.998919</td>
      <td>22.336920</td>
      <td>32.397027</td>
      <td>52.542665</td>
      <td>15.623913</td>
      <td>77.034952</td>
      <td>55.537789</td>
      <td>59.215583</td>
      <td>...</td>
      <td>29.498735</td>
      <td>29.819408</td>
      <td>62.681340</td>
      <td>30.624876</td>
      <td>36.782826</td>
      <td>24.768914</td>
      <td>53.553370</td>
      <td>534.299863</td>
      <td>40.549269</td>
      <td>104.380668</td>
    </tr>
    <tr>
      <th>std</th>
      <td>9.474130</td>
      <td>4.988952</td>
      <td>13.759206</td>
      <td>4.049079</td>
      <td>3.178473</td>
      <td>20.129762</td>
      <td>4.052675</td>
      <td>21.216151</td>
      <td>9.369185</td>
      <td>13.551177</td>
      <td>...</td>
      <td>4.838681</td>
      <td>23.446983</td>
      <td>15.011134</td>
      <td>25.081843</td>
      <td>7.050096</td>
      <td>8.713336</td>
      <td>25.852954</td>
      <td>71.294603</td>
      <td>13.384499</td>
      <td>41.295902</td>
    </tr>
    <tr>
      <th>min</th>
      <td>20.118600</td>
      <td>28.081400</td>
      <td>24.750000</td>
      <td>13.280000</td>
      <td>23.150000</td>
      <td>12.570000</td>
      <td>8.872200</td>
      <td>46.728500</td>
      <td>40.309000</td>
      <td>38.117400</td>
      <td>...</td>
      <td>20.741700</td>
      <td>3.780000</td>
      <td>38.963800</td>
      <td>9.233700</td>
      <td>25.207900</td>
      <td>6.475600</td>
      <td>19.560000</td>
      <td>400.952700</td>
      <td>15.230000</td>
      <td>46.930000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>25.957675</td>
      <td>34.702850</td>
      <td>38.800000</td>
      <td>19.390000</td>
      <td>30.370000</td>
      <td>40.435800</td>
      <td>12.782225</td>
      <td>58.763400</td>
      <td>47.042025</td>
      <td>45.533175</td>
      <td>...</td>
      <td>24.339550</td>
      <td>12.060000</td>
      <td>48.495775</td>
      <td>15.573700</td>
      <td>30.984350</td>
      <td>19.100425</td>
      <td>33.210000</td>
      <td>471.868100</td>
      <td>32.002500</td>
      <td>74.200000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>34.380800</td>
      <td>39.909300</td>
      <td>46.660000</td>
      <td>22.055000</td>
      <td>32.425000</td>
      <td>58.764300</td>
      <td>14.581900</td>
      <td>68.115800</td>
      <td>53.312150</td>
      <td>58.312100</td>
      <td>...</td>
      <td>29.706150</td>
      <td>21.115000</td>
      <td>61.817800</td>
      <td>19.171300</td>
      <td>34.616700</td>
      <td>23.906400</td>
      <td>45.895000</td>
      <td>535.896500</td>
      <td>39.565000</td>
      <td>90.710000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>42.126325</td>
      <td>43.673550</td>
      <td>59.717500</td>
      <td>24.070000</td>
      <td>34.537500</td>
      <td>66.641100</td>
      <td>17.854850</td>
      <td>97.221700</td>
      <td>63.243550</td>
      <td>73.313825</td>
      <td>...</td>
      <td>33.338075</td>
      <td>44.240000</td>
      <td>74.387375</td>
      <td>34.439375</td>
      <td>42.889700</td>
      <td>32.300300</td>
      <td>64.747500</td>
      <td>598.808925</td>
      <td>45.815000</td>
      <td>123.212500</td>
    </tr>
    <tr>
      <th>max</th>
      <td>59.210000</td>
      <td>48.853300</td>
      <td>82.970000</td>
      <td>35.520000</td>
      <td>40.130000</td>
      <td>84.046800</td>
      <td>28.421700</td>
      <td>114.972900</td>
      <td>74.930000</td>
      <td>83.830000</td>
      <td>...</td>
      <td>39.060000</td>
      <td>103.090000</td>
      <td>112.340000</td>
      <td>99.250000</td>
      <td>54.884400</td>
      <td>45.690400</td>
      <td>114.540000</td>
      <td>666.310000</td>
      <td>84.960000</td>
      <td>235.440000</td>
    </tr>
  </tbody>
</table>
<p>8 rows × 400 columns</p>
</div>



## S&P SmallCap 600 Index

Eyeball


```python
sp_data['sp600'].close.shape
```




    (1258, 601)




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
      <th>2014-05-09</th>
      <td>17.57</td>
      <td>19.6925</td>
      <td>30.3797</td>
      <td>35.88</td>
      <td>13.26</td>
      <td>19.7560</td>
      <td>62.85</td>
      <td>24.2267</td>
      <td>NaN</td>
      <td>6.48</td>
      <td>...</td>
      <td>NaN</td>
      <td>19.9735</td>
      <td>77.30</td>
      <td>9.2092</td>
      <td>51.2889</td>
      <td>24.7151</td>
      <td>NaN</td>
      <td>18.3379</td>
      <td>23.3449</td>
      <td>28.28</td>
    </tr>
    <tr>
      <th>2014-05-12</th>
      <td>17.44</td>
      <td>20.4546</td>
      <td>30.1511</td>
      <td>36.36</td>
      <td>14.06</td>
      <td>20.1009</td>
      <td>64.66</td>
      <td>24.8687</td>
      <td>NaN</td>
      <td>6.76</td>
      <td>...</td>
      <td>NaN</td>
      <td>20.0868</td>
      <td>79.30</td>
      <td>9.2678</td>
      <td>52.4801</td>
      <td>25.1019</td>
      <td>NaN</td>
      <td>18.5836</td>
      <td>24.3523</td>
      <td>28.93</td>
    </tr>
    <tr>
      <th>2014-05-13</th>
      <td>17.26</td>
      <td>19.8847</td>
      <td>30.1248</td>
      <td>37.40</td>
      <td>13.64</td>
      <td>19.9764</td>
      <td>63.44</td>
      <td>24.5703</td>
      <td>NaN</td>
      <td>6.56</td>
      <td>...</td>
      <td>NaN</td>
      <td>19.8683</td>
      <td>79.31</td>
      <td>9.1571</td>
      <td>52.0263</td>
      <td>24.7246</td>
      <td>NaN</td>
      <td>18.5243</td>
      <td>23.6579</td>
      <td>28.52</td>
    </tr>
    <tr>
      <th>2014-05-14</th>
      <td>17.22</td>
      <td>19.2827</td>
      <td>29.9577</td>
      <td>36.51</td>
      <td>13.01</td>
      <td>19.4878</td>
      <td>62.94</td>
      <td>23.9554</td>
      <td>NaN</td>
      <td>6.24</td>
      <td>...</td>
      <td>11.9624</td>
      <td>19.8279</td>
      <td>77.66</td>
      <td>9.1115</td>
      <td>50.9485</td>
      <td>24.1208</td>
      <td>NaN</td>
      <td>18.1260</td>
      <td>22.7092</td>
      <td>27.72</td>
    </tr>
    <tr>
      <th>2014-05-15</th>
      <td>18.15</td>
      <td>19.1354</td>
      <td>29.7819</td>
      <td>36.49</td>
      <td>13.05</td>
      <td>19.0566</td>
      <td>62.47</td>
      <td>23.7474</td>
      <td>NaN</td>
      <td>6.24</td>
      <td>...</td>
      <td>12.4454</td>
      <td>19.8764</td>
      <td>77.35</td>
      <td>9.0529</td>
      <td>50.2185</td>
      <td>24.0831</td>
      <td>NaN</td>
      <td>18.0921</td>
      <td>22.4745</td>
      <td>27.35</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 601 columns</p>
</div>




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
      <th>2019-05-02</th>
      <td>12.09</td>
      <td>49.52</td>
      <td>46.12</td>
      <td>45.91</td>
      <td>64.26</td>
      <td>36.75</td>
      <td>79.07</td>
      <td>37.42</td>
      <td>30.29</td>
      <td>21.27</td>
      <td>...</td>
      <td>4.39</td>
      <td>27.89</td>
      <td>129.49</td>
      <td>12.50</td>
      <td>84.36</td>
      <td>37.04</td>
      <td>22.31</td>
      <td>24.88</td>
      <td>16.14</td>
      <td>25.79</td>
    </tr>
    <tr>
      <th>2019-05-03</th>
      <td>12.60</td>
      <td>51.81</td>
      <td>46.22</td>
      <td>46.85</td>
      <td>65.30</td>
      <td>37.58</td>
      <td>80.45</td>
      <td>38.36</td>
      <td>37.43</td>
      <td>21.96</td>
      <td>...</td>
      <td>4.77</td>
      <td>28.36</td>
      <td>133.86</td>
      <td>12.73</td>
      <td>86.49</td>
      <td>37.50</td>
      <td>22.99</td>
      <td>25.47</td>
      <td>17.61</td>
      <td>26.94</td>
    </tr>
    <tr>
      <th>2019-05-06</th>
      <td>12.69</td>
      <td>49.35</td>
      <td>46.18</td>
      <td>44.95</td>
      <td>65.84</td>
      <td>37.39</td>
      <td>80.58</td>
      <td>38.41</td>
      <td>37.62</td>
      <td>22.08</td>
      <td>...</td>
      <td>4.86</td>
      <td>28.36</td>
      <td>132.62</td>
      <td>12.78</td>
      <td>87.51</td>
      <td>36.98</td>
      <td>23.00</td>
      <td>25.08</td>
      <td>17.02</td>
      <td>26.53</td>
    </tr>
    <tr>
      <th>2019-05-07</th>
      <td>12.08</td>
      <td>47.38</td>
      <td>45.06</td>
      <td>43.66</td>
      <td>64.64</td>
      <td>36.63</td>
      <td>79.80</td>
      <td>37.96</td>
      <td>36.60</td>
      <td>21.96</td>
      <td>...</td>
      <td>4.77</td>
      <td>27.66</td>
      <td>130.74</td>
      <td>12.62</td>
      <td>85.18</td>
      <td>35.71</td>
      <td>22.62</td>
      <td>24.73</td>
      <td>16.45</td>
      <td>26.12</td>
    </tr>
    <tr>
      <th>2019-05-08</th>
      <td>12.06</td>
      <td>47.35</td>
      <td>45.31</td>
      <td>43.13</td>
      <td>67.00</td>
      <td>36.25</td>
      <td>79.84</td>
      <td>37.93</td>
      <td>36.29</td>
      <td>19.08</td>
      <td>...</td>
      <td>4.73</td>
      <td>27.59</td>
      <td>131.16</td>
      <td>12.52</td>
      <td>84.56</td>
      <td>35.25</td>
      <td>22.68</td>
      <td>24.56</td>
      <td>16.31</td>
      <td>25.95</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 601 columns</p>
</div>




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
      <td>130.000000</td>
      <td>1258.000000</td>
      <td>...</td>
      <td>1254.000000</td>
      <td>1258.00000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1072.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
      <td>1258.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>26.698299</td>
      <td>29.224559</td>
      <td>37.109340</td>
      <td>49.388490</td>
      <td>30.650918</td>
      <td>36.185047</td>
      <td>67.231924</td>
      <td>32.523740</td>
      <td>29.667508</td>
      <td>15.859793</td>
      <td>...</td>
      <td>7.591375</td>
      <td>26.06919</td>
      <td>74.501331</td>
      <td>10.776059</td>
      <td>63.070520</td>
      <td>26.637300</td>
      <td>17.379708</td>
      <td>26.422728</td>
      <td>18.881719</td>
      <td>22.917444</td>
    </tr>
    <tr>
      <th>std</th>
      <td>16.993067</td>
      <td>7.540154</td>
      <td>3.538491</td>
      <td>10.202854</td>
      <td>15.357067</td>
      <td>10.727266</td>
      <td>10.031661</td>
      <td>5.652387</td>
      <td>2.884428</td>
      <td>6.630228</td>
      <td>...</td>
      <td>1.937716</td>
      <td>3.23872</td>
      <td>27.062866</td>
      <td>1.389293</td>
      <td>10.452208</td>
      <td>5.747909</td>
      <td>3.191832</td>
      <td>6.741623</td>
      <td>4.449880</td>
      <td>6.830660</td>
    </tr>
    <tr>
      <th>min</th>
      <td>8.380000</td>
      <td>16.343700</td>
      <td>29.352300</td>
      <td>31.400000</td>
      <td>10.500000</td>
      <td>18.903300</td>
      <td>45.070000</td>
      <td>22.347500</td>
      <td>20.930200</td>
      <td>6.240000</td>
      <td>...</td>
      <td>4.390000</td>
      <td>19.78740</td>
      <td>26.700000</td>
      <td>7.392600</td>
      <td>43.328400</td>
      <td>14.741000</td>
      <td>10.508700</td>
      <td>12.051000</td>
      <td>8.168700</td>
      <td>11.450000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>15.060000</td>
      <td>22.108125</td>
      <td>35.180575</td>
      <td>40.910000</td>
      <td>21.952500</td>
      <td>25.651450</td>
      <td>59.167500</td>
      <td>27.874650</td>
      <td>28.616775</td>
      <td>10.520000</td>
      <td>...</td>
      <td>6.081100</td>
      <td>22.95650</td>
      <td>50.137500</td>
      <td>9.812600</td>
      <td>54.135425</td>
      <td>23.000950</td>
      <td>14.881325</td>
      <td>21.071525</td>
      <td>16.473675</td>
      <td>17.500000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>20.035000</td>
      <td>29.230200</td>
      <td>37.422800</td>
      <td>49.150000</td>
      <td>25.005000</td>
      <td>34.638350</td>
      <td>67.240000</td>
      <td>31.102800</td>
      <td>29.716800</td>
      <td>13.650000</td>
      <td>...</td>
      <td>7.017050</td>
      <td>26.39365</td>
      <td>76.692500</td>
      <td>10.822850</td>
      <td>60.847400</td>
      <td>26.401450</td>
      <td>17.724600</td>
      <td>26.908300</td>
      <td>19.274100</td>
      <td>22.075000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>33.927500</td>
      <td>35.262375</td>
      <td>39.002725</td>
      <td>56.542500</td>
      <td>33.810000</td>
      <td>45.666850</td>
      <td>73.522500</td>
      <td>37.398775</td>
      <td>31.059900</td>
      <td>20.950000</td>
      <td>...</td>
      <td>8.848850</td>
      <td>29.16485</td>
      <td>97.620000</td>
      <td>11.707150</td>
      <td>71.798750</td>
      <td>30.380675</td>
      <td>19.508725</td>
      <td>30.757175</td>
      <td>22.262275</td>
      <td>27.030000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>99.610000</td>
      <td>51.810000</td>
      <td>46.770000</td>
      <td>74.000000</td>
      <td>74.890000</td>
      <td>57.403000</td>
      <td>95.540000</td>
      <td>43.220900</td>
      <td>37.620000</td>
      <td>36.625000</td>
      <td>...</td>
      <td>12.678000</td>
      <td>31.60290</td>
      <td>133.860000</td>
      <td>14.214700</td>
      <td>87.510000</td>
      <td>39.469500</td>
      <td>24.438200</td>
      <td>42.148900</td>
      <td>30.787100</td>
      <td>41.400000</td>
    </tr>
  </tbody>
</table>
<p>8 rows × 601 columns</p>
</div>



## Summary

In this article, we loaded the constituents' symbols from the S&P 500, S&P MidCap 400 and S&P SmallCap 600 indices, and then passed them to pandas-datareader to retrieve historical data from the Investors Exchange. The data collection is a relatively straightforward operation, only requiring a couple of minutes to gather 5 year historical data of all the constituents. In the next article, we code a simple trading strategy and backtest it using the historical data just collected.
