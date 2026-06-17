# 1 C++基础知识点总结

一般来说，面向对象里的对象是class的实例(包括了可能存在的封装、继承、多态等特性)，面向过程里的对象是struct的实例。struct默认public，class默认private。

## 指针vs引用
``` cpp
   // 引用示例    
   const int ci = 10;  
   //int& r = ci;  // 错误：非const引用不能绑定到const对象
   const int& cr = ci;  //正确：const引用可以绑定到const对象
    //指针示例
   int a = 10;
   const int* p1 = &a;  // p1指向的值是const的
   //*p1 = 20;  //错误：不能通过p1修改a的值
   int* const p2 = &a;  // p2本身是const的，不能改变指向
   //p2 = &b;  //错误：不能改变p2指向的对象
```

## static用法

1. 在函数内初始化
2. 函数或变量，仅当前文件可见，`extern`无法引用
3. 类共享（Utility,Factory等设计模式）
``` cpp
class MyClass {
public:
   static int staticVar;  // 声明静态成员变量
};
int MyClass::staticVar = 0;  // 类外定义并初始化静态成员变量
```
或
``` cpp
class MyClass {
public:
   static int add(int a, int b) {
      return a + b;
   } 
};
int x = MyClass::add(3, 4);  // 调用静态成员函数
MyClass myObj;
int y = myObj.add(5, 6);  // 也可以通过对象,但本质还是类调用，并不依赖对象实例
```

## define vs typedef
``` cpp
#define PI 3.14159  // 预处理器宏定义，文本替换
typedef int MyInt;  // 类型别名，编译器理解为MyInt是int的别名
inline int add(int a, int b) {
   return a + b;
}// 内联函数，编译器会尝试将函数调用替换为函数体代码，减少函数调用开销,不能递归、循环、过多条件判断等等。
```
## new vs malloc
``` cpp
int* p1 = (int*)malloc(sizeof(int));  // malloc分配内存，返回void*，需要强制类型转换
*p1 = 10;  // 使用malloc分配的内存需要手动初始化
int* p2 = new int;  // new分配内存，返回正确类型的指针，并调用构造函数（如果是类对象）
*p2 = 20;
delete p2;  // 使用new分配的内存需要使用delete释放，调用析构函数（如果是类对象）
free(p1);  // 使用malloc分配的内存需要使用free释放
```

## volatile关键字
``` cpp
for(volatile int i=0; i<100000; i++); // 它会执行，不会被优化掉  
```
通常用在多线程编程中，告诉编译器该变量可能被多个线程修改，禁止对其进行优化，确保每次访问都从内存中读取最新值.

## 线程安全

### thread, processor and program
- thread: 线程，是程序执行的最小单位，一个程序可以有多个线程同时执行。
- processor: 处理器，计算机的核心组件，负责执行指令和处理数据。一个processor可以有多个核心，每个核心可以同时执行一个线程。processor是一个物理概念，抽象为操作系统中的调度单位。
- program: 程序，是一组指令的集合，告诉计算机如何执行特定任务。
- process: 进程，是程序的一个实例，包含程序代码和当前活动的状态（如寄存器、内存等）。一个程序可以有多个进程，每个进程可以有多个线程。

线程共享程序的内存空间和资源，但每个线程有自己的执行上下文（如寄存器、堆栈等）。处理器可以同时执行多个线程，具体取决于处理器的核心数量和调度算法。

processor通过时间片轮转等调度算法来管理线程的执行，确保每个线程都有机会运行。程序中的线程可以通过同步机制（如互斥锁、条件变量等）来协调访问共享资源，避免竞争条件和死锁等问题。

affinity: 线程亲和性，指的是将线程绑定到特定的处理器核心上，以提高性能和减少上下文切换的开销。

### std::atomic

``` cpp
#include <atomic>
std::atomic<int> counter(0);  // 定义一个原子整数，初始值为0
counter++;  // 原子操作，线程安全的递增
// linux中不能直接对counter进行操作
counter.fetch_add(1);  // 原子操作，线程安全的递增
```


