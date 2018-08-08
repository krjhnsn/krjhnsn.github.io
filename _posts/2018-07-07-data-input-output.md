---
layout: post
title: "Data input/output in Python - useful concepts for any dataset"
author: "Keir Johnson"
---

A wise person once said:

<br>
> "Data is the new bacon."
<br>

I'm not sure what that actually means, but I like bacon and I also like data, so I guess it works! Whether it is extracting data from a database, reading flat text files, or exchanging data via an API, understanding data exchange is fundamental. In the article below, I've highlighted what I consider to be important concepts for working with data in Python, specifically, for data processing or data science applications.

### Pandas for data input/output

Just mention the words 'data' and 'python' and without a doubt, one of the very next words you'll hear is 'pandas'. Pandas is an extremely popular and powerful Python package for working with data. Pandas can read a variety of file types (e.g., .txt, .csv, .xlsx) to create a Python DataFrame, which is basically just a data table that can be easily manipulated using python. Pandas has a ton of functionality and you'll use it A LOT. It's definitely worth learning about what pandas can do from the [official documentation.](https://pandas.pydata.org/pandas-docs/stable/)

With so much functionality built into pandas, it can be hard to know what's needed for a typical data input/output task. Let's walk through an example to highlight some of the key concepts. A typical workflow for reading and working with a data set might look like this:

1. Read in data from an external file, system, or process.
2. Validate, cleanse, and format data.
3. Manipulate data, apply calculations or logic, combine with other data, train a model, etc.
4. Write the output data to a file or return the data to another system or process.

For our example, suppose we have a data set of recent customer transactions that looks like the table below. The table contains a mix of data types (numbers, string, dates) and you can see that some data is missing in the 'Product SKU' column. Let's read this file using pandas and see how the workflow above applies.

Transaction ID | Store ID | Product SKU | Purchase Date | Purchase Price
| --- | --- | --- | --- | --- |
001 | ABC | X959 | 6/7/2018 | $5.00
002 | CDE | X951 | 6/15/2018 | $1.00
003 | CDE | | 7/1/2018 | $3.00
004 | ABC | J788 | 7/4/2018 | $8.00
005 | FGH | K001 | 7/6/2018 | $12.00
006 | ABC | | 7/6/2018 | $15.00

<br>
The Python below demonstrates how we might read this data from a .csv file. Note that we specify the delimiting character (' , ' in this case).

After reading in a file, it's usually a good idea to take a look at the first few rows using `head()` to check that we see some of the data we expect and that the data structure is intact. We can use `shape` to validate that the number of rows and columns in the resulting DataFrame matches what we expect from the source file.

It's always a good idea to cleanse incoming data. There are quite a few possibilities for doing this depending on the type of data and the application. However, one option that I've found useful is to strip extra whitespace and newline characters using `strip()`. These sneaky extra characters can cause headaches when you're trying to join multiple datasets together or perform comparisons. In the example below, I use a `for` loop to iterate all the columns in the DataFrame and apply the `strip` function. Regex is also a useful tool for cleansing data. You can use regex to replace unneeded characters or validate that a string of characters (e.g., a phone number) follows a certain format (e.g., 111-111-1111).

Depending on your application you may want to replace missing (null) values. Pandas provides an easy way to do this with the `fillna()` function. Below I have replaced missing values with the text 'NA'.

```python
# import pandas
import pandas as pd

# specify path to the .csv file
path = '/Users/kjohnson/example_transaction_data.csv'

# read .csv to dataframe using pandas, specify the delimiter (',' in this case)
df = pd.read_csv(path, sep = ',')

# get list of column names in the dataframe (to be used later)
colHeaders = df.columns.values.tolist()

# inspect the first 3 rows of the dataframe to ensure the data was read in by pandas as expected
df.head(3)

# number of rows and columns (rows, columns)
df.shape

# remove extra whitespace and newlines using 'strip()'
for column in column_list: # it's handy to have a list of the columns!
    df[column] = df[column].str.strip()

# fill missing values in the DataFrame with the text 'NA'
df = df.fillna(value = 'NA')
```

As pandas reads in a file, it will do its best to interpret the data type for each column. It's good to get into the habit of checking the data types of your DataFrame to avoid errors later. To see the data types pandas ended up assigning while reading in the file, you can use `info()`. 

```python
# display columns in the DataFrame and their type (e.g., float, string, datetime)
df.info()
```

After running `info()`, we can see that the 'Purchase Date' column was assigned the `object` data type. This will cause issues if we want to do date comparisons and so we'll need to convert this to a 'datetime' data type.

The 'Purchase Price' column also needs some work. The dollar sign on purchase price has resulted in the data being assigned the `object` data type which will cause problems if we try to perform calculations on this column, for example, calculate the total amount purchased at each store. We will have to convert this column to a numeric (e.g., int, float) data type.

The output of running ```df.info()``` can be seen below:

