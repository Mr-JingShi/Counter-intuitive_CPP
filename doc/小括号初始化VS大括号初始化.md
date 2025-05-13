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

# 奇怪的知识增加了
vt1通过小括号完成初始化，vt1初始内容为3个6；vt2通过大括号完成初始化，vt2的初始内容为3和6。