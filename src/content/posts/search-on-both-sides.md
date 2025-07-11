---
title: "双向搜索总结"
published: 2023-08-29
description: '旧文搬运'
image: ''
tags: [双向同时搜索, 'meet in the middle']
category: 'Algorithm'
draft: false 
lang: ''
---

双向搜索主要分为「双向同时搜索」和 「meet in the middle」 两种算法。

## 双向同时搜索

双向同时搜索算法主要用于解决一些给出了起点和终点，单次搜索的代价不大，同时题目中对答案的路径具有一定限制而导致每次都暴力搜索来寻找一条合法路径的代价过高的问题。

顾名思义，双向同时搜索就是从起点和终点同时出发，各自搜索到达每个位置的合法路径，然后根据限制统合二者的状态来得到答案。

### 「Luogu P5195」Knights of Ni S

> 贝茜遇到了一件很麻烦的事：她无意中闯入了森林里的一座城堡，如果她想回家，就必须穿过这片由骑士们守护着的森林。为了能安全地离开，贝茜不得不按照骑士们的要求，在森林寻找一种特殊的灌木并带一棵给他们。
>
> 当然，贝茜想早点离开这可怕的森林，于是她必须尽快完成骑士们给的任务，贝茜随身带着这片森林的地图，地图上的森林被放入了直角坐标系，并按 $x,y $ 轴上的单位长度划分成了 $ W \times H\  ( 1 \leq W,H \leq 1000 )$ 块，贝茜在地图上查出了她自己以及骑士们所在的位置，当然地图上也标注了她所需要的灌木生长的区域。某些区域是不能通过的（比如说沼泽地，悬崖，以及食人兔的聚居地）。在没有找到灌木之前，贝茜不能通过骑士们所在的那个区域，为了确保她自己不会迷路，贝茜只向正北、正东、正南、正西四个方向移动（注意，她不会走对角线）。她要走整整一天，才能从某块区域走到与它相邻的那块区域。
>
> 贝茜希望你能帮她计算一下，她最少需要多少天才可脱离这可怕的地方？输入数据保证贝茜一定能完成骑士的任务。

