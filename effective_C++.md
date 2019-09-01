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
  local) ，其他static对象称为non-local static对象。程序结束时static对象会被自动销毁，也就
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
  class FileSystem { ... };
  FileSystem& tfs(){
    static FileSystem fs;
    return fs;
  }

  //另一个程序库中
  class Directory { ... };
  Directory::Directory(params){
      ...
      std::size_t disks=tfs().numDisks();
  }
  Directory& tempDir(){
    static Directory td;
    return td;
  }
  然后这样使用
  Directory tempDir();
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


# 设计和声明
## item18 Make interfaces easy to use correctly and hard to use incorrectly
+ 让接口容易被正确使用，不易被误用。
+ 定义一个month可以这样使用
  ```
  class Month{
    public:
      static Month Jan(){return Month(1);}
      ...
      static Month Dec() { return Month(12); }
      ...
    private:
      explicit Month(int m);//阻止生成新的月份，这是月份专属数据
  };
  ```
+ 预防客户错误的另一个办法是，限制类型内什么事可做，什么事不能做。常见的限制是加上const。
  除非有好理由，否则应该尽量令你的types的行为与内置types一致。
+ 避免无端与内置类型不兼容，真正的理由是为了提供行为一致的接口。
+ 任何接口如果要求客户必须记得做某些事情，就是有着"不正确使用"的倾向。例如这个函数
  ```
  Investment* createInvestment();
  ```
  为避免资源泄漏， createInvestment返回的指针最终必须被删除，但那至少开启了两个客户错误机会
  :没有删除指针，或删除同一个指针超过一次。所以，最好的设计是
  ```
  std::trl::shared_ptr<Investment> createInvestment();
  ```
  而为了使用正确的资源析构机制，我们可以将删除器绑定在智能指针上。而为了shared_ptr第一个参数
  要求一定是一个指针，我们使用cast转换
  ```
  std::tr1::shared_ptr<Investment> createInvestment(){
    std::tr1::shared_ptr<Investment> retVal(static_cast<Investment*>(0),getRidOfInvestment);
    retVal=...;
    return retVal;
  }
  ```
+ 当然啦，如果被pInv管理的原始指针(raw pointer)可以在建立plnv之前先确定下来，那么"将原始指
  针传给plnv构造函数"会比"先将pInv初始化为null再对它做一次赋值操作"为佳。
+ trl::shared_ptr有一个特别好的性质是它会自动使用它的"每个指针专属的删除器"，因而消除另一个
  潜在的客户错误:所谓的"cross-DLLproblem"。这个问题发生于"对象在动态连接程序库(DLL)中被new
  创建，却在另一个DLL内被delete销毁"。
+ 所以
  1. 好的接口很容易被正确使用，不容易被误用。你应该在你的所有接口中努力达成这些性质。
  2. "促进正确使用"的办法包括接口的一致性，以及与内置类型的行为兼容。
  3. "阻止误用"的办法包括建立新类型、限制类型上的操作，束缚对象值，以及消除客户的资源管理责任。
  4. trl::shared_ptr支持定制型删除器(custom deleter)。这可防范DLL问题，可被用来自动解除互
  斥锁 (mutexes; 见条款 14) 等等。
## item19 Treat class design as type design
+ 设计class犹如设计type。
+ 那么，如何设计高效的classes呢？
  1. 新type的对象应该如何被创建和销毁，这会影响class的构造函数和析构函数以及内存分配函数
  和释放函数。
  2. 对象的初始化和对象的赋值该有什么样的差别？决定构造函数和赋值操作符的行为。
  3. 新type的对象如果被passed by value,意味着什么？copy构造函数用来顶一个一个type的pass
  by value该如何实现。
  4. 什么是新type的合法值。对class的成员变量而言，通常只有某些数值集是有效的。那些数值集
  决定了你的class必须维护的约束条件(invariants) .也就决定了你的成员函数(特别是构造函数、
  赋值操作符和所谓 "setter" 函数)必须进行的错误检查工作。它也影响函数抛出的异常、以及
  (极少被使用的)函数异常明细列(exception specifications)。
  5. 你的新type需要配合某个继承图系么？如果你继承自某些既有的classes，你就受到那些classes
  的设计的束缚，特别是受到"它们的函数是virtual或non-virtual"的影响(见条款 34 和条款 36)。
  如果你允许其他classes继承你的class ，那会影响你所声明的函数尤其是析构函数是否为virtual。
  6. 你的新type需要什么样的转换？如果你希望允许类型Tl之物被隐式转换为类型T2之物，就必须在
  class Tl内写一个类型转换函数(operator T2)或在class T2内写一个non-explicit-one-argument
  (可被单一实参调用)的构造函数。如果你只允许explicit构造函数存在，就得写出专门负责执行转换的
  函数，且不得为类型转换操作符(type conversion operators)或non-exp1icit-one-argument构造函数。
  7. 什么样的操作符和函数对新type是合理的？决定你为class声明那些函数。其中某些该是member函数
  而某些不是。
  8. 什么样的标准函数应该驳回？决定那些必须声明为private。
  9. 谁该取用新type成员？决定那个成员为public，那个为protected，那个为private。
  10. 什么是新type的未声明接口？你在这些方面提供的保证将为你的class实现代码加上相应的约束条件。
  11. 你的新type有多么一般化？或许你其实并非定义一个新type，而是定义一整个types家族。果真如此
  你就不该定义一个新class，而是应该定义一个新的class template。
  12. 你真的需要一个新type么？如果只是定义新的derived class以便为既有的class添加机能，那么
  说不定单纯定义一或多个non-member函数或templates.更能够达到目标。
