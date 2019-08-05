# 导读
+ 我们一般将构造函数声明为explicit，因为这样可以阻止隐式转换。
+ 以byvalue传递用户自定义类型通常是个坏主意，Pass-by-reference-to-const往往是比较好的选择。


# 让自己习惯C++
## item1 View C++ as a federation of language
+ 将C++看作由四个语言组成的语言联邦
  1. C
  2. Object-Oriented C++
  3. Template C++.属于C++的泛型编程部分
  4. STL
## item2 Prefer consts,enums,and inlines to #define
+ 即尽量以编译器代替预处理器。即尽量不要使用
  ```
  #define ASPECT_RATIO 1.653
  ```
  而是
  ```
  const double ASPECTratio = 1.653
  ```
  常量定义式通常放在头文件内，所以要将指针也声明为const。
  ```
  const char* const authorName = "Scott Meyers";
  或者
  const std:: string authorName ("Scott Meyers");
  ```
  而在类中的常量，必须是一个static成员。
  不要用#define实现宏。即写出这样的代码 #define CALL WITH MAX (a, b) f ((a) > (b) ? (a) : (b).
  要实现上面的功能可以使用template inline函数
  ```
  template<typename T>
  inline void callWithMax(const T& a,const T& b)
  {
    f(a>b?a:b);
  }
  ```
+ 所以
  1. 对于单纯变量，最好以const对象或者enums替换#define
  2. 对于形似函数的宏(macros)，最好改用inline函数替换#defines。
## item3 Use const whenever possible
+ 如果const出现在星号左边，表示被指物为常量。如果出现在星号右边，表示指针自身是常量。
+ STL选代器系以指针为根据塑模出来，所以迭代器的作用就像个T*指针。声明选代器为const就像
  声明指针为const一样(即声明一个T* const指针)，表示这个迭代器不得指向不同的东西但它所
  指的东西的值是可以改动.
+ const可以用于函数声明时的应用。在一个函数声明式内，const可以和函数返回值、各参数、函数自身
  (如果是成员函数)产生关联。如下
  ```
  class Rational {...};
  const Rational operator*(const Rational& lhs,const Rational& rhs);
  ```
  这是为了防止做出 Rational a , b, c;  ...  (a * b) = c;这样的操作。
  而使用const参数可以防止我们犯想要输入"=="却意见的输入了"="这样的错误。
+ 将 const 实施于成员函数的目的，是为了确认该成员函数可作用于const对象身上。
  1. 它们使class接口比较容易被理解。这是因为，得知哪个函数可以改动对象内容而哪个函数不行，很是重要。
  2. 它们使"操作const对象"成为可能。
+ 注意，如果要在const成员函数中改变non-static成员变量的值，可以将这些值声明为mutable
  ```
  class CTextBlock{
    public:
       ...
       std::size_t length() const;
    private:
       char *pText;
       mutable std::size_t textLength;
       mutable bool lengthIsValid;
  };
  std::size_t CTextBlock::length() const{
    if(!lengthIsValid){
      textLength=std::strlen(pText);//在const成员函数中改变了成员变量的值
      lengthIsValid = true;
    }
    return textLength;
  }
  ```
+ 有时候我们需要写两个版本的功能一样的函数，他们的区别只是一个是const成员函数，而另外一个是
  非常量函数。可能会出现下面的情况
  ```
  class TextBlock{
    public:
       ...
       const char& operator[](std::size_t position) consts{
         ...
         ...
         ...
         return text[position];
       }
       char& operator[](std::size_t position)
       {
         ...
         ...
         ...
         return text[position];
       }
   private:
     std::string text;
  };
  ```
  可以看到上面的代码有很多的重复，导致效率下降。我们现在需要做的是实现operator[]一次然后使用
  它两次。即用其中一个调用另外一个。
  这种情况下如果将返回值的const转除是安全的，因为不论谁调用non-const operator[]都一定首先有个
  non-const对象，否则就不能够调用 non-const 函数。所以令 non-const operator[]调用其const兄弟是
  个避免代码重复的安全做法即使过程中需要一个转型动作。如下
  ```
  class TextBlock{
    public:
       ...
       const char& operator[](std::size_t position) consts{
         ...
         ...
         ...
         return text[position];
       }
       char& operator[](std::size_t position){
         return const_cast<char&>(static_cast<const TextBlock&>(*this)[position])
       }
  };
  ```
  上面的代码在一个在non-const成员函数中调用const成员函数，为了实现这个功能，必须先将*this即
  原始类型转换为const TextBlock&。然后从const operator[]的返回值中移除const。
  上面将non-const转变为const对象，强迫进行了一次安全转型，使用了static_cast。然后移除const
  只能使用const_cast完成
+ 运用const成员函数实现出其non-const孪生兄弟"是很好的。更值得了解的是，反向做法一一令const版本
  调用non-const版本以避免重复，并不是你该做的事。因为对象有可能被改变。
+ 所以
  1. 将某些东西声明为const可帮助编译器侦测出错误用法。const可被施加于任何作用域内的对象、
  函数参数、函数返回类型、成员函数本体。
  2. 编译器强制实施 bitwise constness ，但你编写程序时应该使用"概念上的常量性".
  3. 当const和non-const成员函数有着实质等价的实现时，令non-const版本调用const版本可避免代
  码重复。
## item 4
