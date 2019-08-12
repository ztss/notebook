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
+ 尽量以const，enum,inline替换#define.
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
+ 尽可能使用const。
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
## item4 Make sure that objects are initialized before they're used
+ 确定对象被使用前巳先被初始化。
+ 确定对象在使用前已先被初始化，对于内置类型以外的类型，确保每一个构造函数都将对象的每一个
  成员初始化。
+ C++ 规定，对象的成员变量的初始化动作发生在进入构造函数本体之前。所以，一般构造函数采用
  member initialized list替换赋值动作。
  ```
  ABEntry::ABEntry(const std::string& name,const std::string address,
    const std::list<PhoneNumber>& phones):
    theName(name),
    theAddress(adress),
    thePhones(phones),
    numTimesConsulted(0){  }
  ```
  为什么使用这种方法呢？因为初始值列表只调用拷贝构造函数，比先调用默认构造函数然后再调用拷贝赋值
  操作符要快得多。总是使用成员初值列。这样做有时候绝对必要，且又往往比赋值更高效。
+ 将"内置型成员变量"明确地加以初始化，而且也确保你的构造函数运用"成员初值列"初始化base classes
  和成员变量。还要关注不同编译单元内定义之non-local static对象的初始化次序。
+ static对象包括global对象、定义于namespace作用域内的对象、在classes内、在函数内、以及在file
  作用域内被声明为static的对象。函数内的static对象称为local static对象(因为它们对函数而言是
  local) ，其他static对象称为non-localstatic对象。程序结束时static对象会被自动销毁，也就
  是它们的析构函数会在main()结束时被自动调用。
