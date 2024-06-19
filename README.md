# Introduction and Usage Guidelines

This notebook is designed as a base for analysis for Trackmania's physics. 

You can view API documentation here: http://localhost:8080/swagger-ui/index.html#, in the 'out' controller section. 

(The 'in' section is what API is used by the GigaChad Collector (GCC) from within Trackmania.)


```python
import pandas as pd
import requests
import matplotlib.pyplot as plt

# optional - makes plots pop out. highly recommended
# %matplotlib qt

# no trailing slash
url_prod = "http://gigachad.justinjschmitz.com:21532";
url_dev = "http://localhost:8080";

SERVER_PATH = url_prod

SWAGGER_PATH = f"{SERVER_PATH}/swagger-ui/index.html"

print(f"Current documentation page: {SWAGGER_PATH}")

```

    Current documentation page: http://gigachad.justinjschmitz.com:21532/swagger-ui/index.html
    

## Data Selection

### "Maptags" vs "Runkeys" 

This is kind of an arbitrary convention, but:
* Map tags are for general surface use. 
* Runkeys are for specific study use. 

E.g., if you wanted to just look at how the stadium car drives on dirt, then you can pull the map tag 'stadium - dirt' and you'll have all that data. 

But, if you were doing some novel study (like trying to 2 wheel the rally car, or something) and wanted *just* that data, then make sure you set that as the the underlying runkey. That way, you're not pulling in more than you need to. 




```python
maptags = requests.get(f"{SERVER_PATH}/out/maptag")
runkeys = requests.get(f"{SERVER_PATH}/out/runkey")
maptags_df = pd.json_normalize(maptags.json())
runkeys_df = pd.json_normalize(runkeys.json())
display(maptags_df)
display(runkeys_df)

# change these values with 'tag' and 'name' respectively

SELECTED_MAPTAG = 'stadium - dirt'
SELECTED_RUNKEY = 'test2'

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
      <th>mtId</th>
      <th>mapUuid</th>
      <th>tag</th>
      <th>username</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2</td>
      <td>bu5OUM_XDFQ7w_vnhQOopNsyJQ8</td>
      <td>dirt - stadium</td>
      <td>sgt_bigbird</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3</td>
      <td>nDqOzuoWkPUJ9m74q0ihDVlTFn7</td>
      <td>dirt - snow</td>
      <td>sgt_bigbird</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4</td>
      <td>OuHxc71KtYLTkb7r6AmJcHCbbz8</td>
      <td>dirt - rally</td>
      <td>sgt_bigbird</td>
    </tr>
    <tr>
      <th>3</th>
      <td>5</td>
      <td>M92iB3BWeAsGChHVL6QqqtAnNC6</td>
      <td>dirt - desert</td>
      <td>sgt_bigbird</td>
    </tr>
  </tbody>
</table>
</div>



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
      <th>rkId</th>
      <th>name</th>
      <th>mode</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>102</td>
      <td>default</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



```python
# For 'maptag' data:
maptag_data = requests.get(f"{SERVER_PATH}/out/data/tag", params={"tag": SELECTED_MAPTAG})
# For 'runkey' data:
runkey_data = requests.get(f"{SERVER_PATH}/out/data/runkey", params={"runkey": SELECTED_RUNKEY})

print(f"Loaded {len(maptag_data.text)} chars of maptag data, {len(runkey_data.text)} chars of runkey data")
```

    Loaded 113 chars of maptag data, 116 chars of runkey data
    


```python
data = maptag_data
# data = runkey_data
df = pd.json_normalize(data.json())
```


```python
# Example: Graph of acceleration at different slide angles

## Preprocess: 

# Acceleration isn't natively provided - do .diff() for this and any other derivatives you want to calculate
# You can do a rolling average, too, depending on usage
df["acc"] = df["speed"].diff()

## Filter:

df = df[abs(df["acc"]) < 0.5]

## Now scatter the data 
plt.scatter(x=df["slipDir"], y=df["acc"], c=df["speed"], alpha=1)
plt.show()
```


    ---------------------------------------------------------------------------

    KeyError                                  Traceback (most recent call last)

    File ~\anaconda3\lib\site-packages\pandas\core\indexes\base.py:3621, in Index.get_loc(self, key, method, tolerance)
       3620 try:
    -> 3621     return self._engine.get_loc(casted_key)
       3622 except KeyError as err:
    

    File ~\anaconda3\lib\site-packages\pandas\_libs\index.pyx:136, in pandas._libs.index.IndexEngine.get_loc()
    

    File ~\anaconda3\lib\site-packages\pandas\_libs\index.pyx:163, in pandas._libs.index.IndexEngine.get_loc()
    

    File pandas\_libs\hashtable_class_helper.pxi:5198, in pandas._libs.hashtable.PyObjectHashTable.get_item()
    

    File pandas\_libs\hashtable_class_helper.pxi:5206, in pandas._libs.hashtable.PyObjectHashTable.get_item()
    

    KeyError: 'speed'

    
    The above exception was the direct cause of the following exception:
    

    KeyError                                  Traceback (most recent call last)

    Input In [5], in <cell line: 7>()
          1 # Example: Graph of acceleration at different slide angles
          2 
          3 ## Preprocess: 
          4 
          5 # Acceleration isn't natively provided - do .diff() for this and any other derivatives you want to calculate
          6 # You can do a rolling average, too, depending on usage
    ----> 7 df["acc"] = df["speed"].diff()
          9 ## Filter:
         11 df = df[abs(df["acc"]) < 0.5]
    

    File ~\anaconda3\lib\site-packages\pandas\core\frame.py:3505, in DataFrame.__getitem__(self, key)
       3503 if self.columns.nlevels > 1:
       3504     return self._getitem_multilevel(key)
    -> 3505 indexer = self.columns.get_loc(key)
       3506 if is_integer(indexer):
       3507     indexer = [indexer]
    

    File ~\anaconda3\lib\site-packages\pandas\core\indexes\base.py:3623, in Index.get_loc(self, key, method, tolerance)
       3621     return self._engine.get_loc(casted_key)
       3622 except KeyError as err:
    -> 3623     raise KeyError(key) from err
       3624 except TypeError:
       3625     # If we have a listlike key, _check_indexing_error will raise
       3626     #  InvalidIndexError. Otherwise we fall through and re-raise
       3627     #  the TypeError.
       3628     self._check_indexing_error(key)
    

    KeyError: 'speed'

