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
## 派生与继承
