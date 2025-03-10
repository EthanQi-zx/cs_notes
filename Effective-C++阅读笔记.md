# 1、让自己习惯C++

## 条款1：视C++为一个语言联邦

C++是从四个次语言组成的联邦政府，每个次语言都有自己的规约。分别是：
1、C：例如区块，语句，预处理器，数组，内置数据类型等来自C的内容。
2、Object-Oriented C++：例如类，封装，继承，多态等面向对象的内容。
3、Template C++：C++泛型编程部分
4、STL：是一个template程序库，对容器，迭代器，算法以及函数对象的规约有极佳的配合与协调。

## 条款2：尽量以const,enum,inline替换#define

1、条款名称也可以理解为：以编译器替换预处理器

2、#define RATIO 1.53
用const double Ratio = 1.653;替换上述宏(#define)更好一些，因为可能编译器在开始处理源码之前宏就被预处理器移走了

3、string通常比char*更适用：

```c++
// 表示指针和指针所指的内容都是const
const char* const authorName = "Scott Meyers";
const string authorName("Scott Meyers"); //更好
```

4、#define不能够用来定义class专属常量，因为#define并不重视作用域；也不能提供任何封装性，即没有private #define，而const成员变量是可以被封装也可以定义class专属常量

5、enum是C++中的一种数据类型，用于定义枚举数据类型，为一组整型常量赋予有意义的符号名称，例如

```c++
//没有显示给定整数值，默认从0开始递增
enum Color {
    RED,
    GREEN,
    BLUE
};//其中RED表示0，GREEN表示1，BLUE表示2
//显示给定整数值
enum Color {
    RED = 10,
    GREEN = 20,
    BLUE = 30
};
//对应的 RED就是整数10，GREEN就是20，BLUE就是30
```

enum有点像#define，例如
enum {Num = 5}; 和 #define NUM 5

6、用inline修饰函数，函数内容会在调用点直接展开，从而避免了函数调用带来的开销。一般inline函数适用于函数体较小和频繁调用的情况下，例如：

```c++
inline int add(int a, int b) {
    return a + b;
}
int main() {
    int a = 5,b = 10;
    int c = add(a,b); //这里展开之后相当于c = a+b; 省去了调用add函数不必要的开销
}
```

7、某些情况下用inline来取代#define

```c++
//以a和b的较大值作为参数调用函数f
//务必注意为宏中所有的实参加上小括号 避免不必要的麻烦
#define CALL_WITH_MAX(a,b) f((a)>(b)?(a):(b))
//这样定义有太多缺点了，比如：
int a = 5;
CALL_WITH_MAX(++a,0);
//等价于f((++a)>(0)?(++a):(0)); 这个式子相当于直接替换的
// 由于++a大于0，最后结果是++a,所以a被累加两次
CALL_WITH_MAX(++a,10); 
//等价于f((++a)>(10)?(++a):(10));
// 由于++a小于10，最后结果是10,所以a被累加一次
//a被累加的次数取决于跟谁来比较，不合理
```

```c++
template<typename T>
inline void call_with_max(const T& a,const T& b) {
    f(a > b ? a : b);
} //这样就避免了上述的错误
```

8、有了const，enum，inline，我们对于处理器(尤其是#define)的需求降低了，但是并不是完全不需要了，#include仍然是必需品，#ifdef/#ifndef在控制编译器时也很重要

9、总结：1️⃣对于单纯的常量，最好用const或enum来替换#define
2️⃣对于形似函数的宏，最好改用inline函数替换#define

## 条款3：尽可能使用const

1、const修饰的几种情况：

```c++
const char* p = "greeting"; //p指的内容是常量 p不是常量
char* const p = "greeting"; //指针p是常量 p指的内容不是常量
const char* const p = "greeting"; //二者都是常量

//对于函数参数来说，const关键字写在星号之前或者之后是一样的,例如
void f1(const int* pw);
void f2(int* const pw); //都是表示指针指向的值是常量
```

2、如何用const修饰迭代器

```c++
vector<int> vec;
//不能理所应当的把vector<int>::iterator用char*代替 attention
const vector<int>::iterator iter = vec.begin(); //这类似于char* const p;即表示迭代器本身是常量，迭代器所指的内容不是常量
vector<int>::const_vector iter = vec.begin(); //这类似于const char* p;表示迭代器所指的内容是常量，迭代器本身不是常量
```

3、const可以和函数返回值，各参数，函数自身(如果是成员函数)等产生关联，避免程序员误修改产生意外~

4、如果两个成员函数只是常量性不同，则可以被重载

5、成员函数用const修饰时，分为两种不同解释：
1️⃣"Bitwise Constness" 指的是在成员函数中，被声明为const的函数不会修改对象的任何成员变量。这种常量性是对对象数据成员的直接修改进行限制的。只限制对象的数据成员，而不限制对象的指向的数据的修改。
2️⃣"Logical Constness" 概念更广泛，它指的是在语义上，一个被声明为 const 的函数不会改变对象的逻辑状态。这种常量性更多地关注对象的逻辑状态维护。比如文件的打开关闭状态指的就是逻辑状态。这种解释下，成员函数内的有些变量有可能需要被适当修改，由于编译器强制实施bitwise constness，所以要给可能被修改的变量前加上mutable来释放掉bitwise constness约束，使其仍然满足const

6、总结：1️⃣讲某些内容声明为const可以帮助编译器侦测出错误
2️⃣编译器强制实施bitwise constness
3️⃣const成员函数调用non-const成员函数是错误的，因为可能会修改变量值
4️⃣当const和non-const成员函数有着实质等价的实现时，令non-const版本调用const版本可避免代码重复

## 条款4：确定对象被使用前已经先被初始化

1、要给内置的对象进行初始化，因为C++不保证会初始化它们

2、构造函数最好使用成员初值列，而不是在构造函数内部使用赋值操作

```c++
class Person {
    std::string name;
    int age;
public:
    // 使用成员初值列初始化name和age
    Person(const std::string& n, int a) : name(n), age(a) {} //鼓励用这个
    // 使用赋值操作初始化name和age
	Person(const std::string& n, int a) {
        name = n;
        age = a;
    }
};
//除此之外，初始值列出的成员变量，其排列次序应该和它们在class中的声明次序相同
```

3、由于不确定non-local static对象的初始化顺序，则用其中一个去初始另一个对象会出错，所以可以给non-local static对象的初始化套一个函数，需要定义这个值的时候直接调用函数即可，保证了调用的时候一定被初始化了。淘一个函数之后non-local static也变成了local static变量

# 2、构造/析构/赋值运算

## 条款5：了解C++默默编写并调用哪些函数

1、如果你定义一个空类，编译器会自动为你声明一个默认构造函数、拷贝构造函数、析构函数和拷贝赋值操作符，例如：

```c++
// 当你声明
class Empty{};
// 编译器会自动帮你补全
class Empty{
public:
    Empty() {} //默认构造函数
    Empty(const Empty& rhs) {} //拷贝构造函数
    ~Empty(){} //析构函数
    Empty& operator=(const Empty& rhs){} //拷贝赋值操作符
    //当类内含有引用成员(&)，或者含有常量成员(const)，编译器会拒绝编译这一行赋值操作符，需要自己单独定义
}
```

2、编译器自动生成的这些函数权限都是public

## 条款6：若不想使用编译器自动生成的函数，就应该明确拒绝

1、如果你定义的类不允许被拷贝，但是你不定义拷贝赋值操作符编译器也会自动帮你生成，这时可以采用自己写出空函数，且权限为private：

```c++
class HomeForSale{
public:

private:
    HomeForSale(const HomeForSale&); //只有声明 由于函数都不会被调用且没有内容 甚至参数名都不用给出
    HomeForSale& operator=(const HomeForSale&);
};
```

2、如果是成员函数或者友元拥有private权限的，可以专门设计一个基类：

```c++
class Uncopyable {
protected:
    Uncopyable() {}
    ~Uncopyable() {}
private:
    Uncopyable(const Uncopyable&);
    Uncopyable& operator=(const Uncopyable&);
};
```

然后用HomeForSale类继承基类Uncopyable

```c++
class HomeForSale:private Uncopyable {
    
};
```

此时，任何人(即使是成员函数或者友元)想要拷贝HomeForSale对象，编译器就会尝试生成一个拷贝构造函数和一个拷贝赋值操作符，这些函数的“编译器生成版”会尝试调用其基类的对应兄弟，由于基类的拷贝函数是private，所以调用会被拒绝

## 条款7：为多态基类声明virtual析构函数

1、浅浅复习下vitrual函数：

```c++
class Base {
public:
    virtual void display() {
        cout << "Base display()" << endl;
    }
};

class Derived : public Base {
public:
    void display() {
        cout << "Derived display()" << endl;
    }
};

Base* b = new Derived();
b->display(); // 输出 "Derived display()"
//基类和派生类都有这个同名函数 但是基类中加了virtual 为动态绑定，即使是基类指针接收派生类对象，也会执行该派生类中的display函数
```

```c++
class Base {
public:
    void display() {
        cout << "Base display()" << endl;
    }
};

class Derived : public Base {
public:
    void display() {
        cout << "Derived display()" << endl;
    }
};

Base* b = new Derived();
b->display(); // 输出 "Base display()"
//由于不是virtual函数，用基类指针接收派生对象，由于是静态绑定，在编译时就确定了要调用的函数，所以调用的是基类中的display
```

2、基类TimeKeeper中的析构函数不是虚函数，可能会导致“局部销毁”现象

```c++
class TimeKeeper{
public:
    TimeKeeper();
    ~TimeKeeper();
};//基类
class AtomicClock:public TimeKeeper{}; //派生类

TimeKeeper* ptk = getTimeKeeper(); //TimerKeeper是基类，而函数getTimeKeeper返回的是一个派生类的对象指针
delete ptk; //此时由于是用基类指针接收的 所以执行的是基类的析构函数，且基类带着的是non-virtual析构函数，所以执行的是基类中的析构函数而不是派生类中的析构函数，就会导致对象的派生成员未被销毁，只销毁了基类的部分
//这种局部销毁现象会形成资源泄露
```

解决办法是：给基类写一个virtual析构函数:

```c++
class TimeKeeper{
public:
    TimeKeeper();
    virtual ~TimeKeeper(); //virtual析构函数
};
TimeKeeper* ptk = getTimeKeeper();
delete ptk; //此时调用的是派生类中的析构函数，会销毁整个派生对象 即使是用基类指针接收派生类对象
```

3、如果一个类中不含有virtual函数，通常表示它并不愿意被用作一个基类；许多人的心得是：只有当类内至少含有至少一个virtual函数时才将它的析构函数声明为虚函数

4、解释下多态：多态性通过虚函数和继承实现，可以理解成子类中写与父类中同名的函数，父类中的函数是虚函数

5、带有多态性质的基类应该声明一个virtual析构函数；
如果类中带有任何virtual函数，它就应该拥有一个virtual析构函数

6、设计类的目的如果不是作为基类或者不是为了具备多态性，就不应该声明virtual析构函数。

## 条款8：别让异常逃离析构函数

1、如果析构函数内部发生异常，不要让其离开这个析构函数，因为如果允许离开的话，连续多次执行析构的话可能会连续抛出多个异常，会导致一些不明确的行为，不可取。几种方法防止异常逃离析构函数：
1️⃣抛出异常就结束程序，通常用abort完成：

```c++
DBConn::~DBConn() {
    try {
        db.close();//数据库关闭
    } catch() {
        //记录close操作的失败
        abort(); //调用abort函数强制结束程序
    }
}
```

2️⃣吞下因调用close而发生的异常：

```c++
DBConn::~DBConn() {
    try {
        db.close();
    } catch() {
        //记录close调用失败
    }
}
```

一般而言直接吞掉异常是个坏主意，因为隐瞒了“某些动作失败”的重要信息。但是有时候吞下异常也比草草结束程序好。

3️⃣上述两种方法的缺点就是无法对“导致close抛出异常”的情况作出反应，一个较佳策略是重新设置DBConn接口close，允许客户手动调用close函数来关闭数据库的连接，如果关闭不掉的话析构函数中仍然含有一个关闭操作，双保险。客户手动调用close关闭连接，如果出现异常，由于不是在析构函数中出现异常，就可以对异常进行响应处理。

```c++
class DBConn {
public:
    void close() {
        db.close();
        closed = true;
    }
    ~DBConn() {
        if(!closed) {
            try {
                db.close();
            } catch() {
                //制作运转记录 记录close调用失败
            }
        }
    }
private:
    DBConnection db;
    bool closed;
};
```

2、析构函数绝对不要吐出任何异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下它们(不传播)或者结束程序

3、如果客户需要对某个操作函数运行期间抛出的异常作出反应，那么class应该提供一个普通函数(而非在析构函数中)执行该操作。

## 条款9：绝不在构造和析构过程中调用virtual函数

1、假如在构造函数中调用了virtual函数，由于基类构造函数的执行更早于派生类构造函数，当基类构造函数执行时，派生类的成员变量尚未被初始化，此期间调用的virtual函数下降到派生类阶层，但是派生类成员变量还未被初始化，so，dangerous！

换句话说，你无法使用virtual函数从基类向下调用，你可以改成令派生类将必要的构造信息向上传递至基类构造函数

## 条款10：令operator=返回一个对*this的引用

1、为了实现“连锁赋值“，operator=返回值必须是对*this的引用

```c++
x = y = z = 15;
x = (y = (z = 15)); //二者等价
Widget& operator=(const Widget& rhs) {
    //操作
    return *this;
} //返回值是Widget的引用，才能继续跟左侧进行=操作
```

2、其他操作符：+=，-=，*=  等等也是适用的

## 条款11：在operator=中处理“自我赋值”

1、自我赋值发生在对象被赋值给自己

```c++
class Bitmap {};
class Widget {

private:
    Bitmap* pb;
}

Widget& Widget::operator=(const Widget& rhs) {
    delete pb; //这里释放的就是this->pb
    pb = new Bitmap(*rhs.pb);
    return *this;
} //此时如果*this和rhs是同一个对象，就会报错，因为提前释放了this->pb的内存,rhs.pb也是这个内存，也被释放了
```

2、可以使用三种方法防止自我赋值报错：
1️⃣方法一：添加证同测试 

```c++
Widget& Widget::operator=(const Widget& rhs) {
    if(this == &rhs) return *this; //判断是否相同
    delete pb; //这里释放的就是this->pb
    pb = new Bitmap(*rhs.pb);
    return *this;
}
//但是由于实际很少有自己给自己赋值需要证同测试判断的，于是这一行几乎都是!=，很少被用到，所以会降低执行速度


```

2️⃣方法二：修改代码顺序 在复制pb所指东西之前别删除pb即可

```c++
Widget& Widget::operator=(const Widget& rhs) {
    Bitmap* pOrig = pb;
    pb = new Bitmap(*rhs.pb);
    delete pOrig;
    return *this;
}
```

3️⃣方法三：copy and swap

```c++
//先在widget类中添加swap函数
class Widget {
    void swap(Widget& rhs); //交换*this和rhs的数据
}
Widget& Widget::operator=(const Widget& rhs) {
    Widget temp(rhs); //为rhs copy一份副本 用副本的值跟*this交换 赋值对象本身不变
    swap(temp); //之后再进行swap 此之谓copy and swap
    return *this;
}

//或者参数不用引用 这样交换也不会改变传进来参数的值
Widget& Widget::operator=(Widget rhs) {
    swap(temp); //反正传的不是引用 直接交换无所谓
    return *this;
}
```

## 条款12：复制对象时勿忘其每一个成分

首先说明：copying函数指的是：拷贝构造函数和operator=函数，二者都可以被称作是一个copying函数

1、当你手动编写copying函数时，请确保：
1️⃣复制对象内的所有成员变量
2️⃣调用所有父类中适当的copying函数去复制所有的父类成员变量

```c++
PriorityCustomer& PriorityCustomer::operator=(const PriorityCusmoter& rhs) {
    Customer::operator=(rhs); //对父类中的成员进行赋值操作,由于父类中的这些成员往往是private 所以还是瑶调用一下父类的赋值函数 不太好直接访问赋值
	priority = rhs.priority;
    return *this;
}
```

2、用赋值(operator=)函数调用拷贝构造函数是不合理的，这就像试图构造一个已经存在的对象；反过来拷贝构造函数调用赋值(operator=)函数也是不合理的。

如果你发现二者有相同的代码，将重复部分建立一个新的成员函数供二者调用，这样的成员函数往往是private且常常被命名为init

# 3、资源管理

所谓资源就是，一旦用了它，将来必须还给系统。

## 条款13：以对象管理资源

1、为了确保资源总是能够被释放，我们需要将资源放进对象内

```c++
class Investment {
    //操作
};
Investment* createInvestment() {
    //操作
}
void f() {
    Investment* p = createInvestment();
    //操作
    delete p;
}
```

delete p;上面函数的操作有可能存在return或者有异常被抛出，导致delete不能执行，也就是资源得不到及时释放。为了确保资源总是能够被释放，我们需要将资源放进对象内，当控制流离开f，该对象的析构函数会自动释放那些资源。

2、标准库提供的auto_ptr是一个“类指针对象”，即所谓“智能指针”，其析构函数会自动对其所指对象调用delete。即把资源放进auto_ptr中进行管理。auto_ptr使用方法如下:

```c++
void f() {
    auto_ptr<Investment> p(createInvestment());
    //操作
}
//函数f结束后，p会经由auto_ptr的析构函数自动删除
```

由于auto_ptr被销毁时会自动删除它所指之物，所以一定要注意别让多个auto_ptr同时指向一个对象。为了预防这个，auto_ptr有一个性质：若通过拷贝构造函数或者赋值操作符来复制它们，它们本身会变成null，复制所得的指针取得资源的唯一拥有权！

3、由于某些类需要有正常的复制行为，所以auto_ptr有时也行不通，此时shared_ptr登场，shared_ptr是“引用计数型智慧指针(RCSP)”，也是个智能指针，持续追踪有多少对象指向某个资源，在无人指向这个资源的时候自动删除该资源

```c++
void f() {
    shared_ptr<Investment> p1(createInvestment());
    shared_ptr<Investment> p2(p1); //拷贝构造函数构造p2，p2和p1指向一个对象
    p1 = p2; //也是一样，没有任何改变
}
//函数f结束，p1和p2被销毁，其指向的对象也被销毁
```

4、auto_ptr和shared_ptr两者都在其析构函数内做delete而不是delete[]，意味着在动态分配而得的array身上使用auto_ptr或者shared_ptr是个馊主意

## 条款14：在资源管理类中小心copying行为

1、并非所有的资源都是基于堆的(heap-based)，所以对这类资源auto_ptr和shared_ptr往往不适合作为资源掌管者(因为他们是delete释放 释放的是堆中的资源)。所以有时候你需要建立自己的资源管理的RAII类，举例如下：

```c++
void lock(Mutex* pm); //锁定pm所指的互斥器
void unlock(Mutex* pm); //将互斥器解除锁定
class Lock {
public:
    //加explicit关键字意味着必须显式调用该构造函数来创建对象，不能将其他数据隐式转换成类的对象。比如MyClass obj2(7);是对的.而MyClass obj1 = 5;就是错的
    explicit Lock(Mutex* pm):mutexPtr(pm) {lock(mutexPtr);} // 获得资源
    ~Lock() {
        unlock(mutexPtr); //释放资源
    }
private:
	Mutex *mutexPtr;
};
```

客户对Lock的用法符合RAII(资源取得时机便是初始化时机)方式:

```c++
Mutex m;
{
    Lock m1(&m); //锁定m
    // 操作
} //在区块最末尾，会自动解除锁定
```

如果Lock对象被复制：

```c++
Lock m1(&m); //锁定m
Lock m2(m1); //将m1复制到m2身上(这里传的不是Mutex*，所以不是锁定，传的是Lock对象 是拷贝构造)
```

有两种可能选择：
1️⃣禁止复制
2️⃣对底层资源祭出"引用计数法"
解释下2️⃣，有时候我们希望保有资源直到最后一个指向它的对象被销毁，在这种情况下复制RAII对象的时候，应该将该资源的“被引用数”递增，shared_ptr就是如此。
通常RAII类只要内含一个shared_ptr成员变量即可满足2️⃣。对Mutex*进行修改，如下:

```c++
class Lock {
public:
    explicit Lock(Mutex* pm):mutexPtr(pm,unlock) {lock(mutexPtr.get());} // 获得资源
private:
	shared_ptr<Mutex> mutexPtr;//修改成shared_ptr指针之后就不需要再写析构函数了 由于shared_ptr中的析构函数会在mutexPtr引用次数为0的时候调用删除器，这个删除器默认是delete指针，但是可以自己指定，此时需要的是unlock指针，所以要在上面的构造函数中添加第二个参数unlcok，表示以unlock函数为删除器，而不是默认的delete
};
```

2、1️⃣复制RAII对象必须一起复制它所管理的资源，所以资源的copying行为决定RAII对象的copying行为。
2️⃣普遍而常见的RAII类的复制行为是实行引用计数法，抑制copying。

## 条款15：在资源管理类中提供对原始资源的访问

1、由于许多接口(API)直接指涉资源，所以绕过资源管理对象直接访问原始资源是很难避免的。

2、将RAII类对象转换为其所含的原始对象:分为显示转换和隐式转换两种做法

```c++
shared_ptr<Investment> p(createInvestment());

int daysHeld(const Investment* p) {} 
//调用函数
int days = daysHeld(p); //报错！因为p是shared_ptr对象，而函数需要的是Investment* 指针！

//显示转换 
//利用get函数 shared_ptr和auto_ptr都提供了这个函数
int days = daysHeld(p.get()); //将p内的原始指针传给daysHeld,这个就对了

//隐式转换
//->和*可以隐式地将指针转换成原始底部指针
p->isTaxFree();
(*p).isTaxFree(); //二者都可通过shared_ptr p来访问管理的资源
```

3、对于自己设计的资源管理类(RAII类)，以Font类为例：

```c++
FontHandle getFont(); //获取资源
void releaseFont(FontHandle fh); //释放资源

class Font {
public:
    explicit Font(FontHandle fh):f(fh) {}
    ~Font() {releaseFont(f);}
private:
    FontHandle f; //原始(raw)字体资源，即我们使用自己定义的资源管理类Font管理原始字体资源FontHandle
};
```

将Font类转化成FontHandle类会是一种很频繁的需求，同样有两种转换方式：显示转换和隐式转换
1️⃣显示转换

```c++
//使用get函数
class Font {
public:
    //略
    FontHandle get() const {return f;} //显示转换函数
	//略
};
//缺点是客户每当想要使用api的时候就要调用get函数
void function(FontHandle f) {}
Font f(getFont());
function(f.get()); //调用get取得Font类中的FontHandle
```

2️⃣隐式转换

```c++
class Font {
public:
    //略
    operator FontHandle() const {return f;} //隐式转换函数
    //略
};

void function(FontHandle f) {}
Font f(getFont());
function(f); //直接传入Font对象f就行，编译器会自动调用隐式转换函数来将Font对象转换为FontHandle类型

//但是隐式转换也会导致类型转换的意外发生
Font f1(getFont());
FontHandle f2 = f1;
//本意是要拷贝一个Font对象，但是f1中的隐式转换函数自动被调用将f1转换成了FontHandle对象然后再复制的 发生了错误！
```

3、对原始资源的访问可能是显示转换或者隐式转换，一般而言显示转换比较安全，但隐式转换对客户比较方便。

## 条款16：成对使用new和delete时要采取相同形式

1、如果你调用new时使用[]，必须在调用delete时也使用[];
如果你调用new时没使用[]，在调用delete时也不该使用[].

```c++
string* p1 = new string;
string* p2 = new string[10];

delete p1;
delete [] p2;//加[]是告诉delete，指针指向一个数组，不加[]的话可能不会调用足够的析构函数去销毁对象
```

2、尽量不要给数组名做typedef操作，很容易造成new和delete不匹配

```C++
typedef string Address[4]; //Address就是string[4]的别名
string* p = new Address;
delete p; //错误 以为Address是string[4]的别名
delete [] p; //正确
```

## 条款17：以独立语句将new的对象置入智能指针

1、new创建对象然后放入智能指针的这个过程，应该与其他过程独立开来，避免混到一起导致new创建了对象但是没有成功放入智能指针中导致资源泄露:

```c++
int priority() {}
void processWidget(shared_ptr<Widget> pw,int priority) {}

processWidget(shared_ptr<Widget>(new Widget),priority()); 
//此时编译器会做三件事:1.调用priority 2.执行new Widget 3.调用shared_ptr构造函数 编译器执行顺序不确定，只能确定new Widget一定在shared_ptr构造函数之前，因为前者是后者的参数，所以很有可能的顺序是：1.执行new Widget 2.调用priority 3.调用shared_ptr构造函数 当priority的调用出现异常的话，3就不会被执行，1中new的资源没能得到及时释放 导致资源泄露
```

解决方法也很简单，将语句分离开来即可：

```c++
shared_ptr<Widget> pw(new Widget);
processWidget(pw,priority()); //此时就不会出现执行顺序的问题了
```

# 4、设计与声明

## 条款18、让接口容易被正确使用，不易被误用

1、cross-DLL problem:对象在动态连接程序库(DLL)中被new创建，却在另一个DLL内被delete销毁。
shared_ptr有一个特别好的性质：它会自动使用它的“每个指针专属的删除器deleter”，从而消除cross-DLL problem.

2、"阻止误用"的办法包括：建立新类型，限制类型上的操作，束缚对象值，以及消除客户的资源管理责任

3、shared_ptr支持定制型删除器，这可以预防DLL问题，可以被用来自动解除互斥锁等等~

## 条款19：设计class犹如设计type

1、class的设计就是type的设计，在定义一个新type之前，请确定已经考虑过：新type的对象如何被创建和销毁？什么样的操作符和函数对此新type而言是合理的？你定义的新type有什么作用？等等问题

## 条款20：宁以pass-by-reference-to-const替换pass-by-value

1、缺省情况下C++以by value方式传递对象至函数，函数参数都是以实际参数的副本为初值，调用端获得的也是函数返回值的一个副本。这些副本都是由对象的copy构造函数产出。

2、以by value方式传递参数容易有很多次构造函数和析构函数被调用，比如：

```c++
class Person {
public:
    Person();
    virtual ~Person();
private:
    string name;
    string address;
};

class Student:public Person { //Person的子类
public:
    Student();
    ~Student();
private:
    string schoolName;
    string schoolAddress;
};

bool validateStudent(Student s){}
Student plato;
bool platoIsok = validateStudent(plato);
```

以by value的方式将student对象传入函数的时候，需要进行的操作有：Person构造函数，Student构造函数，四个string的构造函数，以及前面六个构造函数对应的六个析构函数，总成本是：六构造，六析构！

解决方法是pass by reference-to-const(以const引用方式传递):
bool validateStudent(const Student& s);
由于没有任何新对象被创建，所以没有任何构造函数或析构函数被调用。声明const防止函数修改原参数。

3、通过引用方式传参还可以解决切割问题(slicing):(一般是切除子类中的特化信息)

```c++
class Windows {
public:
    virtual void display() const;
};
class WindowsWithScrollBars:public Window {
public:
    virtual void display() const;
};

void Display(Window w) {
    w.display();
}
WindowsWithScrollBars wwsb;
Display(wwsb); //此时由于是pass by value，参数w会被构造成一个Windows对象，导致子类WindowsWithScrollBars所有的特化信息都被删除，因此无论传来的是Windows的什么子类，都是执行Windows::display();

//解决方法：pass by reference-to-const
void Display(const Windows& w) {
    w.display(); //此时就是传的什么子类 就是哪个子类的display函数被执行
}
```

4、对于内置类型(int,double等)以及STL的迭代器和函数对象来说，pass by value往往比较合适。

## 条款21：必须返回对象时，别妄想返回其reference(引用)

1、绝对不要返回指针或者引用指向一个local stack对象，或者返回引用指向一个堆空间分配的对象。例如：

```c++
const Rational& operator*(const Rational& lhs,const Rational& rhs) {
    Rational result(lhs.n*rhs.n,lhs.d*rhs.d);
    return result; //这是不对的 因为构造函数构造的result是个local对象，存储在栈里，随着函数结束会被销毁，那返回一个引用指向该位置就没有意义了
}

const Rational& operator*(const Rational& lhs,const Rational& rhs) {
    Rational* result = new Rational(lhs.n*rhs.n,lhs.d*rhs.d); //在堆中开辟result空间 虽然解决了上述问题，但是付出了构造函数调用的代价，以及，谁该对着你new的对象实施delete？可能出现资源泄露问题！
    return result; 
}

//综上可以看出都是返回值是reference捣的鬼
```

## 条款22：将成员变量声明为private

1、切记将成员变量声明为private，这可赋予客户访问数据的一致性、可细微划分访问控制、允诺约束条件获得保证，并提供class作者以充分的实现弹性。

2、protected并不比public更具封装性，从封装的角度看，其实只有两种访问权限：private(提供封装)和其他(不提供封装)。

## 条款23：宁以non-member、non-friend替换member函数

1、使用非成员函数和非友元函数来代替成员函数可以增加封装性、包裹弹性和机能扩充性。因为member和friend函数都可以访问private中的成员变量，会导致封装性降低，更容易与外界产生关联，因此我们就越不好去修改它。

## 条款24：若所有参数皆需类型转换，请为此采用non-member函数

1、若参数需要类型转换，使用member函数可能会产生错误：

```c++
class Rational {
public:
    const Rational operator* (const Rational& rhs) const;
};

Rational a(1,2); //构造函数 1是分子，2是分母
Rational b(1,8);
Ratioanl result = a*b; //正确
result = a * 2; //正确 可以写成a.operator*(2);这里应该传入的是Rational对象，但是传入了2，发生了隐式类型转换，编译器会自动调用Rational构造函数并赋予你提供的2，于是便产生了一个临时的Rational对象。当然如果构造函数前加了explicit则明确表示不允许进行隐式类型转换 此时这个编译也不通过了
result = 2 * a; //错误 可以写成2.operator*(a);只有当参数在参数列只能才能参与隐式转换，这个2不在参数列表中
```

使用non-member函数就可以避免这个问题：

```c++
const Rational operator*(const Rational& lhs,const Rational& rhs) {}
result = 2 * a; //可以写成operator*(2,a)
result = a * 2; //可以写成operator*(a,2)
//两个式子中2都在参数列表中 可以进行隐式转换成Rational对象
```

2、与此同时，operator*也不适合成为Rational类的一个友元函数。无论何时，friend函数可以避免就要避免，就像真实世界一样，朋友带来的麻烦往往多过其价值。

## 条款25：考虑写出一个不抛异常的swap函数

1、如果你提供一个member  swap函数，也该提供一个non-member swap来调用前者。

# 5、实现

## 条款26：尽可能延后变量定义式的出现时间

1、只要你定义了一个变量而其类型带有一个构造函数或析构函数，那么当程序的控制流到达这个变量定义式时，你便得成受构造成本；当这个变量离开其作用域时，你便得承受析构成本，即使这个变量最终没有被使用过。举例如下：

```c++
string encryptPassWord(const string& password) {
    if(password.length() < MinNum) {
        throw logic_error("Password is too short!");
    }
    string encrypted; //这个定义应该尽可能晚 如果放在if的上面，很有可能因为if中抛出了异常导致虽然没有用到encrypted，但是仍然付出了encrypted的构造和析构成本。
    //对encrypted进行操作
    return encrypted; 
}
```

2、"通过默认构造函数构造出一个对象，然后再对它赋初值"  比  "直接在构造时指定初值"效率差。所以我们可以将对对象的定义延后到能给它初始值为止，此时再使用第二种构造方式，高效得很。

3、在for循环中不同定义时机对性能的影响比较：

```c++
//第一种
Widget w;
for(int i = 0; i < n; ++i) {
    w = i;
} //1个构造函数+1个析构函数+n个赋值操作

//第二种
for(int i = 0; i < n; ++i) {
    Widget w(i);
} //n个构造函数+n个析构函数

//第一种大体上比较高效，但是不排除有的type的赋值操作花费很高，这时候第二种稍好点~
```

## 条款27：尽量少做转型动作

1、C++中的转型(casts)相对于C，Java，C#更具危险性。转型的语法如下：

```c++
(T)a; //将a转型为类型T，C语言风格
T(a); //将a转型成类型T，函数风格
//以上两种转型方式为“旧式转型”
//四种新式转型
const_cast<T>(a); //常被用来将对象的常量性转除
dynamic_cast<T>(a); //主要用来执行“安全向下转型”，即用来决定某对象是否归属继承体系中的某个类型
reinterpret_cast<T>(a); //意图执行低级转型，例如将int*转型成int,很少见
static_cast<T>(a); //用来强迫隐式转换，例如可以将non-const转为const，将int转为double，将pointer-to-base转为pointer-to-derived；但是它无法将const转为non-const，这个只有const_cast才办得到。
```

新式转型用的较多。本书作者唯一使用旧式转型的时机如下：

```c++
class Widget {
public:
    explicit Widget(int size);
};
void doSomeWork(const Widget& w);
doSomeWork(Widget(15)); //由于Widget构造函数前面有关键词explicit，所以无法进行隐式转换，只能手动转换。

doSomeWork(static_cast<Widget>(15)); //当然用新式转型也是ok的
```

2、单一对象可能拥有一个以上的地址，比如“以pointer-to-Base指向他的地址”和“以pointer-to-Derived指向它的地址”

```c++
class Base {};
class Dervied: public Base {};
Derived d;
Base* pb = &d; //隐式将Derived*转换成Base*，但是有时候上述两个指针值并不相同，这时会有一个偏移量在运行期间施于Derived*指针上，用以取得正确的Base*指针值。
```

这种事情在C,Java,C#中是不可能发生的，但是C++可能。实际上，一旦使用多重继承，这事几乎一直发生着，单一继承也可能发生。

3、1️⃣如果可以，尽量避免转型，特别是在注重效率的代码中避免dynamic_cast
2️⃣如果转型是必要的，试着将它隐藏在某个函数背后，客户可以随时调用该函数，而不需要将转型放到他们自己的代码中。

## 条款28：避免返回handles指向对象内部成分

1、handles包括引用(&)，指针，迭代器等

2、你不应该令成员函数返回一个指向private成员的handles，举例如下：
表示矩形(rectangle)类的时候，矩形可以使用左上角和右下角两个点来表示。可以用一个辅助的struct存放矩形的点信息，再用Rectangle类去指向它：

```c++
class Point { //表示点的类
public:
    Point(int x,int y);
    void setX(int newVal);
    void setY(int newVal);
};

struct RectData { //存放矩形的点信息
    Point ulhc; //左上角
    Point lrhc; //右下角
};

class Rectangle { //矩形类
public:
    Point& upperLeft() const {
        return pData->ulhc; //返回左上角的点
    }
    Point& lowerRight() const {
        return pData->lrhc; //返回右下角的点
    }
private:
    shared_ptr<RectData> pData;
};
```

这样声明可以通过编译，但是是错的，因为upperLeft和lowerRight两个成员函数虽然都加了const，但是返回值是对private内容的引用，导致调用者可以随意修改。返回值是迭代器和指针时也是同理。

也是可以解决的，就是在前面再加一个const：

```c++
class Rectangle { //矩形类
public:
    const Point& upperLeft() const {
        return pData->ulhc; //返回左上角的点
    }
    const Point& lowerRight() const {
        return pData->lrhc; //返回右下角的点
    }
private:
    shared_ptr<RectData> pData;
};
```

这样返回的引用也是不可修改的了，但是还是返回了指向内部的handle，有时handle会比其所指的对象更加长寿，有风险！

## 条款29：为“异常安全”而努力是值得的

1、当异常被抛出时，带有异常安全性的函数会：
1️⃣不泄露任何资源
2️⃣不允许数据败坏

```c++
void PrettyMenu::changeBackground(istream& imgSrc) {
    lock(&mutex);
    delete bgImage;
    ++imageChanges;
    bgImage = new Image(imgSrc);
    unlock(&mutex);
}
```

显然上述函数就不带有异常安全性，因为一旦new Image异常 unlock就不会执行，导致资源泄露，且bgImage就会指向一个已经被删除的对象，导致数据败坏。

2、异常安全函数提供以下三个保证其中一种：
1️⃣基本承诺：如果异常被抛出，程序内的任何事物仍然保持在有效状态下
2️⃣强烈保证：如果异常被抛出，程序状态不改变
3️⃣不抛掷保证：承诺绝不抛出异常。

If possible，尽可能提供不抛掷保证，但是大部分函数的保证往往落在基本保证和强烈保证之间

强烈保证往往能够以copy-and-swap实现出来，但是强烈保证并非对所有函数都可实现或具备现实意义。因为copy-and-swap的关键在于“修改对象数据的副本，然后在一个不抛异常的函数中将修改后的数据和原件互换”，因此必须为每一个即将被改动的对象生成一个副本，比较费时费空间。

3、函数提供的“异常安全保证”通常只等于其所调用的各个函数的“异常安全保证”中的最弱者，比如：

```c++
void someFunc() {
    //略
    f1();
    f2();
    //略
}
//函数someFunc的异常安全性等于f1和f2中安全性较弱的那个
```

## 条款30：透彻了解inlining的里里外外

1、inline虽然可以免除函数调用成本，但是过度热衷inline会造成程序体积太大，即使拥有虚内存。要将大多数inline限制在小型、被频繁调用的函数身上。

2、inline函数通常一定被置于头文件内，因为大多数建置环境在编译过程中进行inline，而为了将一个“函数调用”替换为“被调用函数的本体”，编译器必须知道那个函数长什么样子。

3、template通常也被置于头文件内，因为它一旦被使用，编译器为了将它具现化，需要知道它长什么样子。

4、对所有的virtual函数(除了最平淡无奇的)调用也会使inline落空，因为virtual意味着“等待，直到运行期才确定调用哪个函数”，而inline意味着“执行前，先将调用动作替换为被调用函数的本体”。

5、inline函数的调用有可能被inlined，也有可能不被inlined，例如：编译器通常不对“通过函数指针而进行的调用”实施inline

```c++
inline void f() {}
void (*pf)() = f; //函数指针pf指向f

f(); //此调用会被inlined
pf(); //此调用或许不会被inlined
```

即使你从未使用过函数指针，“未被成功inlined”的inline函数还是有可能缠住你，因为程序员并非唯一要求函数指针的人。

6、构造函数和析构函数不适合使用inline

7、大部分调试器面对inline函数都束手无策，因为无法在一个并不存在的函数内设置断点。

8、派生类中对基类函数的调用是inline调用，这是因为函数调用在编译时被静态解析并进行内联展开，而不是在运行时进行动态绑定。

## 条款31：将文件间的编译依存关系降至最低

1、连串编译依存关系会对许多项目造成难以形容的灾难：

```c++
#include<string>
#include "date.h"
#include "address.h"
class Person {
public:
    Perosn(const string& name,const Date& birthday,const Address& addr);
    string name() const;
    string birthDate() const;
    string address() const;
private:
    string theName;
    Date theBirthdate;
    Address theAddress;
}
```

如果上述头文件中有任何一个被改变，或者这些头文件所依赖的其他头文件被改变，那么每一个含入Person类的文件就得重新编译，任何使用Person类的文件也必须重新编译！

2、1️⃣如果使用对象引用或者对象指针就能完成任务，尽量不要使用对象本身。
2️⃣如果能够，尽量以class声明式替换class定义式；当你声明一个函数而它用到某个class时，你并不需要该class的定义，纵使函数以by value的方式传递该类型的参数：

```c++
class Date; //Date类的声明
Date today();
void clearAppointments(Date d); //ok的，这里并不需要Date类的定义式
```

3️⃣为声明式和定义式提供不同的头文件

3、C++提供关键字export，允许将template声明式和template定义式分割于不同的文件内。

4、支持“编译依存性最小化”的一般构想是：相依于声明式，不要相依于定义式。基于此构想的两个手段是Handle classes和Interface classes。
1️⃣Handle Classes：通过指针或引用封装实现，减少对实现类的依赖。

```c++
class Implementation;

class Handle {
public:
    Handle();
    ~Handle();

    void doSomething();

private:
    Implementation* impl;
};
//在这里，Handle类对Implementation类的依赖被限制在一个指针上，因此用户只需知道Handle的接口即可。
```

2️⃣Interface Classes：类中定义纯虚函数，该类即为抽象基类(接口类)，其他类具体的实现通过继承该接口类来实现。

```c++
class Interface {
public:
    virtual ~Interface() {}
    virtual void doSomething() = 0; //存在纯虚函数 所以类是抽象基类
};

class ConcreteImplementation : public Interface {
public:
    void doSomething() override {
        // Specific implementation
    }
};
//用户只需依赖Interface类，而具体的实现可以在不同的类中变化
```

5、程序库头文件应该以“完全且仅有声明式”的形式存在，这意味着头文件只应包含函数、类、模板的声明，而非定义。这种做法不论是否涉及template都适用。好处如下：
1️⃣仅声明可以减少对实现细节的依赖，改变实现时无需重新编译依赖此头文件的所有代码。
2️⃣将实现隐藏在源文件中，增强封装性和安全性，防止用户直接访问内部实现。
3️⃣通过只暴露接口，避免不必要的接口变化对用户代码造成影响。

# 6、继承与面向对象设计

解释下is-a:表示一个类是另一个类的一种类型或者子类。比如She  is  a girl.中间的is-a就体现出来了

## 条款32：确定你的public继承塑模出is-a关系

1、设Student类通过public继承Person类，任何函数如果期望获得一个类型为Person的实参，都也愿意接受一个Student对象，但是反过来则不行。不过这个论点只对public继承才成立，private继承的意义与此完全不同。

2、assert是一个宏，用于在程序调试中添加断言，当条件为假时，assert宏将终止程序的执行，并输出一条出错信息。assert宏通常用于在代码中检查某些假设是否为真，如果假设不成立，就会触发断言失败。

```c++
#include <cassert>

int main() {
    int x = 10;
    assert(x == 5); // 如果 x 不等于 5，程序将终止执行并输出错误信息

    return 0;
}
```

需要注意的是，assert 通常在调试时启用，在发布版本中可能会被禁用。

3、“public继承”就意味着is-a，适用于基类身上的每一件事情也一定要适用于派生类身上，因为每一个派生类对象也都是一个基类对象。所以public继承中不要出现以下行为：

```c++
class Square:public Rectangle {}; //正方形类public继承矩形类
Square s;

assert(s.width() == s.height()); //由于是正方形 所以为真
makeBigger(s); //height+10，width不变
assert(s.width() == s.height()); //对所有正方形应该为真，但是函数makeBigger导致长宽不等了
//即makeBigger适用于基类矩形，但不适用于派生类正方形。
```

在设计pubic继承的时候，一定要注意！

## 条款33：不要遮掩继承而来的名称

1、局部同名变量会遮掩全局同名变量：

```c++
int x;
void func() {
    double x;
    cin >> x;
}
//在func内的x是local变量，int x是global变量，本例中名为x的double变量就把名为x的int变量遮掩了，所以起名还要慎重
```

2、关于继承，派生类的作用域被嵌套在了基类的作用域内。在类继承中也存在同名遮掩的问题：

```c++
class Base {
private:
    int x;
public:
    virtual void mf1() = 0;
    virtual void mf1(int);
    virtual void mf2();
    void mf3();
    void mf3(double);
};

class Derived:public Base {
public:
    virtual void mf1();
    void mf3();
    void mf4();
};
```

这段代码中，以作用域为基础的“名称遮掩规则”并没有改变，因此基类内所有名为mf1和mf3的函数都被派生类内的mf1和mf3函数遮掩掉了。于是有：

```c++
Derived d;
int x;
d.mf1(); //正确 调用了Derived::mf1
d.mf1(x); //错误 因为Derived::mf1遮掩了Base::mf1
d.mf2(); //正确
d.mf3(); //正确
d.mf3(x); //错误 因为Derived::mf3遮掩了Base::mf3
```

解决方法是使用using关键字：

```c++
class Derived:public Base {
public:
    //让基类内名为mf1和mf3的所有东西在派生类作用域内都可见
    using Base::mf1;
    using Base::mf3;
    virtual void mf1();
    void mf3();
    void mf4();
};

d.mf1(x); 
d.mf3(x);
// 此时这两个函数调用便都是正确的了
```

上述例子意味着，如果你继承基类并加上重载函数，而你又希望定义或重写其中一部分，那么你必须为那些原本会被遮掩的每个名称引入一个using声明式，否则会被遮掩！

3、如果有些老旧编译器不让使用using关键字，可以使用“inline转交函数法”：

```c++
class Base {
public:
    virtual void mf1() = 0;
    virtual void mf1(int);
};
class Derived:private Base {
public:
    virtual void mf1() { //转交函数
        Base::mf1(); //暗自成为inline，省去了函数调用的开销
    }
};
```

## 条款34：区分接口继承和实现继承

1、public继承概念分为：函数接口继承和函数实现继承。

2、成员函数的接口总是会被继承。

3、在类中声明纯虚函数(pure virtual)的目的是为了让派生类只继承函数接口。
比如圆形和矩形都继承与图形shape类，图形类中的Draw函数就是纯虚函数，每个子类如何Draw需要每个子类自己单独实现，基类中无法给出合理的默认实现。

但是出人意料的是，我们也可以为基类shape中的Draw函数提供一份代码，C++不会发出怨言，但是调用其的唯一方式是通过类名，即shape::Draw()

4、在类中声明非纯虚函数(impure virtual)的目的是让派生类中继承该函数的接口和缺省实现。

5、在类中声明非虚函数(non-virtual)的目的是为了令派生类继承函数的接口以及一份强制性实现。

## 条款35：考虑virtual函数以外的其他选择

1、NVI(non-virtual interface)手法：通过public非虚成员函数间接调用private虚函数。

```c++
//正常情况下：
class GameCharacter {
public:
    virtual int healthValue() const;
    
}; //此时派生类会重构healthValue函数，表示不同的角色健康值的计算方式不同；但是基类也会有一个缺省算法，因为不是纯虚函数

//NVI手法
class GameCharacter {
public:
    int healthValue() const { //这个non-virtual函数被称为virtual函数的外覆器        
        int retVal = doHealthValue();
        return retVal;
    }
private:
    virtual int doHealthValue() const {
        //缺省算法
    }
};
//在NVI手法下virtual也未必一定是private，有时候子类一定要调用，就可能是protected，或者是具备多态性质的基类的析构函数，就得是public
```

2、Function Pointer手法实现Strategy模式：可以使构造函数接受一个指针，指向一个函数：

```c++
class GameCharacter; //前置声明类
int defaultHealthCalc(const GameCharacter& gc); //缺省算法

class GameCharacter {
public:
    typedef int(*HealthCalcFunc)(const GameCharacter&);
    //构造函数
    //explicit 关键字用于防止编译器进行隐式类型转换，只允许显式调用构造函数
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc): healthFunc(hcf) {}
    
    int healthValue() const {
        return healthFunc(*this); //你构造时传的是啥函数指针，这里就用哪种函数方法去算
    }
private:
    HealthCalcFunc healthFunc;
};
```

这种做法提供了某些弹性：
1️⃣同一人物类型的不同实体可以有不同的健康计算函数，例如：

```c++
class EvilBadGuy:public GameCharacter {
public:
    explicit EvilBadGuy(HealthCalcFunc hcf = defaultHealthCalc):GameCharacter(hcf) {}
};
//定义两种健康指数计算函数
int loseHealthQuickly(const GameCharacter&);
int loseHealthSlowly(const GameCharacter&);

//同一人物类型的不同实体搭配不同的健康计算方式
EvilBadGuy ebg1(loseHealthQuickly);
EvilBadGuy ebg2(loseHealthSlowly);
```

2️⃣某个已知人物的健康指数计算函数可以在运行期变更。例如，GameCharacter类可以提供一个成员函数setHealthCalculator，用来替换当前的健康指数计算函数。

这些计算函数不再是GameCharacter继承体系内的成员函数，他们并未访问即将被计算的那个对象的非public成分，如果需要非public信息进行精确计算，就有问题了。

3、tr1:function是 C++ 技术报告1（TR1）中引入的函数对象包装器，用于将函数或函数对象包装成对象，类似于一个函数指针。tr1::function可以存储、复制、调用任何可调用对象（函数、函数指针、函数对象等），并且提供了函数调用的方式。举例如下：

```c++
#include <iostream>
#include <tr1/functional>
using namespace std;
int add(int a, int b) {
    return a + b;
}
int main() {
    //签名式<int(int,int)>表示该对象可以存储接受两个int参数并返回int类型的函数
    tr1::function<int(int, int)> func = add;
    cout << func(3, 4) << endl;  // 输出 7
    return 0;
}
```

4、用tr1::function完成Strategy模式

```c++
class GameCharacter; //前置声明类
int defaultHealthCalc(const GameCharacter& gc); //缺省算法

class GameCharacter {
public:
    typedef tr1::function<int (const GameCharacter&)> HealthCalcFunc;
    //构造函数
    //explicit 关键字用于防止编译器进行隐式类型转换，只允许显式调用构造函数
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc): healthFunc(hcf) {}
    
    int healthValue() const {
        return healthFunc(*this); //你构造时传的是啥函数指针，这里就用哪种函数方法去算
    }
private:
    HealthCalcFunc healthFunc;
};
```

tr1::function对象的行为就像一般函数指针，这样的对象可接纳“与给定的目标签名式兼容”的所有可调用物。

## 条款36：绝不重新定义继承而来的non-virtual函数

1、假设B是基类，D是B的派生类，B中public定义了non-virtual函数mf，在D中倘若重定义了继承而来的mf，那么就会发生“覆盖”，导致通过D指针调用mf实现的 是D中的mf函数而不是基类的mf函数。

## 条款37：绝不重新定义继承而来的缺省参数值

1、本条款成立的理由：virtual函数是动态绑定，但是缺省参数值(即virtual函数中的默认参数值)却是静态绑定。

```c++
class Shape {
public:
    enum ShapeColor {Red,Green,Blue};
    virtual void draw(ShapeColor color = Red) const = 0; //纯虚函数
};
```

2、静态类型和动态类型：
静态类型：声明的时候所采用的类型。
动态类型：目前所指对象的类型。可以在程序执行过程中发生改变。
举例如下：

```c++
class Shape {
public:
    enum ShapeColor {Red,Green,Blue};
    virtual void draw(ShapeColor color = Red) const = 0; //纯虚函数
};

class Circle:public Shape {
public:
    virtual void draw(ShapeColor color) const;
};
class Rectangle:public Shape {
public:
    virtual void draw(ShapeColor color = Green) const; //给出缺省参数值Green
};

Shape* ps;
Shape* pc = new Circle;
Shape* pr = new Rectangle;
//三者的静态类型都是Shape*
//ps没有动态类型，因为尚未指定任何对象
//pc的动态类型是Circle*
//pr的动态类型是Rectangle*
```

3、virtual函数动态绑定，就是调用一个virtual函数时，究竟调用哪一份函数实现代码，取决于发出调用的那个对象的动态类型。

```c++
pc-draw(Shape::Red); //调用Circle::draw(Shape::Red)
pr->draw(Shape::Red); //调用Rectangle::draw(Shape::Red)
```

虽然virtual函数是动态绑定，但是其内的缺省参数值却是静态绑定：

```c++
pr->draw(); //调用的是Rectangle::draw(Shape::Red)，即使Rectangle自己定义了draw的缺省参数值Green，但还是用的是基类Shape提供的缺省参数值Red，那是因为pr的静态类型还是Shape*
```

上述例子即使把指针(*)换成引用(&)，问题依然会存在

4、上述的解决方法就是NVI手法替代virtual函数：

```c++
class Shape {
public:
    enum ShapeColor {Red,Green,Blue};
    //non-virutal函数
    void draw(ShapeColor color = Red) const {
        doDraw(color);
    }
private:
    virtual void doDraw(ShapeColor color) const = 0; //真正的工作在此处完成
};

class Rectangle:public Shape {
public:
    //略
private:
    virtual void doDraw(ShapeColor color) const; //不必指定缺省参数值
};
```

由于non-virtual函数绝对不应该被derived classes重写，所以这个设计很清楚的使得draw函数的color缺省参数值总是为Red。

## 条款38：通过复合塑模出has-a或"根据某物实现出"

1、在C++中,"has-a"关系指的是一个类包含另一个类作为其成员变量。例如Car类包含Engine类，就可以说"Car has a Engine"

2、在C++中,"is-implemented-in-terms-of(根据某物实现出)",意味着一个类（或者函数）的实现依赖于另一个类（或者函数）的实现.举个例子，考虑一个Stack类，它想要实现一个后进先出（LIFO）的数据结构。这时，Stack类可能会使用一个LinkedList类来存储数据，而不是在Stack类中直接实现数据结构。在这种情况下，我们可以说Stack类是使用LinkedList类来实现的，也就是说Stack类是在LinkedList类的基础上实现的。

3、复合是类型关系的一种，当某种类型的对象内含其他类型的对象，便是这种关系。复合(composition)的意义和public继承完全不同。

## 条款39：明智而审慎地使用private继承

1、private继承并不意味着“is-a”关系，即编译器不会自动将一个派生类对象转换为一个基类对象。举例如下：

``` c++
class Person {};
class Student:private Person {};
void eat(const Person& p);
void study(const Student& s);

Person p;
Student s;
eat(p); //正确
eat(s); //报错 因为是private继承 不是is-a关系了
```

2、private继承意味着implemented-in-terms-of(根据某物实现出)，即：如果D以private形式继承B。意思是D对象根据B对象实现而得。

3、条款38中的“复合”意义也是“根据某物实现出”，在二者之间，尽量使用“复合”，但是当派生类需要访问基类的protected成员或需要重新定义继承而来的virtual函数时，使用private继承更合理。

4、假如我们想要记录Widget类中成员函数被调用的次数，可以使用已有的Timer类中的成员函数来记录，用Widget来继承Timer类，但是不能是public继承，因为Timer not is-a Widget，不满足is-a关系，用private继承更为合适：

```c++
class Timer {
public:
    explicit Timer(int tickFrequency);
    virtual void onTick() const;
};

class Widget:private Timer {
private:
    virtual void onTick() const;
};
```

## 条款40：明智而审慎地使用多重继承

1、多重继承意味着继承一个以上的类，这些类通常没有父类了，否则会导致要命的“钻石型(菱形)多重继承”。

2、virtual继承继承是一种特殊的继承方式，用于解决菱形继承问题（diamond inheritance problem）。当一个类通过 virtual 继承另外一个类时，它将共享其基类的子对象，而不是创建一个新的子对象。这意味着如果多个子类从同一个基类进行virtual继承，它们将共享这个基类的实例，从而避免了多次继承同一个类所带来的问题。示例如下：

```c++
class Animal {
public:
    virtual void makeSound() {
        std::cout << "Animal makes a sound" << std::endl;
    }
};
class Dog : virtual public Animal {
public:
    void makeSound() override {
        std::cout << "Dog barks" << std::endl;
    }
};
class Cat : virtual public Animal {
public:
    void makeSound() override {
        std::cout << "Cat meows" << std::endl;
    }
};
class CatDog : public Dog, public Cat {
};
```

在上面的示例中，Dog和Cat类都通过virtual继承自Animal类。当CatDog类继承自Dog和Cat类时，由于Animal类被虚继承，CatDog 类将只包含一个Animal类的实例，避免了菱形继承问题。

平常还是尽量使用non-virtual继承，因为编译器需要提供若干“幕后戏法”来避免继承得来的成员变量重复，且实用virtual继承的那些类所产生的对象往往比使用non-virtual继承的体积大，访问virtual base class的成员变量时速度也更慢。但是如果virtual base class不带任何数据，virtual继承还是很有用的。

3、在C++中，多重继承可以用于实现一些特定的设计模式或解决特定的问题。其中一种情况涉及将“public 继承某个 Interface class”  和  “private 继承某个协助实现的 class”结合起来的方式。示例如下：

```c++
// 接口类
class Interface {
public:
    virtual void performAction() = 0;
};

// 协助实现的类
class Helper {
public:
    void help() {
        std::cout << "Helper is helping..." << std::endl;
    }
};

// 多重继承
class MyClass : public Interface, private Helper {
public:
    void performAction() override {
        std::cout << "MyClass is performing an action." << std::endl;
        help();
    }
};
```

1️⃣MyClass通过public继承Interface类，意味着MyClass可以被视为实现了Interface接口，必须实现performAction方法。
2️⃣MyClass通过private继承Helper类，意味着MyClass可以使用Helper类提供的功能，但外部代码无法直接访问Helper类的成员或方法。

这种结合使用不仅允许MyClass实现某个接口，还可以利用Helper类提供的功能，同时通过private继承限制了外部对Helper类的访问，从而控制了代码的复杂性和耦合度

# 7、模板与泛型编程

泛型编程：泛型编程是一种编程范式，旨在提高代码的复用性、灵活性和类型安全性。在泛型编程中，通用算法和数据结构可以在不指定具体类型的情况下编写，从而使其可以适用于多种类型。泛型编程的主要目标是实现代码重用，减少重复编写相似功能的代码，并提高代码的可维护性。

## 条款41：了解隐式接口和编译期多态

1、编译期多态和运行期多态的区别：

1️⃣编译期多态：编译期多态是指在编译时根据变量的声明类型(静态类型)确定调用的方法。在编译期，编译器根据变量的类型来决定调用哪个方法，这种多态也称为静态多态。

2️⃣运行期多态：运行期多态是指在运行时根据对象的动态类型确定调用的方法。在运行时，根据对象的实际类型来调用相应的方法，这种多态也称为动态多态。

编译期多态考虑“哪一个重载函数应该被调用(发生在编译期)”
运行期多态考虑“哪一个virtual函数该被绑定(发生在运行期)”

2、C++中什么是接口(interface)：在 C++ 中，接口通常是通过抽象基类（Abstract Base Class）来实现的，它是一种纯虚函数的集合，没有实现任何功能，只定义了接口的方法和行为。在 C++ 中，没有像其他语言（如 Java）中专门的 interface 关键字来定义接口，而是通过抽象类和纯虚函数来实现接口的概念。举例如下：

```c++
// 抽象基类（接口）
class Shape {
public:
    virtual double area() const = 0; // 纯虚函数
};
// 具体类 Circle 继承自 Shape
class Circle : public Shape {
private:
    double radius;
public:
    Circle(double r) : radius(r) {}
    double area() const override {
        return 3.14 * radius * radius;
    }
};
```

在这个示例中，Shape 类是一个抽象基类（接口），它定义了纯虚函数 area()，需要具体类Circle来实现该方法。

3、隐式接口和显式接口的区别：

1️⃣显示接口：显式接口是一种清晰定义和声明的接口(通过抽象基类实现)，开发人员必须明确地实现这些接口，以确保符合接口定义规范。

```c++
class Shape {
public:
    virtual double area() = 0;
};
class Circle : public Shape {
private:
    double radius;
public:
    Circle(double r) : radius(r) {}
    double area() override {
        return 3.14 * radius * radius;
    }
};
class Square : public Shape {
private:
    double sideLength;
public:
    Square(double s) : sideLength(s) {}

    double area() override {
        return sideLength * sideLength;
    }
};
```

在这个示例中，Shape 类是一个显式接口，它定义了一个纯虚函数 area()，需要子类明确实现。Circle 和 Square 类都显式地实现了 Shape 接口中的 area() 函数。

2️⃣隐式接口：隐式接口是一种基于约定和惯例的接口，不需要明确的接口声明，而是通过对象或函数的行为来确定其接口。

```c++
class Dog {
public:
    void speak() {
        std::cout << "Woof! Woof!" << std::endl;
    }
};
class Cat {
public:
    void speak() {
        std::cout << "Meow! Meow!" << std::endl;
    }
};
```

在这个例子中，Dog 类和 Cat 类都具有 speak() 方法，尽管没有明确的接口声明，但它们可以被视为实现了一个隐式的“speak”接口。

4、在面向对象编程世界中，总是以显式接口(explicit interfaces)和运行期多态(runtime polymorphism)解决问题。举例如下：

```c++
class Widget {
public:
    Widget();
    virtual ~Widget();
    virtual std::size_t size() const;
    virtual void normalize();
    void swap(Widget& other);
};

void doProcessing(Widget& w) {
    if(w.size() > 10 && w != someNastyWidget) {
        Widget temp(w);
        temp.normalize();
        temp.swap(w);
    }
}
```

我们可以这样说doProcessing内的w：
1️⃣参数w的类型被明确指定为 Widget&，这是一个显式的类型声明；函数中对Widget类的对象进行了操作，这种操作是基于确切已知的类型，因此是显式接口的一种体现
2️⃣函数依赖于传入对象w的实际类型(动态类型)来确定方法的调用，所以是运行期多态。

5、模板与泛型编程的世界，与面向对象有根本上的不同，在此世界中显式接口和运行期多态仍然存在，但重要性降低；反倒是隐式接口(implicit interfaces)和编译期多态(compile-time polymorphism)移到前头了。我们将上述的doProcessing函数转变成模板函数：

```c++
template<typename T>
void doProcessing(T& w) {
    if(w.size() > 10 && w != someNastyWidget) {
        T temp(w);
        temp.normalize();
        temp.swap(w);
    }
}

```

现在我们说doProcessing内的w：
1️⃣参数w的类型是一个模板类型 T，这是一个泛型声明，函数可以接受任何类型的参数；函数中调用了 w.size()、w != someNastyWidget 这些方法，这些方法并非在泛型类型 T 中显式定义的接口，而是假定 T 类型的对象有这些方法，这是一种隐式的约定。
2️⃣模板函数会在编译时根据具体的类型生成相应的代码，而不需要在运行时进行动态绑定，所以是编译期多态。

6、请记住：
1️⃣类和模板都支持接口和多态。

2️⃣对类而言接口是显式的，以多态签名为中心；多态则是通过virtual函数发生于运行期(即显示接口+运行期多态)。

3️⃣对模板参数而言，接口是隐式的，奠基于有效表达式；多态则是通过template具现化和函数重载解析发生于编译期(即隐式接口+编译期多态)

## 条款42：了解typename的双重意义

1、template声明式中，class与typename完全相同：

```c++
template<class T>
template<typename T>
```

除此之外，template中也可以有多个参数：

```c++
template <typename T, typename U>
```

然而 扫一并不总是把 class typenarne 视为等价。有时候你一定得使用typename

2、template与函数结合：泛型函数

```c++
// 定义一个函数模板
template <typename T>
T add(T a, T b) {
    return a + b;
}
int main() {
    // 使用函数模板，传入整数类型
    int result1 = add(5, 10);
    std::cout << "Result 1: " << result1 << std::endl;
    // 使用函数模板，传入浮点数类型
    double result2 = add(3.14, 2.71);
    std::cout << "Result 2: " << result2 << std::endl;
    return 0;
}
```

3、template与类结合：泛型类

```c++
// 定义一个类模板
template <typename T>
class Pair {
private:
    T first;
    T second;
public:
    //构造函数
    Pair(T f, T s) : first(f), second(s) {}
};
int main() {
    // 实例化一个 Pair 类模板，指定 int 类型 注意用法是类名后面➕<>
    Pair<int> intPair(5, 10);
    // 实例化一个 Pair 类模板，指定 double 类型
    Pair<double> doublePair(3.14, 6.28);
    return 0;
}
```

4、 C++ 的模板中，可以使用从属名称（dependent name）和嵌套从属名称（nested dependent name）来引用模板参数中的类型或值。这些名称在模板实例化时可能会受到模板参数的影响，因此称为“从属名称”。在template中指涉(refer to)两种名称：
1️⃣从属名称：

```c++
template <typename T>
class Container {
public:
    void printSize() {
        std::cout << "Size of T: " << sizeof(T) << std::endl;
    }
};
int main() {
    Container<int> intContainer;
    intContainer.printSize();

    Container<double> doubleContainer;
    doubleContainer.printSize();

    return 0;
}
```

在printSize方法中，sizeof(T)使用了从属名称 T，它依赖于模板参数。

2️⃣嵌套从属名称：

```c++
template <typename T>
class Wrapper {
public:
    struct Nested {
        T value;
    };
};
int main() {
    Wrapper<int>::Nested nestedInt{5};
    Wrapper<string>::Nested nestedString{"Hello"};
    return 0;
}
//Wrapper 类具有一个嵌套结构 Nested，在 Nested 结构中，使用了模板参数 T。当实例化 Wrapper<int>::Nested 和 Wrapper<string>::Nested 时，T 被替换为相应的类型，使得 value 成为具体的类型
```

5、任何时候当你想要在template中指涉一个嵌套从属类型名称，就必须在紧临它的前一个位置放上关键字typename：

```c++
template<typename C>
void print2nd(const C& container) {
    C::const_iterator* x;
}//这里会有歧义，因为C的类型不确定，所以不确定C::const_iterator是个类型还是说C中有个变量名叫const_iterator，如果还有个global变量x，name这个本来是指针的定义就变成了const_iterator乘以x

//在前面加个typename关键字即可改正
template<typename C>
void print2nd(const C& container) {
    typename C::const_iterator* x;
}//此时会被成功编译
```

typename只被用来验明嵌套从属类型名称，其他名称不该有它存在。

```c++
template<typename C>
void f(const C& container,typename C::iterator iter); //f的参数列表中，第一个不允许用typename，第二个一定要用typename
```

“typename必须作为嵌套从属类型名称的前缀词”这一规则的例外是，typename不可以出现在base class list内的嵌套从属类型名称之前，也不可以在成员初始列中作为base class修饰符。举例如下：

```c++
//基类列表中不能使用typename
template <typename T>
class Derived : public T::BaseClass {  // 不能使用typename，因为在基类列表中，编译器知道你是在指定基类
public:
    Derived() {}
};

//成员初始化列表中不能使用typename
template <typename T>
class Derived : public T::BaseClass {
public:
    Derived() : T::BaseClass() {}  // 不能使用typename，因为编译器知道这是构造函数的调用
};
```

## 条款43：学习处理模板化基类内的名称

1、template修饰基类的情况下，派生类调用基类函数需谨慎：

```c++
class MsgInfo {};

template<typename Company>
class MsgSender {
public:
    void sendClear(const MsgInfo& info) {
        string msg;
        Company c;
        c.sendCleartext(msg);
    }
    void sendSecret(Const MsgInfo& info) {
        //略
    }
};

template<typename Company>
class LoggingMsgSender:public MsgSender<Company> {
    void sendClearMsg (const MsgInfo& info) {
        //传送前的msg写至log
        sendClear(info); //调用base class函数 但是代码无法编译！
        //传送后的msg写至log
    }
};
//调用基类函数sendClear(info)出错的原因是：编译器知道LoggingMsgSender类继承的是MsgSender<Company>，但是其中的company是个template参数，不到LoggingMsgSender类被具现化无法确切知道它是什么，所以无法确定其中是否有个sendClear函数
```

当我们从Object Oriented C++跨进Template C++，继承就不像以前那般畅行无阻了。

上述问题究其原因是编译器不进入base class的作用域内进行查找，有三种解决方法：
1️⃣在base class函数调用之前加上"this->":

```c++
template<typename Company>
class LoggingMsgSender:public MsgSender<Company> {
    void sendClearMsg (const MsgInfo& info) {
        //传送前的msg写至log
        this->sendClear(info);//成立，假设sendClear将被继承 
        //传送后的msg写至log
    }
};
```

2️⃣使用using声明式：

```c++
template<typename Company>
class LoggingMsgSender:public MsgSender<Company> {
    //告诉编译器，请它假设sendClear位于base class 内
    using MsgSender<Company>::senderClear;
    void sendClearMsg (const MsgInfo& info) {
        //传送前的msg写至log
        sendClear(info); 
        //传送后的msg写至log
    }
};
```

3️⃣明白指出被调用的函数位于base class内：

```c++
template<typename Company>
class LoggingMsgSender:public MsgSender<Company> {
    void sendClearMsg (const MsgInfo& info) {
        //传送前的msg写至log
        MsgSender<Company>::sendClear(info); //假设sendClaer将被继承下来
        //传送后的msg写至log
    }
};
```

3️⃣是最糟糕的方法，因为如果调用的是virtual函数，上述的明确资格修饰会关闭“virtual绑定行为”。

2、如果一般性的基类模板对于某个类不合适，可以针对这个类写一个全特化的模板：

```c++
template<typename Company>
class MsgSender {
public:
    void sendClear(const MsgInfo& info) {
        string msg;
        Company c;
        c.sendCleartext(msg);
    }
    void sendSecret(Const MsgInfo& info) {
        //略
    }
};

class CompanyZ {
public:
    void sendEncrypted(const string& msg);
}//由于CompanyZ类不提供sendCleartxt函数，所以不适合MsgSender模板，所以我们针对CompanyZ写一个特化模板:

template<>  //象征这即不是template也不是标准class
class MsgSender<CompanyZ> {
public:
    void sendSecret(const MsgInfo& info) {
        //略
    }
}; //只有在template的实参是CompanyZ的时候被使用
```

## 条款44：将与参数无关的代码抽离templates

1、模板代码膨胀：在C++中，模板代码膨胀（Template Code Bloat）是指由于模板的特性导致每个具体参数化的模板实例都会生成一份独立的代码副本，即使这些实例具有相同的二进制表述。这可能会导致编译时间增加、二进制文件体积膨胀等问题。

2、避免template内的代码重复：

```c++
template<typename T,std::size_t n> //size_t表示非类型参数
class SquareMatrix {
public:
    //略
    void invert(); //求逆矩阵
}

SquareMatrix<double,5> sm1;
SquareMatrix<double,10> sm2;
sm1.invert(); //调用SquareMatrix<double,5>::invert
sm2.invert();//调用SquareMatrix<double,10>::invert
```

SquareMatrix<double,5>和SquareMatrix<double,10>这两个类都有invert函数，而且这两个invert函数中除了常量5,10其他的部分完全相同。这是template引出代码膨胀的一个典型例子。

常见解决方案是单独定义一个template参数只有类型的类：

```c++
template<typename T>
class SquareMatrixBase {
protected:
    //略
    void invert(std::size_t matrixSize); //以给定的尺寸求逆矩阵
};

template<typename T,std::size_t n>
class SquareMatrix {
private:
    using SquareMatrixBase<T>::invert; //避免遮掩base版的invert
public:
    //略
    void invert() {
        this->invert(); //函数调用成本是0，因为是inline调用，见条款30；同时这里要用this->记号，不然模板基类的函数名会被派生类中的掩盖
    }
}
//此时SquareMatrix<double,5>和SquareMatrix<double,10>这两个类调用的invert函数都来自相同的基类SquareMatrixBase<double>
```

3、请记住：
1️⃣template生成多个类和多个函数，所以任何template代码都不该与某个造成膨胀的template参数产生相依关系。举例如下：

```c++
template<typename T>
class Example {
public:
    void doSomething() {
        // 基于 T 的复杂操作
    }
};
//每次用不同类型 T 实例化 Example 时，都会生成一个新的 doSomething()。为了减少膨胀，可以把与 T 无关的代码提取到非模板函数中。
```

2️⃣因非类型模板参数而造成的代码膨胀，往往可消除，做法是以函数参数或class成员变量替换template参数(让template少个参数)。比如上面的求逆矩阵，单独定义一个模板类，template中只有一个参数T，就不会因为template的第二个size_t参数的不同而导致代码膨胀。

3️⃣因类型参数而造成的代码膨胀，往往可降低，做法是让带有完全相同的二进制表述的具现类型共享实现码。比如在许多平台上int和long有相同的二进制表述，而template被具象化为int和long的两个版本，造成代码膨胀。

## 条款45：运用成员函数模板接受所有兼容类型

1、成员模板函数可用作类的构造函数：

```c++
template<typename T>
class SmartPtr {
public:
    explicit SmartPtr(T* realPtr);
};

SmartPtr<Top> pt1;
SmartPtr<Middle> pt2;
SmartPtr<Bottom> pt3;
//此时由于T的类型不同，我们需要的构造函数的数量也在增多，所以我们可以给它写一个模板构造函数

template<typename T>
class SmartPtr {
public:
    //泛化copy构造函数，一旦类型T和U相同，就会被具现化为“正常的”copy构造函数
    template<typename U>
    SmartPtr(const SmartPtr<U>& other);
};
//此时，对于任意类型T和类型U，这里可以根据SmartPtr<U>生成一个SmartPtr<T>,因为在SmartPtr<T>中的构造函数接受SmartPtr<U>作为参数。								
```

2、成员模板函数除了用于构造函数，常扮演的另一个角色是支持赋值操作。以智能指针tr1::shared_ptr为例

```c++
template<class T>
class shared_ptr {
public:
    //构造来自任何兼容的内置指针
    template<class Y>
    explicit shared_ptr(Y* p); 
    
    //构造shared_ptr指针
    template<class Y>
    shared_ptr(shared_ptr<Y> const& r);
    
    //构造weak_ptr指针
    template<class Y>
    explicit shared_ptr(weak_ptr<Y> const& r);
    
    //构造auto_ptr指针
    template<class Y>
    explicit shared_ptr(auto_ptr<Y> const& r);
    
    //赋值，来自任何兼容的shared_ptr<Y>和auto_ptr<Y>
    template<class Y>
    shared_ptr& operator=(shared_ptr<Y> const& r);
    template<class Y>
    shared_ptr& operator=(auto_ptr<Y>& r);
};
```

3、如果你声明member template用于“泛化copy构造函数”或“泛化assignment操作”，你还是需要声明正常的copy构造函数和copy assignment操作符。因为在class内声明泛化copy构造函数并不会阻止编译器生成他们自己的copy构造函数，所以如果你想控制copy构造的方方面面，也需要自己声明“正常的”copy构造函数。举例如下：

```c++
template<class T>
class shared_ptr {
public:
    //copy构造函数
    shared_ptr(shared_ptr const& r);
    
    //泛化copy构造函数
    template<class Y>
    shared_ptr(shared_ptr<Y> const & r);    
    
    //copy assignment
    shared_ptr& operator=(shared_ptr const& r);
    
    //泛化copy assignment
    template<class Y>
    shared_ptr& operator=(shared_ptr<Y> const& r);
};
```

## 条款46：需要类型转换时请为模板定义非成员函数

1、将条款24中的有理数Rational类和operator*函数模板化：

```c++
template<typename T>
class Rational {
public:
    Rational(const T& numerator = 0,const T& denominator = 1);
    const T numerator() const; //分子
    const T denominator() const; //分母
};

template<typename T>
const Rational<T> operator* (const Rational<T>& lhs,const Rational<T>& rhs) {
    //略
}


Rational<int> oneHalf(1,2); //初始化有理数值oneHalf为二分之一
Rational<int> result = oneHalf * 2; //错误！无法通过编译 
//编译器在编译oneHalf*2之前要先推导出T的类型，在operator*中，第一个参数是Rational<T>，传入的oneHalf的类型是Rational<int>，很容易推断出T是int，但是第二个参数也是Rational<T>，传入的2类型是int，但是由于在template实参推导过程中从不将隐式类型转换函数纳入考虑，所以无法通过编译！
```

解决方法是：在Rational类模板中声明operator*函数为友元函数

```c++
template<typename T>
class Rational {
public:
    //声明opeartor*友元函数，这里面的<T>被省略了 当然写上也是对的
    friend const Rational operator*(const Rational& lhs,const Rational& rhs); 
};

template<typename T>
const Rational<T> operator* (const Rational<T>& lhs,const Rational<T>& rhs) {
    //略
}

Rational<int> oneHalf(1,2); //初始化有理数值oneHalf为二分之一
Rational<int> result = oneHalf * 2; //此时可以通过编译了
//此时对operator*的混合式调用就可以通过编译了，因为当对象oneHalf被声明为Rational<int>时，class Rational<int>就被具现化出来了，而作为过程的一部分，friend函数operator*也会被自动声明出来，被声明出来后就是一个non-template函数，编译器可以在调用它的时候使用隐式转换函数，所以能调用成功。
```

但是上述虽然能编译成功，却无法连接成功，那是因为我们只在Rational类内提供了operator*的声明，并没有定义式，所以连接器当然找不到！最简单可行的方法就是把operator的函数本体合并到声明式中：

```c++
template<typename T>
class Rational {
public:
    friend const Rational operator*(const Rational& lhs,const Rational& rhs) {
        return Rational(lhs.numerator()*rhs.numerator(),lhs.denominator()*rhs.denominator())
    } 
};
```

此时历经万难，终于编译连接并执行了！万岁！

2、当我们编写一个class template，而它所提供的“与此template相关的” 函数支持 “所有参数的隐式类型转换”时，请将那些函数定义为“class template内部的friend函数”。

## 条款47：请使用traits classes 表现类型信息

1、STL共有五种迭代器分类：

1️⃣Input迭代器：只能向前移动，一次一步，客户只能读取(不能涂写)它们所指的东西，而且只能读取一次；C++程序库中的istream_iterator是这一分类的代表。

2️⃣Output迭代器：只能向前移动，一次一步，客户只可涂写它们所指的东西，而且只能涂写一次；ostream_iterator是这一分类的代表。

1️⃣和2️⃣只适用于“一次性操作算法”。

3️⃣forward迭代器：这种迭代器可以做前述两种分类所能做的每一件事，而且可以读或者写其所指物一次以上。所以适用于“多次性操作算法”。单向链表就属于这类迭代器。

4️⃣Bidirectional迭代器：相比于3️⃣，它除了可以向前移动，还可以向后移动。STL中的list迭代器就属于这一分类，set，multiset，map，multimap的迭代器也是。

5️⃣random access迭代器：这种迭代器比4️⃣的威力大的原因是它可以执行“迭代器算术”，也就是它可以在常量时间内向前或向后跳跃任意距离。内置指针，vector，deque和string提供的迭代器都属于这一分类。

对于这五种分类，C++标准程序库分别提供专属的tag struct加以确认：

```c++
struct input_iterator_tag {};
struct output_iterator_tag {};
struct forward_iterator_tag: public input_iterator_tag {};
struct bidirectional_iterator_tag:public forward_iterator_tag {};
struct random_access_iterator_tag:public bidirectional_iterator_tag {};
```

这些struct之间的继承关系是有效的is-a关系。

2、typedef关键字

```c++
typedef int Integer; //给一个数据类型起别名

Integer m = 10;
```

3、C++中，"traits class" 是一种用来封装类型特征（type traits）信息的技术。Type traits 是编程中一种机制，用于在编译期根据类型属性做出决策。Traits class 提供了一种将类型相关信息与操作分离的方法，从而提高代码的通用性和可维护性。Traits class 通常是一个模板类，它包含一系列公开的静态成员，这些成员用于提供有关特定类型的信息。通过使用 traits class，程序员可以在不直接检查类型的情况下，根据类型的属性执行不同的操作。

4、习惯上traits总是被实现为struct，但是往往被称为trait class. 

用来处理迭代器分类的相关信息的模板：

```c++
template<typename IterT>
struct iterator_traits; 
// iterator_traits运作方式是：针对每一个类型IterT，在struct iterator_traits<IterT>内一定声明某个typedef名为iterator_category，这个typedef用来确认IterT的迭代器分类
```

iterator_traits分两个部分实现上述所言：
1️⃣首先它要求每一个“用户自定义的迭代器类型”必须嵌套一个typedef，名为iterator_category，用来确认适当的tag struct。例如deque的迭代器可以随机访问(random access类型)，所以针对deque迭代器设计的class应该是：

```c++
template<....>
class deque {
public:
    class iterator {
    public:
        typedef random_access_iterator_tag iterator_category;
    };
};
```

类型IterT的iterator_category其实就是用来表现"IterT说它自己是什么"。

至于iterator_traits，只是鹦鹉学舌般地响应iterator class的嵌套式 typedef：

```c++
template<typename IterT>
struct iterator_traits {
    typedef typename IterT::iterator_category iterator_category;
}
```

2️⃣为了支持指针迭代器，iterator_triats特别针对指针类型提供一个偏特化版本。由于指针的行径与random access迭代器类似，所以iterator_traits为指定的迭代器类型是：

```c++
template<typename IterT>
struct iterator_traits<IterT*> {
    typedef random_access_iterator_tag iterator_category;
}
```

现在，我们可以总结出设计一个trait  class的方法了：
1️⃣确认若干你希望将来可以取得的类型相关信息。例如对迭代器而言，我们希望将来可取得其分类。
2️⃣为该信息选择一个名称(例如iterator_category)。
3️⃣提供一个template和一组特化版本(例如之前说的iterator_traits)，内含你希望支持的类型相关信息。

5、如何使用一个trait class：
1️⃣建立一组重载函数(劳工)或函数模板，彼此间的差异只在于各自的traits参数。令每个函数实现码与其接受的traits信息相应和。
2️⃣建立一组控制函数(工头)或函数模板，它调用上述那些“劳工函数”，并传递traits class所提供的信息。

6、整合重载技术后，traits classes有可能在编译期对类型执行if..else测试。举例如下：

```c++
// Traits class
template <typename T>
struct TypeChecker {
    typedef typename std::conditional<std::is_pointer<T>::value, int, double>::type ResultType;
};

// Function overloads based on traits
template <typename T>
typename TypeChecker<T>::ResultType processType(T value) {
    return value * 2;
}

template <>
int processType(int* value) {
    return *value * 2;
}
int main() {
    int num = 10;
    int* ptr = &num;
    std::cout << processType(num) << std::endl;  // Output: 20
    std::cout << processType(ptr) << std::endl;  // Output: 20
    return 0;
}
```

在上面的示例中，TypeChecker 是一个 traits class，根据传入的类型 T 是否为指针，选择返回 int 或 double。然后，processType 函数根据传入的值类型，选择合适的函数重载进行处理。

## 条款48：认识template元编程

1、在 C++ 中，元编程指的是在编译期间进行编程，通过生成、转换、或操作代码来提高程序的效率、可维护性和灵活性。C++ 元编程主要使用模板元编程技术，这种技术基于 C++ 的模板系统，允许编写在编译期间执行的代码。

2、TMP（Template Metaprogramming）是一种利用模板特化、递归等特性，在编译期执行计算和生成代码的技术。TMP有两个伟大的效力：
1️⃣让某些事情变的更容易(废话)
2️⃣由于template metaprogams执行于C++编译期，因此可将工作从运行期转移到编译期，这会导致：某些错误原本通常在运行期才能检测到，现在在编译器就可以找出来了。而且使用TMP的C++程序可能在每一方面都高效：较小的可执行文件、较短的运行期、较少的内存需求。

但是将工作从运行期转移到编译期的另一个结果是，编译时间变长了。

3、TMP可被用来生成“基于政策选择组合”的客户制定代码，也可用来避免生成对某些特殊类型并不适合的代码。

# 8、定制new和delete

STL容器所使用的heap内存是由容器所拥有的分配器对象管理，不是被new和delete直接管理。

## 条款49：了解new-handler的行为

1、在C++中，new-handler是一个函数指针，当operator new抛出异常以反映一个未获满足的内存需求之前，C++会调用new-handler指向的函数来处理这种情况
可以通过set_new_handler函数来设置new-handler。

```c++
namespace std {
    //new_handler是个typedef，其定义出一个指针，指向函数，该函数没有参数也不返回任何东西
    typedef void(*new_handler) ();
 	//set_new_handler函数的参数和返回值都是new_handler
    new_handler set_new_handler(new_handler p) throw(); //throw()是一份异常明细，表示该函数不抛出任何异常
}
```

set_new_handler函数的参数是个指针，这个参数指针指向operator new无法分配足够内存时应该被调用的函数；set_new_handler函数的返回值也是个指针，这个返回值指针指向set_new_handler函数被调用前正在执行的那个new-handler函数。用法如下：

```c++
void outOfMem() {
    cerr << "Unable to satisfy request for memory\n";
    abort();//让程序夭折
}
int main() {
    set_new_handler(outOfMem);
    int* pBigDataArray = new int[10000000L];
}
//如果operator new无法为10000000个整数分配足够空间，outOfMem就会被调用，于是程序就会在发出一个信息后夭折。
```

2、operator内含一个无穷循环，退出此循环的唯一方法是不：内存被成功分配或者做了一件下述事情：让更多内存可用、安装另一个new-handler、卸除new-handler、抛出bad_alloc异常、承认失败直接return。

3、当operator new无法满足内存申请时，它会不断调用new-handler函数，直到找到足够内存。一个设计良好的new-handler函数必须做以下事情:
1️⃣让更多内存可被使用。比如程序一开始执行就分配一大块内存，而后当new-handler第一次被调用，再将它们还给程序使用。
2️⃣安装另一个new-handler。如果目前这个new-handler无法取得更多可用内存，或许它知道另外哪个new-handler有此能力，如果这样的话，这个new-handler就可以安装另外那个替换自己，下一次new就会调用新的new-handler。
3️⃣卸除new-handler。即将null传递给set_new_handler，一旦没有安装任何new-handler，operator new会在内存分配不成功时抛出异常。
4️⃣抛出bad_alloc(或派生自bad_alloc)的异常。这样的异常不会被operator new捕捉，因此会被传播到内存索求处。
5️⃣不返回，通常调用abort或exit。

4、可以以不同的方式处理内存分配失败的情况:

```c++
class X {
public:
    static void outOfMemory();
};
class Y {
public:
    static void outOfMemory();
};
X* p1 = new X;//如果分配不成功，调用X::outOfMemory
Y* p2 = new Y;//如果分配不成功，调用Y::outOfMemory
```

5、假设我们打算处理Widget类的内存分配失败的情况。我们需要声明一个类型为new_handler的static成员，用以指向Widget类的new-handler，static成员必须在class定义式之外被定义:

```c++
class Widget {
public:
    static new_handler set_new_handler(new_handler p) throw();
    static void* operator new(size_t size) throw(bad_alloc);
private:
    static new_handler currentHandler;
};

new_handler Widget::currentHandler = 0; //初始化为null
```

Widget内的set_new_handler函数会将它获得的指针存储起来，然后返回先前存储的指针：

```c++
new_handler Widget::set_new_handler(new_handler p) throw() {
    new_handler oldHandler = currentHandler;
    currentHandler = p;
    return oldHandler;
}
```

最后，Widget的new操作符做以下事情：
1️⃣调用标准set_new_handler，告知Widget的错误处理函数。这会将Widget的new-handler安装为全局的new-handler。
2️⃣调用全局的new，执行实际的内存分配。如果分配失败，全局new会调用Widget的new-handler。
3️⃣如果全局的new能够分配足够一个Widget对象所用的内存，Widget的operator new会返回一个指针，指向分配所得。

6、在C++中，nothrow new是一种内存分配方式，用于在分配内存时不抛出异常。通常，当使用new操作符进行内存分配时，如果内存分配失败，C++会抛出std::bad_alloc异常。但是，通过使用nothrow new，可以在内存不足时返回一个空指针而不是抛出异常。举例如下：

```c++
#include <iostream>
#include <new>
int main() {
    int* ptr = new (std::nothrow) int[1000000000000]; // 尝试分配一个非常大的整型数组
    if (ptr == nullptr) {
        std::cout << "Memory allocation failed!\n";
    } else {
        // 内存分配成功，继续处理
        delete[] ptr;
    }
    return 0;
}
```

## 条款50：了解new和delete的合理替换时机

1、想要替换编译器提供的operator new和operator delete，以下是几种常见的理由：
1️⃣用来检测运用上的错误。
2️⃣为了增加分配和归还的速度。
3️⃣为了收集动态分配内存的使用统计信息。
4️⃣为了降低缺省内存管理器带来的额外空间开销。

## 条款51：编写new和delete时需固守常规

1、operator new的返回值十分单纯，如果有能力分配内存，就返回一个指针指向那块内存，如果没有那个能力，就抛出一个bad_alloc异常。

2、奇怪的是C++规定，即使客户要求0bytes，operator new也得返回一个合法的指针。

3、operator delete应该在收到null指针时不做任何事。class专属版本则还应该处理“比正确大小更大的(错误)申请”，即当使用new操作符为类专属的operator new分配内存时，如果请求的内存大小超过了实际需要的大小，需要在类的专属版本的operator delete函数中检查并处理这种情况。

## 条款52：写了placement new也要写placement delete

1、Placement new 是一个特殊形式的 new 操作符，允许在预先分配的内存块上构造对象，而不是在堆上动态分配内存。它接收一个指向已分配内存位置的指针，并在该位置上构造对象。通常用于需要精确控制对象内存位置的情况。示例如下：

```c++
#include <iostream>
#include <new>
class Widget {
public:
    Widget(int value) : value(value) {
        std::cout << "Widget constructed with value: " << value << std::endl;
    }
    ~Widget() {
        std::cout << "Widget destroyed" << std::endl;
    }
private:
    int value;
};
int main() {
    // 分配内存
    void* buffer = malloc(sizeof(Widget));
    // 在预分配的内存位置上构造对象
    Widget* widget = new (buffer) Widget(42);
    // 手动调用析构函数
    widget->~Widget();
    // 释放内存
    free(buffer);
    return 0;
}
```

2、在 C++ 中，如果你使用了 placement new 来在特定的内存位置上构造对象，那么在适当的时候，你应该手动调用对象的析构函数，然后根据情况释放该内存。

# 9、杂项讨论

## 条款53：不要轻易忽略编译器的警告

1、警告信息天生和编译器相依，不同的编译器有不同的警告标准，不是只有错误才值得注意。

## 条款54：让自己熟悉包括TR1在内的标准程序库

1、TR1(Technical Report 1)是指 C++ 技术报告 1，它是由 C++ 标准委员会发布的技术报告，旨在提供一些新的库组件，这些组件最终被包含在 C++11 标准中。

TR1 包含了一系列新的库组件，主要是一些标准库的扩展，这些扩展提供了一些 C++ 标准库中没有提供的功能。TR1 的目标之一是为 C++ 标准库引入一些常见的功能，以便开发人员能够更轻松地实现复杂的任务。

```

```

2、TR1 中包含了一些重要的组件，包括但不限于：

1️⃣智能指针（Smart Pointers）：引入了 shared_ptr、weak_ptr 和 unique_ptr，这些智能指针提供了更安全和方便的内存管理方式。
2️⃣元组（Tuples）：引入了 tuple 类型，用于存储多个元素的组合，并提供了一系列函数用于访问和操作元组中的元素。
3️⃣正则表达式（Regular Expressions）：引入了支持正则表达式的类和函数，用于进行文本模式匹配操作。
4️⃣函数对象（tr1::function）：引入了 function 类型，允许将函数作为对象进行操作，方便泛型编程。

3、TR1 的一些组件在 C++11 标准中被纳入了标准库，因此在使用较新版本的 C++ 标准时，可能不再需要显式地引入 TR1 库。

4、TR1自身只是一份规范。为了获得TR1提供的好处，你需要一份实物。一个好的实物来源是Boost。

## 条款55：让自己熟悉Boost

1、Boost是一个社群，也是一个网站。致力于免费、源码开放、同僚复审的C++程序库开发。Boost在C++标准化过程中扮演深具影响力的角色。

2、Boost提供许多TR1组件实现产品，以及其他许多程序库。
