---
abbrlink: CF1895C
categories:
- - 题解
date: '2023-11-04T18:06:18.791599+08:00'
tags:
- 题解
- 学术
- OI
- Codeforces
title: CF1895C-Torn-Lucky-Ticket题解
updated: 2023-11-4T18:59:31.526+8:0
---
# Torn Lucky Ticket

## 约定

约定对于字串 $a$，$|a|$ 表示 $a$ 的长度， $a_i$ 表示 $a$ 中第 $i$ 个数码的值。

字串间的 $+$ 运算表示字串拼接，即 $a+b$ 表示将 $b$ 的头部接在 $a$ 的尾部得到的字符串，如 $\texttt{13}+\texttt{37}=\texttt{1337}.$

## 题意

给你 $n (1\le n\le 2\cdot10^5)$ 个非空字串 $s_1,s_2,\dots,s_n$，保证 $s_{i,j}\in[0,9]\cap \mathbb{N},1\le |s_i|\le 5$。

求有多少个二元组 $(i,j)$ 满足：

1. $ 1\le i,j\le n $，注意这代表 $i<j,i=j,i>j$ 都是合法的，且当 $i\neq j$ 时 $(i,j)$ 与 $(j,i)$ 算作不同方案；
2. 设字串 $t=s_i+s_j$，它满足：
   - ${|t|}$ 是偶数
   - $\sum_{k=1}^{\frac{|t|}{2}}t_k=\sum_{k=\frac{|t|}{2}+1}^{|t|}t_k=\dfrac{1}{2}\sum_{k=1}^{|t|}t_k$，即 $t$ 的前一半数码的值之和与后一半数码的值之和相等，都为整串 $t$ 的数码的值之和的一半。

## 解题思路

考虑对于字串 $s_i$ 计算它作为二元组的第一元和第二元且第一元和第二元不相等时的总贡献 $f_i$，显然 $ans=\underset{(i,i)\text{ 形式的二元组个数}}{n}+\sum_{i=1}^nf_i$。考虑如何计算 $f_i$。

我们看到限制 2，发现当将拼起来的字串分成两半时，若长度不同，则二元组中长度长的字串一定会被分割。于是我们可以先对 $s_1,s_2,\dots,s_n$ 以长度为第一关键字从小到大排序，可以证明这不影响答案。枚举 $s_i$ 时强制跟前面长度不超过 $|s_i|$ 的字串 $s_k(1\le k<i)$ 组合，至于到底是 $(i,k)$ 还是 $(k,i)$ 之后再讨论。

于是对于字串 $s_i$ 枚举从哪个位置 $j(1\le j\le |s_i|)$ 断开，其中 $s_{i,j}$ 是包含的。另外在之前要预处理出所有 $s_i$ 的前缀和 $pre_i$ 和后缀和 $suf_i$。那么

$$
\begin{aligned}
f_i=\sum_{j=1}^{|s_i|}\sum_{k=1}^{i-1}&\underbrace{[\underset{\text{左半部分长度}}{|s_k|+(j-1)}=\underset{\text{右半部分长度}}{|s_i|-j+1}\land \underset{\text{左半部分总和}}{suf_{k,1}+pre_{i,j-1}}= \underset{\text{右半部分总和}}{{suf}_{i,j}}]}_{{(k,i)\text{ 形式的二元组的贡献}}}\\&+\underbrace{[\underset{\text{右半部分长度}}{|s_k|+(|s_i|-j)}=\underset{\text{左半部分长度}}{j}\land \underset{\text{右半部分总和}}{suf_{k,1}+suf_{i,j+1}}=\underset{\text{左半部分总和}}{pre_{i,j}}]}_{(i,k)\text{ 形式的二元组的贡献}}
\end{aligned}
$$

最里层求和的意思就是满足 $(k,i)$ 的 $k$ 的个数与满足 $(i,k)$ 的 $k$ 的个数的总和。直接这么找是 $O(5n^2)=O(n^2)$ 的，显然无法通过，考虑优化掉最里面的那层循环。我们把式子中间的那两个条件整理成成对 $k$ 的限制，就是：

