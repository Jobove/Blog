[TOC]


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 前言 

看到朋友圈里的大佬写了文章记录自己的做题心得, 心血来潮也想跟风一下,于是有了这篇文章~ 新手请轻喷, 如果有更好的思路, 或者文章有所错漏, 请不吝赐教.



# 一、读题


无论是做什么题, 读懂题都是十分重要的, 边界条件? 输入的数据范围? 输入输出格式等等等等都需要注意, 有的时候你的程序不能顺利AC就是因为没有注意到题目给出的提示或要求.
当然这道题还是相当容易读懂的 ~~(不比最后一题, 看了好久也没将问题转化成动态规划问题)~~ 可以注意到的是输入的整数均小于2^31^, 因此我们可以放心的用int来存放数值.

# 二、思路
## 1.递推


要说到素数(质数), 恐怕都知道其定义: 除了1和其自身外没有其他因数的数. 由此便引出了最简单的方法, 使用循环令n对(1, n)这个开区间里的每一个数求模, 如果n不能被所有数整除, 便说明其为质数,代码实现如下:
```cpp
bool isPrime(int n){
	for(int i = 2, i < n; i++){
		if(n % i == 0)			//被范围内的数整除
			return false;
	}
	return true;
}
```
然而在此基础上我们可以对这个代码进行简单的优化--将循环的结束条件改为为 $i < \sqrt{n} + 1$, 即
```cpp
bool isPrime(int n){
	for(int i = 2, i < sqrt(n) + 1; i++){			//sqrt()函数包含于cmath头文件中,使用前需声明, 当然其返回值类型为double, 这里的使用并不十分规范
		if(n % i == 0)			//被范围内的数整除
			return false;
	}
	return true;
}
```
这个优化不难理解, 假设$n$不是质数, 那么其至少有两个其他因数, 而sqrt(n)的平方为n本身, 则n的这两个(或更多)因数至少有一个是不大于$\sqrt{n}$的, 因此我们只需要令循环判断$i < \sqrt{n} + 1$即可.


## 2.筛法求素数

### i.普通筛法
然而如此代码仍然不够快, 我们可以通过筛法求素数来进一步压缩代码运行时间, 其原理也是十分简单的 -- 在循环的判断中, 对所有判断为素数的数的倍数均标记为合数即可:



```cpp
const int N = 10000;
bool judge[N + 1] = {false, false, true};        //假设需要判断10000是否为素数, 值为1时说明为素数
bool isPrime(){
    for(int i = 2; i <= N; i++){
        if(judge[i]){           //如果i为素数, 标记它的倍数为合数
            for(int j = i; j <= N; j += i)
                judge[j] = false;
        }
    }
    return judge[N];
}
```
筛法求素数可有效的减少程序判断素数花费的时间且简单易懂, 但这并不是尽头.






### ii.快速线性筛法

实际上我们还能继续对上面的算法继续优化, 因为我们不难发现有许多合数是会被重复标记的——如i = 2时会标记4, 6, ...;而i = 3的时候会标记6, 9, ...这样6及其他2和3的公倍数都会被重复标记, 这样便创造了继续优化的空间, 快速线性筛法由此而生.
它的初步实现很简单: 对每个2到n之间的数, 筛去只有这个数可以筛到而后面的数筛不到的数便可以做到不重复, 具体如何实现?
我们先贴上代码来进行分析吧:
```cpp
const int N = 200000;
int prime[N] = {0}, num_prime = 0;      //数组用以记录素数, 变量用以记录素数个数
bool isPrime[N + 1] = {false, false};    //若i为素数, 则 isPrime[i]值为true
int main(){
    for(int i = 2; i <= N; i++){
        if(isPrime[i])          //如果i是素数, 将其添加到素数数列 prime[]中
            prime[num_prime++]=i;
            //关键点1
        for(int j = 0; j < num_prime && i * prime[j] <= N; j++){
            isPrime[i * prime[j]] = false;       //将合数i与素数 prime[j]的乘积标记为合数
            //关键点2
            if(!(i % prime[j]))     //如果合数i可以被 prime[j]整除, 中断循环
                break;
        }
    }
    return 0;
}
```
这是经过对[一位大佬的代码](https://www.cnblogs.com/JRicardo/p/6819481.html)简单修改后得到的, 大家可以去他的文章里看看他对于这个代码的说明, 我对这个算法的两个关键问题稍微说明一下:

>  1. 为什么这个算法不会重复筛除?
>  2. 每一个合数都能确保筛去吗?


 对于第一个问题, 我们将所有合数分为两种: 由两个素数相乘得到的合数 和  由一个合数 与 一个不小于合数的最小质因数的 素数 相乘得到的合数.

 举两个例子: 

 对于15, 我们认为其是 3 * 5, 这样 15 将会在 i = 5 时被标记, 对更小的 i, 15不会被重复标记, 因为当 i 是素数的时候只能标记其与更小素数的乘积, 对更大的i, 其不会被重复标记, 这是自然的, 因为更大的 i 乘以 5 或 3 自然不会等于 15;
 对于75, 我们认为其是 3 * 25, 这样 75 将会在 i = 25 时被标记, 而不会被其他的 i 标记, 因为75 的 最小质因数是 3 , 所对应的 i 有且仅有 25, 故不可能被重复标记.


 对于第二个问题, 由于 (2, n] 这个区间内的所有合数 x 都至少有一对由 x 的最小质因数p 和 一个素数或合数q 组成的因数, 因此当 i = q 时 x 一定会被标记, 得证.

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 最终代码
介绍了这么多算法, 回到这道题, 实际上你会发现, 由于题目给出的范围过大, 用数组来存放难免产生内存问题(我在使用快速线性筛法来重写这道题提交时在测试点2便出现了问题, 如有错误, 敬请指出), 因此我最终决定使用优化后的递推来完成这道题, 代码如下:

```cpp
#include <cmath>
#include <iostream>

using namespace std;

bool isPrime(int n)
{
    if (n == 1)
    {
        return 0;
    }
    for (int i = 2; i <= sqrt(n); i++)
    {
        if (n % i == 0)
        {
            return 0;
        }
    }
    return 1;
}

int main()
{
    int a, b;
    cin >> a;
    for (int i = 0; i < a; i++)
    {
        cin >> b;
        if (isPrime(b))
            cout << "Yes\n";
        else
            cout << "No\n";
    }
    return 0;
}
```
结果如下:
![AC](https://img-blog.csdnimg.cn/20201107163947103.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI2MDg3NDgx,size_16,color_FFFFFF,t_70#pic_center)
虽然是一个差强人意的结果, 但题目到此也就完成了.

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

#  ~~PS~~ 

 1. ~~花了好久才看懂快速线性筛法求素数, 最后居然没有AC着实令我伤心.~~ 
 2. ~~没想到写这一篇文章花的时间比我做的最久的一道题花的时间还长, 不过正所谓"教学相长", 能学到一个新的算法还是很开心的(逃~~ 
 3. 是个萌新, 请轻喷(悲
 4. 证明处参考了 [这篇文章](https://blog.csdn.net/nuanxin_520/article/details/41207145?utm_source=blogxgwz0) ,有效地帮助了我理解, 万分感谢!