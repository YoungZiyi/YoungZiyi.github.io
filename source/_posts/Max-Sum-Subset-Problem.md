---
title: Max Sum Subset Problem
date: 2017-3-14 11:11:11 +0800
categories: 技术
tags: 算法
---
最大子列和问题  
最简单、时间复杂度为O(n^3)的实现方法  
``` python
def get_max_sum_subset_n3(x):
    max_sum = 0
    index = []
    # 子列起始元素下标
    for i in range(len(x)):
        # 子列终止元素下标
        for j in range(i, len(x)):
            sum = 0 # 初始子列和
            # 计算子列和，from i to j
            for k in range(i, j+1):
                sum = sum + x[k]
            # 如果当前子列和大于最大子列和
            if sum > max_sum:
                # 子列和赋给最大子列和
                max_sum = sum
                # 子列下标
                index = range(i, j+1)
    # 返回最大子列和以及子列
    return [max_sum, [x[h] for h in index]]

```

时间复杂度为O(n^2)的实现方法
```python
def get_max_sum_subset_n2(x):
    max_sum = 0 # 最大子列和
    start_index = -1 # 实时起始下标
    the_start_index = -1 # 最大子列起始下标
    the_stop_index = -1 # 最大子列终止下标

    for i in range(len(x)):
        sum = 0 # 初始子列和为0
        for j in range(i, len(x)):
            # 如果实时和为0，则当前元素为子列起始位置
            if sum == 0:
                start_index = j
            # 实时子列和加上当前元素
            sum = sum + x[j]
            # 如果当前子列和大于最大子列和
            if sum > max_sum:
                # 把当前子列和赋给最大子列和
                max_sum = sum
                # 记录最大子列和起始下标
                the_start_index = start_index
                # 记录终止下标
                the_stop_index = j
            
            # 如果当前子列和为负数，重置当前子列和为0
            if sum < 0
                sum = 0

    return [max_sum, [x[k] for k in range(the_start_index, the_stop_index+1)]
```

速度最快、时间复杂度为O(n)的实现方法  
``` python
def get_max_sum_subset_n(x):
    # 空间换时间，用多个变量记录各个状态下的信息
    sum = 0 # 实时和值
    max_sum = 0 # 实时最大值
    start_index = -1 # 实时起始下标
    the_max_sum_ever = 0 # 循环到目前为止，子列最大和
    the_start_index = -1 # 循环到目前为止，子列起始下标
    the_stop_index = -1 # 循环到目前为止，子列终止下标
    
    for i in range(len(x)):
        # 第i个元素之前的最大子列和加上第i个元素
        sum = max_sum + x[i]
        # 判断当前和是否大于0
        if sum > 0:
            # 实时和大于0，当前子列还未中断
            if max_sum == 0:
                # 如果实时最大和等于0，则说明当前下标为子列起始下标
                start_index = i
            # 子列未中断，则将实时和赋给实时最大和
            max_sum = sum
        else:
            # 如果当前和小于0，则可认为当前第i个元素终止了最大子列
            # 实时最大和归0
            max_sum = 0
        # 实时值与历史值对比
        if max_sum > the_max_sum_ever:
            # 如果实时最大和大于历史最大和
            # 则将实时最大和赋给历史最大和
            the_max_sum_ever = max_sum
            # 当前下标作为终止下标
            the_stop_index = i
            # 当前起始下标作为历史起始下标
            the_start_index = start_index
    # 返回历史最大和以及子列
    return [the_max_sum_ever, [x[j] for j in range(the_start_index, the_stop_index+1)]]
```

