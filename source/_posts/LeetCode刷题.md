---
title: LeetCode刷题
date: 2020-05-25 20:48:42
categories: Coding
tags: LeetCode
---

记录一下每日刷题 $\surd$

<!--more-->

[TOC]

# 4. 寻找两个正序数组的中位数

https://leetcode-cn.com/problems/median-of-two-sorted-arrays/

## 题目描述

给定两个大小为 m 和 n 的正序（从小到大）数组 `nums1` 和 `nums2`。

请你找出这两个正序数组的中位数，并且要求算法的时间复杂度为 O(log(m + n))。

你可以假设 `nums1` 和 `nums2` 不会同时为空。

 

示例 1:

```
nums1 = [1, 3]
nums2 = [2]

则中位数是 2.0
```

示例 2:

```
nums1 = [1, 2]
nums2 = [3, 4]

则中位数是 (2 + 3)/2 = 2.5
```



## 思路

题目要求时间复杂度为 O(log(m + n))，知道要用二分查找，但是具体方法想不出来，只能看了一篇题解中的[找第 k 小的数的方法](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/solution/xiang-xi-tong-su-de-si-lu-fen-xi-duo-jie-fa-by-w-2/)。

主要思想是：

* 比较两个数组的第 `k/2` 个数字（向下取整），哪个小则可以直接排除那个数组的前 `k/2` 个数字（因为两个数组都是有序的）；
* 上一步已经排除了 `k/2` 个数，因此 `k = k - k/2`，再次比较两个数组（其中一个数组为排除了 `k/2` 个数后的新数组）的第 `k/2` 个数字，直到 `k = 1` 或其中一个数组中的元素被全部排除了：
  1. 若 `k = 1` 则返回当前两个数组的第一个元素中较小的那一个；
  2. 若其中一个数组被全部排除，则返回另一个数组中的第 `k` 个元素。



## 代码

```python
class Solution:
    def findMedianSortedArrays(self, nums1: List[int], nums2: List[int]) -> float:
        n, m = len(nums1), len(nums2)
        left = (n + m + 1) // 2
        right = (n + m + 2) // 2
        return (self.recur(left, nums1, 0, n-1, nums2, 0, m-1) + self.recur(right, nums1, 0, n-1, nums2, 0, m-1)) * 0.5

    def recur(self, k, arr1, s1, e1, arr2, s2, e2):
        l1 = e1 - s1 + 1
        l2 = e2 - s2 + 1
        if l1 > l2:
            return self.recur(k, arr2, s2, e2, arr1, s1, e1)
        if l1 == 0:
            return arr2[s2+k-1]
        if k == 1:
            return min(arr1[s1], arr2[s2])
        p1 = s1 + min(k//2, l1) - 1
        p2 = s2 + min(k//2, l2) - 1
        if arr1[p1] < arr2[p2]:
            return self.recur(k-min(k//2, l1), arr1, p1+1, e1, arr2, s2, e2)
        else:
            return self.recur(k-min(k//2, l2), arr1, s1, e1, arr2, p2+1, e2)
```



# 6. Z字形变换

https://leetcode-cn.com/problems/zigzag-conversion/

## 题目描述

将一个给定字符串根据给定的行数，以从上往下、从左到右进行 Z 字形排列。

比如输入字符串为 `"LEETCODEISHIRING"` 行数为 3 时，排列如下：

```
L   C   I   R
E T O E S I I G
E   D   H   N
```

之后，你的输出需要从左往右逐行读取，产生出一个新的字符串，比如：`"LCIRETOESIIGEDHN"`。

请你实现这个将字符串进行指定行数变换的函数：

```
string convert(string s, int numRows);
```

示例 1:

```
输入: s = "LEETCODEISHIRING", numRows = 3
输出: "LCIRETOESIIGEDHN"
```

示例 2:

```
输入: s = "LEETCODEISHIRING", numRows = 4
输出: "LDREOEIIECIHNTSG"
解释:

L     D     R
E   O E   I I
E C   I H   N
T     S     G
```



## 思路

用一个二维列表 `tmp` 存储 Z 字形字符串中每一行的元素，最后将 `tmp` 中的字符串逐行拼接即得到结果，具体步骤如下：

* 从头到尾遍历`s`，`cnt` 表示当前字符的下标，`curRow` 表示当前字符 `s[cnt]` 在 Z 字形字符串中所在的行数，`step` 表示当前的方向（因为 Z 字形中字符都是往上往下这样循环）；
* 将 `s[cnt]` 加入 `tmp[curRow]`，然后 `cnt` 自增 1，`curRow` 自增 `step`。当 `curRow` 为 `0` 或 `numRows` 时，转换方向，`step *= -1` 。



## 代码

```python
class Solution:
    def convert(self, s: str, numRows: int) -> str:
        if numRows == 1: return s
        tmp = ["" for _ in range(numRows)]
        cnt = curRow = 0
        step = -1
        while cnt < len(s):
            tmp[curRow] += s[cnt]
            cnt += 1
            if curRow == numRows - 1 or curRow == 0:
                step *= -1
            curRow += step
        return ''.join(tmp)
```



# 14. 最长公共前缀

https://leetcode-cn.com/problems/longest-common-prefix/

## 题目描述

编写一个函数来查找字符串数组中的最长公共前缀。

如果不存在公共前缀，返回空字符串 `""`。

示例 1:

```
输入: ["flower","flow","flight"]
输出: "fl"
```

示例 2:

```
输入: ["dog","racecar","car"]
输出: ""
解释: 输入不存在公共前缀。
```


说明:

所有输入只包含小写字母 `a-z` 。



## 思路

Python中的`max()`和`min()`可以比较字符串，按照ASCII值逐位比较：比如`cba`、`cbab`、`cbd`中最大为`cbd`，最小为`cba`。因此只要比较最大最小的两个字符串即可找到最大公共前缀。



## 代码

```python
class Solution:
    def longestCommonPrefix(self, strs: List[str]) -> str:
        if not strs: return ""
        str1 = min(strs)
        str2 = max(strs)
        for i,x in enumerate(str1):
            if x != str2[i]:
                return str2[:i]
        return str1
```



# 21. 合并两个有序链表

https://leetcode-cn.com/problems/merge-two-sorted-lists/

## 题目描述

将两个升序链表合并为一个新的 升序 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

 

示例：

```
输入：1->2->4, 1->3->4
输出：1->1->2->3->4->4
```



## 思路

因为链表有序，可以用两个指针分别遍历两个链表，将两个指针指向的节点中值较小的节点加入到新链表中，然后该指针往后挪，直到遍历完两个链表。

时间复杂度为 O(M + N)，M 和 N 分别为两个链表的长度。



## 代码

```python
class Solution:
    def mergeTwoLists(self, l1: ListNode, l2: ListNode) -> ListNode:
        if not l1 and not l2: return None
        p1, p2 = l1, l2
        pre = None
        while p1 and p2:
            if p2.val < p1.val:
                if not pre:
                    root = p2
                    pre = root
                else:
                    pre.next = p2
                    pre = p2
                p2 = p2.next
            else:
                if not pre:
                    root = p1
                    pre = root
                else:
                    pre.next = p1
                    pre = p1
                p1 = p1.next
        if p1:
            if not pre:
                return p1
            pre.next = p1
        else:
            if not pre:
                return p2
            pre.next = p2
        return root
```





# 32. 最长有效括号

https://leetcode-cn.com/problems/longest-valid-parentheses/

## 题目描述

给定一个只包含 `'('` 和 `')'` 的字符串，找出最长的包含有效括号的子串的长度。

示例 1:

```
输入: "(()"
输出: 2
解释: 最长有效括号子串为 "()"
```


示例 2:

```
输入: ")()())"
输出: 4
解释: 最长有效括号子串为 "()()"
```



## 思路

动态规划：用 $dp[i]$ 表示字符串中以第 $i$ 个字符结尾的字符串的最大有效子串长度。根据题意，$s[i]$ 为 `'('` 时 $dp[i]=0$，有效的子串结尾肯定是 `')'`，可列出动态方程：

当 $s[i]$ 为 `')'` 且 $s[i-1]$ 为 `'('` 时，也就是 "$......()$"，
$$
dp[i] = dp[i-2] + 2
$$
当 $s[i]$ 为 `')'` 且 $s[i-1]$ 为 `')'` 时，也就是 "$......))$"，如果倒数第二个 `')'` 是一个有效字符串 $sub_s$ 的一部分，且 $s[i-\mbox{len}(sub_s)-1]$ 为 `'('` 。此时的最长有效子串的长度即为 $sub_s$ 加上 $2$ 再加上 $dp[i-\mbox{len}(sub_s)-2]$：
$$
dp[i] = dp[i-1] + 2 + dp[i-dp[i-1]-2]
$$
最后的结果为 $dp$ 数组中的最大值。

上面两条方程都要注意判断下标 $i-2$ 和 $i-dp[i-1]-2$ 的值，不要小于 $0$。



## 代码

