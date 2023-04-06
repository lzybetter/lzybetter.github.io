---
title: pandas.dataframe切片总结
date: 2020-09-20 23:12:45
tags:
	- Python
	- Pandas
---

1. df[‘column_name’] ，df[row_start_index, row_end_index]
   1. 可以通过df[[‘column_name1’, ‘column_name2’]]的方式来指定想选择的单个或多个列；
   2. 可以通过df[df.index == ‘index’]来选择指定的单个行；
   3. 可以通过行的序号来指定想选择的单个或多个行；
   4. 不能通过df[‘index’]的方式来选择指定的行；
   5. 不能通过列的序号来指定想选择的列；

   <!-- more -->

2. loc函数
   1. 可以通过df.loc[[‘index_name1’,’index_name2’], [‘column_name1’, ‘column_name2’]]来指定想选择的单个或多个行和列;
   2. 可以通过行的序号来指定想选择的单个或多个行；
   3. 可以通过df.loc[df.index == ‘index’]来选择指定的单个行；

3. iloc函数
   1. 可以通过df.iloc[df.index == ‘index’]来选择指定的单个行；
   2. 可以通过行列的序号来指定想要选择的行和列；