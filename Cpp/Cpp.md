# 变量

## 修饰符
### friend
含有友元声明的函或类可以访问类的private和protected成员，该函数并不是成员函数
```cpp
class MyClass{
	private:
	int var;
	public:
	friend void print(const MyClass& myClass);
}

void print(const MyClass& myClass){
	std::cout<<myClass.var;
}
```
### const
成员函数后的const修饰承诺该函数不修改成员变量
```cpp
class MyClass{
	private:
	int var;
	public:
	int getVar()const{
		return var;
	};	
};
```
### constexpr
编译时常量，在编译时计算值，提升运行效率
```cpp
constexpr int b=20;
int x=5;
const int c=x;//合法，运行时常量
constexpr int d=x;//错误，x不是常量表达式

constexpr int factorial(int n){
	return n<=1?1:n*factorial(n-1);
}
constexpr int result=factorial(5);//编译时计算，结果为120
```
### mutable
被mutable修饰的成员变量可以在const对象中被修改
### volatile
修饰的变量可能会被改变，防止编译优化造成错误
### static
定义静态变量
### noexcept
在函数后修饰，表示该函数不会抛出异常
```cpp
void func()noexcept{
	..
}
```
### inline
被修饰的函数在编译时会将函数体直接插入调用处，提高运行效率
### explicit
用于修饰**单参数构造函数**和**转换运算符**，防止编译器进行隐式类型转换。
## 引用
### 左值引用（有址引用）
```cpp
int x = 10;
int& ref1 = x;        // 正确：绑定到左值
// int& ref2 = 10;    // 错误：不能绑定到右值

const int& ref3 = 10; // 正确：const左值引用可以绑定到右值
const int& ref4 = x;  // 正确：也可以绑定到左值
```
### 右值引用（无址引用）
```cpp
int x = 10;
int&& ref1 = 10;           // 正确：绑定到右值
int&& ref2 = x + 5;        // 正确：绑定到表达式结果
// int&& ref3 = x;         // 错误：不能绑定到左值

int&& ref4 = std::move(x); // 正确：move将左值转为右值引用
```

## 类型推导与转换
### 隐式转换
编译器自动进行的类型转换：
```cpp
int a = 10;
double b = a;  // 隐式转换：int → double

float f = 3.14f;
double d = f;  // 隐式转换：float → double

char c = 'A';
int i = c;     // 隐式转换：char → int
```
### 显式转换
程序员明确指定的类型转换：
```cpp
double d = 3.14159;
int a = (int)d;        // C风格：double → int
int b = int(d);        // 函数风格：double → int

float f = 2.5f;
int c = static_cast<int>(f);  // C++风格
```
### cast系列转换
#### static_cast
1. 基本类型之间的转换，如从int到double，从enum到int等。但是 如果转换是 narrowing（如从double到int），可能会丢失信息。 
2. 将指针或引用从派生类向上转换到基类（这是安全的，因为派 生类对象包含基类部分）。 
3. 将指针或引用从基类向下转换到派生类，但是在这种情况下， 使用static_cast是不安全的，因为编译器无法在运行时检查转换 的正确性。通常在这种情况下，推荐使用dynamic_cast（如果基 类有虚函数）。 
4. 任意类型与void* 之间的转换，但是需要确保转换是安全的。 
5. 使用用户定义的类型转换函数进行转换，如果类定义了转换函 数，那么static_cast可以调用这些函数。 
6. 将非const对象转换为const对象，这是安全的。 7.将左值转换为右值引用，例如用于实现移动语义。
#### const_cast
用于修改类型的const或volatile属性：
```cpp
const int a=0;
*const_cast<int*>(&a)=20;
const_cast<int&>(a)=20;

*static_cast<int*>(&a)=20;//无法从const int*转为int*
static_cast<int&>(a)=20;//无法从const int转为int&
```
#### dynamic_cast
```cpp
class Base{public: virtual ~Base(){}};
class Derived:public Base{};
Base* base=new Derived();

Derived* derived=dynamic_cast<Derived*>(base);//安全的向下转换
//从上往下转换，若基类无虚函数则编译报错：基类不具有多态

Base* p=dynamic_cast<Base>(derived);//从下往上的转换总是安全的
```
#### reinterpret_cast
低级别的重新解释转换
```cpp
// 不同类型指针的转换
int* ip = new int(65);
char* cp = reinterpret_cast<char*>(ip);

// 指针转整数
int* ptr = new int(100);
uintptr_t int_val = reinterpret_cast<uintptr_t>(ptr);
std::cout << "Pointer as integer: " << int_val << std::endl;

// 整数转指针
uintptr_t address = 0x12345678;
int* new_ptr = reinterpret_cast<int*>(address);

// 函数指针类型转换
void func1() { std::cout << "Function 1" << std::endl; }
void func2() { std::cout << "Function 2" << std::endl; }
void (*fp1)() = func1;
void (*fp2)() = reinterpret_cast<void(*)()>(fp1);
fp2();  // 调用 func1

// 查看浮点数的二进制表示
float f = 3.14f;
int* int_view = reinterpret_cast<int*>(&f);
std::cout << "Float bits: " << std::hex << *int_view << std::endl;
```
### typeid
```cpp
typeid(表达式)  // 返回 std::type_info 对象的引用
typeid(类型)    // 返回类型的 std::type_info 对象引用
cout<<typeid(a).name(); // 获取类型信息
if(typeid(a)==typeid(b)) // 类型比较
```
#### std::type_info类
```cpp
#include <typeinfo>
#include <iostream>
#include <vector>
class MyClass {};
int main() {
    // name() - 返回类型名称（实现定义格式）
    std::cout << "类型名称: " << typeid(int).name() << std::endl;
    std::cout << "类型名称: " << typeid(std::vector<int>).name() << std::endl;
    
    // operator== 和 operator!= - 类型比较
    if (typeid(int) == typeid(double)) {
        std::cout << "int 和 double 类型相同" << std::endl;
    } else {
        std::cout << "int 和 double 类型不同" << std::endl;
    }
    
    // before() - 类型排序（用于在map中作为key）
    if (typeid(int).before(typeid(double))) {
        std::cout << "int 在 double 之前" << std::endl;
    } else {
        std::cout << "double 在 int 之前" << std::endl;
    }
    
    // hash_code() - C++11，返回类型的哈希值
    std::cout << "int 的哈希值: " << typeid(int).hash_code() << std::endl;
    std::cout << "double 的哈希值: " << typeid(double).hash_code() << std::endl;
    
    return 0;
}
```

