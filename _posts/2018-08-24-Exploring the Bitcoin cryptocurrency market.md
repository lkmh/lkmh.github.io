
## 1. Bitcoin. Cryptocurrencies. So hot right now.
<p>Since the <a href="https://newfronttest.bitcoin.com/bitcoin.pdf">launch of Bitcoin in 2008</a>, hundreds of similar projects based on the blockchain technology have emerged. We call these cryptocurrencies (also coins or cryptos in the Internet slang). Some are extremely valuable nowadays, and others may have the potential to become extremely valuable in the future<sup>1</sup>. In fact, the 6th of December of 2017 Bitcoin has a <a href="https://en.wikipedia.org/wiki/Market_capitalization">market capitalization</a> above $200 billion. </p>
<p><center>
<img src="https://s3.amazonaws.com/assets.datacamp.com/production/project_82/img/bitcoint_market_cap_2017.png" style="width:500px"> <br> 
<em>The astonishing increase of Bitcoin market capitalization in 2017.</em></center></p>
<p>*<sup>1</sup>- <strong>WARNING</strong>: The cryptocurrency market is exceptionally volatile and any money you put in might disappear into thin air.  Cryptocurrencies mentioned here <strong>might be scams</strong> similar to <a href="https://en.wikipedia.org/wiki/Ponzi_scheme">Ponzi Schemes</a> or have many other issues (overvaluation, technical, etc.). <strong>Please do not mistake this for investment advice</strong>. *</p>
<p>That said, let's get to business. As a first task, we will load the current data from the <a href="https://api.coinmarketcap.com">coinmarketcap API</a> and display it in the output.</p>


```python
# Importing pandas
import pandas as pd

# Importing matplotlib and setting aesthetics for plotting later.
import matplotlib.pyplot as plt
%matplotlib inline
%config InlineBackend.figure_format = 'svg' 
plt.style.use('fivethirtyeight')

# Reading in current data from coinmarketcap.com
current = pd.read_json("https://api.coinmarketcap.com/v1/ticker/")

# Printing out the first few lines
# ... YOUR CODE FOR TASK 1 ...

print(current.head())
```

       24h_volume_usd  available_supply            id  last_updated  \
    0    4.385363e+09          17229475       bitcoin    1535178022   
    1    1.505727e+09         101539256      ethereum    1535178037   
    2    2.641081e+08       39524508956        ripple    1535178003   
    3    3.173608e+08          17311575  bitcoin-cash    1535178022   
    4    6.557044e+08         906245118           eos    1535178028   
    
       market_cap_usd    max_supply          name  percent_change_1h  \
    0    116182935912  2.100000e+07       Bitcoin              -0.11   
    1     28485515253           NaN      Ethereum              -0.34   
    2     12961291639  1.000000e+11           XRP              -0.34   
    3      9279774490  2.100000e+07  Bitcoin Cash              -0.11   
    4      4501214387           NaN           EOS              -0.88   
    
       percent_change_24h  percent_change_7d  price_btc    price_usd  rank symbol  \
    0                3.26               3.07   1.000000  6743.266171     1    BTC   
    1                1.45             -10.17   0.041714   280.536970     2    ETH   
    2                1.05              -6.14   0.000049     0.327930     3    XRP   
    3                1.32              -8.80   0.079706   536.044496     4    BCH   
    4                1.25              -7.95   0.000739     4.966884     5    EOS   
    
       total_supply  
    0      17229475  
    1     101539256  
    2   99991865246  
    3      17311575  
    4    1006245120  


## 2. Full dataset, filtering, and reproducibility
<p>The previous API call returns only the first 100 coins, and we want to explore as many coins as possible. Moreover, we can't produce reproducible analysis with live online data. To solve these problems, we will load a CSV we conveniently saved on the 6th of December of 2017 using the API call <code>https://api.coinmarketcap.com/v1/ticker/?limit=0</code> named <code>datasets/coinmarketcap_06122017.csv</code>. </p>


```python
# Reading datasets/coinmarketcap_06122017.csv into pandas
dec6 = pd.read_csv('datasets/coinmarketcap_06122017.csv')

# Selecting the 'id' and the 'market_cap_usd' columns
market_cap_raw = dec6[['id','market_cap_usd']]

# Counting the number of values
# ... YOUR CODE FOR TASK 2 ...
print(market_cap_raw.count())
```

    id                1326
    market_cap_usd    1031
    dtype: int64


## 3. Discard the cryptocurrencies without a market capitalization
<p>Why do the <code>count()</code> for <code>id</code> and <code>market_cap_usd</code> differ above? It is because some cryptocurrencies listed in coinmarketcap.com have no known market capitalization, this is represented by <code>NaN</code> in the data, and <code>NaN</code>s are not counted by <code>count()</code>. These cryptocurrencies are of little interest to us in this analysis, so they are safe to remove.</p>


