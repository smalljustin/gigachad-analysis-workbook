# Introduction and Usage Guidelines

This notebook is designed as a base for analysis for Trackmania's physics. 

I'm running a small web server that collects and distributes collected map data from users using the Gigachad Collector plugin. 

(The 'in' section of the API is used by the GigaChad Collector (GCC) from within Trackmania.)


```python
import pandas as pd
import requests
import matplotlib.pyplot as plt
import numpy as np

# optional - makes plots pop out. highly recommended. disabled for rendering in git
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

# 'legacy' for all of my old training data 
# these frames can be hard to work with
SELECTED_MAPTAG = 'tarmac - stadium' 
SELECTED_RUNKEY = 'default'

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
    <tr>
      <th>5</th>
      <td>52</td>
      <td>5Ueetpp0vqJmnj30uQNev6gRQP4</td>
      <td>tarmac - stadium</td>
      <td>sgt_bigbird</td>
    </tr>
    <tr>
      <th>6</th>
      <td>53</td>
      <td>YgwaP_ZzGgQcnUIGRtPAY3Geuk8</td>
      <td>tarmac - snow</td>
      <td>sgt_bigbird</td>
    </tr>
    <tr>
      <th>7</th>
      <td>54</td>
      <td>c3oi3jiS87yJQO9H9SybE5EgHG5</td>
      <td>tarmac - rally</td>
      <td>sgt_bigbird</td>
    </tr>
    <tr>
      <th>8</th>
      <td>55</td>
      <td>PNhVaUx13dJzCJ_vAmjE6OFi6mg</td>
      <td>tarmac - desert</td>
      <td>sgt_bigbird</td>
    </tr>
    <tr>
      <th>9</th>
      <td>102</td>
      <td>LzPfBPmW_lb2I9EDF1I40MyAfJb</td>
      <td>wood - stadium - wet, icy</td>
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

    Loaded 10723562 chars of maptag data, 31735739 chars of runkey data
    


```python
data = maptag_data
# data = runkey_data
# data = all_data

df = pd.json_normalize(data.json())

# Example: Graph of acceleration at different slide angles

## Preprocess: 

# Acceleration isn't natively provided - do .diff() for this for the derivative.
# Make sure to divide by the time quantity 'dt'.
df["acc"] = df["speed"].diff() / df["dt"]

## Filter:

df = df[abs(df["acc"]) < 0.2]

# Use this type of call to see what surfaces are available
print(df["frGroundContactMaterial"].unique())

# Example: If you want to see what the acceleration curve looks like for ice, between 40 and 60 m/s: 
# df = df[df["speed"] > 40]
# df = df[df["speed"] < 60]

selected_material = "Asphalt"

df = df[df["flGroundContactMaterial"] == selected_material]
df = df[df["frGroundContactMaterial"] == selected_material]
df = df[df["rlGroundContactMaterial"] == selected_material]
df = df[df["rrGroundContactMaterial"] == selected_material]

## Now scatter the data 
plt.scatter(x=df["slipDir"], y=df["acc"], c=df["speed"], alpha=0.1)
plt.show()
```

    ['Asphalt' 'RoadIce' 'Concrete' 'XXX_Null']
    


    
![png](output_5_1.png)
    


# Usage Example: Deriving ideal tarmac speedslide curve

For a workable example, here's exactly how to derive the ideal tarmac speedslide curve.

This method is based around finding the slip angle for each given speed that produces the highest acceleration. 

Note that the 'base' acceleration here is comparing against the baseline acceleration the car experiences with no sliding at all.


```python
# This is the profile for plastic and road. 
# Grass and dirt are similar, except for a uniform 10% reduction 
# source: https://www.youtube.com/watch?v=KfMIT5cbO2g&t=260s

# Format: 
# [(start_speed, acc), ...]

# Unit here is km/h, km/h/s 
BASE_ACCEL_RAW = [
    (100, 57),
    (200, 40),
    (400, 25),
    (1000, 20)
]