+ 所以
  1. Class的设计就是type的设计。在定义一个新type之前，请确定你己经考虑过本条款覆盖的所有讨论主题。
## item20 Prefer pass-by-reference-to-const to pass-by-value.
+ 宁以pass-by-reference-to-const替换pass-by-value。
+ 有一种情况，如下一个类
  ```
  class Window{
    public:
      ...
      std::string name() const;
      virtual void display() const;
  };

  class WindowWithScrollBars: public Window{
    ...
    virtual void display() const;
  };
  然后有这样一个函数
  void printNameAndDisplay(Window w)
  {
    std::cout << w.name();
    w.display();
  }
  ```
  上面这个函数有个缺陷，当你这样调用函数的时候
  ```
  WindowWithScrollBars wwsb;
  printNameAndDisplay(wwsb);
  ```
  由于printNameAndDisplay函数是个传值函数，所以wwsb会被切割，它会变成window对象，而只会
  调用window::display函数。所以为了解决切割问题，我们需要传递一个引用进去
  ```
  void printNameAndDisplay(const Window& w)
  {
    std::cout << w.name();
    w.display();
  }
  ```
+ 然而，因此如果你有个对象属于内置类型(例如int) , pass by value往往比pass by reference
  的效率高些。
+ 所以
  1. 尽量以pass-by-reference-to-const替换pass-by-valueo前者通常比较高效，并可避免切割
  问题 (slicing problem)。
  2. 以上规则并不适用于内置类型，以及STL的迭代器和函数对象。对它们而言，pass-by-value往往
  比较适当。
## item21 Don't try to return a reference when you must return an object
+ 必须返回对象时，别妄想返回其reference。
+ 如果我们有这样一个函数
  ```
  const Rational& operator* (const Rational& lhs , const Rational& rhs){
    Rational result(lhs.n * rhs.n , lhs.d * rhs.d);
    return result;
  }
  ```
  上面这个函数要求返回一个引用，但是你返回了一个已经被析构的对象的引用，因为函数体内构造的
  栈空间的临时变量，所以必须的在heap内构造一个对象，并且返回一个引用。使用new构造
  ```
  const Rational& operator* (const Rational& lhs , const Rational& rhs){
    Rational *result=new Rational(lhs.n * rhs.n , lhs.d * rhs.d);
    return *result;
  }
  ```
  但是返回的对象怎么被delete呢？我们没有合理的办法取得operator*返回的reference背后隐藏的
  指针。这将导致资源泄露
  所以，当一个函数必须返回一个新对象，那么只能让函数返回一个新对象。
  ```
  inline const Rational operator * (const Rational& lhs, const Rational& rhs)
  {
    return Rational(lhs.n * rhs.n, lhs.d * rhs.d);
  }
  ```
+ 所以
  1. 绝不要返回pointer或reference指向一个local stack对象，或返回reference指向一个heap
  -allocated对象，或返回pointer或reference指向一个local static对象而有可能同时需要多个
  这样的对象。条款4已经为"在单线程环境中合理返回reference指向一个local static对象"提供了
  一份设计实例。
## item22 Declare data members private
+ 将成员变量声明为private。
+ 如果成员变量不是public，客户唯一能够访问对象的办法就是通过成员函数。如果public接口内的每
  样东西都是函数，客户就不需要在打算访问class成员时迷惑地试着记住是否该使用小括号(圆括号)。
  他们只要做就是了，因为每样东西都是函数。
+ 使用函数可以让你对成员变量的处理有更精确的控制。如果你令成员变量为public，每个人都可以读写
  它，但如果你以函数取得或设定其值，你就可以实现出"不准访问"、 "只读访问" 以及"读写访问"。
+ 将成员变量隐藏在函数接口的背后，可以为"所有可能的实现"提供弹性。例如这可使得成员变量被读
  或被写时轻松通知其他对象、可以验证class的约束条件以及函数的前提和事后状态、可以在多线程
  环境中执行同步控制……等等。
+ 假设我们有一个protected成员变量，而我们最终取消了它，有多少代码被破坏?晤，所有使用它的
  derived classes都会被破坏，那往往也是个不可知的大量。
+ 从封装的角度观看，其实只有两种访问权限：private(提供封装的)和其他(不提供封装)。所以protected
  并不封装。
+ 所以
  1. 切记将成员变量声明为private。这可赋予客户访问数据的一致性、可细微划分访问控制、允诺
  约束条件获得保证，并提供class作者以充分的实现弹性。
  2. protected并不比public更具封装性。
## item23 Prefer non-member non-friend functions to member function
+ 宁以non-member、non-friend替换member函数。
+ 面向对象守则要求数据应该尽可能被封装，然而与直观相反地， member函数clearEverything带来
  的封装性比non-member函数clearBrowser低。此外，提供non-member函数可允许对WebBrowser相
  关机能有较大的包裹弹性(packaging flexibility)，而那最终导致较低的编译相依度，增加WebBrowser
  的可延伸性。
