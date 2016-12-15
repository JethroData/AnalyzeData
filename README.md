#AnalyzeData

AnalyzeData reads a sample of the input data that is going to be loaded into Jethro

###Prerequisites:
Python 2.6 or higher installed.
Python tabulate package (not required if report is issued in csv mode. If not installed, report will automatically be set to csv mode).
To install tabulate:
Linux: 
	`TABULATE_INSTALL=lib-only pip install tabulate`
Windows:
	```
	set TABULATE_INSTALL=lib-only
	pip install tabulate
	```

If pip is not installed:
	```
	wget https://bootstrap.pypa.io/get-pip.py
	python get-pip.py
	```
	
###Run

`AnalyzeData.py [-i <rows to read>] [-d <delimiter>] [-q <quote char>] [-n] [-c] [-g <table name>] [<input file>]`

####Where
* -i: Number of rows to read from the input. Default=unlimited.
* -d: The input data delimiter. Default="," (comma).
* -q: The input data quote character. This allows a delimiter character inside quotes as part of the data. Default=" (double quotes).
* -n: If specified, treats the first row as headers row which contains the column names.
* -c: CSV formatted output. Write the output report as a tab delimited file instead of a formatted table. Installing tabulate is not required in this mode.
* -g: Generate a create table script and a description file using the given table name.
* input file: The input file to read. If not specified, read from standard input.
	
The input data is expected to be delimited rows of data. 
The data is read and analyzed and then a report such as the following is generated:

Number | Name | Rows | Type | Category | Percent | Exceptions | Distinct | Samples
------ | ---- | ---- | ---- | -------- | ------- | ---------- | -------- | -------
1 | name | 6 | STRING | Primary Key | 100 | | 6 | "aaa" "bbb" "ccc" "ddd" "eee" "fff"
2 | age | 6 | BIGINT | | 66 | "NULL" | 4 | "1234568674737747372" "-12" "341"
3 | birth date | 6 | TIMESTAMP | Date | 100 | | 5 | "2016-01-07" "2016-12-23" "2016-07-1" "20160714" "2016-07-14"
4 | balance | 6 | DOUBLE | | 83 | "-" | 6 | "1.23" "0.34" "12.0" "11" "0.00000537774"
5 | phone | 6 | STRING | Phone Number | 100 | | 5 | "201 239 3244" "201-345-2136" "" "(212) 435 9884" "917-234-0890"


* Number: The column serial number.
* Name: The column name, if the data contains headers. Otherwise it is c1..cN.
* Rows: The number of rows for the column.
* Type: The suggested type to use based on the data. A non string type is suggested in case non string values are found and there are no more than 5 distinct exception values.
* Category: For certain values, a category can be detected based on regular expressions. It also specifies "Primary Key" when a column has unique values and "High Cardinality" if it has many unique values.
* Percent: The percentage of the values of the suggested type out of all values.
* Exceptions: A list of up to 5 exception values. Exception values are values that do not match the suggested type.
* Distinct: The number of distinct values.
* Samples: Sample values of the suggested type.

In addition, if the -g parameter is specified with a table name, then a create table script and a description file is generated based on the data.
For the above data, the following scripts are generated as test.ddl and test.desc when given the table name "test":

```
create table test
(
name STRING,
age BIGINT,
birth date TIMESTAMP,
balance DOUBLE,
phone STRING
);

table test
row format delimited
	fields terminated by '|'
	null defined as 'NULL'
OPTIONS
	SKIP 1
(
name,
age,
birth date format='yyyy-M-d',
balance null defined as '-',
phone
)
```

####Notes
* If the suggested type is not STRING, but the exception list has more than one value, then the generated type will become STRING.
* The null definition is based on the most common exception value.
* If a column has a single exception value that is different that the most common null value, then a column specific null definition is generated for that column.
* If the input contains a header row, then a SKIP 1 option is added.
* The timestamp format is generated based on the first timestamp value in the column.

###Examples
* Analyze data in a csv file "data.csv"
```
python AnalyzeData.py -d '|' data.csv
```

* Analyze data in a csv file "data.csv" that contains headers, generate scripts for a table "customers" and store the output into a file in csv mode
```
python AnalyzeData.py -d '\001' -n -c -g customers data.csv > output.csv
```

* Analyze data stored in a hive table "sales.customers", limiting the input to 10000 rows 
```
hive -e "select * from sales.customers" | python AnalyzeData.py -i 10000 -d '\t'
```



