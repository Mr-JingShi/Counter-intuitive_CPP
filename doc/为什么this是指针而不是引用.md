# 为什么this是指针而不是引用

java、c#等语言的this被设计成了引用，而c++的this却被设计成了指针，且c++的this指针有别于其他普通指针：this指针在语言层面被设计为永远指向被操作的对象（this指针永远不为空），这使的this指针更加倾向于c++的引用，c++把this设计成指针而不是引用，有点反直觉。

# 奇怪的知识增加了
答案很简单：当this被添加到c++中时，引用还不存在。
> Why is "this" not a reference?
>
> Because "this" was introduced into C++ (really into C with Classes) before references were added. Also, I chose "this" to follow Simula usage, rather than the (later) Smalltalk use of "self".

[Why is "this" not a reference?](https://www.stroustrup.com/bs_faq2.html#this)

[Why is 'this' a pointer and not a reference?](https://stackoverflow.com/questions/645994/why-is-this-a-pointer-and-not-a-reference)

# 隐藏的this指针

```
#include <iostream>

class Test {
public:
    Test(int value) : value(value) {
        std::cout << "constructor\n";
    }
    ~Test() {
        std::cout << "destructor\n";
    }
    Test(const Test& test) {
        std::cout << "copy constructor\n";
        value = test.value;
    }
    Test& operator=(const Test& test) {
        std::cout << "copy assignment operator\n";
        
        if (this == &test) {
            return *this;
        }
        
        value = test.value;
        
        return *this;
    }
    
    int getValue() const {
        return value;
    }
    
    void setValue(int value) {
        this->value = value;
    }
    
private:
    int value;
};

int main() {
    Test test(10);
    std::cout << "test.value:" << test.getValue() << "\n";
    test.setValue(100);
    std::cout << "test.value:" << test.getValue() << "\n";

    return 0;
}
```
输出
```
constructor
test.value:10
test.value:100
destructor
```

this指针是类非静态成员函数的一个隐式参数（左边第一个参数），this指针指向隐藏的对象。 `int Test::getValue()`可以展开为`int getValue(Test* const this)`，`void Test::setValue(int value)`可以展开为`int setValue(Test* const this, int value)`，因此`test.getValue()`最终被转换为`getValue(&this)`，`test.setValue(100)`最终被转换为`setValue(&test, 100)`。

## this指针为什么被设计为左边第一个参数？
试想一下函数隐藏参数可以被设计在左边第一个、中间、右边第一个。但因为我们的函数缺省参数是从函数的右边第一个开始的，如果某个函数存在缺省参数，把this指针填充在右边第一个时，先前的缺省参数就变成右边第二个，而右边第一个this指针并不是缺省的，此时会编译错误，同理，this指针也不能被放置到中间，只能被放置到左边第一个。

## this指针占用类的存储空间吗？ 
`this`指针只是类非静态成员函数的一个函数参数，而非类的隐藏成员变量，因此也不用担心this指针使类在内存上变大。

## this指针是个左值，可以使用const修饰this指针
```
void Test::modefyThis(int value) { this = nullptr; } // 编译失败
```
输出
```
main.cpp: In member function 'void Test::modefyThis(int)':
main.cpp:35:34: error: lvalue required as left operand of assignment
   35 |     void modefyThis(int value) { this = nullptr; }
      |                                  ^~~~
```
从上述`Test* const this`可以看出this是一个常量指针，常量指针不可以修改其指向，但可以修改所指向的内容。
```
void Test::setValue(int value) {
    this->value = value;
}
```
另外可以使用const修饰this指针，这样this指针就是一个指向常量的常量指针，其指向和所指向的内容都不可被修改。
```
void Test::setValue(int value) const {
    this->value = value; // 编译报错
}
```
输出
```
main.cpp: In member function 'void Test::setValue(int) const':
main.cpp:32:21: error: assignment of member 'Test::value' in read-only object
   32 |         Test::value = value;
      |         ~~~~~~~~~~~~^~~~~~~
```
另外c++还允许使用mutable修饰成员变量，告诉编译器在const修饰this指针的非静态成员函数里可以修改此成员变量。
```
    void setValue(int value) const {
        this->value = value; // 编译ok
    }
    
    
private:
    mutable int value;
```

# 使用this指针

## 不推荐显式调用this指针

通常情况下，不推荐显式调用this指针，拿getValue函数举例：
```
int Test::getValue() { return value; } // 隐式调用this
int Test::getValue() { return this->value; } // 显式调用this
```
隐式调用this和显式调用this效果是一样的，编译器会自动把隐式调用this转换给显式调用this，隐式调用this明显比显式调用this少敲几个字符（放到一个大工程中可能是少敲成千上万个字符）且看起来更加整洁干净，何乐而不为。但有几个情况必须显式调用this。

## 显示调用this

### 非静态函数参数名称与成员变量同名
如果你有一个成员函数，它有一个与成员变量同名的参数，你可以使用`this`来消除歧义。在`setValue`函数内部，`value`会被当成参数`value`而不是成员变量`value`，因为函数参数变量会隐藏类成员变量。如果你想使用成员变量`value`你必须明确的指出。有三种方式可供参考：
```
void Test::setValue(int value) { this->value = value; } // 显式调用this
void Test::setValue(int value) { Test::value = value; } // Test::限定符
void Test::setValue(int value) { mValue = value; } // 以m为前缀修改成员变量名称
```

### copy assignment operator
```
Test& Test::operator=(const Test& test) {
    std::cout << "copy assignment operator\n";
    
    if (this == &test) {
        return *this;
    }
    
    value = test.value;
    
    return *this;
}
```
使用this指针对比地址，返回*this。

### 链式表达式
```
Test& Test::add(int add) { value += add; return *this; }
Test& Test::sub(int sub) { value -= sub; return *this; }
Test& Test::mult(int mult) { value *= mult; return *this; }
```
```
Test test(10);
test.add(1).sub(2).mult(3);
```
非常像java的build模式，`test.add(1).sub(2).mult(3);`被解释成`(test.add(1)).sub(2).mult(3);`即`test.sub(2).mult(3);`，`test.sub(2).mult(3);`被解释成`(test.sub(2)).mult(3);`即`test.mult(3);`。

### 非静态成员函数向外部（函数或变量）传递类指针或类引用
```
// 全局变量
Test* gTestPtr = nullptr;
// 非成员函数
void fun(Test* test) {
    std::cout << "fun(Test*)\n";
    test->setValue(200);
}

// 非成员函数
void fun(const Test& test) {
    std::cout << "fun(const Test& test)\n";
    test.getValue();
}

// 成员函数
void Test::doSomeThing() {
    gTestPtr = this;
    fun(this);
    fun(*this);
}
```
非静态成员函数`doSomeThing`内部需要向外部传递类指针或类引用。注意构造函数内部需要向外部传递类指针或类引用时容易可能引发半构造问题：即成员变量还未全部初始化完成前，把`this`或`*this`传递给了外部，外部直接访问了还未完成初始化的成员变量。

### 非静态成员函数向外部（函数或变量）传递`std::shared_ptr`
当我们熟练使用`std::shared_ptr`之后，如果想在非静态成员函数向外部（函数或变量）传递`std::shared_ptr`，不能直接使用`this`,而是使用`shared_from_this`代替`this`。
```
#include <iostream>
#include <memory>

class Test;
void fun(Test* test);
void fun(const Test& test);
void fun(const std::shared_ptr<Test>& test);

Test* gTestPtr = nullptr;

class Test : public std::enable_shared_from_this<Test> {
public:
    Test(int value) : value(value) {
        std::cout << "constructor\n";
    }
    ~Test() {
        std::cout << "destructor\n";
    }
    Test(const Test& test) {
        std::cout << "copy constructor\n";
        value = test.value;
    }
    Test& operator=(const Test& test) {
        std::cout << "copy assignment operator\n";
        
        if (this == &test) {
            return *this;
        }
        
        value = test.value;
        
        return *this;
    }
    
    int getValue() const {
        return value;
    }
    
    void setValue(int value) {
        this->value = value;
    }
    
    // 成员函数
    void doSomeThing() {
        gTestPtr = this;
        fun(this);
        fun(*this);
    }
    
    void doSomeOtherThing() {
        fun(shared_from_this());
    }
    
private:
    int value;
};

// 非成员函数
void fun(Test* test) {
    std::cout << "fun(Test*)\n";
    test->setValue(200);
}

// 非成员函数
void fun(const Test& test) {
    std::cout << "fun(const Test& test)\n";
    std::cout << "test.getValue():" << test.getValue() << "\n";
}

// 非成员函数
void fun(const std::shared_ptr<Test>& test) {
    std::cout << "fun(const std::shared_ptr<Test>& test)\n";
    std::cout << "test->getValue():" << test->getValue() << "\n";
}

int main() {
    Test test(10);
    std::cout << "test.value:" << test.getValue() << "\n";
    test.setValue(100);
    std::cout << "test.value:" << test.getValue() << "\n";
    test.doSomeThing();
    
    auto test1 = std::make_shared<Test>(20);
    test1->doSomeOtherThing();

    return 0;
}
```
输出：
```
constructor
test.value:10
test.value:100
fun(Test*)
fun(const Test& test)
test.getValue():200
constructor
fun(const std::shared_ptr<Test>& test)
test->getValue():20
destructor
destructor
```
