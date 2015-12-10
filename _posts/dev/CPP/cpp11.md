title: 开发者都应该使用的10个C++11特性
date: 2015-10-26 20:19:56
tags:
category: c++
---

### auto
---

在C++11之前，auto关键字用来指定存储期。在新标准中，它的功能变为类型推断。auto现在成了一个类型的占位符，通知编译器去根据初始化代码推断所声明变量的真实类型。各种作用域内声明变量都可以用到它。例如，名空间中，程序块中，或是for循环的初始化语句中。
```c++
auto i = 42;        // i is an int
auto l = 42LL;      // l is an long long
auto p = new foo(); // p is a foo*
```

使用auto通常意味着更短的代码（除非你所用类型是int，它会比auto少一个字母）。试想一下当你遍历STL容器时需要声明的那些迭代器（iterator）。现在不需要去声明那些typedef就可以得到简洁的代码了。

```c++
std::map<std::string, std::vector<int>> map;
for(auto it = begin(map); it != end(map); ++it) {}
```

需要注意的是，auto不能用来声明函数的返回值。但如果函数有一个尾随的返回类型时，auto是可以出现在函数声明中返回值位置。这种情况下，auto并不是告诉编译器去推断返回类型，而是指引编译器去函数的末端寻找返回值类型。在下面这个例子中，函数的返回值类型就是operator+操作符作用在T1、T2类型变量上的返回值类型。
```c++
template <typename T1, typename T2>
auto compose(T1 t1, T2 t2) -> decltype(t1 + t2)
{
   return t1+t2;
}
auto v = compose(2, 3.14); // v's type is double
```

### nullptr
---

以前都是用0来表示空指针的，但由于0可以被隐式类型转换为整形，这就会存在一些问题。关键字nullptr是std::nullptr_t类型的值，用来指代空指针。nullptr和任何指针类型以及类成员指针类型的空值之间可以发生隐式类型转换，同样也可以隐式转换为bool型（取值为false）。但是不存在到整形的隐式类型转换。
```c++
void foo(int* p) {}
void bar(std::shared_ptr<int> p) {}
int* p1 = NULL;
int* p2 = nullptr;   
if(p1 == p2) {}
foo(nullptr);
bar(nullptr);
bool f = nullptr;
int i = nullptr; // error: A native nullptr can only be converted to bool or, using reinterpret_cast, to an integral type
```
为了向前兼容，0仍然是个合法的空指针值。

### Range-based for loops （基于范围的for循环）
---

为了在遍历容器时支持”foreach”用法，C++11扩展了for语句的语法。用这个新的写法，可以遍历C类型的数组、初始化列表以及任何重载了非成员的begin()和end()函数的类型。如果你只是想对集合或数组的每个元素做一些操作，而不关心下标、迭代器位置或者元素个数，那么这种foreach的for循环将会非常有用。

```c++
std::map<std::string, std::vector<int>> map;
std::vector<int> v;
v.push_back(1);
v.push_back(2);
v.push_back(3);
map["one"] = v;
for(const auto& kvp : map) 
{
  std::cout << kvp.first << std::endl;
  for(auto v : kvp.second)
  {
     std::cout << v << std::endl;
  }
}
int arr[] = {1,2,3,4,5};
for(int& e : arr) 
{
  e = e*e;
}
```

### override和final
---

我总觉得 C++中虚函数的设计很差劲，因为时至今日仍然没有一个强制的机制来标识虚函数会在派生类里被改写。vitual关键字是可选的，这使得阅读代码变得很费劲。因为可能需要追溯到继承体系的源头才能确定某个方法是否是虚函数。为了增加可读性，我总是在派生类里也写上virtual关键字，并且也鼓励大家都这么做。即使这样，仍然会产生一些微妙的错误。看下面这个例子：

```c++
class A 
{
public:
   virtual void f(short) {std::cout << "A::f" << std::endl;}
};
class B : public A
{
public:
   virtual void f(int) {std::cout << "B::f" << std::endl;}
};
```

B::f 按理应当重写 A::f。然而二者的声明是不同的，一个参数是short，另一个是int。因此B::f只是拥有同样名字的另一个函数（重载）而不是重写。当你通过A类型的指针调用f()可能会期望打印出B::f，但实际上则会打出 f(int)而不是f(short) 。另一个很微妙的错误情况：参数相同，但是基类的函数是const的，派生类的函数却不是。

