
在本章中将完整构造一个Vector DS，下面先说明一下ADT与DS的区别
* 抽象数据类型ADT = 数据模型 + 定义在该模型上的一组操作
* 数据结构DS = 基于某种特定语言，实现的一整套算法

##从数组到向量
####数组
数组arr[]中的元素与[0,n)内的编号一一对应，反之，每个元素均可以通过编号直接访问。
<br>数组的每个元素在内存空间中是相邻的，也就是线性的（所以数组也被称作线性数组），因此arr[i]的物理地址 = A + i * s , s为单个元素占用的空间量
####向量
向量是数组的抽象与泛化，由一组元素按线性次序封装而成，编号被泛化为秩（rank），元素与秩一一对应
<br>经过泛化以后元素的类型不限于基本元素
####向量ADT提供的接口
表

##c++实现Vector模版类结构
####Vactor接口规范模版
```c++
typedef int Rank; //秩
#define DEFAULT_CAPACITY 3  //默认初始容量（实际应用中可以设置更大）

template<typename T> class Vector { //向量模版类
protected:
    Rank _size; //规模
    int _capacity;  //容量
    T *_elem;   //向量所在，指针，指向数据区首地址
    void copyFrom(T const *A, Rank lo, Rank hi); //复制
    void expand(); //空间不足时扩容
    void shrink(); //装填因子过小时缩容
    void bubble(Rank lo, Rank hi);  //扫描交换
    void bubbleSort();  //起泡排序算法
    Rank max(Rank lo, Rankhi);  //选取最大元素
    void selectionSort(Rank lo, Rank hi);   //选择排序
    void merge(Rank lo, Rank mi, Rank hi);  //归并算法
    void mergeSort(Rank lo, Rank hi);   //归并排序
    void heapSort(Rank lo, Rank hi);    //堆排序
    Rank partition(Rank lo, Rank hi);   //轴点构造算法
    void quickSort(Rank lo, Rank hi);   //快速排序
    void shellSort(Rank lo, Rank hi);   //希尔排序
public:
    //构造函数
    Vector(int c = DEFAULT_CAPACITY, int s = 0, T v = 0) {  //容量为c、规模为s、所有元素初始为v
        _elem = new T[_capacity = c];      //申请初始容量的内存空间，并返回指针
        for (_size = 0; _size < s; _elem[_size ++] = v);    //为vector中每个元素赋值，每赋值依次_size增大
    }
    Vector(T const *A, Rank n) {copyFrom(A, 0, n);}  //复制构造，作用于数组，整体复制
    Vector(T const *A, Rank lo, Rank hi) {copyFrom(A, lo, hi);}     //复制构造，作用于数组，区间复制
    Vector(Vector<T> const &V) {copyFrom(V._elem, 0, V._size);}  //复制构造，作用于向量，整体复制
    Vector(Vector<T> const &V, Rank lo, Rank hi) {copyFrom(V._elem, lo, hi);}  //复制构造，作用于向量，区间复制

    //析构函数
    ~Vector() {delete[] _elem;}

    //只读访问接口
    Rank size() const {return _size;} //返回规模
    bool empty() const {return  !_size;}    //判空，为0则为空返回1
    Rank find(T const &e) const {return find(e, 0, _size);}   //无序向量整体查找
    Rank find(T const &e, Rank lo, Rank hi) const;  //无序向量区间查找
    Rank search(T const &e) const {return search(e, 0, _size);}   //有序向量整体查找
    Rank search(T const &e, Rank lo, Rank hi) const;    //有序向量区间查找
    void visit(T const &e) const {std::cout << e << std::endl;}   //为了遍历而写了个visit

    //可写访问接口
    T & operator[](Rank r);  //重载下标操作符
    const T & operator[](Rank r) const; //仅限用于右值，即只读的重载版本
    Vector<T> & operator=(Vector<T> const &);//重载赋值操作符，以便于直接克隆，传入的是一个引用，其实等价于T *e
    T remove(Rank r);   //删除秩为r的元素，返回该元素
    int remove(Rank lo, Rank hi);   //删除秩在区间[lo, hi)之内的元素，返回删除元素的个数
    Rank insert(Rank r, T const &e);    //插入元素
    Rank insert(T const &e);    //默认插入到最后
    void sort(Rank lo, Rank hi);    //对[lo, hi)排序
    void sort() {sort(0, _size);}   //对整体排序
    void unsort(Rank lo, Rank hi);  //对[lo, hi)置乱
    void unsort() {unsort(0, _size);}  //对[lo, hi)排序
    int deduplicate();  //无序去重
    int uniquify(); //有序去重
    void traverse(void (*)(T&));    //遍历（使用函数指针，只读或局部修改）
    template<typename VST> void traverse(VST&); //遍历（使用函数对象，全局修改）
};
```
####构造与析构函数
```c++
//构造函数
Vector(int c = DEFAULT_CAPACITY, int s = 0, T v = 0) {  //容量为c、规模为s、所有元素初始为v
    _elem = new T[_capacity = c];      //申请初始容量的内存空间，并返回指针
    for (_size = 0; _size < s; _elem[_size ++] = v);    //为vector中每个元素赋值，每赋值依次_size增大
}
Vector(T const *A, Rank n) {copyFrom(A, 0, n);}  //复制构造，作用于数组，整体复制
Vector(T const *A, Rank lo, Rank hi) {copyFrom(A, lo, hi);}     //复制构造，作用于数组，区间复制
Vector(Vector<T> const &V) {copyFrom(V._elem, 0, V._size);}  //复制构造，作用于向量，整体复制
Vector(Vector<T> const &V, Rank lo, Rank hi) {copyFrom(V._elem, lo, hi);}  //复制构造，作用于向量，区间复制
```
####复制构造函数
```c++
template<typename T>
void Vector<T>::copyFrom(const T *A, Rank lo, Rank hi) {
    _elem = new T[_capacity = (hi - lo) << 1];   //分配空间
    _size = 0;  //将规模置0
    while(lo < hi) {
        _elem[_size ++] = A[lo ++];     //将原数组的元素依次复制到新空间
    }
}
```
##为向量添加可扩充性算法
开辟内部数组 _elem[] 并使用一段地址连续的物理空间  <br>
capacity : 总容量<br>
size: 当前实际规模n<br>
###若不使用可扩充性算法 - 静态空间管理
容量_capacity固定会出现的情况：<br>
####1.上溢(overflow):_elem[]不足以存放接下来的元素，尽管系统仍有足够的空间<br>
####2.下溢(underflow):_elem[]中的元素寥寥无几，装填因子 _size/_capacity << 50%<br>
####而一般情况下，想要一开始就找到合适的容量是很难得
###使用可扩充性算法 - 动态空间管理<蝉的哲学>
即将发生上溢时适当的扩大内部数组的容量，
<br>而具体方法则是参考蝉的方式，申请新的空间，蜕去原来的空间（即释放）
<br>扩容算法实现:
```c++
template<typename T>
void Vector<T>::expand() {
    if(_size < _capacity) return; //未满员时不必扩容
    if(_capacity < DEFAULT_CAPACITY) _capacity = DEFAULT_CAPACITY;  //不得低于最小容量
    T *oldElem = _elem;     //原空间分配给另一个指针，该指针在本方法执行结束后会被丢弃
    _elem = new T[(_capacity << 1 ) + 1];  //新空间容量加倍+1
    for (int i = 0; i < _size; i++) {
        _elem[i] = oldElem[i];
    }
    delete[] oldElem;
}
```
得益于向量的封装，内部向量的指针都是统一的，不会出现野指针的状况
###扩充策略
####1.容量递增策略
```c++
T* oldElem = _elem;
_elem = new T[_capacity += INCREMENT];  //追加固定大小的容量，时间成本记为i
```
最坏情况：在初始容量为0的空向量中，连续插入 n = 次数m * INCREMENT 个元素<br>
即使忽略申请空间操作，每次扩容过程中复制原向量的时间成本依次为: 0, i, 2i, ... ,(m-1)*i //算数级数<br>
总体耗时 = O(n^2)，每次扩容的分摊时间成本为O(n),每次扩容是装填因子都接近100%
<br><br>
####2.容量加倍策略
最坏情况：在初始容量为1的满向量中，连续插入 n = 2^m 个元素<br>
即插入2,4,8 ... 2^m个元素，则在插入第一个元素时一次扩容，复制原向量的时间成本为1，依此类推时间成本依次为:1, 2, 4, 8, ..., 2^m<br>
总体耗时为O(n),每次扩容的分摊成本为O(1),每次扩容是装填因子都在50% ~ 100%之间，**实际上是空间换时间**(这也是很多优化算法所使用的策略)