+ 愈多函数可访问它，数据的封装性就愈低。能够访问private成员变量的函数只有class的member函数
  加上friend函数而已。
+ 如果要你在一个member函数(它不只可以访问class内的private数据，也可以取用private函数、
  enums 、typedefs等等)和一个non-member，non-friend函数(它无法访问上述任何东西)之间做抉择，
  而且两者提供相同机能，那么，导致较大封装性的是non-member non-friend函数，因为它并不增加"
  能够访问class内之private成分"的函数数量。
+ 还有就是成为一个类的非成员函数并不代表这个函数不可以是其他类的成员函数，因为其他类的成员
  函数显然不能访问本类的private成员。
+ 当所有同一类的便利函数放在同一个头文件，但是不同类型的便利函数放在不同的头文件但是放在同
  一个命名空间，这是C++标准程序库的组织方式，这样可以减少编译相依赖关系，因为用户可能只关注
  一个类某一个功能。而这个功能则可能包含多个相关函数。这样客服也可以轻松的扩展这一组便利函数。
  只需要将更多的非成员函数，非友元函数添加进去即可。
+ 例如，如果某个WebBrowser客户决定写些与影像下载相关的便利函数，他只需要在WebBrowserStuff
  命名空间内建立一个头文件，内含那些函数的声明即可。新函数就像其他旧有的便利函数那样可用且
  整合为一体。这样也是我们宁愿定义非成员函数的一个理由。
+ 所以
  1. 宁可拿non-member non-friend函数替换member函数。这样做可以增加封装性、包裹弹性(packaging
  flexibility)和机能扩充性。
## item24 Declare non-member functions when type conversions should apply to all parameters
+ 若所育参数皆需类型转换，请为此采用non-member函数。
+ 令classes支持隐式类型转换通常是个糟糕的主意。当然这条规则有其例外，最常见的例外是在建立
  数值类型时。假设你设计一个class用来表现有理数，允许整数"隐式转换"为有理数似乎颇为合理。
+ 如果我们将operator*定义为类的成员函数，比如
  ```
  const Rational operator* (const Rational& rhs) const;
  ```
  那么，如果我们使用下面两个式子进行调用
  ```
  result = oneHalf * 2;//正确
  result = 2 * oneHalf;//错误
  为什么呢，请看
  result = oneHalf.operator*(2);
  result = 2.operator*(oneHalf);
  一一对应，第一个式子将2隐式转换为重载*的类。而第二个式子2没有operator*，所以显然调用失败
  ```
  所以为了让我们的乘法可以满足交换律，那么就需要让operator*成为非成员函数，这样允许编译器
  在每一个实参身上执行隐式类型转换：
  ```
  class Rational{
    ...
  };
  const Rational operator*(const Rational& lhs , const Rational& rhs){
    return Rational(lhs.numerator() * rhs.numerator( ), lhs.denominator() * rhs.denominator());
  }
  ```
  这样，前面的两个调用都能成功。还有就是，operator*是否应该是一个友元函数呢？
  记住，无论何时如果你可以避免friend函数就该避免，因为就像真实世界一样，朋友带来的麻烦往往多
  过其价值。
+ 所以
  1. 如果你需要为某个函数的所有参数(包括被this指针所指的那个隐喻参数)进行类型转换，那么这个
  函数必须是个non-member。
## item25 Consider support for a non-throwing swap
+ 考虑写出一个不抛异常的swap函数。
+ 缺省情况下swap动作可由标准程序库提供的swap算法完成
  ```
  namespace std{
    template<typename T>
    void swap(T& a,T& b){
      T temp(a);
      a=b;
      b=temp;
    }
  }
  ```
  只要类型T支持copying(通过copy构造函数和copy assignment操作符完成) ,缺省的swap实现代码
  就会帮你置换类型为T的对象，你不需要为此另外再做任何工作。a复制到temp，b复制到a，以及temp
  复制到b。但是对某些类型而言，这些复制动作无一必要:对它们而言swap缺省行为等于是把高速铁路
  铺设在慢速小巷弄内。
