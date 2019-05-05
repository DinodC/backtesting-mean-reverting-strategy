
# How to Use S&P Indices as Investment Universe

## Introduction
I'm currently researching quantitative trading strategies, and would like to share how to use the Standard & Poor's indices as your investment universe.

When you're backtesting an investment strategy, the initial step is to decide the universe of stocks which you want to consider. A natural choice is to pick the Standard & Poor's 500 Index as your universe because it is composed of shares of the 500 largest companies in the US, and more importantly, it provides a list of the most liquid stocks. 

Threading your way through this direction means that you need to collect the list of S&P indices constituents. However, this information is not available on the S&P Dow Jones Indices' website https://us.spindices.com. Also, I contacted their office in Hong Kong, and was informed that the list is only provided to the company's subscribers. Consequently, you will need to retrieve the list from alternative sources.

After a quick search on Google, two alternative sources presented itself: a possible source is https://en.wikipedia.org; and another is https://www.barchart.com. Wikipedia provides the S&P index constituents in the form of a html table, which we will need to scrape using Python library [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/). Barchart provides S&P index constituents as convenient downloadble csv files as long as you have an account (the basic one is free).  

Below I will provide a step-by-step guide on getting and wrangling data from the providers mentioned above. But before that, the next section will provide an overview of the S&P 500 Index as well as the S&P MidCap 400 and S&P SmallCap 600 indices.

## Review of Standard & Poor's Indices
In this section, I provide a quick presentation of the indices which we could consider as stock universe.

