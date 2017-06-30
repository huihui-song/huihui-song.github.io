---
layout: post
title:  "leedcode系列: Spiral Matrix"
date:   2017-06-29 +0800
category: code
---

# Spiral Matrix

## Question
Given a matrix of m x n elements (m rows, n columns), return all elements of the matrix in spiral order.

For example,
Given the following matrix:
```python
[
 [ 1, 2, 3 ],
 [ 4, 5, 6 ],
 [ 7, 8, 9 ]
]
```
You should return [1,2,3,6,9,8,7,4,5].

## Answer

首先明确题目中的限定条件：
1. 起始值是左上角;
2. 遍历矩阵的方向是顺时针螺旋方向;

分析条件
1. 条件1限定计算的起始点是矩阵的一个角，其特点是使得我们可以在有限的起始n步中完整的走完一行或者一列;
2. 条件2，结合上述例子，我们可以将一个完整的螺旋圈（不含中间的5）的遍历顺序是： [1,2,3,6,9,8,7,4]，从上下左右四个边的角度来区分这8个数，可有两种方法：<br/>
  方法1. 两条边的公共点算前一条边：[1,2,3]-->[6, 9]-->[8,7]-->[4]，<br/>
  方法2. 两条边的公共点算后一条边：[1,2]-->[3,6]-->[9, 8]-->[7, 4]，<br/>
  
  仔细思考，其实两种分割方法可以得到两种不同思想的算法;<br>
  方法1. 由条件1作为基础，这个方法的特点是前一条边可以完整走完，走完后剩下的数字位置依然能形成一个完整的矩阵，而第二步的两个点[6, 9]又刚好成为剩下的矩阵的一条完整边，以此类推，这种分法使得每一步的一条边都刚好是剩下矩阵的一个完整边，这为递归方法的应用创造了条件; 同时由于每走完一条边之后剩下的点构成完整矩阵，这使得每一条边的数据都可以通过矩阵旋转使其变到左上角的起始位置. 可以得到代码如下：
```python
  class Solution(object):
    def spiralOrder(self, matrix):
        """
        :type matrix: List[List[int]]
        :rtype: List[int]
        """
        return (list(matrix.pop(0)) + self.spiralOrder(zip(*matrix)[::-1])) if matrix else []
        
```
  方法2看起来每一条边的点数是固定的，固定为行或者列-1，这为循环取值创造了条件，若考虑到可以将矩阵整体逆时针旋转，这种分法的每一条边都是矩阵上面边除去最右点的顺序集合，而总共旋转的圈数是固定知道的：min([ceil(m/2), ceil(n/2)])，总共四条边构成一个圈，除去当前圈后的完整矩阵也是能计算的：rotate(anti_rotate(array[1:-1])[1:-1])，可以得到代码如下：
```python
class Solution(object):
    def spiralOrder(self, matrix):
        """
        :type matrix: List[List[int]]
        :rtype: List[int]
        """
        if not matrix:
            return []
        max_cycle, series = min([(len(matrix) + 1) / 2, (len(matrix[0]) + 1) / 2]), []
        for idx in range(max_cycle):
            if len(matrix) == 1:
                return series + matrix[0]
            if len(matrix[0]) == 1:
                last = [val[0] for val in matrix]
                return series + last
            for edge in range(4):
               series += matrix[0][:-1]
               matrix = zip(*matrix)[::-1]
            matrix = map(list, zip(*(matrix[1:-1]))[::-1])[1:-1]
            matrix = map(list, zip(*matrix[::-1]))
        return series
```
从实际测试看，两种方法都是可以得到正确结果的，从方法1的计算效率较方法2略高一些，主要是因为方法2在求子矩阵时多了两次旋转（一个顺时针一个逆时针），当然这两种方法可以用更复杂精巧的方法代替，这里不再细推。但方法1采用了递归方法极大地简化了代码计算逻辑，并且只有一行核心代码，这正是简洁明了简单可依赖的思想体现。

## Optimization
1. 上述两种方法除了核心思想不同外，代码计算上没有明显地冗余内嵌循环，方法2的求子矩阵应该有更好的办法避免两次旋转带来的开销，
2. 若引入numpy包，旋转计算可以间接使用C语言实现，应该可以提高计算效率。

## Technique
1. 递归思想可以简化逻辑，使代码简洁明了，但是一定要注意使用条件，
2. zip可以实现矩阵的转秩，但其返回的内部列表是tuple，可以用map+list映射为list列表，<br/>
map(list, zip(*array[::-1])) 实现矩阵顺时针旋转，<br/>
map(list, zip(*array)[::-1]) 实现矩阵顺时针旋转，<br/>
更多zip技巧参考：<br/>
[python zip document](https://docs.python.org/3/library/functions.html?highlight=zip#zip)<br/>
[博客园-复制乔布斯-PYTHON中ZIP()函数用法](http://www.cnblogs.com/blogofwyl/p/4658571.html)<br/>
