
---

_You are currently looking at **version 1.5** of this notebook. To download notebooks and datafiles, as well as get help on Jupyter notebooks in the Coursera platform, visit the [Jupyter Notebook FAQ](https://www.coursera.org/learn/python-data-analysis/resources/0dhYG) course resource._

---

# Assignment 3 - More Pandas
This assignment requires more individual learning then the last one did - you are encouraged to check out the [pandas documentation](http://pandas.pydata.org/pandas-docs/stable/) to find functions or methods you might not have used yet, or ask questions on [Stack Overflow](http://stackoverflow.com/) and tag them as pandas and python related. And of course, the discussion forums are open for interaction with your peers and the course staff.

### Question 1 (20%)
Load the energy data from the file `Energy Indicators.xls`, which is a list of indicators of [energy supply and renewable electricity production](Energy%20Indicators.xls) from the [United Nations](http://unstats.un.org/unsd/environment/excel_file_tables/2013/Energy%20Indicators.xls) for the year 2013, and should be put into a DataFrame with the variable name of **energy**.

Keep in mind that this is an Excel file, and not a comma separated values file. Also, make sure to exclude the footer and header information from the datafile. The first two columns are unneccessary, so you should get rid of them, and you should change the column labels so that the columns are:

`['Country', 'Energy Supply', 'Energy Supply per Capita', '% Renewable']`

Convert `Energy Supply` to gigajoules (there are 1,000,000 gigajoules in a petajoule). For all countries which have missing data (e.g. data with "...") make sure this is reflected as `np.NaN` values.

Rename the following list of countries (for use in later questions):

```"Republic of Korea": "South Korea",
"United States of America": "United States",
"United Kingdom of Great Britain and Northern Ireland": "United Kingdom",
"China, Hong Kong Special Administrative Region": "Hong Kong"```

There are also several countries with numbers and/or parenthesis in their name. Be sure to remove these, 

e.g. 

`'Bolivia (Plurinational State of)'` should be `'Bolivia'`, 

`'Switzerland17'` should be `'Switzerland'`.

<br>

Next, load the GDP data from the file `world_bank.csv`, which is a csv containing countries' GDP from 1960 to 2015 from [World Bank](http://data.worldbank.org/indicator/NY.GDP.MKTP.CD). Call this DataFrame **GDP**. 

Make sure to skip the header, and rename the following list of countries:

```"Korea, Rep.": "South Korea", 
"Iran, Islamic Rep.": "Iran",
"Hong Kong SAR, China": "Hong Kong"```

<br>

Finally, load the [Sciamgo Journal and Country Rank data for Energy Engineering and Power Technology](http://www.scimagojr.com/countryrank.php?category=2102) from the file `scimagojr-3.xlsx`, which ranks countries based on their journal contributions in the aforementioned area. Call this DataFrame **ScimEn**.

Join the three datasets: GDP, Energy, and ScimEn into a new dataset (using the intersection of country names). Use only the last 10 years (2006-2015) of GDP data and only the top 15 countries by Scimagojr 'Rank' (Rank 1 through 15). 

The index of this DataFrame should be the name of the country, and the columns should be ['Rank', 'Documents', 'Citable documents', 'Citations', 'Self-citations',
       'Citations per document', 'H index', 'Energy Supply',
       'Energy Supply per Capita', '% Renewable', '2006', '2007', '2008',
       '2009', '2010', '2011', '2012', '2013', '2014', '2015'].

*This function should return a DataFrame with 20 columns and 15 entries.*


```python
import pandas as pd
import numpy as np
import re
```


```python
def energy():
    energy = pd.read_excel('Energy Indicators.xls', header = None, skip_footer = 38, \
                           skiprows=[0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17])
    
    energy = energy.drop([0, 1], axis = 1)
    
    energy = energy.rename(columns={2:"Country", 3:'Energy Supply', \
                                   4:'Energy Supply per Capita', 5:'% Renewable'})
    
    energy.replace('...', np.nan, inplace = True)
    
    energy['Energy Supply'] *= 1000000

    change_countries = {"Republic of Korea": "South Korea",
    "United States of America": "United States",
    "United Kingdom of Great Britain and Northern Ireland": "United Kingdom",
    "China, Hong Kong Special Administrative Region": "Hong Kong"}
    
    def clean(row):
        if re.search(r'[0-9]+', row):
            row = re.sub(r'[0-9]', '', row)
        if row.find("(") != -1:
            row = row[:row.find("(")]
        return row.strip()
    
    energy['Country'] = energy['Country'].apply(clean)
    energy = energy.replace({'Country': change_countries})
            
    return energy
```


```python
def gdp():
    GDP = pd.read_csv('world_bank.csv', skiprows=[0,1,2,3])

    GDP = GDP[['Country Name', '2006', '2007', '2008', '2009', '2010', \
               '2011', '2012', '2013', '2014', '2015']]

    GDP = GDP.rename(columns={'Country Name':"Country"})

    change_names = {"Korea, Rep.": "South Korea", 
    "Iran, Islamic Rep.": "Iran",
    "Hong Kong SAR, China": "Hong Kong"}
    
    GDP.replace({"Country": change_names}, inplace = True)
    
    return GDP
```


```python
def scimen():
    ScimEn = pd.read_excel('scimagojr-3.xlsx')
    return ScimEn
```


```python
def answer_one():
    
    energy1, GDP1, ScimEn1 = energy(),gdp(),scimen()
    
    tdf = pd.merge(energy1, GDP1, how = 'inner', on = 'Country')
    tdf = pd.merge(tdf, ScimEn1, how = 'inner', on = 'Country')
    tdf.set_index('Country', inplace = True)
    top15 = tdf['Rank'].isin([1,2,3,4,5,6,7,8,9,10,11,12,13,14,15])
    tdf = tdf.loc[top15]
    tdf = tdf[['Rank', 'Documents', 'Citable documents', 'Citations', 'Self-citations',\
               'Citations per document', 'H index', 'Energy Supply', 'Energy Supply per Capita', '% Renewable', \
               '2006', '2007', '2008', '2009', '2010', '2011', '2012', '2013', '2014', '2015']]
    tdf.sort_values('Rank', inplace = True)
    
    return tdf

answer_one()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Rank</th>
      <th>Documents</th>
      <th>Citable documents</th>
      <th>Citations</th>
      <th>Self-citations</th>
      <th>Citations per document</th>
      <th>H index</th>
      <th>Energy Supply</th>
      <th>Energy Supply per Capita</th>
      <th>% Renewable</th>
      <th>2006</th>
      <th>2007</th>
      <th>2008</th>
      <th>2009</th>
      <th>2010</th>
      <th>2011</th>
      <th>2012</th>
      <th>2013</th>
      <th>2014</th>
      <th>2015</th>
    </tr>
    <tr>
      <th>Country</th>
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
      <th>China</th>
      <td>1</td>
      <td>127050</td>
      <td>126767</td>
      <td>597237</td>
      <td>411683</td>
      <td>4.70</td>
      <td>138</td>
      <td>1.271910e+11</td>
      <td>93.0</td>
      <td>19.754910</td>
      <td>3.992331e+12</td>
      <td>4.559041e+12</td>
      <td>4.997775e+12</td>
      <td>5.459247e+12</td>
      <td>6.039659e+12</td>
      <td>6.612490e+12</td>
      <td>7.124978e+12</td>
      <td>7.672448e+12</td>
      <td>8.230121e+12</td>
      <td>8.797999e+12</td>
    </tr>
    <tr>
      <th>United States</th>
      <td>2</td>
      <td>96661</td>
      <td>94747</td>
      <td>792274</td>
      <td>265436</td>
      <td>8.20</td>
      <td>230</td>
      <td>9.083800e+10</td>
      <td>286.0</td>
      <td>11.570980</td>
      <td>1.479230e+13</td>
      <td>1.505540e+13</td>
      <td>1.501149e+13</td>
      <td>1.459484e+13</td>
      <td>1.496437e+13</td>
      <td>1.520402e+13</td>
      <td>1.554216e+13</td>
      <td>1.577367e+13</td>
      <td>1.615662e+13</td>
      <td>1.654857e+13</td>
    </tr>
    <tr>
      <th>Japan</th>
      <td>3</td>
      <td>30504</td>
      <td>30287</td>
      <td>223024</td>
      <td>61554</td>
      <td>7.31</td>
      <td>134</td>
      <td>1.898400e+10</td>
      <td>149.0</td>
      <td>10.232820</td>
      <td>5.496542e+12</td>
      <td>5.617036e+12</td>
      <td>5.558527e+12</td>
      <td>5.251308e+12</td>
      <td>5.498718e+12</td>
      <td>5.473738e+12</td>
      <td>5.569102e+12</td>
      <td>5.644659e+12</td>
      <td>5.642884e+12</td>
      <td>5.669563e+12</td>
    </tr>
    <tr>
      <th>United Kingdom</th>
      <td>4</td>
      <td>20944</td>
      <td>20357</td>
      <td>206091</td>
      <td>37874</td>
      <td>9.84</td>
      <td>139</td>
      <td>7.920000e+09</td>
      <td>124.0</td>
      <td>10.600470</td>
      <td>2.419631e+12</td>
      <td>2.482203e+12</td>
      <td>2.470614e+12</td>
      <td>2.367048e+12</td>
      <td>2.403504e+12</td>
      <td>2.450911e+12</td>
      <td>2.479809e+12</td>
      <td>2.533370e+12</td>
      <td>2.605643e+12</td>
      <td>2.666333e+12</td>
    </tr>
    <tr>
      <th>Russian Federation</th>
      <td>5</td>
      <td>18534</td>
      <td>18301</td>
      <td>34266</td>
      <td>12422</td>
      <td>1.85</td>
      <td>57</td>
      <td>3.070900e+10</td>
      <td>214.0</td>
      <td>17.288680</td>
      <td>1.385793e+12</td>
      <td>1.504071e+12</td>
      <td>1.583004e+12</td>
      <td>1.459199e+12</td>
      <td>1.524917e+12</td>
      <td>1.589943e+12</td>
      <td>1.645876e+12</td>
      <td>1.666934e+12</td>
      <td>1.678709e+12</td>
      <td>1.616149e+12</td>
    </tr>
    <tr>
      <th>Canada</th>
      <td>6</td>
      <td>17899</td>
      <td>17620</td>
      <td>215003</td>
      <td>40930</td>
      <td>12.01</td>
      <td>149</td>
      <td>1.043100e+10</td>
      <td>296.0</td>
      <td>61.945430</td>
      <td>1.564469e+12</td>
      <td>1.596740e+12</td>
      <td>1.612713e+12</td>
      <td>1.565145e+12</td>
      <td>1.613406e+12</td>
      <td>1.664087e+12</td>
      <td>1.693133e+12</td>
      <td>1.730688e+12</td>
      <td>1.773486e+12</td>
      <td>1.792609e+12</td>
    </tr>
    <tr>
      <th>Germany</th>
      <td>7</td>
      <td>17027</td>
      <td>16831</td>
      <td>140566</td>
      <td>27426</td>
      <td>8.26</td>
      <td>126</td>
      <td>1.326100e+10</td>
      <td>165.0</td>
      <td>17.901530</td>
      <td>3.332891e+12</td>
      <td>3.441561e+12</td>
      <td>3.478809e+12</td>
      <td>3.283340e+12</td>
      <td>3.417298e+12</td>
      <td>3.542371e+12</td>
      <td>3.556724e+12</td>
      <td>3.567317e+12</td>
      <td>3.624386e+12</td>
      <td>3.685556e+12</td>
    </tr>
    <tr>
      <th>India</th>
      <td>8</td>
      <td>15005</td>
      <td>14841</td>
      <td>128763</td>
      <td>37209</td>
      <td>8.58</td>
      <td>115</td>
      <td>3.319500e+10</td>
      <td>26.0</td>
      <td>14.969080</td>
      <td>1.265894e+12</td>
      <td>1.374865e+12</td>
      <td>1.428361e+12</td>
      <td>1.549483e+12</td>
      <td>1.708459e+12</td>
      <td>1.821872e+12</td>
      <td>1.924235e+12</td>
      <td>2.051982e+12</td>
      <td>2.200617e+12</td>
      <td>2.367206e+12</td>
    </tr>
    <tr>
      <th>France</th>
      <td>9</td>
      <td>13153</td>
      <td>12973</td>
      <td>130632</td>
      <td>28601</td>
      <td>9.93</td>
      <td>114</td>
      <td>1.059700e+10</td>
      <td>166.0</td>
      <td>17.020280</td>
      <td>2.607840e+12</td>
      <td>2.669424e+12</td>
      <td>2.674637e+12</td>
      <td>2.595967e+12</td>
      <td>2.646995e+12</td>
      <td>2.702032e+12</td>
      <td>2.706968e+12</td>
      <td>2.722567e+12</td>
      <td>2.729632e+12</td>
      <td>2.761185e+12</td>
    </tr>
    <tr>
      <th>South Korea</th>
      <td>10</td>
      <td>11983</td>
      <td>11923</td>
      <td>114675</td>
      <td>22595</td>
      <td>9.57</td>
      <td>104</td>
      <td>1.100700e+10</td>
      <td>221.0</td>
      <td>2.279353</td>
      <td>9.410199e+11</td>
      <td>9.924316e+11</td>
      <td>1.020510e+12</td>
      <td>1.027730e+12</td>
      <td>1.094499e+12</td>
      <td>1.134796e+12</td>
      <td>1.160809e+12</td>
      <td>1.194429e+12</td>
      <td>1.234340e+12</td>
      <td>1.266580e+12</td>
    </tr>
    <tr>
      <th>Italy</th>
      <td>11</td>
      <td>10964</td>
      <td>10794</td>
      <td>111850</td>
      <td>26661</td>
      <td>10.20</td>
      <td>106</td>
      <td>6.530000e+09</td>
      <td>109.0</td>
      <td>33.667230</td>
      <td>2.202170e+12</td>
      <td>2.234627e+12</td>
      <td>2.211154e+12</td>
      <td>2.089938e+12</td>
      <td>2.125185e+12</td>
      <td>2.137439e+12</td>
      <td>2.077184e+12</td>
      <td>2.040871e+12</td>
      <td>2.033868e+12</td>
      <td>2.049316e+12</td>
    </tr>
    <tr>
      <th>Spain</th>
      <td>12</td>
      <td>9428</td>
      <td>9330</td>
      <td>123336</td>
      <td>23964</td>
      <td>13.08</td>
      <td>115</td>
      <td>4.923000e+09</td>
      <td>106.0</td>
      <td>37.968590</td>
      <td>1.414823e+12</td>
      <td>1.468146e+12</td>
      <td>1.484530e+12</td>
      <td>1.431475e+12</td>
      <td>1.431673e+12</td>
      <td>1.417355e+12</td>
      <td>1.380216e+12</td>
      <td>1.357139e+12</td>
      <td>1.375605e+12</td>
      <td>1.419821e+12</td>
    </tr>
    <tr>
      <th>Iran</th>
      <td>13</td>
      <td>8896</td>
      <td>8819</td>
      <td>57470</td>
      <td>19125</td>
      <td>6.46</td>
      <td>72</td>
      <td>9.172000e+09</td>
      <td>119.0</td>
      <td>5.707721</td>
      <td>3.895523e+11</td>
      <td>4.250646e+11</td>
      <td>4.289909e+11</td>
      <td>4.389208e+11</td>
      <td>4.677902e+11</td>
      <td>4.853309e+11</td>
      <td>4.532569e+11</td>
      <td>4.445926e+11</td>
      <td>4.639027e+11</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>Australia</th>
      <td>14</td>
      <td>8831</td>
      <td>8725</td>
      <td>90765</td>
      <td>15606</td>
      <td>10.28</td>
      <td>107</td>
      <td>5.386000e+09</td>
      <td>231.0</td>
      <td>11.810810</td>
      <td>1.021939e+12</td>
      <td>1.060340e+12</td>
      <td>1.099644e+12</td>
      <td>1.119654e+12</td>
      <td>1.142251e+12</td>
      <td>1.169431e+12</td>
      <td>1.211913e+12</td>
      <td>1.241484e+12</td>
      <td>1.272520e+12</td>
      <td>1.301251e+12</td>
    </tr>
    <tr>
      <th>Brazil</th>
      <td>15</td>
      <td>8668</td>
      <td>8596</td>
      <td>60702</td>
      <td>14396</td>
      <td>7.00</td>
      <td>86</td>
      <td>1.214900e+10</td>
      <td>59.0</td>
      <td>69.648030</td>
      <td>1.845080e+12</td>
      <td>1.957118e+12</td>
      <td>2.056809e+12</td>
      <td>2.054215e+12</td>
      <td>2.208872e+12</td>
      <td>2.295245e+12</td>
      <td>2.339209e+12</td>
      <td>2.409740e+12</td>
      <td>2.412231e+12</td>
      <td>2.319423e+12</td>
    </tr>
  </tbody>
</table>
</div>



### Question 2 (6.6%)
The previous question joined three datasets then reduced this to just the top 15 entries. When you joined the datasets, but before you reduced this to the top 15 items, how many entries did you lose?

*This function should return a single number.*


```python
%%HTML
<svg width="800" height="300">
  <circle cx="150" cy="180" r="80" fill-opacity="0.2" stroke="black" stroke-width="2" fill="blue" />
  <circle cx="200" cy="100" r="80" fill-opacity="0.2" stroke="black" stroke-width="2" fill="red" />
  <circle cx="100" cy="100" r="80" fill-opacity="0.2" stroke="black" stroke-width="2" fill="green" />
  <line x1="150" y1="125" x2="300" y2="150" stroke="black" stroke-width="2" fill="black" stroke-dasharray="5,3"/>
  <text  x="300" y="165" font-family="Verdana" font-size="35">Everything but this!</text>
</svg>
```


<svg width="800" height="300">
  <circle cx="150" cy="180" r="80" fill-opacity="0.2" stroke="black" stroke-width="2" fill="blue" />
  <circle cx="200" cy="100" r="80" fill-opacity="0.2" stroke="black" stroke-width="2" fill="red" />
  <circle cx="100" cy="100" r="80" fill-opacity="0.2" stroke="black" stroke-width="2" fill="green" />
  <line x1="150" y1="125" x2="300" y2="150" stroke="black" stroke-width="2" fill="black" stroke-dasharray="5,3"/>
  <text  x="300" y="165" font-family="Verdana" font-size="35">Everything but this!</text>
</svg>



```python
def answer_two():
    
    energy1, GDP1, ScimEn1 = energy(),gdp(),scimen()
    
    tdfIntersection = pd.merge(energy1, GDP1, how = 'inner', on = 'Country')
    tdfIntersection = pd.merge(tdfIntersection, ScimEn1, how = 'inner', on = 'Country')
    
    tdfUnion = pd.merge(energy1, GDP1, how = 'outer', on = 'Country')
    tdfUnion = pd.merge(tdfUnion, ScimEn1, how = 'outer', on = 'Country')
    
    return (len(tdfUnion) - len(tdfIntersection))
```

## Answer the following questions in the context of only the top 15 countries by Scimagojr Rank (aka the DataFrame returned by `answer_one()`)

### Question 3 (6.6%)
What is the average GDP over the last 10 years for each country? (exclude missing values from this calculation.)

*This function should return a Series named `avgGDP` with 15 countries and their average GDP sorted in descending order.*


```python
def answer_three():
    Top15 = answer_one()
    Top15 = Top15[['2006', '2007', '2008', '2009', '2010', '2011', '2012', '2013', '2014', '2015']]
    avgGDP = Top15.mean(axis = 1).sort_values(ascending = False).rename("avgGDP")
    return avgGDP
```

### Question 4 (6.6%)
By how much had the GDP changed over the 10 year span for the country with the 6th largest average GDP?

*This function should return a single number.*


```python
def answer_four():
    Top15 = answer_one()
    changefrom = Top15.loc['United Kingdom']['2006']
    changeto = Top15.loc['United Kingdom']['2015']
    return (changeto - changefrom)
```

### Question 5 (6.6%)
What is the mean `Energy Supply per Capita`?

*This function should return a single number.*


```python
def answer_five():
    Top15 = answer_one()
    return Top15['Energy Supply per Capita'].mean()
```

### Question 6 (6.6%)
What country has the maximum % Renewable and what is the percentage?

*This function should return a tuple with the name of the country and the percentage.*


```python
def answer_six():
    Top15 = answer_one()
    Top15 = Top15['% Renewable']
    return Top15.idxmax(), Top15.max()
```

### Question 7 (6.6%)
Create a new column that is the ratio of Self-Citations to Total Citations. 
What is the maximum value for this new column, and what country has the highest ratio?

*This function should return a tuple with the name of the country and the ratio.*


```python
def answer_seven():
    Top15 = answer_one()
    Top15['ratio Citations'] = Top15['Self-citations']/Top15['Citations']

    return Top15['ratio Citations'].idxmax(), Top15['ratio Citations'].max() 
```

### Question 8 (6.6%)

Create a column that estimates the population using Energy Supply and Energy Supply per capita. 
What is the third most populous country according to this estimate?

*This function should return a single string value.*


```python
def answer_eight():
    Top15 = answer_one()
    Top15['pop'] = Top15['Energy Supply']/Top15['Energy Supply per Capita']
    Top15 = Top15.sort_values(by = 'pop', ascending = False)
    return Top15.iloc[2].name
```

### Question 9 (6.6%)
Create a column that estimates the number of citable documents per person. 
What is the correlation between the number of citable documents per capita and the energy supply per capita? Use the `.corr()` method, (Pearson's correlation).

*This function should return a single number.*

*(Optional: Use the built-in function `plot9()` to visualize the relationship between Energy Supply per Capita vs. Citable docs per Capita)*


```python
def answer_nine():
    Top15 = answer_one()
    Top15['PopEst'] = Top15['Energy Supply'] / Top15['Energy Supply per Capita']
    Top15['Citable docs per Capita'] = Top15['Citable documents'] / Top15['PopEst']
    return Top15['Citable docs per Capita'].corr(Top15['Energy Supply per Capita'])

```


```python
def plot9():
    import matplotlib as plt
    %matplotlib inline
    
    Top15 = answer_one()
    Top15['PopEst'] = Top15['Energy Supply'] / Top15['Energy Supply per Capita']
    Top15['Citable docs per Capita'] = Top15['Citable documents'] / Top15['PopEst']
    Top15.plot(x='Citable docs per Capita', y='Energy Supply per Capita', kind='scatter', xlim=[0, 0.0006])
```


```python
#plot9() # Be sure to comment out plot9() before submitting the assignment!
```

### Question 10 (6.6%)
Create a new column with a 1 if the country's % Renewable value is at or above the median for all countries in the top 15, and a 0 if the country's % Renewable value is below the median.

*This function should return a series named `HighRenew` whose index is the country name sorted in ascending order of rank.*


```python
def answer_ten():
    Top15 = answer_one()
    median = Top15['% Renewable'].median()
    def one(row):
        if row >= median:
            return 1
        else:
            return 0
            
    Top15['HighRenew'] = Top15['% Renewable'].apply(one)        
        
    return Top15['HighRenew'].sort_values()
```

### Question 11 (6.6%)
Use the following dictionary to group the Countries by Continent, then create a dateframe that displays the sample size (the number of countries in each continent bin), and the sum, mean, and std deviation for the estimated population of each country.

```python
ContinentDict  = {'China':'Asia', 
                  'United States':'North America', 
                  'Japan':'Asia', 
                  'United Kingdom':'Europe', 
                  'Russian Federation':'Europe', 
                  'Canada':'North America', 
                  'Germany':'Europe', 
                  'India':'Asia',
                  'France':'Europe', 
                  'South Korea':'Asia', 
                  'Italy':'Europe', 
                  'Spain':'Europe', 
                  'Iran':'Asia',
                  'Australia':'Australia', 
                  'Brazil':'South America'}
```

*This function should return a DataFrame with index named Continent `['Asia', 'Australia', 'Europe', 'North America', 'South America']` and columns `['size', 'sum', 'mean', 'std']`*


```python
def answer_eleven():
    Top15 = answer_one()
    ContinentDict  = {'China':'Asia', 
                  'United States':'North America', 
                  'Japan':'Asia', 
                  'United Kingdom':'Europe', 
                  'Russian Federation':'Europe', 
                  'Canada':'North America', 
                  'Germany':'Europe', 
                  'India':'Asia',
                  'France':'Europe', 
                  'South Korea':'Asia', 
                  'Italy':'Europe', 
                  'Spain':'Europe', 
                  'Iran':'Asia',
                  'Australia':'Australia', 
                  'Brazil':'South America'}
    
    def set_continent(row):
        if row in ContinentDict:
            return ContinentDict[row]
            
    
    Top15['pop'] = Top15['Energy Supply']/Top15['Energy Supply per Capita']
    Top15.reset_index(inplace = True)
    Top15['Continent'] = Top15['Country'].apply(set_continent)
    group = Top15.set_index('Continent').groupby(level = 0)['pop'].agg({'size': np.size, 'sum': np.sum, 'mean': np.mean, 'std': np.std})
    return group
```

### Question 12 (6.6%)
Cut % Renewable into 5 bins. Group Top15 by the Continent, as well as these new % Renewable bins. How many countries are in each of these groups?

*This function should return a __Series__ with a MultiIndex of `Continent`, then the bins for `% Renewable`. Do not include groups with no countries.*


```python
def answer_twelve():
    Top15 = answer_one()
    ContinentDict  = {'China':'Asia', 
                  'United States':'North America', 
                  'Japan':'Asia', 
                  'United Kingdom':'Europe', 
                  'Russian Federation':'Europe', 
                  'Canada':'North America', 
                  'Germany':'Europe', 
                  'India':'Asia',
                  'France':'Europe', 
                  'South Korea':'Asia', 
                  'Italy':'Europe', 
                  'Spain':'Europe', 
                  'Iran':'Asia',
                  'Australia':'Australia', 
                  'Brazil':'South America'}
    
    def set_continent(row):
        if row in ContinentDict:
            return ContinentDict[row]
            
    Top15.reset_index(inplace = True)
    Top15['Continent'] = Top15['Country'].apply(set_continent)
    
    
    Top15['bin'] = pd.cut(Top15['% Renewable'], 5)
    Top15.groupby(['Continent', 'bin']).size()
    
    return Top15.groupby(['Continent', 'bin']).size()
```

### Question 13 (6.6%)
Convert the Population Estimate series to a string with thousands separator (using commas). Do not round the results.

e.g. 317615384.61538464 -> 317,615,384.61538464

*This function should return a Series `PopEst` whose index is the country name and whose values are the population estimate string.*


```python
def answer_thirteen():
    Top15 = answer_one()
    Top15['popEst'] = Top15['Energy Supply']/Top15['Energy Supply per Capita']
    
    def convert(row):
        y = str(row).split('.')
        counter = 1
        string = ""
        for i in reversed(y[0]):
            if counter == 3:
                string += i + ','
                counter = 0
            else:
                string += i
            counter += 1
        final = string[::-1] + '.' + y[1]
        if final[0] == ',':
            return final[1:]
        return final
    
    popEst = Top15['popEst'].apply(convert)
    
    return popEst
```

### Optional

Use the built in function `plot_optional()` to see an example visualization.


```python
def plot_optional():
    import matplotlib as plt
    %matplotlib inline
    Top15 = answer_one()
    ax = Top15.plot(x='Rank', y='% Renewable', kind='scatter', 
                    c=['#e41a1c','#377eb8','#e41a1c','#4daf4a','#4daf4a','#377eb8','#4daf4a','#e41a1c',
                       '#4daf4a','#e41a1c','#4daf4a','#4daf4a','#e41a1c','#dede00','#ff7f00'], 
                    xticks=range(1,16), s=6*Top15['2014']/10**10, alpha=.75, figsize=[16,6]);

    for i, txt in enumerate(Top15.index):
        ax.annotate(txt, [Top15['Rank'][i], Top15['% Renewable'][i]], ha='center')

    print("This is an example of a visualization that can be created to help understand the data. \
This is a bubble chart showing % Renewable vs. Rank. The size of the bubble corresponds to the countries' \
2014 GDP, and the color corresponds to the continent.")
```
