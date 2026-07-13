std::string_view 是 C++17 引入的一个高性能、只读、零拷贝的字符串视图类。它的底层只包含一个指向字符串首地址的指针和一个表示长度的整数。
它的核心目的是作为函数形参，统一代替 const std::string& 和 const char*，从而彻底避免在传递字符串时产生不必要的内存分配（Memory Allocation）和拷贝。
------------------------------
## 🧱 为什么需要 std::string_view？（性能痛点）
在 C++17 之前，如果我们写一个需要只读字符串的函数，通常有两种写法，但它们都有性能缺陷：

// 写法 1：只接收 const char*void print_1(const char* s); // 缺陷：如果传入 std::string，需要调用 s.c_str()。而且无法高效获取长度（需要 O(N) 的 strlen）。
// 写法 2：只接收 const string&void print_2(const std::string& s); // 缺陷：如果传入字符串字面量 "hello"，会隐式构造一个临时的 std::string，在堆上分配内存（产生拷贝开销）。

std::string_view 完美解决了这个矛盾：它不拷贝数据，只是一个“看”这段内存的窗口（视图）。
------------------------------
## 💻 经典用法与核心代码示例## 1. 作为函数形参（最推荐的通用场景）
它可以无缝接收 std::string、const char* 和字符串字面量，且全部是零拷贝。
```cpp
#include <iostream>
#include <string>
#include <string_view>
void process_string(std::string_view sv) { 
    // 🌟 1. 注意：由于极轻量（通常16字节），推荐直接值传递（Pass by value）
    std::cout << "Data: " << sv << ", Length: " << sv.length() << "\n";
}
int main() {
    std::string str = "Hello World";
    
    process_string(str);            // 接收 std::string，零拷贝
    process_string("Hello World");  // 接收字符串字面量，零拷贝
    process_string(str.c_str());    // 接收 const char*，零拷贝
    
    return 0;
}
```
## 2. 高效切片/子串（O(1) 复杂度的 substr）
对 std::string 调用 substr() 会生成一个新的字符串并发生内存拷贝。
而对 std::string_view 调用 substr() 只是移动了内部的指针并修改了长度，时间复杂度是 O(1)，无任何内存开销。
```cpp
#include <iostream>
#include <string_view>
int main() {
    std::string_view sv = "C++17_string_view_tutorial";
    
    // O(1) 获取子串视图
    std::string_view sub = sv.substr(6, 11); // 从下标 6 开始，截取 11 个字符
    
    std::cout << sub << "\n"; // 输出: string_view
    return 0;
}
```
------------------------------
## ⚠️ 面试大厂必问：使用 std::string_view 的三大致命陷阱
std::string_view 虽然高效，但由于它不拥有（Own）内存，在工程和面试中极易引发严重的 Bug：
## 陷阱一：生命周期悬空（Dangling View）—— 最常见崩溃原因
如果它指向的原始字符串对象被销毁了，string_view 就会变成一个野指针集合，访问它会导致未定义行为（Undefined Behavior）。
```cpp
std::string_view get_corrupted_view() {
    std::string temp = "I will be destroyed";
    return temp; // ❌ 致命错误！temp 是局部变量，函数结束后被销毁
} // 此时返回的 string_view 指向了一块已被释放的内存
int main() {
    std::string_view bad_sv = get_corrupted_view();
    std::cout << bad_sv; // 💥 崩溃或输出脏数据
}
```
## 陷阱二：不保证以 \0（空字符）结尾
std::string 和 const char* 一定是以 \0 结尾的。但 std::string_view 允许指向字符串的某一个子部分，它的末尾不一定有 \0。

* 致命后果：绝对不能把 sv.data() 直接传给需要标准 C 风格字符串的函数（如 printf、strlen、fopen）。
```cpp
std::string_view sv = "Hello World";
std::string_view sub = sv.substr(0, 5); // "Hello"
// ❌ 错误用法！printf 会一直向后打印，直到在内存里遇到 \0 才会停止
printf("%s\n", sub.data()); // 输出可能是 "Hello World" 甚至崩溃
// 🔑 正确用法：在现代 C++ 中直接用 std::cout 打印，或者将其显式转为 std::string
std::cout << sub << "\n"; // cout 知道长度，可以安全打印 "Hello"
std::string safe_str(sub); // 显式拷贝构造为 string
```

## 陷阱三：无法修改内容（只读）
std::string_view 内部的指针是指向 const char 的，你无法通过它去修改原始字符串。如果需要修改，依然需要使用 std::string。
------------------------------
## 📊 面试对比复习卡片
```
| 特性 | const char* | const std::string& | std::string_view (C++17) |
|---|---|---|---|
| 内存所有权 | 不拥有 | 拥有（或共享引用） | 不拥有 |
| 获取长度复杂度 | O(N) (需要 strlen) | O(1) | O(1) |
| 传入字面量开销 | 零开销 | 有开销（可能引发堆分配） | 零开销 |
| 内部子串切片 | 无法直接切片 | O(N) (拷贝产生新 string) | O(1) (零开销改变视图) |
| 空字符 \0 结尾 | 保证 | 保证 | 不保证 |
```
