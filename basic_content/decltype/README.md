# decltype那些事

## 关于作者：

个人公众号：

![](../img/wechat.jpg)

## 1.基本使用
decltype的语法是:

```
decltype (expression)
```

这里的括号是必不可少的,decltype的作用是“查询表达式的类型”，因此，上面语句的效果是，返回 expression 表达式的类型。注意，decltype 仅仅“查询”表达式的类型，并不会对表达式进行“求值”。


### 1.1 推导出表达式类型

```c++
int i = 4;
decltype(i) a; //推导结果为int。a的类型为int。

//decltype处理顶层const和引用的方式与auto有些许不同，如果decltype使用的表达式是一个变量，则decltype返回该变量的类型（包括顶层const和引用在内）
//需要指出的是引用从来都作为其所指对象的同义词出现，只有用在decltype处是一个例外
const int ci = 0, &cj = ci;
decltype(ci) x = 0; // x的类型是 const int
decltype(cj) y = x; // y的类型是 const int& ,y 绑定到变量x
decltype(cj) z; //错误：z是一个引用，必须初始化，

//如果declpyte使用的表达式不是一个变量，则decltype返回 表达式结果 对应的类型
//如果表达式的内容是解引用操作，则decltype将得到引用类型。这意味着该表达式的结果对象能作为一条赋值语句的左值。如：解引用指针可以得到指针所指的对象，而且还能给这个对象赋值。
int i = 42, *p = &i, &r = i;
decltype(r+0) b; //正确：加法表达式的结果是int，因此b是一个（未初始化的）int
decltype(*p) c; //错误：c是int& ，必须初始化

//decltype与auto另一个重要区别：decltype的结果类型与表达式形式密切相关。有一种情况要特别注意：
//对于decltype所用的表达式来说，如果变量名加上了一对括号，则得到的类型与不加括号时会有所不同。如果使用的是不加括号的变量，则得到的结果就是该变量的类型；如果给变量加上了一层或多层括号，编译器就会把它当成是一个表达式。变量是一种可以作为赋值语句左值的特殊表达式，所以这样的decltype就会得到引用类型。
decltype((i)) d; //错误：d是int&，必须初始化
decltype(i) e; //正确：e是一个未初始化的int

```

### 1.2 与using/typedef合用，用于定义类型。

```c++
using size_t = decltype(sizeof(0));//sizeof(a)的返回值为size_t类型
using ptrdiff_t = decltype((int*)0 - (int*)0);
using nullptr_t = decltype(nullptr);
vector<int >vec;
typedef decltype(vec.begin()) vectype;
for (vectype i = vec.begin; i != vec.end(); i++)
{
//...
}
```

这样和auto一样，也提高了代码的可读性。

### 1.3 重用匿名类型

在C++中，我们有时候会遇上一些匿名类型，如:

```c++
struct 
{
    int d ;
    double b;
}anon_s;
```

而借助decltype，我们可以重新使用这个匿名的结构体：

```c++
decltype(anon_s) as ;//定义了一个上面匿名的结构体
```

### 1.4 泛型编程中结合auto，用于追踪函数的返回值类型

这也是decltype最大的用途了。

```c++
template <typename T>
auto multiply(T x, T y)->decltype(x*y)
{
	return x*y;
}
```

完整代码见：[decltype.cpp](decltype.cpp)

## 2.判别规则

对于decltype(e)而言，其判别结果受以下条件的影响：

如果e是一个没有带括号的标记符表达式或者类成员访问表达式，那么的decltype（e）就是e所命名的实体的类型。此外，如果e是一个被重载的函数，则会导致编译错误。
否则 ，假设e的类型是T，如果e是一个将亡值，那么decltype（e）为T&&
否则，假设e的类型是T，如果e是一个左值，那么decltype（e）为T&。
否则，假设e的类型是T，则decltype（e）为T。

标记符指的是除去关键字、字面量等编译器需要使用的标记之外的程序员自己定义的标记，而单个标记符对应的表达式即为标记符表达式。例如：
```c++
int arr[4]
```
则arr为一个标记符表达式，而arr[3]+0不是。

举例如下：

```c++
int i = 4;
int arr[5] = { 0 };
int *ptr = arr;
struct S{ double d; }s ;
void Overloaded(int);
void Overloaded(char);//重载的函数
int && RvalRef();
const bool Func(int);

//规则一：推导为其类型
decltype (arr) var1; //int 标记符表达式

decltype (ptr) var2;//int *  标记符表达式

decltype(s.d) var3;//doubel 成员访问表达式

//decltype(Overloaded) var4;//重载函数。编译错误。

//规则二：将亡值。推导为类型的右值引用。

decltype (RvalRef()) var5 = 1;

//规则三：左值，推导为类型的引用。

decltype ((i))var6 = i;     //int&

decltype (true ? i : i) var7 = i; //int&  条件表达式返回左值。

decltype (++i) var8 = i; //int&  ++i返回i的左值。

decltype(arr[5]) var9 = i;//int&. []操作返回左值

decltype(*ptr)var10 = i;//int& *操作返回左值

decltype("hello")var11 = "hello"; //const char(&)[9]  字符串字面常量为左值，且为const左值。


//规则四：以上都不是，则推导为本类型

decltype(1) var12;//const int

decltype(Func(1)) var13=true;//const bool

decltype(i++) var14 = i;//int i++返回右值
```

学习参考：https://www.cnblogs.com/QG-whz/p/4952980.html
