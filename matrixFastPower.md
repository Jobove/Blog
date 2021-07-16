[TOC](文章目录)


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 前言

前段时间(校赛之前)看了快速幂, 没有深挖而后在校赛中遇到了D题, 一道几乎是矩阵快速幂的模板的题目, 经过对矩阵快速幂的简单学习终于AC了这道题,  于是  ~~迫不期待写篇博客嘚瑟一下~~  在这里记下笔记.

# 一、普通快速幂

普通的幂的求法十分简单:

 $$ a ^ {i} = a  *...*a$$
这需要花费$$i - 1$$次运算来达到目的, 然而将 a 的 i 次幂拆分开来便可以大大减少运算次数使时间复杂度减少到 $$O(\log_{2}{n})$$ , 如 $$ a ^ 6 =  (a * a ^ 2) ^ 2$$ , 具体代码实现如下:

```cpp
typedef unsigned long long ull;
ull fastNormalPower(ull base, ull power, ull mod){
	ull ans = 1;
	while(power){
		if(power % 2)
			ans = ans * base % mod;
		power /= 2;
		base = base * base % mod;
	}
	return ans;
}
```

进一步优化:

```cpp
ull fastNormalPower(ull base, ull power, ull mod){
	ull ans = 1;
	while(power){
		if(power & 1)
			ans = ans * base % mod;
		power >>= 1;
		base = base * base % mod;
	}
	return ans;
}
```

简单解释:

> 1. 如果一个数为奇数, 则其二进制表示最后一位一定为1, 则与1进行与操作得到的结果为1, 即当power为奇数时power & 1的返回值为true
> 2. 将一个数除以2在二进制表示中即将其所有位均向右移动一位

# 二、矩阵快速幂

## 1.矩阵乘法

