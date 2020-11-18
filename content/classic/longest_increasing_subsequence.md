---
title: "最长上升子序列"
date: 2020-11-18T10:47:10+08:00
draft: false
series:
  - Classic
tags:
  - DP
  - TODO
---

## 问题定义

设有序列$\\{a_i\\}$，从中选择一些下标$b_1 < b_2 < b_3 < \cdots < b_k$，
要求满足$a_{b_1} < a_{b_2} < a_{b_3} < \cdots < a_{b_k}$，
问最大的$k$是多少？

为叙述方便，后文简称最长上升子序列为LIS(Longest Increasing Subsequence)。

## 解法1: 动态规划

- 状态定义：`dp[i]`表示以`a[i]`结尾的LIS的长度
- 状态转移：`dp[i] = max(dp[j]+1, dp[i]), (1 <= j < i, a[j] < a[i])`
- 边界处理：`dp[i] = 1 (1 <= i <= n)`
- 时间复杂度：$\mathcal{O} (n^2)$

{{< admonition type=example >}}
[LeetCode 300 最长上升子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/)
```cpp
class Solution {
public:
    int lengthOfLIS(const vector<int>& nums) {
        const int n = nums.size();
        if (n < 2) return n;

        vector<int> dp(n, 1);
        for (int i = 1; i < n; ++i)
            for (int j = 0; j < i; ++j)
                if (nums[j] < nums[i])
                    dp[i] = max(dp[i], dp[j]+1);

        return accumulate(dp.begin(), dp.end(), 1,
                          [] (int a, int b) { return max(a, b); });
    }
};
```
{{< /admonition >}}

## 解法2: 贪心+二分

设`low[i]`表示长度为`i`的LIS的结尾元素的最小值。
对于一个上升子序列，其结尾元素越小，越容易在后边接更多的元素。
设当前LIS的长度为`c`，遍历`a[i]`时，如果`a[i]>low[c]`，
那么将`a[i]`接到当前LIS的最后，并更新`a[++c] = a[i]`；
否则，在`low[]`中找到第一个大于等于`a[i]`的元素`low[j]`，
用`a[i]`更新`low[j]`，即`low[j] = a[i]`，这里的含义是：
长度为`j`的上升子序列的结尾元素的最小值需要被更新。

不难发现`low[]`是升序排列的，那么可以借助二分搜索快速定位`j`，
从而总的时间复杂度为$\mathcal{O} (n \log n)$。

{{< admonition type=example >}}
`a[] = {1, 4, 7, 2, 5, 9, 10, 3}`
| a[i] | low[]      | c   |
| :-:  | :-:        | :-: |
| 1    | 1          | 1   |
| 4    | 1,4        | 2   |
| 7    | 1,4,7      | 3   |
| 2    | 1,2,7      | 3   |
| 5    | 1,2,5      | 3   |
| 9    | 1,2,5,9    | 4   |
| 10   | 1,2,5,9,10 | 5   |
| 3    | 1,2,3,9,10 | 5   |
{{< /admonition >}}

{{< admonition type=note >}}
最后的`low[]`并**不是最终的LIS**，只是长度相等。
{{< /admonition >}}

{{< admonition type=example >}}
[LeetCode 300 最长上升子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/)
```cpp
class Solution {
public:
    int lengthOfLIS(const vector<int>& nums) {
        const int n = nums.size();
        if (n < 2) return n;

        vector<int> low(n);
        int ans = 0;
        low[ans] = nums[0];
        for (int i = 1; i < n; ++i) {
            if (nums[i] > low[ans]) {
                low[++ans] = nums[i];
                continue;
            }
            *lower_bound(low.begin(), low.begin()+ans+1, nums[i]) = nums[i];
        }

        return ans + 1;
    }
};
```
{{< /admonition >}}

## 解法3: 树状数组

TODO

## 参考

[最长上升子序列 (LIS) 详解+例题模板 (全)](https://blog.csdn.net/lxt_Lucia/article/details/81206439)
