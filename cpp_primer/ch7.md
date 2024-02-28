# Ch7

## 7.1 定义抽象数据类型

定义一个类，需要哪些组成部分？

Sales_data.h

```c++
#include <string>

struct Sales_data {
  	friend std::istream &read(std::istream &is, Sales_data &item);//友元声明
  	Sales_data() = default;
    Sales_data(const std::string &s):bookNo(s) { }
    Sales_data(const std::string &s, unsigned n, double p):bookNo(s), units_sold(n), revenue(n*p){ }
    std::string isbn() const { return bookNo; }; //类内定义 const成员函数
    Sales_data& combine(const Sales_data&); //类外定义
    
    std::string bookNo;
    unsigned units_sold = 0;
    double revenue = 0.0;
};

Sales_data& Sales_data::combine(const Sales_data& rhs)
{
    units_sold += rhs.units_sold;
    revenue += rhs.revenue;
    return *this;
}

// nonmember functions
std::istream &read(std::istream &is, Sales_data &item)
{
    double price = 0;
    is >> item.bookNo >> item.units_sold >> price;
    item.revenue = price * item.units_sold;
    return is;
}

Sales_data::Sales_data(std::istream &is) //定义在类外的构造函数
{
    read(is, *this);
}
```



- 成员函数：必须声明在类内，定义可以在类外【定义在类内的函数都是隐式的inline函数】
- 非成员的接口函数：声明和定义都在类外

如果在类内和类外同时给成员函数定义会怎么？

```c++
class A { 
  public:
  int m() {//类内，是定义
    return 2;
  }
};
int A::m() {
  return 2;//也是定义，会有redefinition of 'm'异常
}
```



**成员函数部分：**

this:obj是C类型的对象，调用obj的成员方法obj.m()，等价于C::m(&obj)。调用C的m成员，把obj的地址传进去。前面的&obj相当于隐式的形参，它就是this，它是一个常量指针C * const,不允许指向别的对象，指向对象本身。

const成员函数：this默认不是底层const，那要是声明一个const C cobj，岂不是没法用this引用cobj了，cobj也没法去调用非const成员函数？我们没法把this声明为const C * const类型，但是可以定义const的成员函数，这样对于这个函数来说，this就是一个指向常量的常量指针了，此时cobj.isbn()就是合法的。【常量对象及其引用指针都只能调用常量成员函数】

类外定义成员时，函数名前加上类名::前缀,如果是常量成员，那么在定义时后面也要加上const，如isbn()在类外定义时格式是std::string isbn() const，别漏了const。

返回this对象：combine最后返回的是*this，即表示当前对象，可以用Sales_data&去引用

**类相关非成员的接口函数：**

上面的read。一般函数在概念上属于类但是不定义在类中，那么它的定义应该和类的声明在同一个头文件内，这样引入时只要引入一个文件就行。

**构造函数：**

定义类初始化方式，在对象被场景时执行。

- 编译器合成的默认构造函数：没写构造函数时自动合成，初始化方式为

  - 首先是使用类内初始值初始化，比如units_sold类内初始值是0
  - 没指定初始值使用默认初始化，比如上面的bookNo，取的是string类型的默认值，即空字符串

  缺点在于：内置类型要是没指定初始值就会走默认初始化，类是一个作用域，那么内置类型就是未定义的。而且A类内的B成员如果没有实现默认构造函数，那么A类就不能通过默认构造函数初始化B成员。

- 自己设置的默认构造函数：Sales_data() = default;作用完全等价于编译器合成的默认构造函数

- 自己设置的其他构造函数：:bookNo(s)是构造函数初始值列表，格式为成员名(值)。没有出现在构造函数初始值列表的成员以合成的默认构造函数的方式初始化。

构造函数可以定义在类外。要加上类名::前缀

除了构造函数外，对象的拷贝、赋值和析构如果没有自定义也将由编译器合成。

- 对象的拷贝：发生在初始化变量、以值方式 传递或返回一个对象(如函数的参数)

- 对象的赋值：使用赋值运算符
- 对象的析构：对象被销毁时

## 7.2 访问控制与封装

类内的public说明符后定义的成员全程序可访问；private说明符后定义的成员仅类内可访问。

class vs struct：类内第一个访问说明符前定义的成员，class默认是private ，struct默认是public。要是不定义说明符，那struct就全是public,class全是private。

**友元**：如果一个类的私有成员，还希望给非内部类成员访问，该怎么做？(这种需求很奇怪,java里会给私有成员设置一个public的访问方法)。通过友元。

