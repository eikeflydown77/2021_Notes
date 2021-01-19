#第六章：数组和指针

##（一） 数组
###1.1 数组定义
1.类型说明符  `数组名[常量表达式][常量表达式]..`
<br>2.数组必须先定义，后使用
<br>3.数组在内存中是顺次存放，它们的地址是连续的。元素间物理地址上相邻，对应的逻辑次序也相邻

###1.2 一维数组举例
例子一：将前二十个斐波那契数列的数存入数组，并依次输出，每五个换行
```c++
#include <iostream>
using namespace std;
int main() {
    int f[20] = {1,1};
    for (int i = 2; i < 20; i++) {
        f[i] = f[i-1] + f[i-2];
    }
    for (int i = 0; i < 20; ++i) {
        if (i % 5 == 0) cout << endl;  //每五个换行
        cout.width(12); //设置输出宽度为12
        cout << f[i];
    }
    return 0;
}
```
例子二：循环从键盘读入若干组选择题答案，计算并输出每组答案的正确率。
<br>每组连续输入5个正确答案，每个答案可以是'a'...'d'
```c++
#include <iostream>
using namespace std;
int main() {
    const char key[] = {'a','c','b','a','d'};
    const int NUM_QUES = 5;
    char c;
    int ques = 0, numCorrect = 0;
    cout << "Enter the" << NUM_QUES << "question tests: " << endl;
    while(cin.get(c)) { //获取字符依次比对数组内的字符
        if(c != '\n') {
            if(c == key[ques]) {
                numCorrect ++;
                cout << " ";
            } else {
                cout << "*";
            }
            ques ++;
        } else {
            cout << "Score " << (float)(numCorrect) / NUM_QUES * 100 << "%"; //运算括号要注意
            ques = 0;
            numCorrect = 0;
            cout << endl;
        }
    }
    return 0;
}
```
###1.3 数组元素作为函数的参数

数组名作参数，形、实参数都应是数组名，类型要一样，**传送的是数组首地址**。对形参数组的改变会直接影响到实参数组。
```c++
#include <iostream>
using namespace std;

void rowSum(int a[][4], int nRow) {
    for(int i = 0; i < nRow; i++) {
        for(int j = 1; j < 4; j++) {
            a[i][0] += a[i][j];
        }
    }
}
int main() {
    int a[3][4] = {{1,2,3,4},{2,3,4,5},{3,4,5,6}};
    cout << "修改前的数组" << endl;
    for(int i = 0; i < 3; i++) {
        for(int j = 0; j < 4; j++) {
             cout << a[i][j];
        }
        cout << endl;
    }
    rowSum(a,3);
    for (int i = 0; i < 3; ++i) {
        cout << "Sum of row" << i << " is " << a[i][0] << endl;
    }
    cout << "修改后的数组" << endl;
    for(int i = 0; i < 3; i++) {
        for(int j = 0; j < 4; j++) {
            cout << a[i][j];
        }
        cout << endl;
    }
    return 0;
}
```

###1.4 对象数组
就是把对象放入数组，实际上只是创建对象放入相邻的内存位置，数组返回首地址
```c++
#include <iostream>
using namespace std;

class Point{
    static int count;
public:
    Point();
    Point(int x,int y);
    ~Point();
    void move(int newX,int newY);
    int getX() const {
        return x;
    }
    int getY() const {
        return y;
    }
    static void showCount();
private:
    int x,y;
};
Point::Point() : x(0),y(0){count ++;}
Point::Point(int x, int y) : x(x),y(y){count ++;}
Point::~Point() {
    cout << "对象" << x << "," << y << "被删除" << endl;
    count --;
}
void Point::move(int newX, int newY) {
    x = newX;
    y = newY;
}
void Point::showCount() {
    cout << "存在对象个数为： " << count << endl;
}
int Point::count = 0;
int main() {
    cout << "Enter Main" << endl;
    Point a[2]; //自动调用无参构造方法创建对象
    for (int i = 0; i < 2; ++i) {
        a[i].move(i+10 ,i+10);
    }
    Point::showCount();
    return 0;
}
```

###1.5 强for循环：基于范围的for循环
频繁用于处理容器类
```c++
#include <iostream>
using namespace std;

int main() {
    int array[] = {1,2,3};
    for(int &e : array) {
        e += 2;
        cout << e << endl;
    }
    return 0;
}
```
##（二）指针

###2.1 指针的概念
指针：内存地址，用于间接访问内存单元
<br>指针变量：用于存放地址的变量,是数据的首位置
<br>*:指针运算符
<br>&:地址运算符

