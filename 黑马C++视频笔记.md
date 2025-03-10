## C++基础知识(部分)

1、short 2byte -32768~32767 即2^15-1
int 4byte 2^31
long 4byte 2^31
long long 8byte 2^63 63就是2^8-1 
bool类型只占用一个字符的空间

2、sizeof(数据类型/变量) 可以求出数据类型占用内存大小

3、浮点型分为两种: 
单精度float 4字节 7位有效数字
双精度double 8字节 15~16位有效数字 小数点前面的也算有效数字
C++默认情况下输出一个小数会输出6位有效数字

4、3e2表示3乘10^2即300 
3e-2表示3乘10^-2即0.03

5、定义字符时 单引号里面只能有一个字符 例如a = 'c' 只占用一个字符 而a = 'as'就不对 
创建字符型变量时要用单引号 例如char a = "C"就不对
想要得到字符的ASCII码可以使用int(a) 表示的就是字符'C'的ASCII码
常用的ASCII码 'a':97 'A':65 '0':48
ASCII码分为两个部分:
非打印控制字符:0-31分配给了控制字符 用于控制像打印机等一些外围设备
打印字符:32-126分配给了键盘上能找到的字符 当查看或打印文档时就会出现

6、字符串的定义:
C语言风格:char str[] = "hello world"
C++风格:string str = "hello world" (但是使用C++风格时要注意在文件开头加上#include<string>)

7、switch语句

```c++
switch(表达式) {
    case 结果1 :执行语句;break;
    case 结果2 :执行语句;break;
    ...
    default : 执行语句;break;
}
```

switch语句也有缺点 在判断的时候只能是整型或者字符型 不能是区间

8、数组arr[10]，sizeof(arr)表示的是数组整体所占的内存 sizeof(arr)/sizeof(arr[0])就等于数组里面元素的个数 不知道数组的个数的时候就可以这样表示

9、二维数组的定义可以省略行数 但是不能省略列数

10、new返回的是对应类型的指针

```c++
int *a = new int[5]; // 分配了一个包含5个整数的数组
int *b = new int(5); // 分配了一个值为5的整数
```

11、uint64_t是一种无符号整数数据类型，用于存储64位的无符号整数。它可以存储的整数范围是0到2^64-1。在C++中，uint64_t通常是<cstdint>头文件中定义的一个类型别名，实际上它是unsigned long long的一个别名，保证了其大小为64位。

## 程序的内存模型

1、C++程序在执行的时候，将内存大致分为四个区：
1、代码区：存放函数的二进制代码，由操作系统进行管理
代码区存放二进制的CPU执行的机器指令
代码区是共享的，共享的目的是对于频繁被执行的程序，只需要在内存中有一份代码即可
代码区是只读的，原因是防止程序意外修改了他的指令 

2、全局区：存放全局变量和静态变量以及常量，这几类变量的位置都离得很近，在一个区域中，该区域的数据在程序结束后由操作系统管理和释放。注意，局部常量不存放在全局区中，例如在函数里面定义的const int a = 0; a就是局部常量，局部常量跟局部变量放在一起的
静态变量的定义: static int a = 10;

代码区和全局区是程序运行之前分配的区域

3、栈区：由编译器自动分配释放，存放函数的参数值，局部变量等
栈区的数据会在函数执行完之后自动释放，所以函数不能返回局部变量(局部变量存放在栈区)的地址，不然会乱码。

4、堆区：由程序员分配和释放，若程序员不释放，程序结束时由操作系统回收
利用关键字new，可以将数据开辟到堆区(如果正常在函数里定义的话是会存放到栈区，函数执行完之后就会被自动释放，如果不想被系统自动释放就要在堆区开辟)

```c++
int * func() {
    int *p = new int(10); //在堆区开辟一个空间，赋值为10
    //new返回的是该数据类型的指针，所以要创建该类型的变量去接收它
    return p;//将其地址赋值给p并返回
}
```

堆区数据的释放由程序员决定，使用关键字delete

```c++
int * p = func();
delete p;
```

用new在堆区开辟数组:返回的是数组的首地址

```c++
int * arr = new int[10]; //10代表数组有十个元素
//返回的是数组的首地址
for (int i = 0;i<10;i++)
{
    arr[i] = i+10;//给数组的10个元素赋值10-19
}
```

用delete释放数组的时候要加[]，告诉程序你释放的是一个数组

```c++
delete[] arr;
```

栈区和堆区是程序运行后分配的空间

## C++中的引用

1、C++引用：给变量起别名
语法：数据类型 &别名 = 原名   例如：int &b = a;此时b就是a的别名，二者都指向同一个操作内存，如果修改b的值，那么a的值也会被随之改变

引用的注意事项:
1、引用必须初始化，例如直接int &b;就是错的
2、引用在初始化之后不可以改变，一旦b变成a的别名了，就不能再变成c的别名了

引用做函数的返回值
注意事项：
1、不要返回局部变量的引用

```c++
int& test1() {
   	int a = 10;//局部变量存放在四区中的栈区
    return a; 
}
int main() {
     int &ref = test1();
     cout << ref << endl;//第一次能输出10，是因为编译器做了一次保留
     cout << ref << endl;//第二次就会输出很大的数，即乱码了，是因为a的值在test1函数执行之后就被释放了。
}
```

2、函数的调用可以作为左值(即可以被赋值~)

```c++
int& test2() {
      static int a = 10;//静态变量存放在全局区，全局区上的数据在程序结束之后释放
      return a; 
}
int main() {
   int &ref = test2();
   cout << ref << endl;
   cout << ref << endl;//这个时候无论输出多少ref都是10了
   test2() = 1000; //即可作为等号左边被赋值的对象
   cout << ref << endl;
   cout << ref << endl;
}
函数的输出就是10 10 1000 1000
```

3、常量引用，主要用来修饰函数的形参，防止误操作

```c++
void showValue(const int &val) {
   cout << val << endl;
}
// 这个函数的目的只是显示出变量的值，并不打算修改变量的值，所以为了防止误操作，加一个const，如果要在函数里面修改val值 就是误操作 会报错
```

## 函数高级

函数默认参数，例如int func(int a,int b = 20,int c = 30)
调用函数的时候如果为func(10)，就是给a赋值10，如果func(10,30)就是给a赋值10，给b赋值30，如果没给b传的话用的就是函数默认的参数的值20
注意事项：
1、如果某个形参有了默认值，从这个位置往后都要有默认值，例如int func(int a=10,int b)就是错的，有默认值的形参应该放到最后
2、如果函数的声明有默认参数，那么函数的实现就不能有默认参数，例如

```c++
int func(int a = 10,int b = 20); //函数的声明形参有默认参数
int func(int a,int b) //函数的实现就不能再加上默认参数了，直接写就行
{
   return a+b;
}
// 或者是声明里面没有默认参数，但实现里面有也可以。
// 即声明和实现里面只能有一个有默认参数
```

函数占位参数：
返回值类型 函数名(数据类型)
例如 int func(int) 调用的时候传一个int类的数据即可，只是占位用的
占位参数也可以有默认值，例如int func(int 10),此时甚至连值都不用传，知道此概念即可

函数重载满足的条件：
1、同一个作用域下
2、函数名称相同
3、函数参数类型不同或者个数不同或者顺序不同。
注意，函数的返回值并不能作为函数重载的判断条件

函数重载的注意事项
1、引用作为重载的条件

```c++
void func(int &a) {
   cout << "fun(int &a)调用" << endl;
}
void func(const int &a) {
   cout << "fun(const int &a)调用" << endl;
}
// 这两个函数满足重载条件
int main() {
   int a = 10;
   func(a);//调用的是第一个函数，因为a是一个局部变量可读可写，第二个只可读不可写
   func(10);//调用的是第二个函数，因为const int &a = 10;是合法的
}
```

2、函数重载碰到默认参数

```c++
void func(int a,int b = 10) {
   cout << "fun(int a,int b = 10)调用" << endl;
}
void func(int a) {
   cout << "fun(int a)调用" << endl;
}
int main() {
   func(10);//就会出现二义性，即第一个也可以第二个也可以调用，就会报错，这种情况要尽量避免出现。
}


```

## 类和对象

1、C++中面向对象的三大特性为：封装、继承、多态

封装：将属性和行为作为一个整体，表现生活中的事物
将属性和行为加以权限控制

2、类中protected和private的区别：
二者都是类内可以访问，类外不可以访问，但是子类可以访问protected内容，不可以访问private内容

3、C++中struct和class都可以表示类，唯一的区别是默认访问权限不同，前者是public，后者是private

4、C++项目中可以把类分成.h头文件和.cpp源文件，在.h头文件中写成员函数和成员变量的定义即可；在.cpp源文件中写成员函数的定义(注意要在函数名前加上"类名::"，表示是这个类里面的成员函数)，并在文件开头include一下.h头文件

5、默认情况下，c++编译器至少给一个类添加三个函数：
1)默认构造函数
2)默认析构函数
3)默认拷贝构造函数，对属性值进行拷贝