```c++
class A 
{
public:
   virtual void f(int) const {std::cout << "A::f " << std::endl;}
};
class B : public A
{
public:
   virtual void f(int) {std::cout << "B::f" << std::endl;}
};
```

同样，这两个函数是重载而不是重写。幸运的是，现在有一种方式能描述你的意图。新标准加入了两个新的标识符（不是关键字）


override，表示函数必须重写基类中的虚函数，如果派生类没有重写到将编译报错。

final，表示派生类不应当重写这个虚函数，如果派生类重写了基类的虚函数将编译报错。

```c++
class A
{
public:
    virtual void f(short) { std::cout << "A::f" << std::endl; }
    virtual void g(int) final { std::cout << "A::g" << std::endl; }
};
class B : public A
{
public:
    virtual void f(short)  override  { std::cout << "B::f" << std::endl; }
    //virtual void g(int) { std::cout << "A::g" << std::endl; } // error C3248: “main::A::g”:  声明为“final”的函数无法被“main::B::g”重写
    virtual void g(float) { std::cout << "A::g" << std::endl; } // 重载
};
```


### Strongly-typed enums 强类型枚举
---

传统的C++枚举类型存在一些缺陷：它们会将枚举常量暴露在外层作用域中（这可能导致名字冲突，如果同一个作用域中存在两个不同的枚举类型，但是具有相同的枚举常量就会冲突），而且它们会被隐式转换为整形，无法拥有特定的用户定义类型。

在C++11中通过引入了一个称为强类型枚举的新类型，修正了这种情况。强类型枚举由关键字enum class标识。它不会将枚举常量暴露到外层作用域中，也不会隐式转换为整形，并且拥有用户指定的特定类型（传统枚举也增加了这个性质）。

```c++
enum class Options {None, One, All};
Options o = Options::All;
```

### Smart Pointers 智能指针
---

已经有成千上万的文章讨论这个问题了，所以我只想说：现在能使用的，带引用计数，并且能自动释放内存的智能指针包括以下几种：

unique_ptr: 如果内存资源的所有权不需要共享，就应当使用这个（它没有拷贝构造函数），但是它可以转让给另一个unique_ptr（存在move构造函数）。

shared_ptr:  如果内存资源需要共享，那么使用这个（所以叫这个名字）。

weak_ptr: 持有被shared_ptr所管理对象的引用，但是不会改变引用计数值。它被用来打破依赖循环（想象在一个tree结构中，父节点通过一个共享所有权的引用(chared_ptr)引用子节点，同时子节点又必须持有父节点的引用。如果这第二个引用也共享所有权，就会导致一个循环，最终两个节点内存都无法释放）。

另一方面，auto_ptr已经被废弃，不会再使用了。

什么时候使用unique_ptr，什么时候使用shared_ptr取决于对所有权的需求，我建议阅读以下的讨论：http://stackoverflow.com/questions/15648844/using-smart-pointers-for-class-members

```c++
std::unique_ptr<int> p1(new int(42));
std::unique_ptr<int> p2 = std::move(p1); // 移交unique指针
auto sp = std::make_shared<int>(42);
std::weak_ptr<int> wp = sp;
{
    auto sp = wp.lock(); // 提升shared_ptr
    std::cout << *sp << std::endl;
}
sp.reset();
 
if (wp.expired())
    std::cout << "expired" << std::endl;
```

如果你试图锁定(lock)一个过期（指被弱引用对象已经被释放）的weak_ptr，那你将获得一个空的shared_ptr.

### Lambdas
---

匿名函数（也叫lambda）已经加入到C++中，并很快异军突起。这个从函数式编程中借来的强大特性，使很多其他特性以及类库得以实现。你可以在任何使用函数对象或者函子(functor)或std::function的地方使用lambda。你可以从这里（http://msdn.microsoft.com/en-us/library/dd293603.aspx）找到语法说明。