### S&P 500 Index
- The [S&P 500 Index](https://us.spindices.com/indices/equity/sp-500) or the _S&P 500_ is a market capitalization-weighted index of the 500 largest US companies.
- The S&P 500 is viewed as the best gauge of large-cap US equity market.
- The S&P 500 has 505 constituents with a median capitalization of USD 22.3B. Note that some companies has several classes of shares like Alphabet. 


### S&P MidCap 400 Index
- The [S&P MidCap 400 Index](https://us.spindices.com/indices/equity/sp-400) or the _S&P 400_ is a market capitalization-weighted index.
- The S&P 400 serves as a benchmark for mid-cap US equity market.
- The S&P 400 has 400 constituents with a median capitalization of USD 4.2B.

### S&P SmallCap 600 Index
- The [S&P SmallCap 600 Index](https://us.spindices.com/indices/equity/sp-600) or the _S&P 600_ is a market capitalization-weighted index.
- The S&P 600 serves as a benchmark for small-cap US equity market.
- The S&P has 601 constituents with a median capitalization of USD 1.2B.

For more information you could also check https://www.investopedia.com. 

## Main

You can find the code below on https://github.com/DinodC/sp-constituents.

Import packages


```python
import numpy as np
```


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

## Wikipedia

Set the pages we want to scrape S&P indices data from https://en.wikipedia.org. Note that page for S&P 600 list of constituents does not exists. However, we can deduce the list from S&P 1000 which is just a combination of the S&P 400 and S&P 600.


```python
page = ['https://en.wikipedia.org/wiki/List_of_S%26P_500_companies',
        'https://en.wikipedia.org/wiki/List_of_S%26P_400_companies',
        'https://en.wikipedia.org/wiki/List_of_S%26P_1000_companies']
```

Define the files we want to store extracted data to


```python
file = ['sp500_wikipedia.pickle',
        'sp400_wikipedia.pickle',
        'sp1000_wikipedia.pickle']
```

The code below scrapes data using [BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/), and saves the extracted data using [pickle](https://docs.python.org/3/library/pickle.html)


```python
for i in range(len(page)):
    
    # Get URL    
    r = re.get(page[i])
    
    # Create a soup object 
    soup = BeautifulSoup(r.text)
    
    # Find S&P constituents table
    table = soup.find('table', attrs={'class', 'wikitable sortable'})
    
    # Get the rows containing the tickers
    tickers = table.find_all('a', attrs={'class', 'external text'})
    # find_all returns tickers and SEC fillings, get tickers only
    tickers = tickers[::2]
    
    # Create a list containing the tickers
    sp = []
    for j in range(len(tickers)):
        sp.append(tickers[j].text)
        
    # Save the list to a file
    output = open(file[i], 'wb')
    pickle.dump(sp, output)
    output.close()
    
```

Define the S&P constituents lists from Wikipedia


```python
sp500_wikipedia = []
sp400_wikipedia = []
sp1000_wikipedia = []
sp_wikipedia = [sp500_wikipedia,
                sp400_wikipedia,
                sp1000_wikipedia]
```

Load the pickled data to the `sp_wikipedia`


```python
for i in range(len(sp_wikipedia)):
    with open(file[i], 'rb') as f:
        sp_wikipedia[i]= pickle.load(f)
    f.close()
```

Check the number of constituents, it should be equal to 505


```python
sp500_wikipedia = sp_wikipedia[0]
```


```python
len(sp500_wikipedia)
```




    505



Check the number of constituents, it should be equal to 400


```python
sp400_wikipedia = sp_wikipedia[1]
```


```python
len(sp400_wikipedia)
```




    400



Check the number of constituents, it should be equal to 1001


```python
sp1000_wikipedia = sp_wikipedia[2]
```


```python
len(sp1000_wikipedia)
```




    1001



Create a list of S&P 600 constituents given that the S&P 1000 index is the sum of S&P 400 and S&P 600 indices


```python
sp600_wikipedia = list(set(sp1000_wikipedia) - set(sp400_wikipedia))
```


```python
len(sp600_wikipedia)
```




    598



In total, Wikipedia tickers in total are only 598, while [S&P Dow Jones Indices](https://www.spindices.com/indices/equity/sp-600) indicate that there should be 601:
- The 3 tickers missing should be due to the fact that S&P 600 list is deduced from S&P 400 and S&P 100 indices, and 
- There could be a possible timing difference when the consituents of SP 400 and SP 1000 indices where updated on Wikipedia.

## Barchart

We download the below files in csv format from https://www.barchart.com. Note that you need to sign up first, free of charge, before getting access.


```python
path = ['s&p-500-index-05-04-2019.csv',
        'sp-400-index-05-04-2019.csv',
        'sp-600-index-05-04-2019.csv']
```

Define the files we want to store extracted data to


```python
file = ['sp500_barchart.pickle',
        'sp400_barchart.pickle',
        'sp600_barchart.pickle']
```

The code below reads the data from the csv file, stores it to a DataFrame object, and saves the extracted information using [pickle](https://docs.python.org/3/library/pickle.html)


```python
for i in range(len(path)):
    
    # Read data to a DataFrame
    data = pd.read_csv(path[i])
    # Exclude the last line since it does not contain a ticker
    data = data[:-1]
    
    # Create a list containing the tickers
    sp = []
    for j in range(len(data['Symbol'])):
        sp.append(data['Symbol'].iloc[j])
        
    # Save the list to a file
    output = open(file[i], 'wb')
    pickle.dump(sp, output)
    output.close()
```

Define the S&P constituents lists from Barchart


```python
sp500_barchart = []
sp400_barchart = []
sp1000_barchart = []
sp_barchart = [sp500_barchart,
                sp400_barchart,
                sp1000_barchart]
```

Load the pickled data to the `sp_barchart`


```python
for i in range(len(sp_barchart)):
    with open(file[i], 'rb') as f:
        sp_barchart[i]= pickle.load(f)
    f.close()
```

Check the number of constituents, it should be equal to 505 according to [S&P Dow Jones Indices](https://us.spindices.com/indices/equity/sp-500)


```python
sp500_barchart = sp_barchart[0]
```


```python
len(sp500_barchart)
```




    505



Check the number of constituents, it should be equal to 400 according to [S&P Dow Jones Indices](https://us.spindices.com/indices/equity/sp-400)


```python
sp400_barchart = sp_barchart[1]
```


```python
len(sp400_barchart)
```




    400



Check the number of constituents, it should be equal to 601 according to [S&P Dow Jones Indices](https://us.spindices.com/indices/equity/sp-600)


```python
sp600_barchart = sp_barchart[2]
```


```python
len(sp600_barchart)
```




    601



## Comparison between Wikipedia and Barchart

### S&P 500 Index

Sort the lists


```python
sp500_wikipedia.sort()
```


```python
sp500_barchart.sort()
```

Eyeball


```python
sp500_wikipedia[:10]
```




    ['A', 'AAL', 'AAP', 'AAPL', 'ABBV', 'ABC', 'ABMD', 'ABT', 'ACN', 'ADBE']




```python
sp500_barchart[:10]
```




    ['A', 'AAL', 'AAP', 'AAPL', 'ABBV', 'ABC', 'ABMD', 'ABT', 'ACN', 'ADBE']



Difference between Wikipedia and Barchart


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

Eyeball


```python
sp400_wikipedia[:10]
```




    ['AAN', 'ACC', 'ACHC', 'ACIW', 'ACM', 'ADNT', 'AEO', 'AFG', 'AGCO', 'AHL']




```python
sp400_barchart[:10]
```




    ['AAN', 'ACC', 'ACHC', 'ACIW', 'ACM', 'ADNT', 'AEO', 'AFG', 'AGCO', 'ALE']



Difference between Wikipedia and Barchart


```python
diff_wikipedia_barchart = list(set(sp400_wikipedia) - set(sp400_barchart))
```


```python
diff_wikipedia_barchart[:10]
```




    ['PBI', 'DO', 'NBR', 'GPOR', 'LHO', 'TFX', 'UNFI', 'SPN', 'EGN', 'ODP']




```python
len(diff_wikipedia_barchart)
```




    28



Difference between Barchart and Wikipedia


```python
diff_barchart_wikipedia = list(set(sp400_barchart) - set(sp400_wikipedia))
```


```python
diff_barchart_wikipedia[:10]
```




    ['EQT', 'YELP', 'AMED', 'BHF', 'TREX', 'GDOT', 'SMTC', 'PSB', 'WH', 'NGVT']




```python
len(diff_barchart_wikipedia)
```




    28



The difference between the two sources Wikipedia and Barchart lists is 28 tickers, which suggests that either Wikipedia list is outdated (Barchart contains updated tickers), or the inverse (Barchart is outdated).

### S&P SmallCap 600 Index

Sort the lists


```python
sp600_wikipedia.sort()
```


```python
sp600_barchart.sort()
```

Eyeball


```python
sp600_wikipedia[:10]
```




    ['AAOI', 'AAON', 'AAT', 'AAWW', 'AAXN', 'ABCB', 'ABG', 'ABM', 'ACET', 'ACLS']




```python
sp600_barchart[:10]
```




    ['AAOI', 'AAON', 'AAT', 'AAWW', 'AAXN', 'ABCB', 'ABG', 'ABM', 'ACA', 'ACLS']



Difference between Wikipedia and Barchart


```python
diff_wikipedia_barchart = list(set(sp600_wikipedia) - set(sp600_barchart))
```


```python
diff_wikipedia_barchart[:10]
```




    ['HYH', 'ILG', 'PERY', 'SONC', 'NTRI', 'FTD', 'AMED', 'EGL', 'GOV', 'PAY']




```python
len(diff_wikipedia_barchart)
```




    51



Difference between Barchart and Wikipedia


```python
diff_barchart_wikipedia = list(set(sp600_barchart) - set(sp600_wikipedia))
```


```python
diff_barchart_wikipedia[:10]
```




    ['ADUS', 'ARLO', 'NRE', 'GPMT', 'ACA', 'PBI', 'MRCY', 'FOE', 'DO', 'ICHR']




```python
len(diff_barchart_wikipedia)
```




    54



In total, Wikipedia tickers in total are only 598, while Barchart tickers are a 601 (complete):
- As previously noted, there 3 tickers missing in Wikipedia list which could be due to timing difference of update between S&P 400 and S&P 1000.
- The difference between the two sources Wikipedia and Barchart lists is 51 tickers (excluding the 3 tickers which could be due to timing difference), which suggests that either  Wikipedia list is outdated (Barchart contains updated tickers), or the inverse (Barchart is outdated). 

## Conclusion

To resume:
- The S&P 500 constituents are consistent between the alternative sources Wikipedia and Barchart.
- While the S&P 400 constituents between the two sources have 28 different tickers, which suggests that the two data providers were not updated on the same time.
- The same remark can be raised for the S&P 600 constituents comparison where there are 51 different tickers. Also, the S&P 600 constituents list from Wikipedia, which was calculated from S&P 400 and S&P 1000 constituents lists, is missing 3 tickers which would suggests that the two indices were not updated on the same time.

Recall that the initial step of the backtesting process is defining the investment universe, which we have completed in this exercise by retrieving the constituents list for the S&P indices.

For the following step in the backtesting process, we need to gather historical data for each stock on the universe which we just defined. More on this on my next article.
