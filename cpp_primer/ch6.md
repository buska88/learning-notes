

# Ch6

## 6.1 函数基础

函数定义：函数返回类型 + 函数名 + 函数形参列表 + 函数体

函数声明：函数返回类型 + 函数名，函数形参列表可以省略，无需函数体

函数调用：使用实参初始化形参，return后移交控制权给调用者

局部变量vs自动对象：两种有交集。

- 局部变量：形参、定义在函数体内的变量
- 自动对象：声明周期在块执行周期的对象
- 是局部变量但是不是自动对象：局部静态变量，比如在函数体内定义的static int i = 0;它在函数结束后也不会销毁；局部静态变量没有显示初始化后会执行值初始化(基本类型被初始化为0，其他类型由类定义决定)，而不是未定义

在头文件中声明函数，用到函数的源文件需要把头文件include进来

**chc6.h**

```c++
int fact(int val);
int func();

template <typename T> 
T abs(T i)
{
    return i >= 0 ? i : -i;
}
```

**fact.cc**

```c++
#include "chc6.h"
#include <iostream>

int fact(int val)
{
    if (val == 0 || val == 1) return 1;
    else return val * fact(val-1);
}

int func()
{
    int n, ret = 1;
    std::cout << "input a number: ";
    std::cin >> n;
    while (n > 1) ret *= n--;
    return ret;
}
```

**factMain.cc**

```c++
#include "chc6.h"
#include <iostream>

int main()
{
    std::cout << "5! is " << fact(5) << std::endl; 
    std::cout << func() << std::endl; 
    std::cout << abs(-9.78) << std::endl;
}
```

**编译命令**

```shell
 clang++ -c factMain.cc #编译factMain.cc生成factMain.o
 clang++ -c fact.cc #编译fact.cc生成fact.o
 clang++ factMain.o fact.o  #生成可执行文件a.out
 ./a.out #执行a.out
```

https://cloud.tencent.com/developer/article/1778247

编译的时候使用clang++，而不是clang



## 6.2 参数传递

传参：使用实参初始化形参

引用传递：形参是引用类型，绑定到实参上

值传递：形参和实参是两个独立的对象，实参拷贝给形参

判断：f(int *p)这种声明的函数是值传递还是引用传递？是值传递，只不过把指针的值拷贝给形参，形参和实参是两个不同的对象，但是都可以对所指向的同一个对象操作。

指针，引用和值的参数类型，其中值和引用被认为是同一个函数

```c++
#include <iostream>
int reset(int *p) {
   std::cout << "*p"; 
}

int reset(int &p) { //引用类型
   std::cout << "&p"; 
}

//int reset(int p) {//wrong,和上面的reset(int &p)是同一个函数
   //std::cout << "p"; 
//}

int reset(const int &p) {//和上面的reset(int &p)是重载关系
   std::cout << "p"; 
}


int main()
{
    int val = 10;
    reset(val);//可以绑定到reset(int &p)，也可以绑定到reset(const int &p)
    reset(&val);//*p
  	reset(100);//100是常量，这种只能被int reset(const int &p)调用，不能被reset(int &p)
}
```

设计参数类型的考虑：

- 尽量使用引用作为参数类型，避免值传递，因为有些对象拷贝起来开销很大。
- 如果有这样的场景：要吧一个参数传到函数中并在函数体修改它的值，这个时候就应该使用引用类型
- 优先使用const引用而不是普通引用作为参数类型，前者可以接纳的参数范围更广，常量也可以作为实参传进去；后者无法作为常量的左值
- 顶层const/底层const和参数类型：
  - 形参会忽略掉顶层const的特性，即f(const int p)和f(int p)是同一种函数，不管实参是不是const，初始化给const int p 和int p都是一样的，所以它们属于重复定义；
  - 底层const特性不同的形参会被视为两种参数，f(const int &p)和f(int p)是两个不同的函数，前者可以接受f(100)这种字面量的调用，但是后者不行，初始化给它们两完全是不同的。const int &p属于底层const，即p所引用的对象不能再改变了。

把数组作为参数的注意点：

- 数组不允许拷贝初始化给另一个数组
- 数组通常会被转化为指针

这导致数组格式的形参，下面几种定义方式都是一样的

```c++
void f(const int*);
void f(const int[]);
void f(const int[10]);//这里是形参期望的数组维度，但是实际上它也可以引用维度不为10的数组，如{1,2}

//上面3中定义方式都接受下面的调用
int j[] = {1,2}
f(j);//此时函数体内的形参就是指向数组的指针
```

const int[10]能引用{1,2}很奇怪，这是形参变成int*的缘故，反正都是指向int[]的指针，所指向的数组维度是10还是2没啥区别。这样的危险之处在于函数内部不知道数组的实际大小,所以安全起见在开发时最好把数组长度信息传递进去，方式有：