```python
class Solution:
    def longestValidParentheses(self, s: str) -> int:
        if len(s) <= 1:
            return 0
        dp = [0]
        for i in range(1, len(s)):
            if s[i] == ')':
                if s[i-1] == '(':
                    dp.append(dp[max(i-2, 0)] + 2)
                elif s[i-1] == ')' and i-dp[i-1]-1 >= 0 and s[i-dp[i-1]-1] == '(':
                    dp.append(dp[i-1] + 2 + dp[max(i-dp[i-1]-2, 0)])
                else:
                    dp.append(0)
            else:
                dp.append(0)
        return max(dp)
```



# 35. 搜索插入位置

https://leetcode-cn.com/problems/search-insert-position/

## 题目描述

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

你可以假设数组中无重复元素。

示例 1:

```
输入: [1,3,5,6], 5
输出: 2
```

示例 2:

```
输入: [1,3,5,6], 2
输出: 1
```

示例 3:

```
输入: [1,3,5,6], 7
输出: 4
```

示例 4:

```
输入: [1,3,5,6], 0
输出: 0
```



## 思路

二分查找，每次排除当前数组中一半的元素。

注意递归边界为 `left >= right`，`target` 在 `nums` 不存在有两种情况：

* 当 `left > right` 时，数组 `nums` 中不存在与 `target` 相等的元素，此时 `nums[left]` 为 `nums` 中第一个大于 `target` 的元素，`left` 即为 `target` 应该插入的位置；
* 当 `left == right` 时，此时 `left` 和 `right` 应该都指向 `nums` 中最后一个元素（因为 `mid` 总是向下取整，这种情况下 `left` 每次都 `+1`，直到指向最后一个元素），此时需要额外判断 `target` 与最后一个元素的大小，`target` 大则将其插入到数组末尾。



## 代码

```python
class Solution:
    def searchInsert(self, nums: List[int], target: int) -> int:
        return self.getInsertIndex(nums, 0, len(nums)-1, target)

    def getInsertIndex(self, nums, left, right, target):
        if left >= right:
            if target > nums[left]:
                return left + 1
            return left
        mid = (left + right) // 2
        if target == nums[mid]:
            return mid
        if target < nums[mid]:
            return self.getInsertIndex(nums, left, mid-1, target)
        else:
            return self.getInsertIndex(nums, mid+1, right, target)
```



# 94. 二叉树的中序遍历

https://leetcode-cn.com/problems/binary-tree-inorder-traversal/

## 题目描述

给定一个二叉树，返回它的中序 遍历。

示例:

```
输入: [1,null,2,3]
   1
    \
     2
    /
   3

输出: [1,3,2]
```

进阶: 递归算法很简单，你可以通过迭代算法完成吗？



## 思路

这里可以用两种方法，递归和迭代，前者比较简单就不说了，主要讲后者。

迭代算法中需要用到一个栈，每到一个节点先将其入栈，遍历其左子树，然后访问该节点，访问完后该节点就可以出栈了，最后遍历其右子树。



## 代码

数据结构：

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None
```

递归：

```python
class Solution:
    def inorderTraversal(self, root: TreeNode) -> List[int]:
        ans = []

        def helper(root):
            if root:
                helper(root.left)
                ans.append(root.val)
                helper(root.right)
            return

        helper(root)
        return ans
```

迭代：

```python
class Solution:
    def inorderTraversal(self, root: TreeNode) -> List[int]:
        if not root:
            return []
        ans, v = [], []
        cur = root
        while cur or v:
            while cur:
                v.append(cur)
                cur = cur.left
            cur = v.pop()
            ans.append(cur.val)
            cur = cur.right

        return ans
```



# 96. 不同的二叉搜索树

https://leetcode-cn.com/problems/unique-binary-search-trees/

## 题目描述

给定一个整数 n，求以 1 ... n 为节点组成的二叉搜索树有多少种？

示例:

```
输入: 3
输出: 5
解释:
给定 n = 3, 一共有 5 种不同结构的二叉搜索树:

   1         3     3      2      1
    \       /     /      / \      \
     3     2     1      1   3      2
    /     /       \                 \
   2     1         2                 3
```



## 思路

动态规划：

* 用 `dp[n]` 表示以 1 ... n 为节点组成的二叉搜索树的数量，`f[i]` 表示 i 为根节点的二叉搜索树的数量；
* 结果即为以各个节点作为根节点的数量总和：`dp[n] = f[1] + f[2] + ... + f[n]`；
*  当 i 为根节点时，其左子树的节点数为 i-1，右子树的节点数为 n-i，因此有：`f[i] = f[i-1] * f[n-i]`；
* `dp[n] = dp[0]*dp[n-1] + dp[1]*dp[n-2] + ... + dp[n-1]*dp[0]`。

当 n 为1或0时，只有一种情况，即 `dp[0] = dp[1] = 1`。



还有一种方法是递归，也是利用上面的公式。



## 代码

动态规划：

```python
class Solution:
    def numTrees(self, n: int) -> int:
        dp = [0] * (n + 1)
        dp[0] = dp[1] = 1
        for i in range(2, n+1):
            for j in range(1, i+1):
                dp[i] += dp[j-1] * dp[i-j]
        return dp[n]
```

递归：

```python
class Solution:
    def numTrees(self, n: int) -> int:
        memo = [0] * (n + 1)

        def helper(n):
            if n == 1 or n == 0: return 1
            if memo[n] > 0:
                return memo[n]
            for i in range(0, n):
                memo[n] += helper(i) * helper(n-i-1)
            return memo[n]

        return helper(n)
```



# 97. 交错字符串

https://leetcode-cn.com/problems/interleaving-string/

## 题目描述

给定三个字符串 s1, s2, s3, 验证 s3 是否是由 s1 和 s2 交错组成的。

示例 1:

```
输入: s1 = "aabcc", s2 = "dbbca", s3 = "aadbbcbcac"
输出: true
```

示例 2:

```
输入: s1 = "aabcc", s2 = "dbbca", s3 = "aadbbbaccc"
输出: false
```



## 思路

动态规划，用 `dp[i][j]` 表示 `s1` 的前 `i` 个字符和 `s2` 的前 `j` 个字符是否能交错组成 `s3` 的前 `i+j` 个字符。若 `s1[i]==s3[i+j]`，那么 `dp[i][j]` 是否为真则取决于 `s1` 的前 `i-1`（`s2` 的前 `j-1`）个字符和 `s2` 的前 `j` `s1` 的前 `i`）个字符是否能交错组成 `s3` 的前 `i+j-1` 个字符，即 `dp[i-1][j]`（`dp[i][j-1]`）是否为真。

可得状态转移方程：

* `dp[i][j] = (dp[i-1][j] and s1[i-1]==s3[i+j-1]) or (dp[i][j-1] and s2[j-1]==s3[i+j-1])`；
* 边界条件为`dp[0][0] = True`。

当 `s1` 和 `s2` 长度之和不等于 `s3` 长度时，直接输出 `False`。

时间复杂度为 O(MN)。



## 代码

```python
class Solution:
    def isInterleave(self, s1: str, s2: str, s3: str) -> bool:
        n, m = len(s1), len(s2)
        if n + m != len(s3):
            return False
        dp = [[False] * (m + 1) for _ in range(n + 1)]
        dp[0][0] = True
        for i in range(n + 1):
            for j in range(m + 1):
                if i > 0:
                    dp[i][j] = dp[i][j] or (dp[i-1][j] and s1[i-1]==s3[i+j-1])
                if j > 0:
                    dp[i][j] = dp[i][j] or (dp[i][j-1] and s2[j-1]==s3[i+j-1])
        return dp[n][m]
```



# 100. 相同的树

https://leetcode-cn.com/problems/same-tree/

## 题目描述

给定两个二叉树，编写一个函数来检验它们是否相同。

如果两个树在结构上相同，并且节点具有相同的值，则认为它们是相同的。

示例 1:

    输入:       1         1
              / \       / \
             2   3     2   3
    
            [1,2,3],   [1,2,3]
            
    输出: true


示例 2:

    输入:      1          1
              /           \
             2             2
    
            [1,2],     [1,null,2]
            
    输出: false


示例 3:

    输入:       1         1
              / \       / \
             2   1     1   2
    
            [1,2,1],   [1,1,2]
            
    输出: false



## 思路

就是二叉树的遍历，用递归判断两棵树对应节点是否存在以及值是否相等即可。



## 代码

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution:
    def isSameTree(self, p: TreeNode, q: TreeNode) -> bool:
        
        def helper(p, q):
            if not p and not q: return True
            if p and not q or not p and q: return False
            if p.val != q.val: return False
            return helper(p.left, q.left) and helper(p.right, q.right)

        return helper(p, q)
```



# 101. 对称二叉树

https://leetcode-cn.com/problems/symmetric-tree/

## 题目描述

给定一个二叉树，检查它是否是镜像对称的。

 

例如，二叉树 [1,2,2,3,4,4,3] 是对称的。

        1
       / \
      2   2
     / \ / \
    3  4 4  3




但是下面这个 `[1,2,2,null,3,null,3]` 则不是镜像对称的:

        1
       / \
      2   2
       \   \
       3    3




进阶：

你可以运用递归和迭代两种方法解决这个问题吗？



## 思路

可以通过判断根节点的两棵子树 $p,q$ 是否是镜像的。遍历的方式与第100题类似，只是这次两者不再按相同的顺序遍历，而是按镜像的方式：

* $p$ 遍历其左子树时，$q$ 遍历其右子树；
* $p$ 遍历其右子树时，$q$ 遍历其左子树。