```c++
std::vector<int> v;
v.push_back(1);
v.push_back(2);
v.push_back(3);
std::for_each(std::begin(v), std::end(v), [](int n) {std::cout << n << std::endl;});
auto is_odd = [](int n) {return n%2==1;};
auto pos = std::find_if(std::begin(v), std::end(v), is_odd);
if(pos != std::end(v))
    std::cout << *pos << std::endl;
```

更复杂的是递归lambda。考虑一个实现Fibonacci函数的lambda。如果你试图用auto来声明，就会得到一个编译错误。

```c++
auto fib = [&fib](int n) {return n < 2 ? 1 : fib(n-1) + fib(n-2);};
```

error C3533: 'auto &': a parameter cannot have a type that contains 'auto'
error C3531: 'fib': a symbol whose type contains 'auto' must have an initializer
error C3536: 'fib': cannot be used before it is initialized
error C2064: term does not evaluate to a function taking 1 arguments
问题出在auto意味着对象类型由初始表达式决定，然而初始表达式又包含了对其自身的引用，因此要求先知道它的类型，这就导致了无穷递归。解决问题的关键就是打破这种循环依赖，用std::function显式的指定函数类型：

```c++
std::function<int(int)> lfib = [&lfib](int n) {return n < 2 ? 1 : lfib(n-1) + lfib(n-2);};
```

### 非成员begin()和end()
---

也许你注意到了，我在前面的例子中已经用到了非成员begin()和end()函数。他们是新加入标准库的，除了能提高了代码一致性，还有助于更多地使用泛型编程。它们和所有的STL容器兼容。更重要的是，他们是可重载的。所以它们可以被扩展到支持任何类型。对C类型数组的重载已经包含在标准库中了。我们还用上一个例子中的代码来说明，在这个例子中我打印了一个数组然后查找它的第一个偶数元素。如果std::vector被替换成C类型数组。代码可能看起来是这样的：

```c++
int arr[] = {1,2,3};
std::for_each(&arr[0], &arr[0]+sizeof(arr)/sizeof(arr[0]), [](int n) {std::cout << n << std::endl;});
auto is_odd = [](int n) {return n%2==1;};
auto begin = &arr[0];
auto end = &arr[0]+sizeof(arr)/sizeof(arr[0]);
auto pos = std::find_if(begin, end, is_odd);
if(pos != end)
    std::cout << *pos << std::endl;
```

如果使用非成员的begin()和end()来实现，就会是以下这样的：

```c++
int arr[] = {1,2,3};
std::for_each(std::begin(arr), std::end(arr), [](int n) {std::cout << n << std::endl;});
auto is_odd = [](int n) {return n%2==1;};
auto pos = std::find_if(std::begin(arr), std::end(arr), is_odd);
if(pos != std::end(arr))
    std::cout << *pos << std::endl;
```

这基本上和使用std::vecto的代码是完全一样的。这就意味着我们可以写一个泛型函数处理所有支持begin()和end()的类型。

```c++
template <typename Iterator>
void bar(Iterator begin, Iterator end)
{
    std::for_each(begin, end, [](int n) {std::cout << n << std::endl;});
    auto is_odd = [](int n) {return n%2==1;};
    auto pos = std::find_if(begin, end, is_odd);
    if(pos != end)
        std::cout << *pos << std::endl;
}
template <typename C>
void foo(C c)
{
    bar(std::begin(c), std::end(c));
}
template <typename T, size_t N>
void foo(T(&arr)[N])
{
    bar(std::begin(arr), std::end(arr));
}
int arr[] = {1,2,3};
foo(arr);
std::vector<int> v;
v.push_back(1);
v.push_back(2);
v.push_back(3);
foo(v);
```

### static_assert和 type traits
---

static_assert提供一个编译时的断言检查。如果断言为真，什么也不会发生。如果断言为假，编译器会打印一个特殊的错误信息。

```c++
template <typename T, size_t Size>
class Vector
{
   static_assert(Size < 3, "Size is too small");
   T _points[Size];
};
int main()
{
   Vector<int, 16> a1;
   Vector<double, 2> a2;
   return 0;
}
```

error C2338: Size is too small
see reference to class template instantiation 'Vector<T,Size>' being compiled
   with
   [
      T=double,
      Size=2
   ]