6、构造函数：没有返回值也不写void，函数名称跟类名相同；可以有参数，因此可以发生重载
拷贝构造函数写法(以Person为例)

```c++
Person(const Person &p) {
    age = p.age; //将传入的人身上的属性拷贝到我身上
}
```

如果在定义Person对象的时候传入了数据，就是调用了拷贝构造函数，如果没有传入数据，就是调用默认的构造函数，即构造函数的重载。
拷贝构造函数调用时机：
1)、使用一个已经创建完毕的对象来初始化一个新对象 例如Person p2(p);
2)、值传递的方式给函数参数传值
3)、以值的方式返回局部对象

7、析构函数：没有返回值也不写void，函数名称跟类名相同，在名称前加上符号~；析构函数不能有参数，因此不能重载。通常在析构函数里释放在堆区开辟的空间。例如：

```c++
~Person() {
	if(m_height != nullptr) {
        delete m_height; //释放堆中开辟的内存
        m_height = nullptr; //防止出现野指针
    }
}
```

8、如果在类中自己定义了拷贝构造函数，编译器就不会再自动创建默认构造函数；

9、浅拷贝：直接进行赋值，可能会带来问题，就是堆区的内存会被重复释放。

```c++
m_height = p.m_height; //这是编译器默认的拷贝构造，这样的话还是同一个堆内存空间，容易发生问题
```

10、深拷贝：在堆中重新开辟空间进行拷贝，这样的话就不会有堆中一个内存被重复释放的问题发生。
深拷贝的实现：在自己定义的拷贝构造函数里面进行实现：

```c++
m_height = new int(*p.m_height);//重新开辟一个堆空间
```

如果属性有在堆区开辟的，一定要自己提供拷贝构造函数，防止浅拷贝带来的问题

11、用初始化列表初始化数据对象：

```c++
Person(int a,int b,int c):m_A(a),m_B(b),m_C(c){}
```

12、当其他类作为本类成员时，先构造其他类，再构造本类自身。
析构函数的执行顺序与构造函数相反。即先销毁本类，再销毁其他类

13、静态成员变量(static) 
1)所有对象共享一份数据，所以静态成员变量可以通过类名的方式进行访问，例如：Person::m_A就表示变量m_A，p1.m_A和p2.m_A也可以表示，修改其中任意一个，所有的都会随之改变
2)在编译阶段分配内存(即点击.exe文件之前就已经分配好了)
3)类内声明、类外初始化

```c++
class Person {
public:
    static int m_A; //类内声明
};

int Person::m_A = 100; //类外初始化，注意要加上Person::，表示这是Person类内的变量，static在类外声明的时候就不用再写了
```

14、静态成员函数(static)
1)所有对象共享一个函数
2)静态成员函数只能访问静态成员变量

```c++
class Person {
public:
    static void func() {
        m_A = 100;//静态成员函数只能访问m_A，不能访问非静态成员变量m_B,因为m_B是属于特定对象的属性;
        cout << "111" << endl;
    }
    static int m_A;
    int m_B;
}

//两种访问方式：通过对象、通过类名
Person p;
p.func();

Person::func();
```

15、在C++中，类内的成员变量和成员函数分开存储。
空对象占用内存空间为1字节，编译器之所以会给空对象也分配空间，是为了区分空对象占内存时的位置。即

```c++
class Person{
	
};//里面啥也没有
Person p;
//sizeof(p)的值就是1

//添加了一个非静态变量之后
class Person {
    int m_A;
}; 
Person p;
//sizeof(p)的值就是4了 不是1+4，注意，1只是分配给你标记的，你现在有变量了 就不用给你再分配这1字节了

//添加一个静态变量之后
class Person {
    int m_A;
    static int m_B;
}
int Person::m_B = 0;
Person p;
//sizeof(p)的值还是4，因为静态变量不属于某个类的对象，当然不占这个对象的内存

//添加一个非静态成员函数或者静态成员函数也不会影响sizeof(p),因为成员变量和成员函数是分开存储的
```

16、类中的this指针，this指针不可以被修改，即不可以修改指向的位置；但是其指向的值可以被修改

```c++
class Person {
public:
    Person(int age) {
        this->age = age; //用this解决了名称冲突
    }
    Person& PersonAddAge(Person &p) { //返回值是Person&，如果不加引用，那么返回的就是一个拷贝，而不是本体，链式编程就失效了
        this->age += p.age;
        return *this; //返回的是调用对象的本体p2
    }
    int age;
}
void func() {
    Person p1(10);
    Person p2(10);
    p2.PersonAddAge(p1).PersonAddAge(p1).PersonAddAge(p1);//运用了链式编程思想 最后p2.age变成了40
}
```

17、const修饰成员函数

```c++
class Person {
public:
    void showPerson() const {
        this->m_A = 100; //错误，const修饰了成员函数，因此this指针指向的值也不能修改了
        this->m_B = 100; //正确，因为加了mutable修饰
    }
    void func() {
        m_A = 100;
    }
    int m_A;
    mutable int m_B;
}

void test01() {
    const Person p; //const修饰的常对象
    p.showPerson(); //正确，常对象只能调用常函数
    p.func(); //错误，常对象不能调用普通成员函数，因为普通成员函数可以修改属性
}
```

18、全局函数、类、成员函数都可以做友元

```c++
class Person{
    friend class Building; //类做友元 表示Building类可以访问本类中的private内容
	friend void googGay(); //全局函数做友元
    friend void Building::visit(); //Building类中成员函数visit是本类的友元，可以访问private内容
public:
    
private:
    
};
void goodGay() { //全局函数的定义
    //有了上面的友元声明,goodGay就可以访问private里面的内容了
}
```

19、加法运算符重载：可以通过类内成员函数或者全局函数进行重载

```c++
class Person {
public:
    int m_A;
    int m_B;
    // 通过自己写成员函数 可以自定义p1 + p2 即重载+
	Person operator+ (Person &p) {
        Person temp;
        temp.m_A = this->m_A + p.m_A;
        temp.m_B = this->m_B + p.m_B;
        return temp;
    }
};

Person p1; p1.m_A = 10; p1.m_B = 10;
Person p2; p2.m_A = 10; p2.m_B = 10;

// 调用成员函数
Person p3 = p1.operator+(p2);
// 系统给我们简化为了
Person p3 = p1 + p2;  //就是我们预期想要的效果

// 也可以通过全局函数进行+重载
Person operator+(Person &p1,Person &p2) {
    Person temp;
    temp.m_A = p1.m_A + p2.m_A;
    temp.m_B = p1.m_B + p2.m_B;
    return temp;
}
// 调用全局函数
Person p3 = operator+(p1,p2);
// 系统简化为了
Person p3 = p1 + p2;
```

20、左移运算符(<<)重载

```c++
class Person {
	friend ostream& operator<<(ostream& cout,Person& p); //成员变量通常是private的，所以记得给全局函数加友元
private:
    int m_A;
    int m_B;
    //利用成员函数重载<<不太行，因为最终cout要在左边，成员函数不好实现 
};
// 只能利用全局函数重载<<
ostream& operator<<(ostream& cout,Person& p) { //不一定是cout，也可以起别的名字
	cout << m_A << endl;
    cout << m_B << endl;
    return cout;
}
Person p1;
p1.m_A = 10;
p1.m_B = 10;
operator<<(cout,p); //系统简化成了cout << p
cout << p << endl; //最后输出的结果就是上面全局函数重载的
// 如果要连续输入<<,那么返回值一定要有，且设置成cout，即cout << p的返回值是cout，又变成了cout << endl;
```

21、递增运算符(++)重载

