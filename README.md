# Module 6 Challenge

## NASA Data Source Analysis

This notebook analyses the CME and GST data retrieved using a NASA API.

### Requirements

#### Part 1: Request CME data from the NASA API

##### Request

* The query_url_CME is constructed to include CME, dates, and API_KEY

``` python
cme_url = base_url+CME+"?startDate="+startDate+"&endDate="+endDate+"&api_key="+NASA_API_KEY
```

* A GET request is made to retrieve results and the JSON data is stored in a variable called cme_json

``` python
cme_response = requests.get(cme_url)
```

* json.dumps, with the argument indent=4, is used to preview the first results

``` python
print(json.dumps(cme_json[0], indent=4))
```

* cme_json is converted to a Pandas DataFrame

``` python
cme_df = pd.DataFrame(cme_json)
```

##### Preparation for loop

* An empty list called expanded_rows is created 

``` python
expanded_rows = []
```

* A for loop is created to loop through the cme.index list

``` python
for entry in cme_df.index:
```
 ##### Inside the cme.index for loop

 * activityID, startTime, linkedEvents are correctly defined 

 ``` python
entry_activityID = cme_df.iloc[entry]["activityID"]
entry_startTime = cme_df.iloc[entry]["startTime"]
entry_linkedEvents = cme_df.iloc[entry]["linkedEvents"]
 ```

* An inner for loop is created to loop through the linkedEvents list

``` python
for item in entry_linkedEvents:
```

* expanded_rows list is correctly appended with all three variables

``` python
expanded_rows.append({"activityID":entry_activityID,"startTime":entry_startTime,"linkedEvents":item})
```

* A DataFrame is correctly constructed from expanded_rows

``` python
cme_df_expanded = pd.DataFrame(expanded_rows)
```

##### Function extract_activityID_from_dict

* A try-except clause is used

``` python
try:
    return input_dict["activityID"]
# Log the error or print it for debugging
except (ValueError, TypeError) as e:        
    return e
```

* activityID is correctly constructed

``` python
return input_dict["activityID"]
```

* Function extract_activityID_from_dict is correctly used with apply() and lambda

``` python
cme_df_expanded["GST_ActivityID"] = cme_df_expanded["linkedEvents"].apply(lambda x: extract_activityID_from_dict(x))
```

##### Cleaning

* The GST_ActivityID column is correctly converted to string

``` python
cme_df_expanded["GST_ActivityID"] = cme_df_expanded["GST_ActivityID"].astype("string")
```

* The startTime column is correctly converted to datetime and renamed correctly

``` python
cme_df_expanded["startTime"] = pd.to_datetime(cme_df_expanded["startTime"])
cme_df_expanded.rename(columns={"startTime":"startTime_CME","activityID":"cmeID"},inplace=True)
```

* The activityID column is renamed correctly

``` python
cme_df_expanded.rename(columns={"startTime":"startTime_CME","activityID":"cmeID"},inplace=True)
```

* The cme DataFrame is filtered to only keep rows where GST_ActivityID contains 'GST'

``` python
cme_df_expanded = cme_df_expanded[cme_df_expanded["GST_ActivityID"].str.contains("GST")].reset_index(drop=True)
```

#### Part 2: Request GST data from the NASA API 

##### Request

* The query_url_GST is constructed to include CME, dates, and API_KEY

``` python
gst_url = base_url+GST+"?startDate="+startDate+"&endDate="+endDate+"&api_key="+NASA_API_KEY
```

* A GET request is made to retrieve results and the JSON data is stored in a variable called gst_json

``` python
gst_response = requests.get(gst_url)
```

* json.dumps, with the argument indent=4, is used to preview the first results

``` python
print(json.dumps(gst_json[0],indent=4))
```

* gst_json is converted to a Pandas DataFrame

``` python
gst_df = pd.DataFrame(gst_json)
```

##### Expanding the data

* The gst DataFrame is expanded using explode() on the linkedEvents column

``` python
# gst_df.explode("linkedEvents",ignore_index=True)
gst_df_expanded = gst_df.explode("linkedEvents",ignore_index=True).dropna(subset="linkedEvents").reset_index(drop=True)
```

* The index is reset

``` python
# .reset_index(drop=True)
gst_df_expanded = gst_df.explode("linkedEvents",ignore_index=True).dropna(subset="linkedEvents").reset_index(drop=True)
```

* Missing values are dropped from the DataFrame

``` python
# .dropna(subset="linkedEvents")
gst_df_expanded = gst_df.explode("linkedEvents",ignore_index=True).dropna(subset="linkedEvents").reset_index(drop=True)
```

##### Function extract_activityID_from_dict

* Function extract_activityID_from_dict is correctly used with apply() and lambda 

``` python
gst_df_expanded["CME_ActivityID"] = gst_df_expanded["linkedEvents"].apply(lambda x: extract_activityID_from_dict(x))
```

##### Cleaning

* The CME_ActivityID column is correctly converted to string data using the supplied extract_keywords function

``` python
gst_df_expanded["CME_ActivityID"] = gst_df_expanded["CME_ActivityID"].astype("string")
```

* The startTime column is correctly converted to datetime and renamed correctly

``` python
gst_df_expanded["startTime"] = pd.to_datetime(gst_df_expanded["startTime"])
gst_df_expanded.rename(columns={"startTime":"startTime_GST"},inplace=True)
```

* The activityID column is renamed correctly

``` python
# There is no activityID column and the instructions do not state to perform this
```

* The gst DataFrame is filtered to only keep rows where GST_ActivityID contains 'CME'

``` python
gst_df_expanded = gst_df_expanded[gst_df_expanded["CME_ActivityID"].str.contains("CME")].reset_index(drop=True)
```

#### Part 3: Merge and Clean the Data for Export

* The cme and gst DataFrames are merged using gstID and CME_ActivityID for gst and GST_ActivityID and cmeID for cme

``` python
data_joined = cme_df_expanded.merge(gst_df_expanded,left_on=["GST_ActivityID","cmeID"],right_on=["gstID", "CME_ActivityID"])
```

* It is shown with info or shape that the number of rows matches both individual DataFrames

``` python
print(f"cme_df_expanded row count: {cme_df_expanded.shape[0]}")
print(f"gst_df_expanded row count: {gst_df_expanded.shape[0]}")
print(f"data_joined     row count: {data_joined.shape[0]}")
```

* A new column is created that shows the difference between startTime_GST and startTime_CME called 'timeDiff'

``` python
data_joined["timeDiff"] = abs(data_joined["startTime_GST"] - data_joined["startTime_CME"])
```

* describe() is used to show the mean and median

``` python
data_joined.describe().loc[["mean","50%"]]
```

* The DataFrame is exported to a CSV file without the index

``` python
data_joined.to_csv("./output/outputfile.csv",index=False)
```