```python
# Filtering out rows without a market capitalization
cap = market_cap_raw.query('market_cap_usd > 0')

# Counting the number of values again
# ... YOUR CODE FOR TASK 3 ...
print(cap.count())
```

    id                1031
    market_cap_usd    1031
    dtype: int64


## 4. How big is Bitcoin compared with the rest of the cryptocurrencies?
<p>At the time of writing, Bitcoin is under serious competition from other projects, but it is still dominant in market capitalization. Let's plot the market capitalization for the top 10 coins as a barplot to better visualize this.</p>


```python
#Declaring these now for later use in the plots
TOP_CAP_TITLE = 'Top 10 market capitalization'
TOP_CAP_YLABEL = '% of total cap'

# Selecting the first 10 rows and setting the index
cap10 = cap.set_index('id')
cap10 = cap10.iloc[0:10]

# Calculating market_cap_perc
cap10 = cap10.assign(market_cap_perc = lambda x: (x.market_cap_usd / cap.market_cap_usd.sum()) * 100 ) 

# Plotting the barplot with the title defined above 
ax = cap10['market_cap_perc'].plot(kind='bar', title=TOP_CAP_TITLE)

ax.set_ylabel(TOP_CAP_YLABEL)
ax.set_xlabel("")

# Annotating the y axis with the label defined above
# ... YOUR CODE FOR TASK 4 ...
```




    Text(0.5,0,'')




![svg](output_7_1.svg)


## 5. Making the plot easier to read and more informative
<p>While the plot above is informative enough, it can be improved. Bitcoin is too big, and the other coins are hard to distinguish because of this. Instead of the percentage, let's use a log<sup>10</sup> scale of the "raw" capitalization. Plus, let's use color to group similar coins and make the plot more informative<sup>1</sup>. </p>
<p>For the colors rationale: bitcoin-cash and bitcoin-gold are forks of the bitcoin <a href="https://en.wikipedia.org/wiki/Blockchain">blockchain</a><sup>2</sup>. Ethereum and Cardano both offer Turing Complete <a href="https://en.wikipedia.org/wiki/Smart_contract">smart contracts</a>. Iota and Ripple are not minable. Dash, Litecoin, and Monero get their own color.</p>
<p><sup>1</sup> <em>This coloring is a simplification. There are more differences and similarities that are not being represented here.</em></p>
<p><sup>2</sup> <em>The bitcoin forks are actually <strong>very</strong> different, but it is out of scope to talk about them here. Please see the warning above and do your own research.</em></p>


```python
# Colors for the bar plot
COLORS = ['orange', 'green', 'orange', 'cyan', 'cyan', 'blue', 'silver', 'orange', 'red', 'green']

# Plotting market_cap_usd as before but adding the colors and scaling the y-axis  
ax = cap10.market_cap_usd.plot(kind='bar', color=COLORS)
ax.set_yscale('log')
ax.set_ylabel("USD")
ax.set_title("Top 10 market capitalization")
ax.set_xlabel("")
##ax.plot.bar(yscale='log')
# Annotating the y axis with 'USD'
# ... YOUR CODE FOR TASK 5 ...
plt.show()
# Final touch! Removing the xlabel as it is not very informative

# ... YOUR CODE FOR TASK 5 ...
```


![svg](output_9_0.svg)


## 6. What is going on?! Volatility in cryptocurrencies
<p>The cryptocurrencies market has been spectacularly volatile since the first exchange opened. This notebook didn't start with a big, bold warning for nothing. Let's explore this volatility a bit more! We will begin by selecting and plotting the 24 hours and 7 days percentage change, which we already have available.</p>


```python
# Selecting the id, percent_change_24h and percent_change_7d columns
volatility = dec6[['id','percent_change_24h','percent_change_7d']]

# Setting the index to 'id' and dropping all NaN rows
volatility = volatility.set_index('id')
volatility = volatility.dropna(0, how='any')

# Sorting the DataFrame by percent_change_24h in ascending order
volatility = volatility.sort(columns='percent_change_24h',ascending=True)
print(volatility.head(10))
print(volatility.tail(10))
# Checking the first few rows
# ... YOUR CODE FOR TASK 6 ...
```


    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    <ipython-input-6-8fc58f1c15d4> in <module>()
          7 
          8 # Sorting the DataFrame by percent_change_24h in ascending order
    ----> 9 volatility = volatility.sort(columns='percent_change_24h',ascending=True)
         10 print(volatility.head(10))
         11 print(volatility.tail(10))


    /anaconda3/lib/python3.6/site-packages/pandas/core/generic.py in __getattr__(self, name)
       4370             if self._info_axis._can_hold_identifiers_and_holds_name(name):
       4371                 return self[name]
    -> 4372             return object.__getattribute__(self, name)
       4373 
       4374     def __setattr__(self, name, value):


    AttributeError: 'DataFrame' object has no attribute 'sort'