```c++
class MyInteger {
public:
    MyInteger() { //构造函数
        m_Num = 0;
    } 
    //成员函数重载前置++
    MyInteger& operator++() { //前置返回值是类引用 这样保证了后续处理是对同一个值进行处理
        m_Num++;
    }
    //成员函数重载后置++
    MyInteger operator++(int) { //后置不返回引用是因为这里返回的只是当前的值
        //先记录当时的结果
        MyInteger temp = *this;
        //后做递增操作
        m_Num++;
        //最后将当时记录的结果返回
        return temp;
    }
private:
    int m_Num;
};
```

22、赋值运算(=)符重载

```c++
class Person {
public:
    Person(int age) { //构造函数
        m_Age = new int(age);
    }
    ~Person() { //析构函数
        if(m_Age != NULL) {
            delete m_Age;
            m_Age = NULL;
        }
    }
    // 成员函数重载=，返回值是Person&，这样才能进行连等，p1=p2=p3
    Person& operator=(Person& p){
        //编译器默认是浅拷贝
        //m_Age = p.m_Age;
        //防止同一块堆区被重复释放 所以使用深拷贝，即新开辟一块堆区空间
        if(m_Age != NULL) { //应先判断是否有属性在堆区 如果有就先释放干净，然后再深拷贝
            delete m_Age;
            m_Age = NULL;
        }
        m_Age = new int(*p.m_Age); //深拷贝
        return *this;
    }
private:
    int* m_Age;
};
```

23、函数调用运算符()重载

```c++
class MyPrint {
public:
    //成员函数重载函数调用符号()
    void operator()(string test) {
        cout << test << endl;
    }
};

void test01 {
    MyPrint myPrint;
    // myPrint.operator()("hello world");可以简写为如下格式
    myPrint("hello world");//使用起来非常类似于函数调用，于是被称为仿函数
    
    //匿名函数对象 
    MyPrint()("hello world");//当前行结束时候对象就没了 只是单纯的调用一下重载的()
}
```

24、⭐️类和对象之--继承 (重中之重)   继承可以增加代码复用性

三种继承方式：public，protected，private

父类中的private内容，无论子类哪种继承都无法获得

公共继承(public)：

```c++
class BasePage {
public:
    int a;
protected:
    int b;
private:
    int c;
};

class Java:public BasePage {  //Java类被称为子类、派生类；BasePage类又被称为父类、基类
public:
	int a;
protected:
    int b;
// 父类中变量的权限在子类中不变
}
```

保护继承(protected):

```c++
class Java:protected BasePage {
protected:
    int a;
    int b;
    // 父类中的public和protected权限内容都变成了protected权限
}
```

私有继承(private):

```c++
class Java:private BasePage {
private:
    int a;
    int b;
    // 父类中的public和protected内容都变成了private权限
}
```

内存角度，父类中所有的非静态成员属性都会被子类给继承下去，只是父类中的private属性被编译器给隐藏了，所以访问不到，在计算子类所占内存的时候包括父类中的所有非静态成员属性和子类成员的特有属性

25、继承中的构造和析构顺序如下：构造与析构顺序相反~
1️⃣构造父类。2️⃣构造子类。3️⃣析构子类。4️⃣析构父类

26、如果想通过子类对象访问父类中的同名成员，需要加作用域

```c++
Son s;//s是子类对象
//子类Son和父类Base里面都分别定义了变量m_A，二者同名了
s.m_A; //表示子类里的m_A
s.Base::m_A; //表示父类里的m_A

//子类Son和父类Base里面都分别定义了函数func
s.func(); //表示子类里的func
s.Base::func(); //表示父类里的func，需要注意的是，无论父类中的函数是否存在重载与子类的参数是否不同，调用父类的成员函数的时候，都记得要加Base::，举例如下：
s.func(100);// 错误
s.Base::func(100); //正确
```

27、继承中的同名静态成员的访问，基本上与上面的非静态同名成员一样

```c++
// 通过类名访问静态成员变量
Son::m_A; //Son类中的静态成员变量
Base::m_A; //Base类中的静态成员变量
Son::Base::m_A;//Base类中的静态成员变量
```

28、一个类可以继承多个类(多继承)，语法如下：

```c++
class Son:public Base1,public Base2 {
    
};
// 当父类中出现同名成员，需要加作用域区分是哪个父类的同名成员
```

29、菱形继承：羊类和驼类继承动物类，然后羊驼类继承羊类和驼类，这样就形成了一个菱形，有两个问题：
1️⃣羊驼类中变量可能有二义性，解决方法：添加作用域运算符
2️⃣羊驼类中代码会有重复，因为羊类和驼类分别都有动物中的公共内容，解决方法：添加virtual关键字(虚继承)，Animal类称为虚基类

```c++
class Sheep:virtual public Animal {};
class Tuo:virtual public Animal {};
class SheepTuo:public Sheep,public Tuo {}; 
```

30、C++多态：分为静态多态和动态多态
静态多态：函数地址早绑定--编译阶段确定函数地址
动态多态：函数地址晚绑定--运行阶段确定函数地址

动态多态满足条件：
1️⃣有继承关系
2️⃣子类重写父类中的虚函数

动态多态使用：父类的指针或者引用，指向子类对象

```c++
class Animal {
public:
    //虚函数，加virtual是为了让地址不那么早被绑定
    virtual void speak() { 
        cout << "动物在说话" << endl;
    }
};
class Cat:public Animal {
public:
    virtual void speak() { //这个virtual加不加都行
        cout << "小猫在说话" << endl;
    }
};
class Dog:public Animal {
public:
    virtual void speak() {
        cout << "小狗在说话" << endl;
    }
}

void test01() {
    Animal* cat = new Cat; //即父类指针指向子类对象
    cat->speak(); // 由于是指针 所以要用->
    delete cat; //用完后记得销毁 因为new的是堆区的数据
    Animal* dog = new Dog;
    dog->speak();
    delete dog;
    // 输出的是小猫小狗在说话
}
```

具体原理见bilibili黑马C++P136

多态的优点：1️⃣代码组织结构清晰。2️⃣可读性强。3️⃣利于前期和后期的维护和拓展。C++中提倡用多态来设计程序架构~

31、多态中，通常父类中虚函数的实现是毫无意义的，因此可以将虚函数彻底虚化，称为纯虚函数，语法举例为：

```c++
virtual int func() = 0;
```

当类中有了纯虚函数，这个类也被称为抽象类。
抽象类的特点：1️⃣无法实例化对象。2️⃣子类必须重写抽象类中的纯虚函数，否则也属于抽象类

32、多态案例：制作饮品

```c++
class AbstractDrinking {
public:
    // 定义一系列纯虚函数，子类具体做哪种饮料再对父类的纯虚函数进行重写即可
    virtual void Boil() = 0; //煮水
    virtual void Brew() = 0; //冲泡
    virtual void PourInCup() = 0; //倒入杯中
    virtual void PutSomething() = 0; //加入辅料
    
    // 制作饮品
    void makeDrink() {
        Boil();
        Brew();
        PourInCup();
        PutSomething();
    }
};

//制作饮品：咖啡
class Coffee:public AbstractDrinking {
public:
    virtual void Boil() {
        cout << "煮农夫山泉" << endl;
    }
    virtual void Brew() {
        cout << "冲泡咖啡" << endl;
    }
    virtual void PourInCup() {
        cout << "倒入马克杯中" << endl;
    }
    virtual void PutSomething() {
        cout << "加入糖和牛奶" << endl;
    }
};

//制作饮品：茶
class Tea:public AbstractDrinking {
public:
    virtual void Boil() {
        cout << "煮矿泉水" << endl;
    }
    virtual void Brew() {
        cout << "冲泡茶叶" << endl;
    }
    virtual void PourInCup() {
        cout << "倒入保温杯中" << endl;
    }
    virtual void PutSomething() {
        cout << "加入枸杞" << endl;
    }
};

// 制作传入的饮品
void doWork(AbstractDrinking* abs) {
    abs->makeDrink();
}

void test01() {
    doWork(new Coffee); //制作咖啡
    doWork(new Tea); //制作茶叶
}

int main() {
    test01();
}
```

33、多态情况下还有问题，就是无法指向子类对象的父类指针来对子类进行析构，例如：

```c++
Animal* animal = new Cat("Tom");
animal->speak();
delete animal; 
```

此时构造和析构的顺序为：1️⃣构造父类。2️⃣构造子类。3️⃣析构父类。
少了"析构子类"这一步。解决方法为：使用虚析构，即在父类析构函数前面加virtual关键字，例如：