static_assert和type traits一起使用能发挥更大的威力。type traits是一些class，在编译时提供关于类型的信息。在头文件<type_traits>中可以找到它们。这个头文件中有好几种class: helper class，用来产生编译时常量。type traits class，用来在编译时获取类型信息，还有就是type transformation class，他们可以将已存在的类型变换为新的类型。

下面这段代码原本期望只做用于整数类型。

```c++
template <typename T1, typename T2>
auto add(T1 t1, T2 t2)
{
    return t1 + t2;
}
```

但是如果有人写出如下代码，编译器并不会报错

```c++
std::cout << add(1, 3.14) << std::endl;
std::cout << add("one", 2) << std::endl;
```

程序会打印出4.14和”e”。但是如果我们加上编译时断言，那么以上两行将产生编译错误。

```c++
template <typename T1, typename T2>
auto add(T1 t1, T2 t2)
{
   static_assert(std::is_integral<T1>::value, "Type T1 must be integral");
   static_assert(std::is_integral<T2>::value, "Type T2 must be integral");
   return t1 + t2;
}
```

error C2338: Type T2 must be integral
see reference to function template instantiation 'T2 add<int,double>(T1,T2)' being compiled
   with
   [
      T2=double,
      T1=int
   ]
error C2338: Type T1 must be integral
see reference to function template instantiation 'T1 add<const char*,int>(T1,T2)' being compiled
   with
   [
      T1=const char *,
      T2=int
   ]

### Move semantics （Move语义）
---

这是C++11中所涵盖的另一个重要话题。就这个话题可以写出一系列文章，仅用一个段落来说明显然是不够的。因此在这里我不会过多的深入细节，如果你还不是很熟悉这个话题，我鼓励你去阅读更多地资料。C++11加入了右值引用(value reference)的概念（用&&标识），用来区分对左值和右值的引用。左值就是一个有名字的对象，而右值则是一个无名对象（临时对象）。move语义允许修改右值（以前右值被看作是不可修改的，等同于const T&类型）。C++的class或者struct以前都有一些隐含的成员函数：默认构造函数（仅当没有显示定义任何其他构造函数时才存在），拷贝构造函数，析构函数还有拷贝赋值操作符。拷贝构造函数和拷贝赋值操作符提供bit-wise的拷贝（浅拷贝），也就是逐个bit拷贝对象。也就是说，如果你有一个类包含指向其他对象的指针，拷贝时只会拷贝指针的值而不会管指向的对象。在某些情况下这种做法是没问题的，但在很多情况下，实际上你需要的是深拷贝，也就是说你希望拷贝指针所指向的对象。而不是拷贝指针的值。这种情况下，你需要显示地提供拷贝构造函数与拷贝赋值操作符来进行深拷贝。如果你用来初始化或拷贝的源对象是个右值（临时对象）会怎么样呢？你仍然需要拷贝它的值，但随后很快右值就会被释放。这意味着产生了额外的操作开销，包括原本并不需要的空间分配以及内存拷贝。现在说说move constructor和move assignment operator。这两个函数接收T&&类型的参数，也就是一个右值。在这种情况下，它们可以修改右值对象，例如“偷走”它们内部指针所指向的对象。举个例子，一个容器的实现（例如vector或者queue）可能包含一个指向元素数组的指针。当用一个临时对象初始化一个对象时，我们不需要分配另一个数组，从临时对象中把值复制过来，然后在临时对象析构时释放它的内存。我们只需要将指向数组内存的指针值复制过来，由此节约了一次内存分配，一次元数组的复制以及后来的内存释放。以下代码实现了一个简易的buffer。这个buffer有一个成员记录buffer名称（为了便于以下的说明），一个指针（封装在unique_ptr中）指向元素为T类型的数组，还有一个记录数组长度的变量。

