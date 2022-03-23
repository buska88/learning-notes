# 数据结构与算法java

## 1 引论

几个数学概念：

* 1^k + 2^k +……+ N^k近似于 N^(k+1) / |k + 1|  k!=-1，比如k=2时，左式等于N(N+1)(2N+1)/6 = N^3/6，当k = -1时，左边近似等于lnN。

* 模运算 A%N = B，表示为A=B(mod N)，A + C = B + C(mod N)，AD=BD(mod N)

使用接口表示泛型：

比如一个Person类型实现了Comparable接口，重写了compareTo方法，那么可以用Compareble来引用Person类型的对象。

**数组VS泛型**

java中数组是可协变的，泛型集合是不可协变的；像下面method2中传入类型为A[]时编译时无错，运行时有错；泛型集合对数组这一特点做的改良是编译时就会报错，集合是不可协变的。

```java
class A extends Person;
class B extends Person;
// A IS_A Person,B IS_A Person,所以method1方法可以接受类型为A和B的参数
public void method1(Person p);
//参数为数组，但是java规定A[] IS_A Person[],B[] IS_A Person[]，所以实际传入A[]和B[]都是可以的，可以通过编译，但是传入A[]时会在运行时抛出ArrayStoreException
public void method1(Person[] p){
    p[0] = new B();//代码体里可能是这种语句
}
xxx.method1(new A[]);//可以通过编译，运行时抛出ArrayStoreException
public void method1(List<Person> p){
}
xxx.method1(new List<B> b);//编译就不能通过
```

但是如果这样的话就会失去灵活性，所以可以使用通配符增强灵活性：

```java
public void method1(List<？ extends Person> p){
}
xxx.method1(new List<B> b);//编译通过
```

参数p可以被传递所有继承了Person的对象的集合。

List\<Person> IS A List<Person\>但是List\<A> IS NOT A List<Person\>

此时List\<Person> IS A List<？ extends Person>,而且List\<A> IS A List<？ extends Person>

**类型限界**：在尖括号内指定的，要求类型参数必须满足某种性质。实际上就是 \<? extends P\>那么P就是类型限界，发生类型擦除时类型参数不会被擦除为Object而是被擦除为P。

关于泛型的注意事项：

* 不能声明类型与类的参数类型一致的静态域;或参数类型与泛型类型一致的静态方法;或返回类型与泛型类型一致的静态方法。提示信息为Person.this cannot be referenced from a static context

  ```java
  public class Person <T> {
      T a;//ok
      static T b;//wrong
      private static T f1(int x) {//wrong
      }
      private static void f2(T x) {//wrong
      }
      private void f3() {
          T obj = new T();//Type parameter 'T' cannot be instantiated directly
          HashSet<String> [] s = new HashSet<String>[10];//Generic array creation
      }
  }
  ```

  因为类型擦除，Person\<String>和Person\<Integer\>中的static T id;最后都是Object类型，并且在不同的类中共享。

* 参数类型放声明侧可以，但是不能被实例化，见f3。因为T在类型擦除后会由其限界代替，所以可能只是new了一个object，这没有任何意义，它可以用于声明，但是不可以直接实例化。

* 也不能创建泛型数组，因为类型擦除后可能就是一个Object[]，它无法转换为T[]，因为Object IS NOT T。如果真要new数组，使用这种方式：

  ```
  T[] arr = (T[]) new Object[10];
  ```

  

* 不能创建泛型数组，因为类型擦除后它只是创建了一个HashSet []，此时这个数组可以放HashSet<Integer\>，也可以放HashSet<Person\>，会产生异常，所以直接编译时就报错。

## 2 算法分析

相对增长率。注意T(N)=O(f(N)),存在c，n0,当N>n0有T(N)<=cf(N)。N^2 = O(N^3)，f(N)是T(N)的上界。类似，还要下界的定义。

计算f(N)和g(N)的相对增长率，根据结果为0，c或无穷判断f和g之间的增长率关系。

估算程序运行时间的要求：