+ 所谓编译单元<translation unit)是指产出单一目标文件(single object file)的那些源码。基本上
  它是单一源码文件加上其所含入的头文件(#include files)。
+ 我们关注一个问题，它涉及两个源码文件，每一个内含至少一个non-local static对象，如果某个编译
  单元的某个non-local static对象的初始化动作使用了另一个编译单元内的某个non-local static对象。
  它所用到的这个对象可能未被初始化。因为 C++ 对"定义于不同编译单元内的non-local static对象"的
  初始化次序并无明确定义。怎么解决这个问题呢？将每个non-local static对象搬到自己的专属函数内
  (该对象在此函数内被声明为static)。这些函数返回一个reference指向它所含的对象。然后用户调用
  这些函数，而不直接指涉这些对象。换句话说，non-local static对象被local static对象替换了。
  因为C++保证，函数内的local static对象会在该函数被调用期间首次遇上该对象的定义式的时候被初始化。
  ```
  class FileSystem{
    public:
       ...
       std::size_t numDisks() const;
       ...
  };
  extern FileSystem tfs;

  //来自另一个程序库
  class Directory{
    public:
       Directory(params);
       ...
  };
  Directory::Directory(params){
    ...
    std::size_t disks=tfs.numDisks();
  }

  Directory tempDir(params);
  上面的例子就会反映不同编译单元内的non-local static对象初始化顺序的问题。
  ```
  可以改为
  ```
  class FileSystem { ... );
  FileSystem& tfs(){
    static FileSystem fs;
    return fs;
  }

  //另一个程序库中
  class Directory { ... );
  Directory::Directory(params){
      ...
      std::size_t disks=tfs.numDisks();
  }
  Directory& tempDir(){
    static Directory td;
    return td;
  }
  ```
  由于这种类型的reference-returning函数十分简短，所以可以写成inline的。
+ 所以
  1. 为内置型对象进行手工初始化，因为C++不保证初始化它们。
  2. 造函数最好使用成员初值列(member initialization list)，而不要在构造函数本体内使用赋值
  操作(assignment)。初值列列出的成员变量，其排列次序应该和它们在class中的声明次序相同。
  3. 为免除"跨编译单元之初始化次序"问题，请以local static对象替换non-local static对象。


# 构造，析构，赋值运算
## item5 Know what functions C++
+ 了解C++默默编写并调用哪些函数。
+ 如果你打算在一个"内含reference成员"的class内支持赋值操作(assignment)，你必须自己定义copy
  assignment操作符。因为再C++中引用本身不允许被改变，即让引用改指向不同对象。(可能是指引用
  的地址值不允许被改变)。
+ 所以
  1. 编译器可以暗自为class创建default构造函数、copy构造函数、copyassignment操作符，以及
  析构函数。
## item6 Explicitly disallow the use of compiler-generated functions you not want
+ 若不想使用编译器自动生成的函数，就该明确拒绝。
+ 有时候我们希望禁止拷贝操作。即我们希望禁止拷贝构造函数和拷贝赋值函数。我们可以将这些函数
  声明为private的，这样编译器不会自动生成他们，而且其他人也不能调用它们。而且为了阻止member
  函数和friend函数调用private函数，可以只声明而不定义它们。如下面
  ```
  class HomeForSale{
    public:
       ...
    private:
       ...
       HomeForSale(const HomeForSale&);
       HomeForSale& operator=(const HomeForSale&);//只有声明
  };
  ```
  或者使用一个uncopyable基类。然后用类继承这个uncopyable类
  ```
  class Uncopyable{
  protected:
     Uncopyable(){}
     ~Uncopyable(){}
  private:
     Uncopyable(const Uncopyable&);
     Uncopyable& operator=(const Uncopyable&);//阻止copying
  };

  class HomeForSale:private Uncopyable{
    ...
  };
  ```
  当有人调用拷贝操作尝试操作HomeForSale对象，编译器就会生成一个拷贝构造函数和拷贝赋值操作符。
  这些生成的函数会调用基类中的对应兄弟，但是由于基类中的拷贝函数是private的，所以这些调用
  会被编译器拒绝。
+ 所以
  1. 为驳回编译器自动(暗自)提供的机能，可将相应的成员函数声明为private并且不予实现。使用
    像Uncopyable这样的base class也是一种做法。
## item7 Declare destructors virtual in polymorphic base classes
+ 为多态基类声明virtual析构函数。
+ 当我们设计了一个factory函数，他会返回一个基类指针，并且指向新生成的派生类对象。这样在这个对象被
  销毁的时候，它是由基类指针删除的，那么它派生的成员却没有被销毁。所以我们要给基类一个虚析构函数。
+ 请确保在一个类中不是只有虚析构函数，还应该有其他的虚函数。因为如果class不含virtual函数，通常表示
  它并不意图被用做一个base class。当class不企图被当作base class，令其析构函数为virtual往往是个馊
  主意。
  ```
  class Point{
     public:
        Point(int xcoord,int ycoord);
        ~Point();
     private:
        int x,y;
  };
  ```
  如果一个int占用32bit,上面的Point对象可以存入一个64bit的缓存器中，然后这样的一个point对象
  可以当作64bit量传给其他语言C或者Fortran的函数。如果point的析构函数是虚函数。那么由于point
  对象就有多出一个指向虚函数表的指针，那么大小就变为96bit了，那么这个对象就不能放入64bit的缓
  存器中了。那么C++的point对象就不会和其他语言内的相同声明具有一样的结构了，因为其他语言没有
  实现vptr(virtual table pointer)。所以就不能将他传递至其他语言缩写的函数。所以。
  只有当class内含至少一个virtual函数， 才为它声明virtual析构函数。
+ 不要继承一个没有虚析构函数的类。而且如果一些类设计为只是为了当作基类使用，而不实现多态，那么
  就没有必要实现虚析构函数。
+ 所以
  1. polymorphic(带多态性质的)base classes应该声明一个virtual析构函数。如果class带有任何
  virtual函数，它就应该拥有一个virtual析构函数。
  2. Classes的设计目的如果不是作为base classes使用，或不是为了具备多态性(polymorphically)，
  就不该声明virtual析构函数。
## item8 Prevent exception from leaving destructors
+ 别让异常逃离析构函数。
+ 假如析构函数必须执行一个动作，而该动作可能会在失败的时候抛出异常。比如
  ```
  class DBConnection{
    public:
       ...
       static DBConnection create();


       void close();//关闭联机，失败抛出异常
  };
  为了确保客户不忘记在DBConnection对象上调用close()。我们创建一个管理这个类的class。并且在类
  的析构函数中调用close。
  class DBConn{
    public:
      ...
      ~DBConn(){
        db.close();
      }
    private:
      DBConnection db;
  };
  ```
  如果析构函数没有问题，那么就最好，但是如果DBConn析构函数导致异常，就会出现问题。
  可以用下面的方法解决这个问题
  1. 如果close抛出异常就结束程序。使用abort完成
  ```
  DBConn::~DBConn(){
    try{ db.close(); }
    catch(...){
      ...
      std::abort();
    }
  }
  ```
  2. 吞掉因调用close而发生的异常
  ```
  DBConn::~DBConn()
  {
    try{ db.close(); }
    catch(...) {
      ...
    }
  }
  ```
+ 所以
  1. 构函数绝对不要吐出异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何
  异常，然后吞下它们(不传播)或结束程序。
  2. 如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么class应该提供一个普通函数
  (而非在析构函数中)执行该操作。
## item9 Never call virtual functions during construction or destruction.
+ 绝不在构造和析构过程中调用virtual函数。
+ 不要在构造函数和析构函数期间调用virtual函数。比如说，如果在派生类构造的过程中，肯定是先
  调用基类的构造函数，而如果基类的构造函数中调用了虚函数，那么在基类的构造过程中肯定是调用
  基类里的虚函数，而不是目标要构造的派生类里的继承虚函数。而如果基类是个抽象基类，那么基类里
  的虚函数就是纯虚函数，这个时候这个基类里的虚函数就不会被定义，就会出现错误。
+ 还有一种说法
  ```
  class Transaction{
    public:
      Transaction(){
        init();//将多个构造函数中相同的工作放入一个init函数中，这个函数放在private里面
      }
      virtual void logTransaction() const = 0;
      ...
    private:
      void init(){
        ...
        logTransaction();//这里调用虚函数
      }
  };
  ```
  上面这个代码在构造函数中调用了虚函数，但是很难被发现。所以
  确定你的构造函数和析构函数都没有(在对象被创建和被销毁期间)调用virtual函数，而它们调用的
  所有函数也都服从同一约束。
  如何解决这个问题呢？可以在class transaction内将logTransaction函数改为非虚函数，然后
  要求派生类的构造函数传递信息给transaction构造函数，然后这个构造函数就可以调用非虚的
  logTransaction。
  ```
  class Transaction{
    public:
      explicit Transaction(const std::string& logInfo);
      void logTransaction(const std::string& logInfo) const;
      ...
    private:
  };
  Transaction::Transaction(const std::string& logInfo){
    ...
    logTransaction(logInfo);
  }

  class BuyTransaction: public Transaction{
    public:
      BuyTransaction(params):
        Transaction(createLogString(params)){...}
      ...
    private:
      static std::string createLogString(params);
      //由于static修饰的类成员属于类，不属于对象，因此static类成员函数是没有this指针的，this
      //指针是指向本对象的指针。正因为没有this指针，所以static类成员函数不能访问非static的类
      //成员，只能访问 static修饰的类成员。这样就不会访问Buytransaction对象构造期间尚未初始化
      //的成员变量。
  };
  ```
+ 所以
  1. 在构造和析构期间不要调用virtual函数，因为这类调用从不下降至derived class(比起当前执行
  构造函数和析构函数的那层)。
## item10 Hava assignment operators return a reference to * this
+ 令 operator=返回一个reference to * this。
+ 令operator=返回一个reference to * this。例如
  ```
  class Widget{
    public:
      ...
      Widget& operator=(const widget& rhs)
      {
        ...
        retrun *this;
      }
  };
  ```
+ 所以
  1. 令赋值(assignment)操作符返回一个reference to * this。
## item11 Handle assignment to self in operator=
+ 在operator=中处理"自我赋值"
+ 避免自我赋值，w=w,a[i]=a[j],* px=* py。这些都有可能是自我赋值。看以下代码
  ```
  Widget& Widget::operator=(const Widget& rhs)
  {
    delete pb;
    pb=new Bitmap(*rhs.pb);
    return *this;
  }
  ```
  如果widget类出现了自我赋值，那么就会出现问题，因为他把自身先delete了。可以将上面的函数改为
  ```
  Widget& Widget::operator=(const Widget& rhs)
  {
    if(this==&this)
      return *this;//identity test
    delete pb;
    pb=new Bitmap(*rhs.pb);
    return *this;
  }
  ```
  上面的第一版操作符重载函数不具备自我赋值安全性和异常安全性，而第二版仍然存在异常方面的麻烦。
  因为如果"new Bitmap"导致异常(不论是因为分配时内存不足或因为Bitmap的copy构造函数抛出异常)
  ,Widget最终会持有一个指针指向一块被删除的Bitmap。
  很多时候，我们只要让operator=具备异常安全性，那么就会自动的获得自我赋值安全性。如下
  ```
  Widget& Widget::operator=(const Widget& rhs)
  {
    Bitmap* pOrig=pb;
    pb=new Bitmap(*rhs.pb);
    delete pOrig;
    return *this;
  }
  ```
  我们注意在赋值pb之前别删除pb就行了。而如果new Bitmap抛出异常，那么pb可以保持原状。也可以
  使用所谓的copy and swap技术
  ```
  class Widget{
    ...
    void swap(Widget& rhs);
    ...
  };
  Widget& Widget::operator=(const Widget& rhs){
    Widget temp(rhs);//为rhs数据制作一份副本
    swap(temp);//将*this数据和上述副本的数据交换
    return *this;
  }
  ```
  或者，也可以这么写，将operator=接受参数的方式改为传值方式，那么他在operator=函数参数构造
  阶段就会产生拷贝，这样可以节省程序运行时间。
  ```
  Widget& Widget::operator=(Widget rhs){//rhs是被传对象一个副本
    swap(rhs);//将*this数据和上述副本的数据交换
    return *this;
  }
  ```
+ 所以
  1. 确保当对象自我赋值时operator=有良好行为。其中技术包括比较"来源对象" 和"目标对象"的地址、
  精心周到的语句顺序、以及copy-and-swap。
  2. 确定任何函数如果操作一个以上的对象，而其中多个对象是同一个对象时，其行为仍然正确。
## item12 Copy all parts of an object
+ 赋值对象时不要忘记每一个成分。
+ 考虑这样一种情况，如果你在一个类中新添加了一个数据成员，那么你就必须要修改class的所有构造
  函数以及任何非标准形式的operator=。
+ 任何时候只要你承担起"为derived class撰写copying函数"的重责大任，必须很小心地也复制其base
  class成分。那些成分往往是private(见条款22) ，所以你无法直接访问它们，你应该让derived class
  的copying函数调用相应的base class函数。
+ 所以
  1. Copying函数应该确保复制"对象内的所有成员变量"及"所有base class成分"。
  2. 不要尝试以某个copying函数实现另一个copying函数。应该将共同机能放进第三个函数中，并由
  两个coping函数共同调用。


# 资源管理
+ 资源一旦用了，必须还给系统。C++中的资源包括动态分配分配，文件描述器，互斥锁，图形界面中的字型
  和笔刷。数据库连接，网络的sockets。
## item13 Use objects to manage resources
+ 以对象管理资源。
+ 看以下继承体系
  ```
  class Investment {...};//root class
  一个factory function
  Investment* createInvestment();//返回指针，指向investment继承体系内的动态分配对象，
  //调用者有责任删除它，省略了参数
  //删除函数
  void f(){
    Investment* pInv=createInvestment();
    ...
    delete pInv;
  }
  ```
  上面的删除函数可能不会走到最后一步，因为删除函数中间可能抛出异常。为确保createlnvestment
  返回的资源总是被释放，我们需要将资源放进对象内，当控制流离开f，该对象的析构函数会自动释放
  那些资源。
+ 可以使用智能指针auto_ptr。许多资源被动态分配于heap内而后被用于单一区块或函数内。它们应该
  在控制流离开那个区块或函数时被释放。auto_ptr就是实现这个的。
  ```
  void f(){
    std::auto_ptr<Investment> pInv(createInvestment());
    ...
  }
  ```
  这里体现了两个以对象管理资源的想法
  1. 获得资源后立刻放入管理对象内。
  2. 管理对象运用析构函数确保资源被释放。不论控制流如何离开区块，一旦对象离开作用域其析构
  函数就会自动调用，释放资源。
+ 由于auto ptr被销毁时会自动删除它所指之物，所以一定要注意别让多个auto_ptr同时指向同一对象。
  ```
  std::auto_ptr<Investment> pInv(createInvestment());
  std::auto_ptr<Investment> plnv2(plnvl);//现在pInv2指向对象，pInv1被设为null。
  plnvl = plnv2;//现在pInv1指向对象，pInv2设为null。
  ```
+ 这一诡异的复制行为，复加上其底层条件:"受auto_ptrs管理的资源必须绝对没有一个以上的
  auto_ptr同时指向它"，意味auto_ptrs并非管理动态分配资源的神兵利器。举个例子，STL容器
  要求其元素发挥"正常的"复制行为，因此这些容器容不得auto_ptr。
  所以可以使用shared_ptr，即引用计数型智慧指针：reference-counting smart pointer。RCSPs
  提供的行为类似垃圾回收，不同的是它无法打破环状引用(cycles of references，例如两个其实已经
  没被使用的对象彼此互指，因而好像还处在"被使用"状态)。
  ```
  void f(){
    ...
    std::tr1::shared_ptr<Investment> pInv(createInvestment());
    ...
  }
  ```
+ auto_ptr和trl::shared_ptr两者都在其析构函数内做delete而不是delete[]动作(条款16对两者
  的不同有些描述)。那意味在动态分配而得的array身上使用auto_ptr或trl::shared_ptr是个馊主意。
+ 所以
  1. 为防止资源泄漏，请使用RAII对象，它们在构造函数中获得资源并在析构函数中释放资源。
  2. 两个常被使用的RAII classes分别是trl::shared_ptr和auto ptr。前者通常是较佳选择，因为
  其copy行为比较直观。若选择auto_ptr，复制动作会使它(被复制物)指向null。
## item14 Think carefully about copying behavior in resource-managing classes.
+ 在资源管理类中小心coping行为。
+ 资源取得时机就是初始化时机(resource Acquisition is initialization,RAII)。
+ class的基本结构由RAII守则支配，也就是"资源在构造期间获得，在析构期间释放"。
+ 当一个RAII对象被复制的时候，我们需要选择两种做法
  1. 禁止复制，这个时候参考item6，将copying操作声明为private。
  2. 对底层资源使用引用计数法，即使用tr1::shared_ptr。
+ 所以
  1. 复制RAII对象必须一并复制它所管理的资源，所以资源的copying行为决定RAIl对象的copying行为。
  2. 普遍而常见的RAII class copying行为是:抑制copying、施行引用计数法(reference counting)。
  不过其他行为也都可能被实现。
## item15 Provide access to raw resource in resource-managing classes
+ 在资源管理类中提供对原始资源的访问。
+ 假如
  ```
  std::trl::shared ptr<Investment> pInv(createInvestment());
  然后定义这样一个函数
  int daysHeld(const Investment* pi);
  想要调用函数
  int days=daysHeld(pInv);
  这样行不通，因为daysheld需要的是Investment*指针，你传递的是个tr1::shared_ptr<Investment>
  对象
  所以需要将RAII class对象(即tr1::shared_ptr)转换为内含的原始资源(即Investment*)。
  可以使用显式转换或者隐式转换。
  ```
  1. 使用get成员函数进行显式转换。
  ```
  int days=daysHeld(pInv.get());
  ```
  2. 通过重载的指针取值操作符(operator-> operator*)，隐式转换到底部原始指针。
  ```
  class Investment {
    public: bool isTaxFree() const;
    ...
  };
  Investment* createInvestment(); //factory函数
  std::trl::shared_ptr<Investment> pil(createInvestment());
  bool taxablel = !(pil->isTaxFree());
  std::auto_ptr<Investment> pi2(createInvestment());
  bool taxable2 = !((pi2).isTaxFree());
  ```
+ 可以使用隐式转换函数，即形如operator FontHandle() const {return f;}。其中f为RAII类中的
  底层资源。
+ 所以
  1. APls往往要求访问原始资源(raw resources)，所以每一个RAII class应该提供一个"取得其所
  管理之资源"的办法。
  2. 原始资源的访问可能经由显式转换或隐式转换。一般而言显式转换比较安全，但隐式转换对客户
  比较方便。
## item16 Use the same form in corresponding uses of new and delete
+ 成对使用new和delete时要采取相同形式。
+ 使用delete需要知道即将被删除的那个指针，所指的是单一对象或对象数组?这是个必不可缺的问题，
  因为单一对象的内存布局一般而言不同于数组的内存布局。数组所用的内存通常还包括"数组大小"的
  记录，以便delete知道需要调用多少次析构函数。
+ 所以，如果你调用new时使用[].你必须在对应调用delete时也使用[]。如果你调用new时没有使用[].
  那么也不该在对应调用delete时使用[]。
+ 所以，为了避免delete的时候操作失误，我们一般不对数组形式使用typedefs动作。
+ 所以
  1. 如果你在new表达式中使用[].必须在相应的delete表达式中也使用[]。如果你在new表达式中不使用[].
  一定不要在相应的delete表达式中使用[]。
## item17 Store newed objects in smart pointers in standalone statements
+ 以独立语句将newed对象置入智能指针.
+ 我们有下面的两个函数
  ```
  int priority();
  void processWidget(std::tr1::shared_ptr<Widget> pw,int priority);
  ```
  不要使用下面的语句调用函数
  ```
  processWidget(std::trl::shared ptr<Widget> (new Widget) , priority());
  ```
  这个函数可能导致内存泄漏
  因为在创建shared_ptr的时候，可能先调用priority()，如果它调用失败，那么就不会发生异常。
  而是使用下面这样的形式
  ```
  std::tr1::shared_ptr<Widget> pw(new Widget);

  processWidget(pw,priority());
  ```
+ 所以
  1. 以独立语句将newed对象存储于(置入)智能指针内。如果不这样做，一旦异常被抛出，有可能导致
  难以察觉的资源泄漏