[Luogu](https://www.luogu.com.cn/problem/P5195)

#### 分析

已知起点（贝茜的位置）和终点（骑士们的位置），从某点搜索到其他所有点并且途中不经过某些区域的距离的时间是 $\mathcal{O}(WH)$ 的，代价较小，同时题目要求从起点到终点的路径上一定要经过灌木，如果搜索找到起点到灌木的路径，然后再枚举从每个灌木走到终点的路径的时间是 $\mathcal{O}(W^2H^2)$ 的，显然代价太大，因此这是一道双向同时搜索板子题。

我们从起点和终点分别出发，搜索到达每一个位置的最短合法路径，然后再枚举每一个灌木的位置，答案就是对他们分别到当前灌木的最短路之和取 $\min$ 。

时间复杂度：$\mathcal{O}(WH)$

#### 代码

```cpp
#include <bits/stdc++.h>
using namespace std;

int gi() {
    int x = 0, f = 1; char c = getchar();
    for ( ; !isdigit(c); c = getchar()) if (c == '-') f = -1;
    for ( ; isdigit(c); c = getchar()) x = x * 10 + (c ^ 48);
    return x * f;
}

const int N = 1005;

struct hlw {
    int x, y;
} st, ed, wd[N * N];

int n, m, tot, ans;
int g[N][N], dis[2][N][N];
int dx[4] = {0, 0, 1, -1}, dy[4] = {1, -1, 0, 0};
bool vis[2][N][N];

void bfs(int op) {
    queue <hlw> q;
    if (op) {
        q.push(ed), vis[op][ed.x][ed.y] = 1;
        dis[op][ed.x][ed.y] = 0;
    }
    else {
        q.push(st), vis[op][st.x][st.y] = 1;
        dis[op][st.x][st.y] = 0;
    }
    while (!q.empty()) {
        hlw u = q.front();
        q.pop();
        for (int i = 0; i < 4; ++i) {
            hlw v = (hlw){u.x + dx[i], u.y + dy[i]};
            if (v.x <= 0 || v.y <= 0 || v.x > n || v.y > m) continue;
            if (g[v.x][v.y] == 1 || vis[op][v.x][v.y]) continue;
            if (dis[op][v.x][v.y] > dis[op][u.x][u.y] + 1) {
                dis[op][v.x][v.y] = dis[op][u.x][u.y] + 1;
                if (!vis[op][v.x][v.y]) q.push(v), vis[op][v.x][v.y] = 1;
            }
        }
    }
}

int main() {
    m = gi(), n = gi();
    for (int i = 1; i <= n; ++i)
        for (int j = 1; j <= m; ++j) {
            g[i][j] = gi();
            if (g[i][j] == 2) st.x = i, st.y = j;
            else if (g[i][j] == 3) ed.x = i, ed.y = j;
            else if (g[i][j] == 4) wd[++tot].x = i, wd[tot].y = j;
        }
    memset(dis, 0x3f3f3f3f, sizeof(dis));
    bfs(0), bfs(1);
    ans = N * N;
    for (int i = 1; i <= tot; ++i)
        ans = min(ans, dis[0][wd[i].x][wd[i].y] + dis[1][wd[i].x][wd[i].y]);
    printf("%d", ans);
    return 0;
}
```

## meet in the middle

如果题中的数据量很小，但是又没小到可以直接暴搜的时候，可以使用 meet in the middle 算法。

meet in the middle 其实也是直接暴搜，但是它只暴力枚举 $\frac{n}{2}$ 的数据可能产生的状态，首先枚举前一半的数据，记录到达每种状态的代价，然后再搜索后一半的数据，找到一种状态时要注意统合前一半的状态看能否产生答案。

假如原来直接暴搜的时间复杂度是 $\mathcal{O}(2^n)$ ，那么使用 meet in the middle 之后的时间复杂度便是 $\mathcal{O}(2^{\frac{n}{2}})$ 的。

### 「Luogu P2962」Lights G

> 给出一张 $n$ 个点 $m$ 条边的无向图，每个点的初始状态都为 $0$。  
>
> 你可以操作任意一个点，操作结束后该点以及所有与该点相邻的点的状态都会改变，由 $0$ 变成 $1$ 或由 $1$ 变成 $0$。  
>
> 你需要求出最少的操作次数，使得在所有操作完成之后所有 $n$ 个点的状态都是 $1$。
>

[Luogu](https://www.luogu.com.cn/problem/P2962)

#### 分析

容易验证对于一个开关，我们最多按下一次，因此每个开关可能的状态只有两种：打开 or 不打开，直接暴力枚举每个开关的状态然后判断是否满足条件的代价是很明显的指数级别的，从这一点上看就应该想到是否能用 meet in the middle 解决。

再看数据范围 $n\le 35$ 很小，但是直接枚举每个点的状态的代价最大也还是 $2^{35}$ 的承受不起，但是如果把它折半之后就是 $2^{17.5}$ 的，这完全在承受范围之内，所以我们应该用 meet in the middle 暴搜。

时间复杂度：$\mathcal{O}(n\cdot 2^{\frac{n}{2}})$ 

#### 代码

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;

int n, m, ans;
ll a[40];

map <ll, int> f;//枚举每个开关是否打开之后得到所有开关对应的状态与对应打开开关的数量的映射

int main() {
    int u, v;
    cin >> n >> m;
    for (int i = 0; i < n; ++i) a[i] |= (1ll << i);
    for (int i = 1; i <= m; ++i) {
        cin >> u >> v;
        u--, v--;
        a[u] |= (1ll << v), a[v] |= (1ll << u);
    }
    for (ll i = 0; i < (1ll << (n / 2)); ++i) {//前 n/2 个开关是否打开
        ll tmp = 0;//当前所有开关的状态
        int cnt = 0;//打开的开关数，其实就是 i 中 1 的个数
        for (int j = 0; j < n / 2; ++j)
            if ((i >> j) & 1) tmp ^= a[j], cnt++;
        if (f.count(tmp)) f[tmp] = min(f[tmp], cnt);
        else f[tmp] = cnt;
    }
    ans = n;
    for (ll i = 0; i < (1ll << (n - n / 2)); ++i) {
        ll tmp = 0;
        int cnt = 0;
        for (int j = 0; j < n - n / 2; ++j)
            if ((i >> j) & 1) tmp ^= a[j + n / 2], cnt++;
        if (f.count(tmp ^ ((1ll << n) - 1))) ans = min(f[tmp ^ ((1ll << n) - 1)] + cnt, ans);
    }
    printf("%d", ans);
    return 0;
}
```

### 「Luogu P3067」Balanced Cow Subsets G

> 我们定义一个奶牛集合 $S$ 是平衡的，当且仅当满足以下两个条件：
>
> - $S$ 非空。
> - $S$ 可以被**划分**成两个集合 $A,B$，满足 $A$ 里的奶牛产奶量之和等于 $B$ 里的奶牛产奶量之和。划分的含义是，$A\cup B=S$ 且 $A\cap B=\varnothing$。
>
> 现在给定大小为 $n$ 的奶牛集合 $S$，询问它有多少个子集是平衡的。请注意，奶牛之间是互不相同的，但是它们的产奶量可能出现相同。
>

[Luogu](https://www.luogu.com.cn/problem/P3067)

#### 分析

每个元素有三种可能：不在子集中、在子集中被分到集合 $A$ 、在子集中被分到集合 $B$ 。

直接暴搜的时间复杂度同样也是指数级且不可接受的，而折半搜索之后则是可以接受的，因此我们使用 meet in the middle 算法。

当我们搜索到元素 $k$ 时，若它不在子集中，则当前总代价不变；若将它分到集合 $A$ ，则将当前总代价加上 $a[k]$ ；若将它分到集合 $B$ ，则减去 $a[k]$ 。同时我们还需要用一个二进制数 status 记录每个元素是否被选取，并用 map 建立 cost 与 status 的映射，由于同一个 cost 可能由多个（最多 1023 个） status 产生而且折半之后 status 的值最大为 1023 ，所以 map 的第二维我们使用一个 bitset 来将第 status 位标为 1 表示 cost 可以由它产生。

这道题的关键在于去重。在搜索第 $\frac{n}{2}\sim n$ 个元素时，如果前 $\frac{n}{2}$ 个元素可以达到总代价为 cost 的结果，那么他们中的每一种状态都可以和当前状态组合成一个子集。但是同样的 status 因为每个元素被分到的集合不同所以也会产生不同的 cost ，而这不同的 cost 下可能对应了前 $\frac{n}{2}$ 个元素产生的同一状态，他们统合之后的子集显然在前面已经被统计过了，这就产生了重复。

解决方案就是建一个再建 1023 个名为 vis 的 bitset ，如果当前状态 status 与前 $\frac{n}{2}$ 个元素产生的某种状态 statusLast 组合过，就将 vis[status] 的第 statusLast 位标为 1 ，在统计答案之前进行去重。

一点吐槽：

我做的时候仿照前一题直接用 3 进制枚举每个元素的状态，然后统计答案，但是我 sb 没注意看题，最后统计的是不同平衡子集下分成 $A,B$ 两个集合的总方案数，而且一些细节上没考虑到，比如后 $\frac{n}{2}$ 个元素分成的 $A,B$ 集合可以分别整合进前 $\frac{n}{2}$ 个元素分成的 $A,B$ 集合内，这样会产生新的 $A,B$ 集合，当然，产生的子集是一样的。

这下该庆幸当时因为不想调代码去看了题解发现看半天看不明白觉得题解有问题然后再看一遍问题才发现我把问题搞错了（

一些要注意的地方：

1. 将分到不同集合的情况对当前代价的影响分别记为 $+a[k]$ 和 $-a[k]$ 的方式显然要比我的分别用 sum1 和 sum2 来记录 $A,B$ 集合的代价要方便统合得多
2. 在 dfs 前将元素 sort 一遍可以快很多
3. 这里在数据量较小的情况下可以直接将记录状态的二进制数作为记录另一种状态的二进制数的某一位来存储数据，高效方便
4. 最后去重的方式很妙：`tmp &= ~vis[status]`

#### 代码

100pts 正解

```cpp
#include <bits/stdc++.h>
using namespace std;

const int N = 10;

int n, ans;
int a[25];

bitset <1 << N> vis[1 << N];
map <int, bitset <1 << N> > f;

void dfs1(int k, int cost, int status) {
    if (k == n / 2) {
        f[cost].set(status);
        return;
    }
    dfs1(k + 1, cost, status);
    dfs1(k + 1, cost + a[k], status | (1 << k));
    dfs1(k + 1, cost - a[k], status | (1 << k));
}

void dfs2(int k, int cost, int status) {
    if (k == n) {
        if (f.count(cost)) {
            bitset < 1 << N > tmp(f[cost]);
            tmp &= ~vis[status];
            ans += tmp.count();
            vis[status] |= tmp;
        }
        return;
    }
    dfs2(k + 1, cost, status);
    dfs2(k + 1, cost + a[k], status | (1 << (k - n / 2)));
    dfs2(k + 1, cost - a[k], status | (1 << (k - n / 2)));
}

int main() {
    scanf("%d", &n);
    for (int i = 0; i < n; ++i) scanf("%d", &a[i]);
    sort(a, a + n);
    dfs1(0, 0, 0);
    dfs2(n / 2, 0, 0);
    printf("%d", ans - 1);//记得 -1 去掉子集为空集的情况
    return 0;
}
```

我的搞错了题目 + 细节没处理好却还混到了 19pts 的代码

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;

int n;
ll ans;
int a[25], sta[25];

map <int, int> f;
map <ll, bool> vis;

int main() {
    ll ans1 = 0, ans2 = 0, ans3 = 0, ans4 = 0;
    scanf("%d", &n);
    for (int i = 0; i < n; ++i) scanf("%d", &a[i]);
    for (ll i = 0; i < pow(3, n / 2); ++i) {
        ll tmp = i, now = 0;
        int sum1 = 0, sum2 = 0;
        for (int k = 0; k < n / 2; ++k) {
            sta[k] = tmp % 3, tmp /= 3;
            //printf("sta[%d]=%d\n", k, sta[k]);
            if (sta[k]) now |= (1 << k);
            if (sta[k] == 1) sum1 += a[k];
            else if (sta[k] == 2) sum2 += a[k];
        }
        //printf("now:%lld sum1:%d sum2:%d\n", now, sum1, sum2);
        //if (vis[now]) puts("visited");
        //else puts("unvisited");
        if (sum1 == sum2 && sum1) ans1++;
        if (!vis[now]) f[sum1 + sum2]++, vis[now] = 1;
    }
    ans1 /= 2;
    for (ll i = 0; i < pow(3, n - n / 2); ++i) {
        ll tmp = i, now = 0;
        int sum1 = 0, sum2 = 0;
        for (int k = n / 2; k < n; ++k) {
            sta[k] = tmp % 3, tmp /= 3;
            if (sta[k]) now |= (1 << k);
            if (sta[k] == 1) sum1 += a[k];
            else if (sta[k] == 2) sum2 += a[k];
        }
        if (sum1 == sum2 && sum1) ans2++;
        if ((sum1 + sum2 != 0) && !vis[now]) ans3 += f[sum1 + sum2], vis[now] = 1;
        if (sum1 + sum2 != 0) {
            if (f.count(sum1 - sum2)) ans4 += f[sum1 - sum2];
            if (f.count(sum2 - sum1)) ans4 += f[sum2 - sum1];
        }
    }
    ans2 /= 2, ans4 /= 2;
    ans = ans1 + ans2 + ans3 + ans4;
    //printf("ans1:%lld ans2:%lld ans3:%lld ans4:%lld\n", ans1, ans2, ans3, ans4);
    printf("%lld", ans);
    return 0;
}
```