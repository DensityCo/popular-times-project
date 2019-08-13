# Popular Times Project

<img src="https://raw.githubusercontent.com/DensityCo/popular-times-project/master/imgs/density.png" width="75%">

Density is an anonymous people counting system. Companies install our technology to count people inside buildings. We usually sell to large corporations but, as it happens, we’ve found ourselves in a number of locations that also have Google Popular Times enabled. So we decided to use one to estimate the accuracy of the other.

![Google Popular Times Feature](https://raw.githubusercontent.com/DensityCo/popular-times-project/master/imgs/popular-times-phone.png)

You can read about our findings here: https://data.density.io

This repo contains all the data we collected.

### Raw Data

Data is available for the following 15 locations (actual name / address not provided):

```
Airport Lounge 1
Airport Lounge 2
Bar 1
Bar 2
Coffee Shop
College Dining Hall
Furniture Store 1
Furniture Store 2
Furniture Store 3
Furniture Store 4
Furniture Store 5
Furniture Store 6
Furniture Store 7
Furniture Store 8
Furniture Store 9
```

Each location has the following: `capacity`, `timezone`, `density`, `google_real_time`, and `google_forecast`

```python
import json

with open('data.json', 'r') as data_file:
    data = json.load(data_file)
    
location = data['College Dining Hall']
print(location['capacity'])  # 1139
print(location['timezone'])  # US/Eastern
```

**Density's Data**

Density's data is represented as a list of objects with 2 attributes, `timestamp` and `count`. `count` is the total number of people in the location at the given time.

```python
print(location['density`][0])  # {'count': 90, 'timestamp': '2019-04-18T00:00:00.000Z'}
```

**Google's Data**

Google doesn’t count people. They track phones and they show busyness in “relative terms.”
Practically speaking, this means they have no y-axis. Each column’s height is relative to every other column's height.

![Google Popular Times](https://raw.githubusercontent.com/DensityCo/popular-times-project/master/imgs/popular-times-mini.png)

To capture the relative values, we scraped the total number of pixels used for each bar in Google's forecast graphs. The `timestamp` value is when we saw a change in forecast data. We checked for changes in 5 minute intervals.

The data is broken down by day of the week, and then hour of each data. The maximum number of pixels for any given hour is 80.

```python
print(location['google_forecast'][0])

# {
#     'timestamp': '2019-04-19 23:45:07.722180+00:00',
#     'values': {
#         'Monday': {
#             '18': '56', '13': '44', '12': '75', '17': '46', '11': '53', '15': '5', '8': '5', '7': '1', '9': '5', '10': '15', '14': '9', '20': '17', '16': '23', '19': '41'
#         },
#         'Tuesday': {
#             '18': '46', '13': '34', '12': '44', '17': '35', '11': '41', '15': '9', '8': '1', '7': '1', '9': '8', '10': '24', '14': '17', '20': '16', '16': '16', '19': '36'
#         },
#         'Wednesday': {
#             '18': '50', '13': '40', '12': '58', '17': '38', '11': '39', '15': '2', '8': '2', '7': '1', '9': '6', '10': '14', '14': '11', '20': '13', '16': '14', '19': '37'
#         },
#         'Thursday': {
#             '18': '45', '13': '49', '12': '58', '17': '35', '11': '37', '15': '8', '8': '3', '7': '1', '9': '8', '10': '14', '14': '21', '20': '19', '16': '17', '19': '38'
#         },
#         'Friday': {
#             '18': '42', '13': '46', '12': '64', '17': '32', '11': '38', '15': '0', '8': '1', '7': '0', '9': '3', '10': '9', '14': '12', '16': '9', '19': '26'
#         },
#         'Saturday': {
#             '18': '39', '13': '26', '12': '32', '17': '24', '11': '24', '15': '1', '8': '1', '7': '1', '9': '1', '10': '11', '14': '11', '16': '4', '19': '23'
#         },
#         'Sunday': {
#             '18': '53', '13': '24', '12': '32', '17': '38', '11': '25', '15': '1', '8': '1', '7': '1', '9': '1', '10': '10', '14': '9', '16': '10', '19': '31'
#         }
#     }
# }

print(location['google_forecast'][0]['values']['Friday']['14'])  # 12. (i.e. Friday, April 19th @ 2pm Google predicted a busyness level of 12/80. Read more about normalizing the data below.)
```

We also scraped Google's live busyness value when available. This data looks similar to the Density data, where `count` is replaced with the total number of pixels for the bar graph during that time. Google only enables live busyness when it believes it has enough data to accurately predict. Therefore, not all locations have live data or don't have a significant amount of live data. 

```python
location['google_real_time'][0]  # {'value': '18.97', 'timestamp': '2019-04-25 13:22:05.604189+00:00'}
```

**Normalizing the data**

To make sense of Google's data, we have to normalize the pixel values scraped from Google. We used the [following method in our analysis](https://colab.research.google.com/drive/14ZphsDbKp9cluCXLygPVygIOE1EWmUjn#scrollTo=p21QpZBo79sb&line=1&uniqifier=5), though there are other ways to do it.

### Google Colab Notebook

We've created a [Google Colab Notebook](https://colab.research.google.com/drive/14ZphsDbKp9cluCXLygPVygIOE1EWmUjn) to make it easier to explore the data. Make sure to click "Open in playground" to interact with the notebook.