# 2 多态问题

## 函数指针

函数指针是指向函数的指针变量。它可以用于存储函数的地址，允许在运行时动态选择要调用的函数。

``` cpp
#include <iostream>
using namespace std;
// 定义一个函数指针类型
typedef void (*FuncPtr)();
// 定义两个函数
void funcA() {
    cout << "Function A" << endl;
}
void funcB() {
    cout << "Function B" << endl;
}
int main() {
    // 定义一个函数指针变量，并指向funcA
    FuncPtr ptr = funcA;
    ptr();  // 调用funcA，输出 "Function A"
    // 将函数指针指向funcB
    ptr = funcB;
    ptr();  // 调用funcB，输出 "Function B"
    return 0;
}
```

`加减`:
``` cpp
int add(int a, int b) {
    return a + b;
}
int subtract(int a, int b) {
    return a - b;
}
int main() {
   int (*operation)(int, int);  // 定义一个函数指针，指向返回int、参数为两个int的函数
   operation = &add;  // 将函数指针指向add函数
   int result = operation(5, 3);  // 调用add函数，result = 8
   cout << "Addition: " << result << endl;  // 输出
   operation = &subtract;  // 将函数指针指向subtract函数
   result = operation(5, 3);  // 调用subtract函数，result = 2
   cout << "Subtraction: " << result << endl;  // 输出
   return 0;
}
```

场景
- 回调： 将函数指针作为参数传递给另一个函数，在特定事件发生时调用
- 函数指针数组：循环或条件调度多个函数
``` cpp
#include <iostream>
using namespace std;
// 定义三个函数
void funcA() {
    cout << "Function A" << endl;
}
void funcB() {
    cout << "Function B" << endl;
}
void funcC() {
    cout << "Function C" << endl;
}
int main() {
    // 定义一个函数指针数组，存储三个函数的地址
    void (*funcArray[3])() = {funcA, funcB, funcC};
    // 通过循环调用函数指针数组中的函数
    for (int i = 0; i < 3; i++) {
        funcArray[i]();  // 输出 "Function A", "Function B", "Function C"
    }
    return 0;
}
```
- 动态加载库：在运行时加载动态库，并通过函数指针调用库中的函数
- 参数
- 函数映射表
- **多态实现**：虚函数和函数指针结合可以实现多态行为。

## 虚函数

## 智能指针

智能指针是C++11引入的特性，用于管理动态分配的内存。它们提供了自动内存管理的功能，避免了手动管理内存的复杂性和潜在错误。

### std::unique_ptr

`std::unique_ptr` 是一个独占所有权的智能指针，它确保同一时间只有一个指针指向动态分配的内存。当 `std::unique_ptr` 被销毁时，它会自动释放所指向的内存。

``` cpp
#include <memory>
std::unique_ptr<int> ptr(new int(10));  // 创建一个unique_ptr，指向动态分配的int
*ptr = 20;  // 修改指向的值
// 当ptr离开作用域时，自动释放内存
```

### std::shared_ptr

`std::shared_ptr` 是一个共享所有权的智能指针，它允许多个指针同时指向同一个动态分配的内存。当最后一个 `std::shared_ptr` 被销毁时，它会自动释放所指向的内存。

``` cpp
#include <memory>
std::shared_ptr<int> ptr1(new int(10));  // 创建一个shared_ptr
std::shared_ptr<int> ptr2 = ptr1;  // 复制shared_ptr，增加引用计数
*ptr1 = 20;  // 修改指向的值
// 当ptr1和ptr2都离开作用域时，自动释放内存
```

### std::weak_ptr

`std::weak_ptr` 是一个弱引用智能指针，它不增加引用计数。通常与 `std::shared_ptr` 结合使用，用于打破循环引用的问题。

``` cpp
#include <memory>
std::shared_ptr<int> ptr1(new int(10));  // 创建一个shared_ptr
std::weak_ptr<int> ptr2 = ptr1;  // 创建一个weak_ptr，不增加引用计数
// 当ptr1离开作用域时，ptr2将变为expired状态
```