##无序向量
###无序访问 - 循秩访问
通过`V.get(r)`和`V.put(r,e)`接口,已然可以读、写向量元素，但便携性远不如数组的访问方式：A[r],因此可以重载下标操作符
```c++
template<typename T>
T &Vector<T>::operator[](Rank r) {  //默认r合法，实际应用上应该加严格判断
return _elem[r];
}

template<typename T>
const T &Vector<T>::operator[](Rank r) const {  //仅限于做右值
return _elem[r];
}
//这里顺便写了重载赋值符，我这里写的是深度复制，但是正常赋值也可以是浅度复制
template<typename T>
Vector<T> &Vector<T>::operator=(const Vector<T> &v) {
return Vector(v);
}
```
###插入
插入中至关重要的一点是区间元素后移是自后往前的
```c++
template<typename T>
Rank Vector<T>::insert(Rank r, const T &e) {    //默认r合法，实际应用上应该加严格判断
    expand();   //先扩容
    for(int i = _size; i > r; i--) {    //自后往前
        _elem[i] = _elem[i - 1];    //每个元素后移
    }
    _elem[r] = e;   //置入新元素
    _size ++;
    return r;   //返回秩，当然也可以返回插入是否成功
}

template<typename T>
Rank Vector<T>::insert(const T &e) {    //默认插到最后一位
    expand();
    _elem[_size] = e;
    _size ++;
    return _size - 1;
}
```
###删除
```c++
template<typename T>
int Vector<T>::remove(Rank lo, Rank hi) {
    if (lo >= hi) return 0; //出于效率考虑，单独处理特殊情况
    while (hi < _size) _elem[lo ++] = _elem[hi ++];
    _size = lo; //这里实际上通常_size后面还是存储着数据但是将他们视为垃圾数据
    return hi - lo; //返回删除的元素数目
}

template<typename T>
T Vector<T>::remove(Rank r) {   //删除单个元素，将其视为区间元素的特殊形式[r,r-1)
    T e = _elem[r];
    remove(r, r + 1);
    return e;
}
```
其实也可以将删除区间元素当作多次删除单个元素，但实际这样会变得很笨重，原因在于最坏情况下会多次调用区间移动，与插入相对的是，区间移动必须从前往后
###顺序查找
```c++
template<typename T>
Rank Vector<T>::find(const T &e, Rank lo, Rank hi) const {  //一般根据不同的类型要重新设定判等器
    while ((lo < hi --) && (e != _elem[hi]));   //逆向查找，返回的是查找到的秩最大者，如果是正向查找则返回秩最小者
    return hi;  //返回的值比传入的lo小即查找失败，否则返回的是命中元素的秩
}
```
###唯一化(去重)
```c++
template<typename T>
int Vector<T>::deduplicate() {
    int oldSize = _size;    //记录原本鬼目
    Rank i = 1;
    while (i < _size) { //遍历每一个元素
        find(_elem[i], 0, i) < 0 ? i++ :remove(i);  //对每个元素的前序进行查找，如有重复，则直接将i丢弃，丢弃后新的元素仍然是i，如没有则i++
    }   //看似 _size 没有改变，但其实remove中不断在改变 _size 的值
    return oldSize - _size; //返回删除的元素数
}
```
####find()和remove()合起来复杂度为O(n)，因为find查找的是前缀序列，remove移动的是后缀序列
####不变性：当前元素V[i]的前缀序列V[0,i)中，各元素彼此互异
####单调性：随着每步while的迭代，剩余的未处理的序列在不断减少
#### -> 不变性与单调性则证明了算法的正确性
###遍历
````c++
template<typename T>
void Vector<T>::traverse(void (*visit)(T &)) {  //利用函数指针，只读或者局部性修改
    for (int i = 0; i < _size; i++) visit(_elem[i]);
}