比如非成员接口函数，它不是类的一部分，无法访问私有成员，需要把它声明为友元就可以了，格式 friend + 函数声明

注意：

- 友元只能声明在类的内部
- 友元不是类的成员
- 友元声明 <> 友元函数的声明，要是在类外要用这个函数需要再声明一下这个函数。一般都把友元函数的声明和类定义在同一个头文件中，这样引入这个类的时候也能引入友元函数了方便一点。

## 7.3 类的其他特性

```c++
class Screen {
    public:
        using pos = std::string::size_type; //类型成员

        Screen() = default; 
        Screen(pos ht, pos wd):height(ht), width(wd), contents(ht*wd, ' '){ } // 2
        Screen(pos ht, pos wd, char c):height(ht), width(wd), contents(ht*wd, c){ } // 3

        inline char get() const { return contents[cursor]; }//显示声明为内联函数
        char get(pos r, pos c) const { return contents[r*width+c]; }
  			Screen &move(pos r, pos c);
  			void add() const;

    private:
        pos cursor = 0;
        pos height = 0, width = 0;
        std::string contents;
   			mutable size_t ac;//可变成员
};

inline //类外定义声明成inline
Screen &move(pos r, pos c) {
  return *this
}



int main() {
    Screen::pos i = 2;
    Screen::pos j = 6;
    const Screen s(2,6);
    s.add();//如果没加mutable关键字会报cannot assign to non-static data member within const member function 'add'
  
}
```

**定义类型成员**：类内不光可以定义普通成员，也可以定义一个类型成员pos，它就是表示类型。可以用于封装普通成员的实现细节，比如上面的例子没人知道cursor是string，只知道它是pos。使用时，使用Screen::得到pos类型。

类内函数可以被声明为inline的。既可以在类内时加inline，也可以在类外定义时加inline。

成员函数可被重载。

**可变数据成员**：希望某个对象即使是const对象，也可以修改它内部的某个成员，可以在这个对象前面加上mutable关键字，这样即使是常量成员函数内也可以执行修改操作（在一个const对象内修改非mutable的成员的结果可以参见上面例子）

**类内初始值：**可以用=或{}给类内成员赴一个初始值，从设计角度说这样可避免未定义的问题。上面pos cursor = 0;就是用=号的方式。也可以选择pos cursor{0};这种方式。

**返回\*this**:对于普通成员函数和常量成员函数有不同。普通成员函数返回的this*是普通引用，可以修改其成员；const成员函数返回的this*是const引用，不可用修改其成员。

**基于const的重载**：回忆成员函数调用的本质，C::m(this* &obj)。参数类型就是this*，对于常量成员函数来说，this*是const引用，对于成员函数来说，this*是普通引用。所以当obj是const对象时，当它调用普通成员函数，那么等价于用*this(非const)去引用const对象，会报错；其他的调用方式没问题。因为const和非const成员在调用主体上存在区别，所以f()和f const()构成重载关系。

**类的声明**：

- 当声明了一个类但是没有定义它时(class Screen)，可以**定义**指向这个类的指针和引用，可以**声明**(不能定义)以该类为参数和返回类型的参数，不可作为其他类的成员(field has incomplete type 'Money')

**友元**：可以把一个类(friend class B)或者一个类的某个成员函数(friend void B::f())定义成A类友元，这样该类/该成员就可访问A的private成员。

## 7.4 类的作用域

类外使用类内成员，前面加上类::前缀，如上面的Screen::pos。在类内就不需要加了，但是如果参数和成员同名时,可以通过this或者前缀强制指定成员。(set(string name) {Screen::name = name;/this->name = name})。

cpp编译类的顺序：先处理全部声明，再处理成员函数定义。

```c++
//typedef double Money;1
class A {
    public:
        //typedef double Money; 2
        Money balance() { return bal;}
    private:
        //typedef double Money;3
        Money bal;
};
//typedef double Money; 4
int main() {
    A a;
    a.balance();
}
```

**类的成员的声明的名字查找顺序**（以Money为例）：

- 首先是类内在Money balance() { return bal;}之前出现的声明/定义，比如上面的2
- 然后是类外在Money balance之前出现的声明/定义，比如上面的1

上面的例子，定义在3和4的位置都找不到Money，因为声明所使用的名字有一个重要的特点就是**确保可见**。如果在1和2出定义了Money，在3的位置再定义typedef double Money;就不允许了(redefinition of 'Money')。这也是因为确保可见，此时Money已经找到了，就不允许重新定义了(不然Money balance()和3处定义的不是同一个Money，其他用到Money的成员选哪个？)。*在类内成员使用了外层作用域的某个名字，那么类内不允许重新定义该名字*。