## 代码

```python
class Solution:
    def isSymmetric(self, root: TreeNode) -> bool:
        def helper(p, q):
            if not p and not q: return True
            if p and not q or not p and q: return False
            if p.val != q.val: return False
            return helper(p.left, q.right) and helper(p.right, q.left)
        if root and root.left and root.right:
            return helper(root.left, root.right)
        elif root and not root.left and not root.right or not root:
            return True
        else:
            return False
```



# 102. 二叉树的层序遍历

https://leetcode-cn.com/problems/binary-tree-level-order-traversal/

## 题目描述

给你一个二叉树，请你返回其按 **层序遍历** 得到的节点值。 （即逐层地，从左到右访问所有节点）。

 

示例：
二叉树：`[3,9,20,null,null,15,7]`,

        3
       / \
      9  20
        /  \
       15   7

返回其层次遍历结果：

```
[
  [3],
  [9,20],
  [15,7]
]
```



## 思路

最简单的方法是用递归，按前序遍历二叉树中的节点，记录访问节点的层数，每层用一个列表 `ans[d]` ( `d` 表示当前节点所在层的深度)保存节点值。

另一种思路是使用BFS，使用一个队列 `queue` 存放待访问的节点，每到一层都用 `s` 存当前层节点的个数，每访问到一个节点就将其值存入答案列表中对应的位置，并将其左右子节点存入队列。



## 代码

递归，前序遍历：

```python
class Solution:
    def levelOrder(self, root: TreeNode) -> List[List[int]]:
        if not root: return []
        ans = []
        
        def helper(node, d):
            if not node: return
            if len(ans) == d:
                ans.append([])
            ans[d].append(node.val)
            helper(node.left, d+1)
            helper(node.right, d+1)
            return

        helper(root, 0)
        return ans
```



BFS：

```python
class Solution:
    def levelOrder(self, root: TreeNode) -> List[List[int]]:
        queue = collections.deque()
        queue.append(root)
        ans = []
        while queue:
            s = len(queue)
            d = []
            for _ in range(s):
                node = queue.popleft()
                if not node:
                    continue
                d.append(node.val)
                queue.append(node.left)
                queue.append(node.right)
            if d:
                ans.append(d)

        return ans
```



# 107. 二叉树的层次遍历 II

https://leetcode-cn.com/problems/binary-tree-level-order-traversal-ii/

## 题目描述

给定一个二叉树，返回其节点值自底向上的层次遍历。 （即按从叶子节点所在层到根节点所在的层，逐层从左向右遍历）

例如：
给定二叉树 `[3,9,20,null,null,15,7]`,

        3
       / \
      9  20
        /  \
       15   7

返回其自底向上的层次遍历为：

```
[
  [15,7],
  [9,20],
  [3]
]
```



## 思路

与层次遍历一样，只是输出的时候倒序输出即可，也是有两种方法：递归前序遍历和BFS。



## 代码

前序遍历，递归：

```python
class Solution:
    def levelOrderBottom(self, root: TreeNode) -> List[List[int]]:
        ans = []
        self.helper(root, 0, ans)
        return ans[::-1]

    def helper(self, node, depth, ans):
        if not node:
            return
        if len(ans) == depth:
            ans.append([])
        ans[depth].append(node.val)
        self.helper(node.left, depth+1, ans)
        self.helper(node.right, depth+1, ans)
        return
```



BFS：

```python
class Solution:
    def levelOrderBottom(self, root: TreeNode) -> List[List[int]]:
        queue = collections.deque()
        queue.append(root)
        ans = []
        while queue:
            size = len(queue)
            t = []
            for _ in range(size):
                p = queue.popleft()
                if not p:
                    continue
                t.append(p.val)
                queue.append(p.left)
                queue.append(p.right)
            if t:
                ans.append(t)
        return ans[::-1]
```



# 110. 平衡二叉树

https://leetcode-cn.com/problems/balanced-binary-tree/

## 题目描述

给定一个二叉树，判断它是否是高度平衡的二叉树。

本题中，一棵高度平衡二叉树定义为：

*一个二叉树每个节点 的左右两个子树的高度差的绝对值不超过1。*

示例 1:

给定二叉树 `[3,9,20,null,null,15,7]`

        3
       / \
      9  20
        /  \
       15   7


返回 `true` 。

示例 2:

给定二叉树 `[1,2,2,3,3,null,null,4,4]`

           1
          / \
         2   2
        / \
       3   3
      / \
     4   4


返回 `false` 。



## 思路

有两种解法，分别是自顶向下和自底向上。

自顶向下(暴力法)比较容易想到，首先计算根节点左右子树的深度，然后比较它们的差是否小于2，然后计算根节点的左右子节点的左右子树的深度，以此类推。

自底向上则是在计算节点的深度时就判断其左右子树的深度大小，若大于2则直接返回，效率要比自顶向下高很多。



## 代码

自顶向下：

```python
class Solution:
    def isBalanced(self, root: TreeNode) -> bool:
        if not root: return True
        return abs(self.helper(root.left) - self.helper(root.right)) < 2 and \
            self.isBalanced(root.left) and self.isBalanced(root.right)

    def helper(self, node):
        if not node: return 0
        return max(self.helper(node.left), self.helper(node.right))+1
```



自底向上：

```python
class Solution:
    def isBalanced(self, root: TreeNode) -> bool:
        return self.helper(root) != -1

    def helper(self, node):
        if not node: return 0
        left = self.helper(node.left)
        if left == -1: return -1
        right = self.helper(node.right)
        if right == -1: return -1
        return max(left, right)+1 if abs(left - right) < 2 else -1
```



# 112. 路径总和

https://leetcode-cn.com/problems/path-sum/

## 题目描述

给定一个二叉树和一个目标和，判断该树中是否存在根节点到叶子节点的路径，这条路径上所有节点值相加等于目标和。

说明: 叶子节点是指没有子节点的节点。

示例: 
给定如下二叉树，以及目标和 `sum = 22`，

              5
             / \
            4   8
           /   / \
          11  13  4
         /  \      \
        7    2      1

返回 `true`, 因为存在目标和为 22 的根节点到叶子节点的路径 `5->4->11->2`。



## 思路

递归，前序遍历二叉树，统计路径总和，到叶子节点时判断路径总和与目标和 `sum`  是否相等。



## 代码

```python
class Solution:
    def hasPathSum(self, root: TreeNode, sum: int) -> bool:
        if not root: return False

        def helper(node, s):
            if not node: return False
            t = node.val + s
            if not node.left and not node.right:
                if t == sum:
                    return True
                else:
                    return False
            else: return helper(node.left, t) or helper(node.right, t)

        return helper(root, 0)
```



# 120. 三角形最小路径和

https://leetcode-cn.com/problems/triangle/

## 题目描述

给定一个三角形，找出自顶向下的最小路径和。每一步只能移动到下一行中相邻的结点上。

**相邻的结点** 在这里指的是 `下标` 与 `上一层结点下标` 相同或者等于 `上一层结点下标 + 1` 的两个结点。

 

例如，给定三角形：

```
[
     [2],
    [3,4],
   [6,5,7],
  [4,1,8,3]
]
```

自顶向下的最小路径和为 `11`（即，2 + 3 + 5 + 1 = 11）。

 

说明：

如果你可以只使用 O(n) 的额外空间（n 为三角形的总行数）来解决这个问题，那么你的算法会很加分。



## 思路

动态规划：

* `dp[i][j]` 表示第 `i` 行第 `j` 列节点到最底层的最小路径和，自底向上计算。
* `dp[i][j] = triangle[i][j] + min(dp[i+1][j], dp[i+1][j+1])`.

计算完后 `dp[0][0]` 为最终答案。



## 代码

```python
class Solution:
    def minimumTotal(self, triangle: List[List[int]]) -> int:
        dp = triangle[:]
        for i in range(len(triangle)-2, -1, -1):
            for j in range(i+1):
                dp[i][j] = triangle[i][j] + min(dp[i+1][j], dp[i+1][j+1])
        return dp[0][0]
```



# 121. 买卖股票的最佳时机

https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/

## 题目描述

给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。

如果你最多只允许完成一笔交易（即买入和卖出一支股票一次），设计一个算法来计算你所能获取的最大利润。

注意：你不能在买入股票前卖出股票。

 

示例 1:

```
输入: [7,1,5,3,6,4]
输出: 5
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格；同时，你不能在买入前卖出股票。
```

示例 2:

```
输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。
```



## 思路

因为有时间因素，因此不能简单地用最大值减去最小值。可以用维护两个变量`minp`和`maxp`，其中`minp`记录到当前为止的最小价格，`maxp`记录当前为止的最大差价。通过对数组的一遍扫描，每扫描到一个值就与`minp`作比较、计算当前值与`minp`的差价，若大于`maxp`则更新。



## 代码

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        maxp = 0
        if len(prices) > 0:
            minp = prices[0]
            for price in prices:
                if price < minp:
                    minp = price
                if price - minp > maxp:
                    maxp = price - minp
        return maxp
