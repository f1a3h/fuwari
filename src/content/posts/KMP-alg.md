---
title: "KMP算法学习笔记"
published: 2023-08-29
description: '旧文搬运'
image: ''
tags: [String, KMP]
category: 'Algorithm'
draft: false 
lang: ''
---

KMP 算法主要用来解决给出一个文本串和一个模式串，要求模式串在文本串中出现的位置、次数等的问题，它的核心就是前缀函数。

## 前缀函数

给定字符串 $s$ ，若 $s$ 的子串 $s[0\sim i]$ 中存在某个真前缀与真后缀相等，记这个真前缀的长度为 $k$ ，定义前缀函数 $\pi[i]$ 为子串 $s[0\sim i]$ 的最大的这样的 $k$ 。

### 朴素算法

一种暴力求前缀函数的方法就是枚举 $s[0\sim i]$ 的真前缀，判断与对应相等长度的真后缀是否相等，复杂度为 $\mathcal{O}(n^3)$ 。

### 高效算法

假设当前求解 $\pi[i]$ ，可以发现，$\pi[i]$ 至多比 $\pi[i-1]$ 大 $1$ ，因此我们只需枚举长度小于等于 $\pi[i-1]+1$ 的真前缀，并且按照长度从大到小枚举，看起来复杂度还是 $\mathcal{O}(n^3)$ 的，实际上已经降低到了 $\mathcal{O}(n^2)$ 。

进一步可以发现，如果 $s[i]\not= s[\pi[i-1]]$ ，那么下一个满足 $s[0\sim k-1]$ 与 $s[i-k\sim i-1]$ 相等的最大的 $k$ ，即仅比 $\pi[i-1]$ 小的满足 $s[0\sim i-1]$ 的真前缀与真后缀相等的值，它就等于 $\pi[\pi[i-1]-1]$ ，令 $j=\pi[i-1]$ ，如果它大于 $\pi[j-1]$ ，那么 $\pi[i-1]$ 就不满足它的最大值的性质，如果它小于 $\pi[j-1]$ ，这就不满足 $\pi[j-1]$ 的定义。因此，我们下一次判断 $s[\pi[j-1]]$ 是否与 $s[i]$ 相等，如果不相等，那就重复上述操作，令 $j=\pi[j-1]$ 再继续判断直到 $j=0$ 或者匹配成功，时间复杂度 $\mathcal{O}(n)$ 。

### 复杂度证明

先证第一个优化的。如果一次就匹配成功的话，那么计算 $\pi[i]$ 的复杂度就是 $\mathcal{O}(1)$ ，如果失配了，那么最后得到的 $\pi[i]\le \pi[i-1]$ ，我们枚举过的 $s[i]$ 的真前缀在之后都不会再被枚举到，也就是说，以 $s[\pi[i]-1]\sim s[\pi[i-1]]$ 结尾的真前缀之后都不会再被枚举了，因此 $s$ 的任意一个真前缀最多被枚举一次，每次枚举比较的复杂度是 $\mathcal{O}(n)$ ，所以总复杂度是 $\mathcal{O}(n^2)$ 。

再证加上第二个优化的。与前面的证明相同，如果一次匹配成功，则直接停止匹配，否则不断令 $j=\pi[j-1]$ ，这样的 $j$ 是递减的，而且小于 $\pi[i-1]$ ，所以每一个 $\pi[i]$ 只会被枚举一次，而每一次的判断都是 $\mathcal{O}(1)$ 的，所以总复杂度为 $\mathcal{O}(n)$ 。

### 代码

```cpp
vector <int> pre_func(string s) {
	int n = s.length();
	vector <int> pi(n);
	for (int i = 1; i < n; ++i) {
		int j = pi[i - 1];
		while (j && s[i] != s[j]) j = pi[j - 1];
		if (s[i] == s[j]) j++;
		pi[i] = j;
	}
	return pi;
}
```

### 应用

前缀函数在字符串题中有很多种应用，这里要讲的的就是其中的 KMP 算法，其他的应用还包括求解字符串的周期、统计每个前缀的出现次数、一个字符串中本质不同子串的数目、字符串压缩、构建自动机等，这些应用的前缀函数的做法并不是很好想，他们也基本可以用字符串哈希解决，但是相应的，哈希的复杂度没有前缀函数的优秀。