+ 例如，"以指针指向一个对象，内含真正数据"那种类型。
  ```
  class WidgetImpl{//针对Widget数据而设计的class;
    public:
      ...
    private:
    int a,b,c;
    std::vector<double> v;//意味复制时间很长
    ...
  };
  class Widget{//这个class使用 pimpl手法   pimpl是"pointerωimplementation
    public:
      Widget(const Widget& rhs);
      Widget& operator=(const Widget& rhs){
        ...
        *plmpl = *(rhs.plmpl);
        ...
      }
      ...
    private:
      Widgetlmpl* plmpl;//指针，所指对象内含Widget数据。
  };
  ```
  一旦要置换两个Widget对象值，我们唯一需要做的就是置换其plmpl指针。但是缺省的swap却做了太多
  其他的事情。
  所以我们可以将std::swap针对Widget特化。
  ```
  namespace std{
    template<>
    void swap<Widget>(Widget& a,Widget& b){
      swap(a.pImpl,b.pImpl);
    }
  }
  ```
  这个函数一开始的"template<>"表示它是std::swap的一个全特化(totaltemplate specialization)
  版本，函数名称之后的"<Widget>"表示这一特化版本系针对"T是Widget"而设计。
  这个函数无法通过编译，因为它企图访问a和b内的private成员。所以我们需要我们令Widget声明一个
  名为swap的public成员函数做真正的置换工作，然后将std::swap特化。
  ```
  class Widget{//这个class使用 pimpl手法   pimpl是"pointerωimplementation
    public:
      Widget(const Widget& rhs);
      Widget& operator=(const Widget& rhs){
        ...
        *plmpl = *(rhs.plmpl);
        ...
      }
      void swap(Widget& other){
        using std::swap;
        swap(pImpl,other.pImpl);
      }
      ...
    private:
      Widgetlmpl* plmpl;//指针，所指对象内含Widget数据。
  };

  namespace std{
    template<>
    void swap<Widget>(Widget& a,Widget& b){
      swap(a.pImpl,b.pImpl);
    }
  }
  ```
  而如果我们上面的Widget和WidgetImpl是类模板而不是类的话，上面的方法就不管用了。那么该怎么办呢？
  答案很简单，我们还是声明一个non-member swap让它调用member swap.但不再将那个non-member swap
  声明为std::swap的特化版本或重载版本。
  ```
  namespace WidgetStuff{
    ...
    class Widget{//这个class使用 pimpl手法   pimpl是"pointerωimplementation
      public:
        Widget(const Widget& rhs);
        Widget& operator=(const Widget& rhs){
          ...
          *plmpl = *(rhs.plmpl);
          ...
        }
        void swap(Widget& other){
          using std::swap;
          swap(pImpl,other.pImpl);
        }
        ...
      private:
        Widgetlmpl* plmpl;//指针，所指对象内含Widget数据。
    };
    ...
    template<typenarne T>
    void swap(Widget<T>& a , Widget<T>& b){
      a.swap(b);
    }
  }
  为了使我们这里的方法适用于更多的情况，我们还是定义std内的swap特化版本
  namespace std{
    template<>
    void swap<Widget>(Widget& a,Widget& b){
      swap(a.pImpl,b.pImpl);
    }
  }
  ```
+ 此刻，我们已经讨论过default swap、memberswaps、non-memberswap 、std::swap特化版本、
  以及对swap的调用，现在让我把整个形势做个总结。首先，如果swap的缺省实现码对你的class或
  class template提供可接受的效率，你不需要额外做任何事。任何尝试置换(swap)那种对象的人
  都会取得缺省版本，而那将有良好的运作。其次，如果swap缺省实现版的效率不足(那几乎总是意味
  你的class或template使用了某种pimpl手法) ，试着做以下事情：
  1. 提供一个public swap成员函数，让它高效地置换你的类型的两个对象值。稍后我将解释，这个
  函数绝不该抛出异常。
  2. 在你的class或template所在的命名空间内提供一个non-member swap，并令它调用上述swap成员函数。
  3. 如果你正编写一个class(而非class template) ，为你的class特化std::swap。并令它调用你的
  swap成员函数。
  4. 最后，如果你调用swap，请确定包含一个using声明式，以便让std::swap在你的函数内曝光可见，
  然后不加任何namespace修饰符，赤裸裸地调用swap。(这样使得编译器使用名称查找法则以找到最适合
  的swap版本)。即杜绝std::swap(objl, obj2);这种调用。
  还有就是成员版的swap绝不可抛出异常。
+ 所以
  1. 当 std::swap对你的类型效率不高时，提供一个swap成员函数，并确定这个函数不抛出异常。
  2. 如果你提供一个member swap，也该提供一个non-member swap用来调用前者。对于classes
  (而非templates) ，也请特化std::swap。
  3. 调用swap时应针对std::swap使用using声明式，然后调用swap并且不带任何"命名空间资格修饰"。
  4. "用户定义类型"进行std templates全特化是好的，但千万不要尝试在std内加入某些对std而言
  全新的东西。


# 实现
+ 过度使用转型可能导致代码变慢并且难以维护，又可能导致微妙难解的错误。
## item26 Postpone variable definitions as long as possible
+ 尽可能延后变量定义式的出现时间。
+ 只要定义了一个变量，那么在到达这个变量定义式的时候，就得承受构造成本，但变量离开作用域的时候
  就得承受析构成本。所以应该尽量避免这个情况。
+ 这让我们联想起本条款所谓"尽可能延后"的真正意义。你不只应该延后变量的定义，直到非得使用该变量
  的前一刻为止，甚至应该尝试延后这份定义直到能够给它初值实参为止。如果这样，不仅能够避免构造
  (和析构)非必要对象，还可以避免无意义的default构造行为。
+ 所以
  1. 尽可能延后变量定义式的出现。这样做可增加程序的清晰度并改善程序效率。
