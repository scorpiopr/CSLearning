| | new | malloc |
| --- | --- | --- |
| 语法 | `T* p = new T(args); // 分配1个T类型的内存并初始化为args` <br> `T* p = new T[]; // 分配动态数组内存` | `T* p = (T*)malloc(sizeof(T))` |
| 是否调用构造函数 | 如果T是类类型，调用类构造函数 | 否 |
| 内存释放 | `delete p;` <br> `delete[] p;` <br> 类类型调用析构函数，调用operator delete释放内存 | `free(p);` <br> 仅释放内存，不调用析构函数 |
| 返回值类型 | T* | void* (需要显式类型转换) |
| 异常处理 | * 抛出std::bad_alloc异常，适配C++异常处理 <br> * 使用new(std::nothrow)T，则返回nullptr | 返回nullptr，需手动校验返回值 |