template<typename T>
template<typename VST>
void Vector<T>::traverse(VST &visit) {  //利用函数对象，可以进行全局性修改
    for (int i = 0; i < _size; i++) visit(_elem[i]);
}
````
函数对象：这是我第一次见到的名词，我查找了很多资料，发现能当函数使用的对象统称为函数对象，资料举例多是重载运算符的对象，但我认为能用那个对象的某个函数的也叫函数对象吧，笼统的可以认为传过去的是一个对象但是传该对象只是用其内置的函数而已

##有序向量
###有序性
####无序向量中的元素若可比较，就能经过一定的处理转换成有序向量，有序向量的算法比无序向量更易优化
####如果要度量一个向量的有序程度的话，就可以设置一个有序度的度量方式
####有序序列中，任意相邻的一对元素之前总会有顺序/或者逆序，我们可以用相邻逆序对的数目来度量向量的逆序程度
```c++
template<typename T>
int Vector<T>::disordered() const { //统计向量中的逆序相邻元素对
    int n = 0;  //计数器
    for (int i = 0; i < _size; i++) {   //逐一检查各个相邻对
        n += (_elem[i - 1] > _elem[i]); //若逆序则计数器自增
    }
    return n;
}
```
###唯一化(去重)
####由于在有序向量中，重复的元素必然相互紧邻构成一个区间，因此每个区间只需要保留单个元素即可
```c++
template<typename T>
int Vector<T>::uniquify() { //低效版本
    int oldSize = _size;
    int i = 1;
    while(i < _size) {  //从前向后，逐一比对各对相邻元素
        _elem[i - 1] == _elem[i] ? remove(i) : i++; //如果相同则remove(),否则i自增前进
    }
    return oldSize - _size; //这里的_size的变化由remove()隐式地完成
}
```
这个版本低效的原因是每次重复的时候都要调用一次remove()，不过比无序向量省去了大部分find()操作
####显然，如果能将雷同的元素区间成批删除，性能必将改进
```c++
template<typename T>
int Vector<T>::uniquify() {
    Rank i = 0, j = 0;
    while(++ j < _size) {   //逐一扫描直到末元素
        //i取到相同元素的第一个元素，使j自增，当找到第一个非相同元素时，将该元素移至i后
        if(_elem[i] != _elem[j]) _elem[++i] = _elem[j];
    }
    _size = ++i;
    shrink();
    return j - i; //此时j = oldSize, i = _size
}
```
这种删除十分的高明，前面的例子也有
###查找算法
####统一接口
```c++
template<typename T>
Rank Vector<T>::search(const T &e, Rank lo, Rank hi) const {
    return (rand()%2) ? binSearch(_elem, e, lo, hi) : fibSearch(_elem, e, lo, hi);  //随机使用两种查找算法
}
```
**特殊情况：**
* 目标元素不存在
* 目标元素存在多个
####原理
* 减而治之：以任意元素 x = S[mi] 为界（mi实际上是轴点），可将带查找的区间分为三部分，每比较一次就可以选择其中一部分而放弃另外两部分
* 如果是mi元素则命中，否则选择区间递归深入
####使用二分折半策略
<br>tip:建议使用小于号，便于理解算法，因为这样与有序大小的排列顺序相符
```c++
template<typename T>
static Rank binSearch(T *A, T const &e, Rank lo, Rank, hi) {
    while(lo < hi) {    //没步迭代可能要做两次比较判断，有三个分支
        Rank mi = (lo + hi) >> 1;   //取中点为轴点
        if(e < A[mi]) hi = mi;  //深入前半段
        else if(A[mi] < e) lo = mi + 1; //深入后半段
        else return mi; //命中
    }
    return -1;  //查找失败
}
```
O(logn),算法的复杂度主要来源于比较，复杂度如下例图所示：
####图
转向左、右分支的比较次数不等，但是递归深度却相同
####斐波那契查找
在上例中，若能通过递归深度的不均衡，对转向成本的不均衡进行补偿
* 向左查找的成本小于向右查找 ——> 尽可能多做向左查找，即每次给左边的区间分配的更多
* 利用斐波那契数列，设n = fib(k) - 1,则可以取mi = fib(k - 1) - 1,按照斐波那契数来划分向量
* 斐波那契查找的ASL平均是优于二分查找的
####先写一个创造斐波那契数列的类
```c++
class Fib {
private:int f,g; //f = fib(k-1);g = fib(k) int型很快就会溢出65535，可对输入的n做限制
public:
    Fib(int n) {f = 1;g = 0;while(g < n) next();}//初始化不小于n的fib项
    int get() {return g;}
    int next() {g += f; f = g - f; return g;}
    int prev() {f = g - f; g -= f; return g;}
};
```
```c++
template<typename T>
static Rank fibSearch(T *A, T const &e, Rank lo, Rank, hi) {
    Fib fib(hi - lo);
    while(lo < hi) {
        while(hi - lo < fib.get()) fib.prev();  //通过前序查找找到分割点
        Rank mi = lo + fib.get() - 1;
        if(e < A[mi]) hi = mi;  //深入前半段
        else if(A[mi] < e) lo = mi + 1; //深入后半段
        else return mi; //命中
    }
    return -1; //查找失败
}
```
斐波那契查找和二分查找的区别只是在于mi的取值

