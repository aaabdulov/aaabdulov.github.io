# Phrase scoring

## Assignment

For each scrape find 10 most prominent phrases.
Consider phrases up to 4 words.
### Dataset

[scrapes.csv](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/610644d8-ce87-408f-8c25-84268b3ca151/scrapes.csv)

### Dataset description

This dataset contains data from 14 different scrapes. Each scrape has multiple crawled pages. Each crawled page has multiple words.

- `scrape` - collection of crawled pages which were returned in the Google SERP
- `scrape_keyword` - keyword used in given Google Search
- `words` - vector of words extracted from given crawled page
- `scores` - vector of integer word prominence scores. Each word should have a corresponding score.

## Solution

Let's load the data after from the working directory


```python
import pandas as pd
scrapes=pd.read_csv('scrapes.csv')
scrapes.head()
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
      <th>scrape_id</th>
      <th>scrape_keyword</th>
      <th>crawled_page_id</th>
      <th>crawled_page_url</th>
      <th>words</th>
      <th>scores</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>244568</td>
      <td>seo course 2020</td>
      <td>3242198</td>
      <td>https://www.quicksprout.com/best-seo-courses-a...</td>
      <td>["our", "content", "is", "reader", "supported"...</td>
      <td>[4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, ...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>244568</td>
      <td>seo course 2020</td>
      <td>3242200</td>
      <td>https://digitaldefynd.com/best-seo-courses-tra...</td>
      <td>["skip", "to", "content", "trending", "10", "b...</td>
      <td>[3, 3, 3, 8, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>244568</td>
      <td>seo course 2020</td>
      <td>3242201</td>
      <td>https://www.trumplearning.com/best-seo-course-...</td>
      <td>["toggle", "navigation", "contact", "us", "hom...</td>
      <td>[6, 6, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>244568</td>
      <td>seo course 2020</td>
      <td>3242199</td>
      <td>https://ippei.com/best-seo-course/</td>
      <td>["currently", "set", "to", "index", "currently...</td>
      <td>[0, 0, 0, 0, 0, 0, 0, 0, 10, 10, 10, 7, 7, 7, ...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>244568</td>
      <td>seo course 2020</td>
      <td>3242195</td>
      <td>https://www.searchenginejournal.com/best-free-...</td>
      <td>["seo", "all", "seo", "ask", "an", "seo", "beg...</td>
      <td>[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ...</td>
    </tr>
  </tbody>
</table>
</div>



Constants used in the program below:


```python
window=4 #4-words phrases
top=10 #top 10 phrases
```

Dropping rows with no words and reseting the index to avoid gaps:


```python
scrapes=scrapes[scrapes.words!='[]']
scrapes=scrapes.reset_index(drop=True)
```

Removing unnecessary symbols so that we can convert data to another data types


```python
scrapes['words']=scrapes['words'].str.strip('["]')
scrapes['scores']=scrapes['scores'].str.strip('[]')
scrapes.head()
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
      <th>scrape_id</th>
      <th>scrape_keyword</th>
      <th>crawled_page_id</th>
      <th>crawled_page_url</th>
      <th>words</th>
      <th>scores</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>244568</td>
      <td>seo course 2020</td>
      <td>3242198</td>
      <td>https://www.quicksprout.com/best-seo-courses-a...</td>
      <td>our", "content", "is", "reader", "supported", ...</td>
      <td>4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>244568</td>
      <td>seo course 2020</td>
      <td>3242200</td>
      <td>https://digitaldefynd.com/best-seo-courses-tra...</td>
      <td>skip", "to", "content", "trending", "10", "bes...</td>
      <td>3, 3, 3, 8, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>244568</td>
      <td>seo course 2020</td>
      <td>3242201</td>
      <td>https://www.trumplearning.com/best-seo-course-...</td>
      <td>toggle", "navigation", "contact", "us", "home"...</td>
      <td>6, 6, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>244568</td>
      <td>seo course 2020</td>
      <td>3242199</td>
      <td>https://ippei.com/best-seo-course/</td>
      <td>currently", "set", "to", "index", "currently",...</td>
      <td>0, 0, 0, 0, 0, 0, 0, 0, 10, 10, 10, 7, 7, 7, 9...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>244568</td>
      <td>seo course 2020</td>
      <td>3242195</td>
      <td>https://www.searchenginejournal.com/best-free-...</td>
      <td>seo", "all", "seo", "ask", "an", "seo", "begin...</td>
      <td>0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0...</td>
    </tr>
  </tbody>
</table>
</div>



Functions to convert words to tuples and scores to lists


```python
def Convert_words(string): 
    li = tuple(string.split('", "')) 
    return li 
def Convert_scores(string): 
    li = list(string.split(', ')) 
    return li 
```

Let's create 2 separate disctionaries for all words and scores


```python
words={}
scores={}
for i in range(0,len(scrapes.index)-1):
    words[i]=Convert_words(scrapes['words'][i])
    scores[i]=list(map(int,Convert_scores(scrapes['scores'][i])))
```

Creating a new DataFrame with phrases:


