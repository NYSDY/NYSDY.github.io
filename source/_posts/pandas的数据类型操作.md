---
title: pandas的数据类型操作
date: 2018-12-01 13:11:30
tags:
- pandas
- python
- 数据分析
english_title: Pandas_data_type_manipulation
categories:
- pandas
---

> 在[原文链接](https://juejin.im/post/5acc36e66fb9a028d043c2a5)中摘抄出部分信息作为记录形成本文。

<!-- more -->

### 数据类型 

| Pandas dtype  | Python 类型 | NumPy 类型                                                   | 用途                 |
| ------------- | ----------- | ------------------------------------------------------------ | -------------------- |
| object        | str         | string_, unicode_                                            | 文本                 |
| int64         | int         | int_, int8, int16, int32, int64, uint8, uint16, uint32, uint64 | 整数                 |
| float64       | float       | float_, float16, float32, float64                            | 浮点数               |
| bool          | bool        | bool_                                                        | 布尔值               |
| datetime64    | NA          | NA                                                           | 日期时间             |
| timedelta[ns] | NA          | NA                                                           | 时间差               |
| category      | NA          | NA                                                           | 有限长度的文本值列表 |



### 数据类型操作

- 使用`df.dtypes`可以显示数据所有列的类型

- `df.info（）` 函数可以显示更有用的信息

## 使用 `astype()` 函数

### 使用条件

- 数据是干净的，可以简单地解释为一个数字
- 你想要将一个数值转换为一个字符串对象

如果数据具有非数字字符或它们间不同质（homogeneous），那么 `astype()` 并不是类型转换的好选择。你需要进行额外的变换才能完成正确的类型转换。

### 使用方式

为了真正修改原始 dataframe 中数据类型，记得把 `astype()` 函数的返回值重新赋值给 dataframe，因为 `astype()` 仅返回数据的副本而不原地修改。

### 参考链接

- https://juejin.im/post/5acc36e66fb9a028d043c2a5