```c++
class Animal {
    // 虚析构
	virtual ~Animal() {
    	cout << "Animal析构函数调用" << endl;
	}
    // 纯虚析构 需要在类外进行实现 因为析构函数需要有具体的函数实现 不然会报错
    virtual ~Animal() = 0;
};

Animal::~Animal() {
    cout << "纯虚析构函数的调用" <<  endl;
}

```

有了纯虚析构之后，这个类也属于抽象类，无法实例化对象
如果子类中没有堆区数据需要释放，也可以不写虚析构和纯虚析构

## C++文件操作

1、C++中对文件操作需要先引用头文件：

```c++
#include<fstream>   //读写操作，通常只引用这个就够了
#include<ofstream>  //写操作
#include<ifstream>  //读操作
```

2、文件类型分为两种：
1️⃣文本文件：文件以文本的ASCII码形式存储在计算机中。
2️⃣二进制文件：文件以文本的二进制形式存储在计算机中。

3、写入文件步骤如下：

```c++
#include<fstream>
//创建流对象
ofstream ofs; 
//打开文件
ofs.open("文件路径",打开方式); 
// 写入数据
ofs << "写入的数据";
//关闭文件
ofs.close(); 
```

举例如下:

```c++
#include<iostream>
#include<fstream>
using namespace std;
void test01() {
    ofstream ofs;
    ofs.open("test.txt",ios::out);
    ofs << "hello world" << endl;
}
```

4、读文件步骤如下：

```c++
#include<fstream>
//创建流对象
ifstream ifs;
//打开文件
ifs.open("文件路径",打开方式);
//判断文件是否被打开
if(!ifs.is_open()) { //判断文件是否被打开
    cout << "文件打开失败" << endl;
    return ;
}
//读数据
四种方式读取
//关闭文件
ifs.close();
```

举例如下：

```c++
#include<iostream>
#include<fstream>
#include<string>
using namespace std;
void test() {
    ifstream ifs;
    ifs.open("test.txt",ios::in);
    if(!ifs.is_open()) { //判断文件是否被打开
        cout << "文件打开失败" << endl;
        return ;
    }
    //读数据
    //第一种
    char buf[1024] = {0}; 
    while(ifs >> buf) {//把文件中的内容读到buf中，然后再输出
        cout << buf <<  endl;
    }
    //第二种
    char buf[1024] = {0};
    //使用getline来一行一行读取到buf中，每一行读取成功就返回true
    //getline的第二个参数表示每一行最多读多少个字符
    while(ifs.getline(buf,sizeof(buf))) { 
        cout << buf << endl;
    }
    //第三种
    string buf;
    // 从ifs中把内容一行一行读到buf中
    while(getline(ifs,buf)) {
        cout << buf << endl;
    }
    //第四种 不太推荐 因为效率不高
    char c;
    // EOF指文件尾，get是读取一个字符，一直读直到读到文件尾停止
    while((c = ifs.get()) != EOF) {
        cout << c;
    }
    
    ifs.close();
    
} 
```

5、二进制下写文件：

```c++
#include<fstream>
//创建流对象
ofstream ofs;
//打开文件
ofs.open("person.txt",ios::out | ios::binary);
//写文件
Person p = {"张三",18};
ofs.write((const char*)&p,sizeof(Person));//强制类型转换成char指针
//关闭文件
ofs.close();
```

6、二进制下读文件：

```c++
#include<fstream>
//创建流对象
ifstream ifs;
//打开文件，并判断是否打开成功
ifs.open("person.txt",ios::in | ios::binary);
if(!ifs.is_open()) {
    cout << "文件打开失败" << endl;
    return ;
}
//读文件
Person p;
ifs.read((char *)&p,sizeof(Person));
//此时就已经将person.txt中的内容读取到了Person对象p中
```

## 模板

1、模板就是建立通用的模具，大大提高复用性
C++提供两种模板：函数模板和类模板

2、函数模板：

```c++
template<typename T> //T是通用数据类型 起什么名字都可以
//template<class T>跟上面是一样的，只是有些人喜欢类用class，函数用typename，其实都一样
void mySwap(T &a,T &b) {
    T temp = a;
    a = b; 
    b = temp;
}

void test01() {
    int a = 10,b = 20;
    //1、自动推导类型
    mySwap(a,b);
    //2、显示指定类型
    mySwap<int>(a,b); //用<int>指定T为int
}
//函数模板的两个注意事项：
//1、自动类型推导，必须推导出一致的数据类型T才可以使用
//2、模板必须要确定出T的数据类型，才可以使用，例如
template<class T>
void func() {
	cout << "func调用" << endl;
}

void test02() {
    func<int>(); //必须要显示指定才能调用了，因为此时无法自动推导了
}


```

3、普通函数调用可以发生隐式类型转换~比如10 + 'a'会把字符a转换成 ascii码数字去跟10相加；
函数模板通过显示指定类型，可以发生隐式类型转换~因为显示指定类型指定了要隐式转换成哪种类型~
函数模板通过自动类型推导，不可以发生隐式类型转换~
因为10+'a'编译器不知道是要把10转换成字符还是把'a'转换成对应的ascii码~

4、普通函数和函数模板的调用规则：
1️⃣如果二者都可以调用，优先调用普通函数
2️⃣可以通过空模板参数列表 强制调用函数模板，比如mySwap<>(a,b);
3️⃣函数模板可以发生函数重载
4️⃣如果函数模板可以产生更好的匹配，优先调用函数模板
比如mySwap('a','b'); 普通函数参数是int，显然模板里面的T推导出char更好

5、模板也有局限性，对于某些特定数据类型，需要用具体化方式做特殊实现，比如类对象，举例如下：

```c++
template<class T>
bool myCompare(T &a,T &b) {
    return a == b;
}

class Person {
public:
    Person(string name,int age) {
        this->m_Name = name;
        this->m_Age = age;
    }
    string m_Name;
    int m_Age;
};
void test() {
    Person p1("Tom",10);
    Person p2("Tom",10);
    if(myCompare(p1,p2)) {
        cout << "p1 == p2" << endl;
    }else {
        cout << "p1 != p2" << endl;
    }
}
//直接运行test函数会报错，因为编译器不知道p1==p2咋判断，一种方法是在类中进行运算符"=="重载，另一种是给模版专门写一个特殊的类型
//在函数前面加上template<>
template<> bool myCompare(Person &p1,Person &p2) {
    return p1.m_Name == p2.m_Name && p1.m_Age == p2.m_Age;
} //也就是说模板在检测到类型是Person时会走这个函数而不是上面那个通用函数了
```

6、类模板

```c++
template<class T1,class T2> //类里面的数据类型可能有很多个，中间用"，"连接
class Person {
public:
    Person(T1 name,T2 age) {
        this->m_Name = name;
        this->m_Age = age;
    }
    void showPerson() {
        cout << "name:" << this->m_Name << "age:" << this->m_Age << endl;
    }
    T1 m_Name;
    T2 m_Age;
}

void test() {
    //类模板的使用
    Person<string,int>p1("孙悟空"，18);
}

//如果想查看类模板中的T1类型变成了什么，可以用如下代码：
typeid(T1).name();
```

类模板没有自动类型推导的使用方式，也就是说调用只能用显示指定类型的方式。
类模板在模板参数列表中可以有默认参数，举例如下：

```c++
template<class T1,class T2 = int>
Person<string>p2("猪八戒"，20);//此时就可以不用写int了，因为有默认值int，当然要是写了其他的值就会覆盖掉int
```

7、类模板中的成员函数只有在被调用时才会被创建~

8、用类模板生成的对象作为函数参数，有三种用法：

```c++
template<class T1,class T2>
Person<string,int>p1("孙悟空"，100)；
//1、指定传入类型(最常用)
void printPerson(Person<string,int>& p) {
    //......
}
//2、将参数模板化
void printPerson(Person<T1,T2>& p) {
    //......
}
//3、将整个Person类模板化
void printPerson(T &p) {
    //......
}
```

9、类模板与继承:

```c++
template<class T>
class Base {
    T m;
};
//必须要知道父类模板中T的类型才能继承给子类
class Son:public Base<int> { //这里加上<int>才是对的 不加就报错了
    
};

//如果想灵活指定父类中的T类型，子类也需要变类模板
template<class T1,class T2>
class Son2:public Base<T2> { 
    T1 obj;
}
void test() {
    Son2<int,char>s2; //就是把char给了T2，也就是相当于给父类传下来的T赋值，然后子类自身的T1赋值为int，实现了父类中T类型的灵活赋值~
}
```