###2.2 指针变量的初始化
`存储类型 数据类型 *指针名 = 初始地址;`
<br>通过变量获得的初始地址:&变量名
```c++
static int i;
static int *ptr = &i;
*ptr = 3; //指针运算：寻找地址的过程，将变量赋到找到的地址中
```
注意事项：
<br>1.用变量作为初值时，该变量必须在指针初始化之前已声明过，且变量类型应与指针类型一致
<br>2.可以用一个已有合法值的指针去初始化另一个指针变量
<br>3.不要用内部非静态变量去赋值静态指针
<br>4.数据类型与指针类型必须相符
<br>5.整数0可以赋值给指针，表示空指针，c++11后使用nullptr
<br>6.允许定义或声明指向void类型的指针，该指针可以被赋予任何类型对象的地址，但是不能访问指向的内存空间与对象
```c++
#include <iostream>
using namespace std;

int main() {
    int i;
    int* ptr = &i;
    i = 10;
    cout << "i=" << i << endl;
    cout << "*ptr=" << *ptr << endl;

    //void指针
    void *pv;
    int j = 5;
    pv = &j; //但是该指针只能存放地址，不能访问地址空间
    int *pint = static_cast<int*>(pv); //void指针转向int指针
    cout << "*pint=" << *pint << endl;

    //指向常量的指针,指针本身不是常量，但指针指向的对象是常量，起码不能通过指针改变对象的值
    int a;
    const int *p1 = &a;
    int b;
    p1 = &b;//指针本身的值是可以改变的
    //*p1 = 1; //编译就会出错

    //指针本身为常量
    int const *p2 = &a;
    //p2 = &b; //指针本身的值不能改变

    return 0;
}
```

###2.3 指针类型的算术运算
指针的运算是以一个完整数据的地址为单位进行运算
<br>指针在同类型之间是可以关系运算的(与0关系运算则判断的是是否为空指针)
```c++
short a[4];
short *pa = a;
// *pa -> a[0]
// *(pa+1) -> a[1]
```

###2.4 指针数组
存放指针变量的数组
```c++
#include <iostream>
using namespace std;

int main() {
    int line1[] = {1,0,0};
    int line2[] = {0,1,0};
    int line3[] = {0,0,1};

    int *pLine[3] = {line1, line2, line3};
    //输出矩阵
    //只输出每个行数组的首元素，可以看出数组变量实际上存放的是数组行首元素的地址
    for (int i = 0; i < 3; i++) {
        cout << *pLine[i] << " ";
    }
    cout << endl;
    cout << "----------" << endl;
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; ++j) {
            cout << pLine[i][j] << " "; //基本等价于普通二维数组
        }
        cout << endl;
    }
    return 0;
}
```
与二维数组的区别在于每个行数组地址位置并不一定相邻

###2.5 以指针作为函数的参数
1.需要数据双向传递时（等价于引用传递，或者说指针是引用传递的一种）
<br>2.需要传递一组数据，只传首地址运行效率比较高
<br>3.如果希望传地址，但是不希望修改对象，那就传指向常量的指针即`const 对象类型 *变量名`
```c++
#include <iostream>
using namespace std;

void spitFloat(float x, int *intpart, float  *fracpart) {
    *intpart = static_cast<int>(x);
    *fracpart = x - *intpart;
}
int main() {
    cout << "Enter 3 float point number: " << endl;
    for (int i = 0; i < 3; ++i) {
        float x ,f;
        int n;
        cin >> x;
        spitFloat(x,&n,&f); //传入的是n与f的地址
        cout << "Integer Part = " << n << " Fraction Part = " << f << endl;
    }

    return 0;
}
```

###2.6 指针类型的函数
返回值是一个指针的函数
<br>注意：不要将非静态局部地址用作函数的返回值

###2.7 指向函数的指针（区别于2.6）
容纳函数代码存储区的起始地址的指针<br>
`存储类型 数据类型 (*函数指针名)();`
<br>1.实现函数回调

<br>举例：
<br>1)编写一个计算函数compute,对两个整数进行各种计算。有一个形参为指向具体算法函数的指针，根据不同的实参函数，用不同的算法进行计算
<br>2)编写三个函数：分别求两个整数的最大值、最小值、和。分别测试compute函数
```c++
#include <iostream>
using namespace std;

int compute(int a, int b, int(*func)(int, int)){
    return func(a,b);
}
int max(int a, int b){
    return ((a > b) ? a : b);
}
int min(int a, int b){
    return ((a < b) ? a : b);
}
int sum(int a, int b){
    return  a+b;
}

int main() {
    int a,b,res;
    cout << "请输入整数a: "; cin >> a;
    cout << "请输入整数b: "; cin >> b;

    res = compute(a, b, &max);
    cout << res << endl;
    res = compute(a, b, &min);
    cout << res << endl;
    res = compute(a, b, &sum);
    cout << res << endl;
    return 0;
}
```