$$
\begin{aligned}
&[|s_k|=|s_i|-j+1-(j-1)>0\land suf_{k,1}= {suf}_{i,j}-pre_{i,j-1}>0]\\&+[|s_k|=j-(|s_i|-j)>0\land suf_{k,1}=pre_{i,j}-suf_{i,j+1}>0 \space]
\end{aligned}
$$

发现这其实就是求在 $i$ 前面的 $|s_k|$ 和 $suf_{k,1}$  为指定值的 $k$ 的个数，一看数据范围小的很呐，最坏一个 $5$ 一个 $9\times5=45$，考虑使用桶动态维护。开一个桶 $buc$，令 $buc_{i,j}$ 表示长度为 $i$，总和（字串 $s_x$ 的总和就等于 $suf_{x,1}$）为 $j$ 的字串个数。则对于每一轮 $s_i (1\le i\le n)$ 的枚举，进行如下维护：

1. 枚举断开位置 $j(1\le j\le |s_i|)$，随后累加 $f_i$：
   - 若 $|s_i|-j+1-(j-1)>0\land {suf}_{i,j}-pre_{i,j-1}>0$，更新 $f_i\leftarrow f_i+buc_{(|s_i|-j+1-(j-1)),({suf}_{i,j}-pre_{i,j-1})}$;
   - 若 $j-(|s_i|-j)>0 \land pre_{i,j}-suf_{i,j+1}>0$，更新 $f_i\leftarrow f_i+buc_{(j-(|s_i|-j)),(pre_{i,j}-suf_{i,j+1})}$
   - 上两步中判断长度与总和大于 $0$ 是必要的，否则不合题意且会出现负下标。
2. 更新 $buc_{|s_i|,suf_{i,1}}\leftarrow buc_{|s_i|,suf_{i,1}}+1$.

最后统计答案 $ans={n}+\sum_{i=1}^nf_i$ 即可。

排序复杂度 $O(n\log n)$，预处理与计算的复杂度 $O(5n)$。总时间复杂度为 $O(n\log n)$。不慢不快吧，对于这道题是绰绰有余。瓶颈主要在排序上，其实有不用排序的方法，留给读者自行思考。

**别忘记开 `long long` ！！**

## AC CODE

```cpp
#include<bits/stdc++.h>
using namespace std;
#define int long long
//别忘记开 long long！！！
constexpr int N=2e5+10;
int n,a[N];
int dig[N][10],tmp[10];
int pre[N][10],ful[N][10];
int f[N],ans,buc[10][60];
bool cmp(int x,int y){
    int lenx=log10(x)+1;
    int leny=log10(y)+1;
    //这里计算长度偷了点懒哈哈，正规来写的话可以输入时用结构体把值、串、长度都存起来，而不是每次计算log10。
    if(lenx==leny) return x<y;
    return lenx<leny;
}
signed main(){
    int i,j;
    cin>>n;
    for(i=1;i<=n;i++){
        cin>>a[i];
    }
    sort(a+1,a+1+n,cmp); //以长度为第一关键字排序
    for(i=1;i<=n;i++){ // 预处理前缀后缀和
        int it=0;
        while(a[i]){
            tmp[++it]=a[i]%10;
            a[i]/=10;
        }
        dig[i][0]=it;
        for(j=1;j<=it;j++){
            dig[i][j]=tmp[it-j+1];
            pre[i][j]=pre[i][j-1]+dig[i][j];
        }
        for(j=it;j>=1;j--){
            ful[i][j]=ful[i][j+1]+dig[i][j];
        }
    }
    for(i=1;i<=n;i++){
        int len=dig[i][0];
        for(j=1;j<=len;j++){ //累加 f[i]
            if(len-j+1-(j-1)>0&&ful[i][j]-pre[i][j-1]>0)f[i]+=buc[len-j+1-(j-1)][ful[i][j]-pre[i][j-1]];
            if(j-(len-j)>0&&pre[i][j]-ful[i][j+1]>0) f[i]+=buc[j-(len-j)][pre[i][j]-ful[i][j+1]];
        }
        buc[len][ful[i][1]]++;
    }
    ans=n;
    for(i=1;i<=n;i++){ //统计答案
        ans+=f[i];
    }
    cout<<ans;
    return 0;
}
```
