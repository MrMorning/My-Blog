---
title: '[bzoj3122][SDOI2013]随机数生成器 ——BSGS，数列'
tags: [数论, BSGS]
date: 2017-01-25 17:48:00
---

# 题目大意
给定递推序列：
F[i] = a*F[i-1] + b (mod c)
求一个最小的i使得F[i] == t

# 题解
我们首先要化简这个数列，作为一个学渣，我查阅了一些资料：
http://d.g.wanfangdata.com.cn/Periodical_cczl200924107.aspx
http://wenku.baidu.com/view/7162471b650e52ea5518982d.html
推一下，就有：
$$
a_{n+1}=ba_n+c\\
a_{n+1}+\frac{c}{b-1}=ba_n+c+\frac{c }{ b-1}=b(a_n+\frac{c}{b-1})\\
a_{n+1}+\frac c{b-1}=b^{n-1}(a_1+\frac c{b-1})
$$
$$F[i] = (F[1] + \frac{b}{a-1}) * a^{i-1} - \frac{b}{a-1}$$
令F[i] = t;
可以知道:
a^(i-1) = (t+b/(a-1)) / (x1+b/(a-1))
对于这个式子，我们直接调用BSGS算法求解即可。
特别的，某些情况需要特判。

# 代码
```cpp
#include <bits/stdc++.h>
#define ll long long
using namespace std;
ll p, a, b, X1, t, T;
ll pow(ll a, ll b, ll p) {
    ll ans = 1;
    while(b) {
        if(b & 1) ans = ans * a % p;
        b >>= 1;
        a = a * a % p;
    }
    return ans;
}
ll inv(ll a, ll p) {
    return pow(a, p-2, p);
}
map<ll, ll> mp;
ll BSGS(ll A, ll B, ll C) {
    mp.clear();
    if(A % C == 0) return -2;
    ll m = ceil(sqrt(C));
    ll ans;
    for(int i = 0; i <= m; i++) {
        if(i == 0) {
            ans = B % C;
            mp[ans] = i;
            continue;
        }
        ans = (ans * A) % C;
        mp[ans] = i;
    }
    ll t = pow(A, m, C); 
    ans = t;
    for(int i = 1; i <= m; i++) {
        if(i != 1)ans = ans * t % C;
        if(mp.count(ans)) {
            int ret = i * m % C - mp[ans] % C;
            return (ret % C + C)%C;
        }
    }
    return -2;
} 
int main() {
   // freopen("input", "r", stdin);
    scanf("%lld", &T);
    while(T--) {
        scanf("%lld %lld %lld %lld %lld", &p, &a, &b, &X1, &t);
        if(X1 == t) {
            printf("%d\n", 1);
            continue;
        }
        if(a == 0) {
            if(t == b) {
                printf("%d\n", 2);
            }
            else printf("%d\n", -1);
            continue;
        }
        if(a == 1) {
            if(b == 0) {
                printf("%d\n", -1);
                continue;
            }
            ll ans = (((t-X1)%p + p)%p * inv(b, p)) % p;
            printf("%lld\n", ans+1);
            continue;
        }
        X1 %= p, a %= p, b %= p, t%= p;
        ll tmp = (b%p * inv(a-1, p))%p;
        ll B = ((t+tmp)%p * inv((X1+tmp) % p, p)) % p;
        ll A = a;
        ll ans = BSGS(A, B, p);
        printf("%lld\n", ans+1);
    }
    return 0;
}
```