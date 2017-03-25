
## 1. Set Up


```python
import pandas as pd
import numpy as np

import matplotlib
import matplotlib.pyplot as plt
import seaborn as sns
%matplotlib inline

from scipy.misc import imread 
import re
from wordcloud import WordCloud, ImageColorGenerator, STOPWORDS
import string

import json
import warnings
warnings.filterwarnings('ignore')
```


```python
%load_ext version_information
%version_information pandas, numpy, matplotlib, seaborn, scipy, re, wordcloud, json
```




<table><tr><th>Software</th><th>Version</th></tr><tr><td>Python</td><td>3.5.2 64bit [GCC 4.2.1 Compatible Apple LLVM 4.2 (clang-425.0.28)]</td></tr><tr><td>IPython</td><td>5.1.0</td></tr><tr><td>OS</td><td>Darwin 16.1.0 x86_64 i386 64bit</td></tr><tr><td>pandas</td><td>0.18.1</td></tr><tr><td>numpy</td><td>1.11.1</td></tr><tr><td>matplotlib</td><td>1.5.3</td></tr><tr><td>seaborn</td><td>0.7.1</td></tr><tr><td>scipy</td><td>0.18.1</td></tr><tr><td>re</td><td>2.2.1</td></tr><tr><td>wordcloud</td><td>1.2.1</td></tr><tr><td>json</td><td>2.0.9</td></tr><tr><td colspan='2'>Fri Mar 24 12:51:26 2017 PDT</td></tr></table>




```python
# customized plot style
s = json.load( open("plot_style.json") )
matplotlib.rcParams.update(s)
```

## 2. Read and Clean Data 

