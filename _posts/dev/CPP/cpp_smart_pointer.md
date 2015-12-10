date: 2015-10-27 20:47:00
title: C++智能指针(Smart Pointer)
tags: 
category: dev

----------

### 1. 传统指针存在的问题
---

传统指针存在诸多的问题，比如指针所指向的对象的生命周期问题，挂起引用(dangling references)，以及内存泄露(memory leaks). 如下是一个传统指针的使用过程
```c++
void Foo() {
    int *iPtr = new int[5];
    // manipulate the memory block
    // ...
    // ...
    // ...
    delete[] iPtr;
}
```
以上代码将正常运行且内存将被合理释放，但是使用指针常会发生一些意想不到的事情，比如访问一个非法的内存单元，除0操作，以及根据一些判断条件处理的返回return 语句。

### 2. 什么是智能指针(Smart Pointer)
---

智能指针是RAII（Resource Acquisition is initialization）用来动态的分配内存。它提供了普通指针的所有接口外加少数异常处理。在构造阶段，它将分配内存，而在非其作用域内将自动释放所占有的内存。 

在C++98中，使用 auto_ptr来解决上述问题。

#### 2.1 auto_ptr

```c++
#include <iostream>
#include <memory>
using namespace std;

class Test {
public:
    Test(int a = 0) : m_a(a) {}
    ~Test() {
        cout << "Calling destructor" << endl;
    }

public:
    int m_a;
};

int main()
{
    auto_ptr<Test> p(new Test(5));
    cout << p->m_a << endl;
    return 0;
}
```

输出结果： 

![](/images/cpp/sp1.jpg)

上述代码将智能的释放相关的内存。

```c++
#include <iostream>
#include <memory>

using namespace std;

class Test {
public:
    Test(int a = 0) : m_a(a) {}
    ~Test() {
        cout << "Calling destructor" << endl;
    }

public:
    int m_a;
};

void Fun()
{
    int a = 0, b = 5, c;
    if(a == 0) {
        throw "Invalid divisor";
    }
    c = b / a;
    return;
}

int main()
{
    try {
        auto_ptr<Test> p(new Test(5));
        Fun();
        cout << p->m_a << endl;
    }
    catch(...) {
        cout << "Something has gone wrong" << endl;
    }

    return 0;
}
```

在上述代码中，即使异常抛出，指针照样被正常释放。这是因为当异常抛出时，栈松绑(stack unwinding)。 当所有属于try block的本地对象被销毁时，指针p不在作用域中而被释放。

##### 问题1.

至少从目前来说auto_ptr是智能的。但是它有一个非常本质的缺点：auto_ptr会传递它本身的ownership当其被赋值给另一个auto_ptr对象。正如下述程序所示，一个auto_ptr对象传递给函数Fun()中的auto_ptr对象时，其ownership,或者说是smartness将不再返回给原auto_ptr所指向的p。

```c++
#include <iostream>
#include <memory>

using namespace std;

class Test {
public:
    Test(int a = 0) : m_a(a) {}
    ~Test() {
        cout << "Calling destructor" << endl;
    }

public:
    int m_a;
};

void Fun(auto_ptr<Test> p1)
{
    cout << p1->m_a << endl;
    cout << "Fun() end" << endl;
}

int main()
{
    auto_ptr<Test> p(new Test(5));
    Fun(p);
    cout << p->m_a << endl;

    return 0;
}
```

运行结果： 

![](/images/cpp/sp2.jpg)
 
以上程序将crash因为存在野指针auto_ptr。上述代码奔溃的主要原因是，原先的p占有对其自身分配内存的管理权。然而通过Fun()函数将其管理权转给p1，此时因为p1的smartness，在其结束时将释放其所指向的内存块。而此时原先的p将再拥有任何内存，而导致在访问时出现了空指针，野指针的引用问题。

##### 问题2.

auto_ptr不能使用于数组对象。 这里的意思是不能使用于操作符new[]。
```c++
int main()
{
    auto_ptr<Test> p(new Test[5]);
    return 0;
}
```
上述代码将runtime error。因为对于auto_ptr而言，其调用的是delete而非delete []。

##### 问题3.

auto_ptr不能使用于一些标准的容器库。比如vector，list，map等等。

C++11提出了新型的智能指针，并且都赋予了其相应的意图。

#### 2.2 shared_ptr

creation :
shared_ptr设计的目的很简单：多个共享指针可以指向同一个对象，而当最后一个共享指针在作用域范围内结束时，内存才会被自动的释放。

