# delete又未完全delete
假设我们要定义一个这样的类：只支持拷贝操作不支持移动操作。
```
#include <iostream>

class Test {
public:
    Test() {
        std::cout << "constructor\n";
    }

    Test(const Test& test) {
        std::cout << "copy constructor\n";
    }
    Test& operator=(const Test& test) {
        std::cout << "copy assignment operator\n";
        
        return *this;
    };

    Test(Test&& test) = delete;
    Test& operator=(Test&& test) = delete;
};

Test getTest() {
    Test test;
    return test;
}

int main() {
    Test test{ getTest() };

    return 0;
}
```
输出
```
main.cpp: In function 'Test getTest()':
main.cpp:16:12: error: use of deleted function 'Test::Test(Test&&)'
   16 |     return test;
      |            ^~~~
main.cpp:10:5: note: declared here
   10 |     Test(Test&& test) = delete;
      |     ^~~~
main.cpp:16:12: note: use '-fdiagnostics-all-candidates' to display considered candidates
   16 |     return test;
      |            ^~~~
```
从编译错误信息可以看出，被显式`delete`的`Test(Test&& test)`参与到了重载决议，且被认为比拷贝构造版本更好，但移动构造版本被指定了删除不能被使用，所以编译器报错。
指定了删除，却未完全删除，参与到了重载决议，有点反直觉。

# 奇怪的知识增加了
删除的移动构造函数仍然是一个已声明的函数，有资格进行重载解析。`getTest`函数按值返回将优先选择删除的移动构造函数而非未删除的复制构造函数。

```
#include <iostream>

class Test {
public:
    Test() {
        std::cout << "constructor\n";
    }

    Test(const Test& test) {
        std::cout << "copy constructor\n";
    }
    Test& operator=(const Test& test) {
        std::cout << "copy assignment operator\n";
        
        return *this;
    };

    Test(Test&& test) = default;
    Test& operator=(Test&& test) = delete;
};

Test getTest() {
    Test test;
    return test;
}

int main() {
    Test test{ getTest() };

    return 0;
}
```
输出
```
constructor
```
```
#include <iostream>

class Test {
public:
    Test() {
        std::cout << "constructor\n";
    }

    Test(const Test& test) {
        std::cout << "copy constructor\n";
    }
    Test& operator=(const Test& test) {
        std::cout << "copy assignment operator\n";
        
        return *this;
    };

    Test(Test&& test) {
        std::cout << "move constructor\n";
    }
    Test& operator=(Test&& test) = delete;
};

Test getTest() {
    Test test;
    return test;
}

int main() {
    Test test{ getTest() };

    return 0;
}
```
输出
```
constructor
```
将`Test(Test&& test) = delete`修改为`Test(Test&& test) = default`或者显式定义虽可以修复此问题，然而，之前的本意就是将移动构造删除，让其使用拷贝构造啊，如此修改违背了本意。
更进一步的观察，竟然未输出`move constructor`！重载决议选择了移动构造，但未执行移动构造，有点反直觉，此问题请参考尾返回值优化。

C++11之前可以通过“只声明不实现”的方式要删除函数，既然尾返回值优化可以优化掉拷贝构造的调用，那我们可以保证编译通过即可。
```
#include <iostream>

class Test {
public:
    Test() {
        std::cout << "constructor\n";
    }

    Test(const Test& test) {
        std::cout << "copy constructor\n";
    }
    Test& operator=(const Test& test) {
        std::cout << "copy assignment operator\n";
        
        return *this;
    };

    Test(Test&& test);
    Test& operator=(Test&& test) = delete;
};

Test getTest() {
    Test test;
    return test;
}

int main() {
    Test test{ getTest() };

    return 0;
}
```
输出
```
constructor
```
从上述代码表现来看，移动构造只参与了重载决议，未参与链接和运行期运行！

最后我们再尝试一个移动构造参与链接器链接的版本：
```
#include <iostream>

class Test {
public:
    Test() {
        std::cout << "constructor\n";
    }

    Test(const Test& test) {
        std::cout << "copy constructor\n";
    }
    Test& operator=(const Test& test) {
        std::cout << "copy assignment operator\n";
        
        return *this;
    };

    Test(Test&& test);
    Test& operator=(Test&& test) = delete;
};

Test getTest() {
    Test test;
    return std::move(test);
}

int main() {
    Test test{ getTest() };

    return 0;
}
```
输出
```
/tmp/cc3TH4WE.o: In function `getTest()':
main.cpp:(.text+0x32): undefined reference to `Test::Test(Test&&)'
collect2: error: ld returned 1 exit status
```
重载决议选择了移动构造，但是链接时未找到移动构造的实现！
