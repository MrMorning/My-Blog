---
title: '[bzoj1010][HNOI2008]玩具装箱TOY'
tags: [动态规划, 斜率优化]
date: 2017-01-22 17:25:00
---

一道经典题，题目描述略。
第一写斜率优化，好紧张啊~~~~
首先就写了一个大暴力：
定义f[i]为已经分配了i个玩具时的最小费用。
方程容易写出:
f[i] = f[j] + w[j][i];
其中$w[j][i] = (sum_i - sum_j + i - (j+1) - l ) ^ 2$
明显地，这是一个O(n2)的算法，只能得到题目30的分数。
所以我们上斜率优化。
首先我们分析复杂度:状态为O(n)，转移为O(n)
状态已经可以认为无法继续优化了（除非使用其他算法），所以我们考虑优化转移。
曾经我们做过很多题目，可以使用单调队列来获得均摊O(1)的转移复杂度，这里也是类似思路。
经过一些推导（这里略去，http://medalplus.com/?p=1751#Solution写的很不错），我们可以知道决策i比决策j优当且仅当(i>j)
$$\frac{f[j]-f[k]+(G[j]+M)^2-(G[k]+M)^2}{2(G[j]-G[k])} < G(x)$$
其中，G[i] = S[i] + i, 常数M = 1 + l, S[i] = S[i-1]+C[i]，x是当前需要进行决策的点.
根据G[x]的定义，可以知道G[x]是单调递增的。
所以我们证明这样两个性质：
性质一：
如果我们把每一个状态抽象成一个点，那么原式可以看作i到j连线的斜率。
我们说：对于一个决策x，其最优决策一定在x决策前面的决策的点连成的一个下凸包上。
对于这个结论：
考虑有一个要加入的点k>j, 且k[j-1][j] > k[j][k]
因为j点已经在这个下凸包上，所以j一定比j-1更优，所以k[j-1][j] < G[x]
又因为k[j][k] < k[j-1][j]，
所以k[j][k] < G[x]
所以k比j更优
所以我们把j点舍弃，在操作中就是从尾端弹出双端队列。
直接把j-1点连上k即可。
这样我们就证明了性质一。
性质二：
因为G[x]单调递增，所以我们可以从前向后弹出最前端元素来保证k[head][head+1] > G[i]
这时k[head]一定是最优的。
根据定义不难得出这个结论。
在两个性质的推导中，我们也得到了维护下凸包的方法，具体详见代码。
下面是代码。

```
#include <bits/stdc++.h>
#define ll long long 
using namespace std;
const int maxn = 50005;
ll n, l;
ll f[maxn];
ll c[maxn], s[maxn], g[maxn];
ll pow(ll n) {return n*n;}
ll que[maxn];
int alpha;
int head, tail, size;
double calck(int a, int b) {
    return (double)(f[b]-f[a]+pow(g[b]+alpha)-pow(g[a]+alpha))/(double)(2*(g[b]-g[a]));
}
int main() {
    scanf("%d %d", &n, &l);
    alpha = l+1;
    s[0] = 0;
    for(int i = 1; i <= n; i++) {
       scanf("%d", &c[i]); 
       s[i] = s[i-1] + c[i];
       g[i] = s[i] + i; 
    }
    f[0] = 0;
    head = 1;
    tail = 1;
    size = 1;
    que[head] = 0;
    for(int i = 1; i <= n; i++) {
        while(size >= 2) {
            int a = que[head];
            int b = que[head+1];
            if(calck(a, b) < g[i]) {
                head++;
                size--;
            }
            else break;
        }   
        int n = que[head];
        f[i] = f[n]+pow(s[i]-s[n]+i-n-1-l);
        if(size>=2){int x = que[tail];
        int y = que[tail-1];
        while(calck(x, i) < calck(y, x)) {
            tail--;
            size--;
            if(size < 2) break;
            x = que[tail];
            y = que[tail-1];
        }}
        tail++;
        que[tail] = i;
        size++;
    } 
    //for(int i = 1; i <= n; i++) cout << f[i] << ' ';
    //cout << endl;
    printf("%lld\n", f[n]);
}
```