```



# 152. 乘积最大子数组

https://leetcode-cn.com/problems/maximum-product-subarray/

## 题目描述

给你一个整数数组 `nums` ，请你找出数组中乘积最大的连续子数组（该子数组中至少包含一个数字），并返回该子数组所对应的乘积。

示例 1:

```
输入: [2,3,-2,4]
输出: 6
解释: 子数组 [2,3] 有最大乘积 6。
```

示例 2:

```
输入: [-2,0,-1]
输出: 0
解释: 结果不能为 2, 因为 [-2,-1] 不是子数组。
```



## 思路

可以用动态规划。因为负负得正，所以当前的数为负的话，与前一个数的最小值相乘有可能得到比最大值更大的数。因此需要维护两个变量：当前的最大值和最小值。

动态方程如下：
$$
\begin{aligned}
maxDP[i+1] &= \max(dmax[i+1] \cdot nums[i], \  nums[i],\  dmin[i] \cdot nums[i])\\
minDP[i+1] &= \min(dmax[i+1] \cdot nums[i],\  nums[i],\  dmin[i] \cdot nums[i])\\
DP[i+1] &= \max(DP[i],\ maxDP[i+1])
\end{aligned}
$$
当`nums[i]`为0时，`dmax`和`dmin`都为0，于是需要从`nums[i+1]`重新开始。



## 代码

```python
class Solution:
    def maxProduct(self, nums: List[int]) -> int:
        if len(nums)==0:
            return 0
        elif len(nums)==1:
            return nums[0]
        m = dmax = dmin = nums[0]
        for i in range(1, len(nums)):
            tmp = dmax
            dmax = max(max(dmax*nums[i], nums[i]), dmin*nums[i])
            dmin = min(min(tmp*nums[i], nums[i]), dmin*nums[i])
            m = max(dmax, m)
        return m
```



# 167. 两数之和 II - 输入有序数组

https://leetcode-cn.com/problems/two-sum-ii-input-array-is-sorted/

## 题目描述

给定一个已按照升序排列 的有序数组，找到两个数使得它们相加之和等于目标数。

函数应该返回这两个下标值 index1 和 index2，其中 index1 必须小于 index2。

说明:

* 返回的下标值（index1 和 index2）不是从零开始的。
* 你可以假设每个输入只对应唯一的答案，而且你不可以重复使用相同的元素。

示例:

```
输入: bers = [2, 7, 11, 15], target = 9
输出: [1,2]
解释: 2 与 7 之和等于目标数 9 。因此 index1 = 1, index2 = 2 。
```



## 思路

遍历数组，将访问过的元素的索引存储在哈希表 `d` 中，`d[val]` 存储的是值为 `val` 的元素的索引。

* 每访问到一个元素，计算其与目标数的差值 `k = target - val`；
* 在 `d` 中查找 `k`，若 `k` 存在则直接返回 `[d[k], val的index]`（后访问的 `index` 肯定比先访问的大）；
* 若 `k` 不存在则将 `val` 的 `index` 存储在哈希表中。

时间复杂度 $O(N)$，遍历了一次数组；空间复杂度 $O(N)$，用到了一个哈希表存储数组中的元素的索引值。



## 代码

```python
class Solution:
    def twoSum(self, numbers: List[int], target: int) -> List[int]:
        n = len(numbers)
        d = dict()
        for i in range(n):
            k = target - numbers[i]
            if k in d:
                return [d[k], i+1]
            d[numbers[i]] = i + 1
```



# 169. 多数元素

https://leetcode-cn.com/problems/majority-element/

## 题目描述

给定一个大小为 n 的数组，找到其中的多数元素。多数元素是指在数组中出现次数大于 `⌊ n/2 ⌋` 的元素。

你可以假设数组是非空的，并且给定的数组总是存在多数元素。

 

示例 1:

```
输入: [3,2,3]
输出: 3
```


示例 2:

```
输入: [2,2,1,1,1,2,2]
输出: 2
```



## 思路

这里可以使用摩尔投票法：

候选人`k`初始化为数组第一个元素`nums[0]`，票数`cnt`初始化为1。从第二个元素`nums[1]`开始遍历，遇到与候选人相同的数则把票数加1，遇到不同的则把票数减1，如果票数减完之后为0，则更换当前数`nums[i]`为候选人并把票数重设为1。

因为多数元素的个数肯定比其它元素的个数之和多，因此其票数在最后肯定是`>= 1`的。



## 代码

```python
class Solution:
    def majorityElement(self, nums: List[int]) -> int:
        if len(nums) < 0:
            return 0
        cnt = 1
        k = nums[0]
        for i in range(1, len(nums)):
            if nums[i] == k:
                cnt += 1
            else:
                cnt -= 1
                if cnt == 0:
                    cnt = 1
                    k = nums[i]
        return k
```



# 174. 地下城游戏

https://leetcode-cn.com/problems/dungeon-game/

## 题目描述

一些恶魔抓住了公主（**P**）并将她关在了地下城的右下角。地下城是由 M x N 个房间组成的二维网格。我们英勇的骑士（**K**）最初被安置在左上角的房间里，他必须穿过地下城并通过对抗恶魔来拯救公主。

骑士的初始健康点数为一个正整数。如果他的健康点数在某一时刻降至 0 或以下，他会立即死亡。

有些房间由恶魔守卫，因此骑士在进入这些房间时会失去健康点数（若房间里的值为负整数，则表示骑士将损失健康点数）；其他房间要么是空的（房间里的值为 0），要么包含增加骑士健康点数的魔法球（若房间里的值为正整数，则表示骑士将增加健康点数）。

为了尽快到达公主，骑士决定每次只向右或向下移动一步。

 

**编写一个函数来计算确保骑士能够拯救到公主所需的最低初始健康点数。**

例如，考虑到如下布局的地下城，如果骑士遵循最佳路径 `右 -> 右 -> 下 -> 下`，则骑士的初始健康点数至少为 **7**。

![](http://images.yingwai.top/picgo/174f1.png)


说明:

* 骑士的健康点数没有上限。

* 任何房间都可能对骑士的健康点数造成威胁，也可能增加骑士的健康点数，包括骑士进入的左上角房间以及公主被监禁的右下角房间。



## 思路

反向动态规划，从右下往左上遍历列表，用 `dp[i][j]` 来表示从房间 `(i, j)` 到达终点的最低初始健康点数，则可以得到

* 此房间到终点的 `dp` 值为：此房间右边和下边的房间的 `dp` 值中较小的那一个，加上此房间需要消耗的生命值 `dungeon[i][j]` (注意初始值不能小于 **1**)；
* `dp[i][j] = max(min(dp[i+1][j], dp[i][j+1]) - dungeon[i][j], 1)`.

对于最后一行或一列中，除终点以外的元素 `dp[i][j]` 要用到的 `dp[i+1][j]` 和 `dp[i][j+1]`，将它们赋值为无穷大；而对于终点 `dp[m-1][n-1]`，将 `dp[m][n-1]` 和 `dp[m-1][n]` 赋值为 1。



## 代码

```python
class Solution:
    def calculateMinimumHP(self, dungeon: List[List[int]]) -> int:
        m, n = len(dungeon), len(dungeon[0])
        BIG = float("inf")
        dp = [[BIG] * (n + 1) for _ in range(m + 1)]
        dp[m-1][n] = dp[m][n-1] = 1
        for i in range(m-1, -1, -1):
            for j in range(n-1, -1, -1):
                dp[i][j] = max(min(dp[i+1][j], dp[i][j+1]) - dungeon[i][j], 1)
        return dp[0][0]
```



# 198. 打家劫舍

https://leetcode-cn.com/problems/house-robber/

## 题目描述

你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警。

给定一个代表每个房屋存放金额的非负整数数组，计算你不触动警报装置的情况下，一夜之内能够偷窃到的最高金额。

示例 1:

```
输入: [1,2,3,1]
输出: 4
解释: 偷窃 1 号房屋 (金额 = 1) ，然后偷窃 3 号房屋 (金额 = 3)。
     偷窃到的最高金额 = 1 + 3 = 4 。
```

示例 2:

```
输入: [2,7,9,3,1]
输出: 12
解释: 偷窃 1 号房屋 (金额 = 2), 偷窃 3 号房屋 (金额 = 9)，接着偷窃 5 号房屋 (金额 = 1)。
     偷窃到的最高金额 = 2 + 9 + 1 = 12 。
```



## 思路

动态规划，用 $\rm dp[i]$ 表示前 $\rm i$ 个房间能偷到的最大值，根据题目条件有以下动态方程：
$$
\rm dp[i] = \max (dp[i-1], dp[i-2] + nums[i])
$$


## 代码

```python
class Solution:
    def rob(self, nums: List[int]) -> int:
        if len(nums) <= 0:
            return 0
        elif len(nums) == 1:
            return nums[0]
        dp = []
        dp.append(nums[0])
        dp.append(max(nums[0], nums[1]))
        for i in range(2, len(nums)):
            dp.append(max(dp[i-1], dp[i-2] + nums[i]))
        return dp[len(nums)-1]
```



# 257. 二叉树的所有路径

https://leetcode-cn.com/problems/binary-tree-paths/

## 题目描述

给定一个二叉树，返回所有从根节点到叶子节点的路径。

说明: 叶子节点是指没有子节点的节点。

示例:

```
输入:

   1
 /   \
2     3
 \
  5

输出: ["1->2->5", "1->3"]

