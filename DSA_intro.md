#绪论
##计算与算法
借助某种工具（信息处理工具），遵守一定规则（特定计算模型），以明确而机械（确定性、可行性与有穷性）的形式进行
###算法的有穷性
序列Hailstone(n)
```c++
int hailstone(int n) { //计算Hailstone(n)序列的长度
    int length = 1; //记录步数，即长度
    while (1 < n) {
        (n % 2) ? n = 3 * n + 1 : n /= 2;
        length ++;
    }
    return length;  //返回步数
}
```
对于任意的n，总有|Hailstone(n)| < ∞ ?
<br>**目前HailStone序列还未被证明是否有穷,所以它未必是一个算法，即程序不等于算法**
###好算法
1.正确的，对合法的输入<br>
2.健壮的，不合法的输入<br>
3.可读的<br>
4.**效率，速度尽可能快，存储空间尽可能少(最重要的)**

##计算模型
正确性
<br>成本(性能、效率):运行时间 + 所需存储空间
<br>**问题实例的规模，一般是决定计算成本的主要因素**
###TM图灵机与RAM随机寄存器模型
都是对一般的计算工具的简化与抽象，将算法运行的时间转换为需要执行的基本次数
###大O记号
当问题规模逐渐增大时，我们通常关注的是成本的增长趋势
<br>常系数可忽略：O(f(n)) = O(c x f(n))
<br>低次项可忽略：O(n^a + n^b) = O(n^a) when a>b>0
<br>大O通常构成上界
<br>O(1)：常数 -- 一般不br含循环，分支转向，递归
<br>O(logn)：对数 -- 复杂度无限接近于常数
<br>O(nlogn)
<br>O(n^c)：包含线性O(n)
<br>O(2^n)：指数 -- 通常认为不可忍受
<br>从O(n^c)到O(2^n)是有效算法到无效算法的分水岭
##算法分析
###级数
####算数级数：与末项平方同阶
T(n) = 1 + 2 + ... + n = n(n+1)/2 = O(n^2)
####幂方级数：比幂次高出一阶
T(n) = 1^3 + 2^3 + ... + n^3 = O(n^4)
####几何级数(a > 1):与末项同阶
T(n) = 1 + 2 + 4 + ... + 2^n = O(2^n)
####收敛级数:通常为O(1)
####两个特殊级数
h(n) = 1 + 1/2 + ... + 1/n = o(logn)<br>
log1 + log2 + ... + logn = o(nlogn)
###循环
```c++
for(int i = 0; i < n; i++)
    for(int j = 0; j < n; j++)
```
n + n + n + ... + n = n * n = O(n^2)
```c++
for(int i = 0; i < n; i++)
    for(int j = 0; j < i; j++)
```
0 + 1 + ... + (n-1) = n(n-1)/2 = O(n^2) //算数级数
```c++
for(int i = 0; i < n; i++)
    for(int j = 0; j < i; j += 2013)
```
虽然内部步长增大了，但是n -> ∞的时候渐进时间复杂度并没有改变，依然是O(n^2)
```c++
for(int i = 0; i < n; i <<= 1)
    for(int j = 0; j < i; j++)
```
外步长以集合级数变化，内步长线性变化，我以为外步以集合级数增长基本可以看做常数了，因为常数次变换就可以接近计算机存储的最大值了，该方法为O(n)
###取非极端元素
问题：在某一互异集合（集合中元素大于等于三个）中取出不是最小值和最大值的一个元素
<br>算法:
<br>1.在集合中任意取三个元素
<br>2.排除三个元素中的最大者和最小者
<br>3.输出剩余元素
<br>在该算法中无论输入规模n多大，算法执行时间都不变
###冒(起)泡排序
原理：一个序列中，任意相邻元素都是有顺序/逆序
<br>方法：扫描交换，依次比较每一对相邻序列，如有必要则交换，若整趟扫描都没有进行交换，则排序完成;否则再做一趟扫描交换
<br>**不变性**:经过k轮扫描交换后，最大的k个元素必然就位
<br>**单调性**:经过k轮扫描交换后，问题规模缩减至n-k
<br>**正确性**:经过至多n趟扫描后，算法必然终止，且能给出正确答案
```c++
void swap(int &a, int &b);
void bubblesort(int* arr, int n) {
    bool flag = true;
    while (flag) {
        flag = false;
        for (int i = 1; i < n; i++) {
            if (arr[i - 1] > arr[i]){
                swap(arr[i - 1],arr[i]);
                flag = true;
            }
        }
    }
}
void swap(int &a, int &b) {
    int temp = a;
    a = b;
    b = temp;
}
```
###封底估算
对某些具体数值凑好计算的整数（上下取都行）进行估算，看看就得了