####二分查找（改进）
* 直接解决转向代价不平衡的问题，即仅做一次比较就可以找到转向的方向，因此分支要缩减成一个
* 轴点mi取作中点，则查找每深入一层，问题规模也缩减一半
<br> 1.e < x:则e存在于左区间[lo, mi)，故可深入递归
<br> 2.e >= x:则e存在于右区间[mi, hi)，故可以深入递归
* 两种特殊情况：当hi - lo = 1时，即区间只有一个元素时则元素命中
* 牺牲的是即使能够判定元素命中，而必须要缩减到最后一个元素
```c++
template<typename T>
static Rank binSearch(T *A, T const &e, Rank lo, Rank, hi) {
    while(1 < hi - lo) {    //当宽度为1时算法才会终止
        Rank mi = (lo + hi) >> 1;
        e < A[mi] ? hi = mi : lo = mi;  //[lo,mi) 或 [mi,hi)
    }//出口时区间含有一个元素A[lo]
    return e == A[lo] ? lo : -1;    //将最终元素与待查找元素比较
}
```
相对于版本A,最好的情况变坏了，最坏的情况变好了；各种情况下更加接近，整体性能更趋向于稳定
####语义约定
约定返回值返回的数值，便于有序向量自身的维护（也就是被其他算法重复利用） 例如：V.insert(1 + V.search(e), e)
* 查找成功，则返回查找区间的最后一个元素
* 查找失败，比lo元素（最小元素）还要小，则返回lo - 1
* 查找失败，比hi - 1元素大，则返回hi - 1
```c++
template<typename T>
static Rank binSearch(T *A, T const &e, Rank lo, Rank, hi) {
    while(lo < hi) {
        Rank mi = (lo + hi) >> 1;
        e < A[mi] ? hi = mi : lo = mi + 1; //[lo,mi)或(mi,hi)
        //如果命中，lo元素将会是带查找的元素雷同的元素区间中最后一个元素的后一位
    }//出口时,lo == hi
    return --lo;
}
```
这个版本巧妙的利用了lo
* 不变性：A[0,lo) <= e < A[hi,n),A[hi]总是大于e的最小者
####图
###排序算法
####起泡排序
```c++
template<typename T>
Rank Vector<T>::bubble(Rank lo, Rank hi) {
    while(++lo < hi){
        if(_elem[lo - 1] > _elem[lo]) swap(_elem[lo - 1], _elem[lo]);
    }
}

template<typename T>
void Vector<T>::bubbleSort() {
    for (int i = 0; i < _size; i++) {
        bubble(0 , i)
    }
}

template<typename T>
static void swap(T &e1, T &e2) {
    T temp = e1;
    e1 = e2;
    e2 = temp;
}
```
如果按照这种逐趟的扫描交换的话，如果序列本身就是有序的话，后序的各趟都是可以省略的
```c++
template<typename T>
void Vector<T>::bubble(Rank lo, Rank hi) {
    bool sorted = true;
    while(++lo < hi) {
        if(_elem[lo - 1] > _elem[lo]) {
            sorted = false;
            swap(_elem[lo - 1] ,_elem[lo])
        }
    }
}

template<typename T>
void Vector<T>::bubbleSort() {
    while(!bubble(lo, hi--));   //每次排序至少最后一位都是有序的
}

template<typename T>
static void swap(T &e1, T &e2) {
    T temp = e1;
    e1 = e2;
    e2 = temp;
}
```
每一趟的起泡排序都是需要n次比对的，但是有时候实际上所需要排序的都集中在一个子区间中，实际上并不需要n次比对，为了能够识别这种情况，我们对起泡排序进一步的改进
####每一趟都记录最后一次发生交换的位置，因此每次都是需要排序这个位置之前的元素即可
```c++
template<typename T>
Rank Vector<T>::bubble(Rank lo, Rank hi) {
    Rank last = lo;
    while(++lo < hi) {
        if(_elem[lo - 1] > _elem[lo]) {
            last = lo;  //记录最后一次逆序的位置，后面的都是有序的
            swap(_elem[lo - 1], _elem[lo]);
        }
    }
}

template<typename T>
void Vector<T>::bubbleSort() {
    while(lo < (hi = bubble(lo, hi)));
}

template<typename T>
static void swap(T &e1, T &e2) {
    T temp = e1;
    e1 = e2;
    e2 = temp;
}
```
####稳定性：重复元素在输入和输出中的相对次序保持不变，称为稳定性
显然，起泡排序是稳定的，因为只有因为相邻严格逆序才会交换，而两个元素相等并不会交换，但不管怎么改进，起泡排序的最坏情况的时间复杂度还是O(n^2)
####归并排序：分治策略的典型运用
图
* 序列一分为二
* 子序列递归排序（实际上就是不断的深入递归直至子序列只有一个元素，这种情况下子序列必然有序）
* 合并有序子序列（该算法的核心）
####归并主算法
```c++
template<typename T>
void Vector<T>::mergeSort(Rank lo, Rank hi) {
    if(hi - lo < 2) return; //区间为无元素或单元素
    int mi = (lo + hi) >> 1;
    mergeSort(lo, mi);  //左半段排序
    mergeSort(mi, hi);  //右半段排序
    merge(lo, mi, hi);  //两段归并
}
```
####合并有序子序列：二路归并
图
原理：每个子向量都是有序的
```c++
template<typename T>
void Vector<T>::merge(Rank lo, Rank mi, Rank hi) {  //二路归并算法
    T *A = _elem + lo; //将向量的相对初始位置移到lo
    int lb = mi - lo;   //左区间记为B，长度为lb
    T *B = new T[lb];
    for (Rank i = 0; i < lb; B[i] = A[i++]);   //为左区间子向量创建新空间，并复制到新空间
    int lc = hi - mi;
    T *C = _elem + mi;  //后子向量位置
    for (Rank i = 0, j = 0, k = 0; j < lb || k < lc) {  //当左右区间有元素循环就继续
        //该循环为A也就是按序插入数值
        if(j < lb && (lc <= k || B[j] <= C[k])) A[i++] = B[j++];
        //B[j]存在不越界 && (C已经放完 || B[j]中的元素较小))
        if(k < lc && (lb <= j || C[k] < B[j])) A[i++] = C[k++];
    }
}
```
* 算法的时间复杂度只要消耗于for循环
* 每经过一次迭代，j和k都一定会至少有一个加一，也就是最坏的情况下也只要迭代n次
* 二路归并排序的时间复杂度最坏也只有O(nlogn)，这也是排序算法的下界
* 注意：待归并子序列等长并不是必要的，也就是并不一定是一定要一分为二的方式分裂，实际上每次循环后都在破坏两个子序列等长的关系
* 这一算法也适用于另一类序列---列表