解释: 所有根节点到叶子节点的路径为: 1->2->5, 1->3
```



## 思路

递归，用一个字符串 `s` 记录当前的路径，访问到叶子节点时将其加入最终结果的列表 `ans` 中。访问到叶子节点或空节点时返回。



## 代码

```python
class Solution:
    def binaryTreePaths(self, root: TreeNode) -> List[str]:
        ans = []
        s = ""
        self.helper(root, ans, s, 1)
        return ans

    def helper(self, node, ans, s, depth):
        if not node:
            return
        s = s + "->" + str(node.val) if depth > 1 else s + str(node.val)
        if not node.left and not node.right:
            ans.append(s)
            return
        self.helper(node.left, ans, s, depth+1)
        self.helper(node.right, ans, s, depth+1)
        return
```



# 287. 寻找重复数

https://leetcode-cn.com/problems/find-the-duplicate-number/

## 题目描述

给定一个包含 n + 1 个整数的数组 nums，其数字都在 1 到 n 之间（包括 1 和 n），可知至少存在一个重复的整数。假设只有一个重复的整数，找出这个重复的数。

示例 1:

```
输入: [1,3,4,2,2]
输出: 2
```


示例 2:

```
输入: [3,1,3,4,2]
输出: 3
```

说明：

1. 不能更改原数组（假设数组是只读的）。
2. 只能使用额外的 $O(1)$ 的空间。
3. 时间复杂度小于 $O(n^2)$ 。
4. 数组中只有一个重复的数字，但它可能不止重复出现一次。



## 思路

由于题目限制了空间，所以打表法之类的方法就无法使用。可以用二分法：

对于给定题目条件的数组 `nums`，设`mid`为 1 到 n 的中位数。扫描数组，若数组中小于等于`mid`的数的数量严格大于`mid`，则可以确定重复的数就在 1 到`mid`之间，反之则在`mid`到 n 之间。



## 代码

```python
class Solution:
    def findDuplicate(self, nums: List[int]) -> int:
        size = len(nums)
        left = 1
        right = size - 1
        while left < right:
            cnt = 0
            mid = left + (right - left) // 2
            for num in nums:
                if num <= mid:
                    cnt += 1
            if cnt > mid:
                right = mid
            else:
                left = mid + 1
        return left
```



# 300. 最长上升子序列

https://leetcode-cn.com/problems/longest-increasing-subsequence/

## 题目描述

给定一个无序的整数数组，找到其中最长上升子序列的长度。

示例:

```
输入: [10,9,2,5,3,7,101,18]
输出: 4 
解释: 最长的上升子序列是 [2,3,7,101]，它的长度是 4。
```


说明:

* 可能会有多种最长上升子序列的组合，你只需要输出对应的长度即可。
* 你算法的时间复杂度应该为 $O(n^2)$。

进阶: 你能将算法的时间复杂度降低到 $O(n \log{n})$ 吗?



## 思路

$O(n \log{n})$ 还没想到，这里介绍 $O(n^2)$ 的动态规划方法：

用 $\rm dp[i]$ 表示以第 $\rm i$ 个元素结尾的最长上升子序列的长度，则可以得到以下的状态转移方程：
$$
\rm dp[i] = \max(dp[i], dp[j] + 1),\ nums[j] < nums[i],\  j < i.
$$
扫描完整个数组后，$\rm dp$ 数组中的最大值即为结果。



## 代码

```python
class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        if len(nums) <= 0:
            return 0
        dp = []     # 存以第i个元素结尾的最长上升子序列的长度
        for i in range(len(nums)):
            dp.append(1)
            for j in range(i):
                if nums[j] < nums[i]:
                    dp[i] = max(dp[i], dp[j]+1)
        return max(dp)
```



# 309. 最佳买卖股票时机含冷冻期

https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/

## 题目描述

给定一个整数数组，其中第 i 个元素代表了第 i 天的股票价格 。

设计一个算法计算出最大利润。在满足以下约束条件下，你可以尽可能地完成更多的交易（多次买卖一支股票）:

* 你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。
* 卖出股票后，你无法在第二天买入股票 (即冷冻期为 1 天)。

示例:

```
输入: [1,2,3,0,2]
输出: 3 
解释: 对应的交易状态为: [买入, 卖出, 冷冻期, 买入, 卖出]
```



## 思路

动态规划，用两个数组来存储状态：

1. `hold[i]` 表示在第 `i` 天结束时持有股票、此时的最大收益，有两种情况：
   * 昨天持有股票，今天休息；
   * 前天卖出，今天买入；
   * `hold[i] = max(hold[i-1], unhold[i-2] - prices[i])`。
2. `unhold[i]` 表示在第 `i` 天结束时未持有股票、此时的最大收益，也有两种情况：
   * 昨天也没持有股票，今天休息；
   * 昨天持有，今天卖出；
   * `unhold[i] = max(unhold[i-1], hold[i-1] + prices[i])`。

最终结果是 `unhold[n-1]`。

初始情况：

* 第 `0` 天持有股票即在当天买入股票，`hold[0] = -prices[0]`；
* 第 `1` 天持有股票有前一天买入或是当天买入两种情况，`hold[1] = max(-prices[0], -prices[1])`；
* 第 `0` 天未持有股票即当天休息，`unhold[0] = 0`。



## 代码

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        if len(prices) < 2: return 0
        hold, unhold = [], []
        hold.append(-prices[0])
        unhold.append(0)
        for i in range(1, len(prices)):
            if i == 1:
                hold.append(max(-prices[0], -prices[1]))
            else:
                hold.append(max(hold[i-1], unhold[i-2] - prices[i]))
            unhold.append(max(unhold[i-1], hold[i-1] + prices[i]))        
        return unhold[len(prices) - 1]
```



# 312. 戳气球

https://leetcode-cn.com/problems/burst-balloons/

## 题目描述

有 `n` 个气球，编号为 `0` 到 `n-1`，每个气球上都标有一个数字，这些数字存在数组 `nums` 中。

现在要求你戳破所有的气球。如果你戳破气球 `i` ，就可以获得 `nums[left] * nums[i] * nums[right]` 个硬币。 这里的 `left` 和 `right` 代表和 `i` 相邻的两个气球的序号。注意当你戳破了气球 `i` 后，气球 `left` 和气球 `right` 就变成了相邻的气球。

求所能获得硬币的最大数量。

说明:

* 你可以假设 `nums[-1] = nums[n] = 1`，但注意它们不是真实存在的所以并不能被戳破。
* 0 ≤ `n` ≤ 500, 0 ≤ `nums[i]` ≤ 100

示例:

```
输入: [3,1,5,8]
输出: 167 
解释: nums = [3,1,5,8] --> [3,5,8] -->   [3,8]   -->  [8]  --> []
     coins =  3*1*5      +  3*5*8    +  1*3*8      + 1*8*1   = 167
```



## 思路

动态规划，用 `dp[i][j]` 表示戳破气球 `i` 和气球 `j` 之间所有气球所能获得硬币的最大数量：

* 当气球 `i` 和 `j` 相邻时，`dp[i][j]` 为0；
* 令 `k` 为气球 `i` 和气球 `j` 之间最后戳破的气球序号，对于每个 `i` 和 `j`（`i` 小于 `j`），遍历所有的 `k`，取其中的最大值赋值给 `dp[i][j]`，可得以下状态转移方程：
* `dp[i][j] = max(dp[i][j], dp[i][k] + dp[k][j] + nums[i]*nums[k]*nums[j])`.

注意要令 `nums` 的两个边界赋值为1。

时间复杂度为 $O(N^3)$。



## 代码

```python
class Solution:
    def maxCoins(self, nums: List[int]) -> int:
        points = [1] + nums + [1]
        n = len(nums)
        dp = [[0] * (n+2) for _ in range(n+2)]
        for i in range(n, -1, -1):
            for j in range(i+1, n+2):
                for k in range(i+1, j):
                    dp[i][j] = max(dp[i][j], dp[i][k] + dp[k][j] + points[k]*points[i]*points[j])
        return dp[0][-1]
```



# 315. 计算右侧小于当前元素的个数

https://leetcode-cn.com/problems/count-of-smaller-numbers-after-self/

## 题目描述

给定一个整数数组 nums，按要求返回一个新数组 counts。数组 counts 有该性质： `counts[i]` 的值是  `nums[i]` 右侧小于 `nums[i]` 的元素的数量。

示例:

```
输入: [5,2,6,1]
输出: [2,1,1,0] 
解释:
5 的右侧有 2 个更小的元素 (2 和 1).
2 的右侧仅有 1 个更小的元素 (1).
6 的右侧有 1 个更小的元素 (1).
1 的右侧有 0 个更小的元素.
```



## 思路

二分查找，从后向前遍历 `nums`，使用一个辅助数组 `sorted_nums` 保存 `nums[i]` 右侧的元素的升序排列，`counts[i]` 的值即为 `nums[i]` 插入 `sorted_nums` 后的索引：

1. 在 `sorted_nums` 中查找第一个小于等于 `nums[i] ` 的元素的索引，此索引即为`nums` 中 `nums[i]` 右侧小于该元素的元素数量；
2. 将其添加到 `ans` 数组中；
3. 将 `nums[i]` 插入到 `sorted_nums` 中的对应位置，继续遍历，直到遍历完整个数组。