```python
column_names = ["Phrase", "Score","Scrape","Page","Keyword","URL"] #column names for the new dataframe
df = pd.DataFrame(columns = column_names) #empty dataframe
for i in range(len(scrapes.index)-1): #looping through initial 'scrapes' dataframe
    scores_series=pd.Series(scores[i]) #Creating a series of scores for each row from the 'scrapes' dataframe
    words_series=pd.Series(words[i]) #Creating a series of words for each row from the 'scrapes' dataframe
    scores_series=scores_series.rolling(window).sum().sort_values(ascending=False).head(top) #Updating a series of scores with the sums of the rolling 4 words window, sorting it descending and taking 10 top values 
    phrases={} #new dictionary or phrases scores
    for a,b in scores_series.iteritems(): 
        phrases[a]=words[i][a-(window-1):a+1] #populating a dictionary of phrases and scores' indexes for a specific row from 'scrapes' dataframe
    phrases_series=pd.Series(phrases) #we need to make Series for phrases in order to populate it to the dataframe
    scrapes_results = { 'Phrase': phrases_series, 'Score': scores_series,'Scrape':scrapes['scrape_id'][i],'Page':str(scrapes['crawled_page_id'][i]),'Keyword':scrapes['scrape_keyword'][i],'URL':scrapes['crawled_page_url'][i] } 
    result = pd.DataFrame(scrapes_results) #Dataframe with all need data for the i row
    df=df.append(result) #appending the data frame with each iteration
df.head()
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
      <th>Phrase</th>
      <th>Score</th>
      <th>Scrape</th>
      <th>Page</th>
      <th>Keyword</th>
      <th>URL</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>134</th>
      <td>(seo, courses, and, guides)</td>
      <td>80.0</td>
      <td>244568</td>
      <td>3242198</td>
      <td>seo course 2020</td>
      <td>https://www.quicksprout.com/best-seo-courses-a...</td>
    </tr>
    <tr>
      <th>137</th>
      <td>(guides, on, the, internet)</td>
      <td>80.0</td>
      <td>244568</td>
      <td>3242198</td>
      <td>seo course 2020</td>
      <td>https://www.quicksprout.com/best-seo-courses-a...</td>
    </tr>
    <tr>
      <th>133</th>
      <td>(best, seo, courses, and)</td>
      <td>80.0</td>
      <td>244568</td>
      <td>3242198</td>
      <td>seo course 2020</td>
      <td>https://www.quicksprout.com/best-seo-courses-a...</td>
    </tr>
    <tr>
      <th>135</th>
      <td>(courses, and, guides, on)</td>
      <td>80.0</td>
      <td>244568</td>
      <td>3242198</td>
      <td>seo course 2020</td>
      <td>https://www.quicksprout.com/best-seo-courses-a...</td>
    </tr>
    <tr>
      <th>132</th>
      <td>(the, best, seo, courses)</td>
      <td>80.0</td>
      <td>244568</td>
      <td>3242198</td>
      <td>seo course 2020</td>
      <td>https://www.quicksprout.com/best-seo-courses-a...</td>
    </tr>
  </tbody>
</table>
</div>



Now when we have phrases and their scores let's combine identical phrases within a scrape for different pages:


```python
df=df.groupby(['Phrase','Scrape','Keyword','Score']).agg({'URL':', '.join,'Page':', '.join})
df=df.reset_index()
```

## Result - top 10 phrases for each scrape


```python
df=df.sort_values(by=['Scrape','Score'],ascending=False).groupby(by=["Scrape"]).head(10)
df=df.reset_index(drop=True)
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
      <th>Phrase</th>
      <th>Scrape</th>
      <th>Keyword</th>
      <th>Score</th>
      <th>URL</th>
      <th>Page</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>(and, certifications, free, and)</td>
      <td>244568</td>
      <td>seo course 2020</td>
      <td>80.0</td>
      <td>https://www.mobidea.com/academy/seo-training-c...</td>
      <td>3242192</td>
    </tr>
    <tr>
      <th>1</th>
      <td>(and, guides, on, the)</td>
      <td>244568</td>
      <td>seo course 2020</td>
      <td>80.0</td>
      <td>https://www.quicksprout.com/best-seo-courses-a...</td>
      <td>3242198</td>
    </tr>
    <tr>
      <th>2</th>
      <td>(best, seo, courses, and)</td>
      <td>244568</td>
      <td>seo course 2020</td>
      <td>80.0</td>
      <td>https://www.quicksprout.com/best-seo-courses-a...</td>
      <td>3242198</td>
    </tr>
    <tr>
      <th>3</th>
      <td>(best, seo, training, courses)</td>
      <td>244568</td>
      <td>seo course 2020</td>
      <td>80.0</td>
      <td>https://www.mobidea.com/academy/seo-training-c...</td>
      <td>3242192</td>
    </tr>
    <tr>
      <th>4</th>
      <td>(certifications, free, and, paid)</td>
      <td>244568</td>
      <td>seo course 2020</td>
      <td>80.0</td>
      <td>https://www.mobidea.com/academy/seo-training-c...</td>
      <td>3242192</td>
    </tr>
    <tr>
      <th>5</th>
      <td>(courses, and, certifications, free)</td>
      <td>244568</td>
      <td>seo course 2020</td>
      <td>80.0</td>
      <td>https://www.mobidea.com/academy/seo-training-c...</td>
      <td>3242192</td>
    </tr>
    <tr>
      <th>6</th>
      <td>(courses, and, guides, on)</td>
      <td>244568</td>
      <td>seo course 2020</td>
      <td>80.0</td>
      <td>https://www.quicksprout.com/best-seo-courses-a...</td>
      <td>3242198</td>
    </tr>
    <tr>
      <th>7</th>
      <td>(guides, on, the, internet)</td>
      <td>244568</td>
      <td>seo course 2020</td>
      <td>80.0</td>
      <td>https://www.quicksprout.com/best-seo-courses-a...</td>
      <td>3242198</td>
    </tr>
    <tr>
      <th>8</th>
      <td>(hour, guide, to, seo)</td>
      <td>244568</td>
      <td>seo course 2020</td>
      <td>80.0</td>
      <td>https://moz.com/learn/seo</td>
      <td>3242193</td>
    </tr>
    <tr>
      <th>9</th>
      <td>(one, hour, guide, to)</td>
      <td>244568</td>
      <td>seo course 2020</td>
      <td>80.0</td>
      <td>https://moz.com/learn/seo</td>
      <td>3242193</td>
    </tr>
    <tr>
      <th>10</th>
      <td>(share, credits, example, ɪgz)</td>
      <td>244566</td>
      <td>example</td>
      <td>55.0</td>
      <td>https://www.collinsdictionary.com/dictionary/e...</td>
      <td>3242179</td>
    </tr>
    <tr>
      <th>11</th>
      <td>(credits, example, ɪgz, ɑː)</td>
      <td>244566</td>
      <td>example</td>
      <td>53.0</td>
      <td>https://www.collinsdictionary.com/dictionary/e...</td>
      <td>3242179</td>
    </tr>
    <tr>
      <th>12</th>
      <td>(a1, something, that, is)</td>
      <td>244566</td>
      <td>example</td>
      <td>52.0</td>
      <td>https://dictionary.cambridge.org/us/dictionary...</td>
      <td>3242173</td>
    </tr>
    <tr>
      <th>13</th>
      <td>(entries, for, this, word)</td>
      <td>244566</td>
      <td>example</td>
      <td>52.0</td>
      <td>https://www.macmillandictionary.com/us/diction...</td>
      <td>3242175</td>
    </tr>
    <tr>
      <th>14</th>
      <td>(of, thing, that, you)</td>
      <td>244566</td>
      <td>example</td>
      <td>52.0</td>
      <td>https://www.macmillandictionary.com/us/diction...</td>
      <td>3242175</td>
    </tr>
    <tr>
      <th>15</th>
      <td>(of, things, that, it)</td>
      <td>244566</td>
      <td>example</td>
      <td>52.0</td>
      <td>https://dictionary.cambridge.org/us/dictionary...</td>
      <td>3242173</td>
    </tr>
    <tr>
      <th>16</th>
      <td>(other, entries, for, this)</td>
      <td>244566</td>
      <td>example</td>
      <td>52.0</td>
      <td>https://www.macmillandictionary.com/us/diction...</td>
      <td>3242175</td>
    </tr>
    <tr>
      <th>17</th>
      <td>(that, it, is, a)</td>
      <td>244566</td>
      <td>example</td>
      <td>52.0</td>
      <td>https://dictionary.cambridge.org/us/dictionary...</td>
      <td>3242173</td>
    </tr>
    <tr>
      <th>18</th>
      <td>(thing, that, you, are)</td>
      <td>244566</td>
      <td>example</td>
      <td>52.0</td>
      <td>https://www.macmillandictionary.com/us/diction...</td>
      <td>3242175</td>
    </tr>
    <tr>
      <th>19</th>
      <td>(things, that, it, is)</td>
      <td>244566</td>
      <td>example</td>
      <td>52.0</td>
      <td>https://dictionary.cambridge.org/us/dictionary...</td>
      <td>3242173</td>
    </tr>
    <tr>
      <th>20</th>
      <td>(what, is, content, marketing)</td>
      <td>244562</td>
      <td>content</td>
      <td>80.0</td>
      <td>https://contentmarketinginstitute.com/what-is-...</td>
      <td>3242096</td>
    </tr>
    <tr>
      <th>21</th>
      <td>(is, content, marketing, useful)</td>
      <td>244562</td>
      <td>content</td>
      <td>76.0</td>
      <td>https://contentmarketinginstitute.com/what-is-...</td>
      <td>3242096</td>
    </tr>
    <tr>
      <th>22</th>
      <td>(content, marketing, useful, content)</td>
      <td>244562</td>
      <td>content</td>
      <td>72.0</td>
      <td>https://contentmarketinginstitute.com/what-is-...</td>
      <td>3242096</td>
    </tr>
    <tr>
      <th>23</th>
      <td>(marketing, useful, content, should)</td>
      <td>244562</td>
      <td>content</td>
      <td>68.0</td>
      <td>https://contentmarketinginstitute.com/what-is-...</td>
      <td>3242096</td>
    </tr>
    <tr>
      <th>24</th>
      <td>(at, the, core, of)</td>
      <td>244562</td>
      <td>content</td>
      <td>64.0</td>
      <td>https://contentmarketinginstitute.com/what-is-...</td>
      <td>3242096</td>
    </tr>
    <tr>
      <th>25</th>
      <td>(be, at, the, core)</td>
      <td>244562</td>
      <td>content</td>
      <td>64.0</td>
      <td>https://contentmarketinginstitute.com/what-is-...</td>
      <td>3242096</td>
    </tr>
    <tr>
      <th>26</th>
      <td>(core, of, your, marketing)</td>
      <td>244562</td>
      <td>content</td>
      <td>64.0</td>
      <td>https://contentmarketinginstitute.com/what-is-...</td>
      <td>3242096</td>
    </tr>
    <tr>
      <th>27</th>
      <td>(should, be, at, the)</td>
      <td>244562</td>
      <td>content</td>
      <td>64.0</td>
      <td>https://contentmarketinginstitute.com/what-is-...</td>
      <td>3242096</td>
    </tr>
    <tr>
      <th>28</th>
      <td>(the, core, of, your)</td>
      <td>244562</td>
      <td>content</td>
      <td>64.0</td>
      <td>https://contentmarketinginstitute.com/what-is-...</td>
      <td>3242096</td>
    </tr>
    <tr>
      <th>29</th>
      <td>(useful, content, should, be)</td>
      <td>244562</td>
      <td>content</td>
      <td>64.0</td>
      <td>https://contentmarketinginstitute.com/what-is-...</td>
      <td>3242096</td>
    </tr>
    <tr>
      <th>30</th>
      <td>(pozycjonowanie, stron, internetowych, seo)</td>
      <td>244560</td>
      <td>pozycjonowanie stron</td>
      <td>80.0</td>
      <td>https://www.sempire.pl/pozycjonowanie-stron.html</td>
      <td>3242075</td>
    </tr>
    <tr>
      <th>31</th>
      <td>(stron, internetowych, seo, pozycjonowanie)</td>
      <td>244560</td>
      <td>pozycjonowanie stron</td>
      <td>73.0</td>
      <td>https://www.sempire.pl/pozycjonowanie-stron.html</td>
      <td>3242075</td>
    </tr>
    <tr>
      <th>32</th>
      <td>(pozycjonowanie, stron, w, google)</td>
      <td>244560</td>
      <td>pozycjonowanie stron</td>
      <td>68.0</td>
      <td>https://www.szymonslowik.pl/pozycjonowanie-stron/</td>
      <td>3242079</td>
    </tr>
    <tr>
      <th>33</th>
      <td>(internetowych, seo, pozycjonowanie, stron)</td>
      <td>244560</td>
      <td>pozycjonowanie stron</td>
      <td>66.0</td>
      <td>https://www.sempire.pl/pozycjonowanie-stron.html</td>
      <td>3242075</td>
    </tr>
    <tr>
      <th>34</th>
      <td>(czyli, czym, zajmujemy, się)</td>
      <td>244560</td>
      <td>pozycjonowanie stron</td>
      <td>64.0</td>
      <td>https://www.szymonslowik.pl/pozycjonowanie-stron/</td>
      <td>3242079</td>
    </tr>
    <tr>
      <th>35</th>
      <td>(czym, zajmujemy, się, na)</td>
      <td>244560</td>
      <td>pozycjonowanie stron</td>
      <td>64.0</td>
      <td>https://www.szymonslowik.pl/pozycjonowanie-stron/</td>
      <td>3242079</td>
    </tr>
    <tr>
      <th>36</th>
      <td>(faq, o, pozycjonowaniu, stron)</td>
      <td>244560</td>
      <td>pozycjonowanie stron</td>
      <td>64.0</td>
      <td>http://mateuszkozlowski.pl/</td>
      <td>3242080</td>
    </tr>
    <tr>
      <th>37</th>
      <td>(internetowego, nasi, klienci, otrzymują)</td>
      <td>244560</td>
      <td>pozycjonowanie stron</td>
      <td>64.0</td>
      <td>https://widoczni.com/pozycjonowanie-stron/</td>
      <td>3242076</td>
    </tr>
    <tr>
      <th>38</th>
      <td>(klienci, którzy, skorzystali, z)</td>
      <td>244560</td>
      <td>pozycjonowanie stron</td>
      <td>64.0</td>
      <td>https://www.szymonslowik.pl/pozycjonowanie-stron/</td>
      <td>3242079</td>
    </tr>
    <tr>
      <th>39</th>
      <td>(klienci, otrzymują, je, w)</td>
      <td>244560</td>
      <td>pozycjonowanie stron</td>
      <td>64.0</td>
      <td>https://widoczni.com/pozycjonowanie-stron/</td>
      <td>3242076</td>
    </tr>
    <tr>
      <th>40</th>
      <td>(contractors, in, milwaukee, wi)</td>
      <td>244557</td>
      <td>roofing company milwaukee</td>
      <td>80.0</td>
      <td>https://www.homeadvisor.com/c.Roofing.Milwauke...</td>
      <td>3242001</td>
    </tr>
    <tr>
      <th>41</th>
      <td>(roofing, contractors, in, milwaukee)</td>
      <td>244557</td>
      <td>roofing company milwaukee</td>
      <td>80.0</td>
      <td>https://www.homeadvisor.com/c.Roofing.Milwauke...</td>
      <td>3242001</td>
    </tr>
    <tr>
      <th>42</th>
      <td>(top, roofing, contractors, in)</td>
      <td>244557</td>
      <td>roofing company milwaukee</td>
      <td>80.0</td>
      <td>https://www.homeadvisor.com/c.Roofing.Milwauke...</td>
      <td>3242001</td>
    </tr>
    <tr>
      <th>43</th>
      <td>(in, milwaukee, wi, find)</td>
      <td>244557</td>
      <td>roofing company milwaukee</td>
      <td>72.0</td>
      <td>https://www.homeadvisor.com/c.Roofing.Milwauke...</td>
      <td>3242001</td>
    </tr>
    <tr>
      <th>44</th>
      <td>(1975, choose, care, quality)</td>
      <td>244557</td>
      <td>roofing company milwaukee</td>
      <td>68.0</td>
      <td>https://www.communityroofingandrestoration.com/</td>
      <td>3241996</td>
    </tr>
    <tr>
      <th>45</th>
      <td>(choose, care, quality, craftsmanship)</td>
      <td>244557</td>
      <td>roofing company milwaukee</td>
      <td>68.0</td>
      <td>https://www.communityroofingandrestoration.com/</td>
      <td>3241996</td>
    </tr>
    <tr>
      <th>46</th>
      <td>(contractors, since, 1975, choose)</td>
      <td>244557</td>
      <td>roofing company milwaukee</td>
      <td>68.0</td>
      <td>https://www.communityroofingandrestoration.com/</td>
      <td>3241996</td>
    </tr>
    <tr>
      <th>47</th>
      <td>(get, matched, with, top)</td>
      <td>244557</td>
      <td>roofing company milwaukee</td>
      <td>68.0</td>
      <td>https://www.angieslist.com/companylist/milwauk...</td>
      <td>3241995</td>
    </tr>
    <tr>
      <th>48</th>
      <td>(it, from, the, elements)</td>
      <td>244557</td>
      <td>roofing company milwaukee</td>
      <td>68.0</td>
      <td>https://robsroofingllc.com/</td>
      <td>3241994</td>
    </tr>
    <tr>
      <th>49</th>
      <td>(matched, with, top, roofing)</td>
      <td>244557</td>
      <td>roofing company milwaukee</td>
      <td>68.0</td>
      <td>https://www.angieslist.com/companylist/milwauk...</td>
      <td>3241995</td>
    </tr>
    <tr>
      <th>50</th>
      <td>(basics, complete, beginner, s)</td>
      <td>244555</td>
      <td>seo</td>
      <td>80.0</td>
      <td>https://www.wordstream.com/blog/ws/2015/04/30/...</td>
      <td>3241978</td>
    </tr>
    <tr>
      <th>51</th>
      <td>(beginner, s, guide, to)</td>
      <td>244555</td>
      <td>seo</td>
      <td>80.0</td>
      <td>https://www.wordstream.com/blog/ws/2015/04/30/...</td>
      <td>3241978</td>
    </tr>
    <tr>
      <th>52</th>
      <td>(complete, beginner, s, guide)</td>
      <td>244555</td>
      <td>seo</td>
      <td>80.0</td>
      <td>https://www.wordstream.com/blog/ws/2015/04/30/...</td>
      <td>3241978</td>
    </tr>
    <tr>
      <th>53</th>
      <td>(guide, to, search, engine)</td>
      <td>244555</td>
      <td>seo</td>
      <td>80.0</td>
      <td>https://www.wordstream.com/blog/ws/2015/04/30/...</td>
      <td>3241978</td>
    </tr>
    <tr>
      <th>54</th>
      <td>(is, seo, search, engine)</td>
      <td>244555</td>
      <td>seo</td>
      <td>80.0</td>
      <td>https://ahrefs.com/blog/what-is-seo/, https://...</td>
      <td>3241981, 3241975</td>
    </tr>
    <tr>
      <th>55</th>
      <td>(s, guide, to, search)</td>
      <td>244555</td>
      <td>seo</td>
      <td>80.0</td>
      <td>https://www.wordstream.com/blog/ws/2015/04/30/...</td>
      <td>3241978</td>
    </tr>
    <tr>
      <th>56</th>
      <td>(search, engine, optimization, explained)</td>
      <td>244555</td>
      <td>seo</td>
      <td>80.0</td>
      <td>https://ahrefs.com/blog/what-is-seo/</td>
      <td>3241981</td>
    </tr>
    <tr>
      <th>57</th>
      <td>(seo, basics, complete, beginner)</td>
      <td>244555</td>
      <td>seo</td>
      <td>80.0</td>
      <td>https://www.wordstream.com/blog/ws/2015/04/30/...</td>
      <td>3241978</td>
    </tr>
    <tr>
      <th>58</th>
      <td>(seo, search, engine, optimization)</td>
      <td>244555</td>
      <td>seo</td>
      <td>80.0</td>
      <td>https://ahrefs.com/blog/what-is-seo/, https://...</td>
      <td>3241981, 3241975</td>
    </tr>
    <tr>
      <th>59</th>
      <td>(to, search, engine, optimization)</td>
      <td>244555</td>
      <td>seo</td>
      <td>80.0</td>
      <td>https://www.wordstream.com/blog/ws/2015/04/30/...</td>
      <td>3241978</td>
    </tr>
    <tr>
      <th>60</th>
      <td>(2018, content, marketing, toolkit)</td>
      <td>244553</td>
      <td>content marketing planner</td>
      <td>80.0</td>
      <td>https://contentmarketinginstitute.com/2017/12/...</td>
      <td>3241957</td>
    </tr>
    <tr>
      <th>61</th>
      <td>(a, content, marketing, plan)</td>
      <td>244553</td>
      <td>content marketing planner</td>
      <td>80.0</td>
      <td>https://www.smartinsights.com/content-manageme...</td>
      <td>3241962</td>
    </tr>
    <tr>
      <th>62</th>
      <td>(build, the, perfect, content)</td>
      <td>244553</td>
      <td>content marketing planner</td>
      <td>80.0</td>
      <td>https://quuu.co/blog/content-calendar-for-2020/</td>
      <td>3241958</td>
    </tr>
    <tr>
      <th>63</th>
      <td>(content, calendar, for, 2020)</td>
      <td>244553</td>
      <td>content marketing planner</td>
      <td>80.0</td>
      <td>https://quuu.co/blog/content-calendar-for-2020/</td>
      <td>3241958</td>
    </tr>
    <tr>
      <th>64</th>
      <td>(content, marketing, plan, for)</td>
      <td>244553</td>
      <td>content marketing planner</td>
      <td>80.0</td>
      <td>https://www.smartinsights.com/content-manageme...</td>
      <td>3241962</td>
    </tr>
    <tr>
      <th>65</th>
      <td>(content, marketing, plan, how)</td>
      <td>244553</td>
      <td>content marketing planner</td>
      <td>80.0</td>
      <td>https://quuu.co/blog/content-calendar-for-2020/</td>
      <td>3241958</td>
    </tr>
    <tr>
      <th>66</th>
      <td>(content, marketing, planner, for)</td>
      <td>244553</td>
      <td>content marketing planner</td>
      <td>80.0</td>
      <td>https://keap.com/resources/tools/ultimate-cont...</td>
      <td>3241960</td>
    </tr>
    <tr>
      <th>67</th>
      <td>(content, marketing, toolkit, tips)</td>
      <td>244553</td>
      <td>content marketing planner</td>
      <td>80.0</td>
      <td>https://contentmarketinginstitute.com/2017/12/...</td>
      <td>3241957</td>
    </tr>
    <tr>
      <th>68</th>
      <td>(create, a, content, marketing)</td>
      <td>244553</td>
      <td>content marketing planner</td>
      <td>80.0</td>
      <td>https://www.smartinsights.com/content-manageme...</td>
      <td>3241962</td>
    </tr>
    <tr>
      <th>69</th>
      <td>(for, small, business, owners)</td>
      <td>244553</td>
      <td>content marketing planner</td>
      <td>80.0</td>
      <td>https://keap.com/resources/tools/ultimate-cont...</td>
      <td>3241960</td>
    </tr>
    <tr>
      <th>70</th>
      <td>(kosmetyczki, w, puławach, 2020)</td>
      <td>244525</td>
      <td>kosmetyczka puławy</td>
      <td>44.0</td>
      <td>https://www.oferteo.pl/kosmetyczki/pulawy</td>
      <td>3241133</td>
    </tr>
    <tr>
      <th>71</th>
      <td>(materiały, ze, słowem, kluczowym)</td>
      <td>244525</td>
      <td>kosmetyczka puławy</td>
      <td>44.0</td>
      <td>https://pulawy.naszemiasto.pl/tag/kosmetyczka-...</td>
      <td>3241141</td>
    </tr>
    <tr>
      <th>72</th>
      <td>(najlepsze, kosmetyczki, w, puławach)</td>
      <td>244525</td>
      <td>kosmetyczka puławy</td>
      <td>44.0</td>
      <td>https://www.oferteo.pl/kosmetyczki/pulawy</td>
      <td>3241133</td>
    </tr>
    <tr>
      <th>73</th>
      <td>(piękności, agnes, agnieszka, ostańska)</td>
      <td>244525</td>
      <td>kosmetyczka puławy</td>
      <td>44.0</td>
      <td>http://salony-kosmetyczne.com/salon/szczegoly/479</td>
      <td>3241159</td>
    </tr>
    <tr>
      <th>74</th>
      <td>(studio, piękności, agnes, agnieszka)</td>
      <td>244525</td>
      <td>kosmetyczka puławy</td>
      <td>44.0</td>
      <td>http://salony-kosmetyczne.com/salon/szczegoly/479</td>
      <td>3241159</td>
    </tr>
    <tr>
      <th>75</th>
      <td>(puławy, w, mieście, puławy)</td>
      <td>244525</td>
      <td>kosmetyczka puławy</td>
      <td>43.0</td>
      <td>https://pulawy.naszemiasto.pl/tag/kosmetyczka-...</td>
      <td>3241141</td>
    </tr>
    <tr>
      <th>76</th>
      <td>(w, puławach, 2020, 47)</td>
      <td>244525</td>
      <td>kosmetyczka puławy</td>
      <td>43.0</td>
      <td>https://www.oferteo.pl/kosmetyczki/pulawy</td>
      <td>3241133</td>
    </tr>
    <tr>
      <th>77</th>
      <td>(ze, słowem, kluczowym, kosmetyczka)</td>
      <td>244525</td>
      <td>kosmetyczka puławy</td>
      <td>43.0</td>
      <td>https://pulawy.naszemiasto.pl/tag/kosmetyczka-...</td>
      <td>3241141</td>
    </tr>
    <tr>
      <th>78</th>
      <td>(kluczowym, kosmetyczka, puławy, w)</td>
      <td>244525</td>
      <td>kosmetyczka puławy</td>
      <td>42.0</td>
      <td>https://pulawy.naszemiasto.pl/tag/kosmetyczka-...</td>
      <td>3241141</td>
    </tr>
    <tr>
      <th>79</th>
      <td>(kosmetyczka, puławy, w, mieście)</td>
      <td>244525</td>
      <td>kosmetyczka puławy</td>
      <td>42.0</td>
      <td>https://pulawy.naszemiasto.pl/tag/kosmetyczka-...</td>
      <td>3241141</td>
    </tr>
    <tr>
      <th>80</th>
      <td>(cena, 89, 00, zł)</td>
      <td>244524</td>
      <td>kosmetyczka</td>
      <td>44.0</td>
      <td>https://fabrykaform.pl/reisenthel-kosmetyczka-...</td>
      <td>3241223</td>
    </tr>
    <tr>
      <th>81</th>
      <td>(82, 80, zł, dodaj)</td>
      <td>244524</td>
      <td>kosmetyczka</td>
      <td>40.0</td>
      <td>https://taternik-sklep.pl/kosmetyczka-deuter-w...</td>
      <td>3241208</td>
    </tr>
    <tr>
      <th>82</th>
      <td>(dopasowywanie, treści, oraz, promocji)</td>
      <td>244524</td>
      <td>kosmetyczka</td>
      <td>40.0</td>
      <td>https://www.eobuwie.com.pl/akcesoria/kosmetycz...</td>
      <td>3241182</td>
    </tr>
    <tr>
      <th>83</th>
      <td>(i, wyszczuplić, twarz, u)</td>
      <td>244524</td>
      <td>kosmetyczka</td>
      <td>40.0</td>
      <td>https://gloswielkopolski.pl/chciala-powiekszyc...</td>
      <td>3241220</td>
    </tr>
    <tr>
      <th>84</th>
      <td>(kosmetyczki, z, poznania, lekarz)</td>
      <td>244524</td>
      <td>kosmetyczka</td>
      <td>40.0</td>
      <td>https://gloswielkopolski.pl/chciala-powiekszyc...</td>
      <td>3241220</td>
    </tr>
    <tr>
      <th>85</th>
      <td>(lekarz, zszokowany, zabiegiem, skończyło)</td>
      <td>244524</td>
      <td>kosmetyczka</td>
      <td>40.0</td>
      <td>https://gloswielkopolski.pl/chciala-powiekszyc...</td>
      <td>3241220</td>
    </tr>
    <tr>
      <th>86</th>
      <td>(na, dopasowywanie, treści, oraz)</td>
      <td>244524</td>
      <td>kosmetyczka</td>
      <td>40.0</td>
      <td>https://www.eobuwie.com.pl/akcesoria/kosmetycz...</td>
      <td>3241182</td>
    </tr>
    <tr>
      <th>87</th>
      <td>(poznania, lekarz, zszokowany, zabiegiem)</td>
      <td>244524</td>
      <td>kosmetyczka</td>
      <td>40.0</td>
      <td>https://gloswielkopolski.pl/chciala-powiekszyc...</td>
      <td>3241220</td>
    </tr>
    <tr>
      <th>88</th>
      <td>(twarz, u, kosmetyczki, z)</td>
      <td>244524</td>
      <td>kosmetyczka</td>
      <td>40.0</td>
      <td>https://gloswielkopolski.pl/chciala-powiekszyc...</td>
      <td>3241220</td>
    </tr>
    <tr>
      <th>89</th>
      <td>(u, kosmetyczki, z, poznania)</td>
      <td>244524</td>
      <td>kosmetyczka</td>
      <td>40.0</td>
      <td>https://gloswielkopolski.pl/chciala-powiekszyc...</td>
      <td>3241220</td>
    </tr>
    <tr>
      <th>90</th>
      <td>(00, 18, 00, sob)</td>
      <td>244510</td>
      <td>ślusarz warszawa</td>
      <td>40.0</td>
      <td>http://calodobowyslusarz.pl/</td>
      <td>3240956</td>
    </tr>
    <tr>
      <th>91</th>
      <td>(00, sob, 10, 00)</td>
      <td>244510</td>
      <td>ślusarz warszawa</td>
      <td>40.0</td>
      <td>http://calodobowyslusarz.pl/</td>
      <td>3240956</td>
    </tr>
    <tr>
      <th>92</th>
      <td>(18, 00, sob, 10)</td>
      <td>244510</td>
      <td>ślusarz warszawa</td>
      <td>40.0</td>
      <td>http://calodobowyslusarz.pl/</td>
      <td>3240956</td>
    </tr>
    <tr>
      <th>93</th>
      <td>(796, 15, 15, 15)</td>
      <td>244510</td>
      <td>ślusarz warszawa</td>
      <td>40.0</td>
      <td>http://slusarz-warszawa24h.pl/</td>
      <td>3240957</td>
    </tr>
    <tr>
      <th>94</th>
      <td>(9, 00, 18, 00)</td>
      <td>244510</td>
      <td>ślusarz warszawa</td>
      <td>40.0</td>
      <td>http://calodobowyslusarz.pl/</td>
      <td>3240956</td>
    </tr>
    <tr>
      <th>95</th>
      <td>(dorabiania, kluczy, pon, pt)</td>
      <td>244510</td>
      <td>ślusarz warszawa</td>
      <td>40.0</td>
      <td>http://calodobowyslusarz.pl/</td>
      <td>3240956</td>
    </tr>
    <tr>
      <th>96</th>
      <td>(dzwoń, 796, 15, 15)</td>
      <td>244510</td>
      <td>ślusarz warszawa</td>
      <td>40.0</td>
      <td>http://slusarz-warszawa24h.pl/</td>
      <td>3240957</td>
    </tr>
    <tr>
      <th>97</th>
      <td>(kluczy, pon, pt, 9)</td>
      <td>244510</td>
      <td>ślusarz warszawa</td>
      <td>40.0</td>
      <td>http://calodobowyslusarz.pl/</td>
      <td>3240956</td>
    </tr>
    <tr>
      <th>98</th>
      <td>(najlepsi, ślusarze, w, warszawie)</td>
      <td>244510</td>
      <td>ślusarz warszawa</td>
      <td>40.0</td>
      <td>https://www.oferteo.pl/slusarz/warszawa</td>
      <td>3240960</td>
    </tr>
    <tr>
      <th>99</th>
      <td>(pogotowie, zamkowe, warszawa, 24h)</td>
      <td>244510</td>
      <td>ślusarz warszawa</td>
      <td>40.0</td>
      <td>http://www.pogotowiezamkowe.com.pl/</td>
      <td>3240959</td>
    </tr>
    <tr>
      <th>100</th>
      <td>(a, new, era, at)</td>
      <td>244509</td>
      <td>npl</td>
      <td>44.0</td>
      <td>https://gonpl.com/</td>
      <td>3240942</td>
    </tr>
    <tr>
      <th>101</th>
      <td>(at, npl, build, your)</td>
      <td>244509</td>
      <td>npl</td>
      <td>44.0</td>
      <td>https://gonpl.com/</td>
      <td>3240942</td>
    </tr>
    <tr>
      <th>102</th>
      <td>(build, your, career, on)</td>
      <td>244509</td>
      <td>npl</td>
      <td>44.0</td>
      <td>https://gonpl.com/</td>
      <td>3240942</td>
    </tr>
    <tr>
      <th>103</th>
      <td>(career, on, solid, ground)</td>
      <td>244509</td>
      <td>npl</td>
      <td>44.0</td>
      <td>https://gonpl.com/</td>
      <td>3240942</td>
    </tr>
    <tr>
      <th>104</th>
      <td>(era, at, npl, build)</td>
      <td>244509</td>
      <td>npl</td>
      <td>44.0</td>
      <td>https://gonpl.com/</td>
      <td>3240942</td>
    </tr>
    <tr>
      <th>105</th>
      <td>(five, decades, of, expertise)</td>
      <td>244509</td>
      <td>npl</td>
      <td>44.0</td>
      <td>https://gonpl.com/</td>
      <td>3240942</td>
    </tr>
    <tr>
      <th>106</th>
      <td>(new, era, at, npl)</td>
      <td>244509</td>
      <td>npl</td>
      <td>44.0</td>
      <td>https://gonpl.com/</td>
      <td>3240942</td>
    </tr>
    <tr>
      <th>107</th>
      <td>(npl, build, your, career)</td>
      <td>244509</td>
      <td>npl</td>
      <td>44.0</td>
      <td>https://gonpl.com/</td>
      <td>3240942</td>
    </tr>
    <tr>
      <th>108</th>
      <td>(on, five, decades, of)</td>
      <td>244509</td>
      <td>npl</td>
      <td>44.0</td>
      <td>https://gonpl.com/</td>
      <td>3240942</td>
    </tr>
    <tr>
      <th>109</th>
      <td>(your, career, on, solid)</td>
      <td>244509</td>
      <td>npl</td>
      <td>44.0</td>
      <td>https://gonpl.com/</td>
      <td>3240942</td>
    </tr>
  </tbody>
</table>
</div>



Generating a csv file with the results:


```python
df.to_csv('top_10_phrases.csv')
```