```c++
int main()
{
    // share_ptr 常规的创建过程
    shared_ptr<int> sptr1( new int );
    // 使用make_shared 来加速创建过程
    // shared_ptr 自动分配内存，并且保证引用计数
    // 而make_shared则是按照这种方法来初始化
    shared_ptr<int> sptr2 = make_shared<int>( 100 );
    // 可以通过use_count() 来查看引用计数
    cout << "sptr2 referenced count: " << sptr2.use_count() << endl;
    shared_ptr<int> sptr3 = sptr2;
    cout << "sptr2 referenced count: " << sptr2.use_count() << endl;
    cout << "*sptr2 = " << *sptr2 << endl;
    return 0;
}
```

![](/images/cpp/sp3.jpg)

上述代码创建了一个shared_ptr指针指向了一个装着整型值且值为100的内存块，并且引用计数为1,。当其他共享指针通过sptr1来创建时，引用计数将为2。这种引用称为强引用strong reference。除此之外，还有弱引用weak reference。可以通过

destruction ：
```c++
#include <iostream>
#include <memory>

using namespace std;

class Test {
public:
    Test(int a = 0) : m_a(a) {}
    ~Test() {
        cout << "Calling destructor" << endl;
    }

public:
    int m_a;
};


int main()
{
    // 如果用以下形式，则只会调用delete，则不会调用
    // delete[ ]; 此时只会调用一次析构函数
    shared_ptr<Test> sptr1(new Test[5]);

    // 采用以下形式，lambda表达式，显示调用
    // delete[] 来删除所有的对象。
    shared_ptr<Test> sptr2(new Test[5],
                           [](Test* p) {delete[] p;});
    return 0;
}
```

用户可以显式的调用函数，lambda表达式，函数对象来调用对于shared_ptr为数组对象的析构函数delete[]。

##### Interface（接口）

shared_ptr提供解引用*， 以及->来普通指针的相关操作。同时，还提供了以下的接口:

```C++
- get()： To get the resource associated with the shared_ptr.
- reset()： To yield the ownership of the associated memory block. If this is the last shared_ptrowning the resource, then the resource is released automatically.
- unique: To know whether the resource is managed by only this shared_ptr instance.
- operator bool: To check whether the shared_ptr owns a memory block or not. Can be used with an if condition.
```

#### Issue（问题）

- 当一个内存块与shared_ptr绑定相关，并且属于不同组时，将会发生错误。所有的shared_ptr共享一个组的同一个共享引用。

```c++
int main()
{
    shared_ptr<int> sptr1(new int);
    shared_ptr<int> sptr2 = sptr1;
    shared_ptr<int> sptr3;
    sptr3 = sptr2;
    return 0;
}
```

以下表格给出了相应的引用计数 

![](/images/cpp/sp4.jpg)

以上代码运行正常。 
然而当使用以下代码时：

```c++
int main() 
{
    int *p = new int;
    shared_ptr<int> sptr1(p);
    shared_ptr<int> sptr2(p);
    return 0;
}
```

引用计数表如下： 

![](/images/cpp/sp5.jpg)

为了避免这种情况的发生，最好不用从裸指针中建立共享指针。

- 另一个问题是，正如上述问题，如果从一个裸指针中创建一个共享指针，只有一个共享指针时，可以正常运行，但是当裸指针被释放时，共享指针也会crash。

- 循环引用时，如果资源被非恰当释放，也会出现问题。

```c++
#include <iostream>
#include <memory>

using namespace std;

class B;
class A
{
public:
    A() : m_sptrB(nullptr) {} ;
    ~A() {
        cout << "A is destroyed" << endl;
    }
    shared_ptr<B> m_sptrB;
};

class B
{
public:
    B() : m_sptrA(nullptr) {};
    ~B() {
        cout << "B is destroyed" << endl;
    }
    shared_ptr<A> m_sptrA;
};

int main()
{
    shared_ptr<B> sptrB(new B);
    shared_ptr<A> sptrA(new A);
    sptrB->m_sptrA = sptrA;
    sptrA->m_sptrB = sptrB;
    return 0;
}
```

当类A包含了指向B的共享指针，而类B包含了指向A的共享指针时，sptrA和sptrB相关的资源都将不会被释放。结果如下图： 

![](/images/cpp/sp6.jpg)

#### 2.3 weak_ptr

一个弱指针，提供的是一种共享语义定义而不是拥有语义定义。这就意味着一个弱指针可以通过shared_ptr共享资源。所以要创建弱指针，必须是已经拥有资源但是是一个共享指针。

一个弱指针并不允许诸如普通指针所提供的*和->。因为他并不是资源的拥有者。

##### 那么如何利用弱指针呢？ 

