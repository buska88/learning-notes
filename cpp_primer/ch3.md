# Ch3

## 3.1命名空间的using声明

using std::cout

那么代码里可以直接写cout，不用加std前缀；

## 3.2 标准库类型string

使用标准库类型string

```c++
#include <string>
using std::string
```

初始化方式：

- 默认初始化：String s; s是空串
- 拷贝初始化：使用等于号的，String s = s1;此时s是s1的副本
- 直接初始化：使用括号的，String s(s2)；这里s是s2的副本



string常见api：

- getLine(is,s)：遇到换行符返回，但是不会把换行符保存到字符串s中
- string::size_type：s.size()实际返回类型，它是个无符号整数。

处理string内的字符：

引入cctype来处理字符，它是c语言内处理字符的库

```c++
#include <cctype>
for(auto c: str) { //遍历字符串内的字符
  //对c操作
  c = toupper(c);
}
```



## 3.3 标准库类型vector

使用标准库类型vector

```c++
#include <vector>
using std::vector
```

容纳其它类型的容器，是模版，用法：

vector<int> v;

初始化方式：

- 支持拷贝初始化和直接初始化
- 也支持列表初始化，注意列表初始化的要求，当列表中有多个元素时不能使用()，而是要使用{}.(vector<string> a{"a", "b"})

常见api:

- push_back添加元素
- size操作返回vector<T>::size_type类型（注意不是vector::size_type类型，泛型不能丢）

编程要点：遍历vector时不能修改vector内的元素。

## 3.4 迭代器

如果访问string或vector内的元素？可以使用下标，另一种方式是迭代器。所有标准库容器都支持迭代器访问。

迭代器类似指针，提供了间接访问。

获取容器的迭代器的两个方法

```c++
auto b = v.begin(), e = v.end();//b指向容器第一个元素，e指向迭代器的尾后位置
```

尾后位置：容器最后一个元素的下一个位置，该位置以及没有元素了，不能对指向该位置的迭代器做解引用操作

迭代器常见操作：

- 解引用：*iter，返迭代器所指向元素的**引用**
- 移动操作：++、--、+n、-n
- 访问所指对象的成员操作：iter -> 成员名

迭代器具体类型

- vector的迭代器是什么类型（必须包含类型参数信息）：vector<int>::iterator【可以对元素进行读写】、vector<int>::const_iterator【不能通过这种类型的迭代器修改其所指向的元素值】
- string的迭代器是什么类型：string::iterator【可以对元素进行读写】、string::const_iterator【不能通过这种类型的迭代器修改其所指向的元素值】

如果vector和string是常量那么只能使用带const_的迭代器（有点类似常量指针的思想，不允许用可以修改元素的左值去引用常量）

difference_type类型：迭代器减法操作的实际类型



## 3.5 数组

数组定义的要点：

- 必须用常量表达式来声明元素个数：

  ```c++
  int size = 3;
  int arr[size] = {1, 2, 3};//wrong，size不是常量表达式,书上的意思是int arr[size];就会报错，其实不会，但是对这种格式赋值的时候就会报错
  constexpr int s = 3;
  int arr[s] = {1, 2, 3};//ok
  ```

数组初始化的要点：

- 定义在函数体外部的数组默认初始化，定义在函数体内部的数组如果没有显式初始化的话则未定义

- 直接初始化：后面用花括号int a[] = {1,2} ，int a[2] = {1,2}，未指定元素个数时可以通过{}内元素个数推断出数组大小
- 不支持拷贝初始化：不能用一个数组去初始化另一个数组
- 初始化字符数组时注意c风格字符串：char c[] = "c++";//字符串后面还有一个表示字符串结束的'\0'，实际数组长度为4

数组与复杂类型

- int * p[]：p是保存int*的数组
- int (*p)[10]/int(&p)[10]：p是指向数组的指针/引用
- 不存在存放引用的数组，因为引用不是对象

访问数组元素方式：

- 范围for

  ```c++
  for(auto i: arr) {
  }
  ```

- 使用数组下标类型size_t

  ```c++
  for(size_t i = 0; i < 1; i++) {
    cout << arr[i];
  }
  ```

  



如何获得一个指向数组的指针？num是int[]类型数组，方式有：

- int * p = &num[0]; // 直接对某个元素取址赋给指针
- int *p = num;//直接把数组名字赋给p，那么p默认指向数组第一个元素
- begin与end：获取一个数组的头指针和尾后指针

数组与类型推断：

- 定义数组时必须指定数组类型，不能用auto去推断：auto ia = {1, 2, 3};这里不会把ia推断为int[3]；
- auto会把数组推断为指针类型,所以可以用拷贝构造的形式去写
- decltype可以推断出数组类型

```c++
    int ia[] = {1, 2, 3};
    auto ia2 = ia; // ia2是int*，所以可以直接拿ia当右值
    decltype(ia) ia3 = ia;//wrong，ia3是int[3]类型，只能用decltype(ia) ia3 = {1,2,3}这种写法，不能直接拿ia当右值，因为数组不支持拷贝初始化
		int ia4[] = ia;//wrong
```

指向数组的指针的运算：

- p+4:p前进4位
- ptrdiff_t：指向数组的指针做减法运算的实际类型
- 下标操作：数组有下标操作，指向数组的指针同样有下标操作，它表示距离当前指针的偏移并解引用：int *p = &arr[2];p[1]<=> *(p + 1)（arr[3]）;p[-1]<=> *(p - 1) 指向arr[1]

C风格字符串：把字符放在字符数组中并以空字符结束。

cstring头文件：定义了操作c风格字符串的方法

string对象和c风格字符串的混用：

- string类型的字面值类型一般可以使用c风格字符串替换
- string转c风格字符串：使用c_str()方法，返回结果用const char *引用



## 3.6 多维数组

初始化方式：可以直接初始化全部元素，也可以显示初始化某个维度的元素，其他元素默认初始化

下标：如果ia是int ia[3][4]的二维数组：

- ia[i][j]表示具体的元素，可以直接用int引用；
- ia[0]代表ia第1行，可以用指向数组的引用去引用，即int (&a)[4]

遍历多维数组：

- 使用size_t

  ```c++
   for(size_t i = 0;i < 3;i++) {
    for(size_t j = 0;j < 4;j++) {
      //ia[i][j];
    }
  }
  ```

- 使用范围for

  这里加上了&，如果不加会有什么后果？编译报错

  这里row是int (&row)[4]类型，指向数组的引用；col是对int的引用

  ```c++
  for(auto &row: ia) {
    for(auto &col: row) {
      //col;
    }
  }
  ================
  for(auto row: ia) { // 还记得数组与auto的关系吗？此时的row是指向数组首个元素的指针int(*row)[4]
    for(auto col: row) {//这里是对一个指针做范围for，所以编译报错
      //col;
    }
  }
  ```

- 使用指针

  这里利用auto推断数组时会推断成指针的特性

  ```c++
  for(auto p = ia; p != ia + 3;p++) {//p被推断为int(*p)[4]类型，
    for(auto q = *p; q != *p + 4;q++) {//q指向p的解引用结果，q是int*
      //*q
    }
  }
  ```

  