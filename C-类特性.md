---
title: C++类特性
date: 2018-11-09 08:18:44
toc: true
categories: C/C++
---

### 访问限定符说明
1. public修饰的成员在类外可以直接被访问
2. protected和private修饰的成员在类外不能直接被访问(此处protected和private是类似的)
3. 访问权限作用域从该访问限定符出现的位置开始直到下一个访问限定符出现时为止
4. class的默认访问权限为private，struct为public(因为struct要兼容C)
注意：访问限定符只在编译时有用，当数据映射到内存后，没有任何访问限定符上的区别

**C++的class与C的struct的区别**：C++需要兼容C语言，所以C++中struct可以当成结构体去使用。另外C++中struct还可以用来定义类。和class是定义类是一样的，区别是struct的成员默认访问方式是public，class是struct的成员默认访问方式是private
### 类的大小的计算
一个类的大小，实际就是该类中<kbd>成员变量</kbd>之和，当然也要进行内存对齐，注意空类的大小，空类比较特殊，**编译器给了空类一个字节来唯一标识这个类**。

**结构体内存对齐规则:**
1. 第一个成员在与结构体偏移量为0的地址处。
2. 其他成员变量要对齐到某个数字（对齐数）的整数倍的地址处。
注意：对齐数 = 编译器默认的一个对齐数 与 该成员大小的较小值。
VS中默认的对齐数为8，gcc中的对齐数为4
3. 结构体总大小为：最大对齐数（所有变量类型最大者与默认对齐参数取最小）的整数倍。
4. 如果嵌套了结构体的情况，嵌套的结构体对齐到自己的最大对齐数的整数倍处，结构体的整体大小就是所有最大对齐数（含嵌套结构体的对齐数）的整数倍。


### this指针的特性
1. this指针的类型：类类型* const
2. 只能在“成员函数”的内部使用
3. this指针本质上其实是一个成员函数的形参，是对象调用成员函数时，将对象地址作为实参传递给this形参。所以对象中不存储this指针。
4. this指针是成员函数第一个隐含的指针形参，一般情况由编译器通过ecx寄存器自动传递，不需要用户传递



### 类的6个默认成员函数
#### 构造函数
构造函数是一个特殊的成员函数，名字与类名相同,创建类类型对象时由编译器自动调用，保证每个数据成员都有 一个合适的初始值，并且在对象的生命周期内只调用一次！
1. 构造函数是特殊的成员函数，需要注意的是，构造函数的虽然名称叫构造，但是需要注意的是构造函数的主要任务并不是开空间创建对象，而是初始化对象。
2. 如果类中没有显式定义构造函数，则C++编译器会自动生成一个无参的默认构造函数，一旦用户显式定义编译器将不再生成！
3. 无参的构造函数和全缺省的构造函数都称为默认构造函数，并且默认构造函数只能有一个。注意：无参构造函数、全缺省构造函数、我们没写编译器默认生成的构造函数，都可以认为是默认成员函数。
4. 编译器生成默认的构造函数会对自定类型成员调用的它的默认成员函数

```cpp
#include<iostream>
using namespace std;
class Time{
public:
	Time(){
		cout << "Time()" << endl;
		_hour = 0;
		_minute = 0;
		_second = 0;
	}
private:
	int _hour;
	int _minute;
	int _second;
};
class Date{
private:
	// 基本类型(内置类型)
	int _year;
	int _month;
	int _day;
	// 自定义类型
	Time _t;
};
int main(){
	Date d;
	return 0;
}
```

#### 析构函数
与构造函数功能相反，析构函数不是完成对象的销毁，局部对象销毁工作是由编译器完成的。而对象在销毁时会自动调用析构函数，完成类的一些资源清理工作
一个类有且只有一个析构函数。若未显式定义，系统会自动生成默认的析构函数。
对象生命周期结束时，C++编译系统系统自动调用析构函数
####  拷贝构造函数
拷贝构造函数也是特殊的成员函数，其特征如下：
1. 拷贝构造函数是构造函数的一个重载形式。
2. <font color='red'>拷贝构造函数的参数只有一个且必须使用引用传参，使用传值方式会引发无穷递归调用</font>

若未显示定义，系统生成默认的拷贝构造函数。 默认的拷贝构造函数对象按内存存储按字节序完成拷贝，这种拷贝我们叫做浅拷贝，或者值拷贝！

#### 赋值运算符重载
##### 运算符重载
C++为了增强代码的可读性引入了运算符重载，运算符重载是具有特殊函数名的函数，也具有其返回值类型，函数名字以及参数列表，其返回值类型与参数列表与普通的函数类似。
函数名字为：关键字operator后面接需要重载的运算符符号。
函数原型：返回值类型 operator操作符(参数列表)
注意：
* 不能通过连接其他符号来创建新的操作符：比如operator@
* 重载操作符必须有一个类类型或者枚举类型的操作数
* 用于内置类型的操作符，其含义不能改变，例如：内置的整型+，不能改变其含义
* <font color='red'>作为类成员的重载函数时，其形参看起来比操作数数目少1成员函数的操作符有一个默认的形参this，限定为第一个形参!</font>
* `.*` 、`::` 、`sizeof `、`?: `、`.` 注意以上5个运算符不能重载。

