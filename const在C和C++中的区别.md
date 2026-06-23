在 C 和 C++ 中，const 关键字的核心本质有巨大区别：C 语言中的 const 表示“只读变量”，而 C++ 中的 const 在编译期**尽可能**被视为“常量”。
以下是它们在面试中最常被考查的 4 个核心区别：
## 1. 能否用于定义数组长度（核心区别）

* C 语言：const 修饰的是一个不能被修改的变量，它依然是个变量，不是常量。因此它不能用来定义静态数组的长度。
```CPP
const int size = 10;
int arr[size]; // ❌ 错误（C99 之前报错，C99 后作为变长数组 VLA 处理，但不能用于全局静态数组）
```
* C++：const 变量如果**用字面量初始化**，编译器会将其视为真正的常量（类似 #define）。因此它可以用来定义数组长度。
```CPP
const int size = 10;
int arr[size]; //  完全合法
```

## 2. 链接属性（Linkage）不同

* C 语言：全局 const 变量默认具有外部链接属性（extern）。如果在多个源文件中定义同名的全局 const 变量，链接时会发生“符号重定义”错误。
* C++：全局 const 变量默认具有内部链接属性（static）。这意味着即使你在多个 .cpp 文件中包含同一个写了 const int x = 1; 的头文件，各个文件里的 x 也是互不干扰的，不会引发链接冲突。

## 3. 指针间接修改的行为
由于编译器对它们的基本假设不同，通过指针强行修改 const 变量时，结果完全不同：

* C 语言：可以通过指针强行绕过检查并修改其值。
```CPP
const int a = 10;
int *p = (int *)&a;
*p = 20;
printf("%d", a); // 输出 20（值确实被改变了）
```
* C++：由于[常量折叠（Constant Folding）](https://github.com/scorpiopr/C-CplusplusLearning/blob/main/%E5%B8%B8%E9%87%8F%E6%8A%98%E5%8F%A0.md)优化，编译器在编译时就把 a 替换成了 10。
```CPP
const int a = 10;
int *p = const_cast<int*>(&a);
*p = 20;
std::cout << a << " " << *p; // 输出 10 20// 解释：内存里的值可能变了（取决于编译器是否放到了只读数据区），但直接打印 a 依然输出 10！
```

## 4. 类型检查与函数重载（C++ 特有）

* C 语言：没有函数重载，const 仅用于基本的参数只读保护。
* C++：const 是类型系统的一部分。C++ 支持基于 const 的函数重载（特别是类成员函数）。
```CPP
class MyClass {public:
    void func() { /* 非常量对象调用 */ }
    void func() const { /* 常量对象调用 */ }
};
```