最后 `ans` 的倒序即为答案。



## 代码

```python
class Solution:
    def countSmaller(self, nums: List[int]) -> List[int]:
        sorted_nums, ans = [], []
        for i in reversed(range(len(nums))): 
            t = self.findIndex(nums[i], sorted_nums)
            sorted_nums.insert(t, nums[i])
            ans.append(t)
        return ans[::-1]
    
    def findIndex(self, n, sorted_nums):
        if len(sorted_nums) == 0:
            return 0
        left, right = 0, len(sorted_nums)-1
        mid = 0
        while left < right:
            mid = left + right >> 1
            if n > sorted_nums[mid]:
                left = mid + 1
            else:
                right = mid
        if n > sorted_nums[left]:
            left += 1
        return left
```



# 322. 零钱兑换

https://leetcode-cn.com/problems/coin-change/

## 题目描述

给定不同面额的硬币 coins 和一个总金额 amount。编写一个函数来计算可以凑成总金额所需的最少的硬币个数。如果没有任何一种硬币组合能组成总金额，返回 `-1`。

 

示例 1:

```
输入: coins = [1, 2, 5], amount = 11
输出: 3 
解释: 11 = 5 + 5 + 1
```

示例 2:

```
输入: coins = [2], amount = 3
输出: -1
```


说明:
你可以认为每种硬币的数量是无限的。



## 思路

首先想到了贪心，但会复杂一点，迟点再研究一下，这里用了动态规划。根据题意，要求 `amount` 的最少硬币数，可以先对 `coins` 中的每一个面额 `coin` 进行遍历，求子问题 $dp(amount-coin)$ 的最小值。

