# Introduction and Usage Guidelines

This notebook is designed as a base for analysis for Trackmania's physics. 

I'm running a small web server that collects and distributes collected map data from users using the Gigachad Collector plugin. 

(The 'in' section of the API is used by the GigaChad Collector (GCC) from within Trackmania.)


```python
import pandas as pd
import requests
import matplotlib.pyplot as plt

# optional - makes plots pop out. highly recommended. disabled for rendering in git
#%matplotlib qt

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

# 'legacy' for all of my old training data 
# these frames can be hard to work with
SELECTED_MAPTAG = 'legacy' 
SELECTED_RUNKEY = 'legacy'

```


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
    <tr>
      <th>4</th>
      <td>6</td>
      <td>LEGACY</td>
      <td>legacy</td>
      <td>sgt_bigbird</td>
    </tr>
  </tbody>
</table>



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
    <tr>
      <th>1</th>
      <td>103</td>
      <td>legacy</td>
      <td>1</td>
    </tr>
  </tbody>
</table>



```python
# For 'maptag' data:
maptag_data = requests.get(f"{SERVER_PATH}/out/data/tag", params={"tag": SELECTED_MAPTAG})
# For 'runkey' data:
runkey_data = requests.get(f"{SERVER_PATH}/out/data/runkey", params={"runkey": SELECTED_RUNKEY})

# For all available data:
all_data = requests.get(f"{SERVER_PATH}/out/all", params={"runkey": SELECTED_RUNKEY})

# Some columns are hidden from these views to save data. If you want them, add "/verbose" to the end of the route. 
# Same for the 'all data' route. 

print(f"Loaded {len(maptag_data.text)} chars of maptag data, {len(runkey_data.text)} chars of runkey data")
```

    Loaded 140124821 chars of maptag data, 140124821 chars of runkey data
    


```python
# data = maptag_data
data = runkey_data
# data = all_data
```


```python
df = pd.json_normalize(data.json())

# Example: Graph of acceleration at different slide angles

## Preprocess: 

# Acceleration isn't natively provided - do .diff() for this and any other derivatives you want to calculate
# You can do a rolling average, too, depending on usage
df["acc"] = df["speed"].diff()

## Filter:

df = df[abs(df["acc"]) < 0.2]

# Use this type of call to see what surfaces are available
# Legacy is mostly wood - I have more data in .csv format, but haven't uploaded it yet

# print(df["frGroundContactMaterial"].unique())

df = df[df["flGroundContactMaterial"] == "Wood"]
df = df[df["frGroundContactMaterial"] == "Wood"]
df = df[df["rlGroundContactMaterial"] == "Wood"]
df = df[df["rrGroundContactMaterial"] == "Wood"]

## Now scatter the data 
plt.scatter(x=df["slipDir"], y=df["acc"], c=df["speed"], alpha=0.3)
plt.show()
```

    ['XXX_Null' 'Wood' 'RoadIce' 'Water' 'Concrete' 'Green']
    


    
![png](output_6_1.png)
    

