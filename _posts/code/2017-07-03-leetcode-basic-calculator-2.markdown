---
layout: post
title:  "leetcode: Basic Calculator II"
date:   2017-07-03 +0800
category: code
---

# Basic Calculator II

## Question
Implement a basic calculator to evaluate a simple expression string.

The expression string contains only non-negative integers," +, -, *, /" operators and empty spaces . The integer division should truncate toward zero.

You may assume that the given expression is always valid.

Some examples:
```python
"3+2*2" = 7
" 3/2 " = 1
" 3+5 / 2 " = 5
```
Note: Do not use the eval built-in library function.

## Answer

首先明确题目中的限定条件：
1. 不能使用eval内建函数;
2. 计算表达式以字符串形式提供，包含且仅包含"+/-/*//"四个运算符以及数字和空白符;
3. 符合计算优先级规范，即应当优先计算乘除后计算加减，计算顺序从左到右;

分析条件
1. 模拟人工计算顺序和方式，采用递归和优先级区分的方式计算

```python
import re
class Solution(object):
    mode = 1
    def calculate(self, s):
        """
        :type s: str
        :rtype: int
        """
        if self.mode > 2:
            return int(s)
        symbol = '\*|\/' if self.mode == 1 else '\+|\-'
        pattern = re.compile('^(\-?).*?((\d+)\s*([' + symbol + '])\s*(\d+)).*')
        match = pattern.match(s)
        if match:
            num1, num2 = int(match.group(1)+match.group(3)), int(match.group(5))
            if self.mode == 1:
                res = num1 * num2 if match.group(4) == '*' else num1 / num2
            else:
                res = num1 + num2 if match.group(4) == '+' else num1 - num2
            subed = ('\-?\d+\s*[' + symbol + ']\s*\d+') if match.group(1) else ('\d+\s*[' + symbol + ']\s*\d+')
            s = re.compile(subed).sub(str(res), s, 1)
        else:
            self.mode += 1
        return self.calculate(s)
```

2. 乘除减转换为加法，字符串转换为数字

```python
class Solution(object):
    def calculate(self, s):
        """
        :type s: str
        :rtype: int
        """
        def calculate(sign, nums, num):
            num = int(num) if num else 0
            if sign == '+':
                nums.append(num)
            elif sign == '-':
                nums.append(-num)
            elif sign == '*':
                nums.append(nums.pop() * num)
            elif sign == '/':
                pn = nums.pop()
                val = -(abs(pn) / num) if pn < 0 else (pn / num)
                nums.append(val)
            return nums

        # default sign is '+'
        s = s.replace(' ', '')
        sign = '+'
        nums = list()
        num = ''
        for ch in s:
            if ch.isdigit():
                num += ch
            else:
                # get the current num, and calculate by last sign
                nums = calculate(sign, nums, num)
                # get current sign
                sign = ch
                num = ''
        nums = calculate(sign, nums, num)
        return sum(nums)
```

## Optimization
1. 上述两种方法除了核心思想不同外，代码计算上没有明显地冗余内嵌循环，方法2的求子矩阵应该有更好的办法避免两次旋转带来的开销，
2. 若引入numpy包，旋转计算可以间接使用C语言实现，应该可以提高计算效率。

```python
import re

class Solution(object):
    def calculate(self, s):
        """
        :type s: str
        :rtype: int
        """
        mode, index = 1, 0
        while True:
            if mode > 2:
                return int(s)
            symbol = '\*|\/' if mode == 1 else '\+|\-'
            pattern = re.compile('^(\-?)(.*?)((\d+)\s*([' + symbol + '])\s*(\d+))')
            match = pattern.match(s[index:])
            if match:
                num1, num2 = int(match.group(1)+match.group(4)), int(match.group(6))
                if mode == 1:
                    res = num1 * num2 if match.group(5) == '*' else num1 / num2
                    s = s[:index+match.end(2)] + str(res) + s[index+match.end(6):]
                    index += match.end(2)
                else:
                    res = num1 + num2 if match.group(5) == '+' else num1 - num2
                    s = str(res) + s[match.end(6):]
            else:
                mode += 1
                index = 0
```

## Technique
1. 递归思想可以简化逻辑，使代码简洁明了，但是一定要注意使用条件，
2. zip可以实现矩阵的转秩，但其返回的内部列表是tuple，可以用map+list映射为list列表，<br/>
map(list, zip(*array[::-1])) 实现矩阵顺时针旋转，<br/>
map(list, zip(*array)[::-1]) 实现矩阵顺时针旋转，<br/>
更多zip技巧参考：<br/>
[python zip document](https://docs.python.org/3/library/functions.html?highlight=zip#zip)<br/>
[博客园-复制乔布斯-PYTHON中ZIP()函数用法](http://www.cnblogs.com/blogofwyl/p/4658571.html)<br/>
