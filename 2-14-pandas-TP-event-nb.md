---
jupytext:
  custom_cell_magics: kql
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.17.3
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

# grouping by period and category

+++

before starting this TP:
- put the `2-14-pandas-TP-event-nb.md` file in your github repository `p25-numerique-groupe8`  
- put the file `events.csv` and the file `country.csv` in its `data` folder 
- put the files `result-color-w.png`, `result-color-m.png`, `result-color-y.png`,  
   `result-bw-w.png`, `result-bw-m.png` and `result-bw-y.png` in its `media` folder

+++

in this TP we work on 

- data that represents *periods* and not just one timestamp
- checking for overlaps
- grouping by period (week, month, year..)
- then later on, grouping by period *and* category
- and some simple visualization tools

+++

here's an example of the outputs we will obtain

````{grid} 3 3 3 3
```{image} media/result-color-w.png
```
```{image} media/result-color-m.png
```
```{image} media/result-color-y.png
```
````

+++

## imports

```{code-cell} ipython3
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
```

## the data

we have a table of events, each with a begin (`beg`) and `end` time; in addition each is attached to a `country`  
(we do not yet know what these events are)

```{code-cell} ipython3
events = pd.read_csv("data/events.csv")
events.head(10)
```

### adapt the type of each columns

surely the columns dtypes need some care
hints:
1. use the the `datetime` formats described here https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior
2. or find and use the international format described https://en.wikipedia.org/wiki/ISO_8601

```{code-cell} ipython3
events["beg"]= pd.to_datetime(events["beg"])
events["end"]= pd.to_datetime(events["end"])
events
```

```{code-cell} ipython3
# check it

events.dtypes
```

### raincheck

check that the data is well-formed, i.e. **the `end`** timestamp **happens after `beg`**

```{code-cell} ipython3
events["well"] = events["end"] > events["beg"]
events["well"].value_counts()
```

```{code-cell} ipython3
#la table est valide
```

sort the dataframe by the `'beg'` column

+++

### are there any overlapping events ?

+++

check if there are overlapping events  

hints:
1. you can use the `pandas.Series.shift` method  
  (i.e. the method `shift` applied to a `pandas` `Series`)
2. there is a `pandas.Timedelta` function

```{code-cell} ipython3
events = pd.read_csv("data/events.csv")
events["beg"]= pd.to_datetime(events["beg"])
events["end"]= pd.to_datetime(events["end"])
```

```{code-cell} ipython3
events = events.sort_values(by="beg")
events["shift"] = events["end"].shift()
overlap = events["shift"] > events["beg"]
overlap.value_counts()
```

```{code-cell} ipython3
#il n'y a pas de recouvrement d'evenement
```

### timespan

What is the timespan covered by the dataset (**earliest** and **latest** events, and **duration** in-between) ?

```{code-cell} ipython3
events["end"].iloc[-1] - events["beg"].iloc[0]
```

### aggregated duration

so, given that there is no overlap, we can assume this corresponds to "reservations" attached to a unique resource  
write a code that computes the **overall reservation time**, as well as the **average usage ratio** over the overall timespan

keep a column with the duration of each event

```{code-cell} ipython3
events["duration"] = events["end"] - events["beg"]
events["duration"].sum()
```

## visualization - grouping by period

### usage by period

grouping by periods: by week, by month or by year, plot the `bar` of the **total duration in that period**

There are at least 2 options to do this grouping, based on `resample()` and `to_period()` : **write them both**

(you can access the `dt` (datetime) attibuts and methods of a column if needed)


`````{admonition} for now, **just get the grouping right (do not improve your plots yet)**
:class: dropdown

you should produce something like e.g.

````{grid} 3 3 3 3
```{image} media/result-bw-w.png
```
```{image} media/result-bw-m.png
```
```{image} media/result-bw-y.png
```
````
we'll make cosmetic improvements below, and [the final results look like this](#label-events-output), but let's not get ahead of ourselves
`````

```{code-cell} ipython3
events["week"] = events["beg"].dt.to_period("W")
events["month"] = events["beg"].dt.to_period("M")
events["year"] = events["beg"].dt.to_period("Y")

time_week = events.groupby("week")["duration"].sum()
time_month = events.groupby("month")["duration"].sum()
time_year = events.groupby("year")["duration"].sum()
time_week.plot(kind="bar")
```

```{code-cell} ipython3
time_month.plot(kind="bar")
```

### improve the title and bottom ticks

add a title to your visualisations

also, and particularly relevant in the case of the per-week visu, we don't get to read **the labels on the horizontal axis**, because there are **too many of them**  
to improve this, you can use matplotlib's `set_xticks()` function; you can either figure out by yourself, or read the few tips below

````{admonition} a few tips
:class: dropdown tip

- the object that receives the `set_xticks()` method is an instance of `Axes` (one X&Y axes system),  
  which is not the figure itself (a figure may contain several Axes)  
  ask google or chatgpt to find the way you can spot the `Axes` instance in your figure
- it is not that clear in the docs, but all you need to do is to pass `set_xticks` a list of *indices* (integers)  
  i.e. if you have, say, a hundred bars, you could pass `[0, 10, 20, ..., 100]` and you will end up with one tick every 10 bars.