可以列出状态转移方程
$$
dp(n) = \left\{ \begin{array}{lcl}
 -1, & n < 0\\
 0, & n = 0\\
 \min\{dp(n-coin)+1|coin \in coins\}, & n > 0
\end{array}\right.
$$



## 代码

```python
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        memo = dict()
        
        def dp(n):
            if n in memo:
                return memo[n]
            if n == 0:
                return 0
            if n < 0:
                return -1
            ans = float("inf")
            for coin in coins:
                subproblem = dp(n - coin)
                if subproblem == -1:
                    continue
                ans = min(ans, subproblem + 1)
            memo[n] = ans if ans!=float(inf) else -1
            return memo[n]
        
        return dp(amount)
```



# 350. 两个数组的交集 II

https://leetcode-cn.com/problems/intersection-of-two-arrays-ii/

## 题目描述

给定两个数组，编写一个函数来计算它们的交集。

示例 1:

```
输入: nums1 = [1,2,2,1], nums2 = [2,2]
输出: [2,2]
```

示例 2:

```
输入: nums1 = [4,9,5], nums2 = [9,4,9,8,4]
输出: [4,9]
```

说明：

* 输出结果中每个元素出现的次数，应与元素在两个数组中出现的次数一致。
* 我们可以不考虑输出结果的顺序。

进阶:

* 如果给定的数组已经排好序呢？你将如何优化你的算法？
* 如果 nums1 的大小比 nums2 小很多，哪种方法更优？
* 如果 nums2 的元素存储在磁盘上，磁盘内存是有限的，并且你不能一次加载所有的元素到内存中，你该怎么办？



## 思路

两种方法：

* 遍历 `nums1`，用一个字典存储 `d` 其中元素的个数，然后遍历 `nums2` 若扫描到的元素 `nums2[i]` 在字典中存在且 `d[nums2[i]] > 0`，则将该元素加入 `ans` 并将 `d[nums2[i]]` 自减 1。
* 先对两个数组进行排序，然后用两个指针 `i, j` 分别指向排序完的数组的头元素。从头元素开始扫描，当 `nums1[i] > nums2[j]` 时 `j` 往后移，反之 `i` 往后移，两个元素相等时将其加入 `ans`，当遍历完其中一个数组后结束循环。



## 代码

哈希表：

```python
class Solution:
    def intersect(self, nums1: List[int], nums2: List[int]) -> List[int]:
        d = dict()
        ans = []
        for num in nums1:
            if num in d:
                d[num] += 1
            else:
                d[num] = 1
        for num in nums2:
            if num in d and d[num] > 0:
                ans.append(num)
                d[num] -= 1
        return ans
```



排序方法：

```python
class Solution:
    def intersect(self, nums1: List[int], nums2: List[int]) -> List[int]:
        nums1.sort()
        nums2.sort()
        i = j = 0
        ans = []
        while i < len(nums1) and j < len(nums2):
            if nums1[i] == nums2[j]:
                ans.append(nums1[i])
                i += 1
                j += 1
            elif nums1[i] > nums2[j]:
                j += 1
            else:
                i += 1
        return ans
```



# 394. 字符串解码

https://leetcode-cn.com/problems/decode-string/

## 题目描述

给定一个经过编码的字符串，返回它解码后的字符串。

编码规则为: `k[encoded_string]`，表示其中方括号内部的 encoded_string 正好重复 k 次。注意 k 保证为正整数。

你可以认为输入字符串总是有效的；输入字符串中没有额外的空格，且输入的方括号总是符合格式要求的。

此外，你可以认为原始数据不包含数字，所有的数字只表示重复的次数 k ，例如不会出现像 `3a` 或 `2[4]` 的输入。

示例:

```
s = "3[a]2[bc]", 返回 "aaabcbc".
s = "3[a2[c]]", 返回 "accaccacc".
s = "2[abc]3[cd]ef", 返回 "abcabccdcdcdef".
```



## 思路

这里可以使用栈，对字符串按顺序扫描：

1. 将右括号以外的字符全部入栈，直到扫描到右括号；
2. 扫描到右括号则开始退栈，保存在一个字符串`ss`中，直到遇到左括号；
3. 根据题目的条件，在左括号前面的一定是数字，此时就可以统计当前字符串出现的次数`t`，把`ss`复制`t`次重新入栈；
4. 最后把栈中所有元素拼接在一起即可。



## 代码

```python
class Solution:
    def decodeString(self, s: str) -> str:
        if len(s) <= 0:
            return ""
        ans = ""
        tmps = []   # 栈
        i = 1
        tmps.append(s[0])
        ss, t = "", ""	# 分别存当前字符串和当前的次数
        while i < len(s):
            w = s[i]
            if w == ']':	# 扫描到右括号则开始退栈，直到遇到左括号
                while tmps[-1] != '[':
                    ss = tmps.pop() + ss
                tmps.pop()      # 将左括号退栈
                while len(tmps) > 0 and tmps[-1].isdigit():		# 统计次数
                    t = tmps.pop() + t
                tmps.append(ss*int(t))
                ss, t = "", ""
            else:
                tmps.append(w)
            i += 1
        for tmp in tmps:
            ans = ans + tmp
        return ans
```



# 409. 最长回文串

https://leetcode-cn.com/problems/longest-palindrome/

## 题目描述

给定一个包含大写字母和小写字母的字符串，找到通过这些字母构造成的最长的回文串。

在构造过程中，请注意区分大小写。比如 `"Aa"` 不能当做一个回文字符串。

注意:
假设字符串的长度不会超过 1010。

示例 1:

```
输入:
"abccccdd"

输出:
7

解释:
我们可以构造的最长的回文串是"dccaccd", 它的长度是 7。
```



## 思路

由题意可知回文串中奇数的字母只能出现在中间，即只能选取一个来构造回文串。因此可以先统计每个字母在字符串中出现的个数存在字典中，然后遍历字典，将数量为偶数的直接累加；数量为奇数的则判断前面是否已出现了奇数，若不是则直接累加，若是则只选取偶数数量的当前字母，即把当前数量 $-1$ 再累加。



## 代码

```python
class Solution:
    def longestPalindrome(self, s: str) -> int:
        d = dict()
        ans = 0
        flag = 0    # 判断前面有无加奇数
        for w in s:
            if w in d:
                d[w] += 1
            else:
                d[w] = 1
        for value in d.values():
            if value%2==0:
                ans += value
            elif flag:
                ans += value - 1
            else:
                ans += value
                flag = 1
        return ans
```



# 563. 二叉树的坡度

https://leetcode-cn.com/problems/binary-tree-tilt/

## 题目描述

给定一个二叉树，计算整个树的坡度。

一个树的节点的坡度定义即为，该节点左子树的结点之和和右子树结点之和的差的绝对值。空结点的的坡度是0。

整个树的坡度就是其所有节点的坡度之和。

 

示例：

```
输入：
         1
       /   \
      2     3
输出：1
解释：
结点 2 的坡度: 0
结点 3 的坡度: 0
结点 1 的坡度: |2-3| = 1
树的坡度 : 0 + 0 + 1 = 1
```




提示：

任何子树的结点的和不会超过 32 位整数的范围。
坡度的值不会超过 32 位整数的范围。



## 思路

递归，后序遍历二叉树，用一个变量 `tilt` 存储累计坡度，访问节点时更新 `tilt = tilt + |(左子树节点之和 - 右子树节点之和)|`，然后返回以当前节点为根节点的二叉树节点之和。

递归边界为访问到空节点，返回 0。



## 代码

```python
class Solution:
    def findTilt(self, root: TreeNode) -> int:
        tilt = [0]
        self.recur(root, tilt)
        return tilt[0]

    def recur(self, node, tilt):
        if not node:
            return 0
        left = self.recur(node.left, tilt)
        right = self.recur(node.right, tilt)
        tilt[0] += abs(left - right)
        return left + right + node.val
```



# 623. 在二叉树中增加一行

https://leetcode-cn.com/problems/add-one-row-to-tree/

## 题目描述

给定一个二叉树，根节点为第1层，深度为 1。在其第 `d` 层追加一行值为 `v` 的节点。

添加规则：给定一个深度值 `d` （正整数），针对深度为 `d-1` 层的每一非空节点 `N`，为 `N` 创建两个值为 `v` 的左子树和右子树。

将 `N` 原先的左子树，连接为新节点 `v` 的左子树；将 `N` 原先的右子树，连接为新节点 `v` 的右子树。

如果 `d` 的值为 1，深度 d - 1 不存在，则创建一个新的根节点 `v`，原先的整棵树将作为 `v` 的左子树。

示例 1:

```
输入: 
二叉树如下所示:
       4
     /   \
    2     6
   / \   / 
  3   1 5   

v = 1

d = 2

输出: 
       4
      / \
     1   1
    /     \
   2       6
  / \     / 
 3   1   5  
```

示例 2:

```
输入: 
二叉树如下所示:
      4
     /   
    2    
   / \   
  3   1    

v = 1

d = 3

输出: 
      4
     /   
    2
   / \    
  1   1
 /     \  
3       1
```


注意:

输入的深度值 d 的范围是：[1，二叉树最大深度 + 1]。
输入的二叉树至少有一个节点。



## 思路

深度优先搜索(递归)，在访问到深度为 `d-1` 的节点则在它们下面增加子节点然后返回。

注意 `d = 1` 的特殊情况和访问到空节点的情况。



## 代码

```python
class Solution:
    def addOneRow(self, root: TreeNode, v: int, d: int) -> TreeNode:
        if d == 1:
            newnode = TreeNode(v)
            newnode.left = root
            root = newnode
        else:
            self.helper(root, d, v, 1)
        return root
    
    def helper(self, node, d, v, depth):
        if not node: return
        if depth == d - 1:
            lnode, rnode = node.left, node.right
            n1, n2 = TreeNode(v), TreeNode(v)
            n1.left, n2.right = lnode, rnode
            node.left, node.right = n1, n2
            return
        self.helper(node.left, d, v, depth + 1)
        self.helper(node.right, d, v, depth + 1)
        return
```



# 637. 二叉树的层平均值

https://leetcode-cn.com/problems/average-of-levels-in-binary-tree/

## 题目描述

给定一个非空二叉树, 返回一个由每层节点平均值组成的数组。

 

示例 1：

```
输入：
    3
   / \
  9  20
    /  \
   15   7
输出：[3, 14.5, 11]
解释：
第 0 层的平均值是 3 ,  第1层是 14.5 , 第2层是 11 。因此返回 [3, 14.5, 11] 。
```



提示：

节点值的范围在32位有符号整数范围内。



## 思路

BFS，套模板，逐层计算均值即可。



## 代码

```python
class Solution:
    def averageOfLevels(self, root: TreeNode) -> List[float]:
        queue = collections.deque()
        queue.append(root)
        ans = []
        if root:
            while queue:
                size = len(queue)
                s = 0
                for _ in range(size):
                    node = queue.popleft()
                    s += node.val
                    if node.left:
                        queue.append(node.left)
                    if node.right:
                        queue.append(node.right)
                if size:
                    ans.append(s/size)
        return ans
```



# 785. 判断二分图

https://leetcode-cn.com/problems/is-graph-bipartite/

## 题目描述

给定一个无向图 `graph`，当这个图为二分图时返回 `true`。

如果我们能将一个图的节点集合分割成两个独立的子集A和B，并使图中的每一条边的两个节点一个来自A集合，一个来自B集合，我们就将这个图称为二分图。

graph将会以邻接表方式给出，`graph[i]` 表示图中与节点 `i` 相连的所有节点。每个节点都是一个在 `0` 到 `graph.length-1` 之间的整数。这图中没有自环和平行边： `graph[i]` 中不存在 `i`，并且 `graph[i]` 中没有重复的值。

```
示例 1:
输入: [[1,3], [0,2], [1,3], [0,2]]
输出: true
解释: 
无向图如下:
0----1
|    |
|    |
3----2
我们可以将节点分成两组: {0, 2} 和 {1, 3}。
```

```
示例 2:
输入: [[1,2,3], [0,2], [0,1,3], [0,2]]
输出: false
解释: 
无向图如下:
0----1
| \  |
|  \ |
3----2
我们不能将节点分割成两个独立的子集。
```



注意:

* `graph` 的长度范围为 `[1, 100]`。
* `graph[i]` 中的元素的范围为 `[0, graph.length - 1]`。
* `graph[i]` 不会包含 `i` 或者有重复的值。
* 图是无向的: 如果 `j` 在 `graph[i]` 里边, 那么 `i` 也会在 `graph[j]` 里边。



## 思路

BFS，遍历所有节点，用 `visited` 数组标记节点是否已经被访问（初始值为 `0`）：

* 对当前未访问的节点 `i`，
  1. 将 `visited[i]` 标记为 `1`；
  2. 把 `i` 的相邻节点 `j` 标记为 `-1`，然后继续对 `j` 的相邻节点标记为 `-visited[j]`，直到所有连接在一起的节点都被访问；
  3. 如果上一步在标记时，`i` 的某个相邻节点 `j` 的标记值 `visited[j] = visited[i]`，则说明无法将图中节点分割，返回 `false`。



## 代码

```python
class Solution:
    def isBipartite(self, graph: List[List[int]]) -> bool:
        visited = [0] * len(graph)
        queue = collections.deque()
        for i in range(len(graph)):
            if visited[i] != 0:
                continue              
            visited[i] = 1
            queue.append(i)
            while queue:
                cur = queue.popleft()
                for node in graph[cur]:
                    if visited[node] == 0:
                        visited[node] = -visited[cur]
                        queue.append(node)
                    elif visited[node] == visited[cur]:
                        return False
        return True
```



# 974. 和可被 K 整除的子数组

https://leetcode-cn.com/problems/subarray-sums-divisible-by-k/

## 题目描述

给定一个整数数组 A，返回其中元素之和可被 K 整除的（连续、非空）子数组的数目。

 

示例：

```
输入：A = [4,5,0,-2,-3,1], K = 5
输出：7
解释：
有 7 个子数组满足其元素之和可被 K = 5 整除：
[4, 5, 0, -2, -3, 1], [5], [5, 0], [5, 0, -2, -3], [0], [0, -2, -3], [-2, -3]
```




提示：

1. `1 <= A.length <= 30000`
2. `-10000 <= A[i] <= 10000`
3. `2 <= K <= 10000`



## 思路

一个前缀和的问题。设 $\rm presum[i]$是数组 $\rm A$ 第 $\rm i$ 个元素的前缀和，那么 $\rm A[i]$ 就可以表示为 $\rm presum[i] - presum[i-1]$，子数组 $\rm A[k]$ 到 $\rm A[i]$ 的和就是 $\rm presum[i] - presum[k-1]$。

题目要求的是满足 $\rm presum[i] - presum[k-1] \bmod K = 0$ 的子数组 $\rm [A[k],...,A[i]]$ 的个数，根据同余定理，可以把问题转换为求同余的 $\rm presum[i]$ 的个数。每扫描到数组中的一个数，就检查哈希表中有没有同余的数组，若有则计算同余的数量，若无把当前模 $\rm K$ 的余数保存在哈希表中，一次遍历即可解决问题。



## 代码

```python
class Solution:
    def subarraysDivByK(self, A: List[int], K: int) -> int:
        if len(A)==0 or K==0:
            return 0
        d = {0:1}
        presum, cnt = 0, 0
        for a in A:
            presum += a
            m = presum % K
            s = d.get(m, 0)
            cnt += s
            d[m] = s + 1
        return cnt
```



# 994. 腐烂的橘子

https://leetcode-cn.com/problems/rotting-oranges/

## 题目描述

在给定的网格中，每个单元格可以有以下三个值之一：

值 `0` 代表空单元格；
值 `1` 代表新鲜橘子；
值 `2` 代表腐烂的橘子。
每分钟，任何与腐烂的橘子（在 4 个正方向上）相邻的新鲜橘子都会腐烂。

返回直到单元格中没有新鲜橘子为止所必须经过的最小分钟数。如果不可能，返回 `-1`。

 

示例 1：

![](http://images.yingwai.top/picgo/oranges.png)

```
输入：[[2,1,1],[1,1,0],[0,1,1]]
输出：4
```



示例 2：

```
输入：[[2,1,1],[0,1,1],[1,0,1]]
输出：-1
解释：左下角的橘子（第 2 行， 第 0 列）永远不会腐烂，因为腐烂只会发生在 4 个正向上。
```

示例 3：

```
输入：[[0,2]]
输出：0
解释：因为 0 分钟时已经没有新鲜橘子了，所以答案就是 0 。
```


提示：

1. `1 <= grid.length <= 10`
2. `1 <= grid[0].length <= 10`
3. `grid[i][j]`仅为`0`、`1`或`2`



## 思路

广度优先搜索，一圈一圈往外腐蚀。



## 代码

```python
class Solution:
    def orangesRotting(self, grid: List[List[int]]) -> int:
        dx = [-1, 1, 0, 0]
        dy = [0, 0, -1, 1]
        rotlist = list()    # 腐烂橘子的队列
        minute = 0
        for i in range(len(grid)):
            for j in range(len(grid[0])):
                if grid[i][j] == 2:
                    rotlist.append([i, j])

        while rotlist:  # BFS循环
            newrotlist = list()
            for rotorange in rotlist:   # 当前腐烂橘子的坐标
                x0 = rotorange[0]
                y0 = rotorange[1]

                for i in range(4):  # 四个相邻方向的橘子腐烂
                    x = x0 + dx[i]
                    y = y0 + dy[i]
                    if 0<=x<len(grid) and 0<=y<len(grid[0]) and grid[x][y]==1:
                        grid[x][y] = 2
                        newrotlist.append([x, y])

            if not newrotlist:
                break

            minute += 1
            rotlist = newrotlist[:]     # 更新腐烂队列

        for row in grid:
                for i in row:
                    if i == 1:  # 还有新鲜的
                        return -1

        return minute
```



# 1305. 两棵二叉搜索树中的所有元素

https://leetcode-cn.com/problems/all-elements-in-two-binary-search-trees/

## 题目描述

给你 root1 和 root2 这两棵二叉搜索树。

请你返回一个列表，其中包含 两棵树 中的所有整数并按 升序 排序。

 

示例 1：

![](http://images.yingwai.top/picgo/lc1305f1.png)

```
输入：root1 = [2,1,4], root2 = [1,0,3]
输出：[0,1,1,2,3,4]
```

示例 2：

```
输入：root1 = [0,-10,10], root2 = [5,1,7,0,2]
输出：[-10,0,0,1,2,5,7,10]
```

示例 3：

```
输入：root1 = [], root2 = [5,1,7,0,2]
输出：[0,1,2,5,7]
```

示例 4：

```
输入：root1 = [0,-10,10], root2 = []
输出：[-10,0,10]
```

示例 5：

![](http://images.yingwai.top/picgo/lc1305f2.png)

```
输入：root1 = [1,null,8], root2 = [8,1]
输出：[1,1,8,8]
```




提示：

* 每棵树最多有 `5000` 个节点。
* 每个节点的值在 `[-10^5, 10^5]` 之间。



## 思路

利用二叉搜索树性质，中序遍历得到每棵树的节点值的升序排列，然后再使用归并排序得到最终的结果数组。

* 时间复杂度 $O(M + N)$，中序遍历和归并排序都是 $O(M + N)$，其中 $M,N$ 分别为两棵树的节点个数；
* 空间复杂度 $O(M+N)$，用到两个额外的数组来存储每棵树的升序排列。



## 代码

```python
class Solution:
    def getAllElements(self, root1: TreeNode, root2: TreeNode) -> List[int]:
        t1, t2 = [], []
        ans = []
        self.helper(root1, t1)
        self.helper(root2, t2)
        i, j = 0, 0
        while i < len(t1) or j < len(t2):
            if i < len(t1) and (j == len(t2) or t1[i] < t2[j]):
                ans.append(t1[i])
                i += 1
            else:
                ans.append(t2[j])
                j += 1
        return ans
        
    def helper(self, node, t):
        if not node: return
        self.helper(node.left, t)
        t.append(node.val)
        self.helper(node.right, t)
        return
```



# 面试题 16.11. 跳水板

https://leetcode-cn.com/problems/diving-board-lcci/

## 题目描述

你正在使用一堆木板建造跳水板。有两种类型的木板，其中长度较短的木板长度为`shorter`，长度较长的木板长度为`longer`。你必须正好使用`k`块木板。编写一个方法，生成跳水板所有可能的长度。

返回的长度需要从小到大排列。

示例：

```
输入：
shorter = 1
longer = 2
k = 3
输出： {3,4,5,6}
```

提示：

* 0 < shorter <= longer
* 0 <= k <= 100000



## 思路

因为每次都必须要正好使用 `k` 块木板，所以有以下公式
$$
ans[i] = shorter\times i + longer \times (k-i)
$$
其中 $0 \leq i \leq k$。

在 $longer > shorter$ 的情况下，为什么每种组合下建造的跳水板长度都是不一样的？考虑以下两种不同的组合：第一种组合，有 $i$ 块短木板，则跳水板的长度是 $shorter \times i + longer \times (k−i)$；第二种组合，有 $j$ 块短木板，则跳水板的长度是 $shorter \times j+longer \times (k−j)$。其中 $0 \leq i<j \leq k$。则两种不同的组合下的跳水板长度之差为：
$$
(shorter \times j+longer \times (k−j)) - (shorter \times i + longer \times (k−i)) = (longer - shorter) \times (i - j)
$$
因为 $longer > shorter$ 且 $i<j$，因此上式 $<0$。

然后要考虑到极端情况：

* $k = 0$：直接输出空列表。
* $longer = shorter$：输出列表中只有一个元素 $ans = [shorter(longer) \times k]$。



## 代码

```python
class Solution:
    def divingBoard(self, shorter: int, longer: int, k: int) -> List[int]:
        ans = []
        if k > 0:
            s, l = k, 0
            if shorter == longer:
                ans.append(shorter*s)
            else:
                while l <= k:
                    ans.append(shorter*s + longer*l)
                    s -= 1
                    l += 1
        return ans
```



# 面试题 17. 12. BiNode

https://leetcode-cn.com/problems/binode-lcci/

## 题目描述

二叉树数据结构 `TreeNode` 可用来表示单向链表（其中 `left` 置空，`right` 为下一个链表节点）。实现一个方法，把二叉搜索树转换为单向链表，要求依然符合二叉搜索树的性质，转换操作应是原址的，也就是在原始的二叉搜索树上直接修改。

返回转换后的单向链表的头节点。

注意：本题相对原题稍作改动

 

示例：

```
输入： [4,2,5,1,3,null,6,0]
输出： [0,null,1,null,2,null,3,null,4,null,5,null,6]
```

提示：

* 节点数量不会超过 100000。



## 思路

中序遍历，用 `pre` 记录上一个处理的节点，访问到一个新的节点 `p` 时，将 `pre.right` 指向 `p` 并把 `p.left` 置空。

注意要将第一个处理的节点作为根节点。



## 代码

```python
class Solution:
    def convertBiNode(self, root: TreeNode) -> TreeNode:
        if not root:
            return
        s = []
        p, pre = root, None
        while s or p:
            while p:
                s.append(p)
                p = p.left
            p = s.pop()
            if not pre:
                root = p
                root.left = None
            else:
                pre.right = p
                p.left = None
            pre = p
            p = p.right
        return root
```



# 面试题 17.13. 恢复空格

https://leetcode-cn.com/problems/re-space-lcci/

## 题目描述

哦，不！你不小心把一个长篇文章中的空格、标点都删掉了，并且大写也弄成了小写。像句子`"I reset the computer. It still didn’t boot!"`已经变成了`"iresetthecomputeritstilldidntboot"`。在处理标点符号和大小写之前，你得先把它断成词语。当然了，你有一本厚厚的词典`dictionary`，不过，有些词没在词典里。假设文章用`sentence`表示，设计一个算法，把文章断开，要求未识别的字符最少，返回未识别的字符数。

注意：本题相对原题稍作改动，只需返回未识别的字符数

 

示例：

```
输入：
dictionary = ["looked","just","like","her","brother"]
sentence = "jesslookedjustliketimherbrother"
输出： 7
解释： 断句后为"jess looked just like tim her brother"，共7个未识别字符。
```

提示：

* `0 <= len(sentence) <= 1000`
* `dictionary`中总字符数不超过 `150000`。
* 你可以认为`dictionary`和`sentence`中只包含小写字母。



## 思路

动态规划，用 $dp[i]$ 表示`sentence`中以第 $index$ 个字符结尾的字符串中未识别的字符数，其中 $i = index + 1$（这里令 $dp[0]=0$，因为可能出现前 $k$ 个字符组成一个词，这样做利于判断）。对于每个 $index$，都对`dictionary`进行一次遍历，用 $len$ 表示`dictionary`中每个词的长度，则可以得到动态方程：

每扫描到一个 $i$，都先把 $dp[i]$ 初始化为 $i$，然后有

当`sentence`中第 $index - len + 1$ 到 第 $index$ 个字符在`dictionary`中时，
$$
dp[i] = \min (dp[i],dp[i-len])
$$
其它情况，
$$
dp[i] = (dp[i], dp[i-1]+1)
$$
最后 $dp$ 数组中的最后一个元素即为结果。



## 代码

```python
class Solution:
    def respace(self, dictionary: List[str], sentence: str) -> int:
        dp = []
        dp.append(0)
        for i in range(1, len(sentence)+1):
            index = i - 1
            t = i
            for word in dictionary:
                l = len(word)
                if index + 1 - l >= 0 and sentence[index-l+1:index+1] in dictionary:
                    t = min(t, dp[i-l])
                    if i - l == 0:
                        break
                t = min(t, dp[i-1]+1)
            dp.append(t)
        return dp[len(sentence)]
```