**weak_ptr只能用于跟踪一个共享的资源，但并不实际拥有，也不会阻碍资源的释放。读取共享资源前需要先执行lock，得到shared_ptr后才能进行访问。 
当两个对象需要互相引用时，我们总希望其中一个对象拥有另一个对象的强引用，而另一个对象拥有自己的弱引用，如果两个对象都是强引用，则容易引起循环引用，导致两个对象都无法正确释放。**

##### 用weak_ptr作为一个类似share_ptr但却能悬浮的指针 

有一个矛盾，一个灵巧指针可以像shared_ptr 一样方便，但又不参与管理被指对象的所有权。换句话说，需要一个像shared_ptr但又不影响对象引用计数的指针。这类指针会有一个shared_ptr没有的问题：被指的对象有可能已经被销毁。一个良好的灵巧指针应该能处理这种情况，通过跟踪什么时候指针会悬浮，比如在被指对象不复存在的时候。这正是weak_ptr这类型灵巧指针所能做到的。

weak_ptr一般是通过shared_ptr来构造的。当使用shared_ptr来初始化weak_ptr时，weak_ptr就指向了相同的地方，但是不改变所指对象的引用计数。
```c++
int main()
{
    shared_ptr<A> sptr<new A>
    weak_ptr<A> wptr(sptr);
    weak_ptr<A> wptr1 = wptr; 
    return 0;
}
```
weak_ptr创建引用情况 

![](/images/cpp/sp7.jpg)

从上图可以看书，通过将一个weak_ptr赋值给另一个时会增加其弱引用计数。

如果弱引用指针所指向的资源，被其共享指针所释放时，这时候弱指针将会过期。如何检测一个弱指针是否指向一个合法的资源呢？有以下两种途径。

调用use_count()来得到引用计数。注意这里返回的是强引用计数。
调用expired()函数，这比调用use_count要快的多。
同时，我们可以通过对一个weak_ptr调用函数lock()来得到一个shared_ptr。或者直接对一个weak_ptr进行强制转换。
```c++
int main()
{

    shared_ptr<A> sptr<new A>
    weak_ptr<A> wptr(sptr);
    shared_ptr<A> sptr2 = wptr.lock(); 


    shared_ptr<B> sptrB(new B);
    shared_ptr<A> sptrA(new A);
    sptrB->m_sptrA = sptrA;
    sptrA->m_sptrB = sptrB;
    return 0;
}
```
如上方法将增加强引用计数。

以下例子将展示如何使用weak_ptr解决循环引用的问题。
```c++
#include <iostream>
#include <memory>

using namespace std;

class B;
class A
{
public:
    A() : m_a(5) {} ;
    ~A() {
        cout << "A is destroyed" << endl;
    }
    void PrintSpB() ;
    weak_ptr<B> m_sptrB;
    int m_a;
};

class B {
public:
    B() : m_b(10) {} ;
    ~B() {
        cout << "B is destroyed" << endl;
    }
    weak_ptr<A> m_sptrA;
    int m_b;
};

void A::PrintSpB()
{
    if( !m_sptrB.expired() )
        cout << m_sptrB.lock()->m_b << endl;
}

int main()
{
    shared_ptr<B> sptrB(new B);
    shared_ptr<A> sptrA(new A);
    sptrB->m_sptrA = sptrA;
    sptrA->m_sptrB = sptrB;
    sptrA->PrintSpB();

    return 0;
}
```

#### 2.4 unique_ptr

unique_ptr几乎是易出错的auto_ptr的另一种形式。unique_ptr遵循专用所有权语义。在任何时刻，资源只被唯一的一个unique_ptr所占有。当auto_ptr不在作用域范围内时，资源就会被释放。当一个资源被其他资源重写时，如果先前的资源已经被释放，这保证了相关的资源也会被释放。

creation（创建）

unique_ptr创建的过程和shared_ptr创建的过程大同小异，所不同的是创建的数组形式的对象。

```c++
unique_ptr<int> uptr(new int);
```

unique_ptr提供了专用创建数组对象的析构调用delete[]而不是delete当其不在作用域范围内。

```c++
unique_ptr<int[]> uptr1(new int[5]);
```
对于资源的拥有权(ownership)可以从一个unique_ptr通过另一个进行赋值来传递。

需要记住的是：unique_ptr并不提供复制机制copy semantics（包括复制赋值copy assignment以及复制构造函数copy construction）而是一种移动机制。

Interface（接口）

unique_ptr提供的接口与普通常规指针的接口非常相似，但是并不提供指针运算。 
unique_ptr提供release()函数来进行yield the ownership。release()和reset()函数的区别在于，reset()会对资源进行销毁。

参考文献：

http://blog.csdn.net/coolmeme/article/details/43266319
http://www.codeproject.com/Articles/541067/Cplusplus-Smart-Pointers