```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 6 entries, 0 to 5
Data columns (total 5 columns):
Transaction ID    6 non-null int64
Store ID          6 non-null object
Product SKU       5 non-null object
Purchase Date     6 non-null object
Purchase Price    6 non-null object
dtypes: int64(1), object(4)
memory usage: 320.0+ bytes
```

Now that we've reviewed the data types, we can go ahead and convert the 'Purchase Date' and 'Purchase Price' columns to the data types we need. The code below achieves this using pandas `to_datetime()` function to convert the 'Purchase Date' to a 'datetime' data type and some regex to remove the '$' in the 'Purchase Price' and convert it to a floating point number (float). Running `info()` again on the DataFrame will allow you to validate that, indeed, these operations worked as expected (*success!*)

```python
# convert 'Purchase Date' string to datetime
df['Purchase Date'] = pd.to_datetime(df['Purchase Date'])

# convert 'Purchase Price' column to a number by removing the '$' and converting
df['Purchase Price'] = df['Purchase Price'].replace('[$]', '', regex=True).astype(float)
```

All that time and effort spent on data exploration, cleansing, and data type casting now pays off. We can now perform interesting operations to the data such as finding all the purchases that took place in the month of July or calculating the total purchase amount for each store. Pandas makes writing the results back to a .csv file easy with the `to_csv` function.

```python
# get all purchases from July
dfJulyPurchases = df[df['Purchase Date'] >= datetime.date(2018, 7, 1)]

# get total purchase amount by store
dfStoreAmount = df.groupby(['Store ID'])['Purchase Price'].sum()

# write total purchase amount back to a .csv file
outputPath = '/Users/kjohnson/PurchasesByStore.csv'
df.to_csv(outputPath, index = False)
```
### Python's built-in file reader

As your data set becomes bigger and more complex, things inevitably will go wrong. You'll encounter problems with bogus data, poor data structure, unexpected values, etc. To mitigate these problems, it's beneficial to know something about the meta-data, that is, the characteristics of the data set you're working with. 

Meta-data includes things such as the structure (number of rows and columns, delimeter), the type of data (numbers, strings, dates), character encoding, the appearance of null (missing) values, etc. Having an understanding of this meta-data will allow you to perform validation on incoming data to ensure it is up to snuff before adding it to your dataset and will also help you troubleshoot errors when they come up.

In scenarios where it is important to check if incoming data meet our expectations, an alternative approach to using pandas is to use Python's built-in file reader. Rather than having pandas bring in all the data at once, you can read in each row individually and perform validation before retaining it for the final data set.

Going back to the transactions data example, assume that this time the data is provided in a pipe-delimited text file with the contents as seen below. Inspecting row four you can that the data has been corrupted and only contains four elements. We know that we should expect five elements (columns) per row and we can use Python to validate that each row meets this expectation before adding it to our final data set.

```
Transaction ID|Store ID|Product SKU|Purchase Date|Purchase Price
001|ABC|X959|6/7/2018|$5.00
002|CDE|X951|6/15/2018|$1.00
003|CDE||7/1/2018|$3.00
004|ABC|J788|$8.00	<-- this row only contains four pipe-delimited values
005|FGH|K001|7/6/2018|$12.00
006|ABC||7/6/2018|$15.00
```

To process this data, the file is first opened using `with open(filename, 'r') as f:`. Each line is then read into a list using `readlines()`. Next, each line is split into its elements using `split('|')` and we check if there are five resulting elements. If five elements are found, then we add them to the list `keepRows` which is later converted to a DataFrame that holds the final data set. 

If five elements are not found, we add the contents of the row to a list of discarded rows with `discardedRows.append(discardedRow)`. This list could be used to generate a processing report that we can use to investigate why certain rows were discarded. Often, data integrity issues result from upstream processes that generate the original data set and having information about the discarded rows can be useful for troubleshooting!

Obviously, more advanced logic could be applied to validate each row and/or specific elements in each row, but hopefully this illustrates the concept. You'll have to weigh trade-offs between adding more advanced logic and the impact on run time when it comes to implementing data validation in production-level applications.

```python
# path to text file
path = '/Users/kjohnson/Documents/Python/transactions.txt'

# open the file, read it, then close it
with open(path, 'r') as file:
    data = file.readlines()
 
# check each row - if it has 5 elements keep it otherwise discard it
discardedRows = []
rowNum = 0
keepRows = []
for line in data:
	if len(line.split('|')) == 5:
        	keepRows.append(line.split('|'))
	else:
		discardedRow = [rowNum, line]
		discardedRows.append(discardedRow)

	rowNum = rowNum + 1

# first row in file contains column headers, create dataframe with those columns
colHeaders = keepRows[0]

# remove row headers, remaining rows will be used for dataframe
keepRows.pop(0)

# convert to dataframe
df = pd.DataFrame(keepRows, columns = colHeaders)
```

Taking the time to validate, cleanse, explore, and format your data as a preliminary step in your workflow will help avoid downstream issues and allow you to make the most of your data. Hopefully, this article has given you some ideas on how to do that. As always, I recommend consulting documentation to learn more about what's possible with pandas and other Python features. Thank you for reading!
