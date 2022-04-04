```python
import pandas as pd
import seaborn as sns

```

# Analysis of Company Risk Factors and Correlation to Stock Market Returns
Theo Faucher 
3/25/2022

## The Initial Data

Will start with a short breakdown of the data set utilized and the risk factors applied 


```python
sample_csv = "output/sp500_accting_plus_textrisks.csv"
sample = pd.read_csv(sample_csv)
```

'sample' is a DataFrame containing a list of 505 firms (taken from s&p500) with additional information

The most relevant variables include:
- Firm name
- Firm stock ticker
- Firm Industry/Sector 
- 5 risk factors for each firm

See brief descriptive outputs below


```python
sample.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 505 entries, 0 to 504
    Data columns (total 53 columns):
     #   Column                 Non-Null Count  Dtype  
    ---  ------                 --------------  -----  
     0   Symbol                 505 non-null    object 
     1   Security               505 non-null    object 
     2   SEC filings            505 non-null    object 
     3   GICS Sector            505 non-null    object 
     4   GICS Sub-Industry      505 non-null    object 
     5   Headquarters Location  505 non-null    object 
     6   Date first added       460 non-null    object 
     7   CIK                    505 non-null    int64  
     8   Founded                505 non-null    object 
     9   risk2                  505 non-null    float64
     10  risk1a                 505 non-null    float64
     11  risk1b                 505 non-null    float64
     12  risk1c                 505 non-null    float64
     13  risk3                  505 non-null    float64
     14  gvkey                  355 non-null    float64
     15  lpermno                355 non-null    float64
     16  datadate               355 non-null    object 
     17  fyear                  355 non-null    float64
     18  tic                    355 non-null    object 
     19  sic                    355 non-null    float64
     20  sic3                   355 non-null    float64
     21  td                     355 non-null    float64
     22  long_debt_dum          355 non-null    float64
     23  me                     355 non-null    float64
     24  l_a                    355 non-null    float64
     25  l_sale                 355 non-null    float64
     26  div_d                  355 non-null    float64
     27  age                    355 non-null    float64
     28  atr                    355 non-null    float64
     29  smalltaxlosscarry      276 non-null    float64
     30  largetaxlosscarry      276 non-null    float64
     31  l_emp                  355 non-null    float64
     32  l_ppent                355 non-null    float64
     33  l_laborratio           355 non-null    float64
     34  Inv                    355 non-null    float64
     35  Ch_Cash                355 non-null    float64
     36  Div                    355 non-null    float64
     37  Ch_Debt                355 non-null    float64
     38  Ch_Eqty                355 non-null    float64
     39  Ch_WC                  355 non-null    float64
     40  CF                     355 non-null    float64
     41  td_a                   355 non-null    float64
     42  td_mv                  355 non-null    float64
     43  mb                     355 non-null    float64
     44  prof_a                 355 non-null    float64
     45  ppe_a                  355 non-null    float64
     46  cash_a                 355 non-null    float64
     47  xrd_a                  355 non-null    float64
     48  dltt_a                 355 non-null    float64
     49  invopps_FG09           334 non-null    float64
     50  sales_g                0 non-null      float64
     51  dv_a                   355 non-null    float64
     52  short_debt             349 non-null    float64
    dtypes: float64(42), int64(1), object(10)
    memory usage: 209.2+ KB


# RISK FACTORS*
All risk factors were measured by tallying the number of mentions for certain phrases in each firms' 10k filing (all measured using NEAR_regex). A greater number of risks indicated a certain topic was brought up frequently and could therefore have a sizable impact on firm performance. 

## Risk 1: An extensive/global supply chain

One of the longest running effects from the Covid-19 pandemic is its impact on supply chains. As countries responded differently to the crisis (shutting down factories, limiting imports, etc.) trade was disrupted and the crunch of production inputs is still being felt. I reasoned that a firm with extensive discussions regarding supply chain complexity in its 10k filings would be more reliant on global trade and therefore more susceptible to disruption. In order to get a well rounded look at the issue, I attempted to measure this risk in 3 ways.

1. The first strategy looked for key words or phrases in each 10k that would allude to a cross-boarder/international supply chain. For example, 'global' or 'international' would look to be paired with 'suppliers' or 'supply chain'. As these would often occur within the same sentence (if the topic truly was global supply chain), the NEAR-regex limiter was set to 10. 

2. The second strategy aimed to measure discussion of susceptibility to supply chain issues. By searching for 'supply chain' combined with 'blockage' or 'disruption' (among others) I was able to quantify the amount leadership was worried about these issues. More hits could indicate leadership was aware of this weakness and could translate to gains (either positive or negative, will discuss more in depth later). In this case I set the NEAR_regex limiter to 15 in order to provide greater leeway (disruptions were often talked about in conjunction with business outcomes and would not always be super close to the second term set, even when it was the topic of discussion). 

3. The third strategy was more geared towards the complexity or otherwise delicacy of firms' supply chains. Discussion of a delicate or long supply chains (domestic or international) could indicate firms' weakness in this area. Words like 'complex' or 'extensive' were searched for in conjunction with 'suppliers' or 'inputs' etc. As these would often occur within the same sentence (if the topic truly was supply chain complexity), the NEAR-regex limiter was set to 10.

## Risk 2: Reliance on in-person sales/retail

Covid-19 brought the closure of most in-person activities, and firms reliant on brick-and-mortar sales were among those hardest hit. I think this is a good metric to examine because a firm with numerous instances of in-person sales discussions would be more susceptible to a hit to sales during Covid-19 lockdowns (and would therefore expect to see lower returns in the market). In order to capture these risks, I examined 10Ks for phrases like 'in-person' or 'physical' in conjunction with (among others) 'store' or 'sales channels'. I also included in the search 'mall' and 'malls' as these would often be used in 10Ks as a proxy for brick-and-mortar sales locations. Because this search was a little more broad than the others, I limited the NEAR_regex to 8 words in between target words/phrases.

## Risk 3 - Reliance on Exports

Different nations have recovered from Covid-19 at differing rates and success. This has had an effect on global trade beyond supply chain disruptions. Firms that have exports as a significant part of revenues are reliant on foreign economies bouncing back rapidly from the pandemic. The United States economy has mounted an impressive recovery, and these conditions may not be the same for foreign countries the examined firms sell to. In order to quantify this metric I searched 10Ks for words/phrases like 'susceptible' or 'reliant' combined with 'exports' or 'overseas' as well as things like 'tariffs' (which would certainly affect exports but would otherwise not be caught by NEAR_regex). In this search, NEAR_regex was limited to 10 words.




*A more detailed look at the NEAR_regex search parameters can be seen in the 'measure_risk.ipynb' file in this repo


```python
print("Risk 1A : " , sample['risk1a'].describe())
```

    Risk 1A :  count    505.000000
    mean       2.477228
    std        3.239142
    min        0.000000
    25%        0.000000
    50%        1.000000
    75%        3.000000
    max       21.000000
    Name: risk1a, dtype: float64



```python
print("Risk 1B : " , sample['risk1b'].describe())
```

    Risk 1B :  count    505.000000
    mean       0.809901
    std        1.181664
    min        0.000000
    25%        0.000000
    50%        0.000000
    75%        1.000000
    max        7.000000
    Name: risk1b, dtype: float64



```python
print("Risk 1C : " , sample['risk1c'].describe())
```

    Risk 1C :  count    505.000000
    mean       2.560396
    std        3.061670
    min        0.000000
    25%        0.000000
    50%        2.000000
    75%        4.000000
    max       18.000000
    Name: risk1c, dtype: float64



```python
print("Risk 2 : " , sample['risk2'].describe())
```

    Risk 2 :  count    505.000000
    mean       3.570297
    std        9.096676
    min        0.000000
    25%        0.000000
    50%        0.000000
    75%        3.000000
    max       83.000000
    Name: risk2, dtype: float64



```python
print("Risk 3 : " , sample['risk3'].describe())
```

    Risk 3 :  count    505.000000
    mean       0.859406
    std        1.460470
    min        0.000000
    25%        0.000000
    50%        0.000000
    75%        1.000000
    max        9.000000
    Name: risk3, dtype: float64


# Data Validation

Before drawing any conclusions from my data it was important to first validate my searches were incorporating the information I expected/intended it to. Due to the size of the dataset, this posed a bit of a problem but I developed several workarounds. First, I added my risk measures to the firms dataset and then grouped by GISC Industry, finding the average number of hits for each risk factor per industry. This allowed me to take a 'common sense' look at the data and see if it was what I expected. For example, I expected Risk2 (reliance on in-person sales) to be much higher in industries like Consumer Staples than Financials or IT. The average Consumer Staples firm had roughly 10 hits for Risk2, compared to an average of 1.2 (IT) and 1.3 (Financials). Further, I expected firms that deal with physical goods (Industrials, Materials, etc.) to provide more hits for Risk1 (supply chain concerns) than industries operating with more intangible outputs. This was confirmed in my search, with industrials and materials averaging over 3 hits per firm per Risk1 (IT and financials average between 1 and 2). 

Further, I examined several 10Ks to get a rough idea of what my search would pick out. The following example of a successful 'hit' was taken from the AMCOR 10K:

"Supply shortages or disruptions in our supply chains, including as a result of sourcing materials from a single supplier,
could affect our ability to obtain timely delivery of raw materials, equipment and other supplies, and in turn, adversely affect
our ability to supply products to our customers. Such disruptions could have an adverse effect on our business and financial
results" (2019, page 13)


This excerpt would successfully register under Risk1b (which looks at a firms susceptibility to supply chain disruptions).

# Final Data Set

In order to fully combine my analysis and examine the affect of risk factors on market returns, I needed to add some return variables to my 'sample' data set. I calculated the cummulative returns for the week of 3/9/2020 - 3/13/2020. 


```python
final_csv = "output/final_data.csv"
final = pd.read_csv(final_csv)
```


```python
final.describe()
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
      <th>CIK</th>
      <th>risk2</th>
      <th>risk1a</th>
      <th>risk1b</th>
      <th>risk1c</th>
      <th>risk3</th>
      <th>gvkey</th>
      <th>lpermno</th>
      <th>fyear</th>
      <th>sic</th>
      <th>...</th>
      <th>prof_a</th>
      <th>ppe_a</th>
      <th>cash_a</th>
      <th>xrd_a</th>
      <th>dltt_a</th>
      <th>invopps_FG09</th>
      <th>sales_g</th>
      <th>dv_a</th>
      <th>short_debt</th>
      <th>cum_ret</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>5.050000e+02</td>
      <td>505.000000</td>
      <td>505.000000</td>
      <td>505.000000</td>
      <td>505.000000</td>
      <td>505.000000</td>
      <td>355.000000</td>
      <td>355.000000</td>
      <td>355.000000</td>
      <td>355.000000</td>
      <td>...</td>
      <td>355.000000</td>
      <td>355.000000</td>
      <td>355.000000</td>
      <td>355.000000</td>
      <td>355.000000</td>
      <td>334.000000</td>
      <td>0.0</td>
      <td>355.000000</td>
      <td>349.000000</td>
      <td>490.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>7.887301e+05</td>
      <td>3.570297</td>
      <td>2.477228</td>
      <td>0.809901</td>
      <td>2.560396</td>
      <td>0.859406</td>
      <td>45305.952113</td>
      <td>53570.729577</td>
      <td>2018.884507</td>
      <td>4320.836620</td>
      <td>...</td>
      <td>0.151314</td>
      <td>0.247454</td>
      <td>0.126002</td>
      <td>0.031169</td>
      <td>0.296568</td>
      <td>2.698513</td>
      <td>NaN</td>
      <td>0.025464</td>
      <td>0.112481</td>
      <td>-0.121810</td>
    </tr>
    <tr>
      <th>std</th>
      <td>5.501050e+05</td>
      <td>9.096676</td>
      <td>3.239142</td>
      <td>1.181664</td>
      <td>3.061670</td>
      <td>1.460470</td>
      <td>61170.060945</td>
      <td>30143.136238</td>
      <td>0.320067</td>
      <td>1946.653427</td>
      <td>...</td>
      <td>0.074428</td>
      <td>0.218987</td>
      <td>0.138469</td>
      <td>0.050173</td>
      <td>0.181230</td>
      <td>2.107435</td>
      <td>NaN</td>
      <td>0.026991</td>
      <td>0.111168</td>
      <td>0.090491</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.800000e+03</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>1045.000000</td>
      <td>10104.000000</td>
      <td>2018.000000</td>
      <td>100.000000</td>
      <td>...</td>
      <td>-0.323828</td>
      <td>0.009521</td>
      <td>0.002073</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.405435</td>
      <td>NaN</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>-0.610145</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>9.747600e+04</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>6286.000000</td>
      <td>19531.500000</td>
      <td>2019.000000</td>
      <td>2844.000000</td>
      <td>...</td>
      <td>0.102413</td>
      <td>0.091581</td>
      <td>0.031900</td>
      <td>0.000000</td>
      <td>0.177941</td>
      <td>1.234730</td>
      <td>NaN</td>
      <td>0.000000</td>
      <td>0.028043</td>
      <td>-0.159331</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>8.820950e+05</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>2.000000</td>
      <td>0.000000</td>
      <td>13700.000000</td>
      <td>58683.000000</td>
      <td>2019.000000</td>
      <td>3760.000000</td>
      <td>...</td>
      <td>0.138699</td>
      <td>0.162561</td>
      <td>0.072171</td>
      <td>0.009526</td>
      <td>0.285137</td>
      <td>2.155533</td>
      <td>NaN</td>
      <td>0.020454</td>
      <td>0.084992</td>
      <td>-0.106716</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>1.137789e+06</td>
      <td>3.000000</td>
      <td>3.000000</td>
      <td>1.000000</td>
      <td>4.000000</td>
      <td>1.000000</td>
      <td>61582.500000</td>
      <td>82620.000000</td>
      <td>2019.000000</td>
      <td>5455.500000</td>
      <td>...</td>
      <td>0.186883</td>
      <td>0.336729</td>
      <td>0.166690</td>
      <td>0.042936</td>
      <td>0.390672</td>
      <td>3.301717</td>
      <td>NaN</td>
      <td>0.037617</td>
      <td>0.151231</td>
      <td>-0.064739</td>
    </tr>
    <tr>
      <th>max</th>
      <td>1.868275e+06</td>
      <td>83.000000</td>
      <td>21.000000</td>
      <td>7.000000</td>
      <td>18.000000</td>
      <td>9.000000</td>
      <td>316056.000000</td>
      <td>93436.000000</td>
      <td>2019.000000</td>
      <td>8742.000000</td>
      <td>...</td>
      <td>0.390384</td>
      <td>0.928562</td>
      <td>0.694612</td>
      <td>0.336795</td>
      <td>1.071959</td>
      <td>12.164233</td>
      <td>NaN</td>
      <td>0.138594</td>
      <td>0.761029</td>
      <td>0.117748</td>
    </tr>
  </tbody>
