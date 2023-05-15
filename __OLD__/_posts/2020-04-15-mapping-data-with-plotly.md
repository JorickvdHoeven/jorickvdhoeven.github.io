# Plotting data on a map with plotly

Before you start make sure that you have run the following commands in anaconda prompt:

```bash
conda activate analysis-101

conda install git

conda install pycountry

git clone https://github.com/CSSEGISandData/COVID-19.git us_jhu_data
```

### Explore environment and downloaded data

As with any downloaded data, it's prudent to explore a little what data we received and wehther there are obvious patterns.


Let's start by exploring what we see from our notebook. We can do this using the jupyter magic command %ls which behaves like the Unix ls command or similar to the Windows DIR command listing directories from where the notebook is opened from.


```python
%ls 
```

    mapping-data-with-plotly.ipynb
    us_jhu_data/


So it looks like we're in our own folder, we've got our notebook and the data we just downloaded, let's have a look inside.


```python
%ls us_jhu_data
```

    README.md                       csse_covid_19_data/
    archived_data/                 who_covid_19_situation_reports/


That command wasn't super helpful, all we got back from it was a list of more folders to look into. Let's just go nuclear and look at everything together. To do that we can use the -R switch to expand all the files and folders.


```python
%ls -R us_jhu_data/ 
# %ls /S us_jhu_data # use this for windows
```

    README.md                       csse_covid_19_data/
    archived_data/                  who_covid_19_situation_reports/
    
    us_jhu_data//archived_data:
    README.md                       archived_time_series/
    archived_daily_case_updates/
    
    us_jhu_data//archived_data/archived_daily_case_updates:
    01-21-2020_2200.csv  01-29-2020_1430.csv  ...
    
    us_jhu_data//archived_data/archived_time_series:
    README.md
    time_series_19-covid-Confirmed_archived_0325.csv
    ...
    
    us_jhu_data//csse_covid_19_data:
    README.md                       csse_covid_19_daily_reports_us/
    ...
    
    us_jhu_data//csse_covid_19_data/csse_covid_19_daily_reports:
    01-22-2020.csv  02-09-2020.csv  02-27-2020.csv  ...
    
    us_jhu_data//csse_covid_19_data/csse_covid_19_daily_reports_us:
    04-12-2020.csv  04-13-2020.csv  04-14-2020.csv  ...
    
    us_jhu_data//csse_covid_19_data/csse_covid_19_time_series:
    README.md
    time_series_covid19_confirmed_US.csv
    ...
    
    us_jhu_data//who_covid_19_situation_reports:
    README.md                         who_covid_19_sit_rep_time_series/
    who_covid_19_sit_rep_pdfs/
    
    us_jhu_data//who_covid_19_situation_reports/who_covid_19_sit_rep_pdfs:
    20200121-sitrep-1-2019-ncov.pdf     20200222-sitrep-33-covid-19.pdf
    20200122-sitrep-2-2019-ncov.pdf     ...
    
    us_jhu_data//who_covid_19_situation_reports/who_covid_19_sit_rep_time_series:
    who_covid_19_sit_rep_time_series.csv


From the output of ls we can see that we have the following data sets:
- archived daily case updates
- archived time series data
- a lookup table to convert IDs to US FIPS codes (county codes)
- daily reports from jan 22 to april 10
- time series data with different focuses
- unstructured WHO situation reports
- WHO structured time series of the situation reports

For brevity, we will only explore the following two data sets:
- time series data with different focuses
- daily reports from jan 22 to april 10


Let's start with the global time series data


### Start the Python engines, import libararies

Before we start we need to load the libararies we need. Below are a collection of libraries that I reach for time and time again so it's prudent to load them first so that we don't end up needing to load them later.


```python
import pandas as pd
import numpy as np
import os
import plotly.express as px
from pandas_profiling import ProfileReport
```

#### Global time series data


```python
pd.read_csv('us_jhu_data/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_global.csv')\
  .head()

#'us_jhu_data//csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_global.csv' this file
# will be left to participants as an exercise.
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
      <th>Province/State</th>
      <th>Country/Region</th>
      <th>Lat</th>
      <th>Long</th>
      <th>1/22/20</th>
      <th>1/23/20</th>
      <th>1/24/20</th>
      <th>1/25/20</th>
      <th>1/26/20</th>
      <th>1/27/20</th>
      <th>...</th>
      <th>4/6/20</th>
      <th>4/7/20</th>
      <th>4/8/20</th>
      <th>4/9/20</th>
      <th>4/10/20</th>
      <th>4/11/20</th>
      <th>4/12/20</th>
      <th>4/13/20</th>
      <th>4/14/20</th>
      <th>4/15/20</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>Afghanistan</td>
      <td>33.0000</td>
      <td>65.0000</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>367</td>
      <td>423</td>
      <td>444</td>
      <td>484</td>
      <td>521</td>
      <td>555</td>
      <td>607</td>
      <td>665</td>
      <td>714</td>
      <td>784</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NaN</td>
      <td>Albania</td>
      <td>41.1533</td>
      <td>20.1683</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>377</td>
      <td>383</td>
      <td>400</td>
      <td>409</td>
      <td>416</td>
      <td>433</td>
      <td>446</td>
      <td>467</td>
      <td>475</td>
      <td>494</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NaN</td>
      <td>Algeria</td>
      <td>28.0339</td>
      <td>1.6596</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>1423</td>
      <td>1468</td>
      <td>1572</td>
      <td>1666</td>
      <td>1761</td>
      <td>1825</td>
      <td>1914</td>
      <td>1983</td>
      <td>2070</td>
      <td>2160</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NaN</td>
      <td>Andorra</td>
      <td>42.5063</td>
      <td>1.5218</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>525</td>
      <td>545</td>
      <td>564</td>
      <td>583</td>
      <td>601</td>
      <td>601</td>
      <td>638</td>
      <td>646</td>
      <td>659</td>
      <td>673</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NaN</td>
      <td>Angola</td>
      <td>-11.2027</td>
      <td>17.8739</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>16</td>
      <td>17</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
      <td>19</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 89 columns</p>
</div>



This file has almost exactly what we need, but it's kind of cheating, so let's use the other dataset, the daily reports and clean that data up for us to use in a mapping exercise.

### Explore chosen dataset -- Daily Reports

Let's start by peeking into one file and having a look at what's in there


```python
idata = 'us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/'
```


```python
pd.read_csv(os.path.join(idata,'03-13-2020.csv'))
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
      <th>Province/State</th>
      <th>Country/Region</th>
      <th>Last Update</th>
      <th>Confirmed</th>
      <th>Deaths</th>
      <th>Recovered</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Hubei</td>
      <td>China</td>
      <td>2020-03-13T11:09:03</td>
      <td>67786</td>
      <td>3062</td>
      <td>51553</td>
      <td>30.9756</td>
      <td>112.2707</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Guangdong</td>
      <td>China</td>
      <td>2020-03-13T11:09:03</td>
      <td>1356</td>
      <td>8</td>
      <td>1296</td>
      <td>23.3417</td>
      <td>113.4244</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Henan</td>
      <td>China</td>
      <td>2020-03-11T08:13:09</td>
      <td>1273</td>
      <td>22</td>
      <td>1249</td>
      <td>33.8820</td>
      <td>113.6140</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Zhejiang</td>
      <td>China</td>
      <td>2020-03-12T01:33:02</td>
      <td>1215</td>
      <td>1</td>
      <td>1197</td>
      <td>29.1832</td>
      <td>120.0934</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Hunan</td>
      <td>China</td>
      <td>2020-03-13T11:09:03</td>
      <td>1018</td>
      <td>4</td>
      <td>1005</td>
      <td>27.6104</td>
      <td>111.7088</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>225</th>
      <td>NaN</td>
      <td>Afghanistan</td>
      <td>2020-03-11T20:00:00</td>
      <td>7</td>
      <td>0</td>
      <td>0</td>
      <td>33.0000</td>
      <td>65.0000</td>
    </tr>
    <tr>
      <th>226</th>
      <td>NaN</td>
      <td>Monaco</td>
      <td>2020-03-11T20:00:00</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>43.7333</td>
      <td>7.4167</td>
    </tr>
    <tr>
      <th>227</th>
      <td>NaN</td>
      <td>Liechtenstein</td>
      <td>2020-03-11T20:00:00</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>47.1400</td>
      <td>9.5500</td>
    </tr>
    <tr>
      <th>228</th>
      <td>NaN</td>
      <td>Guyana</td>
      <td>2020-03-11T20:00:00</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>5.0000</td>
      <td>-58.7500</td>
    </tr>
    <tr>
      <th>229</th>
      <td>NaN</td>
      <td>Taiwan*</td>
      <td>2020-03-11T20:00:00</td>
      <td>50</td>
      <td>1</td>
      <td>20</td>
      <td>23.7000</td>
      <td>121.0000</td>
    </tr>
  </tbody>
