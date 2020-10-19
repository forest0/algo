---
title: "精确覆盖问题和DLX算法"
date: 2020-10-18T21:02:26+08:00
draft: false
series:
  - AOAPC
tags:
  - DLX
---

## 精确覆盖问题(Exact Cover Problem)

有一些由整数$1 \sim n$组成的集合$S_1,S_2,\cdots,S_r$，
要求选择若干个集合$S_i$，使得$1 \sim n$的每一个整数恰好在一个集合中出现。

[//]: # (这里愚蠢的双斜杠，用默认的markdown parser需要双斜杠；)
[//]: # (切换成pandoc（顶部添加配置 markup: pandoc），不需要双斜杠，但是代码高亮没了。。。)


{{< admonition type=example >}}
$n=7, S_1=\\{1,4,7\\}, S_2=\\{1,4\\}, S_3=\\{4,5,7\\}, S_4=\\{3,5,6\\},
S_5=\\{2,3,6,7\\}, S_6=\\{2,7\\}$，
则一个精确覆盖为$S_2,S_4,S_6$，
因为$\\{1,4\\},\\{3,5,6\\},\\{2,7\\}$无重复、无遗漏地包含了
$1 \sim 7$的所有整数。
{{< /admonition >}}

我们可以用一个 $r$ 行 $n$ 列的 $01$ 矩阵来表示一个精确覆盖问题，
其中第 $i$ 行第 $j$个元素表示 $S_i$ 是否包含元素 $j$(1表示包含，0表示不包含)。

{{< admonition type=example >}}
上边的例子可以表示为

|       |     |     |     |     |     |     |     |
| :-:   | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
|       | 1   | 2   | 3   | 4   | 5   | 6   | 7   |
| $S_1$ | 1   | 0   | 0   | 1   | 0   | 0   | 1   |
| $S_2$ | 1   | 0   | 0   | 1   | 0   | 0   | 0   |
| $S_3$ | 0   | 0   | 0   | 1   | 1   | 0   | 1   |
| $S_4$ | 0   | 0   | 1   | 0   | 1   | 1   | 0   |
| $S_5$ | 0   | 1   | 1   | 0   | 0   | 1   | 1   |
| $S_6$ | 0   | 1   | 0   | 0   | 0   | 0   | 1   |
{{< /admonition >}}

## 算法 X (Algorithm X)

同普通回溯法，可以使用递归过程求解精确覆盖问题。每次选择一个尚未覆盖的元素，
然后选择一个包含它的集合进行覆盖。对应到矩阵中，
如果我们删除所有已经选择的行和已被覆盖的列，则这个递归过程可以描述为：
每次选择一个没有被删除的列，然后枚举该列为 $1$ 的所有行，
尝试删除该行，递归搜索时恢复该行。
删除行时，除了需标记“该行已删除”外，还需把该行中值为 $1$ 的列删除，
恢复时也同样。

但删除列并不是一个简单的操作，因为还需要把能覆盖它的行也一并删除掉（
因为每列只能被一行所覆盖）。 为提高效率，
我们需要一种能高效支持上述操作的数据结构，它就是 Knuth 的舞蹈链，
使用了舞蹈链的算法 X 通常被称为 DLX 算法。

## 舞蹈链 (Dancing Links)

舞蹈链是一个4个方向的循环链表结构，每个普通结点对应矩阵中的一个1；
另外还有 $n+1$ 个虚拟结点，其中每列最上方有一个虚拟结点，
而所有虚拟结点的最前边有一个头结点 $h$[^1]。
如下图所示每个结点记录4个指针：L、R、U和D。

{{< image src="/img/dlx.jpg" height=500 width=500 >}}

- L和R：该结点的左边相邻结点和右边相邻结点。由于是循环链表，
每行的最左边结点的L指针指向这一行的最后一个结点；
最右边结点的R指针指向这一行的第一个结点。
- U和D：该结点所在列的上方相邻结点和下方相邻结点。
每一列的最上方虚拟结点的U指针指向这一列的最下方结点，
最下方结点的D指针指向最上方的结点。

另外，每列还维护一个 $S$ 值，表示这一列的普通结点个数（不含虚拟结点），
这样可以每次可以找一个 $S$ 值最小的未删除列，进一步加快求解速度。

{{< admonition type=question title="为什么每次选择 $S$ 值最小的？" open=false >}}
某列的 $S$ 值表示，该元素可以被 $S$ 个集合覆盖。
优先选择 $S$ 值小的元素(可选择的集合少)，使得靠近解答树根的结点分支较少，
可以有效地减小搜索空间。
{{< /admonition >}}

## 实现

```cpp
#ifndef DLX_H
#define DLX_H

#include <cstring>
#include <vector>

// 顺着链表遍历除s外的其它元素
#define _FOR(i,A,s) for (int i = A[s]; i != s; i = A[i])

using std::vector;

/**
 * 行列编号都从1开始
 * 列编号[1,COLUMN_AMOUNT]
 * 结点0是表头结点
 * 结点[1,n]是各列顶部的虚拟结点
 */
class DLX {
    const int COLUMN_AMOUNT, MAX_NODE_AMOUNT;
    int node_amount;

    int *S;
    int *row, *col;
    int *L, *R, *U, *D;

    vector<int> ans;

    void remove(int c) {
        // 移除顶部虚拟结点，不再考虑元素c
        R[L[c]] = R[c];
        L[R[c]] = L[c];

        // 其它能覆盖元素c的集合也不再考虑，否则会重复覆盖
        // 注意列c的纵向链接是没有改变的
        _FOR(i, D, c)
            _FOR(j, R, i) {
                U[D[j]] = U[j]; D[U[j]] = D[j];
                // 删除该集合后，能覆盖该元素的集合数量减1
                --S[col[j]];
            }
    }

    void restore(int c) {
        // 递归过程，恢复时要严格按照移除的逆序才能恢复原状
        _FOR(i, U, c)
            _FOR(j, L, i) {
                ++S[col[j]];
                U[D[j]] = j; D[U[j]] = j;
            }

        L[R[c]] = c;
        R[L[c]] = c;
    }

    bool dfs() {
        // 所有元素已覆盖完
        if (R[0] == 0) return true;

        // 寻找S值最小的元素，以加快搜索
        int c = R[0];
        _FOR(i,R,0) if (S[i] < S[c]) c = i;

        remove(c); // 当前考虑第c个元素
        _FOR(i,D,c) { // 考虑每个能覆盖元素col[j]的集合
            ans.push_back(row[i]); // 选中集合row[i]
            _FOR(j, R, i) remove(col[j]); // 该集合也覆盖元素col[j]，后续无需再考虑

            if (dfs()) return true;

            _FOR(j, L, i) restore(col[j]);
            ans.pop_back();
        }
        restore(c);

        return false;
    }

public:
    DLX(int column_amount, int max_node_amount)
        : COLUMN_AMOUNT(column_amount), MAX_NODE_AMOUNT(max_node_amount),
        S(new int[COLUMN_AMOUNT+1]),
        row(new int[MAX_NODE_AMOUNT]), col(new int[MAX_NODE_AMOUNT]),
        L(new int[MAX_NODE_AMOUNT]), R(new int[MAX_NODE_AMOUNT]),
        U(new int[MAX_NODE_AMOUNT]), D(new int[MAX_NODE_AMOUNT]),
        ans({})
    {
        // 虚拟结点
        for (int i = 0; i <= column_amount; ++i) {
            U[i] = D[i] = i;
            L[i] = i - 1;
            R[i] = i + 1;
        }
        L[0] = COLUMN_AMOUNT; R[COLUMN_AMOUNT] = 0;

        node_amount = COLUMN_AMOUNT + 1;
        memset(S, 0, sizeof(int)*(COLUMN_AMOUNT+1));
    }

    ~DLX() {
        if (S) delete [] S;
        if (row) delete [] row; if (col) delete [] col;
        if (L) delete [] L;
        if (R) delete [] R;
        if (U) delete [] U;
        if (D) delete [] D;
    }

    void addRow(int r, const vector<int> &columns) {
        int first = node_amount;
        int &idx = node_amount;
        for (auto c : columns) {
            L[idx] = idx - 1;
            R[idx] = idx + 1;
            D[idx] = c;
            U[idx] = U[c];
            D[U[c]] = idx;
            U[c] = idx;

            row[idx] = r; col[idx] = c;
            ++S[c]; ++idx;
        }
        L[first] = idx - 1; R[idx-1] = first;
    }

    bool solve(vector<int> &v) {
        if (!dfs()) return false;
        v = ans;
        return true;
    }

};

#undef _FOR

#endif /* ifndef DLX_H */

#include <cstdio>

int main(int argc, char *argv[]) {
    DLX solver(7, 42);
    vector<int> rows[] = {
        {1,4,7},
        {1,4},
        {4,5,7},
        {3,5,6},
        {2,3,6,7},
        {2,7},
    };
    for (int i = 0; i < 6; ++i) solver.addRow(i+1, rows[i]); // 行编号1开始
    vector<int> res;
    if (solver.solve(res)) {
        for (auto i : res) printf("%d ", i);
        puts("");
    } else puts("no solution");
    return 0;
}
```

## DLX求解数独

[求解一个 $16\times16$ 的数独](https://icpcarchive.ecs.baylor.edu/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=660)，
如下图所示，要求：
- 每行A~P恰好各出现一次
- 每列A~P恰好各出现一次
- 粗线分隔的每个$4\times4$子方阵（共$4\times4$个）中，A~P恰好各出现一次

{{< image src="/img/la_2659.png" width=500 >}}

### 分析

为了套用DLX算法框架，首先需要搞清楚行、列的含义，以及“1”在哪些格子中。

1. 行代表着决策（因为要“选”一些行）。这里每个决策可以用一个三元组$(r,c,v)$表示，
含义为将$(r,c)$这个格子填上字母 $v$，因此共有 $16\times16\times16=4096$ 行。
2. 列代表着任务（因为要让每列有且仅有一个1）。一共有如下4种任务需要完成：
    - `slot(a,b)` 表示位置$(a,b)$需要填有字母
    - `row(a,b)` 表示行$a$填有字母$b$
    - `col(a,b)` 表示列$a$填有字母$b$
    - `sub(a,b)` 表示第$a$个子方阵填有字母$b$

因此共有$16\times16\times4=1024$列。

3. “1”代表一个决策达成了一个任务。上边已将决策写成了三元组$(r,c,v)$的形式，
不难发现它达成了4个任务：`slot(r,c)`、`row(r,v)`、`col(c,v)`和`sub(P(r,c),v$)`，
其中`P(r,c)`表示第`r`行第`c`列的格子所在的子方阵编号。
由此可算出$01$矩阵中一共有$4096\times4=16384$个$1$。

### 实现

```cpp
// LA 2659
#include <cstdio>
#include "dlx.hpp"

const int SLOT = 0;
const int ROW = 1;
const int COL = 2;
const int SUB = 3;

char puzzle[16][20];

bool read() {
    for (int i = 0; i < 16; ++i)
        if (scanf("%s", puzzle[i]) != 1) return false;
    return true;
}

// 行/列统一编码函数，从1开始
int encode(int a, int b, int c) {
    return (a << 8) + (b << 4) + c + 1;
}

void decode(int encoding, int &a, int &b, int &c) {
    --encoding;
    c = encoding & 0xf; encoding >>= 4;
    b = encoding & 0xf; encoding >>= 4;
    a = encoding;
}

int main(int argc, char *argv[]) {
    // freopen("input.txt", "r", stdin);
    int kase = 0;
    while (read()) {
        if (kase) puts("");
        DLX solver(1024, 4096*4+10);
        for (int r = 0; r < 16; ++r)
            for (int c = 0; c < 16; ++c)
                for (int v = 0; v < 16; ++v)
                    if (puzzle[r][c] == '-' || puzzle[r][c] == v + 'A') {
                        vector<int> columns;
                        columns.push_back(encode(SLOT, r, c));
                        columns.push_back(encode(ROW, r, v));
                        columns.push_back(encode(COL, c, v));
                        columns.push_back(encode(SUB, (r/4)*4+c/4, v));
                        solver.addRow(encode(r, c, v), columns);
                    }

        vector<int> ans;
        solver.solve(ans);
        for (auto encoding : ans) {
            int r, c, v; decode(encoding, r, c, v);
            puzzle[r][c] = v + 'A';
        }

        for (int i = 0; i < 16; ++i) puts(puzzle[i]);
    }
    return 0;
}
/*
--A----C-----O-I
-J--A-B-P-CGF-H-
--D--F-I-E----P-
-G-EL-H----M-J--
----E----C--G---
-I--K-GA-B---E-J
D-GP--J-F----A--
-E---C-B--DP--O-
E--F-M--D--L-K-A
-C--------O-I-L-
H-P-C--F-A--B---
---G-OD---J----H
K---J----H-A-P-L
--B--P--E--K--A-
-H--B--K--FI-C--
--F---C--D--H-N-
*/
```

[^1]: 这只是为文字叙述方便。
整个舞蹈链的头结点在代码中并没有保存在特别的变量中，因为它总是结点0。
