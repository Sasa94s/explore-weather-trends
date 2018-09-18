
# Explore Weather Trends
## Summary
Analyzing local and global temperature data and compare the temperature trends in Alexandria (EG) to overall global temperature trends.

## Importing Libraries


```python
from sqlalchemy import create_engine
import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd
import numpy as np
import psycopg2
```

## Jupyter Magic Settings


```python
%matplotlib inline
# Control the default size of figures
%pylab inline
pylab.rcParams['figure.figsize'] = (20, 9)   # Change the size of plots
```

    Populating the interactive namespace from numpy and matplotlib
    

## Database connection
Please read `README.md` file for more info about database setup.


```python
postgres_engine = create_engine('postgresql+psycopg2://postgres:100100@localhost:5432/weather')
```


```python
%reload_ext sql_magic
%config SQL.conn_name = 'postgres_engine'
```



## Getting Started
Exploring and getting familiar with the data.


```python
%%read_sql
SELECT *
FROM city_data
LIMIT 5;
```

    Query started at 06:24:46 AM Egypt Standard Time; Query executed in 0.01 m




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
      <th>year</th>
      <th>city</th>
      <th>country</th>
      <th>avg_temp</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1849</td>
      <td>Abidjan</td>
      <td>Côte D'Ivoire</td>
      <td>25.58</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1850</td>
      <td>Abidjan</td>
      <td>Côte D'Ivoire</td>
      <td>25.52</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1851</td>
      <td>Abidjan</td>
      <td>Côte D'Ivoire</td>
      <td>25.67</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1852</td>
      <td>Abidjan</td>
      <td>Côte D'Ivoire</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1853</td>
      <td>Abidjan</td>
      <td>Côte D'Ivoire</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
%%read_sql
SELECT *
FROM global_data
LIMIT 5;
```

    Query started at 06:24:47 AM Egypt Standard Time; Query executed in 0.00 m




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
      <th>year</th>
      <th>avg_temp</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1750</td>
      <td>8.72</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1751</td>
      <td>7.98</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1752</td>
      <td>5.78</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1753</td>
      <td>8.39</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1754</td>
      <td>8.47</td>
    </tr>
  </tbody>
</table>
</div>



Hiding output result when executing SQL queries.


```python
%config SQL.output_result = False
```

### Grabbing all data
Grabbing all data for global temperature and cities temperature for each year till 2015.


```python
%%read_sql data
SELECT COALESCE(l.year,g.year) AS year, l.avg_temp AS local_temp, g.avg_temp AS global_temp
FROM city_data AS l
FULL OUTER JOIN global_data AS g
ON l.year = g.year;
```

    Query started at 06:24:47 AM Egypt Standard Time; Query executed in 0.01 m


```python
data.tail()
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
      <th>year</th>
      <th>local_temp</th>
      <th>global_temp</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>71308</th>
      <td>2011</td>
      <td>21.55</td>
      <td>9.52</td>
    </tr>
    <tr>
      <th>71309</th>
      <td>2012</td>
      <td>21.52</td>
      <td>9.51</td>
    </tr>
    <tr>
      <th>71310</th>
      <td>2013</td>
      <td>22.19</td>
      <td>9.61</td>
    </tr>
    <tr>
      <th>71311</th>
      <td>2014</td>
      <td>NaN</td>
      <td>9.57</td>
    </tr>
    <tr>
      <th>71312</th>
      <td>2015</td>
      <td>NaN</td>
      <td>9.83</td>
    </tr>
  </tbody>
</table>
</div>



### Moving Average
Calculating Simple Moving Average (MA) for **global** temperature and **Alexandria** city temperature (local temperature) for smoothing data without losing sensible information


```python
%%read_sql alex
WITH local_data AS (SELECT *
               FROM city_data
               WHERE city = 'Alexandria'
               AND country = 'Egypt')
SELECT TO_DATE(l.year, 'YYYY') AS year,
        l.avg_temp AS local_temp,
        g.avg_temp AS global_temp,
        ROUND(AVG(l.avg_temp) OVER (ORDER BY l.year ROWS BETWEEN 9 PRECEDING AND CURRENT ROW), 2) AS localMA10,
        ROUND(AVG(g.avg_temp) OVER (ORDER BY g.year ROWS BETWEEN 9 PRECEDING AND CURRENT ROW), 2) AS globalMA10,
        ROUND(AVG(l.avg_temp) OVER (ORDER BY l.year ROWS BETWEEN 29 PRECEDING AND CURRENT ROW), 2) AS localMA30,
        ROUND(AVG(g.avg_temp) OVER (ORDER BY g.year ROWS BETWEEN 29 PRECEDING AND CURRENT ROW), 2) AS globalMA30
FROM local_data AS l
JOIN global_data AS g
ON l.year = g.year;
```

    Query started at 06:24:48 AM Egypt Standard Time; Query executed in 0.00 m