- there are also means to use smaller fonts, which may help see more relevant info
````

```{code-cell} ipython3
# let's say as arule of thumb
LEGEND = {
    'W': "week",
    'M': "month",
    'Y': "year",
}

SPACES = {
    'W': 12,   # in the per-week visu, show one tick every 12 - so about one every 3 months
    'M': 3,    # one every 3 months
    'Y': 1,    # on all years
}
```

```{code-cell} ipython3
graphe = time_week.plot(kind="bar")
positions = list(range(0, len(time_week), SPACES["W"]))              # toutes les 12 semaines
graphe.set_xticks(positions)
graphe
```

### a function to convert to hours

you are to write a function that converts a `pd.Timedelta` into a number of hours  
1. read and understand the test code for the details of what is expected
2. use it to test your own implementation

note that if an hour has started even by one second, **it is counted**

```{code-cell} ipython3
# your code
import math
# the type of timedelta is pd.Timedelta
# the function returns an int
def convert_timedelta_to_hours(timedelta: pd.Timedelta) -> int:
    total_seconds = timedelta.total_seconds()
    hours = math.ceil(total_seconds / 3600)
    return hours
```

```{code-cell} ipython3
# test it

# if an hour has started even by one second, it is counted
test_cases = ( 
    # input in seconds, expected result in hours
    (0, 0), 
    (1, 1),     (3599, 1),     (3600, 1), 
    (3601, 2),  (7199, 2),     (7200, 2), 
    # 2 hours + 1s -> 3 hours
    (7201, 3),  
    # 3 hours + 2 minutes -> 4 hours
    (pd.Timedelta(3, 'h') + pd.Timedelta(2, 'm'), 4),
    # 2 days -> 48 hours
    (pd.Timedelta(2, 'D'), 48),
)

def test_convert_timedelta_to_hours():
    for seconds, exp in test_cases:
        # convert into pd.Timedelta if not already one
        if not isinstance(seconds, pd.Timedelta):
            timedelta = pd.Timedelta(seconds=seconds)
        else:
            timedelta = seconds
        # compute and compare
        got = convert_timedelta_to_hours(timedelta)
        print(f"with {timedelta=} we get {got} and expected {exp} -> {got == exp}")

test_convert_timedelta_to_hours()
```

```{code-cell} ipython3
# for debugging; this should return 48

convert_timedelta_to_hours(pd.Timedelta(2, 'D'))
```

### use it to display totals in hours

keep the same visu, but display **the Y axis in hours**  
btw, what was the unit in the graphs above ?

hint:  
you can use `map` to apply a function (for example `convert_timedelta_to_hours`) to a `pandas.Series`

```{code-cell} ipython3
time_week_hour = time_week.map(convert_timedelta_to_hours)
graphe = time_week_hour.plot(kind="bar")
positions = list(range(0, len(time_week), SPACES["W"]))               # toutes les 12 semaines
graphe.set_xticks(positions)
graphe
```

```{code-cell} ipython3
time_month_hour = time_month.map(convert_timedelta_to_hours)
graphe = time_month_hour.plot(kind="bar")
positions = list(range(0, len(time_month), SPACES["M"]))               # toutes les 12 semaines
graphe.set_xticks(positions)
graphe
```

## grouping by period and region

the following table allows you to map each country into a region

```{code-cell} ipython3
# load it

countries = pd.read_csv("data/countries.csv")
countries
```

### a glimpse on regions

what's the most effective way to see how many regions and how many countries per region we have ?

```{code-cell} ipython3
nb_regions = len(countries["region"].unique())
nb_regions
```

```{code-cell} ipython3
region = countries.groupby("region")["name"].nunique()
region
```

### attach a region to each event

your mission is to now show the same graphs, but we want to reflect the relative usage of each region, so we want to [split each bar into several colors, one per region see expected result below](#label-events-output)

+++

most likely your first move is to tag all events with a `region` column

remember that you can `pandas.merge` two data frames

```{code-cell} ipython3
events = pd.merge(events, countries, 
         left_on="country", 
         right_on="name", 
         how="left")
events = events.drop(columns=["name"])
events
```

### visu by period by region

you can now produce [the target figures, again they look like this](#label-events-output)

remember that missing values can be filled with the `fillna` method and that timedelta can be computed with `pandas.Timedelta`

```{code-cell} ipython3

time_week_hour = time_week.map(convert_timedelta_to_hours)
color_map = {"Africa" : "yellow", "Oceania" : "orange", "Europe" : "blue", "Americas" : "green", "Asia" : "red" }
colors = events["region"].map(color_map)
graphe = time_week_hour.plot(kind="bar", color=colors)
positions = list(range(0, len(time_week), SPACES["W"]))               # toutes les 12 semaines
graphe.set_xticks(positions)
graphe
```

```{code-cell} ipython3
time_month_hour = time_month.map(convert_timedelta_to_hours)
graphe = time_month_hour.plot(kind="bar", color=colors)
positions = list(range(0, len(time_month), SPACES["M"]))               # toutes les 12 semaines
graphe.set_xticks(positions)
graphe
```

```{code-cell} ipython3
time_year_hour = time_year.map(convert_timedelta_to_hours)
graphe = time_year.plot(kind="bar", color=colors)
graphe
```

***