**成员定义的名字查找**

这次不是声明了而是定义，因为类内声明在定义前都被解析完了，因此查找顺序为：

- 成员函数内
- 类内(不要求确保可见，在类内成员函数后面定义的名字也可以)
- 类外&&成员函数定义之前(确保可见，不能使用定义在成员函数后的名字)



## 7.5 构造函数再探

构造函数不能被声明为const，因为只有构造函数执行完初始化的时候对象才获得常量属性。(p235)

**构造函数初始值列表**：

明确赋值和初始化的区别：当构造函数开始执行时，初始化的过程就已经完成了，构造函数体里执行的把参数赋给成员的操作是**赋值**。在此之前，在类内堆成员的直接初始化和类内成员初始值列表会先执行。那么对于const或者引用类型的成员，不能在构造函数体里给它赋值，而是要么直接初始化，要么最迟在构造函数初始值列表完成初始化。

```c++
class A {
  public:
  	A(int val):a(val) {//构造函数初始值完成初始化
       c = val;//error，这里是赋值操作而不是初始化操作
    };
  private:
  const int a;//通过类内成员初始值列表完成初始化
  const int b = 10;//直接初始化
  const int c;
}
```

最好令类内成员初始值列表的顺序和成员定义顺序保持一致，因为成员的初始化顺序与定义顺序相同而不是与初始值列表里的顺序相同。成员int i定义在int j前，在初始值列表里写j(2),i(j)那么i不会变成2，因为此时i(j)执行在j(2)前，此时j还没有初始化。

**默认实参与默认构造函数**：如果一个构造函数为所有成员都提供了一个默认实参，那么它实际上就是默认构造函数。所以默认构造函数的定义，不是参数列表为空的构造函数，而是A default constructor is a constructor that is used if no initializer is supplied。调用的时候没有指定任何实参，最好会生效的函数那么它就是默认构造函数。指定默认实参的构造函数当然也属于默认构造函数。

**委托构造函数**：成员的初始值列表可以写成其他构造函数，让被委托的函数完成初始化操作。

```c++
class A {
  public:
  	A(int v1, int v2, int v3):a(v1), b(v2), c(v3) {};
  	A(int i): A(i,0,0) {};//委托构造函数，委托A(int v1, int v2, int v3)完成初始化
  	A(){};//显示请求值初始化，它就是默认构造函数
  private:
    int a;
    int b;
    int c;
}
```

什么时候会调用默认构造函数？

对象被默认初始化或者被值初始化时调用默认构造函数。p262介绍了几种默认初始化和值初始化的场景。两者看起来只是使用场景不同(值初始化举的例子是在说vector)？本质好像都差不多。在类里把构造函数定义成T()这种空参数的，就是在显示请求值初始化，进而调用默认构造函数。

使用默认构造函数：A a;后面不要加()，写成A a()那就是在定义一个返回A类型的名字为a的函数了。

**转换构造函数**：只定义了一个参数的那种构造函数，可以隐式的把参数类型转换成类的类型，在某些场合你需呀一个A的类，而这个A的类恰好有一个单个参数且类型为string的构造函数，那么可以直接用这个string去代替这个类去使用

```c++
#include <string>
class A {
    public:
        A(std::string val):s(val) {};//单个参数
        int merge(const A & a) {//要想让merge支持隐式转换的使用方式，那么参数需要声明为const，不能是A& a
            return 2;
        }

    private:
        std::string s;
};

class B {
    public:
        explicit B(std::string val):s(val) {};
        int merge(const B & a) {
            return 2;
        }

    private:
        std::string s; 

};

int main() {
    A a1("hahaha");
    std::string sb = "hehe";
    a1.merge(sb);//这里发生了隐式类型转换，merge需要的参数是A类型，但是这里传过去一个string类型也能通过编译。
    a1.merge("hehe");//error，发生了"hehe" -> string 、string -> A两次类型转换，编译失败
  	B b1("haha");
    b1.merge(sb);//no viable conversion from 'std::string' (aka 'basic_string<char>') to 'const B'
  	A a2 = sb;//ok
  	B b2 = sb;//no viable conversion from 'std::string' (aka 'basic_string<char>') to 'B'
  	b1.merge(static_cast<B>(sb));//ok
}
```

使用隐式转换的注意点：