- f(const int * start, const int * end),传递时把头指针和尾后指针也传进去f(begin(arr),end(arr));
- f(const int * start,size_t size)直接把数组长度传进去

可不可以使用引用的方式接受数组实参？

可以 f(int (&arr)[10])，参数定义成这种方式，但是缺点在于不灵活，只能引用一个长度为10的数组，没有上面f(j)那种方式那么灵活

传递多维数组作为参数：size为(3,4)的数组arr，想把arr作为实参，那么形参格式应该是一个指向长度为[4]的数组的指针：f(int (*p)[4])



通过命令行执行c++函数时，命令行参数和main参数的对应关系？

main定义成下面的格式：

int main(int argc, char * argv[])

- argc： argv中参数的个数
- argv：argv从格式上看是一个包含 指向c风格字符串字符的指针 的数组
- argv[0]：一般都是程序名，不作为参数
- argv被C风格字符串初始化，所以argv最后一个元素指向的是'/0'，这个'/0'也是计入参数个数的



可变参数：

- 如果参数类型一致，可以使用initializer_list类型的形参，f(initializer_list<string> i)，向它传递实参时可以用{}把参数扩起来。initializer_list支持begin和end(**支持他俩就支持范围for了，见5.4.3**)
- 省略符f(a,...)，...和其他参数可以混用，但是需要放在最后一位；对应...这种格式的形参，绑定实参时不做类型check。



## 6.3 返回值

返回值：返回结果会作为一个临时值，拷贝给调用点。

不要返回局部对象的引用和指针，因为函数结束后地址被释放掉。

返回类型为引用类型时，返回值是左值；否则返回值属于右值。

getChar返回的是一个对char的引用

```c++
#include <iostream>


char& getChar(char * charr, int i) {
  return charr[i];
}
char arr[4] = {'1', '2', '3', '4'};

int main() {
    getChar(arr, 0) = 'A';
    std::cout << arr[0];
}
```

函数可以返回列表吗？可以。允许直接return {"a","b"};这种。返回值可以用vector<string>引用；如果返回值是列表格式且想被int引用，那么{}中只能有一个元素，即{1}这种



main函数的返回类型：对于int main()这种函数，可以显示不写return语句，编译器会隐式插入一个return 0;

通过main函数的返回值判定执行结果成败：默认0表示成功，其他值表示失败，但是失败的含义是什么和机器有关；要想规范化返回类型，可以用头文件cstdlib中定义的预处理变量EXIT_SUCCESS和EXIT_FAILURE

返回数组指针类型的函数声明的4种方式：

```c++
//1.使用类型别名
using arr = int[10];
arr* func(int i);//arr就是int[10]的别名，arr*就是指向int[10]的指针

//2.声明返回数组指针的函数
int (*func(parameters))[10];

//3.尾置返回类型，使用auto占据之前应该放返回值类型的地方.任何函数定义都可以使用尾置返回类型，只不过一般都是返回复杂类型时才这么写
auto func(parameters) -> int(*)[10];
auto func(parameters) -> string(&)[10];

//4.使用decltype
int odd[2] = {1,2};
decltype(odd) * func(parameters); //使用decltype只能推断出数组是int[2]类型但是没有把数组转指针的功能，这里手动加上指针
```

## 6.4 函数重载

允许形参不同，其他都一致的函数构成重载关系；仅返回类型不同的函数彼此不构成重载，是错误定义。

重载函数参数关系：

- 省略形参名可以是重载关系：f(int &a)和f(int &)
- 类型别名可以是重载关系：using A = B;f(A)和 F(B)

重载与const：

- 只有顶层const的区别的形参，顶层const会被忽略，不构成重载关系，属于重复定义，如f(int p)和f(const int p)不是重载关系；f(int*p)和f(int * const p)不是重载关系
- 底层const构成重载关系，如f(int &)和f(const int &)是重载关系，重载的原因在于一些常量类型的参数int &引用不了
- 只有f(string &)这一个函数，而且想让它接受const类型参数const string s该怎么办？f(const_cast<string &>(s))，强行把s的底层const去掉即可；const_cast也可以给非const加上底层const属性，string cs;const_cast<const string &>(cs)

重载与作用域的关系：不同作用域的函数不构成重载关系

```c++
print(const string& s);
int func() {
  print(int a);
  print("haha");//wrong；它会覆盖作用域外的print
}
```

原因：c++名字查找发生在类型检查之前，在作用域内找到匹配的函数就不会在继续查了



## 6.5 特殊用途语言特性

**默认实参**：可直接在函数定义里指定参数值，可被覆盖。

void f(int a = 10, int b = 2);

调用时，可以使用f(),f(1),f(1,2)三种方式完成调用。