## KMP 算法

给定文本串 $t$ 和模式串 $s$ ，要求 $s$ 在 $t$ 中出现的所有位置，我们可以构造一个新的字符串 $cur=s+'\#'+t$ ，其中 $\#$ 可以替换为任何在 $s,t$ 中没有出现过的字符，然后我们求出新串的前缀函数，从 $t$ 出现的第一个位置开始枚举，如果当前 $\pi[i]=s.length$ ，就说明这里出现了一次 $t$ .

### 代码

```cpp
vector <int> KMP(string text, string pattern) {
	int n1 = text.length(), n2 = pattern.length();
	string cur = pattern + '#' + text;
	vector <int> pi = pre_func(cur), ret;
	for (int i = n2 + 1; i < n1 + n2 + 1; ++i)
		if (pi[i] == n2) ret.push_back(i + 1 - n2 * 2);
	return ret;
}
```

## 例题

### P3375 KMP字符串匹配

[Luogu](https://www.luogu.com.cn/problem/P3375)

```cpp
#include <bits/stdc++.h>
using namespace std;

string s1, s2;

vector <int> pre_func(string s) {
	int n = s.length();
	vector <int> pi(n);
	for (int i = 1; i < n; ++i) {
		int j = pi[i - 1];
		while (j && s[i] != s[j]) j = pi[j - 1];
		if (s[i] == s[j]) j++;
		pi[i] = j;
	}
	return pi;
}

vector <int> KMP(string text, string pattern) {
	int n1 = text.length(), n2 = pattern.length();
	string cur = pattern + '#' + text;
	vector <int> pi = pre_func(cur), ret;
	for (int i = n2 + 1; i < n1 + n2 + 1; ++i)
		if (pi[i] == n2) ret.push_back(i + 1 - n2 * 2);
	return ret;
}

int main() {
	ios :: sync_with_stdio(false);
	cin.tie(nullptr);
	cin >> s1 >> s2;
	vector <int> ans = KMP(s1, s2);
	for (int i = 0; i < ans.size(); ++i) cout << ans[i] << "\n";
	ans.clear();
	ans = pre_func(s2);
	for (int i = 0; i < ans.size(); ++i) cout << ans[i] << " ";
	return 0;
}
```

### P4391 Radio Transmission

[Luogu](https://www.luogu.com.cn/problem/P4391)

考虑将给出的字符串 $s$ 补全成为由 $s_2$ 自身连接成为的字符串 $s_1$ ，假设当前已经求出 $s_1$ 的前缀函数 $\pi[i]$ ，不妨设 $s_1$ 的长度为 $n$ ，可以发现 $s_1[0\sim\pi[n-1]-1]$ 一定也是由 $s_2$ 连接组成的字符串，而剩下的 $s_1[\pi[n-1]\sim n-1]$ 只有两种情况，一种是它就等于 $s_2$ ，另一种可能是它的长度小于 $s_2$ ，由于 $s_1$ 一定是由 $s_2$ 组成的，所以这一种情况是不可能的，从而剩下的这部分就是 $s_2$ .

接下来考虑 $s$ 开头和结尾需要补全的部分。因为 $s$ 是 $s_1$ 的子串，所以这部分一定是 $s_2$ 的子串，我们不妨将开头这一部分字符与其后它所缺失的部分结合形成一个新的 $s_2'$ ，相当于将 $s_2$ 从中间截断然后交换两头的顺序，这样一来 $s$ 就只有结尾部分不完整了。然后我们进一步可以发现，要求的只是 $s_2$ 的长度，直接求出 $\pi[n-1]$ 再将 $n-\pi[n-1]$ 作为答案是正确的，这种情况下仔细想想可以发现剩下的 $s[\pi[n-1]\sim n-1]$ 其实也只是 $s_2'$ 从中间截断 然后交换两头的顺序的变体而已。

```cpp
#include <bits/stdc++.h>
using namespace std;

vector <int> pre_func(string s) {
	int n = s.length();
	vector <int> pi(n);
	for (int i = 1; i < n; ++i) {
		int j = pi[i - 1];
		while (j && s[j] != s[i]) j = pi[j - 1];
		if (s[j] == s[i]) j++;
		pi[i] = j;
	}
	return pi;
}

int main() {
	int n;
	string s;
	cin >> n >> s;
	vector <int> nxt = pre_func(s);
	cout << n - nxt[n - 1];
	return 0;
}
```

### P3435 OKR-Periods of Words

[Luogu](https://www.luogu.com.cn/problem/P3435)

相当于求最小的前缀和后缀相等的前缀长度 $len$ ，然后对应的周期长度就是 $i+1-len$ （字符串第一位为 0 ），只需考虑 $len$ 该怎么求。

假设当前已经求出 $a[0\sim i-1]$ 的答案，其中以 $a[j]$ 为结尾的前缀的答案为 $ans[j]$ ，那么如果 $a[i]\not=a[0]$ 的话，必然有 $ans[i]>ans[i-1]$ ，容易想到一种暴力：先比较 $s[ans[i-1]]$ 和 $s[i]$ ，相等就令 $ans[i]=ans[i-1]+1$ ，否则就暴力跳 $next[i]$ ，找最小的非0 的那个作为 $ans[i]$ 。但是可以发现，和 $ans[i]>ans[i-1]$ 类似的，将每次跳的 $next$ 值记为 $j$ ，那么一定有 $j<i$ ，于是 $ans[i]\ge ans[j]$ ，所以我们只需要跳一次 $next$ ，然后将它的答案作为 $ans[i]$ 即可。

```cpp
#include <bits/stdc++.h>
using i64 = long long;

const int N = 1e6 + 5;

int n;
i64 sum;
std :: string a;

std :: vector <int> next(N), ans(N);

std :: vector <int> pre_func(std :: string s) {
	std :: vector <int> pi(n);
	for (int i = 1; i < n; ++i) {
		int j = pi[i - 1];
		while (j && s[j] != s[i]) j = pi[j - 1];
		if (s[j] == s[i]) j++;
		pi[i] = j;
	}
	return pi;
}

void Calc(std :: string s) {
	for (int i = 1; i < n; ++i) {
		int j = next[i] - 1;
		if (j < 0) continue;
		ans[i] = ans[j];
		if (!ans[i]) ans[i] = next[i];
		sum += i + 1 - ans[i];
	}
}

int main() {
	std :: ios :: sync_with_stdio(false);
	std :: cin.tie(nullptr);
	std :: cin >> n >> a;
	next = pre_func(a);
	Calc(a);
	std :: cout << sum;
	return 0;
}
```

### P4828 Censoring S

[Luogu ](https://www.luogu.com.cn/problem/P4824)

可以在 KMP 求 `next[]` 的时候用一个栈来储存，一旦发现当前位置出现了字符串 $t$ 就从栈中弹出，然后记得每次 `next[]` 初始化要令 `j = next[stk[top]]` 就好了。

```cpp
#include <bits/stdc++.h>

const int N = 1e6 + 5;

int top;
int stk[N << 1];

void Solve(std :: string s, int n) {
	int m = s.length();
	std :: vector <int> next(m);
	for (int i = 1; i < n; ++i) {
		int j = next[i - 1];
		while (j && s[i] != s[j]) j = next[j - 1];
		if (s[i] == s[j]) j++;
		next[i] = j;
	}
	for (int i = n + 1; i < m; ++i) {
		int j = next[stk[top]];
		stk[++top] = i;
		while (j && s[i] != s[j]) j = next[j - 1];
		if (s[i] == s[j]) j++;
		next[i] = j;
		if (next[i] == n) top -= n;
	}
	for (int i = 1; i <= top; ++i) std :: cout << s[stk[i]];
}

int main() {
	std :: ios :: sync_with_stdio(false);
	std :: cin.tie(nullptr);
	std :: string s, t, str;
	std :: cin >> s >> t;
	str = t + '#' + s;
	int n = t.length();
	Solve(str, n);
	return 0;
}
```