- 只允许一步类型转换，上面的a1.merge("hehe"),传入string类型还好，如果直接传了一个string类型字面量那么会发生字面量到string的隐式转换，编译失败
- 什么样的场景支持这种隐式转换？上面的merge如果把参数定义成A & a那么它在接受string类型的参数会报错。因为隐式转换的本质是：调用A(sb)生成一个**临时量**，再把它作为merge的实参。非const引用是不能绑定一个临时量的，只能用const引用。因为const引用无法修改引用对象的值，所以cpp允许让const引用去引用临时量。注意这里的本质区别，隐式类型转换生成了一个临时量。

抑制构造函数的隐式类型转换：在构造函数的声明前加上explicit，那么上面那种b1.merge(sb);就会报错了。

使用explicit的注意点：

- 不能用explicit使用赋值符去初始化对象。A a2 = sb;这种方式是ok的，它的内容应该是先调用参数为string的构造函数生成一个临时量再去给a2赋值，这个过程涉及到了隐式转换；但是使用explicit的构造函数，不支持这种操作。
- 可以使用static_cast，把string类型显式转换为B，这样explicit是不会限制的。（因为这叫显示转换，不是偷偷摸摸的隐式转换）



**聚合类**：所有成员都是public && 没有定义任何构造函数&& 没有定义类内初始值 && 没有基类和虚函数

聚合类如何初始化：要么是使用默认初始化、要么使用花括号括起来的初始值列表(列表顺序和类内成员定义顺序一致，如果列表元素少于成员个数那么靠后的成员值初始化)如，Data a = {1,2}

**字面值常量类**：constexpr函数的参数和返回值必须是字面值类型。如果想让它返回一个类可以吗？可以，只要把该类定义为字面值常量类即可。

字面值常量类的要求：数据成员必须是字面值类型、至少包含一个constexpr构造函数、类内成员如果被赋予了初始值那么该初始值必须是常量表达式、类内的类类型成员必须实现constexpr构造函数。

满足上面要求的类，我们把它的构造函数声明成constexpr，那么调用该构造函数返回的类型就可以当做constexpr了。

```c++
class A {
  public:
  	constexpr A(int val):a(val) {};//和普通构造函数一样，就是前面多了个constexpr
  private:
  int a;
}

int main() {
  constexpr A a(100);//调用的是constexpr构造函数，返回结果自然也可以被用于constexpr。
}
```



## 7.6 静态成员

声明静态成员：前面加static。

静态成员不与任何对象绑定在一起，不包含this指针，这就给了静态成员两个特性：

- 静态成员函数不能被声明为const，还记着const成员函数的本质是什么吗？顶层const的this指针。它不涉及指针所以不能被声明为const
- 不能在静态函数体内使用this指针。

使用静态成员：1. 类名:: 静态成员  或  2 对象名.静态成员

静态成员的定义规则：

- 在类内声明静态成员。如果选择在类外定义静态成员，不能加static关键字，类内已经加了

- 对于静态成员函数：可以在类内定义也可以在类外定义，类外定义时方式和普通成员函数无异

- 对于静态数据成员：非const静态成员必须定义在类外，例外是也可以在类内把静态成员声明为constexpr并且赋一个字面量初始值，这样在类外定义的时候就不用提供初始值了。

  ```c++
  class A {
    public:
    static int i;//如果你要在类内给i初始化会有这个错 non-const static data member must be initialized out of line
    constexpr static int j = 10;//可以在类内初始化
    static int g();
  };
  int A::i = 100;//类外定义i
  int A::g() {//类外定义g()
    return 10;
  }
  constexpr int A::j;//类外定义j，初始值在类内提供了，这里要是给他赋值会出static data member 'j' already has an initializer
  ```

- 这里梳理下类内相关成员的定义要求：

  - 成员函数不论是static or non-static：都可以在类外定义，也可以在类内定义(声明的时候就定义了）。同时在两个位置定义会有重定义问题。
  - 静态数据成员：非const必须类外定义，类内声明，不会出现声明即定义的问题。const静态成员，必须在类内初始化，constexpr static int j = 10;这种就算声明，不叫定义。类外可定义可不定义，建议定义一下，别提供初始值。
  - 普通nom-static成员走的是初始化，构造函数的路子。



静态成员和普通成员区别：

- 静态成员可以是不完全类型，非静态成员必须是完全类型。一个类A如果要使用不完全类型作为它的成员，要么把它声明为static，要么只能把它声明为指针和引用类型。见P271
- 静态成员可以作为默认实参：A内的参数f，成员int b，f要把数据成员b作为默认实参，那么b必须被声明为static。非静态成员不能作为默认实参，因为它的值本身就是对象的一部分，对象都没初始化完呢就用这个成员做默认实参会引发错误。