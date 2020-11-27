---
title: 8. 字符串转换整数 (atoi)
date: 2020-11-27 10:28:26
categories: LeetCode
tags: [字符串, 有限状态机, medium]
---



https://leetcode-cn.com/problems/string-to-integer-atoi/

<!--more-->



## 题目描述

请你来实现一个 `atoi` 函数，使其能将字符串转换成整数。

首先，该函数会根据需要丢弃无用的开头空格字符，直到寻找到第一个非空格的字符为止。接下来的转化规则如下：

如果第一个非空字符为正或者负号时，则将该符号与之后面尽可能多的连续数字字符组合起来，形成一个有符号整数。
假如第一个非空字符是数字，则直接将其与之后连续的数字字符组合起来，形成一个整数。
该字符串在有效的整数部分之后也可能会存在多余的字符，那么这些字符可以被忽略，它们对函数不应该造成影响。
注意：假如该字符串中的第一个非空格字符不是一个有效整数字符、字符串为空或字符串仅包含空白字符时，则你的函数不需要进行转换，即无法进行有效转换。

在任何情况下，若函数不能进行有效的转换时，请返回 0 。

提示：

* 本题中的空白字符只包括空格字符 `' '` 。
* 假设我们的环境只能存储 32 位大小的有符号整数，那么其数值范围为 [−2^31,  2^31 − 1]。如果数值超过这个范围，请返回  INT_MAX (2^31 − 1) 或 INT_MIN (−2^31) 。



示例 1:

```
输入: "42"
输出: 42
```

示例 2:

```
输入: "   -42"
输出: -42
解释: 第一个非空白字符为 '-', 它是一个负号。
     我们尽可能将负号与后面所有连续出现的数字组合起来，最后得到 -42 。
```

示例 3:

```
输入: "4193 with words"
输出: 4193
解释: 转换截止于数字 '3' ，因为它的下一个字符不为数字。
```

示例 4:

```
输入: "words and 987"
输出: 0
解释: 第一个非空字符是 'w', 但它不是数字或正、负号。
     因此无法执行有效的转换。
```

示例 5:

```
输入: "-91283472332"
输出: -2147483648
解释: 数字 "-91283472332" 超过 32 位有符号整数范围。 
     因此返回 INT_MIN (−231) 。
```



## 思路

有限状态机，根据题意可以确定有四个状态：

* `Start`：起始状态，还没访问到 `' '` 外的其它字符；
* `Signed`：确定了符号的状态，当前字符为 `+/-` 且之前没有访问过除 `' '` 外的其它字符；
* `Into Number`：正在转换为数字的状态，当前字符为数字且该字符前 `k` 个字符都为数字；
* `End`：终止状态。

各个状态的相互转换如下图所示：

![](http://images.yingwai.top/picgo/8. 字符串转换整数.png)



## 代码

状态机：

```java
class Solution {
    public int myAtoi(String str) {
        int status = 1, index = 0, res = 0, signal = 1;
        while (index < str.length()) {
            char c = str.charAt(index);
            status = changeStatus(status, c);
            if (status == 2 && c == '-') signal = -1;
            if (status == 4) break;
            if (status == 3) {
                int pop = Character.getNumericValue(c) * signal;
                // 判断是否越界
                if (res > Integer.MAX_VALUE / 10 || (res == Integer.MAX_VALUE / 10 && pop > 7))
                    return Integer.MAX_VALUE;
                if (res < Integer.MIN_VALUE / 10 || (res == Integer.MIN_VALUE / 10 && pop < -8))
                    return Integer.MIN_VALUE;
                res = res * 10 + pop;
            }
            index++;
        }
        return res;
    }

    private int changeStatus(int status, char c) {
        switch (status) {
            case 1:     // 起始状态
                if(c == ' ') return 1;
                if(c == '+' || c == '-') return 2;
                if(c <= '9' && c >= '0') return 3;
                return 4;
            case 2:     // 有符号状态
                if(c <= '9' && c >= '0') return 3;
                return 4;
            case 3:     // 插入数字状态
                if(c <= '9' && c >= '0') return 3;
                return 4;
        }
        return 4;
    }
}
```

直接模拟：

```java
public int myAtoi(String str) {
        int res = 0, index = 0;
        // 跳过前面多余的空格
        while (index < str.length() && str.charAt(index) == ' ') ++index;
        if (index != str.length()) {
            int signed = 0;
            while (index < str.length()) {
                char c = str.charAt(index);
                if (c == '+' || c == '-') {
                    if (signed != 0) return res;    // 之前已经确定了符号的情况
                    if (c == '+') signed = 1;
                    else signed = -1;
                } else if (c >= '0' && c <= '9') {
                    if (signed == 0) signed = 1;
                    int pop = (c - '0') * signed;
                    // 判断是否越界
                    if (res > Integer.MAX_VALUE / 10 || (res == Integer.MAX_VALUE / 10 && pop > 7))
                        return Integer.MAX_VALUE;
                    if (res < Integer.MIN_VALUE / 10 || (res == Integer.MIN_VALUE / 10 && pop < -8))
                        return Integer.MIN_VALUE;
                    res = res * 10 + pop;
                } else
                    return res;
                ++index;
            }
        }
        return res;
    }
```

