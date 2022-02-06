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

## 尽量不使用#define

- 使用`#define`定义的名称不会被记录到符号表、无法提供任何封装性，且盲目的宏替换过程可能产生更多的代码量。
- 尽量用`const`常量替换`#define`定义的常量，但要注意：
  - 常量定义式被放在头文件时，有必要将指针声明为`const`，如`const char *const authorName = "Scott Meyers";`；
  - 类的专属常量需要为`static`成员，且定义类时所写的只是该成员的**声明式**，往往还需要再使用时提供其**定义式**；
  - 当编译器不支持在类内对静态成员初始化时，可使用名为“enum hack”的补偿做法：**枚举类型的数值可充当ints使用**
    ```cpp
    class GampPlayer {
    private:
      //static const int NumTurns = 5; 可用以下替代
      enum { NumTurns = 5 };

      int scores[NumTurns];
    }
    ```
    - “enum hack”的行为与`#define`类似，具有无法取址的约束，且不会导致非必要的的内存分配；
    - “enum hack”还是模板元编程的基础技术。
- 用`#define`实现的宏（macros）最好用**内联模板函数**（template inline）来替换，与宏的效率相近的同时具有类型安全性：
  ```cpp
  // #define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b)) 可用以下替代
  template<typename T>
  inline void callWithMax(const T& a, const T& b)
  {
    f(a > b ? a : b);
  }
  ```

## 尽可能使用const

- `const`出现在`*`左边，表示被指物为常量，右边则表示指针本身为常量。
- 令函数返回`const`值，可降低代码错误造成的意外，且不至于放弃高效和安全性。
- 将成员函数声明为`const`，是为了说明该成员函数可作用于`const`对象，且**函数可由常量性的不同而被重载**。
  - 此时将存在“bitwise constness”约束，即成员函数中不可更改任何非`static`成员变量；
  - 在非`static`成员前加`mutable`，可释放其“bitwise constness”约束，即可在`const`成员函数内修改该变量，而实现“logical constness”。
  - 当`const`和`non-const`成员函数有实质等价的实现时，**可令`non-const`调用`const`版本**，以避免代码重复：
  ```cpp
  class TextBlock {
  public:
    const char& operator[](size_t pos) const { 
      ...
      return text[pos];
    }
    char& operator[](size_t pos) {
      return const_cast<char&>(static_cast<const TextBlock&>(*this)[pos]);
    }
  }
  ```
  - 第一次为`*this`添加`const`，第二次从`const operator[]`的返回值中移除`const`；
  - 令`const`调用`non-const`版本则是一种冒险行为，因后者可能改变对象的逻辑状态。

## 确定对象使用前已被初始化

-  永远在使用对象之前将其初始化：
   -  对没有任何成员的内置类型必须手动进行初始化，C++不保证它们的初始化；
   -  在内置类型以外，初始化的责任落在构造函数。
-  对象的成员需要通过**成员初始值列表**（memory initialization list）来完成初始化过程，而非构造函数体之内的赋值操作，且前者发生在后者之前。
   -  后者需要先调用default构造函数完成初始化再赋新值，前者则一般只需要进行一次拷贝构造；
   -  `const`或引用类型的成员变量一定需要初值，而不能被赋值。
-  尽管编译器会为自定义类型自动调用default构造函数，但还是需要再初始值列表中列出所有的成员变量，以免发生遗漏。
-  C++存在固定的成员初始化次序：
   -  基类的初始化早于派生类；
   -  类中的成员变量总以其声明的次序被初始化；
      -  因此初始值列表中列出的成员变量次序应与其声明次序一直。
-  函数内的`static`对象为**local static**对象，其他对象则为**non-local static**对象。
   -  C++对定于在不同编译单元内的**non-local static**对象的初始化顺序无明确定义；
   -  使用**local static**对象代替**non-local static**对象可解决以上初始化顺序所带来的问题：
   ```cpp
   class FileSystem { ... };
   FileSystem& tfs()
   {
     static FileSystem fs;
     return fs;
   }

   class Directory { ... };
   Directory::Directory( params )
   {
     ...
     std::size_t disks = tfs().numDisks();
     ...
   }
   ```