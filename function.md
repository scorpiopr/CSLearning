可以用来存储、复制和调用任何可调用对象（Callable Object），包括普通函数、Lambda 表达式、仿函数（重载了 operator() 的类对象）以及类的成员函数。
------------------------------
## 💻 四大核心用法代码示例
## 1. 绑定普通函数与 Lambda 表达式
这是最基础的用法，用于替代传统的、可读性较差的函数指针。
```
#include <iostream>
#include <functional>
int addFunc(int a, int b) { return a + b; }
int main() {
    // 语法：std::function<返回值类型(参数类型1, 参数类型2, ...)>
    std::function<int(int, int)> f;

    // 用法 A：绑定普通函数
    f = addFunc;
    std::cout << f(10, 5) << std::endl; // 输出 15

    // 用法 B：绑定 Lambda 表达式
    f = [](int a, int b) { return a * b; };
    std::cout << f(10, 5) << std::endl; // 输出 50
}
```
## 2. 绑定仿函数（Functor）
仿函数是一个重载了 operator() 的类或结构体。std::function 可以无缝接收它。
```
struct Subtractor {
    int operator()(int a, int b) { return a - b; }
};
int main() {
    std::function<int(int, int)> f = Subtractor();
    std::cout << f(10, 5) << std::endl; // 输出 5
}
```
## 3. 绑定类的成员函数（面试高频考点）
非静态类成员函数在调用时，必须依赖一个具体的类对象（this 指针）。有两种常见写法：
```
class Calculator {
public:
    int multiply(int a, int b) { return a * b; }
};
int main() {
    Calculator obj;

    // 写法一：直接在 std::function 的参数列表里显式传递对象指针
    std::function<int(Calculator*, int, int)> f1 = &Calculator::multiply;
    std::cout << f1(&obj, 10, 5) << std::endl; // 输出 50

    // 写法二：配合 std::bind 提前绑定对象（最推荐，把成员函数包装成普通函数形式）
    // std::placeholders::_1 和 _2 代表留给未来调用的占位符
    std::function<int(int, int)> f2 = std::bind(&Calculator::multiply, &obj, std::placeholders::_1, std::placeholders::_2);
    std::cout << f2(10, 5) << std::endl; // 输出 50
}
```
## 4. 作为“回调函数”参数（项目/八股高频应用）
在网络编程（如线程池、事件驱动回调）中，std::function 通常作为参数传递，用来实现解耦。
```
// 一个触发回调的函数，接收一个 std::functionvoid performTask(int x, std::function<void(int)> callback) {
    int result = x * 2; // 模拟任务执行
    callback(result);   // 执行回调
}
int main() {
    performTask(10, [](int res) {
        std::cout << "Task complete! Result: " << res << std::endl;
    });
}
```
------------------------------
## 🔍 避坑与面试加分点

1. 未初始化调用陷阱（Crash）
   如果声明了 std::function 但没有给它赋值，直接调用它会抛出 std::bad_function_call 异常。
 * 安全写法：在调用前使用布尔判断：
      ```
      if (f) { f(10, 5); }
      ```
2. 性能开销（大厂必问底层）
  * 类型擦除（Type Erasure）：std::function 内部通过虚函数或函数指针抹平了不同对象的类型差异，这带来了一定的虚函数调用开销，无法进行内联优化。
  * 小对象优化（SOO）：如果绑定的对象（如轻量 Lambda 或函数指针）很小，它会存放在栈上；如果绑定的对象很大（捕获了大量变量的 Lambda），它会触发堆内存分配（Heap Allocation），从而带来性能损耗。
   


