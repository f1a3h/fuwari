---
title: "manacher 算法学习笔记"
published: 2023-12-09
description: '线性求所有长度的回文串：manacher 算法'
image: ''
tags: [Palindrome, String]
category: 'Algorithm'
draft: false 
lang: ''
---

由于 4 年半前（惊了，居然已经过了这么久了吗 QAQ）学过一次，所以这严格上来说应该是 Re: 从零开始的 mancher 重拾笔记（话说我怎么和初学的状态一样啊）

## Introduction

我们还是套路式的从 intro 开始吧（

如果想要单独求出某一个回文串，显然弄两个指针 $\mathcal{O}(n)$ 比较一遍就好了，但是如果要求出所有的字符串呢？最容易想到的一个优化就是，只要求出了以下标 $i$ 为中心的最长回文串，那么我们其实也就求出了以下标 $i$ 为中心的所有回文串。在考虑了这个优化下的暴力算法复杂度是 $\mathcal{O}(n^2)$ 的，但是这还是不够优秀。

如果使用字符串哈希呢？仍然采用上面的 trick，要求最长的长度我们就再套一个二分，这样可以在 $\mathcal{O}(n\log n)$ 的时间内求出来。

但是啊，这玩意有一个很简单的算法可以在 $\mathcal{O}(n)$ 的时间内求出来，那就是 manacher 算法（点题！）。道理和 KMP 是类似的，**充分利用**回文串的性质，~~需要有强大的注意力~~。

## Manacher

过程很简单，由于偶数长度的回文串的求解可以通过在原字符串中添加~~奇奇怪怪的~~字符转化为对奇数长度的回文串的求解，因此下面只考虑对奇数长度的回文串的求解。

首先，我们维护一个右端点具有最大下标的回文串的中心下标 $id$ 和右端点下标 $r$，记 $d[i]$ 表示以下标 $i$ 为中心的最长奇数长度回文串的半径大小。

枚举中心 $i$，假如当前枚举的下标 $i<r$，那么由回文串的性质，将 $i$ 以 $id$ 为中心对称过去得到对应的下标 $j=id*2-i$，则几乎可以认为 $d[i]=d[j]$，在下图中这是很显然的（图片来自 oi-wiki）：