## item27 Minimize casting
+ 尽量少做转型动作。
+ C++有四种新式转换
  1. const_cast:cast通常被用来将对象的常量性转除(cast away the constness) 。它也是唯一有
  此能力的C++-style转型操作符。
  2. dynamic_cast:主要用来执行"安全向下转型" (safe downcasting) ，也就是用来决定某对象是
  否归属继承体系中的某个类型。它是唯一无法由旧式语法执行的动作，也是唯一可能耗费重大运行成本
  的转型动作。
  3. reinterpret_cast:执行低级转型，例如将一个pointer to int转型为一个int。
  4. static_cast:cast用来强迫隐式转换(implicit conversions) ，例如将non-const对象转为
  const对象。将void食指针转为typed指针，将pointer-to-base转为pointer-to-derived。
+ 尽量使用上面的新式转型。
+ 单一对象(例如一个类型为Derived的对象)可能拥有一个以上的地址(例如"以 Base* 指向它"时的地址
  和"以 Derived* 指向它"时的地址。真是因为C++中的多重继承。这在其他语言都是不会发生的。
+ 之所以需要dynamic cast，通常是因为你想在一个你认定为derived class对象身上执行derived class
  操作函数，但你的手上却只有一个"指向 base"的pointer或reference，你只能靠它们来处理对象。
+ 所以
  1. 如果可以，尽量避免转型，特别是在注重效率的代码中避免dynamic casts 。 如果有个设计需要
  转型动作，试着发展无需转型的替代设计。
  2. 如果转型是必要的，试着将它隐藏于某个函数背后。客户随后可以调用该函数，而不需将转型放进
  他们自己的代码内。
  3. 宁可使用 C++-style(新式)转型，不要使用旧式转型。前者很容易辨识出来，而且也比较有着分
  门别类的职掌。
## item28 Avoid returning handles to object internals
+ 避免返回handles指向对象内部成分。
+ 尽量避免const成员函数传出一个reference，因为这有可能导致函数的调用者可以修改那笔数据。
+ reference，指针和迭代器就是所谓的handles，即可以用来取得某个对象，而返回一个代表对象内部
  数据的Handel，就会有降低对象封装性的危险。可能导致虽然调用const成员函数却仍然造成对象状态
  被更改的后果。被声明为protected和private的函数也是对象内部的一部分，所以也不应该返回它们
  的handels，即不该令成员函数返回一个指针指向访问级别较低的成员函数。因为这样的话后者的访问
  级别就会提高如同前者。
+ 那么怎么避免这样的问题呢？我们可以在这些函数的返回类型上加上const，如下
  ```
  class Rectangle{
    public:
    ...
       const Point& upperLeft() const {
         return pdata->ulhc;
       }
  };
  ```
+ 上面的做法当然可以使得问题得到缓解，但是还有另外一个问题，就是导致dangling handles(空悬)。
  即handles所指向的东西已经不复存在。关键是，有个handle被传出去了，一旦如此就是暴露在handle
  比所指对象更长寿的风险之下。
+ 所以
  1. 避免返回handles(包括references、指针、迭代器)指向对象内部。遵守这个条款可增加封装性，
  帮助const成员函数的行为像个const，并将发生"虚吊号码牌"(dangling handles)的可能性降至最低。
## item29 Strive for exception-safe
+ 为异常安全而努力是值得的。
+ 看以下这个希望用于多线程环境的class。
  ```
  class Prettymenu{
    public:
       ...
       void changeBackground(std::istream& imgSrc);
       ...
    private:
      Mutex mutex;
      Image* bgImage;
      int imageChanges;
  };
  void Prettymenu::changeBackground(std::istream& imgSrc){
    lock(&mutex);
    delete bgImage;
    ++imageChanges;
    bgimage=new Image(imgSrc);
    unlock(&mutex);
  }
  ```
  从异常安全性来看，上面的更改背景图的函数很糟糕。异常安全有两个条件
  1. 当异常被抛出的时候，带有异常安全性的函数会不泄露任何资源。上面的函数就没有做到，因为一旦
  new Image(imgSrc)导致了异常，那么对于unlock的调用就不会执行，于是互斥器就永远被把持了。
  2. 不允许数据败坏，如果new Image(imgSrc)抛出异常，bgimage就指向一个已经被删除的对象，
  而imageChanges也已经累加了，但其实并没有新的图像安装起来。
+ 为了防止资源泄露，加入确保互斥器被及时释放的方法。可以用一个资源管理类来管理mutex。如下
  ```
  void Prettymenu::changeBackground(std::istream& imgSrc){
    Lock m1(&mutex);//使用资源管理类来管理mutex，即使new不成功，在函数执行完后，也会自动析构
    delete bgImage;
    ++imageChanges;
    bgImage=new Image(imgSrc);
  }
  ```
+ 对于数据败坏，首先介绍以下异常安全函数提供的三个保证，一般而言异常安全函数只需要提供以下三个
  之一即可。
  1. 基本承诺，如果异常被抛出，程序内的任何事物仍然保持在有效状态下。
  2. 强烈保证，如果异常被抛出，程序状态不改变。如果函数失败，程序会回复到调用函数之前的状态。
  3. 不抛掷保证，承诺绝不抛出异常。
  所以我们这个时候的抉择是应该为我们所写的函数提供哪一个保证。可能的话我们一般提供第三种保证，
  但是对于绝大部分函数来说，往往选择的是基本保证和强烈保证。
  比如说对于上面的函数我们如果给它提供强烈保证，那么只需要以智能指针管理资源即可。并且将累加
  次数加一放在更换图像成功之后。
  ```
  class Prettymenu{
    ...
    std::tr1::shared_ptr<Image> bgImage;
    ...
  };

  void Prettymenu::changeBackground(std::istream& imgSrc){
    Lock m1(&mutex);
    bgImage.reset(new Image(imgSrc));

    ++imageChanges;
  }
  ```
  还有一种叫做copy and swap的策略可以很简单的实现强烈保证，即把要修改的对象做出一个副本，在
  副本上修改，只有副本改变成功后。再将这个副本和原对象置换即可。如下
  ```
  struct PMImpl{
    std::tr1::shared_ptr<Image> bgImage;
    int imageChanges;
  };
  class Prettymenu{
    ...
    private:
      Mutex mutex;
      std::tr1::shared_ptr<PMImpl> pImpl;
    ...
  };

  void Prettymenu::changeBackground(std::istream& imgSrc){
    using std::swap;
    Lock ml(&mutex);
    std::tr1::shared_ptr<PMImpl> pNew(new PMImpl(*pImpl));
    pNew->bgImage.reset(new Image(imgSrc)); //修改副本
    ++pNew->imageChanges;
    swap(pImpl,pNew);
  }
  ```
+ 对于许多函数，异常安全性之基本保证是一个最好的选择。如果系统中有一个函数不具备异常安全性，
  整个系统就不具备异常安全性，因为调用这个函数可能导致资源泄露或者数据结构败坏。
+ 在写代码的时候，要思考如何具备异常安全性，首先以对象管理资源，可以阻止资源泄露。然后挑选
  三个异常安全保证中的一个实施于你所写的每一个函数身上。
+ 所以
  1. 异常安全函数(Exception-safe functions)即使发生异常也不会泄漏资源或允许任何数据结构败坏
  。这样的函数区分为三种可能的保证:基本型、强烈型、不抛异常型。
  2. "强烈保证"往往能够以copy-and-swap实现出来，但"强烈保证"并非对所有函数都可实现或具备现
  实意义。
  3. 函数提供的"异常安全保证"通常最高只等于其所调用之各个函数的"异常安全保证"中的最弱者。
## item30 Understand the ins and outs of inlining
+ 透彻了解inlining的里里外外。
+ inline函数通常一定被放置于头文件中。
+ 所以
  1. 将大多数inlining限制在小型、被频繁调用的函数身上。这可使日后的调试过程和二进制升级(binary
  upgradability)
  2. 不要只因为function templates出现在头文件，就将它们声明为inline。
## item31 Minimize compilation dependencies between files
+ 将文件间的编译依存关系降至最低。
+ 为声明式和定义式提供不同的头文件，所以程序库客户应该总是#include一个声明文件而非前置声明
  若干函数。
  ```
  #include "datefwd.h"//这个头文件内声明但是没有定义class Date。
  Date today();
  void clearAppointments(Date d);
  ```
+ Handle classes和Interface classes解除了接口和实现之间的藕合关系，从而降低文件间的编译依
  存性(compilation dependencies)。
+ 所以
  1. 持"编译依存性最小化"的一般构想是:相依于声明式，不要相依于定义式。基于此构想的两个手段是
  Handle classes和Interface classes。
  2. 程序库头文件应该以"完全且仅有声明式"(full and declaration-only forms)的形式存在。这种
  做法不论是否涉及templates都适用。


# 继承与面向对象设计
+ 继承可以是单一继承也可以是多重继承。
+ 每一个继承连接可是public，protected，private，也可以是virtual，non-virtual。
+ 成员函数的各个选项，virtual，non-virtual，pure-virtual。
+ 成员函数和其他语言特性的交互影响，缺省参数值与virtual函数有什么交互影响。
+ 继承如何影响C++的名称查找规则，设计选项有哪些，如果class的行为需要修改，virtual函数是
  最佳选择么。
## item32 Make sure public inheritance models "is-a"
+ 确定你的public继承塑模出is-a关系。
+ 以C++进行面向对象编程，最重要的一个规则是public inheritance意味着is-a(是一种)的关系。
+ 对于public继承，如果一个函数愿意接受一个base的实参，那么他也愿意接受一个派生类对象。对于
  private继承则不同。
+ 所以
  1. public继承意味着is-a，适用于base classes身上的每一件事情也适用于派生类身上，因为每一个
  派生类对象也是一个base classes对象。
## item33 Avoiding hiding inherited names
+ 避免遮掩继承而来的名称。
+ 如果你继承base class并且加上重载函数，而你又希望重新定义或者覆写其中一部分，那么你必须为
  元贝会被遮掩得每个名称引入一个using声明式，否在某些你希望继承得名称会被遮掩。
+ 在public继承下，派生类必须继承base class得所有函数，因为public继承隐含了base和derived
  之间得is-a关系，而在private继承中，derived只想继承函数得其中一个版本。如下
  ```
  class Base{
    public:
       virtual void mf1()=0;
       virtual void mf1(int);
       ...
  };

  class Derived:private Base{
    public:
       virtual void mf1()//这是一个转交函数(forwarding function)。         
       {
         Base::mf1();
       }
       ...
  };
  ...
  Derived d;
  int x;
  d.mf1();//调用的是Derived::mf1
  d.mf1(x);//失败，Base::mf1被遮掩了
  ```
+ 所以
  1. derived classes内的名称会遮掩base classes内的名称。在public继承下从来没有人希望如此。
  2. 为了让被遮掩的名称再见天日，可使用using声明式或转变函数(forwarding functions)。
## item34 Differentiate between inheritance of interface and inheritance of implementation
+ 区分接口继承和实现继承。
+ public接口继承相当于继承声明，如果派生类继承实现，那么就要能够override它们所继承的实现。也
  有一种情况就是继承函数的接口和实现，并且不允许override任何东西。
  ```
  class Shape{
    public:
       virtual void draw() const = 0;
       virtual void error(const std::string& msg);
       int objectID() const;
       ...
  };

  class Rectangle:public Shape{...};
  class Ellipse: public Shape{...};
  ```
  Shape是个抽象class;它的pure virtual函数draw使它成为一个抽象class。所以用户不能创建Shape
  class的实体，只能创建它派生类的实体。
  上面的继承，成员函数的接口总是会被继承。因为public继承的特性。上面的shape有三种函数声明方式。
  不同的声明带来什么结果呢？
  1. pure virtual函数必须被任何继承了它们的具象class重新声明，并且它们在抽象class中通常没有
  定义。所以，声明一个pure virtual函数的目的是为了让derived classes只继承函数接口。Shape::draw
  的声明式乃是对具象derived classes设计者说，"你必须提供一个draw函数，但我不干涉你怎么实现它。"
  当然，pure virtual函数也可以提供定义。
  2. 而impure virtual函数是为了让派生类继承该函数的接口和缺省实现。也就是说Shape::error的声
  明式告诉derived classes的设计者，"你必须支持一个error函数，但如果你不想自己写一个，可以使用
  Shape class提供的缺省版本"。想象以下的继承体系。
  有一个基类定义了一个impure virtual函数，然后有两个派生类以public继承它，那么A,B类中这个
  virtual函数的行为一样，那么我们就不需要重新实现这个impure virtual函数了。但是如果，现在
  多了一个类C他也public继承基类，而他的virtual函数的行为和基类中定义的不一样。或许这里我们
  可以使用上面的pure virtual，定义一个pure virtual函数
  ```
  class Airplane {
    public:
    virtual void fly(const Airport& destination} = 0;
    ...
  };
  void Airplane::fly(const Airport& destination)//pure virtual函数实现
  {
    ...
  }
  class ModelA: public Airplane {
    public:
    virtual void fly(const Airport& destination)
    { Airplane::fly(destination); }
    ...
  };
  class ModelB: public Airplane {
    public:
    virtual void fly(const Airport& destination)
    { Airplane::fly(destination); }
    ...
  };

  class ModelC: public Airplane {
    public:
    virtual void fly(const Airport& destination);
    ...
  };

  void ModelC::fly(const Airport& destination)
  {
    //将c飞机飞到目的地
  }
  ```
  3. 最后是Shape中的non-virtual函数，如果基类中成员函数是个non-virtual函数，意味着它不打算
  在派生类中有不同的行为。声明non-virtual函数的目的是为了令derived classes继承函数的接口及
  一份强制性实现。由于non-virtual函数代表的意义是不变性Cinvariant)凌驾特异性(specialization)
  ,所以它绝不该在derived class中被重新定义。
  上面的三种pure virtual函数，impure virtual函数，non-virtual函数让我们可以精确的指定你想要
  派生类继承的东西：只继承接口，继承接口和一份缺省实现，或是继承接口和一份强制实现。