##迭代与递归
###减而治之-线性递归
**问题1：计算任意n个整数之和**
```c++
int SumArr(int arr[], int n) {
    int sum = 0;  //O(1)
    for (int i = 0; i < n; i++) { //O(n)
        sum += arr[i];  //O(1)
    }
    return sum;  //O(1)
}
```
迭代的复杂度计算一般可以转换为级数与级数求和,所以整个算法的时间复杂度为O(n)
<br>上面的算法中，每处理一步都在减少原来问题的规模，这就是线性递归的思想：**减而治之**
```c++
int SumArr(int arr[], int n) {
    return (n < 1) ? 0 : SumArr(arr,n-1) + arr[n-1]; //最后一项
}
```
递归跟踪分析：检查每个递归实例，累计每个实例的时间和即为算法执行时间，本例子中一共有n个递归实例，单个递归实例自身只需要O(1),则和为n*O(1)=O(n)
<br>递推方程分析：T(n) = T(n-1) + O(1); T(0) = O(1);
<br>**问题2:任给数组A[lo,hi)，将其前后颠倒**
<br>统一接口:`void reverse(int* A,int lo,int hi); //lo为左端点，hi为右端点`
```c++
void reverse(int* arr,int lo,int hi){
    if (lo < hi) {
        swap(arr[lo],arr[hi]);
        reverse(arr,lo + 1, hi - 1);
    }
}
```
递归不一定比迭代的方法好
```c++
void reverse(int* arr,int lo,int hi){
    while(lo < hi) swap(arr[lo++],arr[hi--];
}
```
###分而治之
将大规模的问题划分为若干（通常是两个）子问题，得到子问题的解再将其归并起来
<br>**问题3:数组求和-二分递归**
```c++
int SumArr(int* arr,int lo,int hi) {
    if(lo == hi) return arr[lo];
    int mi = (lo + hi - 1) >> 1;  //二分向左取整
    return SumArr(arr, lo, mi) + SumArr(arr, mi + 1, hi);
}
```
看似递归实例会按几何（指数方式）增长，但实际上底层的递归实例只有n个，因此时间复杂度为O(n)
<br>
<br>**问题4:从数组区间A[lo,hi)中找出最大的两个整数A[x1]和A[x2]，元素比较的次数要尽可能少**
<br>普通迭代方法:
```c++
void max(int* arr, int lo, int hi, int &x1, int &x2) {
    x1 = lo;
    for (int i = lo + 1; i < hi; i++) {
        if(arr[x1] < arr[i]) x1 = i;
    }
    x2 = (x1 == lo) ? lo + 1 : lo;
    for (int i = 0; i < x1; ++i) {
        if(arr[x2] < arr[i]) x2 = i;
    }
    for (int i = x1 + 1; i < hi; ++i) {
        if(arr[x2] < arr[i]) x2 = i;
    }
}
```
无论如何，比较次数总是2n - 3
<br>改进迭代方法:定义并利用A[x1] > A[x2]的特性
```c++
void max2(int* arr, int lo, int hi, int &x1, int &x2) {
    if(arr[x1 = lo] < arr[x1 = lo + 1]) swap(x1, x2);
    for (int i = lo + 2; i < hi; i++) {
        if (arr[x2] < arr[i]){
            if (arr[x1] < arr[x2 = i])
                swap(x1,x2);
        }
    }
}
```
但最坏情况而言比较次数也是2n - 3
<br>分治递归方法:
```c++
void max3(int* arr, int lo, int hi, int &x1, int &x2) {
    if (lo + 2 == hi) { //如果只有两个元素
        if (arr[lo] > arr[lo + 1]) {
            x1 = lo;
            x2 = lo + 1;
        } else {
            x1 = lo + 1;
            x2 = lo;
        }
        return;
    }
    if (lo + 3 == hi) { //如果只有三个元素，找出最小值剔除就行,比较三次
        if(arr[x1 = lo] < arr[x1 = lo + 1]) swap(x1, x2);
            if (arr[x2] < arr[lo + 2]){
                if (arr[x1] < arr[x2 = lo + 2])
                    swap(x1,x2);
            }
        }
        return;
    }
    int mi = (lo + hi) >> 1;
    int x1L, x2L;
    max3(arr, lo, mi, x1L, x2L);
    int x1R, x2R;
    max3(arr, lo, mi, x1R, x2R);
    if (arr[x1L] > arr[x1R]) {
        x1 = x1L;
        x2 = (arr[x1R] > arr[x2L]) ? x1R : x2L;
    } else {
        x1 = x1R;
        x2 = (arr[x1L] > arr[x2R]) ? x1L : x2R;
    }
}
```

###动态规划
通常是递归找到算法的本质和初步的解后再转换为迭代的方法
**<br>典型1：斐波那契数列**
<br>递归:
```c++
int fib(int n) {
    return (n < 2) ? n : fib(n-1) + fib(n-2);
}
```
递归版fib()低效的根源在于各递归实例均被大量重复的调用，即计算一个新的斐波那契数都要重新计算所有斐波那契数
<br>解决办法一：将每个结果制表（比如数组）备查
<br>解决办法二：动态规划-自顶向下递归，自底向上迭代
```c++
int fib(int n) {
    int temp = n - 1;
    int f = 0;
    int g = 1;
    while(0 < temp--) {
        g = g + f;
        f = g - f;
    }
    return (n < 2) ? n : g;
}
```
**典型2:最长公共子序列长度**
<br>子序列：右序列中若干字符，按原相对次序构成
<br>公共子序列：两个或多个都有的子序列
<br>最长公共子序列可能有多个可能有多个，也可能有歧义，但本题中只需要确定最长公共子序列的长度
<br>**问题：对于序列A[0, n]和B[0, m],求处LCS(A,B)的长度**
<br>递归版本:
```c++
//递归解决的方法:
//1)若n = -1或m = -1，则取空序列""     //递归基
//2)若匹配到A[n] = 'x' = B[m]，则去作LCS(A[0, n),B[0,m)) + 'x'  //减而治之
//3)若A[n] != B[m], 则在LCS(A[0, n),B[0,m])和LCS(A[0, n],B[0,m))中取更长者  //分而治之
int LCS(char* A, char* B, int n, int m, int count) {

}
```
递归版的LCS效果上并不理想，根源在于比对不相同时会分解为两个子问题，它们进一步导出的子问题可能会雷同也就是重复
<br>动态规划:
<br>