###2.8对象指针
通过指针访问对象成员
`ptr -> getX()` 等价于 `(*ptr).getX()`
<br>this指针：系统会默认将当前对象的地址赋给this指针，默认会调用，但也可以手动调用
```c++
class Fred; //前向引用声明
class Barney {
    //Fred f; //直接调用会报错，因为类还没有完善
    Fred *f;
};
class Fred {
    Barney b;
};
```

##（三）动态分配内存
### 3.1 动态分配与释放内存
申请:`new 类型名T(初始化参数列表)`
<br>成功:返回T类型的指针<br>
失败:抛出异常
<br>
释放:`delete 指针p`
```c++
//Point类在上文
int main() {
    Poitn *ptr1 = new Point; //调用默认构造函数
    delete ptr1;  //删除对象，释放内存，调用默认的析构函数
    ptr1 = new Point(1,2);
    delete ptr1;
    return 0;
}
```

###3.2 分配与释放动态数组
申请:`new 类型名T[]`
<br>成功:返回T类型的指针<br>
失败:抛出异常
<br>
释放:`delete[] 指针p`
```c++
int main() {
    Poitn *ptr1 = new Point[2]; //创建数组对象，每个元素调用默认构造函数
    ptr1[0].move(5,10); //通过指针访问数组元素成员
    delete[] ptr1;
    return 0;
}
```
多维数组的指针是指向数组的指针，如二维数组的指针指向的是行数组的指针，指针运算以整个数组空间为单位
```c++
#include <iostream>
using namespace std;

int main() {
    int (*cp)[9][8] = new int[7][9][8];
    for (int i = 0; i < 7; ++i) {
        for (int j = 0; j < 9; ++j) {
            for (int k = 0; k < 8; ++k) {
                //*(*(*(cp + i) + j) + k) = (i * 100 + j * 10 + k);
                cp[i][j][k] = (i * 100 + j * 10 + k);
            }
        }
    }
    for (int i = 0; i < 7; ++i) {
        for (int j = 0; j < 9; ++j) {
            for (int k = 0; k < 8; ++k) {
                cout << cp[i][j][k] << " ";
            }
            cout << endl;
        }
        cout << endl;
    }
    delete[] cp;
    return 0;
}
```
显然，动态分配数组太过麻烦，于是将动态数组封装成一个类，下面封装一个Point动态数组类



###3.3 vector对象
1.封装任何类型的动态数组，自动创建和删除
<br>2.数组下表越界检查
<br>3.size()方法返回数组长度
<br>4.可以直接用数组下表访问数组元素
<br>5.创建方式 `vector<元素类型> 数组对象名(数组长度);`
```c++
#include <iostream>
#include <vector>
using namespace std;
double average(const vector<double> &arr) {
    double sum = 0;
    for (int i = 0; i < arr.size(); i++) {
        sum += arr[i];
    }
    return sum / arr.size();
}
int main() {
    vector<double>arr1(3);
    for (int i = 0; i < 3; ++i) {
        cin >> arr1[i];
    }
    cout << average(arr1) << endl;
    vector<int>arr2 = {1,2,3};
    for (auto i = arr2.begin(); i != arr2.end(); ++i) {
        cout << *i << endl;
    }
    for (auto e : arr2) {
        cout << e << endl;
    }
    return 0;
}
```

###3.4 浅层复制与深层复制
**浅层复制**：通常默认的复制构造函数都是浅层构造，只将被复制对象的值一一赋值给了新对象，但实际上新对象与被复制对象指针指向的是同一个内存空间
<br>**深层复制**：为新对象开辟新的内存空间，再给被复制对象的的成员的值一一复制给新对象的成员

###3.5 移动构造
没有必要进行复制，源对象对资源的控制权全部交给目标对象
```c++
#include <iostream>

using namespace std;

class IntNum {
public:
    IntNum(int x = 0) : xptr(new int(x)){} //构造函数
    //IntNum(const IntNum &n) : xptr(n.xptr){} //复制构造函数，浅层复制
    IntNum(const IntNum &n) : xptr(new int(*n.xptr)){} //复制构造函数，深层复制
    IntNum(IntNum &&n) : xptr(n.xptr){ //移动构造函数
        n.xptr = nullptr;  //&&右值引用，右值：即将消亡对象；函数返回的临时变量
    }
    ~IntNum() { //析构函数
        delete xptr;
    }
    int getInt() {
        return  *xptr;
    }
private:
    int *xptr;
};
IntNum getNum() {
    IntNum a;
    return a; //实际上返回的是一个临时无名对象
}
int main() {
    cout << getNum().getInt() << endl;
    return 0;
}
```