+ 通常经验不足的class设计者经常犯两个错误。
  1. 第一个是将所有函数声明为non-virtual。这使得派生类中没有空间进行特化工作，non-virtual析构
  函数尤其会带来问题。
  2. 另一个错误就是将所有成员函数声明为virtual。当然如果你想设计一个interface classes这是可行的
  。
+ 所以
  1. 接口继承和实现继承不同。在public继承之下，derived classes总是继承base class的接口。
  2. pure virtual函数只具体指定接口继承。
  3. impure virtual函数具体指定接口继承及缺省实现继承。
  4. non-virtual函数具体指定接口继承以及强制性实现继承。
## item35 Consider alternatives to virtual functions
+ 考虑virtual函数以外的其他选择。
+ 通过non-virtual interface手法实现template method模式。这个流派主张virtual函数应该是
  private的。将healthvalue设计为public成员函数，并且让他成为non-virtual,并且调用一个
  private virtual函数(doHealthValue)进行实际工作。
  ```
  class GameCharacter{
    public:
       int healthValue() const{
         ...
         int retVal = doHealthValue();
         ...
         return retVal;
       }
       ...
    private:
       virtual int doHealthValue() const{//派生类可以重新定义它
         ...
       }
  };
  ```
  这一手法令用户通过public non-virtual成员函数间接调用private virtual函数，称为non-virtual
  interface(NVI)手法。把这个non-virtual函数称为virtual函数的wrapper。