### 自动类型推导auto，decltype
# 函数
## 可变参数
### C风格
```cpp
#include <iostream>
#include <cstdarg>

// 简单求和函数
int sum(int count, ...) {
    va_list args;
    va_start(args, count);
    
    int total = 0;
    for (int i = 0; i < count; ++i) {
        total += va_arg(args, int);
    }
    
    va_end(args);
    return total;
}

// 使用
int main() {
    std::cout << sum(3, 1, 2, 3) << std::endl;        // 输出6
    std::cout << sum(5, 1, 2, 3, 4, 5) << std::endl;  // 输出15
    return 0;
}
```
相关宏：
- va_list：参数列表类型
- va_start：初始化参数列表，第二个参数为函数的第一个参变量
- va_arg：获取下一个参数，第二个参数为变量类型
- va_end：清理参数列表
实际传参时，省略的参数会与第一个参数在地址上连续传入，即&count+1为第二个参数的地址
### C++可变模板
```cpp
#include <iostream>

// 基准情况
void print() {
    std::cout << std::endl;
}

// 递归情况
template<typename T, typename... Args>
void print(T first, Args... args) {
    std::cout << first;
    if (sizeof...(args) > 0) {
        std::cout << ", ";
    }
    print(args...);  // 递归调用
}

// 使用
int main() {
    print(1, 2.5, "hello", 'A');  // 输出: 1, 2.5, hello, A
    return 0;
}
```
## 函数重载
名字相同，参数列表不同
## inline函数和constexpr函数
# 类
## 声明
```cpp
struct StructName{
	ElementType element;
	..
};

class ClassName{
	ElementType element;
	..
};
```
struct默认访问权限为public，class默认为private
## 访问权限
```cpp
class ClassName{
	private:
	..//私有成员，外部不可访问
	public:
	..//公有成员，外部可以访问
	protected:
	..//保护成员，外部不可访问，非私有派生类可以访问
};
```
## 构造函数
编译器默认提供三种构造函数
```cpp
class ClassName{
	public:
	ClassName(){//默认无参构造
	..
	}
	ClassName(const ClassName& other){//默认复制构造
	..
	}
	ClassName(ClassName&& other){//默认移动构造
	..
	}
};
```
## 析构函数
```cpp
class ClassName{
	~ClassName(){
	..
	}
};
```
析构函数在对象声明周期结束时自动调用，也可手动调用
## 运算符重载
```cpp
class MyClass{
	//赋值
	MyClass& operator=(const MyClass&){..return *this;}//拷贝赋值
	MyClass& operator=(MyClass&&)noexcept{..return *this;}//移动赋值
	MyClass& operator+=(const MyClass&){..return *this;}//复合赋值
	
	//二目 +-*/%
	MyClass operator+(const MyClass&){..return *this;}
	
	//比较 ==,!=,>,<,>=,<=
	bool operator==(const MyClass&)const{..return ..;}
	auto operator<=>(const MyClass&)const=default;//C20方法，自动生成六种比较函数
	
	//下标
	ElementType& operator[](size_t pos){return ..;}//非常量版本
	ElementType operator[](size_t pos)const{return ..;}//常量版本
	
	//函数调用
	ReturnType operator()(type1 ele1,type2 ele2,..){}
	
	//自增自减
	MyClass& operator++(){..return *this;}//前缀
	MyClass& operator++(int){//后缀，int为伪参数
		MyClass temp=*this;
		++(*this);
		return temp;//返回旧值
	}
	
	//流运算
	friend std::ostream operator<<(std::ostream& out,const MyClass& myClass)const{
		out<<..;
		return out;
	}
	friend std::istream operator>>(std::istream& in,MyClass& myClass){
		in>>..;
		return in;
	}
	
	//类型转换
	explicit int()const{return..;}//使用explicit避免意外的隐式转换
	
	//成员访问
	ElementType& operator*()const{return *ptr;}//解引用
	ElementType* operator->()const{return ptr;}//应返回指针对象
	
	//内存管理
	static void* operator new(){return ..;}
	static void* operator new[](size_t size){return ..;}
	static void operator delete()noexcept{}
	static void operator delete[]()noexcept{}
};
```
## 静态成员
局部类不能定义静态数据成员
### 静态数据成员
静态成员独立于类存在，不同对象的静态数据成员共享同一空间，对象的存储空间中不含静态数据成员
### 静态函数成员
无对象的地址，即无隐含参数 this；只能访问静态成员，调用静态函数成员
## 成员指针
### 成员变量指针
```cpp
#include <iostream>
#include <string>

class Person {
public:
    std::string name;
    int age;
    double salary;
};

int main() {
    // 定义成员变量指针
    std::string Person::*name_ptr = &Person::name;
    int Person::*age_ptr = &Person::age;
    
    Person person;
    person.name = "Alice";
    person.age = 25;
    
    // 通过对象使用成员指针
    std::cout << person.*name_ptr << std::endl;  // 输出: Alice
    std::cout << person.*age_ptr << std::endl;   // 输出: 25
    
    // 通过对象指针使用成员指针
    Person* pPerson = &person;
    std::cout << pPerson->*name_ptr << std::endl;  // 输出: Alice
    std::cout << pPerson->*age_ptr << std::endl;   // 输出: 25
    
    return 0;
}
```
### 成员函数指针
```cpp
#include <iostream>
#include <string>

class Calculator {
public:
    int add(int a, int b) { return a + b; }
    int multiply(int a, int b) { return a * b; }
    void print(const std::string& msg) { 
        std::cout << "Calculator: " << msg << std::endl; 
    }
};

int main() {
    // 定义成员函数指针
    int (Calculator::*operation)(int, int) = &Calculator::add;
    
    Calculator calc;
    Calculator* pCalc = &calc;
    
    // 通过对象调用
    int result1 = (calc.*operation)(3, 4);      // 输出: 7
    std::cout << "Result: " << result1 << std::endl;
    
    // 通过对象指针调用
    int result2 = (pCalc->*operation)(5, 6);    // 输出: 11
    std::cout << "Result: " << result2 << std::endl;
    
    // 切换函数指针
    operation = &Calculator::multiply;
    int result3 = (calc.*operation)(3, 4);      // 输出: 12
    std::cout << "Result: " << result3 << std::endl;
    
    // void函数指针
    void (Calculator::*print_func)(const std::string&) = &Calculator::print;
    (calc.*print_func)("Hello World");
    
    return 0;
}
```
### 静态成员指针
使用于普通指针一致
## 派生与继承
### 单继承类
```cpp
class Base{
	protected:
	int x;
	public:
	Base(int x):x(x){}
	~Base(){}
	...
};
class ClassName:public Base final{
	private:
	int y,z;
	public:
	ClassName(int x,int y,int z):Base(x),y(y),z(z){}
	~ClassName(){}
};
```
不继承构造函数和析构函数
### 多继承类
```cpp
struct A{A(){cout<<"A";}};
struct B{const A a;B(){cout<<"B";}};
struct C{C(){cout<<"C";}};
struct D{D(){cout<<"D";}};
struct E:A{E(){cout<<"E";}};
struct F:B,virtual C{F(){cout<<"F";}};
struct G:B{G(){cout<<"G";}};
struct H:virtual C,virtual D{H(){cout<<"H";}};
struct I:E,F,virtual G,H{E e;F f;I(){cout<<"I";}};
```
构造树
![[多继承树.svg]]
#### 构造顺序
对于每个派生类：
1. **虚基类优先**：所有虚基类按照它们在继承列表中出现的顺序构造
2. **非虚基类次之**：然后是非虚基类按照继承顺序构造
3. **成员对象**：接着是类中成员对象按照声明顺序构造
4. **最终派生类**：最后执行派生类自身的构造函数体

