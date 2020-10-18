---
title: 用python3读写csv文档
date: 2018-10-16 21:36:26
tags:
- python
- 文件读取
- csv
english_title: read the CSV document using python3
categories: python
---



> 对于大多数的CSV格式的数据读写问题，都可以使用 `csv` 库。

<!-- more -->

 例如：假设你在一个名叫stocks.csv文件中有一些股票市场数据，就像这样：

```csv
Symbol,Price,Date,Time,Change,Volume
"AA",39.48,"6/11/2007","9:36am",-0.18,181800
"AIG",71.38,"6/11/2007","9:36am",-0.15,195500
"AXP",62.58,"6/11/2007","9:36am",-0.46,935000
"BA",98.31,"6/11/2007","9:36am",+0.12,104800
"C",53.08,"6/11/2007","9:36am",-0.25,360900
"CAT",78.29,"6/11/2007","9:36am",-0.23,225400
```

# csv文档的读取

## 1. 常规读取

下面向你展示如何将这些数据读取为一个元组的序列：

```python
import csv
with open('stocks.csv') as f:
    f_csv = csv.reader(f)
    headers = next(f_csv)
    for row in f_csv:
        # Process row
        ...
```

在上面的代码中， `row` 会是一个列表。因此，为了访问某个字段，你需要使用下标，如 `row[0]`访问Symbol， `row[4]` 访问Change。==这样可以通过外建字典来存储读出的csv数据。==

## 2. 命名元组

由于这种下标访问通常会引起混淆，你可以考虑使用==命名元组==。例如：

```python
from collections import namedtuple
with open('stock.csv') as f:
    f_csv = csv.reader(f)
    headings = next(f_csv)
    Row = namedtuple('Row', headings)
    for r in f_csv:
        row = Row(*r)
        # Process row
        ...
```

它允许你使用列名如 `row.Symbol` 和 `row.Change` 代替下标访问。 需要注意的是这个只有在列名是合法的Python标识符的时候才生效。如果不是的话， 你可能需要修改下原始的列名(如将非标识符字符替换成下划线之类的)。

## 3. 字典

另外一个选择就是将数据读取到一个字典序列中去。可以这样做：

```python
import csv
with open('stocks.csv') as f:
    f_csv = csv.DictReader(f)
    for row in f_csv:
        # process row
        ...
```

在这个版本中，你可以使用列名去访问每一行的数据了。比如，`row['Symbol']` 或者 `row['Change']`。

`fieldnames` 是dict_reader的一个属性，表示CSV文档的数据名称。可以通过`f_csv.fieldnames`来访问数据名称那一行。



# CSV文件写入

为了写入CSV数据，你仍然可以使用csv模块，不过这时候先创建一个 `writer` 对象。例如:

```python
headers = ['Symbol','Price','Date','Time','Change','Volume']
rows = [('AA', 39.48, '6/11/2007', '9:36am', -0.18, 181800),
         ('AIG', 71.38, '6/11/2007', '9:36am', -0.15, 195500),
         ('AXP', 62.58, '6/11/2007', '9:36am', -0.46, 935000),
       ]

with open('stocks.csv','w') as f:
    f_csv = csv.writer(f)
    f_csv.writerow(headers)
    f_csv.writerows(rows)
```

如果你有一个字典序列的数据，可以像这样做：

```python
headers = ['Symbol', 'Price', 'Date', 'Time', 'Change', 'Volume']
rows = [{'Symbol':'AA', 'Price':39.48, 'Date':'6/11/2007',
        'Time':'9:36am', 'Change':-0.18, 'Volume':181800},
        {'Symbol':'AIG', 'Price': 71.38, 'Date':'6/11/2007',
        'Time':'9:36am', 'Change':-0.15, 'Volume': 195500},
        {'Symbol':'AXP', 'Price': 62.58, 'Date':'6/11/2007',
        'Time':'9:36am', 'Change':-0.46, 'Volume': 935000},
        ]

with open('stocks.csv','w') as f:
    f_csv = csv.DictWriter(f, headers)
    f_csv.writeheader()
    f_csv.writerows(rows)
```

其中`f_csv.writeheader()`也可以替换成`f_csv.writerow(dict(zip(headers, headers)))`

# 参考链接

- https://python3-cookbook.readthedocs.io/zh_CN/latest/c06/p01_read_write_csv_data.html
- https://blog.csdn.net/guoziqing506/article/details/52014506