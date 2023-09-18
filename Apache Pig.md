---
reviewed: 2023-09-06
---

#sd #hadoop

- see [[Hadoop]]
- PIG proposes a high level programming language (Pig Latin) to interact with an [[Hadoop]] cluster
- It is designed to abstract the complexity of writing [[MapReduce]] programs

Example below to calculate max closing price

```pig
-- maxclosingprice.pig

-- Load dataset with column names
-- and datatypes
stock_records = LOAD '/user/hirw/input/stocks'
USING PigStorage(',') as
(exchange:chararray, symbol:chararray, date:datetime, open:float, high:float, low:float, close:float,volume:int, adj_close:float);

--Group records by symbol
grp_by_sym = GROUP stock_records
BY symbol;

--Calculate maximum closing price
max_closing = FOREACH grp_by_sym
GENERATE group, MAX(stock_records.close) as maxclose;

--Store output
STORE max_closing
INTO 'output/pig/stocks'
USING PigStorage(',');
```

- Submit script with `pig <script.pig>`
- Note that it will fail if output directory already exists. You can run `hadoop fs -rm -r <path/to/outputdir>` first
- view the result with `hadoop fs -cat output/pig/stocks/part-r-00000`