根据构造树先序遍历，先构造虚基类，再构造非虚基类，然后构造成员，最后执行派生类构造函数
上例中类I构造顺序为：
C ABG D A E AB F H AE CABF I
#### 析构顺序
析构顺序与构造顺序相反，上例类I析构顺序为
I FBAC EA H F BA E A D GBA C

### 虚函数
#### 实现机制
C++通过一种称为 **虚函数表** 的机制来实现运行时多态。

- **虚函数表（vtable）：** 编译器会为每一个 **包含虚函数的类**（或者从包含虚函数的类派生而来的类）生成一个虚函数表。这是一个函数指针数组，其中存放着该类所有虚函数的地址。
- **虚函数指针（vptr）：** 编译器还会在 **每个该类对象的内部** 自动添加一个隐藏的指针成员（vptr）。这个vptr在对象构造时被初始化，指向该对象所属类的虚函数表。

**调用过程：**  
当一个虚函数通过基类指针（或引用）被调用时，程序会：

1. 通过对象的vptr找到其类的vtable。
2. 在vtable中找到该虚函数对应的条目（通常是基于函数在声明时的顺序）。
3. 通过该条目中的函数指针，调用正确的函数。

这个过程发生在运行时，因此被称为 **动态绑定** 或 **晚期绑定**。