10、模板类的成员函数的类外实现：需要加上模板参数列表

```c++
template<class T1,class T2>
class Person {
public:
    Person(T1 name,T2 age);
    
    void showPerson();

    T1 m_Name;
    T2 m_Age;
}

//构造函数类外实现
template<class T1,class T2> //无论函数中用没用到T1,T2都要加上这一行
Person<T1,T2>::Person(T1 name,T2 age) {
    this->m_Name = name;
    this->m_Age = age;
}

//成员函数类外实现
template<class T1,class T2>
//在类名后面，::前面记得要加上<T1,T2>
void Person<T1,T2>::showPerson() {
    cout << "姓名：" << this->m_Name <<
        "年龄" << this->m_Age << endl;
} 
```

11、一般一个类定义是分.cpp和.h文件来写的，.h来写类的声明，以及类中成员函数的声明，.cpp文件是对.h文件中声明的函数进行实现，然后主函数要是想引用这个类只需要include这个类的.h文件即可。
但是对于模板类，直接引用.h文件不可以用，因为模板类中成员函数只有在被调用时才会被创建~解决方法有两种：
1️⃣改成包含.cpp文件，因为.cpp文件头中也有一步包含.h文件，即让编译器既看到了.cpp文件中的函数实现，又看到了.h文件中的声明~(用的少)
2️⃣将.h和.cpp文件中的内容合并到一起，并且将文件后缀改成.hpp即可(常用)

12、类模板与友元：
1️⃣全局函数做友元，且全局函数类内实现，就直接➕friend关键字即可
2️⃣全局函数做友元，但是全局函数类外实现，这个就有点复杂了，具体见C++黑马P181

## STL初识

1、为了建立数据结构和算法的一套标准，诞生了STL(Standard Template Library,标准模板库)
STL从广义上分为：容器、算法、迭代器
容器和算法之间通过迭代器进行无缝连接
STL几乎所有代码都采用了模板类或者模板函数

2、STL大体分为六大组件：容器、算法、迭代器、仿函数、适配器、空间配置器
1️⃣容器：各种数据结构，比如vector，list，deque，set，map等，用来存放数据
2️⃣算法：各种常见算法，比如sort，find，copy等
3️⃣迭代器：扮演了容器和算法之间的胶合剂(iterator和const_iterator)
4️⃣仿函数：行为类似函数，可作为算法的某种策略
5️⃣适配器：一种用来修饰容器、仿函数、迭代器接口的东西
6️⃣空间配置器：负责空间的配置与管理

3、vector中的迭代器

```c++
vector<int> vec;
vector<int>::iterator it = v.begin();//一般用auto就行，但是也得知道不用auto是怎么写的，类型是vector<int>::iterator
cout << *it << endl;//输出的就是第一个元素
//利用for循环遍历
for(vector<int>::iterator it = vec.begin(); it != vec.begin(); ++it) {
    cout << *it << endl;
}
```

4、for_each函数遍历vector

```c++
#include<algorithm>
void myPrint(int val) {
    cout << val << endl;
}
for_each(vec.begin(),vec,end(),myPrint); //注意包含算法的头文件以及自己编写一个myPrint输出格式的函数即可~
```

5、容器嵌套容器迭代器的遍历

```c++
vector<vector<int>> vec;
for(vector<vector<int>>::iterator it = vec.begin();it != vec.end(); ++it) {
    for(vector<int>::iterator vit = (*it).begin();vit != (*it).end(); ++vit) {
        cout << *vit;
    }
    cout << endl;
}
```

## string容器

1、string字符串的构造(定义)方式

```c++
string s1; //默认构造

const char* str = "hello world"; //C语言风格构造
string s2(str);

string s3(s2);

string s4(10,'a'); //10个字符a
```

2、string字符串赋值方式：= 或者 assign

```c++
//可以把单个字符给字符串赋值
string s = 'a';

//用assign进行赋值
s.assign("hello,C++"); //默认是所有的，即s = "hello,C++";
s.assign("hello,C++",5); //表示前五个字符赋值给s，即s = "hello"
s.assign(s1); //把字符串s1的值拷贝给s
s.assign(10,'w'); //10个w构成s
```

3、字符串string拼接：+= 或者append

```c++
str += "hello"; //拼接字符串
str += 'c'; //拼接字符

str.append("love"); //拼接字符串"love"
str.append("game over",4); //把字符串中的前四个字符"game"拼接到str的尾部
str.append(s1); //拼接字符串s1
str.append(s1,0,4); //拼接字符串s1的前四个字符，从下标0开始截取前四个字符
```

4、string字符串查找和替换：

```c++
//查找 find和rfind
string str1 = "abcdefgde";
int pos = str1.find("de"); //此时pos值是3，因为find是从左往右查找第一个de的开头d是在str1下标为3的地方
int pos = str1.find("df"); //没有这个子串,pos的值为-1
int pos = str1.rfind("de"); //此时pos是7，因为rfind是从右往左是查的最右边那个de，此时的d下标为7

//替换 replace
string str = "abcdefg";
str.replace(1,3,"1111"); //从1号位置起 3个字符 替换成"1111" 变成"a1111efg"
```

5、string字符串的比较：compare函数 逐个相比 比较ASCII码值

```c++
string str1 = "hello";
string str2 = "hello";
str1.compare(str2) == 0 //表示相等(compare主要就是用来比较是否相等的)
str1.compare(str2) == 1(或者表示成>0) //表示str1 > str2
str1.compare(str2) == -1(或者表示成<0) //表示str1 < str2
```

6、string字符串中的字符存取：

```c++
//取字符
string str = "hello world";
str[2]和str.at(2) 都可以表示下标为2的字符

//存(修改)字符
str[2] = 'a';
str.at(2) = 'a';
//二者都可以直接用=修改
```

7、字符串插入和删除：insert和erase

```c++
//插入
string str = "hello";
str.insert(1,"111"); //下标1前面插入111，变成"h111ello"

//删除
str.erase(1,3); //从下标1开始删除3个字符
```

8、子串截取：s.substr()

```c++
string str = "abcdef";
string s = str.substr(1,3); //从下标为1开始截取3个字符，即为bcd

//使用操作举例:获取邮箱中的姓名zhangsan
string email = "zhangsan@qq.com";
int pos = email.find("@");
string Name = email.substr(0,pos);//此时Name就是"zhangsan"
```

## vector容器

1、vector与数组十分相似，也被称为单端数组
不同之处在于数组是静态空间，而vector可以动态扩展
务必注意动态扩展并不是在原空间后面接续新空间，而是找更大的内存空间，然后将原数据拷贝到新空间，并释放原空间。

2、vector容器构造

```c++
vector<int> v1; //默认无参构造

vector<int> v2(v1.begin(),v1.end()); //通过区间的方式构造

vector<int> v3(10,1); //n个elem方式构造 10个1

vector<int> v4(v3); //拷贝构造函数
```

3、vector容器赋值操作：=或assign

```c++
vector<int>v2;
v2 = v1;  //直接用=赋值

vector<int>v3;
v3.assign(v1.begin(),v1.end()); //用assign进行区间赋值 

vector<int>v4;
v4.assign(10,1); //用assign赋值 10个1
```

4、vector容器有关容量和大小的操作：

```c++
vector<int>v1;
v1.empty(); //为真表示v1为空
v1.capacity(); //表示vector容器的容量 即目前开辟的空间最多能放多少个元素
v1.size(); //v1里面目前有多少个元素
v1.resize(15); //重新制定大小为15，如果比v1.size()长 缺少的用0填充(默认)
v1.resize(15,1); //多指定一个参数，即缺少的用1进行填充
v1.resize(5); //重新指定长度短的话，超出的部分会被删除掉
```

5、vector的插入和删除

```c++
vector<int>v1;
v1.push_back(10); //尾插
v1.pop_back(); //尾删

v1.insert(v1.begin(),100); //在最开始的位置插入100(用迭代器表示位置)
v1.insert(v1.begin(),2,100); //多加一个参数，表示在最开始的位置插入2个100
v1.erase(v1.begin()); //删除迭代器开始的第一个元素
v1.erase(v1.begin(),v1.end()); //删除一个区间，这里是从头删除到尾 即全部删除，清空了
v1.clear(); //也表示清空~
```