</table>
<p>230 rows × 8 columns</p>
</div>



Check the size and column count for all the files


```python
import glob

dataset_path = os.path.join(idata,'*.csv')

for file in glob.glob(dataset_path):
    print( pd.read_csv(file).shape) # calculate data shape (rows, columns)
```

    (101, 6)
    (105, 6)
    (2883, 12)
    (2911, 12)
    ...



```python
for file in glob.glob(dataset_path):
    print(file # print file path
          , pd.read_csv(file).shape) # calculate data shape (rows, columns)
```

    us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-26-2020.csv (101, 6)
    us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-27-2020.csv (105, 6)
    us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-08-2020.csv (2883, 12)
    us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-09-2020.csv (2911, 12)
    ...



```python
for file in glob.glob(dataset_path):
    print(file.rsplit(os.path.sep,1)[1] # isolate file name
          , pd.read_csv(file).shape) # calculate data shape (rows, columns)
```

    02-26-2020.csv (101, 6)
    02-27-2020.csv (105, 6)
    04-08-2020.csv (2883, 12)
    ...



```python
# Sort the file list

for file in sorted(glob.glob(dataset_path)):
    print(file.rsplit(os.path.sep,1)[1] # isolate file name
          , pd.read_csv(file).shape) # calculate data shape (rows, columns)
```

    01-22-2020.csv (38, 6)
    01-23-2020.csv (46, 6)
    01-24-2020.csv (41, 6)
    ...



```python
import os

for file in sorted(glob.glob(dataset_path)):
       print(file.rsplit(os.path.sep,1)[1] # isolate file name
             , pd.read_csv(file).shape # calculate data shape (rows, columns)
             , os.stat(file).st_size/(1024*1024)) # get the individual file size in MB
```

    01-22-2020.csv (38, 6) 0.0015974044799804688
    ...
    03-01-2020.csv (125, 8) 0.0073528289794921875
    ...
    03-22-2020.csv (3417, 12) 0.3128471374511719
    ...


# What's the goal when preparing this dataset for vizualization?

We are going to use plotly.js to visualize the data, however that means that the data needs to be 'clean'. Basically, the data needs to be in the format and shape that is required for visualization given that plotly won't make assumptions about how to draw your data points. I've seen some beautifully picassoesque and Dr Seuss-like graphs when working with malformed data. 

In this instance we want to create a graph that show's us the number of COVID cases per country per day, and animate through the days. To do this we will need to have the data in the following format:

| Country Name | Country ISO Code | Day        | Number of Cases| 
|--------------|------------------|------------|----------------|
| Switzerland  |  CHE             | 2020-02-29 |  xxx           |
| France       |  FRA             | 2020-02-29 |  yyy           |
| Switzerland  |  CHE             | 2020-03-01 |  zzz           |
| France       |  FRA             | 2020-03-01 |  aaa           |
|   ...        |    ...           |   ...      |  ...           |

To get to that format we'll have to go from whichever format we have in the chosen data set to the format above. To be able to do that we'll need to load and harmonize the daily reports with the following steps:

1. Load the data into memory
2. Analyze the dataset and make decisions about what to do
3. Consolidate data which has changed names over time
4. Clean up the country names to identify the ISO codes for countries
5. Visualize the data on a world map
6. Tweak the visualization
7. Adapt our visualization to look at cases per 100k instead of number of cases


## 1. Load the data set into a pandas dataframe

Load all the data into memory and see how we can combine the data into a single set for us to use with data visualization.


```python
import glob 
# Create two lists to store our file metadata and file data
all_data = []
meta_data = []

# For every file in our data set path
for file in sorted(glob.glob(dataset_path)):
    
    # 1. Read the file to a temporary data frame
    df = pd.read_csv(file)
    
    # 2. Append a dictionary with the file meta_data into the metadata list
    meta_data.append( {  'file_name': file.rsplit(os.path.sep,1)[1]
                       , 'num_rows': df.shape[0]
                       , 'num_cols': df.shape[1]
                       , 'col_names': '\t'.join(sorted(list(df.columns)))} )
    
    # Add the file name to the loaded data
    df['source_file'] = file
    
    # 4. Add the loaded data with the file name to a list with all the loaded data
    all_data.append(df)

# 5. Create a table/dataframe out of our meta_data 
meta_data = pd.DataFrame(meta_data)

# show the metadata in jupyter notebook
meta_data
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
      <th>file_name</th>
      <th>num_rows</th>
      <th>num_cols</th>
      <th>col_names</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>01-22-2020.csv</td>
      <td>38</td>
      <td>6</td>
      <td>Confirmed\tCountry/Region\tDeaths\tLast Update...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>01-23-2020.csv</td>
      <td>46</td>
      <td>6</td>
      <td>Confirmed\tCountry/Region\tDeaths\tLast Update...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>01-24-2020.csv</td>
      <td>41</td>
      <td>6</td>
      <td>Confirmed\tCountry/Region\tDeaths\tLast Update...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>01-25-2020.csv</td>
      <td>44</td>
      <td>6</td>
      <td>Confirmed\tCountry/Region\tDeaths\tLast Update...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>01-26-2020.csv</td>
      <td>47</td>
      <td>6</td>
      <td>Confirmed\tCountry/Region\tDeaths\tLast Update...</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>80</th>
      <td>04-11-2020.csv</td>
      <td>2966</td>
      <td>12</td>
      <td>Active\tAdmin2\tCombined_Key\tConfirmed\tCount...</td>
    </tr>
    <tr>
      <th>81</th>
      <td>04-12-2020.csv</td>
      <td>2989</td>
      <td>12</td>
      <td>Active\tAdmin2\tCombined_Key\tConfirmed\tCount...</td>
    </tr>
    <tr>
      <th>82</th>
      <td>04-13-2020.csv</td>
      <td>3002</td>
      <td>12</td>
      <td>Active\tAdmin2\tCombined_Key\tConfirmed\tCount...</td>
    </tr>
    <tr>
      <th>83</th>
      <td>04-14-2020.csv</td>
      <td>3014</td>
      <td>12</td>
      <td>Active\tAdmin2\tCombined_Key\tConfirmed\tCount...</td>
    </tr>
    <tr>
      <th>84</th>
      <td>04-15-2020.csv</td>
      <td>3027</td>
      <td>12</td>
      <td>Active\tAdmin2\tCombined_Key\tConfirmed\tCount...</td>
    </tr>
  </tbody>
</table>
<p>85 rows × 4 columns</p>
</div>




```python

pd.set_option('max_colwidth', 150)

