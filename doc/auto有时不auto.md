# auto有时不auto
```
#include <iostream>

int main() {
   int a = 1;
   int& b = a;
   auto c = a;
   auto d = b;
   auto& e = b;
  
   std::cout << "a:" << a << " b:" << b << " c:" << c << " d:" << d << " e:" << e << std::endl;
   
   a = 10;
   
   std::cout << "a:" << a << " b:" << b << " c:" << c << " d:" << d << " e:" << e << std::endl;
}
```
输出
```
a:1 b:1 c:1 d:1 e:1
a:10 b:10 c:1 d:1 e:10
```
a是int类型，b是int&类型，c是int类型（同a），d是int类型（与b不同），e是int&类型（同b），auto没有把d推导为和b相同的类型，有点反直觉。

# 奇怪的知识增加了
如果希望推演出来的auto类型为引用（左值引用），需要明确指出，就像变量e定义的那样:
```
auto& e = b;
```
如果希望推演出来的auto类型携带const，也需要明确指出：
```
const auto ca = a;  // ca是const int
```