##### 赋值运算符重载
* 检测是否自己给自己赋值
* 返回*this
* 一个类如果没有显式定义赋值运算符重载，编译器也会生成一个，完成对象按字节序的值拷贝。

#### const成员
##### const修饰类的成员函数
将const修饰的类成员函数称之为const成员函数，const修饰类成员函数，实际修饰该成员函数隐含的this指针，表明在该成员函数中不能对类的任何成员进行修改！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181110104740774.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM4MDMyOTQy,size_16,color_FFFFFF,t_70)
1. const对象可以调用非const成员函数吗？不能
2. 非const对象可以调用const成员函数吗？可以
3. const成员函数内可以调用其它的非const成员函数吗？不能
4. 非const成员函数内可以调用其它的const成员函数吗？可以
5. 总结：<font color='red'>const只能使用const的东西，非const都可以使用！</font>

#### 取地址及const取地址操作符重载
这两个默认成员函数一般不用重新定义 ，编译器默认会生成！
这两个运算符一般不需要重载，使用编译器生成的默认取地址的重载即可，只有特殊情况，才需要重载，比如想让别人获取到指定的内容！

###  再谈构造函数
#### 构造函数体赋值
在创建对象时，编译器通过调用构造函数，给对象中各个成员变量一个合适的初始值
```cpp
class Date{
public:
 	Date(int year, int month, int day){
 		_year = year;
 		_month = month;
 		_day = day;
 	}

private:
 	int _year;
 	int _month;
	int _day;
};
```
虽然上述构造函数调用之后，对象中已经有了一个初始值，但是不能将其称作为类对象成员的初始化，构造函数体中的语句只能将其称作为赋初值，而不能称作初始化。因为初始化只能初始化一次，而构造函数体内可以多次赋值。
#### 初始化列表
初始化列表：以一个冒号开始，接着是一个以逗号分隔的数据成员列表，每个"成员变量"后面跟一个放在括号中的初始值或表达式。
```cpp
class Date{
public:
	Date(size_t year, size_t month, size_t day) :
		_year(year),
		_month(month),
		_day(day){
		
		this->_year = 2018;//这只能叫做赋初值
	}
private:
	size_t _year;
	size_t _month;
	size_t _day;
};
```
注意：
* 每个成员变量在初始化列表中只能出现一次(初始化只能初始化一次)
* <font color='red'>类中包含以下成员，必须放在初始化列表位置进行初始化：
	- 引用成员变量
	- const成员变量
	- 类类型成员(该类没有默认构造函数）
* <font color='red'>尽量使用初始化列表初始化，因为不管你是否使用初始化列表，对于自定义类型成员变量，一定会先使用初始化列表初始化</font>
* </font color='red'>成员变量在类中声明次序就是其在初始化列表中的初始化顺序，与其在初始化列表中的先后次序无关
####  explicit关键字
构造函数不仅可以构造与初始化对象，对于单个参数的构造函数，还具有类型转换的作用。
```cpp
#include <iostream>
using namespace std;

class Date{
public:
	Date(int year) :
		_year(year){
	}
private:
	size_t _year;
};

int main(){
	Date d1(2017);
	Date d2 = 2018;//在这里编译器默认构造一个匿名对象，然后将这个匿名对象赋值给d2
	return 0;
}
```
用explicit修饰构造函数，将会禁止单参构造函数的隐式转换，`Date d2 = 2018；`这样的写法就是无法编译通过的！

###  static成员
声明为static的类成员称为类的静态成员，用static修饰的成员变量，称之为静态成员变量；用static修饰的成员函数，称之为静态成员函数。<font color='red'>静态的成员变量一定要在类外进行初始化！</font>
1. 静态成员为所有类对象所共享，不属于某个具体的实例
2. 静态成员变量必须在类外定义，定义时不添加static关键字
3. 类静态成员即可用类名::静态成员或者对象.静态成员来访问
4. 静态成员函数没有隐藏的this指针，不能访问任何非静态成员(必须非私有)
5. 静态成员和类的普通成员一样，也有public、protected、private3种访问级别，也可以具有返回值，const修饰符等参数！

 静态成员函数可以调用非静态成员函数吗？不能
 非静态成员函数可以调用类的静态成员函数吗？可以
<font color='red'> 静态只能调用静态的函数或变量，非静态访问一切</font>
### C++11 的成员初始化新玩法
非静态成员变量，可以在成员声明时，直接初始化。
```cpp
class A{
private:
	 // 非静态成员变量，可以在成员声明时，直接初始化。
	 int a = 10;
	 int* p = (int*)malloc(4);
	 static int n;
};
int A::n = 10;
```
###  友元
#### 友元函数
假设现在我们尝试去重载operator<<，然后发现我们没办法将operator<<重载成成员函数。因为cout的输出流对象和隐含的this指针在抢占第一个参数的位置。this指针默认是第一个参数也就是左操作数了。但是实际使用中cout需要是第一个形参对象，才能正常使用。所以我们要将operator<<重载成全局函数。但是这样的话，又会导致类外没办法访问成员，那么这里就需要友元来解决。

```cpp
#include <iostream>

using namespace std;
class Date{
friend ostream& operator<<(ostream& out, const Date& d);
public:
	Date(int y, int m, int d) :
		year(y),
		month(m),
		day(d){

	}
private:
	int year;
	int month;
	int day;
};

ostream& operator<<(ostream& out, const Date& d){
	out << d.year << ":" << d.month << ":" << d.day;
	return out;
}
int main(){
	Date d(2018, 11, 10);
	cout << d << endl;;
	return 0;
}
```
友元函数可访问类的私有成员，但不是类的成员函数
友元函数不能用const修饰
友元函数可以在类定义的任何地方声明，不受类访问限定符限制
一个函数可以是多个类的友元函数
友元函数的调用与普通函数的调用和原理相同

#### 友元类
* 友元类的所有成员函数都可以是另一个类的友元函数，都可以访问另一个类中的非公有成员。
* 友元关系是单向的，不具有交换性。
比如上述Time类和Date类，在Time类中声明Date类为其友元类，那么可以在Date类中直接访问Time
* 类的私有成员变量，但想在Time类中访问Date类中私有的成员变量则不行。
* 友元关系不能传递，如果B是A的友元，C是B的友元，则不能说明C时A的友元。

```cpp
class Date; // 前置声明
class Time{
	friend class Date; // 声明日期类为时间类的友元类，则在日期类中就直接访问Time类中的私有成员变量
public:
	Time(int hour, int minute, int second)
		: _hour(hour)
		, _minute(minute)
		, _second(second)
	{	}

private:
	int _hour;
	int _minute;
	int _second;
};
class Date{
public:
	Date(int year = 1900, int month = 1, int day = 1)
		: _year(year)
		, _month(month)
		, _day(day)
	{	}

	void SetTimeOfDate(int hour, int minute, int second){
		// 直接访问时间类私有的成员变量
		_t._hour = hour;
		_t._minute = minute;
		_t.second = second;
	}

private:
	int _year;
	int _month;
	int _day;
	Time _t;
};
```

### 内部类
如果一个类定义在另一个类的内部，这个内部类就叫做内部类。注意此时这个内部类是一个独立的类，它不属于外部类，更不能通过外部类的对象去调用内部类。外部类对内部类没有任何优越的访问权限
内部类就是外部类的友元类。注意友元类的定义，内部类可以通过外部类的对象参数来访问外部类中的所有成员。但是外部类不是内部类的友元
1. 内部类可以定义在外部类的public、protected、private都是可以的。
2. 注意内部类可以直接访问外部类中的static、枚举成员，不需要外部类的对象/类名。
3. **sizeof(外部类)=外部类，和内部类没有任何关系**。

```cpp
class A{
private:
	static int k;
	int h;
public:
	class B{
	public:
		void foo(const A& a)
		{
			cout << k << endl;//OK
			cout << a.h << endl;//OK
		}
	};
};
int A::k = 1;
int main()
{
	A::B b;
	b.foo(A());

	return 0;
}
```
### 再谈三大基本特性：封装，继承，多态
#### 封装
封装就是把客观事物封装成抽象的类，并且类可以把自己的数据和方法只让可信的类或者对象操作，对不可信的进行信息隐藏。一个类就是一个封装了数据以及操作这些数据的代码的逻辑实体。在一个对象内部，某些代码或某些数据可以是私有的，不能被外界访问。通过这种方式，对象对内部数据提供了不同级别的保护，以防止程序中无关的部分意外的改变或错误的使用了对象的私有部分。

#### 继承
让某个类型的对象获得另一个类型的对象的属性的方法。它支持按级分类的概念。继承是指这样一种能力：它可以使用现有类的所有功能，并在无需重新编写原来的类的情况下对这些功能进行扩展。 通过继承创建的新类称为“子类”或“派生类”，被继承的类称为“基类”、“父类”或“超类”。继承的过程，就是从一般到特殊的过程。要实现继承，可以通过 “继承”（Inheritance）和“组合”（Composition）来实现。继承概念的实现方式有二类：实现继承与接口继承。实现继承是指直接使用 基类的属性和方法而无需额外编码的能力；接口继承是指仅使用属性和方法的名称、但是子类必须提供实现的能力。

#### 多态
多态是指一个类实例的相同方法在不同情形有不同表现形式。多态机制使具有不同内部结构的对象可以共享相同的外部接口。这意味着，虽然针对不同对象的具体操作不同，但通过一个公共的类，它们（那些操作）可以通过相同的方式予以调用!