6、vector数据存取操作：

```c++
vector<int>v1;
//从vector中取数据
v1[i]和v1.at(i)都表示v1中下标为i的元素
v1.front(); //v1中的第一个元素
v1.back(); //v1中的第二个元素
```

7、vector容器互换操作:实现两个容器内元素进行互换

```c++
vector<int>v1,v2;
v1.swap(v2); //实现了v1和v2内元素的互换
```

8、vector容器预留空间：reserve(int len)

```c++
vector<int> v1;
v1.reserve(10000); //预留10000的空间，防止一直开辟空间降低性能
```

## deque容器

1、deque容器是双端数组，对于数据的访问速度不如vector容器

2、deque容器的构造：

```c++
deque<int> d1; //默认构造
deque<int> d2(d1.begin(),d1.end());
deque<int> d3(10,100); //10个100
deque<int> d4(d3); //拷贝构造
```

3、deque容器的赋值操作：= 和 assign

```c++
deque<int> d1,d2,d3,d4;
d1 = d2; // = 赋值
d3.assign(d1.begin(),d1.end()); //assign赋值
d4.assign(10,100); //10个100
```

4、deque容器的大小操作：deque里没有capacity，因为可以无限的放数据，只需要给中控器加一个地址即可，不需要额外开辟空间~

```c++
deque<int> d1;
d1.empty(); //为真表示为空 为假表示不为空
d1.size(); //表示d1的大小
d1.resize(15); //比原来长的话会用0来填充
d1.resize(15,1); //比原来长的话用1来填充
d1.resize(5); //比原来段 超出的部分会被删除掉
```

5、deque中的插入和删除

```c++
deque<int> d1,d2;
//两端操作:front在左，back在右
//插入
d1.push_back(10); //尾插
d1.push_front(10); //头插
//删除
d1.pop_back(); //尾删
d1.pop_front(); //头删

//迭代器指定位置插入删除:insert,erase
//insert插入
d1.insert(d1.begin(),1000); //在迭代器表示的元素前插入元素1000，即在第一个元素之前插入1000
d1.insert(d1.begin(),2,1000); //在第一个元素之前插入2个1000
d1.insert(d1.begin(),d2.begin(),d2.end());//在第一个元素之前插入d2两个迭代器区间内的元素，即所有元素
//erase删除
d1.erase(d1.begin()); //删除第一个元素
d1.erase(d1.begin(),d1.end()); //删除给定区间内的元素

d1.clear(); //清空操作，只剩下一个换行符
```

6、deque容器的数据存取：

```c++
deque<int> d1;
//取数据
d1[i]和d1.at(i) 都表示下标为i的元素
d1.front(); //头部元素(最左边元素)
d1.back(); //尾部元素(最右边元素)
```

7、deque排序操作：sort

```c++
deque<int> d1;
sort(d1.begin(),d2.end()); //默认从小到大 升序排序
bool compare(int a,int b) {
    return a > b;
}
sort(d1.begin(),d1.end(),compare); //表示降序排序
```

## stack容器

1、stack是先进后出(FILO)的数据结构，只有一个出口。

2、栈不允许有遍历行为；可以判断是否为空；可以返回栈中元素个数

3、stack的常用接口：

```c++
stack<int> st;
st.push(10);
st.pop();
st.top();
st.empty();
st.size();
```

## queue容器

1、queue是先进先出(FIFO)的数据结构，有两个出口。

2、queue的back(队尾)在左边，front(队头)在右边，跟deque不一样；元素从队尾进(push)，队头出(pop)

3、队列只有队头和队尾能被外界访问，因此也不允许有遍历行为，但是可以返回队列中元素的个数

4、queue的常用接口

```c++
//构造函数
queue<int> q1; //默认构造
queue<int> q2(q1); //拷贝构造

q.empty(); //判空函数
q.size(); //返回队列的大小
q.front(); //队头元素(最右侧)
q.back(); //队尾元素(最左侧)
q.push(10); //从队尾(即back 左侧)添加一个元素
q.pop(); //出队，从队头出一个元素 即front(右侧出)

```

## list容器

1、list(链表)容器是一种物理存储单元上非连续的存储结构，数据元素的逻辑顺序是通过链表中的指针链接实现的。

2、list容器与数组相比：
优点：list可以对任意位置进行快速插入或删除元素；采用动态存储分配，不会造成内存浪费和溢出~
缺点：list容器遍历速度没有数组快，且占用空间比数组大(因为每个节点不仅存数据还要存指针)

3、STL中的list(链表)是双向循环链表，故list中的迭代器是双向迭代器，可以往前也可以往后。

4、由于list是链表，故不能指定访问第几个，所以list的迭代器不能说是begin()+3，只能是++或者--，即一个一个的走~

5、list容器构造：跟其他容器一样

```c++
list<int> L1; //默认构造
list<int> L2(L1.begin(),L1.end()); //区间构造
list<int> L3(L2); //拷贝构造
list<int> L4(10,100); //10个100
```

6、list容器赋值和交换操作：

```c++
list<int> L1,L2,L3,L4;
L1 = L2; //=赋值
L3.assign(L2.begin(),L2.end()); //assign赋值
L4.assign(10,100); //10个100

L1.swap(L2); //交换L1和L2
```

7、list容器大小操作:

```c++
list<int> L1;
L1.empty(); //判空操作
L1.size(); //元素个数
L1.resize(15); //重新指定的比原来长 少的用0来填充
L1.resize(15,1); //少的用1来填充
L1.resize(5); //指定的比原来短，多的部分剔除
```

8、list容器的插入和删除：

```c++
list<int> L;
L.push_back(10); //尾插 插在右边
L.push_front(20); //头插 插在左边
L.pop_back(); //尾删
L.pop_front(); //头删

L.insert(L.begin(),100); //在迭代器表示的位置之前插入100
L.erase(L.begin()); //删除迭代器所在位置的元素

L.remove(100); //移除L中的所有值为100的元素

L.clear(); //清空容器
```

9、list容器的数据存取:不支持[ ]和at的存取方式

```c++
list<int> L;
L.front(); //表示L中第一个元素
L.back(); //表示L中最后一个元素

list<int>::iterator it = L.begin();
//list迭代器只支持++和--
it = it + 1; //会报错 因为不支持随机访问
it++; //正确
```

10、list容器的反转和排序:

```c++
list<int> L1;
L1.reverse(); //对L1进行反转

sort(L1.begin(),L1.end()); //这是错的，因为所有不支持随机访问迭代器的容器，不可以用标准算法(alogrithm)
L1.sort(); //直接调用定义好的成员sort即可，默认是从小到大

bool myCompare(int a,int b) {
    return a > b;
}
L1.sort(myCompare); //降序排列，即从大到小
```

## set/ multiset容器

1、set/multiset容器所有元素在插入时都会被自动排序(升序)

2、set和multiset都属于关联式容器，底层结构是用二叉树实现

3、set和multiset的区别：
set不允许容器有重复的元素而multiset允许

4、set容器构造和赋值：

```c++
set<int> s1; //默认构造
set<int> s2(s1); //拷贝构造

s2 = s1; //=赋值
```

5、set容器大小和交换：

```c++
set<int>s1,s2;
s1.empty(); //判空操作
s1.size(); //s1的大小

s1.swap(s2); //s1和s2互换
```

6、set容器的插入和删除：insert和erase 没有push和pop

```c++
set<int> s1;
s1.insert(10); //插入10 会自动排序
s1.erase(s1.begin()); //删除迭代器所在位置的元素 返回下一个元素的迭代器
s1.erase(s1.begin(),s1.end()); //删除指定区间内的元素，左闭右开，并返回下一个元素的迭代器
s1.erase(30); //删除容器中所有值为30的元素
s1.clear(); //清空容器
```

7、set容器的查找和统计:find和count

```c++
set<int> s1;
s1.insert(10);
set<int>::iterator it = s1.find(10);//如果找到了就返回改元素的迭代器，找不到就返回s1.end()

int num = s1.count(10); //统计元素10的个数，对于set而言 结果为0或1，对multiset而言才可能>1
```

8、set的插入函数insert有返回值，是一个pair对组

```c++
set<int> s;
pair<set<int>::iterator,bool> ret = s.insert(10);
//ret.second为真表示插入成功 否则失败
//set在插入的时候会检测里面有没有一样的数据，如果有就插入失败为false 而multiset不会检测数据，因此可以插入重复数据 
```

