## Ch8

## 8.1 IO类

常见的头文件：iostream(从流中读写数据)，fstream(从文件中读写数据), sstream(从字符串中读写数据)

读写:istream(读)、ostream(写)、iostream(兼具读写)

支持宽字符：wistream(读)、wostream(写)、wiostream(兼具读写)

io类之间的继承关系：ifstream和istringstream都继承自istream，可以使用istream的特性。

io对象不能被拷贝，也不能被赋值：意味着只能以引用方式传递io类型的形参、返回io类型的值。直接把形参和返回类型设成istream而不是istream&会涉及到拷贝问题

读写io对象会更改其状态，因此io对象不能被设为const

io对象的条件状态：strm是io对象

- strm.rdstate()获取当前std::__1::ios_base::iostate类型,strm.goodbit等都是这个类型，可以做位运算。如果复位状态时可以选择只复位一个bit位，其他bit保持不变，如strm.clear(strm.rdstate() & ~strm.failbit)，clear里面的参数，他的failbit肯定是0，其他位保持不变
- strm提供函数查询上面的状态位是否被设置，如strm.good()一旦goodbit被设置了就返回true
- 判断流是否正常工作的最简单条件：直接把流当作条件，如while(cin >> i) cin>>i如果流出错了返回false，流处于有效状态会返回true

缓冲区：os为提升性能会把要io的内容保留在缓冲区里。

- 什么时候刷新缓冲区：程序结束、缓冲区满、关联流读写(cin默认是关联到cout的，这是为了实时交互，比如用户通过cin输入数据，程序通过cout给用户答案，那么必须保证用户每次cin前cout里的东西都能打印出来，读cin的时候cout的缓冲区必须更新)

- 强制更新缓冲区的方法：end/endl/flush/unitbuf(长期有效)

流之间的关联：strm.tie()//返回strm所关联的流的指针(没关联到流就返回nullptr)；strm.tie(指向某个流指针)//把strm和参数的流关联；strm.tie(nullptr)//关联一个空指针，意义是和当前德流解除关联

## 8.2 文件输入输出

- 初始化一个文件流 ：ifstream in("a.txt")【以默认方式打开一个文件】、ifstream in("a.txt",ifstream::in)【指定文件模式打开文件】
- open和close函数：in.open("b.txt")打开b文件。必须close一个文件，才能用流对象打开下一个文件
- 自动构造和析构：一个fstream对象被销毁时会自动调用close()

每个流都有一个关联的文件模式：

- ifstream默认设置in模式(读模式)
- ofstream默认out模式(写模式)
- 写一个文件出现的情况：out模式和trunc模式会把文件内容清空在写入数据；app模式可以在后面追加内容；书上不建议用写的时候用in模式打开，但是测试的时候发现不会报错，它会从头开始覆盖文件内容，比如这次写了10个字节，那么文件前10个字符会被覆盖。
- 保留被ofstream打开的文件，还想保留内容，只有app模式和in模式，前者追加，后者从头开始覆盖
- ofstream::app和ifstream::app功能一样的，测试的时候发现用哪个都一样

```c++
#include <fstream>
using namespace std;
int main() {
    ofstream out("a.txt", ifstream::in);//可以写入数据,不会覆盖数据，但是会覆盖最开始的几个字符
    //ofstream out("a.txt", ifstream::out);//覆盖数据重新写入
    //ofstream out("a.txt", ofstream::app);//可以正常写入数据,会换行
  	//ofstream out("a.txt", ifstream::app);//和ofstream::app一致
    out << "283334888888" << endl;
   // ifstream in("a.txt", ifstream::in);
    //in << "9898" << endl;//报错invalid operands to binary expression ('std::ifstream' (aka 'basic_ifstream<char>') and 'const char [5]')
    out.close();
}
```







## 8.3 string流

定义于sstream头文件下：

- istringstream：从string中读数据
- ostringstream：向string写数据
- stringstream:既可以从string中读数据、也可以向string写数据
- istringstream i;//istringstream i(string s);定义一个istringstream对象/定义一个和字符串s的拷贝绑定的istringstream对象，可以从该字符串读数据【其他对象也有这个功能】
- istringstream.str()//返回所绑定字符串的拷贝

```c++
#include <sstream>
#include <string>
#include <iostream>
using namespace std;
int main() {
    string s = "haha test baba";
    istringstream record(s);
    ostringstream ll(s);
    ostringstream l1;
    string u1 = ll.str();
    cout << u1 << endl;//haha test baba
    string s1;
    string s2;
    record >> s1;
    record >> s2;
    cout << s1 << endl;//haha
    cout << s2 << endl;//test
    ll << " " << s1;
    string u2 = ll.str();
    cout << u2 << endl;// hahatest baba
    l1 << s1;
    string u = l1.str();
    cout << u << endl;//haha
}
```

- 对一个istringstream对象，如果与它绑定的字符串带空格，那么它在执行>>的时候会按照空格分开字符串。
- 对于一个已经有值的ostringstream对象，向里面写入值会从开始的位置覆盖字符串内容，如上面ll前5个字符是haha+空格，后又写入空格+haha，覆盖了旧的前5个字符，所以变成了 hahatest baba