# output result to notebook window
meta_data.groupby(['num_cols'])\
         .agg({ 'num_rows': 'sum'
              , 'file_name': sorted
              , 'col_names': set })

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
      <th>num_rows</th>
      <th>file_name</th>
      <th>col_names</th>
    </tr>
    <tr>
      <th>num_cols</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>6</th>
      <td>2818</td>
      <td>[01-22-2020.csv, 01-23-2020.csv, 01-24-2020.csv, 01-25-2020.csv, 01-26-2020.csv, 01-27-2020.csv, 01-28-2020.csv, 01-29-2020.csv, 01-30-2020.csv, 0...</td>
      <td>{Confirmed\tCountry/Region\tDeaths\tLast Update\tProvince/State\tRecovered}</td>
    </tr>
    <tr>
      <th>8</th>
      <td>4799</td>
      <td>[03-01-2020.csv, 03-02-2020.csv, 03-03-2020.csv, 03-04-2020.csv, 03-05-2020.csv, 03-06-2020.csv, 03-07-2020.csv, 03-08-2020.csv, 03-09-2020.csv, 0...</td>
      <td>{Confirmed\tCountry/Region\tDeaths\tLast Update\tLatitude\tLongitude\tProvince/State\tRecovered}</td>
    </tr>
    <tr>
      <th>12</th>
      <td>75773</td>
      <td>[03-22-2020.csv, 03-23-2020.csv, 03-24-2020.csv, 03-25-2020.csv, 03-26-2020.csv, 03-27-2020.csv, 03-28-2020.csv, 03-29-2020.csv, 03-30-2020.csv, 0...</td>
      <td>{Active\tAdmin2\tCombined_Key\tConfirmed\tCountry_Region\tDeaths\tFIPS\tLast_Update\tLat\tLong_\tProvince_State\tRecovered}</td>
    </tr>
  </tbody>
</table>
</div>




```python
d_data = pd.concat(all_data
                  , axis='index'
                  , join='outer'
                  , ignore_index=True
                  , sort=True)

d_data.head()
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
      <th>Active</th>
      <th>Admin2</th>
      <th>Combined_Key</th>
      <th>Confirmed</th>
      <th>Country/Region</th>
      <th>Country_Region</th>
      <th>Deaths</th>
      <th>FIPS</th>
      <th>Last Update</th>
      <th>Last_Update</th>
      <th>Lat</th>
      <th>Latitude</th>
      <th>Long_</th>
      <th>Longitude</th>
      <th>Province/State</th>
      <th>Province_State</th>
      <th>Recovered</th>
      <th>source_file</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>Mainland China</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1/22/2020 17:00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Anhui</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-22-2020.csv</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>14.0</td>
      <td>Mainland China</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1/22/2020 17:00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Beijing</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-22-2020.csv</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>6.0</td>
      <td>Mainland China</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1/22/2020 17:00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Chongqing</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-22-2020.csv</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>Mainland China</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1/22/2020 17:00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Fujian</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-22-2020.csv</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Mainland China</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1/22/2020 17:00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Gansu</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-22-2020.csv</td>
    </tr>
  </tbody>
</table>
</div>



# 2. Analyze the data set 


```python
pr = ProfileReport(d_data)
```


```python
pr.to_widgets()
```

    Summarize dataset: 100%|██████████| 32/32 [00:19<00:00,  1.68it/s, Completed]
    Generate report structure: 100%|██████████| 1/1 [00:03<00:00,  3.54s/it]
    Render widgets: 100%|██████████| 1/1 [00:13<00:00, 13.76s/it]

    ![Profile Report](/assets/images/posts/mapping-data-with-plotly/profile-report.png)


# 3. Consolidate data whose names may have changed over time

Some of the data columns we looked at ealier looked like they were renamed, let's verify that for each pair and then merge the columns. 


```python

d_data[ 
    (~pd.isnull(d_data['Country/Region'])) # select rows where Country/Region is null
   &                                       # AND
    (~pd.isnull(d_data['Country_Region'])) # select rows where Country_Region is null
]

# we want to see no rows from this query.
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
      <th>Active</th>
      <th>Admin2</th>
      <th>Combined_Key</th>
      <th>Confirmed</th>
      <th>Country/Region</th>
      <th>Country_Region</th>
      <th>Deaths</th>
      <th>FIPS</th>
      <th>Last Update</th>
      <th>Last_Update</th>
      <th>Lat</th>
      <th>Latitude</th>
      <th>Long_</th>
      <th>Longitude</th>
      <th>Province/State</th>
      <th>Province_State</th>
      <th>Recovered</th>
      <th>source_file</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>



Now that we've confirmed that we can merge the columns we can perform the merge operation


```python
d_data['Country'] = d_data['Country/Region'].fillna(d_data['Country_Region']) 
#          ↑                      ↑                              ↑           
#     3. Add to       2. put it here if its null            1. take this     
#     newcolumn  

d_data['Country/Region'].fillna(d_data['Country_Region']) 
```




    0            Mainland China
    1            Mainland China
    2            Mainland China
    3            Mainland China
    4            Mainland China
                    ...        
    83385    West Bank and Gaza
    83386        Western Sahara
    83387                 Yemen
    83388                Zambia
    83389              Zimbabwe
    Name: Country/Region, Length: 83390, dtype: object



This worked well for the first column pair let's do it for the others. We'll automate the check that makes sure the columns don't overlap and will stop execution if the columns overlap.


```python
if len(d_data[ 
    (~pd.isnull(d_data['Province/State']))
   &(~pd.isnull(d_data['Province_State'])) 
    ]) > 0:
    raise ValueError('Columns overlap, further investigation is needed.')


#--yes

d_data['ps'] = d_data['Province/State'].fillna(d_data['Province_State'])
#          ↑                    ↑                            ↑           
#     3. Add to     2. put it here if its null          1. take this     
#     newcolumn  

d_data['Province/State'].fillna(d_data['Province_State'])
```




    0            Anhui
    1          Beijing
    2        Chongqing
    3           Fujian
    4            Gansu
               ...    
    83385          NaN
    83386          NaN
    83387          NaN
    83388          NaN
    83389          NaN
    Name: Province/State, Length: 83390, dtype: object




```python
if len(d_data[ 
    (~pd.isnull(d_data['Long_']))
   &(~pd.isnull(d_data['Longitude'])) 
    ]) > 0:
    raise ValueError('Columns overlap, further investigation is needed.')


#--yes

d_data['long_val'] = d_data['Long_'].fillna(d_data['Longitude'])
#          ↑                   ↑                         ↑           
#     3. Add to    2. put it here if its null       1. take this     
#     newcolumn  

d_data['Long_'].fillna(d_data['Longitude'])
```




    0              NaN
    1              NaN
    2              NaN
    3              NaN
    4              NaN
               ...    
    83385    35.233200
    83386   -12.885800
    83387    48.516388
    83388    27.849332
    83389    29.154857
    Name: Long_, Length: 83390, dtype: float64




```python
if len(d_data[ 
    (~pd.isnull(d_data['Lat']))
   &(~pd.isnull(d_data['Latitude'])) 
    ]) != 0:
    raise ValueError('Columns overlap, further investigation is needed.')

#--yes

d_data['lat_val'] = d_data['Lat'].fillna(d_data['Latitude'])
#          ↑                   ↑                         ↑           
#     3. Add to    2. put it here if its null       1. take this     
#     newcolumn  

d_data['Lat'].fillna(d_data['Latitude'])
```




    0              NaN
    1              NaN
    2              NaN
    3              NaN
    4              NaN
               ...    
    83385    31.952200
    83386    24.215500
    83387    15.552727
    83388   -13.133897
    83389   -19.015438
    Name: Lat, Length: 83390, dtype: float64




```python
if len(d_data[ 
    (~pd.isnull(d_data['Last Update']))
   &(~pd.isnull(d_data['Last_Update'])) 
    ]) != 0:
    raise ValueError('Columns overlap, further investigation is needed.')

#--yes

d_data['updated_on'] = d_data['Last Update'].fillna(d_data['Last_Update']) 
#          ↑                         ↑                            ↑           
#     3. Add to          2. put it here if its null          1. take this   

d_data['Last Update'].fillna(d_data['Last_Update']) 
```




    0            1/22/2020 17:00
    1            1/22/2020 17:00
    2            1/22/2020 17:00
    3            1/22/2020 17:00
    4            1/22/2020 17:00
                    ...         
    83385    2020-04-15 22:56:32
    83386    2020-04-15 22:56:32
    83387    2020-04-15 22:56:32
    83388    2020-04-15 22:56:32
    83389    2020-04-15 22:56:32
    Name: Last Update, Length: 83390, dtype: object



With the columns merged, we can drop the old columns as they are no longer needed.


```python
d_data.columns
```




    Index(['Active', 'Admin2', 'Combined_Key', 'Confirmed', 'Country/Region',
           'Country_Region', 'Deaths', 'FIPS', 'Last Update', 'Last_Update', 'Lat',
           'Latitude', 'Long_', 'Longitude', 'Province/State', 'Province_State',
           'Recovered', 'source_file', 'Country', 'ps', 'long_val', 'lat_val',
           'updated_on'],
          dtype='object')




```python
d_data.drop(columns=['Active', 'Admin2', 'Combined_Key'
                     , 'Country/Region', 'Country_Region'
                     , 'Last Update', 'Last_Update', 'Lat'
                     , 'Latitude', 'Long_', 'Longitude'
                     , 'Province/State', 'Province_State']
            , inplace=True)
```

In order to work with the data later on, let's convert the updated_on column to date types from strings so that we can sort and group it properly.


```python
# convert updated time to a datetime object to work with
d_data['updated_on'] = pd.to_datetime(d_data['updated_on'])
```

# 4. Clean up the country names to identify the ISO codes for countries

In order to add data onto the map we need to have the ISO codes for all the countries. To do that we can use the pycountry library we installed at the beginning of the course. Let's first see if there will be any issues by looking at the country data. 


```python
set(d_data['Country'].unique())
```




    {' Azerbaijan',
     'Afghanistan',
     'Albania',
     ...
     'Zambia',
     'Zimbabwe',
     'occupied Palestinian territory'}




```python
import pycountry

d_data.loc[ d_data['Country']=='Mainland China', 'Country'] = 'China'
d_data.loc[ d_data['Country']=='Macau', 'Country'] = 'China'
d_data.loc[ d_data['Country']=='South Korea', 'Country'] = 'Republic of Korea'
d_data.loc[ d_data['Country']=='Korea, South', 'Country'] = 'Republic of Korea'
d_data.loc[ d_data['Country']=='Ivory Coast', 'Country'] = "Republic of Cote d'Ivoire"
d_data.loc[ d_data['Country']=='North Ireland', 'Country'] = "United Kingdom"
d_data.loc[ d_data['Country']=='Republic of Ireland', 'Country'] = "Ireland"
d_data.loc[ d_data['Country']=='St. Martin', 'Country'] = "France" 
d_data.loc[ d_data['Country']=='Iran (Islamic Republic of)', 'Country'] = "Iran"
d_data.loc[ d_data['Country']=='West Bank and Gaza', 'Country'] = "Palestine"
d_data.loc[ d_data['Country']=='occupied Palestinian territory', 'Country'] = "Palestine"
d_data.loc[ d_data['Country']=='Channel Islands', 'Country'] = "UK" ## Not technically, but effectively, great tax laws
d_data.loc[ d_data['Country'].isin([ 'Congo (Brazzaville)'
                                    ,'Congo (Kinshasa)']), 'Country'] = 'Congo'
d_data.loc[ d_data['Country']=='Gambia, The', 'Country'] = "Gambia" 
d_data.loc[ d_data['Country']=='Bahamas, The', 'Country'] = "Bahamas" 
d_data.loc[ d_data['Country']=='Cape Verde', 'Country'] = 'Republic of Cabo Verde'
d_data.loc[ d_data['Country']=='East Timor', 'Country'] = 'Timor-Leste'
d_data.loc[ d_data['Country']=='Laos', 'Country'] = "Lao People's Democratic Republic" 
d_data.loc[ d_data['Country']=="Burma", 'Country'] = 'Myanmar'

# dropping disputed teritories and not teritories
d_data = d_data.drop(d_data[d_data['Country']=='Others'].index)
d_data = d_data.drop(d_data[d_data['Country']=='Taipei and environs'].index)
d_data = d_data.drop(d_data[d_data['Country']=='MS Zaandam'].index)
d_data = d_data.drop(d_data[d_data['Country']=='Cruise Ship'].index)
d_data = d_data.drop(d_data[d_data['Country']=='Diamond Princess'].index)
```


```python
countries = pd.Series(d_data['Country'].unique())

def get_iso(country_name):
    return {'Country':country_name, 'ISO_3': pycountry.countries.search_fuzzy(country_name)[0].alpha_3}

countries = pd.DataFrame(list(countries.map(get_iso)))
```


```python
d_data.merge(countries
            , on='Country'
            , how='inner'
            , validate='m:1') 
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
      <th>Confirmed</th>
      <th>Deaths</th>
      <th>FIPS</th>
      <th>Recovered</th>
      <th>source_file</th>
      <th>Country</th>
      <th>ps</th>
      <th>long_val</th>
      <th>lat_val</th>
      <th>updated_on</th>
      <th>ISO_3</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-22-2020.csv</td>
      <td>China</td>
      <td>Anhui</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-01-22 17:00:00</td>
      <td>CHN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>14.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-22-2020.csv</td>
      <td>China</td>
      <td>Beijing</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-01-22 17:00:00</td>
      <td>CHN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>6.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-22-2020.csv</td>
      <td>China</td>
      <td>Chongqing</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-01-22 17:00:00</td>
      <td>CHN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-22-2020.csv</td>
      <td>China</td>
      <td>Fujian</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-01-22 17:00:00</td>
      <td>CHN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-22-2020.csv</td>
      <td>China</td>
      <td>Gansu</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-01-22 17:00:00</td>
      <td>CHN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>83206</th>
      <td>1.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-11-2020.csv</td>
      <td>Yemen</td>
      <td>NaN</td>
      <td>48.516388</td>
      <td>15.552727</td>
      <td>2020-04-11 22:45:13</td>
      <td>YEM</td>
    </tr>
    <tr>
      <th>83207</th>
      <td>1.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-12-2020.csv</td>
      <td>Yemen</td>
      <td>NaN</td>
      <td>48.516388</td>
      <td>15.552727</td>
      <td>2020-04-12 23:17:00</td>
      <td>YEM</td>
    </tr>
    <tr>
      <th>83208</th>
      <td>1.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-13-2020.csv</td>
      <td>Yemen</td>
      <td>NaN</td>
      <td>48.516388</td>
      <td>15.552727</td>
      <td>2020-04-13 23:07:34</td>
      <td>YEM</td>
    </tr>
    <tr>
      <th>83209</th>
      <td>1.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-14-2020.csv</td>
      <td>Yemen</td>
      <td>NaN</td>
      <td>48.516388</td>
      <td>15.552727</td>
      <td>2020-04-14 23:33:12</td>
      <td>YEM</td>
    </tr>
    <tr>
      <th>83210</th>
      <td>1.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-15-2020.csv</td>
      <td>Yemen</td>
      <td>NaN</td>
      <td>48.516388</td>
      <td>15.552727</td>
      <td>2020-04-15 22:56:32</td>
      <td>YEM</td>
    </tr>
  </tbody>
</table>
<p>83211 rows × 11 columns</p>
</div>




```python
d_data =\
    d_data.merge(countries
                , on='Country'
                , how='inner'
                , validate='m:1') 
```

### 4. a. identify and fix issues with the last-reported data column for our grouping purposes


```python
pd.set_option("display.max_rows", 300) # increase the number of rows visible 

d_data[d_data['ISO_3'] == 'DEU']
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
      <th>Confirmed</th>
      <th>Deaths</th>
      <th>FIPS</th>
      <th>Recovered</th>
      <th>source_file</th>
      <th>Country</th>
      <th>ps</th>
      <th>long_val</th>
      <th>lat_val</th>
      <th>updated_on</th>
      <th>ISO_3</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>76227</th>
      <td>4.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-28-2020.csv</td>
      <td>Germany</td>
      <td>Bavaria</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-01-28 23:00:00</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76228</th>
      <td>4.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-29-2020.csv</td>
      <td>Germany</td>
      <td>Bavaria</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-01-29 19:30:00</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76229</th>
      <td>4.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-30-2020.csv</td>
      <td>Germany</td>
      <td>Bavaria</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-01-30 16:00:00</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76230</th>
      <td>5.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-31-2020.csv</td>
      <td>Germany</td>
      <td>Bavaria</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-01-31 23:59:00</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76231</th>
      <td>8.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-01-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-01 18:33:00</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76232</th>
      <td>10.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-02-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-02 18:03:05</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76233</th>
      <td>12.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-03-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-03 20:53:02</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76234</th>
      <td>12.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-04-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-03 20:53:02</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76235</th>
      <td>12.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-05-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-03 20:53:02</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76236</th>
      <td>12.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-06-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-03 20:53:02</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76237</th>
      <td>13.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-07-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-07 16:33:03</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76238</th>
      <td>13.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-08-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-07 16:33:03</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76239</th>
      <td>14.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-09-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-09 06:13:11</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76240</th>
      <td>14.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-10-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-09 06:13:11</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76241</th>
      <td>16.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-11-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-11 19:33:03</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76242</th>
      <td>16.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-12-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-11 19:33:03</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76243</th>
      <td>16.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-13-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-13 15:13:11</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76244</th>
      <td>16.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-14-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-13 15:13:11</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76245</th>
      <td>16.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-15-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-13 15:13:11</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76246</th>
      <td>16.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-16-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-13 15:13:11</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76247</th>
      <td>16.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-17-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-13 15:13:11</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76248</th>
      <td>16.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>12.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-18-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-18 17:03:03</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76249</th>
      <td>16.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>12.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-19-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-18 17:03:03</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76250</th>
      <td>16.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>12.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-20-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-18 17:03:03</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76251</th>
      <td>16.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>14.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-21-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-21 23:03:13</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76252</th>
      <td>16.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>14.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-22-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-21 23:03:13</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76253</th>
      <td>16.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>14.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-23-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-21 23:03:13</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76254</th>
      <td>16.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>14.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-24-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-21 23:03:13</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76255</th>
      <td>17.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>14.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-25-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-25 21:33:02</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76256</th>
      <td>27.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>15.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-26-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-26 23:43:03</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76257</th>
      <td>46.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>16.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-27-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-27 23:13:04</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76258</th>
      <td>48.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>16.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-28-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-28 00:13:18</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76259</th>
      <td>79.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>16.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/02-29-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-02-29 14:43:03</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76260</th>
      <td>130.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>16.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-01-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>9.000000</td>
      <td>51.000000</td>
      <td>2020-03-01 23:23:02</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76261</th>
      <td>159.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>16.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-02-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>9.000000</td>
      <td>51.000000</td>
      <td>2020-03-02 20:33:02</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76262</th>
      <td>196.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>16.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-03-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>9.000000</td>
      <td>51.000000</td>
      <td>2020-03-03 20:03:06</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76263</th>
      <td>262.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>16.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-04-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>9.000000</td>
      <td>51.000000</td>
      <td>2020-03-04 19:33:03</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76264</th>
      <td>482.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>16.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-05-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>9.000000</td>
      <td>51.000000</td>
      <td>2020-03-05 17:43:03</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76265</th>
      <td>670.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>17.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-06-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>9.000000</td>
      <td>51.000000</td>
      <td>2020-03-06 17:53:03</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76266</th>
      <td>799.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>18.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-07-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>9.000000</td>
      <td>51.000000</td>
      <td>2020-03-07 17:43:05</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76267</th>
      <td>1040.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>18.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-08-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>9.000000</td>
      <td>51.000000</td>
      <td>2020-03-08 21:03:03</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76268</th>
      <td>1176.0</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>18.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-09-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>9.000000</td>
      <td>51.000000</td>
      <td>2020-03-09 18:13:11</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76269</th>
      <td>1457.0</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>18.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-10-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>9.000000</td>
      <td>51.000000</td>
      <td>2020-03-10 18:53:02</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76270</th>
      <td>1908.0</td>
      <td>3.0</td>
      <td>NaN</td>
      <td>25.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-11-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>9.000000</td>
      <td>51.000000</td>
      <td>2020-03-11 19:13:17</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76271</th>
      <td>2078.0</td>
      <td>3.0</td>
      <td>NaN</td>
      <td>25.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-12-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>9.000000</td>
      <td>51.000000</td>
      <td>2020-03-12 09:53:06</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76272</th>
      <td>3675.0</td>
      <td>7.0</td>
      <td>NaN</td>
      <td>46.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-13-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451500</td>
      <td>51.165700</td>
      <td>2020-03-11 20:00:00</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76273</th>
      <td>4585.0</td>
      <td>9.0</td>
      <td>NaN</td>
      <td>46.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-14-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451500</td>
      <td>51.165700</td>
      <td>2020-03-14 22:13:11</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76274</th>
      <td>5795.0</td>
      <td>11.0</td>
      <td>NaN</td>
      <td>46.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-15-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451500</td>
      <td>51.165700</td>
      <td>2020-03-15 18:20:18</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76275</th>
      <td>7272.0</td>
      <td>17.0</td>
      <td>NaN</td>
      <td>67.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-16-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451500</td>
      <td>51.165700</td>
      <td>2020-03-16 20:13:11</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76276</th>
      <td>9257.0</td>
      <td>24.0</td>
      <td>NaN</td>
      <td>67.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-17-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451500</td>
      <td>51.165700</td>
      <td>2020-03-17 18:53:02</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76277</th>
      <td>12327.0</td>
      <td>28.0</td>
      <td>NaN</td>
      <td>105.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-18-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451500</td>
      <td>51.165700</td>
      <td>2020-03-18 19:33:02</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76278</th>
      <td>15320.0</td>
      <td>44.0</td>
      <td>NaN</td>
      <td>113.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-19-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451500</td>
      <td>51.165700</td>
      <td>2020-03-19 20:13:08</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76279</th>
      <td>19848.0</td>
      <td>67.0</td>
      <td>NaN</td>
      <td>180.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-20-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451500</td>
      <td>51.165700</td>
      <td>2020-03-20 20:13:15</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76280</th>
      <td>22213.0</td>
      <td>84.0</td>
      <td>NaN</td>
      <td>233.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-21-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451500</td>
      <td>51.165700</td>
      <td>2020-03-21 20:43:02</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76281</th>
      <td>24873.0</td>
      <td>94.0</td>
      <td>NaN</td>
      <td>266.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-22-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-03-22 23:45:00</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76282</th>
      <td>29056.0</td>
      <td>123.0</td>
      <td>NaN</td>
      <td>453.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-23-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-03-23 23:19:21</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76283</th>
      <td>32986.0</td>
      <td>157.0</td>
      <td>NaN</td>
      <td>3243.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-24-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-03-24 23:37:15</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76284</th>
      <td>37323.0</td>
      <td>206.0</td>
      <td>NaN</td>
      <td>3547.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-25-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-03-25 23:33:04</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76285</th>
      <td>43938.0</td>
      <td>267.0</td>
      <td>NaN</td>
      <td>5673.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-26-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-03-26 23:48:18</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76286</th>
      <td>50871.0</td>
      <td>342.0</td>
      <td>NaN</td>
      <td>6658.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-27-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-03-27 23:23:03</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76287</th>
      <td>57695.0</td>
      <td>433.0</td>
      <td>NaN</td>
      <td>8481.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-28-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-03-28 23:05:00</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76288</th>
      <td>62095.0</td>
      <td>533.0</td>
      <td>NaN</td>
      <td>9211.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-29-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-03-29 23:08:00</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76289</th>
      <td>66885.0</td>
      <td>645.0</td>
      <td>NaN</td>
      <td>13500.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-30-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-03-30 22:52:00</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76290</th>
      <td>71808.0</td>
      <td>775.0</td>
      <td>NaN</td>
      <td>16100.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/03-31-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-03-31 23:43:43</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76291</th>
      <td>77872.0</td>
      <td>920.0</td>
      <td>NaN</td>
      <td>18700.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-01-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-04-01 21:58:34</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76292</th>
      <td>84794.0</td>
      <td>1107.0</td>
      <td>NaN</td>
      <td>22440.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-02-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-04-02 23:25:00</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76293</th>
      <td>91159.0</td>
      <td>1275.0</td>
      <td>NaN</td>
      <td>24575.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-03-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-04-03 22:46:20</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76294</th>
      <td>96092.0</td>
      <td>1444.0</td>
      <td>NaN</td>
      <td>26400.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-04-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-04-04 23:34:00</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76295</th>
      <td>100123.0</td>
      <td>1584.0</td>
      <td>NaN</td>
      <td>28700.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-05-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-04-05 23:06:26</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76296</th>
      <td>103374.0</td>
      <td>1810.0</td>
      <td>NaN</td>
      <td>28700.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-06-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-04-06 23:21:00</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76297</th>
      <td>107663.0</td>
      <td>2016.0</td>
      <td>NaN</td>
      <td>36081.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-07-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-04-07 23:04:29</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76298</th>
      <td>113296.0</td>
      <td>2349.0</td>
      <td>NaN</td>
      <td>46300.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-08-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-04-08 22:51:39</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76299</th>
      <td>118181.0</td>
      <td>2607.0</td>
      <td>NaN</td>
      <td>52407.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-09-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-04-09 23:02:19</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76300</th>
      <td>122171.0</td>
      <td>2767.0</td>
      <td>NaN</td>
      <td>53913.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-10-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-04-10 22:53:48</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76301</th>
      <td>124908.0</td>
      <td>2736.0</td>
      <td>NaN</td>
      <td>57400.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-11-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-04-11 22:45:13</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76302</th>
      <td>127854.0</td>
      <td>3022.0</td>
      <td>NaN</td>
      <td>60300.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-12-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-04-12 23:17:00</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76303</th>
      <td>130072.0</td>
      <td>3194.0</td>
      <td>NaN</td>
      <td>64300.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-13-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-04-13 23:07:34</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76304</th>
      <td>131359.0</td>
      <td>3294.0</td>
      <td>NaN</td>
      <td>68200.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-14-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-04-14 23:33:12</td>
      <td>DEU</td>
    </tr>
    <tr>
      <th>76305</th>
      <td>134753.0</td>
      <td>3804.0</td>
      <td>NaN</td>
      <td>72600.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-15-2020.csv</td>
      <td>Germany</td>
      <td>NaN</td>
      <td>10.451526</td>
      <td>51.165691</td>
      <td>2020-04-15 22:56:32</td>
      <td>DEU</td>
    </tr>
  </tbody>
</table>
</div>




```python
d_data['report_date'] = \
     pd.to_datetime( 
         d_data['source_file'].astype('str')                      # convert the values in the column to string
                              .str.rsplit(os.path.sep,1, expand=True)[1]  # extract the file name from the file path
                              .str.replace('.csv','')             # remove the csv extension
       , dayfirst=False)                                          # convert the newly extracted string to date
```

# 5. Visualize the data on a map

To take the data we've prepared and put it onto the map we need to sum the number of confirmed cases by the date of the report and the country code. 


```python
d_data.groupby(by=['report_date', 'ISO_3'])\
      .agg({'Confirmed': 'sum'})
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
      <th></th>
      <th>Confirmed</th>
    </tr>
    <tr>
      <th>report_date</th>
      <th>ISO_3</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="5" valign="top">2020-01-22</th>
      <th>CHN</th>
      <td>548.0</td>
    </tr>
    <tr>
      <th>HKG</th>
      <td>0.0</td>
    </tr>
    <tr>
      <th>JPN</th>
      <td>2.0</td>
    </tr>
    <tr>
      <th>PRK</th>
      <td>1.0</td>
    </tr>
    <tr>
      <th>THA</th>
      <td>2.0</td>
    </tr>
    <tr>
      <th>...</th>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th rowspan="5" valign="top">2020-04-15</th>
      <th>VNM</th>
      <td>267.0</td>
    </tr>
    <tr>
      <th>YEM</th>
      <td>1.0</td>
    </tr>
    <tr>
      <th>ZAF</th>
      <td>2506.0</td>
    </tr>
    <tr>
      <th>ZMB</th>
      <td>48.0</td>
    </tr>
    <tr>
      <th>ZWE</th>
      <td>23.0</td>
    </tr>
  </tbody>
</table>
<p>7866 rows × 1 columns</p>
</div>




```python

viz = \
d_data.groupby(by=['report_date', 'ISO_3', 'Country'])\
      .agg({'Confirmed': 'sum'})\
      .reset_index()

```

We also need to convert the report date back to string to display it. So let's convert the report date column to string again.


```python
viz['report_date'] = viz['report_date'].dt.strftime( '%Y-%m-%d')
```


```python
from urllib.request import urlopen
import json
with urlopen('https://raw.githubusercontent.com/plotly/datasets/master/geojson-counties-fips.json') as response:
    counties = json.load(response)
```


```python
subset = viz[viz['report_date'] == '2020-04-14']

px.choropleth(subset
             , locations='ISO_3'
             , locationmode = 'ISO-3'
             , geojson=counties
             , animation_frame='report_date'
             , animation_group='ISO_3'
             , color='Confirmed'
             , color_continuous_scale= 'Greens').write_html('choropleth_1.html', full_html=False, include_plotlyjs='cdn')


```

{% include choropleth_1.html %}
 
# 6. Tweak the vizualization


```python
viz['sqrt_Confirmed'] = np.sqrt(viz['Confirmed'].clip(lower=1)) # calculate the square root of the confirmed
                                                                # cases, clip the columns lower values to at 
                                                                # least 1.

fig = \
px.choropleth(viz
             , locations='ISO_3'
             , locationmode='ISO-3'
             , animation_frame='report_date'
             , hover_data=['Confirmed', 'Country']
             , animation_group='ISO_3'
             , color='sqrt_Confirmed'
             , color_continuous_scale= [[0,  'rgba(0, 255, 0, .07)' ]
                                        ,[0.5, 'green']
                                        ,[1, 'palegreen']]
             , template='plotly_dark')
fig
```

{% include choropleth_2.html %}



```python
fig.show(renderer='browser')
```

# 7. What if we look at the data by proportion of the population?


```python
# Load population data for countries
pop_data = px.data.gapminder()

# Select the most recent year available 
pop_data = pop_data[pop_data['year']==2007]

# Join the population data to our data set based on the country ISO3 code
pop_viz =viz.merge(pop_data[['pop', 'iso_alpha']]
          , left_on='ISO_3'
          , right_on='iso_alpha'
          , how='left').dropna()

# calculate the number of cases per 100k population members
pop_viz['proportion'] = (pop_viz['Confirmed']/pop_viz['pop'])*100_000

pop_viz['sqrt_proportion'] = np.sqrt(pop_viz['proportion'])

# map the data 
px.choropleth(pop_viz
             , locations='ISO_3'
             , locationmode='ISO-3'
             , animation_frame='report_date'
             , hover_data=['Confirmed', 'proportion', 'Country']
             , animation_group='ISO_3'
             , color='sqrt_proportion'
             , color_continuous_scale= [[0,  'rgba(0, 255, 0, .07)' ]
                                        ,[0.5, 'green']
                                        ,[1, 'palegreen']]
             , template='plotly_dark')\
.show(renderer='browser')
```

{% include choropleth_3.html %}

# 8. Extra problems to solve

After this course you should be armed with some tools to work on other datasets and problems. Here are some additional problems which will stretch your abilities a bit and will require extra reading to solve:

1. Can you rewrite the column merging code into a function so that all you need to do is pass the function, a dataframe, two input column names, and an output column name to have it check if the columns can be merged and merge them? 
```python 
def merge_cols(input_df, col1, col2, outcol):
    ...
```
2. Can you change the way we load the files to avoid needing to merge columns? (hint: you will need to use the pandas rename function ` dataframe.rename(columns={'from':'to') ` )

3. Can you create a visualization from the timeline dataset? (hint: you will need to use the ` dataframe.melt() ` command very rapidly demonstrated in python 101 and you will likley need to do clean up on the countries before doing the ISO_3 lookups)





# Solution for problem 1


```python
def merge_cols(input_df, col1, col2, outcol):
    if len(input_df[ 
        (~pd.isnull(input_df[col1]))
       &(~pd.isnull(input_df[col2])) 
        ]) != 0:
        raise ValueError('Columns overlap, further investigation is needed.')

    input_df[outcol] = input_df[col1].fillna(input_df[col2])

    
```


```python
d_data = pd.concat(all_data
                  , axis='index'
                  , join='outer'
                  , ignore_index=True
                  , sort=True)

merge_cols(d_data, 'Last Update', 'Last_Update', 'updated_on')

d_data
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
      <th>Active</th>
      <th>Admin2</th>
      <th>Combined_Key</th>
      <th>Confirmed</th>
      <th>Country/Region</th>
      <th>Country_Region</th>
      <th>Deaths</th>
      <th>FIPS</th>
      <th>Last Update</th>
      <th>Last_Update</th>
      <th>Lat</th>
      <th>Latitude</th>
      <th>Long_</th>
      <th>Longitude</th>
      <th>Province/State</th>
      <th>Province_State</th>
      <th>Recovered</th>
      <th>source_file</th>
      <th>updated_on</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>Mainland China</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1/22/2020 17:00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Anhui</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-22-2020.csv</td>
      <td>1/22/2020 17:00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>14.0</td>
      <td>Mainland China</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1/22/2020 17:00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Beijing</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-22-2020.csv</td>
      <td>1/22/2020 17:00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>6.0</td>
      <td>Mainland China</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1/22/2020 17:00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Chongqing</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-22-2020.csv</td>
      <td>1/22/2020 17:00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>Mainland China</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1/22/2020 17:00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Fujian</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-22-2020.csv</td>
      <td>1/22/2020 17:00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Mainland China</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1/22/2020 17:00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Gansu</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-22-2020.csv</td>
      <td>1/22/2020 17:00</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>83385</th>
      <td>309.0</td>
      <td>NaN</td>
      <td>West Bank and Gaza</td>
      <td>374.0</td>
      <td>NaN</td>
      <td>West Bank and Gaza</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-04-15 22:56:32</td>
      <td>31.952200</td>
      <td>NaN</td>
      <td>35.233200</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>63.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-15-2020.csv</td>
      <td>2020-04-15 22:56:32</td>
    </tr>
    <tr>
      <th>83386</th>
      <td>6.0</td>
      <td>NaN</td>
      <td>Western Sahara</td>
      <td>6.0</td>
      <td>NaN</td>
      <td>Western Sahara</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-04-15 22:56:32</td>
      <td>24.215500</td>
      <td>NaN</td>
      <td>-12.885800</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-15-2020.csv</td>
      <td>2020-04-15 22:56:32</td>
    </tr>
    <tr>
      <th>83387</th>
      <td>1.0</td>
      <td>NaN</td>
      <td>Yemen</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>Yemen</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-04-15 22:56:32</td>
      <td>15.552727</td>
      <td>NaN</td>
      <td>48.516388</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-15-2020.csv</td>
      <td>2020-04-15 22:56:32</td>
    </tr>
    <tr>
      <th>83388</th>
      <td>16.0</td>
      <td>NaN</td>
      <td>Zambia</td>
      <td>48.0</td>
      <td>NaN</td>
      <td>Zambia</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-04-15 22:56:32</td>
      <td>-13.133897</td>
      <td>NaN</td>
      <td>27.849332</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>30.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-15-2020.csv</td>
      <td>2020-04-15 22:56:32</td>
    </tr>
    <tr>
      <th>83389</th>
      <td>19.0</td>
      <td>NaN</td>
      <td>Zimbabwe</td>
      <td>23.0</td>
      <td>NaN</td>
      <td>Zimbabwe</td>
      <td>3.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2020-04-15 22:56:32</td>
      <td>-19.015438</td>
      <td>NaN</td>
      <td>29.154857</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/04-15-2020.csv</td>
      <td>2020-04-15 22:56:32</td>
    </tr>
  </tbody>
</table>
<p>83390 rows × 19 columns</p>
</div>



# Solution for problem 2


```python

col_pairs = {
    'Country/Region': 'Country_Region'
    ,'Province/State': 'Province_State'
    ,'Long_': 'Longitude'
    ,'Lat': 'Latitude'
    ,'Last Update': 'Last_Update'
}

all_data = []
meta_data = []

# For every file in our data set path
for file in sorted(glob.glob(dataset_path)):
    
    # 1. Read the file to a temporary data frame
    df = pd.read_csv(file).rename(columns=col_pairs)
    
    # 2. Append a dictionary with the file meta_data into the metadata list
    meta_data.append( {  'file_name': file.rsplit(os.path.sep,1)[1]
                       , 'num_rows': df.shape[0]
                       , 'num_cols': df.shape[1]
                       , 'col_names': '\t'.join(sorted(list(df.columns)))} )
    
    # Add the file name to the loaded data
    df['source_file'] = file
    
    # 4. Add the loaded data with the file name to a list with all the loaded data
    all_data.append(df)

# 5. Create a table/dataframe out of our meta_data 
meta_data = pd.DataFrame(meta_data)


# output result to notebook window
meta_data.groupby(['num_cols'])\
         .agg({ 'num_rows': 'sum'
              , 'file_name': sorted
              , 'col_names': set })
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
      <th>num_rows</th>
      <th>file_name</th>
      <th>col_names</th>
    </tr>
    <tr>
      <th>num_cols</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>6</th>
      <td>2818</td>
      <td>[01-22-2020.csv, 01-23-2020.csv, 01-24-2020.csv, 01-25-2020.csv, 01-26-2020.csv, 01-27-2020.csv, 01-28-2020.csv, 01-29-2020.csv, 01-30-2020.csv, 0...</td>
      <td>{Confirmed\tCountry_Region\tDeaths\tLast_Update\tProvince_State\tRecovered}</td>
    </tr>
    <tr>
      <th>8</th>
      <td>4799</td>
      <td>[03-01-2020.csv, 03-02-2020.csv, 03-03-2020.csv, 03-04-2020.csv, 03-05-2020.csv, 03-06-2020.csv, 03-07-2020.csv, 03-08-2020.csv, 03-09-2020.csv, 0...</td>
      <td>{Confirmed\tCountry_Region\tDeaths\tLast_Update\tLatitude\tLongitude\tProvince_State\tRecovered}</td>
    </tr>
    <tr>
      <th>12</th>
      <td>75773</td>
      <td>[03-22-2020.csv, 03-23-2020.csv, 03-24-2020.csv, 03-25-2020.csv, 03-26-2020.csv, 03-27-2020.csv, 03-28-2020.csv, 03-29-2020.csv, 03-30-2020.csv, 0...</td>
      <td>{Active\tAdmin2\tCombined_Key\tConfirmed\tCountry_Region\tDeaths\tFIPS\tLast_Update\tLatitude\tLongitude\tProvince_State\tRecovered}</td>
    </tr>
  </tbody>
</table>
</div>




```python
d_data = pd.concat(all_data
                  , axis='index'
                  , join='outer'
                  , ignore_index=True
                  , sort=True)


d_data.head()
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
      <th>Active</th>
      <th>Admin2</th>
      <th>Combined_Key</th>
      <th>Confirmed</th>
      <th>Country_Region</th>
      <th>Deaths</th>
      <th>FIPS</th>
      <th>Last_Update</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Province_State</th>
      <th>Recovered</th>
      <th>source_file</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>Mainland China</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1/22/2020 17:00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Anhui</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-22-2020.csv</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>14.0</td>
      <td>Mainland China</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1/22/2020 17:00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Beijing</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-22-2020.csv</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>6.0</td>
      <td>Mainland China</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1/22/2020 17:00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Chongqing</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-22-2020.csv</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>Mainland China</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1/22/2020 17:00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Fujian</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-22-2020.csv</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Mainland China</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1/22/2020 17:00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Gansu</td>
      <td>NaN</td>
      <td>us_jhu_data/csse_covid_19_data/csse_covid_19_daily_reports/01-22-2020.csv</td>
    </tr>
  </tbody>
</table>
</div>



# Solution for problem 3


```python
df = pd.read_csv('us_jhu_data/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_global.csv')

```


```python
df = df.melt(  id_vars=['Province/State', 'Country/Region', 'Lat', 'Long']
        , var_name='report_date'
        , value_name='confirmed_cases')

df.rename(columns={'Country/Region': 'Country'}, inplace=True)


```


```python
import pycountry

df.loc[ df['Country']=='Mainland China', 'Country'] = 'China'
df.loc[ df['Country']=='Macau', 'Country'] = 'China'
df.loc[ df['Country']=='South Korea', 'Country'] = 'Republic of Korea'
df.loc[ df['Country']=='Korea, South', 'Country'] = 'Republic of Korea'
df.loc[ df['Country']=='Ivory Coast', 'Country'] = "Republic of Cote d'Ivoire"
df.loc[ df['Country']=='North Ireland', 'Country'] = "United Kingdom"
df.loc[ df['Country']=='Republic of Ireland', 'Country'] = "Ireland"
df.loc[ df['Country']=='St. Martin', 'Country'] = "France" 
df.loc[ df['Country']=='Iran (Islamic Republic of)', 'Country'] = "Iran"
df.loc[ df['Country']=='West Bank and Gaza', 'Country'] = "Palestine"
df.loc[ df['Country']=='Channel Islands', 'Country'] = "UK" ## Not technically, but effectively, great tax laws
df.loc[ df['Country'].isin([ 'Congo (Brazzaville)'
                                    ,'Congo (Kinshasa)']), 'Country'] = 'Congo'
df.loc[ df['Country']=='Gambia, The', 'Country'] = "Gambia" 
df.loc[ df['Country']=='Bahamas, The', 'Country'] = "Bahamas" 
df.loc[ df['Country']=='Cape Verde', 'Country'] = 'Republic of Cabo Verde'
df.loc[ df['Country']=='East Timor', 'Country'] = 'Timor-Leste'
df.loc[ df['Country']=='Laos', 'Country'] = "Lao People's Democratic Republic" 
df.loc[ df['Country']=="Burma", 'Country'] = 'Myanmar'

# dropping disputed teritories and not teritories
df = df.drop(df[df['Country']=='Others'].index)
df = df.drop(df[df['Country']=='Taipei and environs'].index)
df = df.drop(df[df['Country']=='occupied Palestinian territory'].index) 
df = df.drop(df[df['Country']=='Taiwan*'].index)
df = df.drop(df[df['Country']=='Taiwan'].index)
df = df.drop(df[df['Country']=='MS Zaandam'].index)
df = df.drop(df[df['Country']=='Cruise Ship'].index)
df = df.drop(df[df['Country']=='Diamond Princess'].index)
```


```python
countries = pd.Series(df['Country'].unique())

def get_iso(country_name):
    return {'Country':country_name, 'ISO_3': pycountry.countries.search_fuzzy(country_name)[0].alpha_3}

countries = pd.DataFrame(list(countries.map(get_iso)))

df = df.merge(countries
            , on='Country'
            , how='inner'
            , validate='m:1') 
```


```python
df
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
      <th>Province/State</th>
      <th>Country</th>
      <th>Lat</th>
      <th>Long</th>
      <th>report_date</th>
      <th>confirmed_cases</th>
      <th>ISO_3</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>Afghanistan</td>
      <td>33.000000</td>
      <td>65.000000</td>
      <td>1/22/20</td>
      <td>0</td>
      <td>AFG</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NaN</td>
      <td>Afghanistan</td>
      <td>33.000000</td>
      <td>65.000000</td>
      <td>1/23/20</td>
      <td>0</td>
      <td>AFG</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NaN</td>
      <td>Afghanistan</td>
      <td>33.000000</td>
      <td>65.000000</td>
      <td>1/24/20</td>
      <td>0</td>
      <td>AFG</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NaN</td>
      <td>Afghanistan</td>
      <td>33.000000</td>
      <td>65.000000</td>
      <td>1/25/20</td>
      <td>0</td>
      <td>AFG</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NaN</td>
      <td>Afghanistan</td>
      <td>33.000000</td>
      <td>65.000000</td>
      <td>1/26/20</td>
      <td>0</td>
      <td>AFG</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>22180</th>
      <td>NaN</td>
      <td>Yemen</td>
      <td>15.552727</td>
      <td>48.516388</td>
      <td>4/11/20</td>
      <td>1</td>
      <td>YEM</td>
    </tr>
    <tr>
      <th>22181</th>
      <td>NaN</td>
      <td>Yemen</td>
      <td>15.552727</td>
      <td>48.516388</td>
      <td>4/12/20</td>
      <td>1</td>
      <td>YEM</td>
    </tr>
    <tr>
      <th>22182</th>
      <td>NaN</td>
      <td>Yemen</td>
      <td>15.552727</td>
      <td>48.516388</td>
      <td>4/13/20</td>
      <td>1</td>
      <td>YEM</td>
    </tr>
    <tr>
      <th>22183</th>
      <td>NaN</td>
      <td>Yemen</td>
      <td>15.552727</td>
      <td>48.516388</td>
      <td>4/14/20</td>
      <td>1</td>
      <td>YEM</td>
    </tr>
    <tr>
      <th>22184</th>
      <td>NaN</td>
      <td>Yemen</td>
      <td>15.552727</td>
      <td>48.516388</td>
      <td>4/15/20</td>
      <td>1</td>
      <td>YEM</td>
    </tr>
  </tbody>
</table>
<p>22185 rows × 7 columns</p>
</div>




```python
df[pd.isnull(df['ISO_3'])]
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
      <th>Province/State</th>
      <th>Country</th>
      <th>Lat</th>
      <th>Long</th>
      <th>report_date</th>
      <th>confirmed_cases</th>
      <th>ISO_3</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
viz2 = \
df.groupby(by=['report_date', 'ISO_3', 'Country'])\
      .agg({'confirmed_cases': 'sum'})\
      .reset_index()
```


```python
viz2['sqrt_Confirmed'] = np.sqrt(viz2['confirmed_cases'].clip(lower=1)) # calculate the square root of the confirmed
                                                                # cases, clip the columns lower values to at 
                                                                # least 1.

fig = \
px.choropleth(viz2
             , locations='ISO_3'
             , locationmode='ISO-3'
             , animation_frame='report_date'
             , hover_data=['confirmed_cases', 'Country']
             , animation_group='ISO_3'
             , color='sqrt_Confirmed'
             , color_continuous_scale= [[0,  'rgba(0, 255, 0, .07)' ]
                                        ,[0.5, 'green']
                                        ,[1, 'palegreen']]
             , template='plotly_dark')
fig
```


