当一条加载指令指示CPU 从主存地址A 中读一个字w时，它将地址A 发送到高速缓存。高速缓存根据映射关系从上到下逐层在A地址对应的位置查找w的副本，如果cache hit，就立即将那个字发回给CPU，否则为cache miss，此时CPU必须等待高速缓存从主存读取包含w的块，存放在一个cache line中，并从中抽取w返回给CPU寄存器。
## Cache分类
1. 直接映射
2. 组相连
3. 全相连
## Cache规格参数 
1. Cache等级，L1、L2等
2. Cache容量C，不包括有效位和标记位
3. 每个组的Cache Line个数E
4. Cache Line字节数B
## 高速缓存查找流程
举例来说, 如果Cache规格为：data cache: 32-KB, 8-way set associative, 64-byte line size, 32位主内存地址。
即C=32K，E=8，B=64, m=32
1. 组选择
   cache组数S = C / E / B = 32K/8/64 = 64
   
3. 
对于32位的内存地址，每个line有2^6=64Byte，所以地址的【0，5】区分line
中的那个字节。一共有64个组。我们取内存地址中间6为来hash查找地址属于那
个组。即内存地址的【6，11】位来确定属于64组的哪一个组。组确定了之后
【12，31】的内存地址与组中8个line挨个比对，如果【12，31】为与某个line
-致，并且这个line为有效，那么缓存命中。



Cache set（组数）

## 参考
[1] 《深入了解计算机系统》6.4.1 
[2] https://zhuanlan.zhihu.com/p/37749443