9、pair对组的创建 ：两种方式

```c++
pair<string,int> p("Tom",20);//p.first和p.second取值

pair<string,int> p = make_pair("Jerry",30);
```

10、set容器手动指定排序规则:利用仿函数

```c++
//1、set存放内置数据类型(int之类的)
//因为set<>里面不能放一个函数名，要放一个数据类型，所以才提前定义一个类，也就是仿函数
class MyCompare {
public:
    //重载运算符(),方便让类的实例像函数一样被调用
    bool operator() (int a,int b){
        return a > b;
    }
}
//要在定义set容器的时候就指定排序规则 这样插入的时候就会自动按着定义的规则排序
set<int,MyCompare> s1; //s1变为从大到小插入

//2、set存放自定义数据类型
// 自定义数据类型都应该在定义的时候指定排序规则 不然编译器不知道咋去比较排序
class Person {
public:
    Person(string name,int age) {
        this->m_Name = name;
        this->m_Age = age;
    }
    string m_Name;
    int m_Age;
}
class compare {
public:
    bool operator()(const Person& p1,const Person& p2){
		return p1.m_Age > p2.m_Age; //按照年龄降序       
    }
}

set<Person,compare> s2;
Person p1("Tom",22);
Person p2("Jerry",33);
Person p3("Jack",44);
s2.insert(p1);
s2.insert(p2);
s2.insert(p3);


```

## map/multimap容器

1、map中所有的元素都是pair，其中pair的第一个元素为key(键值)，起到索引作用，第二个元素为value(实值)；可以根据key值快速找到value值~
所有插入的元素都会根据元素的键值自动排序

2、map/multimap属于关联式容器，底层结构是用二叉树实现

3、map不允许有重复的key值出现(注意是key值不重复，value值可以重复)，multimap允许~

4、map容器的构造和赋值

```c++
map<int,int> m1; //默认构造
map<int,int> m2(m1); //拷贝构造

map<int,int> m3;
m3 = m2; //=赋值
```

5、map容器大小和交换的操作:

```c++
map<int,int> m; 
m.empty(); //判空操作
m.size(); //m的大小

map<int,int>m1,m2;
m1.swap(m2); //交换m1和m2
```

6、map容器的插入和删除：

```c++
map<int,int> m;
//向map中插入数据 
m.insert(pair<int,int>(10,10)); //第一种
m.insert(make_pair(10,10)); //第二种
m.insert(map<int,int>::value_type(10,10)); //第三种 不常用
m[10] = 10; //第四种 map内部重载了[] 所以可以这样插入数据~
//如果m中不存在key为10的pair 那么m[10]自动就是0

//从map中删除数据
m.erase(m.begin()); //删除迭代器下标所在的元素
m.erase(10); //删除key值为10的数据
m.erase(m.begin(),m.end()); //删除给定区间的元素 左闭右开
m.clear(); //清空操作
```

7、map容器的查找和统计：find和count

```c++
map<int,int> m;
map<int,int>::iterator it = m.find(10); //返回的迭代器不是m.end()说明查找到了 是的话说明没找到
int num = m.count(10); // 返回key为10的键值对的个数，map里要么是0要么是1，multimap里才有可能>1
```

8、map容器自定义排序：默认是按照键值key从小到大

```c++
class MyCompare {
public:
    bool operator()(int,v1,int v2) {
        return v1 > v2; //按key降序
    }
}
map<int,int,MyCompare> m; //这样定义的map容器就是从大到小排序的~
```

对于自定义数据类型的排序，map必须要自己定义仿函数去指定排序规则~同set容器

## STL函数对象

1、重载了函数调用操作符的类，这个类的对象通常被称为函数对象~
函数对象在使用重载的"()"时，行为类似函数调用，也叫仿函数

2、函数对象(仿函数)本质是一个类，不是一个函数。

3、函数对象的特点：
1️⃣函数对象在使用时，可以像普通函数那样被调用，可以有参数和返回值
2️⃣函数对象也超出普通函数的概念，可以有自己的状态
3️⃣函数对象可以作为参数传递~

```c++
class MyAdd {
public:
	int operator()(int a,int b) { //可以有参数和返回值
        return a + b;
    }
}
MyAdd my_add; //函数对象
int sum = my_add(10,10); //函数对象像普通函数一样被调用

//函数对象是类，内部可以定义变量去记录自己的状态
```

## 谓词

1、返回bool类型的仿函数称为谓词

2、如果operator()接受一个参数，称为一元谓词；接受两个参数，称为二元谓词

```c++
class MyCompare {
public:
    bool operator()(int v1,int v2) { 
        return v1 > v2;
    }
} //这个仿函数就是二元谓词
vector<int> vec;
sort(vec.begin(),vec.end(),MyCompare());//类名后面加上括号就是匿名函数对象
```



## 内建函数对象

1、概念：STL内建好了一些函数对象，可以拿来就用，但是需要包含头文件:

```c++
#include<functional>
```

2、算数仿函数:

```c++
#include<functional>
void test01() {
    //negate取反仿函数，是一元仿函数
    negate<int> n; //n是创建的函数对象，可以当做普通函数一样调用
    cout << n(20) << endl; //输出的就是-20
    
    //plus是二元仿函数 加法
    plus<int> p;
    cout << p(10,10) << endl; //输出是20
}
```

3、关系仿函数：

```c++
class MyCompare {
public:
    bool operator()(int v1,int v2) {
        return v1 > v2;
    }
}
vector<int> vec;
sort(vec.begin(),vec.end(),MyCompare()); //我们自己定义是这样 事实上可以直接用人家给的函数 包含头文件functional即可
sort(vec.begin(),vec.end(),greater<int>()); //直接使用人家定义好的greater也是可以实现从大到小排序
```

4、逻辑仿函数：

```c++
#include<algorithm>
#include<functional>
vector<bool> v1,v2;
v2.resize(v1.size()); //注意先给v2足够的空间才方便搬运
transform(v1.begin(),v1.end(),v2.begin(),logical_not<bool>()); //对v1所有元素做逻辑非操作(!)然后赋值给v2
```

## 常用算法

```c++
//算法文件主要是由三个头文件组成
#include<algorithm> //是STL中最大的一个 设计比较 交换查找等
#include<functional> //定义了一些模版类，用以声明函数对象
#include<numeric> //体积很小 只包括在几个序列上面进行简单数学运算的模板函数
```

### 遍历算法

1、for_each：for_each(起始迭代器，中止迭代器，函数)
可以理解成，对迭代器区间内的每个元素都执行函数的操作~具体见下例

```c++
vector<int> v;
for(int i = 0; i < 10; ++i) {
    v.push_back(i);
}
void print01(int val) {
    cout << val << " ";
}
class print02 {
public:
    void operator()(int val) {
        cout << val << " ";
    }
}
for_each(v.begin(),v.end(),print01); //普通函数只需要传入函数名
for_each(v.begin(),v.end(),print02()); //仿函数需要传入函数对象，即要加一个括号表示匿名对象~
//二者都是输出1 2 3 4balabala
```

2、transform：搬运容器到另一个容器中
transform(源容器起始迭代器，源容器终止迭代器，目标容器起始迭代器，函数)

```c++
vector<int> v;
for(int i = 0; i < 10; ++i) {
    v.push_back(i);
}
vector<int>vTarget; //目标容器
class Transform {
public:
    int operator()(int v) {
        return v; //可以直接原样返回，即原封不动搬运，也可以对数据进行操作之后再搬运，比如return 100+v;
    }
}
vTarget.resize(v.size()); //记得提前给目标容器开辟空间，不然塞不进去了~
transform(v.begin(),v.end(),vTarget.begin(),Transform());
```

### 查找算法

1、find:查找指定元素 找到返回指定元素的迭代器，找不到返回结束迭代器
find(起始迭代器，终止迭代器，值);

```c++
//1、查找内置数据类型
vector<int> v;
for(int i = 0; i < 10; ++i) {
    v.push_back(i);
}
vector<int>::iterator it = find(v.begin(),v.end(),5);
//2、查找自定义数据类型,务必注意要在自定义的数据类型中重载==，这样才方便查找的时候对自定义数据类型的比对
class Person {
public:
    Person(string name,int age) {
        this->m_Name = name;
        this->m_Age = age;
    }
    bool operator==(const Person& p) {
		return this->m_Name==p.m_Name&&this->m_Age == p.m_Age;
    }
}
//后面的与内置数据类型的查找一样，不做赘述~
```

