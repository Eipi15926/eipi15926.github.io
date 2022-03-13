---
title: 【DailyCoding0004】登机牌条码（CCF-CSP-2021.12）
tags: 
 - CS-DailyCoding
---

此时此刻只有一张图可以表达我复杂的心情：

![十次提交才过](\assets\images\2022-03-13-1.jpg)

> 我写的那是大模拟题吗，我写的那是bug啊……
>
> 一看就会写，一写就出bug，真是……但是话说回来如果连模拟题都写不好那还能写好啥呢……
>
> 来回顾一下到底是怎么写错的。



## 题目回顾

[登机牌条码-CCFCSP官网]([计算机软件能力认证考试系统](http://118.190.20.162/view.page?gpid=T136))

简而言之，就是以一定的方式计算给定输入串对应的数据码和校验码。

40%的测试用例不需要计算校验码，只需要计算数据码。

80%的用例满足S小于等于2，也就是校验码小于等于8个。

## 数据码编码

首先肯定先解决计算数据码的问题，然后就喜提第一个10分了——

嗯，忘记了小写转大写要两步。

后来一直20分30分，经过漫长的debug发现问题在这儿（不满一行要用900填补空白部分）：

```cpp
//正确的代码
while ((mz.size() + 1 + k) % w != 0) {
	mz.push_back(900);
}
```

原来的写法大概是

```cpp
//错误的代码
int x = (mz.size() + 1 + k) % w;
for(int i = 0;i < w - x ; i++){
    ...
}
```

这样当x%w=0的时候，会多填充一整行900……真是太棒了，哈哈哈。

## 校验码编码

好了现在40分了，开始写校验码。

刚开始一直过不了样例。写了一堆很复杂的东西实现多项式乘除法减法，后来发现不需要那么复杂。调试的时候一直输出中间结果看多项式算的对不对。中途对计算机取模运算还产生了一些疑惑，突然想起对负数取模会返回负数，还要改正数。后来发现和答案总是差个负号，好奇怪，一看题目发现题目的$r(x)$前是减号而不是一般带余除法的加号……

加了个负号之后终于样例过了。然后提交，喜提50。

如果刚才已经40的话那说明我算法写的有问题？我懵了，然后开始查错但是并没有发现什么。

结果后来我发现因为s=-1（40分）的情况不需要输出校验码。但是我补充完校验码的代码之后，对s=-1的情况也会输出校验码，而这显然是错的，因为我的校验码计算部分已经默认了s大于等于0……

加了个判断s=-1的条件提交就90了……

那还有10分咋回事？

……因为最开始我把多项式的范围设置为了1000项。

输入数据保证了**数据码字**小于929个，但是多项式的**次数**还要加上k，而k最大可以到512……

然后我把多项式次数最大值改为1999（2000），终于过了……

## 一番感慨

两行就能解决的错误，在无关的地方检查了两小时。

好在终于改对了，泪了。

```cpp
#include <bits/stdc++.h>
using namespace std;

map<char, int>num;
int change[4][4];
vector<int>sz;
vector<int>mz;
int w, s, k;
//校验码字用到的多项式类

const int maxl = 2000;
const int md = 929;

struct pol {
	int deg;//次数
	int val[maxl];//次数为i的项的系数为val[i]
	pol();
};

pol::pol() {
	deg = 0;
	for (int i = 0; i < maxl; i++) {
		val[i] = 0;
	}
}

pol multiple(pol a, pol b) {
	pol ans;
	ans.deg = a.deg + b.deg;
	for (int i = 1; i < ans.deg; i++) {
		ans.val[i] = ((a.val[i - 1] * b.val[1]) % md + (a.val[i] * b.val[0] % md) ) % md;
	}
	ans.val[ans.deg] = a.val[a.deg] * b.val[b.deg] % md;
	ans.val[0] = a.val[0] * b.val[0] % md;
	return ans;
}

string str;

int checkmode(char ch) {
	if (ch >= 'A' && ch <= 'Z')
		return 1;
	if (ch >= 'a' && ch <= 'z')
		return 2;
	return 3;
}

int pw(int s) {
	int ans = 2;
	for (int i = 0; i < s; i++) {
		ans = ans * 2;
	}
	return ans;
}

int main() {
	cin >> w >> s >> str;
	int len = str.length();

	change[1][2] = change[3][2] = 27;
	change[1][3] = change[2][3] = change[3][1] = 28;
	for (int i = 0; i < 26; i++) {
		num['a' + i] = i;
		num['A' + i] = i;
		if (i <= 9)
			num['0' + i] = i;
	}
	int mode = 1;

	for (int i = 0; i < len; i++) {
		int curmode = checkmode(str[i]);
		if (curmode != mode) {
			if (mode == 2 && curmode == 1) {
				sz.push_back(28);
				sz.push_back(28);
			} else {
				sz.push_back(change[mode][curmode]);
			}
			mode = curmode;
		}
		sz.push_back(num[str[i]]);
	}
	if (sz.size() % 2 == 1)
		sz.push_back(29);
	int sze = sz.size();
	for (int i = 0; i < sze; i += 2) {
		mz.push_back(sz[i] * 30 + sz[i + 1]);
	}//将输入字符转换完毕
	k = (s == -1 ? 0 : pw(s)); //校验级别和码字个数
	int msze = mz.size();
	while ((mz.size() + 1 + k) % w != 0) {
		mz.push_back(900);
	}

	//开始计算校验码字
	pol dx, gx, tmp;
	if (s != -1) {
		tmp.deg = 1;
		tmp.val[1] = 1;
		tmp.val[0] = -3;
		gx = tmp;
		//计算gx:
		for (int i = 2; i <= k; i++) {
			tmp.val[0] = (tmp.val[0] * 3) % md;
			gx = multiple(gx, tmp);
		}
		//给xkdx赋值
		dx.deg = mz.size() + k;
		dx.val[dx.deg] = mz.size() + 1;
		for (int i = 0; i < mz.size(); i++) {
			dx.val[mz.size() - 1 - i + k] = mz[i] % md;
		}
		for (int i = 0; i < k; i++) {
			dx.val[i] = 0;
		}
		//计算除法：
		for (int i = dx.deg; i >= k; i--) {
			int pls = dx.val[i] % md;
			for (int j = 0; j <= gx.deg; j++) {
				dx.val[i - gx.deg + j] = (dx.val[i - gx.deg + j] - (gx.val[j] * pls) % md) % md;
				if (dx.val[i - gx.deg + j] < 0) {
					dx.val[i - gx.deg + j] += md;
				}
			}
		}
	}

	//输出
	msze = mz.size();
	cout << msze + 1 << endl;
	for (int i = 0; i < msze; i++) {
		cout << mz[i] << endl;
	}
	if (s != -1) {
		for (int i = k - 1; i >= 0; i--) {
			cout << (md - dx.val[i]) % md << endl;
		}
	}
	return 0;
}
```