- 实参列表个形参列表按顺序绑定。实参列表的第一个参数永远绑定形参列表的第一个参数，只有后面的默认实参才可以省略，f(,2)这种试图省略第一个参数直接用第二个参数的会报错
- 形参列表中一个形参配上默认实参了，后面的参数都要配上默认实参（可能是方便灵活配置参数个数的问题？应该和上面的按顺序绑定规则有关），不然f(1)这种用法，谁知道b是几？

默认实参不能是局部变量

```c++
int main() {
    int wd = 100;
    void screen(int a = wd);//wrong，不能用局部变量wd作为默认实参
}
```



**内联函数&&constexpr函数**

在函数定义前加个inline，编译器就会在函数调用过程中直接把函数体的内容替换到调用点上，这样可以节省函数调用的开销。但是这个内联只是一个请求，具体会不会展开由编译器判断

inline int getSize();

constexpr函数：也属于inline函数，可用于常量表达式的函数，要求函数体内只有一条return语句(最多加上声明类型别名、空语句这种不执行任何操作的语句)

```c++
constexpr int scale(int i) {
    return i;
}

int main() {
    constexpr int size = 1;
  	int a = 2;
    constexpr int s = scale(size);//ok
  	constexpr int sb = scale(a);//wrong，scale(a)的返回结果这里应该是个常量表达式，但是这里的返回值不是。
}
```

constexpr函数可以返回常量也可以选择非常量，编译器不会因为这个报错；但是当你把它的返回结果用到需要常量表达式的场合时，如果不是常量那么编译器报错。

一般在头文件中直接定义constexpr函数和inline函数，原因是constexpr函数和inline函数允许被多次定义(方便不同文件用这个函数时能在文件内找到函数定义并展开)，只不过定义必须相同。既然如此还不如把函数直接定义在头文件中。



**调试帮助**

- assert(expr)：根据expr的真假决定是否终止程序。assert定义在头文件cassert中
- NDEBUG:预处理变量，默认未定义(此时assert生效)，定义了那么assert不生效。定义方式：1.cpp文件里写#define NDEBUG 2.编译器 -D NDEBUG main.cpp
- __func__：编译器定义的局部静态变量，作用是保存当前被调试函数的名字
- __FILE、__LINE：预处理变量，用于调试时打印相关信息

## 6.6 函数匹配

什么是最佳匹配？每个实参的匹配不差于其他可行函数，且至少有一个实参的匹配优于其他可行函数。如f(2,3.14)，f(int,int)和f(double,double)都不是最佳匹配。找不到最佳匹配会报错

最佳匹配的优先级：

完美的匹配(参数一致，忽略顶层const) > const_cast转换 > 类型提升 > 算术类型转换

```c++
#include <iostream>
void f1(const short& s) {
    std::cout << "1";
}

void f(short& s) { //对于非const的实参来说，优先级是最高的
   std::cout << "2"; 
}
void f(const int& s) {//short类型提升为int，
   std::cout << "3"; 
}

void f(const char& s) {//类型转换
   std::cout << "4"; 
}

int main() {
    const short &val = 1;
  	short v = 2;
    f(const_cast<short&>(val));//2，通过类型转换可以被short&调用
    f(val);//可以是 1 3 4,按优先级排序
    f(v);//2，非const，所以绑定到short& s上
}
```

对于const short& s和short& s，前者对于const实参优先级是最高的；后者对于非const实参优先级是最高的。



## 6.7 函数指针

bool f(const string &);

函数类型：参数+返回类型 bool(const string &);

函数指针类型：把函数名替换成函数指针类型 bool (*pf)(const string &);



使用函数指针的方式：

- 赋值： 可以直接把函数类型或者函数类型的取址操作赋给函数指针：pf = f;pf = &f;把函数名当成一个值来用时会被自动转为一个函数指针，这点和数组很像
- 通过函数指针调用函数：pf("jha"); (*pf)("jha");可以直接用指针调用，或者使用对指针解引用后的结果。

重载函数与指针：pf = f;如果f有多个重载版本。那么f最后会指向精确匹配的那个函数

函数指针做形参：

ff(bool f(const string &))或ff(bool (*pf)(const string &))，调用时可以直接传入函数对象。

decltype:可以推断出函数类型，但是没有把函数转函数指针的功能(和数组很像)。decltype(f)：是函数类型。decltype(f)* 这种格式才是函数指针



返回函数指针类型：

```c++
//1.使用类型别名
using PF = int(*)(int);//中间的(*)表示它是函数指针类型
PF func();//返回函数指针类型

//2.直接定义
int (*f(int))(int);//从内往外解读，f(int)是函数 -》*f(int)返回类型是指针-》(*f(int))(int)该指针有形参列表所以指向函数-》int (*f(int))(int)该函数返回int

//3.后置
auto func(int) -> int(*)(int);
```