2、find_if：按条件查找元素
find_if(起始迭代器，终止迭代器，谓词);

```c++
vector<int> v;
for(int i = 0; i < 10; ++i) {
    v.push_back(i);
}
class GreaterFive {
public:
    bool operator()(int val) { //一元谓词
        return val > 5;
    }
}
vector<int>::iterator it = find_if(v.begin(),v.end(),GreaterFive()); //找到大于五的数字6
```

3、adjacent_find：查找相邻重复元素
adjacent_find(起始迭代器，终止迭代器); 返回第一个相邻重复元素的迭代器，没找到就返回end()

```c++
vector<int> v;
for(int i = 0; i < 10; ++i) {
    if(i%2 == 0) v.push_back(i);
    v.push_back(i);
} //偶数是两个连续的 001223445667889
vector<int>::iterator pos = adjacent_find(v.begin(),v.end());
//pos将指向第一个相邻重复元素0的位置.如果没有相邻重复元素,pos 将等于v.end(),表示没有找到相邻重复元素

```

4、binary_search：查找指定元素是否存在(序列有序的时候才能用这个搜索)
bool binary_search(起始迭代器，终止迭代器，值);  查到返回true 否则false

```c++
vector<int> v;
for(int i = 0; i < 10; ++i) {
    v.push_back(i);
}
bool ret = binary_search(v.begin(),v.end(),8);//注意v必须是有序的序列，否则结果就不准！
```

5、count：统计元素个数
count(起始迭代器，终止迭代器，值); 返回的是值的个数

```c++
vector<int> v;
for(int i = 0; i < 10; ++i) {
    if(i%2 == 0) v.push_back(i);
    v.push_back(i);
}
int num = count(v.begin(),v.end(),0); //返回值就是2

//如果是自定义数据类型 记得重载==
```

6、count_if：按条件统计元素个数
count_if(起始迭代器，终止迭代器，谓词); 返回值是在满足谓词条件的元素的个数~

```c++
vector<int> v;
for(int i = 0; i < 10; ++i) {
    if(i%2 == 0) v.push_back(i);
    v.push_back(i);
}
class Greater5 {
public: 
    bool operator()(int val) {
        return val > 5;
    }
}
int num = count_if(v.begin(),v.end(),Greater5());//返回大于5的元素的个数
```

### 排序算法

1、sort：对容器内元素进行排序
sort(起始迭代器，终止迭代器，谓词); 

```c++
vector<int> v;
for(int i = 0; i < 10; ++i) {
    v.push_back(i);
}
void myPrint(int val) {
    cout << val << " ";
}
sort(v.begin(),v.end(),myPrint); //升序输出
sort(v.begin(),v.end(),greater<int>()); //利用人家提供的仿函数，降序排列~

```

2、random_shuffle：洗牌算法，对指定范围内的元素进行随机调整次序
random_shuffle(起始迭代器，终止迭代器);

```c++
vector<int> v;
for(int i = 0; i < 10; ++i) {
    v.push_back(i);
}
srand((unsigned int)time(NULL)); //用的时候记得加上随机数种子 使得每次打乱的顺序都不一样~
random_shuffle(v.begin(),v.end());
```

3、merge：将两个有序序列合并，结果仍然是一个有序序列~
merge(容器一起始迭代器，容器一终止迭代器，容器二起始迭代器，容器二终止迭代器，目标容器起始迭代器);

```c++
vector<int> v1,v2;
for(int i = 0; i < 10; ++i) {
    v1.push_back(i); //0~9
    v2.push_back(i + 1); //1~10
}
vector<int> vTarget; //目标容器
vTarget.resize(v1.size()+v2.size()); //务必不要提前忘记分配空间 不然会报错
merge(v1.begin(),v1.end(),v2.begin(),v2.end(),vTarget.begin()); //将v1和v2两个容器合并起来放到vTarget容器中~
//最后vTarget为011223344556677889910 还是有序序列~
```

4、reverse：将容器内元素进行反转
reverse(起始迭代器，终止迭代器); 

```C++
vector<int> v;
for(int i = 0; i < 10; ++i) {
    v.push_back(i);
}
reverse(v.begin(),v.end());
```

### 拷贝和替换算法

1、copy：容器内指定范围元素拷贝到另一容器中
copy(源容器起始迭代器，源容器终止迭代器，目标容器起始迭代器);

```c++
vector<int> v1;
for(int i = 0; i < 10; ++i) {
    v1.push_back(i);
}
vector<int> v2;
v2.resize(v1.size()); //提前开辟空间！！
copy(v1.begin(),v1.end(),v2.begin());
```

2、replace:将容器内指定范围的旧元素修改为新元素
replace(起始迭代器，终止迭代器，旧元素，新元素);

```c++
vector<int> v;
for(int i = 0; i < 10; ++i) {
    v.push_back(i);
}
replace(v.begin(),v.end(),5,500); //把区间中所有的5都替换成500~
```

3、replace_if：将区间内满足条件的元素，替换成指定元素~
replace_if(起始迭代器，终止迭代器，谓词，新值);

```c++
vector<int> v1;
for(int i = 0; i < 10; ++i) {
    v1.push_back(i);
}
class Greater3 {
public:
    bool operator()(int val) {
        return val >= 3;
    }
}
replace_if(v1.begin(),v1.end(),Greater3(),300); //大于等于3的数替换成300
```

4、swap：互换两个容器中的元素
swap(容器1，容器2); 两个容器类型应该相同，比如都是vector

```c++
vector<int> v1,v2;
for(int i = 0; i < 10; ++i) {
    v1.push_back(i); //0~9
    v2.push_back(i + 10); //10~19
}
swap(v1,v2); //交换v1和v2内的元素
```

### 算术生成算法

这个的头文件要包含numeric

1、accumulate：计算区间内容器元素累计总和
accumulate(起始迭代器，终止迭代器，起始值); 从起始值开始累加

```c++
vector<int> v1;
for(int i = 0; i < 10; ++i) {
    v1.push_back(i);
}
int sum = accumulate(v1.begin(),v1.end(),0); //从0开始累加
```

2、fill：向容器内填充指定的元素
fill(起始迭代器，终止迭代器，指定值);在给定区间内填充指定值

```c++
vector<int> v;
v.resize(10); 

fill(v.begin(),v.end(),100);//填充了10个100
```

### 集合算法

1、set_intersection：求两个有序序列的交集
set_intersection(容器1起始迭代器，容器1终止迭代器，容器2起始迭代器，容器2终止迭代器，目标容器起始迭代器); 返回值是交集的最后一个元素的迭代器

```c++
vector<int> v1,v2;
for(int i = 0; i < 10; ++i) {
    v1.push_back(i); //0~9
    v2.push_back(i + 5); //5~14
}
vector<int> vTarget;
vTarget.resize(min(v1.size(),v2.size())); //取小容器的size即可
vector<int>::iterator itEnd = set_intersection(v1.begin(),v1.end(),v2.begin(),v2.end(),vTarget.begin());//返回值是交集最后一个元素的迭代器
```

2、set_union：求两个有序序列的并集
set_union(容器1起始迭代器，容器1终止迭代器，容器2起始迭代器，容器2终止迭代器，目标容器起始迭代器);返回值是并集的最后一个元素的迭代器

```c++
vector<int> v1,v2;
for(int i = 0; i < 10; ++i) {
    v1.push_back(i); //0~9
    v2.push_back(i + 5); //5~14
}
vector<int> vTarget;
vTarget.resize(v1.size()+v2.size());
vector<int>::iterator itEnd = set_union(v1.begin(),v1.end(),v2.begin(),v2.end(),vTarget.begin());//返回值是并集最后一个元素的迭代器
```

3、set_difference：求两个有序序列的差集
set_difference(容器1起始迭代器，容器1终止迭代器，容器2起始迭代器，容器2终止迭代器，目标容器起始迭代器);返回值是差集的最后一个元素的迭代器

```c++
vector<int> v1,v2;
for(int i = 0; i < 10; ++i) {
    v1.push_back(i); //0~9
    v2.push_back(i + 5); //5~14
}
vector<int> vTarget;
vTarget.resize(max(v1.size(),v2.size()));
vector<int>::iterator itEnd = set_difference(v1.begin(),v1.end(),v2.begin(),v2.end(),vTarget.begin());//返回值是差集最后一个元素的迭代器
```

