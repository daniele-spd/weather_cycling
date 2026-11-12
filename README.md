# Data Science: Weather and Cycling
Analysis of weather data from 2020 to 2025 in the province of Sondrio (Italy).
Freature engineering to obtain an hourly score, then a daily score and then season features.
PCA and clustering to aggregate municipalities.
Forecasting with ML models for 2026 predictions.
Creation of tourist-oriented dashboard to choose the best place to ride.

# Description
The main goals were:
* analyze weather behaviour of all municipalities of the province
* allow users to browse for municipalities (given a time window), or choose the best time to ride in the selected place
* use these data to promote tourism in places less associated with cycle tourism

*Dataset* = data obtained from the public API https://open-meteo.com/en/docs/historical-weather-api

*Tools* = Python, Power BI

# Example analysis
## Python
Obtaining only hours between sunrise and sunset, with a tolerance of 20 minutes
```python
valid_hours = hourly.merge(daily, on=["date", "name"], how="left")

diff_sunrise = valid_hours["time"] - valid_hours["sunrise"]
diff_sunset = valid_hours["sunset"] - valid_hours["time"]
tolerance = 20 # tolerance (in minutes) before and after sunrise/sunset

mask = ((diff_sunrise >= pd.Timedelta(0)) | (diff_sunrise.abs() <= pd.Timedelta(minutes= tolerance))) & ((diff_sunset >= pd.Timedelta(0)) | (diff_sunset.abs() <= pd.Timedelta(minutes= tolerance)))
valid_hours = valid_hours.loc[mask]
```

Feature engineering of the Temperature score, with comfort and acceptability thresholds
```python
T_low__acceptable_threshold = 5
T_high__acceptable_threshold = 30
T_low__comfort_threshold = 15
T_high__comfort_threshold = 25

T = feature_engineering["apparent_temperature"]

T_outside_acceptable = (T < T_low__acceptable_threshold) | (T > T_high__acceptable_threshold)
T_inside_comfort = (T >= T_low__comfort_threshold) & (T <= T_high__comfort_threshold)

T_intermediate = np.where(T < T_low__comfort_threshold,
                        1 - (T_low__comfort_threshold - T) / 
                            (T_low__comfort_threshold - T_low__acceptable_threshold),
                        1 - (T - T_high__comfort_threshold) / 
                            (T_high__acceptable_threshold - T_high__comfort_threshold)

)

feature_engineering["T_score"] = np.where(T_outside_acceptable, 0,
                                            np.where(T_inside_comfort, 1,
                                            T_intermediate)
)
```

Elbow method for selection of number of clusters (K-means)
```python
inertias = []

for k in range(2, 11):
    kmeans = KMeans(n_clusters=k, random_state=42, n_init=10) # n_init means that the algorithm is run 10 times with different initial positions of the clusters, and the final inertia is the lowest
    kmeans.fit(X_pca)
    inertias.append(kmeans.inertia_)

plt.plot(range(2, 11), inertias, marker='o')
plt.xlabel('Number of clusters')
plt.ylabel('Inertia')
plt.title('Elbow Method')
plt.show()
```

## Power BI 
Number of selected days
```dax
Days = 
VAR first = MIN('Calendar'[Date])
VAR last = MAX('Calendar'[Date])

RETURN
    DATEDIFF(first, last, DAY) +1
```

Filter to show only municipalities with non-null weather score
```dax
Show_municipality = 
IF(
    [Score] > 0,
    1,
    0
)
```

Dimension table (Calendar)
```dax
Calendar = 
ADDCOLUMNS(
    CALENDAR(
        DATE(2020,1,1),
        DATE(2026,12,31)
    ),
    "Year", YEAR([Date]),
    "Month", MONTH([Date]),
    "Month name", FORMAT([Date], "MMMM"),
    "Year-month", FORMAT([Date], "YYYY-MM"),
    "Date debug", FORMAT([Date], "yyyy-MM-dd")
)

```

# Insights
The purpose of the project was not to discover something, but to allow the user to explore the data.
However, some examples (found in "ride_the_data.pptx") include: 
* Even if there are 5 geographical divisions of the province, 6 weather clusters are found
* In spring and autumn, the best weather scores are achieved in the Western part of the province
* In summer, the best weather scores are achieved in lateral valleys and high-elevation places

# How to
Jupyter notebooks are sequentially ordered, except from the one for the Exploratory Data Analysis "EDA.ipynb" which can be employed in different moments of the analysis.
The main assumptions and parameters employed during the Python analysis can be found in "documentation/Python analysis.xlsx"

CSV data are in the folders "raw_data" and "processed_data". The missing CSV (in the gitignore) can be downloaded at https://drive.google.com/uc?export=download&id=12UT7-LNTVo78wKUR67AsDM3kxEhD5jw4.
The meaning of weather raw variables is in "documentation/API parameters.xlsx".

User-oriented Power BI report is "report.pbix".

An overview of the project and the corresponding results is in "ride_the_data.pptx".
