
# Backtesting A Trading Strategy Part 1

## How To Scrape S&P Constituents Tickers Using Python

## Introduction
Backtesting is a tool to measure the performance of a trading strategy using historical data. The backtesting process consists of three parts: 1. determining the universe of securities where we will invest in (e.g. equity or fixed income? US or emerging markets?); 2. gathering historical data for the universe of securities; and 3. implementing a trading strategy using the historical data collected.

In this article, I will discuss the initial step in the backtesting process: determining the universe of securities. If we focus our attention on trading US equities, then a natural choice is the Standard and Poor's 500 Index which is composed of shares of the 500 largest companies in the United States. The S&P 500 also provides the most liquid stocks. We can also consider the S&P MidCap 400 and S&P SmallCap 600 indices.

## Standard & Poor's Dow Jones Indices
This section provides a comparison of the different S&P indices which can be considered as possible universe of stocks.

### S&P 500 Index
- is a market capitalization-weighted index of the 500 largest US companies
- is viewed as the best gauge of large-cap US equity market
- has 505 constituents with a median capitalization of USD 22.3B

### S&P MidCap 400 Index
- is a market capitalization-weighted index
- serves as a benchmark for mid-cap US equity market
- has 400 constituents with a median capitalization of USD 4.2B

### S&P SmallCap 600 Index
- is a market capitalization-weighted index
- serves as a benchmark for small-cap US equity market
- has 601 constituents with a median capitalization of USD 1.2B

After identifying potential universe of stocks candidates, we need to collect the list of constituents for each candidate universe. The list of constituents is not available on the official S&P Dow Jones Indices website https://us.spindices.com. The constituents are only provided to product subscribers. We therefore, need to find alternative data providers. After a quick search on Google, two candidates are available: [Wikipedia](https://en.wikipedia.org); and [Barchart](https://www.barchart.com). Wikipedia provides the S&P constituents in the form of a HTML table, which we will need to retrieve using Python package [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) for web scraping. Barchart provides the S&P constituents as convenient downloadable CSV files. You just need to create a basic account with them, which fortunately is free.

## Retrieving S&P Constituents
### Step By Step
1. Collect the S&P tickers from Wikipedia, and then from Barchart.
2. Compare the S&P constituents from the two providers.

You can find the code below on https://github.com/DinodC/backtesting-trading-strategy.

Import packages


```python
import pandas as pd
```


```python
import requests as re
```


```python
from bs4 import BeautifulSoup
```


```python
import pickle
```

### From Wikipedia

Set an id for each index


```python
id = ['sp500', 'sp400', 'sp1000']
```

Set the pages we want to scrape S&P indices data from https://en.wikipedia.org. Note that page for S&P 600 list of constituents does not exists. However, we can deduce the list from S&P 1000 which is just a combination of the S&P 400 and S&P 600.


```python
input_file = {'sp500': 'https://en.wikipedia.org/wiki/List_of_S%26P_500_companies',
              'sp400': 'https://en.wikipedia.org/wiki/List_of_S%26P_400_companies',
              'sp1000': 'https://en.wikipedia.org/wiki/List_of_S%26P_1000_companies'}
```

Define the files we want to store extracted data to


```python
output_file = {'sp500': 'sp500_wikipedia.pickle',
               'sp400': 'sp400_wikipedia.pickle',
               'sp1000': 'sp1000_wikipedia.pickle'}
```

Define the S&P constituents lists from Wikipedia


```python
sp500_wikipedia = []
sp400_wikipedia = []
sp1000_wikipedia = []
sp_wikipedia = {'sp500': sp500_wikipedia,
                'sp400': sp400_wikipedia,
                'sp1000': sp1000_wikipedia}
```

