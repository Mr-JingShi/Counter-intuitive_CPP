# 小括号初始化VS大括号初始化
```
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vt1(3, 6);
    std::vector<int> vt2{3, 6};
    
    std::cout << "vt1:";
    for (auto i : vt1) {
        std::cout << " " << i;
    }
    std::cout << std::endl;
    
    std::cout << "vt2:";
    for (auto i : vt2) {
        std::cout << " " << i;
    }
    std::cout << std::endl;
   
    return 0;
}
```
输出
```
vt1: 6 6 6
vt2: 3 6
```
对于std::vector<int>传递两个参数给构造函数时，使用小括号初始化或大括号初始化表现出的结果却大相径庭，有点反直觉。

## 奇怪的知识增加了
vt1通过小括号完成初始化，vt1初始内容为3个6；vt2通过大括号完成初始化，vt2的初始内容为3和6。

# 大括号初始化
## 直接列表初始化
传统C++指定初始值的初始化方式包括使用小括号、等号：
```
int x(42);
int x = 42;
```
C++11引入了一种统一初始化（也叫列表初始化、大括号初始化）：
```
int x{42};
int x = {42};
```
有没有很熟悉的感觉？和数组初始化极其类似：
```
char msg[] = {0};
int num[] = {3, 6, 9};
```
可试想一下（典型的数组替代物）std::vector<int>如何在创建时持有一个特定的集合值（比如：3、6、9）？
大括号初始化的提出就是为了解决传统语法不能覆盖所有初始化的问题。
使用大括号初始化来指定容器初始值：
```
std::vector<int> vt{3, 6, 9};  // vt的初始内容为3、6、9
```
### 大括号初始化新特性--禁止内建类型之间进行隐式转换
```
double pi = 3.1415926;
int a(pi), b = pi;  // 正确，隐式转换执行，丢掉了部分信息
int c{pi}, d = {pi};  // 错误，隐式转换未执行，因为存在丢失信息的风险
```
### 大括号初始化新特性--对语言最令人苦恼之解析语法免疫
```
Widget w1(10);  // 以10为参数构造了一个Widget类型的w1实例
Widget w2;  // 构造一个Widget类型的w2实例
Widget w3();  // 声明了一个名为w3返回一个widget类型的函数！！！
Widget w4{};  // 构造一个Widget类型的w3实例
```
`C++`规定，任何能够解析为声明的东东都要解析为声明，所以`Widget w3();`被解析为函数声明，对于调用默认构造函数使用`Widget w2;`或许是不错的选择。
### 大括号初始化新特性--大括号初始化其实是std::initializer_list<>类型之物
在构造函数调用时，只要形参中没有任何一个具备std::initializer_list<>型别，那么小括号和大括号没有区别，但如果一个或者多个构造函数声明了一个具备std::initializer_list<>型别的形参，那么采用了大括号初始化语法会强烈地优先选择带有std::initializer_list<>型别形参的版本。另外，当auto遇到{}时，auto会推导出大括号括起来的初始化表达式代表一个std::initializer_list<>，如：
```
auto x = {42}; // x的类别是std::initializer_list<int>，有点反直觉
```
## 复制列表初始化
大括号初始化其他典型应用
### 函数( { 实参1, 实参2, ... } ) 
```
void fun(std::pair<int, int>) { ... }
fun(std::make_pair(3, 6)); // 传统c++
fun({3, 6}); // c++11
```
### return { 实参1, 实参2, ... } ;
```
std::pair<int, int> fun() {  return std::make_pair(3, 6); } // 传统c++
std::pair<int, int> fun() {  return {3, 6}; } // c++11
```