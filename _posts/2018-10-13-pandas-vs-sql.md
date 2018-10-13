---
layout: post
title: "Python Panda Elephant"
author: "Keir Johnson"
---

A python, a panda, and an elephant walk into a bar...

{% include images.html
max-width="100%" file="/assets/python-panda-elephant-notext.png" alt="data animals"
caption="Python, Pandas, PostgreSQL" %}

If you're coming from an SQL background, pandas syntax can feel a bit awkward at first and departing from the trusty select statement ("select ... from ... where ...") feels a bit like blasphemy. Don't worry, while SQL is definitely better suited for some tasks, pandas contains all the essential functionality (and much more) that you could ever need for a data-related project.

That said, if you're learning to switch between SQL and pandas, it's helpful to see how they relate. Below are the pandas operations I use most often with their SQL equivalents. Note, [PostgreSQL](https://www.postgresql.org/) was used in these examples which will have some variation in syntax compared to something like SQL Server.

For additional context, the dataset used in the examples below contains *highly* scientific information about the various members of the animal kingdom. The top five rows are seen below:

animal name | airborne | aquatic | legs | strength | speed | intelligence | last seen date | first seen date
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
aardvark | 0 | 0 | 4 | 80 | 4 | 64 | 7/5/18 | 10/31/17
antelope | 0 | 0 | 4 | 91 | 98 | 72 | 7/6/18 | 12/17/17
bass | 0 | 1 | 0 | 42 | 40 | 59 | 7/7/18 | 12/11/17
bear | 0 | 0 | 4 | 60 | 70 | 90 | 7/8/18 | 11/20/17
boar | 0 | 0 | 4 | 24 | 6 | 70 | 7/9/18 | 9/24/17
<br/>
If you want to give the examples a try, you may want to populate a pandas dataframe object with the example dataset using the code below:

```python
import pandas as pd
path = 'animal_kingdom.csv'
df = pd.read_csv(path, sep = ',')
```

And, an equivalent SQL table would look something like:

```sql
CREATE TABLE public.animal_kingdom
(
    "animal name" character varying(255) COLLATE pg_catalog."default" NOT NULL,
    airborne numeric,
    aquatic numeric,
    legs numeric,
    strength numeric,
    speed numeric,
    intelligence numeric,
    "last seen date" date,
    "first seen date" date,
    CONSTRAINT animal_kingdom_pkey PRIMARY KEY ("animal name")
)
```

Now, forge on to see the code!

### Display a list of columns and their data type
```python
df.dtypes
```
```sql
select column_name, data_type 
from information_schema.columns 
where table_name = 'animal_kingdom'
```

### Display top N number of rows (5 in this case)
```python
df.head(5)
```
```sql
select * from animal_kingdom
limit 5
```

### Rename a column
```python
df = df.rename(columns = {'animal name': 'Animal Name'})
```
```sql
alter table animal_kingdom
rename column "animal name" to "Animal Name"
```

### Select a subset of columns 
```python
dfSubset = df[['animal name'
               ,'strength'
               ,'intelligence'
               ,'speed']]
```
```sql
drop table if exists animal_subset;
select "animal name", "strength", "intelligence"
into temp animal_subset 
from animal_kingdom
```

### Convert a column to a different data type
```python
df['legs'] = df['legs'].astype(str) # convert numeric to string
df['legs'] = pd.to_numeric(df['legs']) # convert string to numeric (or at least attempt to)
df['last seen date'] = pd.to_datetime(df['last seen date']) # convert string to datetime
```
```sql
alter table animal_kingdom alter column legs type text # convert numeric to string
alter table animal_kingdom alter column legs type int using legs::float; # convert string to numeric (or at least attempt to)

select cast(legs as text) -- 'cast' also a good option
from animal_kingdom
```

### Calculate the difference (in days) between two dates
```python
df['days diff'] = (df['last seen date'] - df['first seen date']).dt.days
```
```sql
select
("last seen date" - "first seen date") as "days diff"
from animal_kingdom
```

### Perform arithmetic opertations
```python
df['total attributes'] = df['strength'] + df['speed'] + df['intelligence']
```
```sql
select
("strength" + "speed" + "intelligence") as "total attributes"
from animal_kingdom
```

### Select rows where date field meets some condition
```python
import datetime
df2018 = df[df['first seen date'] > datetime.date(2018, 1, 1)]
```
```sql
select *
from animal_kingdom
where "first seen date" > '2018-01-01'
```

### Select rows based on multiple conditions
```python
dfSubset = df[(df['intelligence'] > 80) & (df['aquatic'] == 1)]
```
```sql
select * from animal_kingdom
where intelligence > 80 and aquatic = 1
```

### Apply a function or if/then/else logic to obtain a result
```python
# define the function
def animalEnvironment(aquatic, airborne):
    # check if an airborne animal
    if (aquatic == 1):
        return 'aquatic'
    # check if an aquatic animal
    elif (airborne == 1):
        return 'airborne'
    # else assume it's a land animal
    else:
        return 'land'
    
# apply the function and store the result in a new column
df['environment'] = df.apply(lambda row: animalEnvironment(row['aquatic'],row['airborne']),axis = 1)
```
```sql
drop table if exists environment;
select *,
case
	when aquatic = 1 then 'aquatic'
    when airborne = 1 then 'airborne'
    else 'land'
end as environment
into temp environment
from animal_kingdom
```

### Calculate aggregate statistics (in this case, average) on numeric columns, group by other column(s)
```python
dfAvg = df.groupby(['environment']).mean().reset_index()
```
```sql
drop table if exists environment_avg;
select
environment,
avg(airborne) airborne_avg,
avg(aquatic) aquatic_avg,
avg(legs) legs_avg,
avg(strength) strength_avg,
avg(speed) speed_avg,
avg(intelligence) intelligence_avg
into temp environment_avg
from environment
group by environment
```

### Calculate aggregate statistics (in this case, count) on numeric columns, group by other column(s)
```python
dfCount = df.groupby(['environment'])['animal name'].count().reset_index()
```
```sql
drop table if exists environment_count;
select
environment,
count(*) environment_count
into temp environment_count
from environment
group by environment
```

### Perform joins
```python
dfJoined = pd.merge(dfAvg # table 1
                    ,dfCount # table 2 to join to table 1
                    ,how = 'inner' # join method (inner, left, right)
                    ,left_on = ['environment'] # column(s) used for join
                    ,right_on = ['environment']) # column(s) used for join
```
```sql
select
avg.*,
cnt.environment_count
from environment_avg avg
join environment_count cnt
on avg.environment = cnt.environment
```

Of course, this only scratches the surface, but I hope this list will serve as a useful reference!

Thank you for reading!
