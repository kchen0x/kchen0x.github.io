title: 数据结构与算法分析（一）：基础算法分析
date: 2016/1/28 00:20:00
update: 2017/02/23 12:36:00
tag: 
- 数据结构
- 算法 
- java

---

由于 Hexo 对 Math 的显示效果不佳，请移步到下方链接阅读这篇博客：

[数据结构与算法分析（一）：基础算法分析](https://www.zybuluo.com/fyywy520/note/277242)

![Mindmap](http://7xin49.com1.z0.glb.clouddn.com/img-mind-datastructure-1.png-960.jpg)

<!--more-->

## 引论：递归例程设计的四条基本法则

1. **基准情形**。必须总要有某些基准情形，它无需递归就能解出。
2. **不断推进**。对于那些需要递归求解的情形，每一次递归调用都必须要使状况朝向一种基准情形推进。
3. **设计法则**。假设所有的递归调用都能运行。
4. **合成效益法则**（compound interest rule）。<span id="CIR"></span>在求解一个问题的同一实例时，切勿在不同的递归调用中做重复性的工作。

> 使用递归计算诸如裴波那切数列之类简单的数学函数的值的想法一般来说并不是一个好主意，其道理正是根据第四条法则。（后面将会介绍）

## 算法分析

解决的主要问题：

- 如何估计一个程序所需要的时间。
- 如何将一个程序的运行时间从天或年降低到秒甚至更少。
- 粗心使用递归的后果。
- 将一个数自乘得到其幂，以及计算两个数的最大公因数的非常有效的算法。

### 1. 数学基础

#### 时间复杂度的基本定义解释

1. $T(N)=O(f(N))$表示$T(N)$的增长率小于或等于$f(N)$的增长率，这种记法称为**大O标记法**。
2. $T(N)=\Omega(g(N))$表示$T(N)$的增长率大于或等于$g(N)$的增长率。
3. $T(N)=\Theta(h(N))$表示$T(N)$的增长率等于$h(N)$的增长率。
4. $T(N)=o(p(N))$表示$T(N)$的增长率小于$p(N)$的增长率，称为**小o标记法**，他不同于大O，因为大O包含增长率相同的情况。
5. $T(N)=O(p(N))$且$T(N)\neq\Theta(p(N))$则$T(n)=o(p(N))$。

#### 重要结论

- 法则1：
    大O标记对加法和乘法是齐次的。
- 法则2：
    如果$T(N)$是一个$k$次多项式，则$T(N)=\Theta(N^k)$
- 法则3：
    对任意常数$k$，$\mathrm{log}^kN=O(N)$。它告诉我们对数增长得非常缓慢。
- 法则4：
    我们总能通过计算极限$\mathrm{lim}_{N\to\infty}f(N)/g(N)$来确定两个函数的相对增长率，有时可能会需要用到洛必达法则[^LHospital]。

#### 常见的函数分类

|函数|名称|
|:-:|:-:|
|$c$|常数|
|$\mathrm{log}N$|对数|
|$\mathrm{log}^2N$|对数平方的|
|$N$|线性的|
|$N\mathrm{log}N$||
|$N^2$|二次的|
|$N^3$|三次的|
|$2^N$|指数的|

### 2. 要分析的问题

典型的情形是，输入的大小是主要的考虑方面。我们定义两个函数$T_{avg}(N)$和$T_{worst}(N)$，分别为算法对于输入量$N$所花费的**平均运行时间**和**最坏情况的运行时间**。

### 3. 运行时间计算

#### 一个简单的例子

这里是计算$\sum_{i=1}^{N}i^3$的一个简单的程序片段：
```java
public static int sum(int n) {
    int partialSum;
    
    partialSum = 0;
    for(int i = 1; i <= n; i ++)
        partialSum += i * i * i;
    return partialSum;
}
```

分析如下：

> 第4行和第8行各占1个时间单位。 第6行每次执行占用4个时间单位（2个乘法、1个加法、1个赋值），而执行$N$次共占用$4N$个时间单位。
> 第5行初始化时占用1个时间单位，所有测试为$N+1$个时间单位，自增运算为$N$个时间单位。
> 忽略调用方法和返回值的开销，得到的总量是$6N+4$个时间单位，因此，我们说该方法是$O(N)$的。

为了简化以上的分析工作，可以利用如下若干一般法则。

#### 一般法则

##### 法则1——for循环

一个`for`循环的运行时间至多是该`for`循环内部那些语句（包括测试）的运行时间乘以迭代次数。

##### 法则2——嵌套的for循环

从里向外分析这些循环，内部语句的总的执运行时间等于语句的运行时间乘以所有`for`循环大小的乘积。

> 例如，下列程序片段为$O(N^2)$
> ```java
> for(i = 0; i < n; i ++){
>   for( j = 0; j < n; j ++)
>       k ++;
> }
> ```

##### 法则3——顺序语句

加和即可，也即各语句的最大值就是运行时间。

##### 法则4——if/else语句

对于程序片段
```java
if( cond )
    S1
else
    S2
```
一个`if/else`语句的运行时间从不超过判断的运`S1`和`S2`中运行时间长者的总的运行时间。

分析的基本策略是从内部（或最深层的部分）向外展开的。如果有方法调用，首先要分析方法调用。如果有**递归过程**，则需要看其是否只是`for`循环的变形，如下程序只是一个简单的循环从而其运行时间是$O(N)$：

```java
public static long factorial(int n) {
    if(n <= 1)
        return 1;
    else
        return n * factorial(n-1);
}
```

当递归正常调用时，将其转换成为一个`for`循环是相当困难的，这样的情况下分析将涉及求解一个递推关系，例如下面的程序：

```java
public static long fib(int n) {
    if(n <= 1)
        return 1;
    else
        return fib(n-1) + fib(n-2);
}
```

对于$N\ge2$，有下列关于`fib(n)`的运行时间公式：
$$T(N)=T(N-1)+T(N-2)+2$$
借由归纳法可以证明$T(N)\ge fib(N)$且$fib(N)\ge(3/2)^N$。从而这个程序的运行时间将以**指数**的速度增长。

这个程序之所以运行缓慢，是因为其存在了大量的多余工作要做，违反了**[合成效益法则](#CIR "在求解一个问题的同一实例时，切勿在不同的递归调用中做重复性的工作。")**。试想，`fib(3)`调用了`fib(2)`和`fib(1)`，而`fib(4)`又调用了`fib(3)`和`fib(2)`，如此下去其中被抛弃的信息量合成起来导致了大量的运行时间。所以，「**计算任何事情不要超过一次**」。

#### 最大子序列和问题

给定（可能有负的）整数$A_1,A_2,\cdots,A_N$，求$\sum_{k=i}^{j}A_k$的最大值。（为方便起见，如果所有整数均为负数，则最大子序列的和为负数0）

> 例如：对于输入-2，11，-4，13，-5，-2，答案为20（从$A_2$到$A_4$）。

对于这个问题我们将讨论求解的四种算法，它们的性能如下：

<table>
    <tr>
        <td rowspan="2">输入大小</td>
        <td colspan="4">算法时间</td>
    </tr>
    <tr>
        <td>1<br>$O(N^3)$</td>
        <td>2<br>$O(N^2)$</td>
        <td>3<br>$O(N\mathrm{log}N)$</td>
        <td>4<br>$O(N)$</td>
    </tr>
    <tr>
        <td>N=10</td>
        <td>0.000 009</td>
        <td>0.000 004</td>
        <td>0.000 006</td>
        <td>0.000 003</td>
    </tr>
    <tr>
        <td>N=100</td>
        <td>0.002 580</td>
        <td>0.000 109</td>
        <td>0.000 045</td>
        <td>0.000 006</td>
    </tr>
    <tr>
        <td>N=1 000</td>
        <td>2.281 013</td>
        <td>0.010 203</td>
        <td>0.000 485</td>
        <td>0.000 031</td>
    </tr>
    <tr>
        <td>N=10 000</td>
        <td>NA</td>
        <td>1.232 9</td>
        <td>0.005 712</td>
        <td>0.000 317</td>
    </tr>
    <tr>
        <td>N=100 000</td>
        <td>NA</td>
        <td>135</td>
        <td>0.064 618</td>
        <td>0.003 206</td>
    </tr>
</table>

不难看出，对于小量的输入，这些算法都在眨眼之间完成，因此如果只是小量输入的情形，那么花费大量的努力去设计聪明的算法恐怕就太不值得了。另一方面，进来对于重写那些不再合理的基于小输入量假设而在五年前变西欧的程序确实存在巨大的市场。对于大量的输入，算法4显然是最好的选择。

##### 算法一

```java
/**
 * Cubic maximum contiguous subsequence sum algorithm.
 */
public static int maxSubSum1(int[] a) {
	int maxSum = 0;

	for (int i = 0; i < a.length; i++) {
		for (int j = i; j < a.length; j++) {
			int thisSum = 0;

			for (int k = i; k <= j; k++) {
				thisSum += a[k];
			}

			if (thisSum > maxSum) {
				maxSum = thisSum;
			}
		}
	}
	return maxSum;
}
```

该算法只是穷举所有的可能，易证明，该算法主要为第12行的加等运算在深度为三级的循环中执行$O(N^3)$的运算。

> 事实上：
> $$\sum_{i=0}^{N-1}\sum_{j=i}^{N-1}\sum_{k=i}^{j}1=\frac{N^3+3N^2+2N}{6}$$

##### 算法二

我们可以通过撤除一个`for`循环来避免三次的运行时间，也就是说算法一中11行和12行的计算是过分消耗了的，改进后的算法如下：

```java
/**
 * Quadratic maximum contiguous subsequence sum algorithm.
 */
public static int maxSubSum2(int[] a) {
	int maxSum = 0;

	for (int i = 0; i < a.length; i++) {
		int thisSum = 0;
		
		for (int j = i; j < a.length; j++) {
			thisSum += a[j];

			if ( thisSum > maxSum)
				maxSum = thisSum;
		}
	}
	return maxSum;
}
```

算法二显然是$O(N^2)$的，其分析比算法一还简单。

##### 算法三*

算法三采用一种「**分治（divide-and-conquer）**」策略。其想法是把问题分成两个大致相等的子问题，然后递归地对他们求解，这是「分」的部分。「治」阶段将两个子问题的解修补到一起并可能再做些少量的附加工作，最后得到整个问题的解。

在这个例子中，最大子序列可能出现在三处，整个数据的左半部、右半部或者横跨中间。前两种情况可以直接递归求解，第三种情况可以通过求解左半部分的右侧最大序列和以及右半部分的左侧最大序列和而得到。

考虑如下输入：
<table>
    <tr>
        <td colspan="4">前半部分</td>
        <td colspan="4">后半部分</td>
    </tr>
    <tr>
        <td>4</td>
        <td>-3</td>
        <td>5</td>
        <td>-2</td>
        <td>-1</td>
        <td>2</td>
        <td>6</td>
        <td>-2</td>
    </tr>
</table>

其前半部分的最大子序列和为6（$A_1$到$A_3$）而后半部分的最大子序列和为8（$A_6$到$A_7$）。

前半部分的右侧最大序列和为4（$A_1$到$A_4$），而后半部分的左侧最大序列和为7（$A_5$到$A_7$）。因此，横跨这两部分的最大和为$4+7=11$（$A_1$到$A_7$）。

故而此例中，最大子序列和就是11。

下面是该算法的一种实现手段：

```java
/**
 * Recursive maximum contiguous subsequence sum algorithm.
 * Finds maximum sum in subarray spanning a[left..right].
 * Does not attempt to maintain actual best sequence. 
 */
private static int maxSumRec(int[] a, int left, int right) {
	if (left == right) {	//Base case
		if (a[left] > 0)
			return a[left];
		else
			return 0;
	}

	int center = (left + right) / 2;
	int maxLeftSum = maxSumRec(a, left, center);
	int maxRightSum = maxSumRec(s, center, right);

	int maxLeftBorderSum = 0, leftBorderSum = 0;
	for (int i = center; i >= left; i--) {
		leftBorderSum += a[i];
		if (leftBorderSum > maxLeftBorderSum)
			maxLeftBorderSum = leftBorderSum;
	}

	int maxRightBorderSum = 0, rightBorderSum = 0;
	for (int i = center; i <= right; i++) {
		rightBorderSum += a[i];
		if (rightBorderSum > maxRightBorderSum)
			maxRightBorderSum = rightBorderSum;
	}

	return max3(maxLeftSum, maxRightSum, maxLeftBorderSum + maxRightBorderSum)
}

/**
 * Driver for divide-and-conquer maximum contiguous
 * subsequence sum algorithm.
 */
public static int maxSubSum3(int[] a) {
	return maxSumRec(a, 0, a.length);
}
```

程序解析：

递归调用的一般形式是传递输入数组以及左边界和右边界，它们界定数组要被处理的部分。单行驱动程序通过传递数组以及边界`0`和`N-1`而将递归过程启动。

程序的第7到12行描述了一种基准情况，第15和16行调用两个递归过程，第18到23和第25到30行分别给出了到达中间分界处的两个和数的最大值，`max3`（未定义）给出了算法中三个部分的最大值。

通过递归过程的调用，我们不难看出：

$$T(1)=O(1)\\
T(N)=T(N/2)+O(N)$$

经过推导，也可知$T(N)=N\mathrm{log}N+N=O(N\mathrm{log}N)$。

若非算法四给出了一个线性时间运行的解法，这个算法就会使体现递归为例的极好的范例了。

##### 算法四

我们不妨这样来理解最大序列和的问题：如果子序列由一个负数`a[i]`开头，那么这个子序列的和必然不能是最大的,因为`a[i+1]`开头的子序列必然更大，所以`i`可以推进到`i+1`。推广一下，如果`a[i]`到`a[j]`的子序列和是负数，那么包含这个子序列的序列和也不可能是最大的，因此直接把`i`推进到`j+1`是没有风险的：我们的最优解也不会错过。

```java
/**
 * Linear-time maximum contiguous subsequence sum algorithm.
 */
public static int maxSubSum4(int[] a) {
	int maxSum = 0, thisSum = 0;

	for (int j = 0; j < a.length; j++) {
		thisSum += a[j];

		if (thisSum > maxSum)
			maxSum = thisSum;
		else if (thisSum < 0)
			thisSum = 0
	}
	return maxSum;
}
```

这个算法是许多聪明算法的典型：**运行时间是显而易见的，但正确性则不那么容易看出来**。

该算法附带的一个优点是，它只对数据进行一次扫描，一旦数组被读入并处理，就不再需要被记忆。不仅如此，在任意时刻，算法都能对他已经读入的数据给出子序列问题的正确答案（其他算法不具备这个特性）。具有这种特性的叫做**联机算法（on-line algorithm）**。

> 仅需要常量空间并以线性时间运行的联机算法几乎是完美的算法！

#### 运行时间中的对数

分析算法最混乱的地方就是在对数上面，某些分治算法将以$O(N\mathrm{log}N)$运行。此外，对数最常出现的规律可以概括为下列一般法则：

> 如果一个算法用常数时间将问题的大小削减为其一部分（通常是1/2），那么该算法就是$O(\mathrm{log}N)$。另一方面，如果使用常数时间只是把问题减少一个一个常数的数量（如将问题减少1），那么这种算法就是$O(N)$的。

通常我们计算这些算法时都假设输入数据已经提前读入。

##### 折半查找（binary search）

给定一个整数$X$和整数$A_0,A_1,\cdots,A_{N-1}$，后者已经预先排序并在内存中，求下标$i$使得$A_i=X$，如果$X$不在序列中，则返回`-1`。

最明显的解法是从头到尾扫描数据，其运行时间是线性的。然而这个算法并没有利用到序列已经排序的事实。一个好的策略应该是判断$X$是不是居中的元素，如果是则返回，如果大于居中的元素，则在左半部分中继续寻找，如果小于居中的元素，则在右半部分中继续寻找：

```java
/**
 * Performs the standard binary search.
 * @return index where item is found, or -1 if not found.
 */
public static <AnyType extends Comparable<? super AnyType>>
int binarySearch(AnyType[] a, AnyType x) {
	int low = 0, high = a.length - 1;

	while (low <= high) {
		int mid = (low + high) / 2;

		if (a[mid].compareTo(x) < 0) 
			low = mid + 1;
		else if (a[mid].compareTo(x) > 0) 
			high = mid - 1;
		else
			return mid;
	}
	return NOT_FOUND;	//NOT_FOUND is defined as -1
}
```

上述程序中每次迭代所有工作花费$O(1)$的时间，因此需要分析循环次数。循环是从`high`-`low`$=N-1$开始并在`high`-`low`$\le-1$时结束，每次循环后，`high`-`low`的值至少会折半，因此不难看出，程序的运行时间是$O(\mathrm{log}N)$。

##### 欧几里得算法

第二个例子是计算最大公因数的欧几里得算法。两个整数的最大公因数（$gcd$）是同时整除二者的最大整数。于是，$gcd(50,15)=5$:

```java
public static long gcd(long m, long n) {
	while (n != 0) {
		long rem = m % n;
		m = n;
		n = rem;
	}
	return m;
}
```

算法连续计算余数直到余数是0为止，最后的非零余数就是最大公因数。因此如果`m`=1989和`n`=1590，则余数序列是399，393，6，3，0，从而$gcd(1989,1590)=3$。

可以证明，在两次迭代后，余数最多是原数值的一半，所以迭代次数至多是$2\mathrm{log}N=O(\mathrm{log}N)$。

> 事实上，欧几里得算法在平均情况下的性能需要大量篇幅的高复杂度数学分析，其迭代的平均次数约为$(12\cdot\mathrm{ln}2\cdot\mathrm{ln}N)/\pi^2+1.47$。

##### 幂运算

最后一个例子是处理一个整数的幂。计算$X^N$的明显算法是进行$N-1$次乘法自乘。有一种递归算法效果更好。$N\le1$是基准情形。否则，若$N$是偶数则$X^N=X^{N/2}\cdot X^{N/2}$，若$N$是奇数则$X^N=X^{(N-1)/2}\cdot X^{(N-1)/2}\cdot X$。

显然，所需要的乘法次数最多是$2\mathrm{log}N$：

```java
public static long pow(long x, int n) {
	if (n == 0)
		return 1;
	if (n == 1)
		return x;
	if (isEven(n))
		return pow(x * x, n / 2);
	else
		return pow(x * x, n / 2) * x;
}
```

其实上述程序有很多可以调整之处，例如第4行和第5行可以不必要，第9行也可以改写成：

```java
return pow(x, n - 1) * x;
```

但如果把第7行改为:

```java
return pow(x, n / 2) * pow(x, n / 2);
```

则会使得程序的效率下降因为此时有两个$N/2$的递归调用而不是一个，其运行时间将不再是$O(\mathrm{log}N)$。

#### 检验分析

一旦分析过后，就要想办法检验分析是否正确，常用的办法是编程来实际观察到运行时间，一般来说，当$N$扩大一倍时：

- 线性程序的运行时间乘以2
- 二次程序的运行时间乘以4
- 三次程序的运行时间乘以8
- 以对数时间运行的程序其运行时间加了一个常数
- 以$O(N\mathrm{log}N)$运行的程序其运行时间则比二倍稍多一些

> 单凭纯时间区分线性程序还是$O(N\mathrm{log}N)$是有一定难度的。

### 4. 小结

至此，我们已经通过实例了解了如何简单地对算法的复杂性进行分析。

[^LHospital]: 洛必达法则（L 'Hospital's rule）说的是，若$\mathrm{lim}_{N\to\infty}f(N)=\infty$且$\mathrm{lim}_{N\to\infty}g(N)=\infty$，则$\mathrm{lim}_{N\to\infty}f(N)/g(N)=\mathrm{lim}_{N\to\infty}f'(N)/g'(N)$。