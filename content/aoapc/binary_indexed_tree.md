---
title: "动态连续和查询和二叉索引树(树状数组)"
date: 2020-11-20T15:36:50+08:00
draft: false
series:
  - AOAPC
tags:
  - BIT
---

## 静态连续和查询

在研究动态连续和查询前，先介绍静态连续和查询问题：
给定一个n元素的数组`a[1..n]`，设计一个数据结构，以支持m次查询操作`q(l,r)`，
其中`q(l,r)=a[l]+a[l+1]+...+a[r]`。

最简单的想法，每次按照定义计算，时间复杂度$\mathcal{O} (n \times m)$。

进一步地，可以现求出前缀和，定义`s[i]=a[1]+a[2]+...+a[i]`，
那么`q(l,r)=s[r]-s[l-1]`。在预处理过程用$\mathcal{O} (n)$的时间计算好`s`，
那么可以用$\mathcal{O} (1)$的时间计算每个`q(l,r)`。
不妨使用$\mathcal{O} (n) - \mathcal{O} (1)$来描述这个时间复杂度：
预处理时间$\mathcal{O} (n)$，单次查询时间$\mathcal{O} (1)$。

## 动态连续和查询

如果查询过程中允许数组的元素发生变化，即动态连续和查询问题：
给定一个n元素的数组`a[1..n]`，设计一个数据结构，以支持m次操作，
操作有两种类型：
- `update(x,d)`: `a[x] += d`
- `q(l,r)`: `a[l]+a[l+1]+...+a[r]`

若按定义计算，每次`update`$\mathcal{O} (1)$，
每次`q`$\mathcal{O} (n)$，总的时间复杂度$\mathcal{O} (n \times m)$。

是否有办法让两个操作都能迅速完成？
这里借用静态连续和查询中的思路并不能使时间复杂度降低。
使用二叉索引树(Binary Indexed Tree, BIT)可以很好地解决这个问题。

## 二叉索引树

因为二叉索引树中使用到了lowbit，这里首先介绍之。

### lowbit

对于正整数`x`，定义`lowbit(x)`为`x`的二进制表达式最右边的1所对应的值
（而不是这个比特对应的序号）。

{{< admonition type=example >}}
232的二进制为`1110 1000`，所以`lowbit(232)=1000b=8`
{{< /admonition >}}

可以使用位运算快速求解lowbit，即`lowbit(x)=x&(-x)`。
因为计算机使用整数采用补码表示，`-x`其实是`x`按位取反再加一，
即`-x == ~x + 1`。

```
 232 = 1110 1000
~232 = 0001 0111
-232 = 0001 1000
```

两者按位与后，前边部分全部变为0，lowbit部分保持不变。

### 二叉索引树结构

下图是一颗典型的BIT，由15个结点组成，编号1～15。

{{< image src="/img/BIT_1.png" title="典型BIT" caption="典型BIT" linked=false width="700" >}}

灰色结点是BIT中的结点（白色长条含义后边叙述），
每层结点的lowbit相同，lowbit越大，越靠近树根。
图中的虚线是BIT中的边[^1]。

{{< admonition type=note >}}
编号为0的结点并不是BIT的一部分，**结点从1开始**使代码实现更为方便。
{{< /admonition >}}

对于结点$i$:
- 如果它是左子结点，那么其父结点为$i+\text{lowbit}(i)$
- 如果它是右子结点，那么其父结点为$i-\text{lowbit}(i)$

现令数组$c_i = a_{i-\text{lowbit}(i)+1} + a_{i-\text{lowbit}(i)+2} + \cdots + a_i$，
即$c_i$对应数组`a[]`中一段连续和。在BIT中，
每个灰色结点$i$都属于以它自身结尾的水平长条
（对于lowbit为1的点，“长条”就是结点自身），
这个长条中的和就是$c_i$。

{{< admonition type=example >}}
结点12的长条为9~12,即$c_{12} = a_9 + a_{10} + a_{11} + a_{12}$
{{< /admonition >}}

### sum

有了数组`c[]`后，计算前缀和`s[i]`就方便了，
顺着结点$i$往左走，边走边“往上爬”，
把沿途遍历到的$c_i$加起来即可，如下图所示：

{{< image src="/img/BIT_2.png" title="BIT查询" caption="BIT查询" linked=false width="700" >}}

`query(l,r) == sum(r)-sum(l-1)`

```cpp
int sum(x) {
    int ret = 0;
    while (x) {
        ret += c[x];
        x -= lowbit(x);
    }
    return ret;
}
```

`sum`时间复杂度为$\mathcal{O} (\log n)$。

### update

如果更新了`a[i]`，那么从`c[i]`开始往右走，边走边“往上爬”，
沿途修改所有的`c[i]`即可，如下图所示：

{{< image src="/img/BIT_3.png" title="BIT更新" caption="BIT更新" linked=false width="700" >}}

```cpp
void update(int x, int d) {
    while (x <= n) {
        c[x] += d;
        x += lowbit(x);
    }
}
```

`update`时间复杂度也为$\mathcal{O} (\log n)$。

### init

```cpp
void init() {
    memset(c, 0, sizeof(c));
    for (int i = 1; i <= n; ++i) update(i, a[i]);
}
```

初始时，将`c[]`清空，然后执行n次`update`即可，
总的时间复杂度为$\mathcal{O} (n \log n)$。

[^1]: 代码中无需保存这些边，这里画出来仅是为了更好地理解BIT。