## 7. Well, we can already see that things are *a bit* crazy
<p>It seems you can lose a lot of money quickly on cryptocurrencies. Let's plot the top 10 biggest gainers and top 10 losers in market capitalization.</p>


```python
#Defining a function with 2 parameters, the series to plot and the title
def top10_subplot(volatility_series, title):
    # Making the subplot and the figure for two side by side plots
    fig, axes = plt.subplots(nrows=1, ncols=2, figsize=(10, 6))
    
    # Plotting with pandas the barchart for the top 10 losers
    volatility_series[:10].plot.bar(color='darkred',ax=axes[0])
    
    # Setting the figure's main title to the text passed as parameter
    # ... YOUR CODE FOR TASK 7 ...
    fig.suptitle(title)
    
    # Setting the ylabel to '% change'
    # ... YOUR CODE FOR TASK 7 ...
    
    
    # Same as above, but for the top 10 winners
    volatility_series[-10:].plot.bar(color='darkblue',ax=axes[1])
  
    # Returning this for good practice, might use later
    return fig, ax

DTITLE = "24 hours top losers and winners"

# Calling the function above with the 24 hours period series and title DTITLE  
fig, ax = top10_subplot(volatility.percent_change_24h, DTITLE)
```


![svg](output_13_0.svg)


## 8. Ok, those are... interesting. Let's check the weekly Series too.
<p>800% daily increase?! Why are we doing this tutorial and not buying random coins?<sup>1</sup></p>
<p>After calming down, let's reuse the function defined above to see what is going weekly instead of daily.</p>
<p><em><sup>1</sup> Please take a moment to understand the implications of the red plots on how much value some cryptocurrencies lose in such short periods of time</em></p>


```python
# Sorting in ascending order
volatility7d = volatility.sort_values(by='percent_change_7d')

WTITLE = "Weekly top losers and winners"

# Calling the top10_subplot function
fig, ax = top10_subplot(volatility7d.percent_change_7d, WTITLE)
```


![svg](output_15_0.svg)


## 9. How small is small?
<p>The names of the cryptocurrencies above are quite unknown, and there is a considerable fluctuation between the 1 and 7 days percentage changes. As with stocks, and many other financial products, the smaller the capitalization, the bigger the risk and reward. Smaller cryptocurrencies are less stable projects in general, and therefore even riskier investments than the bigger ones<sup>1</sup>. Let's classify our dataset based on Investopedia's capitalization <a href="https://www.investopedia.com/video/play/large-cap/">definitions</a> for company stocks. </p>
<p><sup>1</sup> <em>Cryptocurrencies are a new asset class, so they are not directly comparable to stocks. Furthermore, there are no limits set in stone for what a "small" or "large" stock is. Finally, some investors argue that bitcoin is similar to gold, this would make them more comparable to a <a href="https://www.investopedia.com/terms/c/commodity.asp">commodity</a> instead.</em></p>


```python
# Selecting everything bigger than 10 billion 
largecaps = cap.query('market_cap_usd > 10000000000')

# Printing out largecaps
# ... YOUR CODE FOR TASK 9 ...
print(largecaps)
```

                 id  market_cap_usd
    0       bitcoin    2.130493e+11
    1      ethereum    4.352945e+10
    2  bitcoin-cash    2.529585e+10
    3          iota    1.475225e+10


## 10. Most coins are tiny
<p>Note that many coins are not comparable to large companies in market cap, so let's divert from the original Investopedia definition by merging categories.</p>
<p><em>This is all for now. Thanks for completing this project!</em></p>


```python
# Making a nice function for counting different marketcaps from the
# "cap" DataFrame. Returns an int.
# INSTRUCTORS NOTE: Since you made it to the end, consider it a gift :D
def capcount(query_string):
    return cap.query(query_string).count().id

# Labels for the plot
LABELS = ["biggish", "micro", "nano"]

# Using capcount count the biggish cryptos
biggish = capcount('market_cap_usd > 300000000') 
print(biggish)
# Same as above for micro ...
micro = capcount('market_cap_usd > 50000000') - capcount('market_cap_usd > 300000000')
print(micro)
# ... and for nano
nano =  capcount('market_cap_usd < 50000000')
print(nano)
# Making a list with the 3 counts
values = [biggish, micro,nano]
print(values)
# Plotting them with matplotlib 
# ... YOUR CODE FOR TASK 10 ...
plt.bar(range(len(values)), values, tick_label=LABELS)
```

    39
    96
    896
    [39, 96, 896]





    <BarContainer object of 3 artists>




![svg](output_19_2.svg)