The code below scrapes data using [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/), and saves the extracted data using [pickle](https://docs.python.org/3/library/pickle.html)


```python
for i in input_file:

    # Get URL    
    r = re.get(input_file[i])

    # Create a soup object
    soup = BeautifulSoup(r.text)

    # Find S&P constituents table
    table = soup.find('table', attrs={'class', 'wikitable sortable'})

    # Get the rows containing the tickers
    tickers = table.find_all('a', attrs={'class', 'external text'})
    # find_all returns tickers and SEC fillings, get tickers only
    tickers = tickers[::2]

    # Create a list containing the tickers
    for j in range(len(tickers)):
        sp_wikipedia[i].append(tickers[j].text)

    # Save the list to a file
    with open(output_file[i], 'wb') as f:
        pickle.dump(sp_wikipedia[i], f)
    f.close()

```

Check the number of constituents, it should be equal to 505


```python
len(sp500_wikipedia)
```




    505



Check the number of constituents, it should be equal to 400


```python
len(sp400_wikipedia)
```




    400



Check the number of constituents, it should be equal to 1001


```python
len(sp1000_wikipedia)
```




    1001



Create a list of S&P 600 constituents given that the S&P 1000 index is the sum of S&P 400 and S&P 600 indices


```python
sp600_wikipedia = list(set(sp1000_wikipedia) - set(sp400_wikipedia))
```

Check the number of constituents, it should be equal to 601


```python
len(sp600_wikipedia)
```




    598



In total, Wikipedia tickers sum up to 598 only, while [S&P Dow Jones Indices](https://www.spindices.com/indices/equity/sp-600) indicate that there should be 601. The missing tickers, 3 in total, could be due to timing difference in updating the S&P 400 and S&P 1000 lists.

### From Barchart


```python
id = ['sp500', 'sp400', 'sp600']
```

We download the below files in csv format from https://www.barchart.com. Note that you need to sign up first, free of charge, before getting access.


```python
input_file = {'sp500': 's&p-500-index-05-04-2019.csv',
              'sp400': 'sp-400-index-05-04-2019.csv',
              'sp600': 'sp-600-index-05-04-2019.csv'}
```

Define the files we want to store extracted data to


```python
output_file = {'sp500': 'sp500_barchart.pickle',
               'sp400': 'sp400_barchart.pickle',
               'sp600': 'sp600_barchart.pickle'}
```

Define the S&P constituents lists from Barchart


```python
sp500_barchart = []
sp400_barchart = []
sp600_barchart = []
sp_barchart = {'sp500': sp500_barchart,
              'sp400': sp400_barchart,
              'sp600': sp600_barchart}
```

The code below reads the data from the csv file, stores it to a DataFrame object, and saves the extracted information using [pickle](https://docs.python.org/3/library/pickle.html)


```python
for i in input_file:

    # Read data to a DataFrame
    data = pd.read_csv(input_file[i])
    # Exclude the last line since it does not contain a ticker
    data = data[:-1]

    # Create a list containing the tickers
    for j in range(len(data['Symbol'])):
        sp_barchart[i].append(data['Symbol'].iloc[j])

    # Save the list to a file
    with open(output_file[i], 'wb') as f:
        pickle.dump(sp_barchart[i], f)
    f.close()
```

Check the number of constituents, it should be equal to 505 according to [S&P Dow Jones Indices](https://us.spindices.com/indices/equity/sp-500)


```python
len(sp500_barchart)
```




    505



Check the number of constituents, it should be equal to 400 according to [S&P Dow Jones Indices](https://us.spindices.com/indices/equity/sp-400)


```python
len(sp400_barchart)
```




    400



Check the number of constituents, it should be equal to 601 according to [S&P Dow Jones Indices](https://us.spindices.com/indices/equity/sp-600)


```python
len(sp600_barchart)
```




    601



## Comparison Between Wikipedia and Barchart

### S&P 500 Index

Sort the lists


```python
sp500_wikipedia.sort()
```


```python
sp500_barchart.sort()
```

Compare the first 10 tickers


```python
sp500_wikipedia[:10]
```




    ['A', 'AAL', 'AAP', 'AAPL', 'ABBV', 'ABC', 'ABMD', 'ABT', 'ACN', 'ADBE']




```python
sp500_barchart[:10]
```




    ['A', 'AAL', 'AAP', 'AAPL', 'ABBV', 'ABC', 'ABMD', 'ABT', 'ACN', 'ADBE']



Compare all the tickers by calculating the difference between Wikipedia and Barchart


```python
diff_wikipedia_barchart = list(set(sp500_wikipedia) - set(sp500_barchart))
```


```python
diff_wikipedia_barchart
```




    []



There is no difference between the Wikipedia and Barchart lists.

### S&P MidCap 400 Index

Sort the lists


```python
sp400_wikipedia.sort()
```


```python
sp400_barchart.sort()
```

Compare the first 10 tickers


```python
sp400_wikipedia[:10]
```




    ['AAN', 'ACC', 'ACHC', 'ACIW', 'ACM', 'ADNT', 'AEO', 'AFG', 'AGCO', 'AHL']




```python
sp400_barchart[:10]
```




    ['AAN', 'ACC', 'ACHC', 'ACIW', 'ACM', 'ADNT', 'AEO', 'AFG', 'AGCO', 'ALE']



Compare all the tickers by calculating the difference between Wikipedia and Barchart


```python
diff_wikipedia_barchart = list(set(sp400_wikipedia) - set(sp400_barchart))
```


```python
diff_wikipedia_barchart[:10]
```




    ['DNB', 'ODP', 'IDTI', 'LPNT', 'SPN', 'EGN', 'AKRX', 'NBR', 'ATO', 'WTW']




```python
len(diff_wikipedia_barchart)
```




    28



Now, compare all the tickers by calculating the difference between Barchart and Wikipedia


```python
diff_barchart_wikipedia = list(set(sp400_barchart) - set(sp400_wikipedia))
```


```python
diff_barchart_wikipedia[:10]
```




    ['NGVT', 'EQT', 'YELP', 'OLED', 'BHF', 'TREX', 'GT', 'CACI', 'BRX', 'CZR']




```python
len(diff_barchart_wikipedia)
```




    28



The difference between the two providers Wikipedia and Barchart is 28 tickers, which suggests that one of the two providers has a more up to date list.

### S&P SmallCap 600 Index

Sort the lists


```python
sp600_wikipedia.sort()
```


```python
sp600_barchart.sort()
```

Compare the first 10 tickers


```python
sp600_wikipedia[:10]
```




    ['AAOI', 'AAON', 'AAT', 'AAWW', 'AAXN', 'ABCB', 'ABG', 'ABM', 'ACET', 'ACLS']




```python
sp600_barchart[:10]
```




    ['AAOI', 'AAON', 'AAT', 'AAWW', 'AAXN', 'ABCB', 'ABG', 'ABM', 'ACA', 'ACLS']



Compare all the tickers by calculating the difference between Wikipedia and Barchart


```python
diff_wikipedia_barchart = list(set(sp600_wikipedia) - set(sp600_barchart))
```


```python
diff_wikipedia_barchart[:10]
```




    ['GOV', 'NGVT', 'MHLD', 'SONC', 'XOXO', 'EGL', 'MDXG', 'CVG', 'CLD', 'QHC']




```python
len(diff_wikipedia_barchart)
```




    51



Now, compare all the tickers by calculating the difference between Barchart and Wikipedia


```python
diff_barchart_wikipedia = list(set(sp600_barchart) - set(sp600_wikipedia))
```


```python
diff_barchart_wikipedia[:10]
```




    ['NRE', 'NEO', 'AX', 'ACA', 'ODP', 'MEDP', 'SGH', 'WRE', 'SPN', 'GTX']




```python
len(diff_barchart_wikipedia)
```




    54



In total, Wikipedia constituents sum up to 598 only, while Barchart sum up to 601 (complete):
1. As previously noted, there are 3 tickers missing in Wikipedia list which could be due to timing difference in updating the S&P 400 and S&P 1000 lists.
2. The difference between the two providers Wikipedia and Barchart is 51 tickers, which suggests that one of the two providers has a more up to date list.

## Summary
In this article, we identified universe of securities candidates such as the S&P 500, S&P MidCap 400 and S&P SmallCap indices. We retrieved the constituents of each index from alternative data providers, namely Wikipedia and Barchart. The list of tickers for the S&P 500 is consistent between the two sources, while the list of tickers for the S&P MidCap 400 and S&P SmallCap 600 are not identical. There seems to be an inconsistency between the components of the S&P 400 and S&P 1000 indices from Wikipedia. As a result, we will consider the S&P constituents from Barchart in the next article where we will retrieve historical data for every ticker in every index.
