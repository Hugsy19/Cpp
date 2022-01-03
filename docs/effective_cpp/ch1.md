# 习惯C++

## 视C++为一个语言联邦

- C++由四个部分组成：
  - C：C++的基础部分；
  - Object-Oriented C++：面向对象编程；
  - Template C++：泛型编程；
  - STL：标准模板库。
- 使用C++进行高效编程的守则因涉及的部分而异：
  - 对内置类型，按值传递（pass-by-value）比引用传递（pass-by-reference）高效；
  - 面向对象、泛型编程时，尽可能使用const引用传递（pass-by-reference-to-const）；
  - 对STL，迭代器和函数对象都从C指针上塑造而来，此时遵循上面的第一条。

## 尽量不使用宏定义

- 使用`#define`定义的名称不会被记录到符号表、无法提供任何封装性，且盲目的宏替换过程可能产生更多的代码量。
- 尽量用`const`常量替换`#define`，但要注意：
  - 常量定义式被放在头文件时，有必要将指针声明为`const`，如`const char* const authorName = "Scott Meyers";`；
  - 类的专属常量需要为`static`成员，且定义类时所写的只是该成员的**声明式**，往往还需要再使用时提供其**定义式**；
- 

## 尽可能使用const