</table>
<p>8 rows × 44 columns</p>
</div>




```python
final.head(5)
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
      <th>Symbol</th>
      <th>Security</th>
      <th>SEC filings</th>
      <th>GICS Sector</th>
      <th>GICS Sub-Industry</th>
      <th>Headquarters Location</th>
      <th>Date first added</th>
      <th>CIK</th>
      <th>Founded</th>
      <th>risk2</th>
      <th>...</th>
      <th>prof_a</th>
      <th>ppe_a</th>
      <th>cash_a</th>
      <th>xrd_a</th>
      <th>dltt_a</th>
      <th>invopps_FG09</th>
      <th>sales_g</th>
      <th>dv_a</th>
      <th>short_debt</th>
      <th>cum_ret</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>MMM</td>
      <td>3M</td>
      <td>reports</td>
      <td>Industrials</td>
      <td>Industrial Conglomerates</td>
      <td>Saint Paul, Minnesota</td>
      <td>1976-08-09</td>
      <td>66740</td>
      <td>1902</td>
      <td>4.0</td>
      <td>...</td>
      <td>0.193936</td>
      <td>0.228196</td>
      <td>0.065407</td>
      <td>0.042791</td>
      <td>0.408339</td>
      <td>2.749554</td>
      <td>NaN</td>
      <td>0.074252</td>
      <td>0.143810</td>
      <td>-0.077905</td>
    </tr>
    <tr>
      <th>1</th>
      <td>AOS</td>
      <td>A. O. Smith</td>
      <td>reports</td>
      <td>Industrials</td>
      <td>Building Products</td>
      <td>Milwaukee, Wisconsin</td>
      <td>2017-07-26</td>
      <td>91142</td>
      <td>1916</td>
      <td>3.0</td>
      <td>...</td>
      <td>0.177698</td>
      <td>0.193689</td>
      <td>0.180314</td>
      <td>0.028744</td>
      <td>0.103303</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.048790</td>
      <td>0.056170</td>
      <td>-0.028109</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ABT</td>
      <td>Abbott</td>
      <td>reports</td>
      <td>Health Care</td>
      <td>Health Care Equipment</td>
      <td>North Chicago, Illinois</td>
      <td>1964-03-31</td>
      <td>1800</td>
      <td>1888</td>
      <td>2.0</td>
      <td>...</td>
      <td>0.118653</td>
      <td>0.132161</td>
      <td>0.060984</td>
      <td>0.035942</td>
      <td>0.256544</td>
      <td>2.520681</td>
      <td>NaN</td>
      <td>0.033438</td>
      <td>0.088120</td>
      <td>-0.001101</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ABBV</td>
      <td>AbbVie</td>
      <td>reports</td>
      <td>Health Care</td>
      <td>Pharmaceuticals</td>
      <td>North Chicago, Illinois</td>
      <td>2012-12-31</td>
      <td>1551152</td>
      <td>2013 (1888)</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.178107</td>
      <td>0.037098</td>
      <td>0.448005</td>
      <td>0.076216</td>
      <td>0.709488</td>
      <td>2.211589</td>
      <td>NaN</td>
      <td>0.071436</td>
      <td>0.057566</td>
      <td>-0.038844</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ABMD</td>
      <td>Abiomed</td>
      <td>reports</td>
      <td>Health Care</td>
      <td>Health Care Equipment</td>
      <td>Danvers, Massachusetts</td>
      <td>2018-05-31</td>
      <td>815094</td>
      <td>1981</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.225749</td>
      <td>0.137531</td>
      <td>0.466354</td>
      <td>0.088683</td>
      <td>0.000000</td>
      <td>12.164233</td>
      <td>NaN</td>
      <td>0.000000</td>
      <td>NaN</td>
      <td>-0.090781</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 54 columns</p>
</div>



## EDA to watch for 

1. Not all firms from list (505) had 10Ks available to download 
2. Only had returns for 490 of the listed firms 
3. Cumulative returns were generally negative across the board (fits expectations)
4. Final dataframe has lots of columns not relevant to this analysis (e.g. 'Headquarters Location') which could possibly be trimmed if working more extensively with the data 
5. Some risk variables had stark outliers (e.g. Risk2 has a max of 83 hits with an average of 3.6 hits). I eliminated this outlier for my analysis

Possible Concerns - My analysis was limited to just the week of 3/9/2020 - 3/13/2020 and analysis could be bolstered by expanding that range or adding more return variables. 

# Results

I did not find any strong relationships between the measured risk variables and stock market returns. While I had expected to find some correlation, especially between supply chain disruptions and negative returns, I am not deterred. 

Theories on why no correlation can be found in the data:
1. No correlation exists in reality 
2. My risk measurement analysis was misapplied or improperly built 
3. Discussion of risk factors can signal a firm is more susceptible to said factor OR that it is better prepared
    - For example, a firm with an extensive supply chain would dedicate a lot of time and brainpower to that supply chain. This would result in more mentions in the 10K (driving up the 'risk'), but would also mean the firm is more prepared to that potential risk. The relationship between risk and returns is being pulled down on one hand (more hits = greater risk) while being pushed up on the other (more hits = better prepared for said risk). This problem could be examined further by expanding my analysis of returns and further narrowing my risk parameters in the hopes of mitigating the preparedness factor.

## Graphs


```python
ax = sns.regplot(data=final, x='risk1a',y='cum_ret')
```


    
![png](output_20_0.png)
    



```python
ax = sns.regplot(data=final, x='risk1b',y='cum_ret')
```


    
![png](output_21_0.png)
    



```python
ax = sns.regplot(data=final, x='risk1c',y='cum_ret')
```


    
![png](output_22_0.png)
    



```python
ax = sns.regplot(data=final, x='risk2',y='cum_ret')
```


    
![png](output_23_0.png)
    



```python
ax = sns.regplot(data=final, x='risk3',y='cum_ret')
```


    
![png](output_24_0.png)
    



```python

```