```c++
template <typename T>
class Buffer 
{
   std::string          _name;
   size_t               _size;
   std::unique_ptr<T[]> _buffer;
  
public:
   // default constructor
   Buffer():
      _size(16),
      _buffer(new T[16])
   {}
  
   // constructor
   Buffer(const std::string& name, size_t size):
      _name(name),
      _size(size),
      _buffer(new T[size])
   {}
  
   // copy constructor
   Buffer(const Buffer& copy):
      _name(copy._name),
      _size(copy._size),
      _buffer(new T[copy._size])
   {
      T* source = copy._buffer.get();
      T* dest = _buffer.get();
      std::copy(source, source + copy._size, dest);
   }
  
   // copy assignment operator
   Buffer& operator=(const Buffer& copy)
   {
      if(this != ©)
      {
         _name = copy._name;
  
         if(_size != copy._size)
         {
            _buffer = nullptr;
            _size = copy._size;
            _buffer = _size > 0 > new T[_size] : nullptr;
         }
  
         T* source = copy._buffer.get();
         T* dest = _buffer.get();
         std::copy(source, source + copy._size, dest);
      }
  
      return *this;
   }
  
   // move constructor
   Buffer(Buffer&& temp):
      _name(std::move(temp._name)),
      _size(temp._size),
      _buffer(std::move(temp._buffer))
   {
      temp._buffer = nullptr;
      temp._size = 0;
   }
  
   // move assignment operator
   Buffer& operator=(Buffer&& temp)
   {
      assert(this != &temp); // assert if this is not a temporary
  
      _buffer = nullptr;
      _size = temp._size;
      _buffer = std::move(temp._buffer);
  
      _name = std::move(temp._name);
  
      temp._buffer = nullptr;
      temp._size = 0;
  
      return *this;
   }
};
  
template <typename T>
Buffer<T> getBuffer(const std::string& name) 
{
   Buffer<T> b(name, 128);
   return b;
}
int main()
{
   Buffer<int> b1;
   Buffer<int> b2("buf2", 64);
   Buffer<int> b3 = b2;
   Buffer<int> b4 = getBuffer<int>("buf4");
   b1 = getBuffer<int>("buf5");
   return 0;
}
```

默认的copy constructor以及copy assignment operator大家应该很熟悉了。C++11中新增的是move constructor以及move assignment operator，这两个函数根据上文所描述的move语义实现。如果你运行这段代码，你就会发现b4构造时，move constructor会被调用。同样，对b1赋值时，move assignment operator会被调用。原因就在于getBuffer()的返回值是一个临时对象——也就是右值。你也许注意到了，move constuctor中当我们初始化变量name和指向buffer的指针时，我们使用了std::move。name实际上是一个string，std::string实现了move语义。std::unique_ptr也一样。但是如果我们写_name(temp._name)，那么copy constructor将会被调用。不过对于_buffer来说不能这么写，因为std::unique_ptr没有copy constructor。但为什么std::string的move constructor此时没有被调到呢？这是因为虽然我们使用一个右值调用了Buffer的move constructor，但在这个构造函数内，它实际上是个左值。为什么？因为它是有名字的——“temp”。一个有名字的对象就是左值。为了再把它变为右值（以便调用move constructor)必须使用std::move。这个函数仅仅是把一个左值引用变为一个右值引用。更新：虽然这个例子是为了说明如何实现move constructor以及move assignment operator，但具体的实现方式并不是唯一的。某同学提供了另一种可能的实现。为了方便查看，我把它也列在下面：

```c++
template <typename T>
class Buffer
{
   std::string          _name;
   size_t               _size;
   std::unique_ptr<T[]> _buffer;
  
public:
   // constructor
   Buffer(const std::string& name = "", size_t size = 16):
      _name(name),
      _size(size),
      _buffer(size? new T[size] : nullptr)
   {}
  
   // copy constructor
   Buffer(const Buffer& copy):
      _name(copy._name),
      _size(copy._size),
      _buffer(copy._size? new T[copy._size] : nullptr)
   {
      T* source = copy._buffer.get();
      T* dest = _buffer.get();
      std::copy(source, source + copy._size, dest);
   }

   // copy assignment operator
   Buffer& operator=(Buffer copy)
   {
       swap(*this, copy);
       return *this;
   }
  
   // move constructor
   Buffer(Buffer&& temp):Buffer()
   {
      swap(*this, temp);
   }
  
   friend void swap(Buffer& first, Buffer& second) noexcept
   {
       using std::swap;
       swap(first._name  , second._name);
       swap(first._size  , second._size);
       swap(first._buffer, second._buffer);
   }
};
```