这是线性代数的基础知识了, 偷张图来放一下原理好了:
![引用自cmmdc的博客](https://img-blog.csdnimg.cn/img_convert/d933c3ae5eaba09cf9d6f23a56eb7312.png#pic_center)
该图引用自[cmmdc的博客](https://www.cnblogs.com/cmmdc/p/6936196.html), 也可以看看这位大佬对矩阵快速幂的基础讲解.

那么代码实现也一目了然了:

```cpp
const ull maxN = 2;
void mul(ull arr1[][n], ull arr2[][n]){
	ull ans[maxN][maxN] = {};
	for(int i = 0; i < n; i++)         //确定ans矩阵的第i行的
            for(int j = 0; j < n; j++)     //第j列的元素
                for(int k = 0; k < n; k++)
                    ans[i][j] += arr1[i][k] * arr2[k][j] % MOD;	//为防止数字过大进行取模
}
```

## 2.矩阵快速幂

完成矩阵乘法之后矩阵快速幂的实现可以说是呼之欲出了 --- 将普通快速幂的操作数换成矩阵, 乘法操作换成矩阵乘法即可, 下面放出伪代码(我自己并不是这样实现的, 你可以自己写写看):

```cpp
ull ans[maxN][maxN] = {};
void matrixPow(ull arr[][n], ull power){
	for(int i = 0; i < n; i++)
            ans[i][i] = 1;
	while(power){
		if(power & 1)
			mul(ans[n][n], arr[n][n]);
		power >>= 1;
		mul(arr[n][n], arr[n][n]);
	}
}
```

## 3.花里胡哨的实现

由于看的时候受到了[看的那篇文章](https://zhuanlan.zhihu.com/p/42639682)的影响, 所以我自己写的时候用的是结构体等等的骚操作(~~我觉得代码很好看~~), 代码如下:

```cpp
#define MOD 1000000007
const int n = 2;

struct Matrix{
    ull matrix[n][n] = {};	//初始化
    friend Matrix operator *(Matrix a, Matrix b){	//运算符重载, 即对Matrix类型下的"*"运算符重新定义
        Matrix ans;
        for(int i = 0; i < n; i++)         //确定ans矩阵的第i行的
            for(int j = 0; j < n; j++)     //第j列的元素
                for(int k = 0; k < n; k++)
                    ans.matrix[i][j] += a.matrix[i][k] * b.matrix[k][j] % MOD;
        return ans;
    }
    void I(){		//构造单位矩阵
        for(int i = 0; i < n; i++)
            matrix[i][i] = 1;
    }
    friend Matrix operator ^(Matrix base, ull times){	//运算符重载, 矩阵快速幂
        Matrix ans;
        ans.I();
        while(times){
            if(times & 1)
                ans = ans * base;
            times >>= 1;
            base = base * base;
        }
        return ans;
    };
};
```

## 4.矩阵快速幂求Fibonacci数列第n项

学会了矩阵快速幂, 我们便可以通过Fibonacci数列的矩阵乘法表示来求其第n项了.
首先是Fibonacci数列的定义:
$$
f(n) =
\begin{cases}
1 	&n = 1\\
1 	&n = 2\\
f(n - 1) + f(n - 2) &n \geq 3
\end{cases}
$$
据此我们可以得到以矩阵乘法表示的递推式:
$$
 \begin{bmatrix} f(n) \\ f(n-1) \end{bmatrix} = \begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix} \begin{bmatrix} f(n-1) \\ f(n-2) \end{bmatrix} 
$$
进一步推导得:
$$
\begin{bmatrix} f(n + 1) & f(n) \\ f(n) & f(n-1) \end{bmatrix} = {\begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}} ^ {n-1} \begin{bmatrix} f(2) &f(1) \\ f(1) & f(0) \end{bmatrix}
$$
因此可以通过矩阵快速幂进行实现:

```cpp
#include <bits/stdc++.h>
#define sync ios::sync_with_stdio(false)
#define MOD 1000000007
#define debug freopen("D:\\Learn C++\\w.txt", "w", stdout)

using namespace std;

typedef unsigned long long ull;

const int n = 2;

struct Matrix{
    ull matrix[n][n] = {};
    friend Matrix operator *(Matrix a, Matrix b){
        Matrix ans;
        for(int i = 0; i < n; i++)         //确定ans矩阵的第i行的
            for(int j = 0; j < n; j++)     //第j列的元素
                for(int k = 0; k < n; k++)
                    ans.matrix[i][j] += a.matrix[i][k] * b.matrix[k][j] % MOD;
        return ans;
    }
    void I(){	//创建单位矩阵I
        for(int i = 0; i < n; i++)
            matrix[i][i] = 1;
    }
    friend Matrix operator ^(Matrix base, ull times){	//矩阵快速幂
        Matrix ans;
        ans.I();
        while(times){
            if(times & 1)
                ans = ans * base;
            times >>= 1;
            base = base * base;
        }
        return ans;
    };
};

int main(){
    sync;
    int T;
    cin >> T;
    while(T--) {
        int i;
        cin >> i;
        Matrix init = {.matrix = {{1, 1}, {1, 0}}}, ans = {.matrix = {{1, 1}, {1, 0}}};
        init = init ^ (i - 1), ans = ans * init;
        cout << ans.matrix[1][0] % MOD << '\n';
    }
    return 0;
}
```

# 三、校赛D题的题解

## 1.题面

首先令 $$f(n) =
\begin{cases}
1 	&n = 1\\
1 	&n = 2\\
f(n - 1) + f(n - 2) &n \geq 3
\end{cases}$$ ,$$F(i) = \sum_{i=1}^n f(i^2)$$

输入的第一行为一个整数```n```且满足$$ 1 \leq n \leq 10^4$$，接下来的n行中每行都有一个整数$$a_i$$满足$$1 \leq a_i \leq 10 ^ {18}$$

要求输出n行, 每行均为$$ F(a_i) \mod {10^9 +7}$$

## 2.重要公式

有了上面的基础知识, 我们终于可以着手解决这道题了!
然而并不是.
实际上你需要知道以下的式子才能动手解决, 毕竟你不能每项都进行平方取和的操作:
$$F(n) = f(n) *f(n+1)$$

## 3.代码实现

至此, 我们终于可以写下这道题的代码并提交了:

```cpp
#include <bits/stdc++.h>
#define sync ios::sync_with_stdio(false)
#define MOD 1000000007
#define debug freopen("D:\\Learn C++\\w.txt", "w", stdout)

using namespace std;

typedef unsigned long long ull;

const int n = 2;

struct Matrix{
    ull matrix[n][n] = {};
    friend Matrix operator *(Matrix a, Matrix b){
        Matrix ans;
        for(int i = 0; i < n; i++)         //确定ans矩阵的第i行的
            for(int j = 0; j < n; j++)     //第j列的元素
                for(int k = 0; k < n; k++)
                    ans.matrix[i][j] += a.matrix[i][k] * b.matrix[k][j] % MOD;
        return ans;
    }
    void I(){	//创建单位矩阵I
        for(int i = 0; i < n; i++)
            matrix[i][i] = 1;
    }
    friend Matrix operator ^(Matrix base, ull times){
        Matrix ans;
        ans.I();
        while(times){
            if(times & 1)
                ans = ans * base;
            times >>= 1;
            base = base * base;
        }
        return ans;
    };
};

int main(){
    sync;
    int T;
    cin >> T;
    while(T--) {
        ull i;
        cin >> i;
        Matrix init = {.matrix = {{1, 1}, {1, 0}}};
        init = init ^ i;
        cout << ans.matrix[1][0] * ans.matrix[1][1] % MOD << '\n';
    }
    return 0;
}
```

实际上只需要把上面的代码稍作删改即可得到这里的代码, 提交, AC!

~~PS: 实际上复现赛提交的时候因为不慎将main函数里的i定义为了int类型怒WA三发, 我果然是个演员(逃~~ 

# 四、尾声

1. 这是[洛谷的模板题](https://www.luogu.com.cn/problem/P3390#submit), 不妨在这里试试你刚学到的矩阵快速幂或是自己写的矩阵快速幂代码.
2. 感谢[这篇博客](https://blog.csdn.net/wust_zzwh/article/details/52058209)和[这篇文章](https://zhuanlan.zhihu.com/p/42639682)的讲解, 你也可以去看看他们的讲解.
3. 苣鶸的第二篇博客, 如有错漏, 请不吝赐教.