```python
alex.tail()
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
      <th>year</th>
      <th>local_temp</th>
      <th>global_temp</th>
      <th>localma10</th>
      <th>globalma10</th>
      <th>localma30</th>
      <th>globalma30</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>218</th>
      <td>2009-01-01</td>
      <td>21.67</td>
      <td>9.51</td>
      <td>21.31</td>
      <td>9.49</td>
      <td>20.85</td>
      <td>9.19</td>
    </tr>
    <tr>
      <th>219</th>
      <td>2010-01-01</td>
      <td>22.46</td>
      <td>9.70</td>
      <td>21.48</td>
      <td>9.54</td>
      <td>20.92</td>
      <td>9.21</td>
    </tr>
    <tr>
      <th>220</th>
      <td>2011-01-01</td>
      <td>21.18</td>
      <td>9.52</td>
      <td>21.45</td>
      <td>9.55</td>
      <td>20.93</td>
      <td>9.22</td>
    </tr>
    <tr>
      <th>221</th>
      <td>2012-01-01</td>
      <td>21.55</td>
      <td>9.51</td>
      <td>21.46</td>
      <td>9.55</td>
      <td>20.98</td>
      <td>9.25</td>
    </tr>
    <tr>
      <th>222</th>
      <td>2013-01-01</td>
      <td>21.44</td>
      <td>9.61</td>
      <td>21.48</td>
      <td>9.56</td>
      <td>21.03</td>
      <td>9.27</td>
    </tr>
  </tbody>
</table>
</div>




```python
def preprocessing(df):
    """Preprocesses data by filling missing values"""
    # Forward and Backward filling missing values
    df.fillna(method='ffill', inplace=True)
    df.fillna(method='bfill', inplace=True)
```


```python
preprocessing(data)
preprocessing(alex)
```

### Visualizing Temperature Data
Plotting both **Global** average temperature and **Alexandria** average temperature.


```python
# Set default Seaborn style
sns.set()
```


```python
_ = plt.plot('year', 'local_temp', data=alex, label='Alexandria Temperature', linewidth=3)
_ = plt.plot('year', 'global_temp', data=alex, label='Global Temperature', linewidth=3)
_ = plt.legend(loc='right', fontsize='xx-large')

_ = plt.xlabel('Year', fontsize='x-large')
_ = plt.ylabel('Temperature (ºC)', fontsize='x-large')

plt.show()
```


![png](https://github.com/Sasa94s/explore-weather-trends/blob/master/visualization/output_23_0.png)


Plotting 10-day and 30-day Moving Average for both **Global** temperature and **Alexandria** temperature. You'll see they're have relatively the same volatility in both 10-day moving average and the long-term 30-day moving average.


```python
_ = plt.plot('year', 'localma30', data=alex, label='Alexandria Temperature 30-day MA', linewidth=3)
_ = plt.plot('year', 'localma10', data=alex, label='Alexandria Temperature 10-day MA', linewidth=3)
_ = plt.plot('year', 'globalma30', data=alex, label='Global Temperature 30-day MA', linewidth=3)
_ = plt.plot('year', 'globalma10', data=alex, label='Global Temperature 10-day MA', linewidth=3)
_ = plt.legend(loc='right', fontsize='xx-large')

_ = plt.xlabel('Year', fontsize='x-large')
_ = plt.ylabel('Temperature (ºC)', fontsize='x-large')

plt.show()
```


![png](https://github.com/Sasa94s/explore-weather-trends/blob/master/visualization/output_25_0.png)


In order to trace the trend and determine the type of correlation between **Global** long term moving average and **Alexandria** long term moving average, I've set a secondary y-axis for getting both line plots closer from each other.


```python
alex_ma30 = alex.set_index('year')[['localma30','globalma30']]
fig, ax = plt.subplots()
_ = alex_ma30.plot(secondary_y=['globalma30'], ax=ax, legend=False)

_ = plt.ylabel('Global Temperature (ºC)', fontsize='x-large')
ax.set_ylabel('Alexandria Temperature (ºC)', fontsize='x-large')
ax.set_xlabel('Year', fontsize='x-large')

_ = plt.legend(labels=['Global'], loc='lower right', fontsize='xx-large')
ax.legend(labels=['Alexandria'], loc='lower left', fontsize='xx-large')

plt.show()
```


![png](https://github.com/Sasa94s/explore-weather-trends/blob/master/visualization/output_27_0.png)


Both **Global** average temperature and **Alexandria** average temperature have the same shape of distribution; as they're approximately normally distributed.


```python
fig, ax = plt.subplots(1,2)
ax[0].hist(alex.set_index('year')['local_temp'], bins=6, label='Alexandria Temp')
ax[0].set_xlabel('Alexandria Temperature (ºC)', fontsize='x-large')

ax[1].hist(alex.set_index('year')['global_temp'], color='g', bins=6, label='Global Temperature')
ax[1].set_xlabel('Global Temperature (ºC)', fontsize='x-large')

plt.show()
```


![png](https://github.com/Sasa94s/explore-weather-trends/blob/master/visualization/output_29_0.png)


#### Correlation Coefficient
There is a positive correlation between **Global** average temperature and **Alexandria** average temperature.


```python
# Correlation Matrix Heatmap
fig, ax = plt.subplots()
corr = alex[['local_temp','global_temp']]
corr.columns = ['Alexandria', 'Global']
corr = corr.corr()

hm = sns.heatmap(round(corr,2), annot=True, ax=ax, cmap="coolwarm",fmt='.2f',
                 linewidths=.05)
fig.subplots_adjust(top=0.93)
t= fig.suptitle('Temperature Correlation Heatmap', fontsize=14)
```


![png](https://github.com/Sasa94s/explore-weather-trends/blob/master/visualization/output_31_0.png)

