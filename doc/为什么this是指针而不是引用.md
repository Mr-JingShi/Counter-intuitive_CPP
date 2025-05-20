# 为什么this是指针而不是引用

java、c#等语言的this被设计成了引用，而c++的this却被设计成了指针，且c++的this指针有别于其他普通指针：this指针在语言层面被设计为永远指向被操作的对象（this指针永远不为空），这使的this指针更加倾向于c++的引用，c++把this设计成指针而不是引用，有点反直觉。

# 奇怪的知识增加了
答案很简单：当this被添加到c++中时，引用还不存在。
> Why is "this" not a reference?
>
> Because "this" was introduced into C++ (really into C with Classes) before references were added. Also, I chose "this" to follow Simula usage, rather than the (later) Smalltalk use of "self".

[Why is "this" not a reference?](https://www.stroustrup.com/bs_faq2.html#this)
[Why is 'this' a pointer and not a reference?](https://stackoverflow.com/questions/645994/why-is-this-a-pointer-and-not-a-reference)

```
#include <iostream>

class Test
{
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
    
    int getValue() {
        return this->value;
    }
    
    void setValue(int value) {
        this->value = value;
    }
    
private:
    int value = 0;
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

this指针是类非静态成员函数的一个隐式参数（左边第一个参数），this指针指向隐藏的对象。 `int Test::getValue()`可以展开为`int getValue(Test* this)`，`void Test::setValue(int value)`可以展开为`int setValue(Test* this, int value)`，因此`test.getValue()`最终被转换为`getValue(&this)`，`test.setValue(100)`最终被转换为`setValue(&test, 100)`，this指针只是类非静态成员函数的一个函数参数，而非类的隐藏成员变量，因此也不会使类在内存上变大。