+ 第二种替代virtual函数的是通过function pointers实现strategy模式。即我们可以要求每个人物的
  构造函数接受一个指针，指向一个健康计算函数，而我们可以调用该函数进行实际计算。
  ```
  class GameCharacter;//前置声明
  int defaultHealthValueCalc(const GameCharacter& gc);
  class GameCharacter{
    public:
       typedef int (*HealthCalcFunc) (const GameCharacter&);
       explicit GameCharacter(HealthCalcFunc hcf = defaultHealthValueCalc) : healthFunc(hcf)
       {}
       int healthValue() consts{
         return healthFunc(*this);
       }
       ...
    private:
       HealthCalcFunc healthFunc;
  };
  ```
  运用函数指针替换virtual函数，其优点(像是"每个对象可各自拥有自己的健康计算函数"和"可在运行
  期改变计算函数" )是否足以弥补缺点(例如可能必须降低GameCharacter封装性) ，是你必须根据每个
  设计情况的不同而抉择的。因为非成员函数想访问class的non-public成员必须减弱class的封装。
+ 第三种是藉由tr1::function完成strategy模式，这里我们不再使用函数指针，而是改用一个类型为
  tr1::function的对象。
  ```
  class GameCharacter;//前置声明
  int defaultHealthValueCalc(const GameCharacter& gc);

  class GameCharacter{
    public:
       //HealthCalcFunc可以是任何"可调用物" (callable entity) ，可被调用并接受
       //任何兼容于GameCharacter之物,返回任何兼容于int的东西。详下。
       typedef std::tr1::function<int (const GameCharacter&)> HealthCalcFunc;
       explicit GameCharacter(HealthCalcFunc hcf = defaultHealthValueCalc) : healthFunc(hcf)
       {}
       int healthValue() consts{
         return healthFunc(*this);
       }
       ...
    private:
       HealthCalcFunc healthFunc;
  };
  ```
  所谓兼容，意思是这个可调用物的参数可被隐式转换为const GameCharacter&，而其返回类型可被隐式
  转换为int。
  ```
  short calcHealth(const GameCharacter&);//健康计算函数

  struct HealthCalcculator{//为计算健康而设计的函数对象
    int operator() (const GameCharacter&) const{
      ...
    }
  };

  class GameLevel{
    public:
       float health(const GameCharacter&) const;//成员函数，用以计算健康
       ...
  };

  class EvilBadGuy: public GameCharacter{
    ...
  };

  class EyeCandyCharacter: public GameCharacter{
    ...
  };

  EvilBadGuy ebg1(calcHealth);//人物1，使用某个函数计算健康

  EyeCandyCharacter ecc1(HealthCalcculator());//人物2，使用某个函数对象计算健康

  GameLevel currentLevel;
  ...
  EvilBadGuy ebg2(std::tr1::bind(&GameLevel::health,currentLevel,_1));//人物3 使用某个
  //成员函数计算健康
  ```
  tr1::bind的作为:它指出ebg2的健康计算函数应该总是以currentLevel作为Gamelevel对象。
+ 还有一种古典的strategy模式。
  这图只是告诉你GameCharacter是某个继承体系的根类，体系中的EvilBadGuy和EyeCandyCharacter
  都是derived classes;HealthCalcFunc是另一个继承体系的根类，体系中的SlowHealthLoser 和
  FastHealthLoser都是derived classes，每一个GameCharacter对象都内含一个指针，指向一个来
  自HealthCalcFunc继承体系的对象。
  ```
  class GameCharacter;
  class HealthCalcFunc{
    public:
       ...
       virtual int calc(const GameCharacter& gc) const{...}
       ...
  };

  HealthCalcFunc defaultHealthValueCalc;

  class GameCharacter{
    public:
       explicit GameCharacter(HealthCalcFunc* phcf = &defaultHealthCalc):
       pHealthCalc(phcf){}
       int healthValue() consts{
         return pHealthCalc->calc(*this);
       }
       ...
    private:
       HealthCalcFunc* pHealthCalc;
  };
  ```
+ 所以，本条框的根本忠告在于，当你为解决问题而寻找某个设计方法时，不姑考虑virtual函数的替代方案。
  1. 使用non-virtual interface(NYI)手法，那是Template Method设计模式的一种特殊形式。它以
  public non-virtual成员函数包裹较低访问性(private或protected)的virtual函数。
  2. 将virtual函数替换为"函数指针成员变量"，这是Strategy设计模式的一种分解表现形式。
  3. 以trl::function成员变量替换virtual函数，因而允许使用任何可调用物(callable entity)搭配
  一个兼容于需求的签名式。这也是Strategy设计模式的某种形式。
  4. 将继承体系内的virtual函数替换为另一个继承体系内的virtual函数。这是Strategy设计模式的
  传统实现手法。
+ 所以
  1. virtual函数的替代方案包括NYI手法及Strategy设计模式的多种形式。NYI手法自身是一个特殊形
  式的Template Method设计模式。
  2. 将机能从成员函数移到class外部函数，带来的一个缺点是，非成员函数无法访问class的non-public
  成员。
  3. trl::function对象的行为就像一般函数指针。这样的对象可接纳"与给定之目标签名式(target
  signature)兼容"的所有可调用物(callable entities)。
