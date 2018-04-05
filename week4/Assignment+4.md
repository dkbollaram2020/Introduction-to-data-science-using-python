
---

_You are currently looking at **version 1.1** of this notebook. To download notebooks and datafiles, as well as get help on Jupyter notebooks in the Coursera platform, visit the [Jupyter Notebook FAQ](https://www.coursera.org/learn/python-data-analysis/resources/0dhYG) course resource._

---


```python
import pandas as pd
import numpy as np
from scipy.stats import ttest_ind
```

# Assignment 4 - Hypothesis Testing
This assignment requires more individual learning than previous assignments - you are encouraged to check out the [pandas documentation](http://pandas.pydata.org/pandas-docs/stable/) to find functions or methods you might not have used yet, or ask questions on [Stack Overflow](http://stackoverflow.com/) and tag them as pandas and python related. And of course, the discussion forums are open for interaction with your peers and the course staff.

Definitions:
* A _quarter_ is a specific three month period, Q1 is January through March, Q2 is April through June, Q3 is July through September, Q4 is October through December.
* A _recession_ is defined as starting with two consecutive quarters of GDP decline, and ending with two consecutive quarters of GDP growth.
* A _recession bottom_ is the quarter within a recession which had the lowest GDP.
* A _university town_ is a city which has a high percentage of university students compared to the total population of the city.

**Hypothesis**: University towns have their mean housing prices less effected by recessions. Run a t-test to compare the ratio of the mean price of houses in university towns the quarter before the recession starts compared to the recession bottom. (`price_ratio=quarter_before_recession/recession_bottom`)

The following data files are available for this assignment:
* From the [Zillow research data site](http://www.zillow.com/research/data/) there is housing data for the United States. In particular the datafile for [all homes at a city level](http://files.zillowstatic.com/research/public/City/City_Zhvi_AllHomes.csv), ```City_Zhvi_AllHomes.csv```, has median home sale prices at a fine grained level.
* From the Wikipedia page on college towns is a list of [university towns in the United States](https://en.wikipedia.org/wiki/List_of_college_towns#College_towns_in_the_United_States) which has been copy and pasted into the file ```university_towns.txt```.
* From Bureau of Economic Analysis, US Department of Commerce, the [GDP over time](http://www.bea.gov/national/index.htm#gdp) of the United States in current dollars (use the chained value in 2009 dollars), in quarterly intervals, in the file ```gdplev.xls```. For this assignment, only look at GDP data from the first quarter of 2000 onward.

Each function in this assignment below is worth 10%, with the exception of ```run_ttest()```, which is worth 50%.


```python
# Use this dictionary to map state names to two letter acronyms
states = {'OH': 'Ohio', 'KY': 'Kentucky', 'AS': 'American Samoa', 'NV': 'Nevada', 'WY': 'Wyoming', 'NA': 'National', 'AL': 'Alabama', 'MD': 'Maryland', 'AK': 'Alaska', 'UT': 'Utah', 'OR': 'Oregon', 'MT': 'Montana', 'IL': 'Illinois', 'TN': 'Tennessee', 'DC': 'District of Columbia', 'VT': 'Vermont', 'ID': 'Idaho', 'AR': 'Arkansas', 'ME': 'Maine', 'WA': 'Washington', 'HI': 'Hawaii', 'WI': 'Wisconsin', 'MI': 'Michigan', 'IN': 'Indiana', 'NJ': 'New Jersey', 'AZ': 'Arizona', 'GU': 'Guam', 'MS': 'Mississippi', 'PR': 'Puerto Rico', 'NC': 'North Carolina', 'TX': 'Texas', 'SD': 'South Dakota', 'MP': 'Northern Mariana Islands', 'IA': 'Iowa', 'MO': 'Missouri', 'CT': 'Connecticut', 'WV': 'West Virginia', 'SC': 'South Carolina', 'LA': 'Louisiana', 'KS': 'Kansas', 'NY': 'New York', 'NE': 'Nebraska', 'OK': 'Oklahoma', 'FL': 'Florida', 'CA': 'California', 'CO': 'Colorado', 'PA': 'Pennsylvania', 'DE': 'Delaware', 'NM': 'New Mexico', 'RI': 'Rhode Island', 'MN': 'Minnesota', 'VI': 'Virgin Islands', 'NH': 'New Hampshire', 'MA': 'Massachusetts', 'GA': 'Georgia', 'ND': 'North Dakota', 'VA': 'Virginia'}
```


```python
def get_list_of_university_towns():
    '''Returns a DataFrame of towns and the states they are in from the 
    university_towns.txt list. The format of the DataFrame should be:
    DataFrame( [ ["Michigan", "Ann Arbor"], ["Michigan", "Yipsilanti"] ], 
    columns=["State", "RegionName"]  )
    
    The following cleaning needs to be done:

    1. For "State", removing characters from "[" to the end.
    2. For "RegionName", when applicable, removing every character from " (" to the end.
    3. Depending on how you read the data, you may need to remove newline character '\n'. '''
    
    univTowns = open("university_towns.txt")
    lst = []
    state = ""
    region = ""
    for line in univTowns:
        if line.find("[ed") != -1:
            state = (line[:line.find("[")]).strip()
        elif line.find("(") != -1:
            region = (line[:line.find("(")]).strip()
            lst.append([state, region])
        else:
            lst.append([state, line.strip()])
    
    df = pd.DataFrame(lst, columns=["State", "RegionName"])
    
    return df

get_list_of_university_towns()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>State</th>
      <th>RegionName</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Alabama</td>
      <td>Auburn</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Alabama</td>
      <td>Florence</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Alabama</td>
      <td>Jacksonville</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Alabama</td>
      <td>Livingston</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Alabama</td>
      <td>Montevallo</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Alabama</td>
      <td>Troy</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Alabama</td>
      <td>Tuscaloosa</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Alabama</td>
      <td>Tuskegee</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Alaska</td>
      <td>Fairbanks</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Arizona</td>
      <td>Flagstaff</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Arizona</td>
      <td>Tempe</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Arizona</td>
      <td>Tucson</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Arkansas</td>
      <td>Arkadelphia</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Arkansas</td>
      <td>Conway</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Arkansas</td>
      <td>Fayetteville</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Arkansas</td>
      <td>Jonesboro</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Arkansas</td>
      <td>Magnolia</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Arkansas</td>
      <td>Monticello</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Arkansas</td>
      <td>Russellville</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Arkansas</td>
      <td>Searcy</td>
    </tr>
    <tr>
      <th>20</th>
      <td>California</td>
      <td>Angwin</td>
    </tr>
    <tr>
      <th>21</th>
      <td>California</td>
      <td>Arcata</td>
    </tr>
    <tr>
      <th>22</th>
      <td>California</td>
      <td>Berkeley</td>
    </tr>
    <tr>
      <th>23</th>
      <td>California</td>
      <td>Chico</td>
    </tr>
    <tr>
      <th>24</th>
      <td>California</td>
      <td>Claremont</td>
    </tr>
    <tr>
      <th>25</th>
      <td>California</td>
      <td>Cotati</td>
    </tr>
    <tr>
      <th>26</th>
      <td>California</td>
      <td>Davis</td>
    </tr>
    <tr>
      <th>27</th>
      <td>California</td>
      <td>Irvine</td>
    </tr>
    <tr>
      <th>28</th>
      <td>California</td>
      <td>Isla Vista</td>
    </tr>
    <tr>
      <th>29</th>
      <td>California</td>
      <td>University Park, Los Angeles</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>487</th>
      <td>Virginia</td>
      <td>Wise</td>
    </tr>
    <tr>
      <th>488</th>
      <td>Virginia</td>
      <td>Chesapeake</td>
    </tr>
    <tr>
      <th>489</th>
      <td>Washington</td>
      <td>Bellingham</td>
    </tr>
    <tr>
      <th>490</th>
      <td>Washington</td>
      <td>Cheney</td>
    </tr>
    <tr>
      <th>491</th>
      <td>Washington</td>
      <td>Ellensburg</td>
    </tr>
    <tr>
      <th>492</th>
      <td>Washington</td>
      <td>Pullman</td>
    </tr>
    <tr>
      <th>493</th>
      <td>Washington</td>
      <td>University District, Seattle</td>
    </tr>
    <tr>
      <th>494</th>
      <td>West Virginia</td>
      <td>Athens</td>
    </tr>
    <tr>
      <th>495</th>
      <td>West Virginia</td>
      <td>Buckhannon</td>
    </tr>
    <tr>
      <th>496</th>
      <td>West Virginia</td>
      <td>Fairmont</td>
    </tr>
    <tr>
      <th>497</th>
      <td>West Virginia</td>
      <td>Glenville</td>
    </tr>
    <tr>
      <th>498</th>
      <td>West Virginia</td>
      <td>Huntington</td>
    </tr>
    <tr>
      <th>499</th>
      <td>West Virginia</td>
      <td>Montgomery</td>
    </tr>
    <tr>
      <th>500</th>
      <td>West Virginia</td>
      <td>Morgantown</td>
    </tr>
    <tr>
      <th>501</th>
      <td>West Virginia</td>
      <td>Shepherdstown</td>
    </tr>
    <tr>
      <th>502</th>
      <td>West Virginia</td>
      <td>West Liberty</td>
    </tr>
    <tr>
      <th>503</th>
      <td>Wisconsin</td>
      <td>Appleton</td>
    </tr>
    <tr>
      <th>504</th>
      <td>Wisconsin</td>
      <td>Eau Claire</td>
    </tr>
    <tr>
      <th>505</th>
      <td>Wisconsin</td>
      <td>Green Bay</td>
    </tr>
    <tr>
      <th>506</th>
      <td>Wisconsin</td>
      <td>La Crosse</td>
    </tr>
    <tr>
      <th>507</th>
      <td>Wisconsin</td>
      <td>Madison</td>
    </tr>
    <tr>
      <th>508</th>
      <td>Wisconsin</td>
      <td>Menomonie</td>
    </tr>
    <tr>
      <th>509</th>
      <td>Wisconsin</td>
      <td>Milwaukee</td>
    </tr>
    <tr>
      <th>510</th>
      <td>Wisconsin</td>
      <td>Oshkosh</td>
    </tr>
    <tr>
      <th>511</th>
      <td>Wisconsin</td>
      <td>Platteville</td>
    </tr>
    <tr>
      <th>512</th>
      <td>Wisconsin</td>
      <td>River Falls</td>
    </tr>
    <tr>
      <th>513</th>
      <td>Wisconsin</td>
      <td>Stevens Point</td>
    </tr>
    <tr>
      <th>514</th>
      <td>Wisconsin</td>
      <td>Waukesha</td>
    </tr>
    <tr>
      <th>515</th>
      <td>Wisconsin</td>
      <td>Whitewater</td>
    </tr>
    <tr>
      <th>516</th>
      <td>Wyoming</td>
      <td>Laramie</td>
    </tr>
  </tbody>
</table>
<p>517 rows × 2 columns</p>
</div>




```python
#importing gdp data and cleaning to find recession period 

gdp = pd.read_excel("gdplev.xls", skiprows = 5)

gdp.drop(['Unnamed: 7', 'Unnamed: 0', 'GDP in billions of current dollars','GDP in billions of current dollars.1', 'GDP in billions of chained 2009 dollars', 'Unnamed: 3'], inplace = True, axis = 1)

gdp.rename(columns = {'Unnamed: 4': 'year quater', 'GDP in billions of chained 2009 dollars.1': 'GDP in billions of chained 2009 dollars'}, inplace = True)

gdp = gdp.iloc[214:]

gdplist = gdp['GDP in billions of chained 2009 dollars'].tolist()


change = []
for i in range(len(gdplist)):
    if i == 0:
        change.append(np.nan)
        continue
    elif i == (len(gdplist)):
        change.append(np.nan)
        break
    else:
        if (gdplist[i] - gdplist[i-1]) > 0:
            change.append(1)
        else:
            change.append(-1)
                        
                
gdp['change'] = pd.Series(change).values        

recession = []
counterDecrease = 0
counterIncrease = 0

counter = 0
decrease_trigger = 0
for first, second in zip(change, change[1:]):
    if (first == -1 and second == -1) and decrease_trigger == 0:
        recession.append([counter, "Decrease"])
        decrease_trigger += 1
    elif (first == 1 and second == 1) and decrease_trigger > 0:
        recession.append([counter, "Increase"])
        decrease_trigger = 0
    counter += 1    
    
gdp.reset_index(inplace=True)

gdp.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>year quater</th>
      <th>GDP in billions of chained 2009 dollars</th>
      <th>change</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>214</td>
      <td>2000q1</td>
      <td>12359.1</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>215</td>
      <td>2000q2</td>
      <td>12592.5</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>216</td>
      <td>2000q3</td>
      <td>12607.7</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>217</td>
      <td>2000q4</td>
      <td>12679.3</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>218</td>
      <td>2001q1</td>
      <td>12643.3</td>
      <td>-1.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
def get_recession_start():
    '''Returns the year and quarter of the recession start time as a 
    string value in a format such as 2005q3'''
    
    
    return gdp.iloc[recession[0][0]]['year quater']

get_recession_start()
```




    '2008q3'




```python
def get_recession_end():
    '''Returns the year and quarter of the recession end time as a 
    string value in a format such as 2005q3'''
       
    return gdp.iloc[(recession[1][0]) + 1]['year quater']

get_recession_end()
```




    '2009q4'




```python
def get_recession_bottom():
    '''Returns the year and quarter of the recession bottom time as a 
    string value in a format such as 2005q3'''
    
    return gdp.iloc[(recession[1][0]) - 1]['year quater']

get_recession_bottom()
```




    '2009q2'




```python
# created for fun

quater = []
quaters = ['Q1', 'Q2', 'Q3', 'Q4']
year = 2000
yearCounter = 0
quaterCounter = 1
for i in range(200):
    if yearCounter < 12:
        if quaterCounter < 4:
            quater.append(str(year) + 'Q1')
            quaterCounter += 1
        elif quaterCounter > 3 and quaterCounter < 7:
            quater.append(str(year) + 'Q2')
            quaterCounter+= 1
        elif quaterCounter > 6 and quaterCounter < 10:
            quater.append(str(year) + 'Q3')
            quaterCounter += 1
        elif quaterCounter > 9 and quaterCounter < 13:
            quater.append(str(year) + 'Q4')
            quaterCounter += 1
        if quaterCounter == 13:
            quaterCounter = 1
        yearCounter += 1
        if yearCounter == 12:
            year += 1
            yearCounter = 0          
    
```


```python
def convert_housing_data_to_quarters():
    '''Converts the housing data to quarters and returns it as mean 
    values in a dataframe. This dataframe should be a dataframe with
    columns for 2000q1 through 2016q3, and should have a multi-index
    in the shape of ["State","RegionName"].
    
    Note: Quarters are defined in the assignment description, they are
    not arbitrary three month periods.
    
    The resulting dataframe should have 67 columns, and 10,730 rows.
    '''
    housing = pd.read_csv('City_Zhvi_AllHomes.csv')
    housing.replace({"State": states}, inplace = True)
    housing.set_index(['State','RegionName'], inplace = True)
    
    housing = housing.loc[:,'2000-01':'2016-08']
    
    housing.columns = pd.to_datetime(housing.columns)
    housing = housing.resample('Q',axis=1).mean()
    housing = housing.rename(columns=lambda x: str(x.to_period('Q')).lower())
    
    
    return housing
```


```python
def run_ttest():
    '''First creates new data showing the decline or growth of housing prices
    between the recession start and the recession bottom. Then runs a ttest
    comparing the university town values to the non-university towns values, 
    return whether the alternative hypothesis (that the two groups are the same)
    is true or not as well as the p-value of the confidence. 
    
    Return the tuple (different, p, better) where different=True if the t-test is
    True at a p<0.01 (we reject the null hypothesis), or different=False if 
    otherwise (we cannot reject the null hypothesis). The variable p should
    be equal to the exact p value returned from scipy.stats.ttest_ind(). The
    value for better should be either "university town" or "non-university town"
    depending on which has a lower mean price ratio (which is equivilent to a
    reduced market loss).'''
    
    # importing data from other functons
    hdf = convert_housing_data_to_quarters()
    rec_start = gdp.iloc[recession[0][0] - 1]['year quater']
    rec_bottom = get_recession_bottom()
    ul = get_list_of_university_towns()
    
    # creating price ration column
    hdf['PriceRatio'] = hdf[rec_start].div(hdf[rec_bottom])
    
    df = pd.merge(hdf.reset_index(), ul, how = 'outer', on = ul.columns.tolist(), indicator = '_flag')
    
    # two groups
    universityTowns = df[df['_flag'] == 'both']
    nonUniversityTowns = df[df['_flag'] != 'both']
    
    
    # t- test
    t_statistic, p_value = ttest_ind(universityTowns['PriceRatio'], nonUniversityTowns['PriceRatio'], nan_policy='omit')
    
    def better(univ, nonUniv):
        if univ < nonUniv:
            return 'university town'
        else:
            return 'non-university town'
    
    different = True
    if p_value > 0.01:
        different = False
    return different, p_value, better((universityTowns['PriceRatio'].mean()), (nonUniversityTowns['PriceRatio'].mean()))

run_ttest()
```




    (True, 0.0027240637047531249, 'university town')




```python

```