# Transforms this into meters/second, meters/second/second 
BASE_ACCEL_MS = list(map(lambda x: (x[0] / 3.6, x[1] / 3600), BASE_ACCEL_RAW))

def analyze_fs_df(df, minv=400, maxv=999, is_slow_surface=False):
    df = df[df["speed"] > (minv / 3.6)]
    df = df[df["speed"] < (maxv / 3.6)].copy()
    
    # Set the 'base' acceleration for each row. This is how fast the car would accelerate without sliding.
    df["base_accel"] = df["speed"].apply(lambda val: next(filter(lambda tup: tup[0] > val, BASE_ACCEL_MS))[1])
    
    # Use rolling average of acceleration instead of raw 
    df["acc_rolling"] = df["acc"].rolling(1).mean()
    
    # Sidespeed is easier to work with than raw angles - this is really just a conventional choice, though, so
    # it doesn't really matter. It's what I did for GCP :) 
    df["sidespeed"] = df["speed"] * np.sin(df["slipDir"])
    
    # Break into subgroups based on speed. 
    df["speed_group"] = pd.cut(df["speed"], 10)
    
    if is_slow_surface:
        df["base_accel"] *= 0.9

    max_acc = {}
    for group in df["speed_group"].unique():
        max_acc[group] = np.percentile(df[df["speed_group"] == group]["acc_rolling"], 95)   
    
    # Now max-acc has a mapping of 'speed group' -> 'max acceleration for that speed group'. 
    
    # Normalize acceleration, so it's between 0 and 1 (approximately).
    df["acc_normalized"] = df.apply(lambda row: row["acc_rolling"] / max_acc[row["speed_group"]], axis=1)
    
    # Now filter, so we're only looking at the fastest accelerations.
    at_max_acc_df = df[df["acc_normalized"] > 0.99]
    
    plt.xlabel("Speed (in m/s)")
    plt.ylabel("Sidespeed (in m/s)")
    plt.title("Ideal slide angle, for peak acceleration")
    
    # And from here, do manual analysis. Try to draw the line through the data that best represents it.
    plt.scatter(
        x=at_max_acc_df["speed"], 
        y=at_max_acc_df["sidespeed"], 
        c=at_max_acc_df["acc_normalized"],
        alpha=0.2
    )
    plt.show()
    
    # For analysis for GCP, we're not done here. We need: 
    # -> Max acceleration line 
    # -> Acceleration equal to baseline acceleration line
    # -> Zero acceleration line
    
    # Slice the dataframe accordingly to make these views, and then apply the same graphical analysis method. 

analyze_fs_df(df, minv=400, maxv=999, is_slow_surface=False)

```


    
![png](output_7_0.png)
    


# Working Section 

This area of the notebook is my in-progress projects. This will be less refined than the examples above, and also I'm not going to put as much effort into commenting and explaining things. 

If you have any questions about anything in this area, feel free to reach out on Discord and I'll get back to you when I can. 

## Icy Wet Wood - Stadium


```python
SELECTED_MAPTAG = 'wood - stadium - wet, icy'
data = requests.get(f"{SERVER_PATH}/out/data/tag", params={"tag": SELECTED_MAPTAG})
df = pd.json_normalize(data.json())

df["acc"] = df["speed"].diff() / df["dt"]

selected_material = "Wood"

df = df[df["flGroundContactMaterial"] == selected_material]
df = df[df["frGroundContactMaterial"] == selected_material]
df = df[df["rlGroundContactMaterial"] == selected_material]
df = df[df["rrGroundContactMaterial"] == selected_material]

df = df[df["flIcing01"] > 0]
df = df[df["frIcing01"] > 0]
df = df[df["rlIcing01"] > 0]
df = df[df["rrIcing01"] > 0]

df = df[abs(df["acc"]) < 0.2]

df = df.copy()
```


```python
plt.scatter(x=df["slipDir"], y=df["acc"], c=df["speed"])
plt.show()
```


    
![png](output_11_0.png)
    