* 抛弃低阶项
* return fib(n-1) + fib(n-2)这种递归调用时，设执行fib(n)的时间为T(N)，可得T(N) = T(N-1) +T(N-2)，得到T(N)是指数级的数
* 二分查找O(logN)，一种算法将原来为O(N)的问题削减为原来的一半，时间复杂度类似有N个元素的二叉树的高，为lgN
* 辗转相除法的时间复杂度为O(logN)，因为M%N<M/2，所以每次计算出的余数都要小于M的一半，把问题压缩了。
* 计算一个数的N次幂，一般做法是O(N)，但是可以使用递归得方式pow(x,n)=>pow(x*x,n/2)，这里还要讨论奇数和偶数情况。时间复杂为O(logN)

## 3 表、栈、队列

Collection接口实现了Iterable接口，要求实现iterator方法，该方法返回Iterator对象。实现了Iterable接口的集合可以用增强for遍历，编译器看到一个集合使用增强for遍历时，它会将其编译为使用这个集合的迭代器遍历集合。

Collection和Iterator对象内都实现了remove方法，后者删除最近一次调用next返回的项，只有调用next后才可以调用remove，不能连续调用两次remove。

直接使用Iterator时，如果修改了正在被迭代的集合的结构（比如调用add,remove等方法），那么迭代器就不再合法，此时出现并发修改异常。如果使用迭代器遍历集合时使用Collection的remove方法修改了集合是会出现并发修改异常的，而使用Iterator自身的remove后迭代器仍然合法。

在表头添加元素的耗费：LinkedList:O(1),ArrayList:O(N)(涉及到移动数据)

搜索的耗费：LinkedList:O(N),ArrayList:O(1)

ArrayList的实现：

* 数组、size、capacity
* clear方法：清空所有元素，实现方法选择重新new一个大小为默认大小的数组。

* 扩容函数ensureSize，自定义的扩容方式可以选择乘以2
* add、remove、set、get
* 迭代器，用于实现增强for，在内部实现一个继承Iterator的内部类，调用iterator方法时返回它。

这里注意内部类可以访问外部类的域和方法。书上称static内部类为嵌套类、非static内部类为内部类，这里使用内部类而不是嵌套类的原因是对于内部类，编译器会自动加上一个外部类对象的隐式引用，这也是内部类可以访问外部类成员的关键。而使用static的嵌套类，只能使用版本3的形式明确地让嵌套类引用一个内部类对象，否则其中是没有一个具体的外部类对象的的隐式引用的。代码更麻烦。

而且static的内部类无法访问外部类的非static成员。

LinkedList的实现：

* size、头节点和尾节点（空LIst内只有头节点和尾节点，这样设计可以简化代码）,modcnt(对链表的改变次数)
* clear方法：清空所有元素，实现方法选择重新建立头节点，尾节点
* add、remove、set、get
* 一些辅助性方法：getNode 根据id查找一个元素；addBefore 找到第i个元素之后在它前面加上新元素，它用于辅助实现add(id,x)

modcnt有什么用？如果我们要加入并发修改异常这一逻辑，那么就需要在迭代器内保存一个初始的modcnt，当迭代时先检查迭代器内保存的modcnt和表中的modcnt是否一致，如果不一致就说明在迭代期间修改了表，那就抛出异常，没有问题后才可以进行下一步的迭代。

迭代过程中如何防止连续执行remove？设置一个ok2remove域，执行next时把它设为true，执行remove时把它设为false，执行remove前检查ok2remove为true了才可以进行删除。

书上没有在ArrayList的实现中选择没有加入错误检测逻辑，但是在LinkedList的实现中选择加入了。

栈可以用ArrayList或LinkedList实现。

栈的应用：

* 后缀表达式（逆波兰表达式）：1 * 2 + 3 * 4以后缀表达式的写法为1 2 * 3 4 * +，把两个数的运算符写在后面，然后把遍历表达式，遇到一个数就入栈，遇到运算符就把两个数弹出栈计算并将计算结果入栈。
* 普通表达式转后缀表达式：数字输出，运算符入栈，当遇到一个优先级低的运算符时，弹出运算符并输出，左括号优先级最高，遇到右括号时弹出。

队列实现：循环数组，back-font = -1表示队列为空 back == font表示队列满

























