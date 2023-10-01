<h1>Unemployment and Participation Analysis by State</h1>
<p align="center">
<img src="https://github.com/andrew-disario/unemployment-and-participation-analysis-by-state/blob/main/unemployment-and-participation-analysis-by-state.png?raw=true" height="100%" width="100%" alt="unemployment-and-participation-analysis-by-state"/>
<br />
In this project, we analyze state unemployment and participation rates using the Federal Reserve of Economic Data website, Python, Pandas, Matplotlib, Plotly, Time and FredAPI.
  
<h2>Part I - State Unemployment Rate Analysis</h2>

<b>Set Up Notebook and Fred Object</b>
1. Import libraries.
```
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import plotly.express as px
import time
from fredapi import Fred
```
2. Set style, Pandas parameters and color palette.
```
plt.style.use('fivethirtyeight')
pd.set_option('max_columns', 500)
color_pal = plt.rcParams["axes.prop_cycle"].by_key()["color"]
```
3. Set up Fred key and Fred object using your Fred API key.
  - You can get your own Fred API key at [fred.stlouisfed.org/docs/api/api_key](https://fred.stlouisfed.org/docs/api/api_key.html).
```
fred_key = 'ENTER_YOUR_API_KEY_HERE' 
fred = Fred(api_key=fred_key)
```
<b>Pull Data from FRED into Python</b>
1. Search FRED for "Unemployment rate state" filtered for monthly frequency and output results to dataframe.
```
unemp_df = fred.search('unemployment rate state', filter=('frequency','Monthly'))
```
2. Query dataframe where ```seasonal_adjustment = 'Seasonally Adjusted'``` and ```units = Percent```.
```
unemp_df = unemp_df.query('seasonal_adjustment == "Seasonally Adjusted" and units == "Percent"')
```
3. Filter dataframe to where title contains "Unemployment Rate".
```
unemp_df = unemp_df.loc[unemp_df['title'].str.contains('Unemployment Rate')]
```
<b>Filter and Clean Data in Python</b>
1. Iterate over index using ```get_series``` and appending results to list of series of unemployment rates.
	- Use ```time.sleep(0.1)``` to avoid requesting too fast and getting blocked.
```
all_results = []

for myid in unemp_df.index:
    results = fred.get_series(myid)
    results = results.to_frame(name=myid)
    all_results.append(results)
    time.sleep(0.1)
```
2. Concatenate list of series into dataframe.
```
uemp_results = pd.concat(all_results, axis = 1)
```
3. Iterate over columns and append names longer than four characters to get list of column names to drop.
```
cols_to_drop = []

for i in uemp_results:
    if len(i) > 4:
        cols_to_drop.append(i)
```
4. Drop columns with column name in list.
```
uemp_results = uemp_results.drop(columns = cols_to_drop, axis=1)
```
5. Copy dataframe and drop null values.
```
uemp_states = uemp_results.copy()
uemp_states = uemp_states.dropna()
```
<b>Finish Cleaning and Plot Data</b>
1. Create dictionary of titles with "Unemployment Rate in" replaced with "".
```
id_to_state = unemp_df['title'].str.replace('Unemployment Rate in ','').to_dict()
```
2. Replace column names with titles from ```id_to_state```.
```
uemp_states.columns = [id_to_state[c] for c in uemp_states.columns]
```
3. Plot dataframe using ```plotly.express.line( )``` a.k.a. ```px.line( )```.
```
px.line(uemp_states)
```
<p align="center">
<img src="https://github.com/andrew-disario/economic-data-analysis/blob/main/state_unemp_line.png?raw=true" height="100%" width="100%" alt="state-unemp-line"/>
<br />

<b>Plot Bar Chart for State Unemployment - May, 2020</b>
1. Use ```.loc[ ]``` to filter index (date) on ```'2020-05-01'```
2. Sort values for ```'2020-05-01'``` from lowest to highest unemployment rates
3. Transform data by flipping columns and rows  with```.T``` 
4. Plot data with parameters ```.plot(kind = 'bar', figsize = (8, 12), width = 0.7, edgecolor = 'black', title = 'Unemployment Rate by State - May, 2020'```
5. Remove the legend with ```ax.legend( ).remove( )```
6. Set x-label to ```'% Unemployed'```
7. Set up bar chart for May 1st, 2020
8. Show plot
```
ax = uemp_states.loc[uemp_states.index == '2020-05-01'].T \
    .sort_values('2020-05-01') \
    .plot(kind='bar', figsize=(8, 12), width=0.7, edgecolor='black',
          title='Unemployment Rate by State, May 2020')
ax.legend().remove()
ax.set_xlabel('% Unemployed')
plt.show()
```
<p align="center">
<img src="https://github.com/andrew-disario/economic-data-analysis/blob/main/state_unemp_2020-05-01_bar.png?raw=true" height="55%" width="55%" alt="state-unemp-2020-05-01"/>
<br />

<h2>Part II - Participation & Unemployment by State</h2>

<b>Pull data from FRED into Python</b>
1. Search Fred for "participation rate state" filtered by monthly frequency and output results to dataframe.
```
part_df = fred.search('participation rate state', filter=('frequency','Monthly'))```
```
2. Query dataframe where ```seasonal_adjustment = 'Seasonally Adjusted'``` and ```units = Percent```.
```
part_df = part_df.query('seasonal_adjustment == "Seasonally Adjusted" and units == "Percent"')
```
<b>Filter and Clean Data in Python</b>
1. Create dictionary of titles with "Labor Force Participation Rate for" replaced with "".
```
part_id_to_state = part_df['title'].str.replace('Labor Force Participation Rate for ','').to_dict()
```
2. Iterate over index using ```get_series``` and appending results to list of series of participationrates.
```
all_results = []
for myid in part_df.index:
    results = fred.get_series(myid)
    results = results.to_frame(name=myid)
    all_results.append(results)
    time.sleep(0.1)
```
3. Concatenate list of series into data frame.
```
part_states = pd.concat(all_results, axis=1)
```
4. Replace column names with titles from ```part_id_to_state```.
```
part_states.columns = [part_id_to_state[c] for c in part_states.columns]
```
5. Rename "the District of Columbia" column to "District of Columbia"
```
uemp_states = uemp_states.rename(columns={'the District of Columbia':'District Of Columbia'})
```
6. Set up plot, subplot and axes parameters
```
fig, axs = plt.subplots(10, 5, figsize=(30, 30), sharex=True)
axs = axs.flatten()
```
7. Set counter to ```0```
8. Iterate over columns
	- Create ```if``` ```continue``` for District of Columbia and Puerto Rico
	- Set up dual axes using ```.twinx( )```
	- Query and plot state 2020 - 2022 unemployment in blue
	- Query and plot state 2020 - 2022 participation in red
	- Set grid lines to ```False``` 
	- Set title as state name
	- Increase counter by ```1```
9. Adjust subplot parameters so the subplots fit in the figure area
```
i = 0
for state in uemp_states.columns:
    if state in ["District Of Columbia","Puerto Rico"]:
        continue
    ax2 = axs[i].twinx()
    uemp_states.query('index >= 2020 and index < 2022')[state] \
        .plot(ax=axs[i], label='Unemployment')
    part_states.query('index >= 2020 and index < 2022')[state] \
        .plot(ax=ax2, label='Participation', color=color_pal[1])
    ax2.grid(False)
    axs[i].set_title(state)
    i += 1
plt.tight_layout()
plt.show()
```
<p align="center">
<img src="https://github.com/andrew-disario/economic-data-analysis/blob/main/state_part_unemp_line.png?raw=true" height="90%" width="90%" alt="state-part-unemp-line"/>
<br />

<b>Plot California Participation & Unemployment Rates</b>
1. Set state to ```'California'```
2. Set up figure and axes paramters
3. Set up dual axes using ```.twinx( )```
4. Create new dataframe with frequency as ```'MS'```
5. Query and plot California 2020 - 2022 unemployment data in blue
6. Query and plot California 2020 - 2022 participation data in red
7. Set grid lines to ```False```
8. Set title to ```'California'```
9. Set legend labels to ```Unemployment``` and ```Participation```
10. Show plot
```
state = 'California'
fig, ax = plt.subplots(figsize=(10, 5), sharex=True)
ax2 = ax.twinx()
uemp_states2 = uemp_states.asfreq('MS')
l1 = uemp_states2.query('index >= 2020 and index < 2022')[state] \
    .plot(ax=ax, label='Unemployment')
l2 = part_states.dropna().query('index >= 2020 and index < 2022')[state] \
    .plot(ax=ax2, label='Participation', color=color_pal[1])
ax2.grid(False)
ax.set_title(state)
fig.legend(labels=['Unemployment','Participation'])
plt.show()
```
<p align="center">
<img src="https://github.com/andrew-disario/economic-data-analysis/blob/main/ca_part_unemp_line.png?raw=true" height="70%" width="70%" alt="ca-part-unemp-line"/>
<br />