#### 虚析构函数
当使用基类指针删除一个派生类对象时，如果基类的析构函数不是虚的，那么只会调用基类的析构函数，而不会调用派生类的析构函数。
如果一个类打算被其他类继承（即作为基类），那么它的析构函数**必须是虚函数**。

# 异常与断言

## 异常处理
```cpp
try{
	... throw ...;
}
catch(Derived d){
}
catch(Base b){//父类可以捕获子类异常，故子类异常应在父类之前捕获
	throw;//catch中无参数的throw可以重新向上级抛出当前异常
}
catch(const volatile void*){//捕获任意指针类型
}
catch(...){//捕获任意类型
}
```

## 异常类型
C++标准异常类型
```text
std::exception
├── std::logic_error
│   ├── std::invalid_argument
│   ├── std::domain_error
│   ├── std::length_error
│   └── std::out_of_range
├── std::runtime_error
│   ├── std::range_error
│   ├── std::overflow_error
│   ├── std::underflow_error
│   └── std::system_error
└── std::bad_alloc
    └── std::bad_array_new_length
```

自定义异常类型
```cpp
#include <exception>
#include <string>

class FileNotFoundException : public std::exception {
private:
    std::string filename;
    std::string full_message;
public:
    FileNotFoundException(const std::string& file) 
        : filename(file) {
        full_message = "File not found: " + filename;
    }
    const char* what() const noexcept override {
        return full_message.c_str();
    }
    const std::string& getFilename() const {
        return filename;
    }
};
```

# 函数式

## lambda表达式
```cpp
[capture](parameters) -> return_type {
    // 函数体
}
```
### 实现机制
编译时创建了一个匿名类的对象，并重载了operator()运算

### 捕获列表
```cpp
auto lambda1 = [x]()mutable{ // 值捕获，mutable声明表示可以修改捕获对象
	x += 5; // 不会修改外部变量，改变的是匿名类的成员变量
    return x + 5;  // x 被复制到lambda中
};
auto lambda2 = [x](){ // 没有mutable声明相当于const
	x=5; // 错误，不可修改
}
auto lambda3 = [&y]() { // 引用捕获
    y += 10;  // 直接修改外部变量y
};
auto lambda4 = [=]() { return a + b; };  // 值捕获所有外部变量
auto lambda5 = [&]() { a++; b++; };      // 引用捕获所有外部变量
auto lambda6 = [=, &n]() {  // 值捕获所有，但n是引用捕获
    return m + n;
};
```

# 模板