*Dataset was downloaded from [Kaggle](https://www.kaggle.com/tovarischsukhov/southparklines)*


```python
lines = pd.read_csv('All-seasons.csv')
print(lines.shape)
lines.head()
```

    (70896, 4)





<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Season</th>
      <th>Episode</th>
      <th>Character</th>
      <th>Line</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10</td>
      <td>1</td>
      <td>Stan</td>
      <td>You guys, you guys! Chef is going away. \n</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10</td>
      <td>1</td>
      <td>Kyle</td>
      <td>Going away? For how long?\n</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10</td>
      <td>1</td>
      <td>Stan</td>
      <td>Forever.\n</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10</td>
      <td>1</td>
      <td>Chef</td>
      <td>I'm sorry boys.\n</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10</td>
      <td>1</td>
      <td>Stan</td>
      <td>Chef said he's been bored, so he joining a gro...</td>
    </tr>
  </tbody>
</table>
</div>




```python
# check na
pd.isnull(lines).sum()
```




    Season       0
    Episode      0
    Character    0
    Line         0
    dtype: int64




```python
# unique seasons
lines.Season.unique()
```




    array(['10', 'Season', '11', '12', '13', '14', '15', '16', '17', '18', '1',
           '2', '3', '4', '5', '6', '7', '8', '9'], dtype=object)




```python
# 'Season' as one of the unique seasons. something is wrong
lines[lines['Season'] == 'Season'].head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Season</th>
      <th>Episode</th>
      <th>Character</th>
      <th>Line</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3471</th>
      <td>Season</td>
      <td>Episode</td>
      <td>Character</td>
      <td>Line</td>
    </tr>
    <tr>
      <th>6950</th>
      <td>Season</td>
      <td>Episode</td>
      <td>Character</td>
      <td>Line</td>
    </tr>
    <tr>
      <th>10258</th>
      <td>Season</td>
      <td>Episode</td>
      <td>Character</td>
      <td>Line</td>
    </tr>
    <tr>
      <th>13516</th>
      <td>Season</td>
      <td>Episode</td>
      <td>Character</td>
      <td>Line</td>
    </tr>
    <tr>
      <th>16863</th>
      <td>Season</td>
      <td>Episode</td>
      <td>Character</td>
      <td>Line</td>
    </tr>
  </tbody>
</table>
</div>




```python
# remove rows not filled with correct information
lines = lines[(lines['Season'] != 'Season') | (lines['Episode'] != 'Episode') | (lines['Character'] != 'Character') | (lines['Line'] != 'Line')]
```


```python
# convert 'Season', 'Episode' to integer
lines['Season'] = lines['Season'].astype(int)
lines['Episode'] = lines['Episode'].astype(int)
```


```python
# clean the lines
# remove newline
lines['Line'] = lines['Line'].apply(lambda x: re.sub('\n', '', x))

# remove punctuations in the lines and convert to lower case -> as a new variable
exclude = list(string.punctuation)
exclude.remove("'") # not to mess up with words like I'm, He'll
lines['Line_nopunc'] = lines['Line'].apply(lambda x: ''.join([s.lower() for s in x if s not in exclude]))
```


```python
lines.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Season</th>
      <th>Episode</th>
      <th>Character</th>
      <th>Line</th>
      <th>Line_nopunc</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10</td>
      <td>1</td>
      <td>Stan</td>
      <td>You guys, you guys! Chef is going away.</td>
      <td>you guys you guys chef is going away</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10</td>
      <td>1</td>
      <td>Kyle</td>
      <td>Going away? For how long?</td>
      <td>going away for how long</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10</td>
      <td>1</td>
      <td>Stan</td>
      <td>Forever.</td>
      <td>forever</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10</td>
      <td>1</td>
      <td>Chef</td>
      <td>I'm sorry boys.</td>
      <td>i'm sorry boys</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10</td>
      <td>1</td>
      <td>Stan</td>
      <td>Chef said he's been bored, so he joining a gro...</td>
      <td>chef said he's been bored so he joining a grou...</td>
    </tr>
  </tbody>
</table>
</div>




```python
print("Total number of lines: {}".format(lines.shape[0]))
```

    Total number of lines: 70879


## 3. Data Visualization

### 3-1. Line Length (Number of Words)


```python
# add a new variable 'Word' representing number of words for each line
lines['Word'] = lines['Line_nopunc'].apply(lambda x: len(x.split()))
```


```python
# some statistics
lines.Word.describe()
```




    count    70879.000000
    mean        11.446211
    std         13.318312
    min          0.000000
    25%          4.000000
    50%          8.000000
    75%         14.000000
    max        312.000000
    Name: Word, dtype: float64




```python
# plot the distribution
p = sns.distplot(lines.Word, bins = 50, color = 'blue')
p.set(xlabel = 'Number of Words per Line', ylabel = 'Frequency')
p.set_title('Distribution of Line Length', fontweight = 'bold')
```




    <matplotlib.text.Text at 0x11c0067f0>




![png](output_19_1.png)


Over 75% of the lines are short with less than 15 words. 


```python
print("Total number of words: {}".format(lines.Word.sum()))
```

    Total number of words: 811296


### 3-2. Basic Information over Seasons


```python
# group by season
byseason = lines.groupby('Season').agg({'Episode': lambda x: x.nunique(), \
                                        'Character': lambda x: x.nunique(), \
                                        'Line': 'count', 'Word': 'sum'}).reset_index()
```


```python
byseason['Line_per_Episode'] = byseason['Line']/byseason['Episode']
byseason['Word_per_Episode'] = byseason['Word']/byseason['Episode']
```


```python
byseason.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Season</th>
      <th>Episode</th>
      <th>Word</th>
      <th>Character</th>
      <th>Line</th>
      <th>Line_per_Episode</th>
      <th>Word_per_Episode</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>13</td>
      <td>40025</td>
      <td>247</td>
      <td>4170</td>
      <td>320.769231</td>
      <td>3078.846154</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>18</td>
      <td>63352</td>
      <td>442</td>
      <td>6416</td>
      <td>356.444444</td>
      <td>3519.555556</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>17</td>
      <td>58023</td>
      <td>447</td>
      <td>5798</td>
      <td>341.058824</td>
      <td>3413.117647</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>17</td>
      <td>60250</td>
      <td>478</td>
      <td>5680</td>
      <td>334.117647</td>
      <td>3544.117647</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>14</td>
      <td>48783</td>
      <td>379</td>
      <td>4414</td>
      <td>315.285714</td>
      <td>3484.500000</td>
    </tr>
  </tbody>
</table>
</div>




```python
byseason.describe()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Season</th>
      <th>Episode</th>
      <th>Word</th>
      <th>Character</th>
      <th>Line</th>
      <th>Line_per_Episode</th>
      <th>Word_per_Episode</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>18.000000</td>
      <td>18.000000</td>
      <td>18.000000</td>
      <td>18.000000</td>
      <td>18.000000</td>
      <td>18.000000</td>
      <td>18.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>9.500000</td>
      <td>14.277778</td>
      <td>45072.000000</td>
      <td>362.333333</td>
      <td>3937.722222</td>
      <td>271.791236</td>
      <td>3141.538844</td>
    </tr>
    <tr>
      <th>std</th>
      <td>5.338539</td>
      <td>2.108960</td>
      <td>9236.274873</td>
      <td>69.473651</td>
      <td>1147.732435</td>
      <td>44.566578</td>
      <td>253.328793</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.000000</td>
      <td>10.000000</td>
      <td>32073.000000</td>
      <td>238.000000</td>
      <td>2305.000000</td>
      <td>221.500000</td>
      <td>2710.357143</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>5.250000</td>
      <td>14.000000</td>
      <td>39953.000000</td>
      <td>330.000000</td>
      <td>3269.500000</td>
      <td>236.910714</td>
      <td>2984.535714</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>9.500000</td>
      <td>14.000000</td>
      <td>42142.500000</td>
      <td>372.500000</td>
      <td>3502.000000</td>
      <td>252.028571</td>
      <td>3070.565934</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>13.750000</td>
      <td>14.750000</td>
      <td>48750.000000</td>
      <td>399.250000</td>
      <td>4369.500000</td>
      <td>311.920168</td>
      <td>3370.688235</td>
    </tr>
    <tr>
      <th>max</th>
      <td>18.000000</td>
      <td>18.000000</td>
      <td>63352.000000</td>
      <td>478.000000</td>
      <td>6416.000000</td>
      <td>356.444444</td>
      <td>3544.117647</td>
    </tr>
  </tbody>
</table>
</div>




```python
fig, ax = plt.subplots(nrows=4, ncols=1, sharex=True, sharey=False, figsize=(7, 6))

ax[0].plot(byseason.Season, byseason.Episode, marker = 'o', color = 'orange', linestyle=':', \
           linewidth = 0.5, label = 'Episodes')
ax[0].axhline(y=byseason.Episode.mean(), c="black",linewidth=2, linestyle=':', label = 'Mean')
ax[0].set_xlim([0, 20])
ax[0].set_ylim([9, 19])
ax[0].grid(False)
ax[0].legend()

ax[1].plot(byseason.Season, byseason.Character, marker = 'o', color = 'green', linestyle=':', \
        linewidth = 0.5, label = 'Characters')
ax[1].axhline(y=byseason.Character.mean(), c="black",linewidth=2, linestyle=':', label = 'Mean')
ax[1].set_xlim([0, 20])
ax[1].grid(False)
ax[1].legend()

ax[2].plot(byseason.Season, byseason.Line_per_Episode, marker = 'o', color = 'r', linestyle=':', \
        linewidth = 0.5, label = 'Lines (per Episode Average)')
ax[2].axhline(y=byseason.Line_per_Episode.mean(), c="black",linewidth=2, linestyle=':', label = 'Mean')
ax[2].set_xlim([0, 20])
ax[2].set_ylim([210, 370])
ax[2].grid(False)
ax[2].legend()

ax[3].plot(byseason.Season, byseason.Word_per_Episode, marker = 'o', color = 'b', linestyle=':', \
       linewidth = 0.5, label = 'Words (per Episode Average)')
ax[3].axhline(y=byseason.Word_per_Episode.mean(), c='black', linewidth=2, linestyle=':', label='Mean')
ax[3].set_xlim([0, 20])
ax[3].set_ylim([2600, 3700])
ax[3].grid(False)
ax[3].legend()
           
fig.text(0.5, 0, 'Season', ha='center')
fig.text(0, 0.5, 'Count', va='center', rotation='vertical')
fig.suptitle('', fontsize = 14, y = 1.02)
fig.tight_layout()
```


![png](output_27_0.png)



```python
print("Total episodes in the 18 seasons: {}".format(byseason.Episode.sum()))
```

    Total episodes in the 18 seasons: 257


### 3-3. Characters Who Appreared in All the Seasons


```python
# recuring characters
seasons = lines.Season.unique()
characters = lines.Character.unique()
print("Total number of unique characters: {}".format(len(characters)))
for season in seasons:
    character = lines[lines['Season'] == season].Character.unique()
    characters = [item for item in characters if item in character]
```

    Total number of unique characters: 3949



```python
print("Number of characters who appear in all 18 seasons: {}".format(len(characters)))
print(characters)
```

    Number of characters who appear in all 18 seasons: 16
    ['Stan', 'Kyle', 'Cartman', 'Gerald', 'Mr. Mackey', 'Randy', 'Kenny', 'Clyde', 'Woman', 'Sheila', 'Man', 'Announcer', 'Sharon', 'Wendy', 'Principal Victoria', 'Liane']


### 3-4. Characters Who Spoke Most


```python
# group by character
bycharacter = lines.groupby('Character').agg({'Line': 'count', 'Word': 'sum'}).reset_index()
bycharacter = bycharacter.sort_values(['Line', 'Word'], ascending = False)
bycharacter.head(10)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Character</th>
      <th>Word</th>
      <th>Line</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>621</th>
      <td>Cartman</td>
      <td>129680</td>
      <td>9774</td>
    </tr>
    <tr>
      <th>3384</th>
      <td>Stan</td>
      <td>68901</td>
      <td>7680</td>
    </tr>
    <tr>
      <th>1832</th>
      <td>Kyle</td>
      <td>63642</td>
      <td>7099</td>
    </tr>
    <tr>
      <th>567</th>
      <td>Butters</td>
      <td>29322</td>
      <td>2602</td>
    </tr>
    <tr>
      <th>2942</th>
      <td>Randy</td>
      <td>31087</td>
      <td>2467</td>
    </tr>
    <tr>
      <th>2314</th>
      <td>Mr. Garrison</td>
      <td>15038</td>
      <td>1002</td>
    </tr>
    <tr>
      <th>681</th>
      <td>Chef</td>
      <td>11082</td>
      <td>917</td>
    </tr>
    <tr>
      <th>1785</th>
      <td>Kenny</td>
      <td>4159</td>
      <td>881</td>
    </tr>
    <tr>
      <th>3228</th>
      <td>Sharon</td>
      <td>8600</td>
      <td>862</td>
    </tr>
    <tr>
      <th>2341</th>
      <td>Mr. Mackey</td>
      <td>10680</td>
      <td>633</td>
    </tr>
  </tbody>
</table>
</div>



Some of the top characters actually did not appear in all the seasons, such as Butters (who entered the show in Season 2) and Mr. Garrison.


```python
fig, ax = plt.subplots(nrows=1, ncols=2, figsize=(11, 4))
sns.barplot(x = 'Line', y='Character', data=bycharacter.head(10), palette = 'coolwarm_r', ax=ax[0])
ax[0].set(xlabel='Number of Lines', ylabel='')

sns.barplot(x = 'Word', y='Character', data=bycharacter.sort_values(['Word', 'Line'], ascending = False).head(10), \
            palette = 'coolwarm_r', ax=ax[1])
ax[1].set(xlabel='Number of Words', ylabel='')

fig.suptitle('Top Speakers')
```




    <matplotlib.text.Text at 0x11f23d6d8>




![png](output_35_1.png)



```python
# add new variable 'Length': average length of the lines
bycharacter['Length'] = bycharacter['Word']/bycharacter['Line']
bycharacter.head(10)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Character</th>
      <th>Word</th>
      <th>Line</th>
      <th>Length</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>621</th>
      <td>Cartman</td>
      <td>129680</td>
      <td>9774</td>
      <td>13.267853</td>
    </tr>
    <tr>
      <th>3384</th>
      <td>Stan</td>
      <td>68901</td>
      <td>7680</td>
      <td>8.971484</td>
    </tr>
    <tr>
      <th>1832</th>
      <td>Kyle</td>
      <td>63642</td>
      <td>7099</td>
      <td>8.964925</td>
    </tr>
    <tr>
      <th>567</th>
      <td>Butters</td>
      <td>29322</td>
      <td>2602</td>
      <td>11.269024</td>
    </tr>
    <tr>
      <th>2942</th>
      <td>Randy</td>
      <td>31087</td>
      <td>2467</td>
      <td>12.601135</td>
    </tr>
    <tr>
      <th>2314</th>
      <td>Mr. Garrison</td>
      <td>15038</td>
      <td>1002</td>
      <td>15.007984</td>
    </tr>
    <tr>
      <th>681</th>
      <td>Chef</td>
      <td>11082</td>
      <td>917</td>
      <td>12.085060</td>
    </tr>
    <tr>
      <th>1785</th>
      <td>Kenny</td>
      <td>4159</td>
      <td>881</td>
      <td>4.720772</td>
    </tr>
    <tr>
      <th>3228</th>
      <td>Sharon</td>
      <td>8600</td>
      <td>862</td>
      <td>9.976798</td>
    </tr>
    <tr>
      <th>2341</th>
      <td>Mr. Mackey</td>
      <td>10680</td>
      <td>633</td>
      <td>16.872038</td>
    </tr>
  </tbody>
</table>
</div>




```python
# statistics
bycharacter.describe()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Word</th>
      <th>Line</th>
      <th>Length</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>3949.000000</td>
      <td>3949.000000</td>
      <td>3949.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>205.443403</td>
      <td>17.948595</td>
      <td>10.864565</td>
    </tr>
    <tr>
      <th>std</th>
      <td>2688.393553</td>
      <td>238.722578</td>
      <td>11.164474</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>6.000000</td>
      <td>1.000000</td>
      <td>4.166667</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>21.000000</td>
      <td>2.000000</td>
      <td>8.391304</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>72.000000</td>
      <td>6.000000</td>
      <td>13.658537</td>
    </tr>
    <tr>
      <th>max</th>
      <td>129680.000000</td>
      <td>9774.000000</td>
      <td>163.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
# distribution of average line length by character
p = sns.distplot(bycharacter.Length, bins = 50, color = 'blue')
p.set(xlabel = 'Average Line Length (by Character)', ylabel = 'Frequency')
p.set_title('Distribution of Average Line Length', fontweight = 'bold')
```




    <matplotlib.text.Text at 0x11f6824e0>




![png](output_38_1.png)



```python
# plot the number of lines and the average line length for the top 5 speakers
N = 5
ind = np.arange(N) 
width = 0.35

fig, ax1 = plt.subplots()
ax1.bar(ind, bycharacter.Line[:5], width, color='r')
ax1.grid(False)
ax1.set_ylabel('Number of Lines', color='r', alpha = 0.7)
ax1.tick_params('y', colors='r')

ax2 = ax1.twinx()
ax2.bar(ind + width, bycharacter.Length[:5], width, color='b', alpha = 0.7)
ax2.grid(False)
ax2.set_ylabel('Average Line Length', color='b')
ax2.tick_params('y', colors='b')
ax2.set_xlim([-width, N])
ax2.set_xticks(ind + width)
ax2.set_xticklabels(('Cartman', 'Stan', 'Kyle', 'Butters', 'Randy'))

fig.suptitle('Top Speakers', fontsize = 14)
```




    <matplotlib.text.Text at 0x11f8cb9e8>




![png](output_39_1.png)


- Cartman, as the No.1 speaker, spoke the most lines and his lines are relatively long sentences (average line length around the 75 pencentile)
- Butters and Randy have significantly less line shares compared with Stan and Kyle, but their lines are a little longer

### 3-5. All Lines


```python
# all lines
text = ' '.join(lines.Line_nopunc)

# all words in these lines (no stopwords)
word_nostopword = [word for word in text.split() if word not in STOPWORDS]

# unique words in all lines 
print("Total number of unique words (including stopwords): {}".format(len(set(text.split()))))
print("Total number of unique words (excluding stopwords): {}".format(len(set(word_nostopword))))
```

    Total number of unique words (including stopwords): 30347
    Total number of unique words (excluding stopwords): 30168



```python
# wordcloud visualization
mask = imread('cartman.jpg') # mask
image_colors = ImageColorGenerator(mask) # custom color using mask image

wc = WordCloud(background_color = 'white', mask = mask, max_words = 3000, \
               stopwords = STOPWORDS, color_func = image_colors, scale = 1.0, random_state= 12)
wc.generate(text)
wc.to_file('cartman_wordcloud2.png')

plt.figure(figsize = (6, 6), facecolor = 'white')
plt.imshow(wc)
plt.axis('off')
```




    (-0.5, 1702.5, 1574.5, -0.5)




![png](output_43_1.png)


### 3-6. Lines of top speakers


```python
# top_speakers = ['Cartman', 'Stan', 'Kyle', 'Butters', 'Randy']
cartman = lines[lines['Character'] == 'Cartman']
cartman_byseason = cartman.groupby('Season').agg({'Line': 'count', 'Word': 'sum'}).reset_index()

stan = lines[lines['Character'] == 'Stan']
stan_byseason = stan.groupby('Season').agg({'Line': 'count', 'Word': 'sum'}).reset_index()

kyle = lines[lines['Character'] == 'Kyle']
kyle_byseason = kyle.groupby('Season').agg({'Line': 'count', 'Word': 'sum'}).reset_index()

butters = lines[lines['Character'] == 'Butters']
butters_byseason = butters.groupby('Season').agg({'Line': 'count', 'Word': 'sum'}).reset_index()

randy = lines[lines['Character'] == 'Randy']
randy_byseason = randy.groupby('Season').agg({'Line': 'count', 'Word': 'sum'}).reset_index()
```


```python
# butters did not appear in season 1
butters_byseason.loc[-1] = [1, 0, 0]
butters_byseason.index = butters_byseason.index + 1  # shifting index
butters_byseason = butters_byseason.sort()  # sorting by index
```


```python
# plot percent of words spoken by top characters in each season
plt.plot(cartman_byseason.Season, cartman_byseason.Word/byseason.Word, marker = 'o', color = 'c', linestyle=':', \
        linewidth = 0.5, markersize = 5, label = 'Cartman')
plt.plot(stan_byseason.Season, stan_byseason.Word/byseason.Word, marker = 'o', color = 'purple', linestyle=':', \
        linewidth = 0.5, markersize = 5, label = 'Stan')
plt.plot(kyle_byseason.Season, kyle_byseason.Word/byseason.Word, marker = 'o', color = 'green', linestyle=':', \
        linewidth = 0.5, markersize = 5, label = 'Kyle')
plt.plot(butters_byseason.Season, butters_byseason.Word/byseason.Word, marker = 'o', color = 'orange', linestyle=':', \
        linewidth = 0.5, markersize = 5, label = 'Butters')
plt.plot(randy_byseason.Season, randy_byseason.Word/byseason.Word, marker = 'o', color = 'black', linestyle=':', \
        linewidth = 0.5, markersize = 5, label = 'Randy')

plt.xlim([0, 20])
plt.ylim([-0.02, 0.22])
plt.xlabel('Season')
plt.ylabel('Word Share')
plt.title('Word Shares of Top Speakers by Season', fontweight = 'bold')
plt.legend(bbox_to_anchor=(1, 0.75), loc='upper left', ncol=1)
```




    <matplotlib.legend.Legend at 0x1204b0fd0>




![png](output_47_1.png)



```python
# plot line length of top characters in each season
plt.plot(cartman_byseason.Season, cartman_byseason.Word/cartman_byseason.Line, marker = 'o', color = 'c', linestyle=':', \
        linewidth = 0.5, markersize = 5, label = 'Cartman')
plt.plot(stan_byseason.Season, stan_byseason.Word/stan_byseason.Line, marker = 'o', color = 'purple', linestyle=':', \
        linewidth = 0.5, markersize = 5, label = 'Stan')
plt.plot(kyle_byseason.Season, kyle_byseason.Word/kyle_byseason.Line, marker = 'o', color = 'green', linestyle=':', \
        linewidth = 0.5, markersize = 5, label = 'Kyle')
plt.plot(butters_byseason.Season, butters_byseason.Word/(butters_byseason.Line + 1), marker = 'o', color = 'orange', linestyle=':', \
        linewidth = 0.5, markersize = 5, label = 'Butters')
plt.plot(randy_byseason.Season, randy_byseason.Word/randy_byseason.Line, marker = 'o', color = 'black', linestyle=':', \
        linewidth = 0.5, markersize = 5, label = 'Randy')

plt.xlim([0, 20])
plt.ylim([-0.5, 20.5])
plt.xlabel('Season')
plt.ylabel('Average Line Length')
plt.title('Line Length of Top Speakers by Season', fontweight = 'bold')
plt.legend(bbox_to_anchor=(1, 0.75), loc='upper left', ncol=1)
```




    <matplotlib.legend.Legend at 0x1206811d0>




![png](output_48_1.png)


Cartman remained to be No. 1 speaker throughout the seasons. Stan and Kyle started off similarly like Cartman but decreases later. Butters and Randy did not speak much in the first 4 seasons but they rise to be top speakers and have similar word shares as Stan and Kyle.


```python
# What did butters say in season 2?
butters[butters['Season'] == 2]['Line']
```




    35651         Me, too!
    35964    Pass this up.
    Name: Line, dtype: object




```python
# who else are top speakers when the show started (season 1)?
bycharacter_s1 = lines[lines['Season'] == 1].groupby('Character').agg({'Line': 'count', 'Word': 'sum'}).reset_index()
bycharacter_s1 = bycharacter_s1.sort_values(['Line', 'Word'], ascending = False)
bycharacter_s1.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Character</th>
      <th>Word</th>
      <th>Line</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>209</th>
      <td>Stan</td>
      <td>4934</td>
      <td>651</td>
    </tr>
    <tr>
      <th>33</th>
      <td>Cartman</td>
      <td>6238</td>
      <td>624</td>
    </tr>
    <tr>
      <th>111</th>
      <td>Kyle</td>
      <td>4296</td>
      <td>555</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Chef</td>
      <td>3057</td>
      <td>251</td>
    </tr>
    <tr>
      <th>142</th>
      <td>Mr. Garrison</td>
      <td>2898</td>
      <td>212</td>
    </tr>
  </tbody>
</table>
</div>



### 3-7. Words by Characters

#### Word frequency


```python
# combine lines by character
character_lines = lines.groupby('Character')['Line_nopunc'].apply(list).reset_index()
character_lines['Line_nopunc'] = character_lines['Line_nopunc'].apply(lambda x: ' '.join(x))
```


```python
# put lines together into a text file for each top speaker
cartman_text = ''.join(character_lines[character_lines['Character'] == 'Cartman']['Line_nopunc'])
stan_text = ''.join(character_lines[character_lines['Character'] == 'Stan']['Line_nopunc'])
kyle_text = ''.join(character_lines[character_lines['Character'] == 'Kyle']['Line_nopunc'])
butters_text = ''.join(character_lines[character_lines['Character'] == 'Butters']['Line_nopunc'])
randy_text = ''.join(character_lines[character_lines['Character'] == 'Randy']['Line_nopunc'])
```


```python
from collections import Counter

# mostly spoken words for the top speakers (exclude stopwords)
# cartman 
cartman_word = [w for w in cartman_text.split()]
cartman_nostopword = [w for w in cartman_word if w not in list(STOPWORDS)]
cartman_mostfreq = Counter(cartman_nostopword).most_common(20)
cartman_df = pd.DataFrame(cartman_mostfreq, columns = ['Word', 'Count'])

# stan
stan_word = [w for w in stan_text.split()]
stan_nostopword = [w for w in stan_word if w not in list(STOPWORDS)]
stan_mostfreq = Counter(stan_nostopword).most_common(20)
stan_df = pd.DataFrame(stan_mostfreq, columns = ['Word', 'Count'])

# kyle
kyle_word = [w for w in kyle_text.split()]
kyle_nostopword = [w for w in kyle_word if w not in list(STOPWORDS)]
kyle_mostfreq = Counter(kyle_nostopword).most_common(20)
kyle_df = pd.DataFrame(kyle_mostfreq, columns = ['Word', 'Count'])

# butters
butters_word = [w for w in butters_text.split()]
butters_nostopword = [w for w in butters_word if w not in list(STOPWORDS)]
butters_mostfreq = Counter(butters_nostopword).most_common(20)
butters_df = pd.DataFrame(butters_mostfreq, columns = ['Word', 'Count'])

# randy
randy_word = [w for w in randy_text.split()]
randy_nostopword = [w for w in randy_word if w not in list(STOPWORDS)]
randy_mostfreq = Counter(randy_nostopword).most_common(20)
randy_df = pd.DataFrame(randy_mostfreq, columns = ['Word', 'Count'])
```


```python
fig = plt.figure(figsize=(6, 5.7), facecolor = 'white')
ax = fig.add_subplot(111)

ax.text(0.075, 0, '\n'.join(cartman_df.Word), bbox={'facecolor': 'red', 'edgecolor': 'black', 'alpha': 0.3, 'pad':5})
ax.text(0.075, 0.95, 'Cartman', fontweight='bold')
ax.text(0.275, 0, '\n'.join(stan_df.Word), bbox={'facecolor': 'red', 'edgecolor': 'black', 'alpha': 0.3, 'pad':5})
ax.text(0.275, 0.95, 'Stan', fontweight='bold')
ax.text(0.475, 0, '\n'.join(kyle_df.Word), bbox={'facecolor': 'red', 'edgecolor': 'black', 'alpha': 0.3, 'pad':5})
ax.text(0.475, 0.95, 'Kyle', fontweight='bold')
ax.text(0.675, 0, '\n'.join(butters_df.Word), bbox={'facecolor': 'red', 'edgecolor': 'black', 'alpha': 0.3, 'pad':5})
ax.text(0.675, 0.95, 'Butters', fontweight='bold')
ax.text(0.875, 0, '\n'.join(randy_df.Word), bbox={'facecolor': 'red', 'edgecolor': 'black', 'alpha': 0.3, 'pad':5})
ax.text(0.875, 0.95, 'Randy', fontweight='bold')

fig.suptitle('Top 20 Most Frequent Words', fontweight='bold')
plt.axis('off')
```




    (0.0, 1.0, 0.0, 1.0)




![png](output_57_1.png)


#### TF IDF


```python
def top_tfidf_feats(row, features, top_n=20):
    ''' Get top n tfidf values in row and return them with their corresponding feature names.'''
    topn_ids = np.argsort(row)[::-1][:top_n]
    top_feats = [(features[i], row[i]) for i in topn_ids]
    df = pd.DataFrame(top_feats)
    df.columns = ['feature', 'tfidf']
    return df

def top_feats_in_doc(vec, features, row_id, top_n=20):
    ''' Top tfidf features in specific document (matrix row) '''
    row = np.squeeze(vec[row_id].toarray())
    return top_tfidf_feats(row, features, top_n)
```


```python
from sklearn.feature_extraction.text import TfidfVectorizer
vec_1g = TfidfVectorizer(ngram_range=(1, 1), stop_words=STOPWORDS)
vec_2g = TfidfVectorizer(ngram_range=(2, 3), stop_words=STOPWORDS)

doc = character_lines.Line_nopunc

vec_1g.fit(doc)
features_1g = vec_1g.get_feature_names()
vec1 = vec_1g.transform([' '.join(cartman_nostopword)] + [' '.join(stan_nostopword)] + \
                        [' '.join(kyle_nostopword)] + [' '.join(butters_nostopword)] + \
                        [' '.join(randy_nostopword)])

vec_2g.fit(doc)
features_2g = vec_2g.get_feature_names()
vec2 = vec_2g.transform([' '.join(cartman_nostopword)] + [' '.join(stan_nostopword)] + \
                        [' '.join(kyle_nostopword)] + [' '.join(butters_nostopword)] + \
                        [' '.join(randy_nostopword)])
```


```python
# find out words with the highest tfidf
cartman_top1g = top_feats_in_doc(vec1, features_1g, row_id=0, top_n=20)
stan_top1g = top_feats_in_doc(vec1, features_1g, row_id=1, top_n=20)
kyle_top1g = top_feats_in_doc(vec1, features_1g, row_id=2, top_n=20)
butters_top1g = top_feats_in_doc(vec1, features_1g, row_id=3, top_n=20)
randy_top1g = top_feats_in_doc(vec1, features_1g, row_id=4, top_n=20)

cartman_top2g = top_feats_in_doc(vec2, features_2g, row_id=0, top_n=20)
stan_top2g = top_feats_in_doc(vec2, features_2g, row_id=1, top_n=20)
kyle_top2g = top_feats_in_doc(vec2, features_2g, row_id=2, top_n=20)
butters_top2g = top_feats_in_doc(vec2, features_2g, row_id=3, top_n=20)
randy_top2g = top_feats_in_doc(vec2, features_2g, row_id=4, top_n=20)
```


```python
fig = plt.figure(figsize=(6, 5.7), facecolor = 'white')
ax = fig.add_subplot(111)

ax.text(0.075, 0, '\n'.join(cartman_top1g.feature), bbox={'facecolor': 'skyblue', 'edgecolor': 'black', 'alpha': 0.5, 'pad':5})
ax.text(0.075, 0.95, 'Cartman', fontweight='bold')
ax.text(0.275, 0, '\n'.join(stan_top1g.feature), bbox={'facecolor': 'skyblue', 'edgecolor': 'black', 'alpha': 0.5, 'pad':5})
ax.text(0.275, 0.95, 'Stan', fontweight='bold')
ax.text(0.475, 0, '\n'.join(kyle_top1g.feature), bbox={'facecolor': 'skyblue', 'edgecolor': 'black', 'alpha': 0.5, 'pad':5})
ax.text(0.475, 0.95, 'Kyle', fontweight='bold')
ax.text(0.675, 0, '\n'.join(butters_top1g.feature), bbox={'facecolor': 'skyblue', 'edgecolor': 'black', 'alpha': 0.5, 'pad':5})
ax.text(0.675, 0.95, 'Butters', fontweight='bold')
ax.text(0.875, 0, '\n'.join(randy_top1g.feature), bbox={'facecolor': 'skyblue', 'edgecolor': 'black', 'alpha': 0.5, 'pad':5})
ax.text(0.875, 0.95, 'Randy', fontweight='bold')

fig.suptitle('Top 20 Most Important Words', fontweight='bold')
plt.axis('off')
```




    (0.0, 1.0, 0.0, 1.0)




![png](output_62_1.png)



```python
fig = plt.figure(figsize=(10, 5.7), facecolor = 'white')
ax = fig.add_subplot(111)

ax.text(0.0, 0, '\n'.join(cartman_top2g.feature), bbox={'facecolor': 'skyblue', 'edgecolor': 'black', 'alpha': 0.5, 'pad':5})
ax.text(0.0, 0.95, 'Cartman', fontweight='bold')
ax.text(0.19, 0, '\n'.join(stan_top2g.feature), bbox={'facecolor': 'skyblue', 'edgecolor': 'black', 'alpha': 0.5, 'pad':5})
ax.text(0.19, 0.95, 'Stan', fontweight='bold')
ax.text(0.375, 0, '\n'.join(kyle_top2g.feature), bbox={'facecolor': 'skyblue', 'edgecolor': 'black', 'alpha': 0.5, 'pad':5})
ax.text(0.375, 0.95, 'Kyle', fontweight='bold')
ax.text(0.625, 0, '\n'.join(butters_top2g.feature), bbox={'facecolor': 'skyblue', 'edgecolor': 'black', 'alpha': 0.5, 'pad':5})
ax.text(0.625, 0.95, 'Butters', fontweight='bold')
ax.text(0.82, 0, '\n'.join(randy_top2g.feature), bbox={'facecolor': 'skyblue', 'edgecolor': 'black', 'alpha': 0.5, 'pad':5})
ax.text(0.82, 0.95, 'Randy', fontweight='bold')

fig.suptitle('Top 20 Most Important Phrases', fontweight='bold')
plt.axis('off')
```




    (0.0, 1.0, 0.0, 1.0)




![png](output_63_1.png)



```python
# find some special bigrams (they are more likely to form bigram than as individual words) 
from sklearn.feature_extraction.text import CountVectorizer

def get_count(n):
    '''return a dictionary with n-gram and their counts'''
    vect = CountVectorizer(stop_words='english', ngram_range=(n, n))
    counts = vect.fit_transform(doc)
    gram = vect.get_feature_names()
    counts_list = counts.sum(axis = 0)
    counts_list = np.asarray(counts_list).reshape(-1)
    gram_count = dict(zip(gram, counts_list))
    
    return gram_count

count_1g = get_count(1)
count_2g = get_count(2)

import operator

def find_special(count1, count2):
    '''return the top 20 special bigrams'''
    ratio = {}
    for bigram in count2:
        words = bigram.split()
        k1 = words[0]
        k2 = words[1]
        if k1 in count1 and k2 in count1:
            ratio[bigram] = float(count2[bigram])/((count1[k1] + 50)*(count1[k2] + 50)) # bayesian smoothing
    sorted_ratio = sorted(ratio.items(), key=operator.itemgetter(1), reverse=True)
    return sorted_ratio[:20]

find_special(count_1g, count_2g)
```




    [('mel gibson', 0.0047743055555555559),
     ('casa bonita', 0.0047332185886402754),
     ('trapper keeper', 0.0046474035158617904),
     ('prime minister', 0.0045582047685834501),
     ('kathie lee', 0.0044973544973544973),
     ('global warming', 0.0044532409698169219),
     ('los angeles', 0.0043186895011169029),
     ('rabble rabble', 0.0043044077134986227),
     ('cheesy poofs', 0.0043010752688172043),
     ('barbra streisand', 0.004097818902842036),
     ('meow meow', 0.003955895968352832),
     ('founding fathers', 0.0038377192982456138),
     ('phil collins', 0.0037071362372567192),
     ('san francisco', 0.0036910457963089541),
     ('pinewood derby', 0.0035502958579881655),
     ('mintberry crunch', 0.0035035035035035035),
     ('biggie smalls', 0.003472892147404978),
     ('okama gamesphere', 0.003383084577114428),
     ('customer service', 0.0033663366336633663),
     ('christopher reeve', 0.003335804299481097)]



#### What did Kenny say


```python
kenny_text = ''.join(character_lines[character_lines['Character'] == 'Kenny']['Line_nopunc'])
```


```python
mask = imread('kenny.jpg') # mask
image_colors = ImageColorGenerator(mask) # custom color using mask image

wc = WordCloud(background_color = 'white', mask = mask, max_words = 3000, \
               stopwords = STOPWORDS, color_func = image_colors, scale = 1.0, random_state= 10)
wc.generate(kenny_text)
wc.to_file('kenny_wordcloud.png')

plt.figure(figsize = (6, 6), facecolor = 'white')
plt.imshow(wc)
plt.axis('off')
```




    (-0.5, 354.5, 478.5, -0.5)




![png](output_67_1.png)



```python
kenny_word = [w for w in kenny_text.split()]
kenny_nostopword = [w for w in kenny_word if w not in list(STOPWORDS)]

vec1_k = vec_1g.transform([' '.join(kenny_nostopword)])
vec2_k = vec_2g.transform([' '.join(kenny_nostopword)])
kenny_top1g = top_feats_in_doc(vec1_k, features_1g, row_id=0, top_n=20)
kenny_top2g = top_feats_in_doc(vec2_k, features_2g, row_id=0, top_n=20)
```


```python
kenny_top1g
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>feature</th>
      <th>tfidf</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>yeah</td>
      <td>0.434546</td>
    </tr>
    <tr>
      <th>1</th>
      <td>fuck</td>
      <td>0.282016</td>
    </tr>
    <tr>
      <th>2</th>
      <td>woohoo</td>
      <td>0.278473</td>
    </tr>
    <tr>
      <th>3</th>
      <td>dude</td>
      <td>0.246265</td>
    </tr>
    <tr>
      <th>4</th>
      <td>guys</td>
      <td>0.233252</td>
    </tr>
    <tr>
      <th>5</th>
      <td>hey</td>
      <td>0.233143</td>
    </tr>
    <tr>
      <th>6</th>
      <td>oh</td>
      <td>0.215685</td>
    </tr>
    <tr>
      <th>7</th>
      <td>fucking</td>
      <td>0.162086</td>
    </tr>
    <tr>
      <th>8</th>
      <td>okay</td>
      <td>0.135240</td>
    </tr>
    <tr>
      <th>9</th>
      <td>huh</td>
      <td>0.125770</td>
    </tr>
    <tr>
      <th>10</th>
      <td>uh</td>
      <td>0.120562</td>
    </tr>
    <tr>
      <th>11</th>
      <td>fuckin</td>
      <td>0.110786</td>
    </tr>
    <tr>
      <th>12</th>
      <td>cartman</td>
      <td>0.099954</td>
    </tr>
    <tr>
      <th>13</th>
      <td>kyle</td>
      <td>0.086085</td>
    </tr>
    <tr>
      <th>14</th>
      <td>awesome</td>
      <td>0.085089</td>
    </tr>
    <tr>
      <th>15</th>
      <td>know</td>
      <td>0.082031</td>
    </tr>
    <tr>
      <th>16</th>
      <td>gotta</td>
      <td>0.079955</td>
    </tr>
    <tr>
      <th>17</th>
      <td>ow</td>
      <td>0.075277</td>
    </tr>
    <tr>
      <th>18</th>
      <td>io</td>
      <td>0.074782</td>
    </tr>
    <tr>
      <th>19</th>
      <td>ha</td>
      <td>0.074569</td>
    </tr>
  </tbody>
</table>
</div>




```python
kenny_top2g
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>feature</th>
      <th>tfidf</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>hey guys</td>
      <td>0.141637</td>
    </tr>
    <tr>
      <th>1</th>
      <td>yeah yeah</td>
      <td>0.113497</td>
    </tr>
    <tr>
      <th>2</th>
      <td>uh huh</td>
      <td>0.095976</td>
    </tr>
    <tr>
      <th>3</th>
      <td>dude fuck</td>
      <td>0.085868</td>
    </tr>
    <tr>
      <th>4</th>
      <td>yeah totally</td>
      <td>0.074966</td>
    </tr>
    <tr>
      <th>5</th>
      <td>fuck yeah</td>
      <td>0.073601</td>
    </tr>
    <tr>
      <th>6</th>
      <td>woohoo yeah</td>
      <td>0.071321</td>
    </tr>
    <tr>
      <th>7</th>
      <td>con te</td>
      <td>0.069687</td>
    </tr>
    <tr>
      <th>8</th>
      <td>oh god</td>
      <td>0.066066</td>
    </tr>
    <tr>
      <th>9</th>
      <td>purincessu kenni</td>
      <td>0.063784</td>
    </tr>
    <tr>
      <th>10</th>
      <td>ha ha</td>
      <td>0.062374</td>
    </tr>
    <tr>
      <th>11</th>
      <td>fuck dude</td>
      <td>0.055750</td>
    </tr>
    <tr>
      <th>12</th>
      <td>ha ha ha</td>
      <td>0.053968</td>
    </tr>
    <tr>
      <th>13</th>
      <td>uh oh</td>
      <td>0.053899</td>
    </tr>
    <tr>
      <th>14</th>
      <td>yeah fuck</td>
      <td>0.051027</td>
    </tr>
    <tr>
      <th>15</th>
      <td>know yeah</td>
      <td>0.049977</td>
    </tr>
    <tr>
      <th>16</th>
      <td>hey hey</td>
      <td>0.047844</td>
    </tr>
    <tr>
      <th>17</th>
      <td>nuh uh</td>
      <td>0.047547</td>
    </tr>
    <tr>
      <th>18</th>
      <td>yeah woohoo</td>
      <td>0.044345</td>
    </tr>
    <tr>
      <th>19</th>
      <td>huh uh</td>
      <td>0.044345</td>
    </tr>
  </tbody>
</table>
</div>


