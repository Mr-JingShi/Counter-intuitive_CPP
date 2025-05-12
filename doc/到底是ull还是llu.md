# ull or llu
```
#include <cstdio>
#include <iostream>

int main() {
    unsigned long long a = 1000ULL;
    printf("a:%llu\n", a);
    
    std::string b = "2000";
    std::cout << "std::stoull(b):" << std::stoull(b) << std::endl;
}
```
输出
```
a:1000
std::stoull(b):2000
```
printf中用llu标识unsigned long long，std::stoull中却用ull标识unsigned long long，有点反直觉。

# 奇怪的知识增加了