![](https://fastly.jsdelivr.net/gh/f1a3h/imgs/20231210002328.png)

不过考虑到 $j$ 的最长回文串的左端点可能落在 $id$ 的最长回文串之外，因此我们需要砍掉多出去的部分，然后再暴力向外扩展 $d[i]$。

另一种情况是，如果当前枚举的下标 $i\ge r$，那么就直接暴力枚举。

最后都要记得用 $i$ 的结果更新 $id$ 和 $r$。

## Complexity

在以上过程中，只要 $i$ 的最长回文串是完整落在 $id$ 的最长回文串内部的，那么对于 $i$ 的求解是 $\mathcal{O}(1)$ 的，而一旦 $i$ 的最长回文串落在了 $id$ 的最长回文串外，我们只会枚举落在外面的部分，并且枚举完成后会立即更新 $id$ 和 $r$，也就是说每个下标最多只会被枚举两次，一次是成为左端点，另一次是成为右端点，所以 manacher 算法是 $\mathcal{O}(n)$ 的。

## Some Naive Thoughts

总觉得这种字符串中 KMP 和 manacher 解决的问题和对数组进行排序的问题有点像（~~强行联想~~），但是为什么字符串中这两个算法都能做到 $\mathcal{O}(n)$ 而排序的上限平均就是 $\mathcal{O}(n\log n)$ 的呢？

想起之前在知乎上刷到过为什么排序算法的上限是 $\mathcal{O}(n\log n)$ 的问题，底下关于信息论的回答令我大呼妙哉，居然还有这种角度？！自此之后，信息论成为了我心中的一本圣经，总觉得这玩意简直就是这世界的真理，它可以回答任何问题，可以求出任何优化的极限，用信息论看问题简直和开了神之眼俯视世界一样。

我们再回到这个例子，用（我以为的）信息论来考虑，那一定是 $d[]$ 和 $next[]$ 数组相比于排序问题仅仅提供了大小关系，他们提供了更丰富的信息来降低不确定性，信息量更大，因此优化的理论极限复杂度会更低。~~话说这不是扯了一堆最后结论是句废话嘛~~

但是嘛，一直没找到时间好好学学信息论，所以这仅仅是一些「naive thoughts」QwQ

有空的时候一定会认真学信息论的（确信

## Problems

板子就不特意放上来了，仅仅处理奇数长度回文串的板子和使用 trick 求出奇偶长度回文串的板子都在这两题里有体现。

### 「Luogu P4555」[国家集训队]最长双回文串

[Luogu](https://www.luogu.com.cn/problem/P4555)

递推求出下标 $i$ 分别作为回文串（不一定是 manacher 中的最长回文串）的左右端点时的最长的回文串长度，然后枚举割点取 max 即可。

当然也可以二分答案。

```cpp
#include <bits/stdc++.h>

using std::cin;		using std::cout;
using std::string;	using std::vector;

const int N = 1e5 + 5;

int n;
char s[N], t[N << 1];
vector <int> l(N << 1), r(N << 1), R(N << 1);

int modify() {
	int len = 0;
	t[++len] = '$', t[++len] = '#';
	for (int i = 0; i < strlen(s); ++i) t[++len] = s[i], t[++len] = '#';
	t[++len] = '\0';
	return len;
}

void manacher() {
	int mx = 0, mid = 0;
	for (int i = 1; i <= n; ++i) {
		R[i] = (i < mx) ? std::min(R[(mid << 1) - i], mx - i + 1) : 1;
		while (i - R[i] > 0 && i + R[i] <= n && t[i + R[i]] == t[i - R[i]]) R[i]++;
		if (R[i] + i - 1 > mx) mx = R[i] + i - 1, mid = i;

		l[i - R[i] + 1] = std::max(l[i - R[i] + 1], R[i] - 1);
		r[i + R[i] - 1] = std::max(r[i + R[i] - 1], R[i] - 1);
	}
}

int main() {
	std::ios::sync_with_stdio(false);
	cin.tie(0), cout.tie(0);

	cin >> s;
	n = modify();

	manacher();
	for (int i = 2; i <= n; i += 2) l[i] = std::max(l[i], l[i - 2] - 2);
	for (int i = n - 2; i > 0; i -= 2) r[i] = std::max(r[i], r[i + 2] - 2);

	int ans = 0;
	for (int i = 2; i <= n; i += 2)
		if (l[i] && r[i]) ans = std::max(ans, l[i] + r[i]);

	cout << ans;
	return 0;
}
```

### 「Luogu P1659」[国家集训队]拉拉队排练

[Luogu](https://www.luogu.com.cn/problem/P1659)

只需要求出奇数长度的最长回文串，按照长度 sort 一遍枚举即可，注意这里需要用到快速幂。

```cpp
#include <bits/stdc++.h>

using std::cin;		using std::cout;
using std::string;	using std::vector;
using i64 = long long;

const int mod = 19930726;
const int N = 1e6 + 5;

int n;
i64 k, ans;
string s;
vector <int> r, d;

void manacher() {
	r = vector <int> (n);
	for (int i = 0, id = 0, rr = -1; i < n; ++i) {
		r[i] = (i < rr) ? std::min(r[(id << 1) - i], rr - i + 1) : 1;
		while (i - r[i] >= 0 && i + r[i] < n && s[i - r[i]] == s[i + r[i]]) ++r[i];
		if (i + r[i] - 1 > rr) id = i, rr = i + r[i] - 1;
	}
}

i64 qpow(i64 a, int b) {
	i64 ret = 1;
	while (b) {
		if (b & 1) ret = ret * a % mod;
		a = a * a % mod;
		b >>= 1;
	}
	return ret;
}

void count() {
	int cnt = 0;
	i64 tot = 0;
	ans = 1;

	for (int len = r[0], p = 0; len > 0 && tot < k; len -= 2) {
		while (p >= 0 && r[p] == len) cnt++, p++;
		if (cnt > k - tot) cnt = k - tot;
		ans = ans * qpow(1ll * len, cnt) % mod;
		tot += cnt;
	}
}

int main() {
	std::ios::sync_with_stdio(false);
	cin.tie(0), cout.tie(0);

	cin >> n >> k;
	cin >> s;

	manacher();
	std::sort(r.begin(), r.end(), [](int a, int b) { return a > b; });
	for (int i = 0; i < n; ++i) r[i] = (r[i] << 1) - 1;
	count();

	cout << ans;
	return 0;
}
```