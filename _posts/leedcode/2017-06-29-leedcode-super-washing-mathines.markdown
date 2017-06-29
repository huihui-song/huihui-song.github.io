---
layout: post
title:  "leedcode系列: Super Washing Machines"
date:   2017-06-29 15:46:47 +0800
categories: leedcode
---



# Super Washing Machines

## Question：
You have n super washing machines on a line. Initially, each washing machine has some dresses or is empty.<br/>
For each move, you could choose any m (1 ≤ m ≤ n) washing machines, and pass one dress of each washing machine to one of its adjacent washing machines at the same time .<br/>

Given an integer array representing the number of dresses in each washing machine from left to right on the line, you should find the minimum number of moves to make all the washing machines have the same number of dresses. If it is not possible to do it, return -1.<br/>

Example1

> Input: [1,0,5]

> Output: 3

Explanation: <br/>
1st move:    1     0 <-- 5    =>    1     1     4
2nd move:    1 <-- 1 <-- 4    =>    2     1     3    
3rd move:    2     1 <-- 3    =>    2     2     2   
Example2

> Input: [0,3,0]

> Output: 2

Explanation: <br/>
1st move:    0 <-- 3     0    =>    1     2     0    
2nd move:    1     2 --> 0    =>    1     1     1     
Example3

> Input: [0,2,0]

> Output: -1

Explanation: <br/>
It's impossible to make all the three washing machines have the same number of dresses. <br/>
Note:<br/>

The range of n is [1, 10000].<br/>
The range of dresses number in a super washing machine is [0, 1e5].<br/>

## Anwser:
题目中的三个重要限定条件是：
> 1. 每台洗衣机同时只能拿出一件衣服，但可以同时放进去多件衣服；
> 2. 从某台洗衣机拿出来的一件衣服只能放入隔壁相邻左右两边任意一台洗衣机中；
> 3. 每台洗衣机最后衣服数相等；

由此分析， 假设输入序列：array=[0, 0, 11, 5]，假设当前选定第idx台洗衣机，则
> 1. 条件1：若当前洗衣机衣服数array[idx]>平均数average，则从该洗衣机中拿出多余的衣服最少需要array[idx]-average次；反之，则将该洗衣机衣服数补齐到平均数最少需要：向上取整ceil(0.5*(average-array[idx]))次；
> 2. 条件2：假设衣服平均前洗衣机idx左边的洗衣机衣服总数是：left_sum，平均后的总数是average*idx，同理右边的数据分别是：right_sum、average*(len(array)-1-idx)；我们规定平均前的数据为保有量contain，平均后为期望量：expect，由于衣服只能挪动一台洗衣机的位置，所以将两边衣服从contain变到expect，衣服都需要经过当前第idx台洗衣机，所以平均化最少需要操作max([abs(left_contain-left_expect), abs(right_contain-right_expect)])次；
> 3. 条件3：假设衣服总数不能被洗衣机总数整除，则确定不能平均化；

综合上述三个条件写出代码如下：
```python
class Solution(object):
    def findMinMoves(self, machines):
        """
        :type machines: List[int]
        :rtype: int
        """
        min_num, all_sum, length = 0, sum(machines), len(machines)
        if all_sum % length != 0:
            return -1
        ave, left_sum = all_sum / length, 0
        for idx in range(len(machines)):
            idx_val = machines[idx]
            left_sum = sum(machines[:idx])
            right_sum = all_sum - left_sum - ave
            left_num = left_sum - idx * ave
            right_num = abs(left_num) - abs(ave - idx_val)
            max_curr_pick = abs(idx_val - ave) if idx_val > ave else ceil(0.5*abs(idx_val - ave))
            min_num = max([abs(left_num), abs(right_num), max_curr_pick, min_num])
        return min_num
```

## Optimize:
上述代码中有两个地方可以优化：
> 1. 对于左边的求和：left_sum = sum(machines[:idx])，在machines长度很长的时候，开销是很大的，因为这相当于内嵌了一层循环，可以改成left_sum += machines[idx - 1] if idx > 0 else 0的累加模式，去掉了内嵌循环；
> 2. 计算right_num时right_num = abs(left_num) - abs(ave - idx_val)，可见右边是恒不大于左边的，因此，右边的计算是多余的，可以去掉右边的计算，得到优化后的代码如下：<br/>

```python
class Solution(object):
    def findMinMoves(self, machines):
        """
        :type machines: List[int]
        :rtype: int
        """
        min_num, all_sum, length = 0, sum(machines), len(machines)
        if all_sum % length != 0:
            return -1
        ave, left_sum = all_sum / length, 0
        for idx in range(len(machines)):
            idx_val = machines[idx]
            left_sum += machines[idx - 1] if idx > 0 else 0
            left_num = left_sum - idx * ave
            max_curr_pick = abs(idx_val - ave) if idx_val > ave else ((abs(idx_val - ave)+1)/2)
            min_num = max([abs(left_num), max_curr_pick, min_num])
        return min_num
```

## Tecknique:
1. 累加方式取代内嵌循环求和；
2. 数学逻辑去掉不必要的计算；

