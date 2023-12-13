# C++八股文

## 基础

### 1.说一下C++用户控件内存分区

栈区：存储的是函数的局部变量/函数参数/返回地址等，是由编译器自动分配和释放。

堆区：动态申请的内存空间，就是malloc分配的内存块，是由程序员控制分配和释放，如果程序结束还没有有释放，系统会自动回收。

全局/静态存储区：存储的是全局变量和静态变量，程序结束会自动释放。

常量存储区：存放的是常量，不允许修改，程序结束也会自动释放。

代码区：存放代码，不允许修改，编译后的二进制文件存放在这里。



### 2.说一下static关键字的作用？

全局静态变量：存储在静态存储区 程序运行就存在 对外部文件不可见

局部静态变量	存储在静态存储区 在局部作用域内可以访问 离开局部作用域任存在 但不能访问

静态函数	就是在函数前面加static ；一般函数默认是extern，是可以导出的，但是加了static，就只能在本文件访问，不能被外部调用。

类的静态成员  静态成员在类的所有对象中是共享的。如果不存在其他的初始化语句，在创建第一个对象时，所有的静态数据都会被初始化为零。不需要类名就可以访问，类的静态成员变量是可以修改的，通过<对象名>::<静态成员>进行访问。

类的静态函数 如果把函数成员声明为静态的，就可以把函数与类的任何特定对象独立开来。静态成员函数即使在类对象不存在的情况下也能被调用。静态函数只要使用类名加范围解析运算符 :: 就可以访问。

`***静态成员函数与普通成员函数的区别：***`

- `*静态成员函数没有 this 指针，只能访问静态成员（包括静态成员变量和静态成员函数）。*`
- `*普通成员函数有 this 指针，可以访问类中的任意成员；而静态成员函数没有 this 指针。*`

### 3.讲一下C++里面的四种强制转换？

static_cast/  const_cast/  reinterpret_cast/  dynamic_cast



**static_cast 关键字**

只能用于良性转换，这样的转换风险较低，一般不会发生什么意外，例如：

- 原有的自动类型转换，例如 short 转 int、int 转 double、const 转非 const、向上转型等；
- void 指针和具体类型指针之间的转换，例如`void *`转`int *`、`char *`转`void *`等；
- 有转换构造函数或者类型转换函数的类与其它类型之间的转换，例如 double 转 Complex（调用转换构造函数）、Complex 转 double（调用类型转换函数）

需要注意的是，**static_cast 不能用于无关类型之间的转换，因为这些转换都是有风险的**，例如：

- 两个具体类型指针之间的转换，例如`int *`转`double *`、`Student *`转`int *`等。不同类型的数据存储格式不一样，长度也不一样，用 A 类型的指针指向 B 类型的数据后，会按照 A 类型的方式来处理数据：如果是读取操作，可能会得到一堆没有意义的值；如果是写入操作，可能会使 B 类型的数据遭到破坏，当再次以 B 类型的方式读取数据时会得到一堆没有意义的值。
- int 和指针之间的转换。将一个具体的地址赋值给指针变量是非常危险的，因为该地址上的内存可能没有分配，也可能没有读写权限，恰好是可用内存反而是小概率事件

```c++
    //下面是正确的用法
    int m = 100;
    Complex c(12.5, 23.8);
    long n = static_cast<long>(m);  //宽转换，没有信息丢失
    char ch = static_cast<char>(m);  //窄转换，可能会丢失信息
    int *p1 = static_cast<int*>( malloc(10 * sizeof(int)) );  //将void指针转换为具体类型指针
    void *p2 = static_cast<void*>(p1);  //将具体类型指针，转换为void指针
    double real= static_cast<double>(c);  //调用类型转换函数
   
    //下面的用法是错误的
    float *p3 = static_cast<float*>(p1);  //不能在两个具体类型的指针之间进行转换
    p3 = static_cast<float*>(0X2DF9);  //不能将整数转换为指针类型
```



**const_cast 关键字**

它用来去掉表达式的 const 修饰或 volatile 修饰。换句话说，const_cast 就是用来将 const/volatile 类型转换为非 const/volatile 类型。

```c++
 	const int n = 100;
    int *p = const_cast<int*>(&n);
    *p = 234;
    cout<<"n = "<<n<<endl;
    cout<<"*p = "<<*p<<endl;
    return 0;
```

&n 获取的是n的地址，它为const int *类型，必须使用const_cast转化为int *后才能赋值给p。由于p指向了n，并且n占用的是栈内存，有写入权限，所以可以通过p来修改n的值；换句话说，使用 const_cast 进行强制类型转换可以突破 C/C++ 的常数限制，修改常数的值，因此有一定的危险性；但是程序员如果这样做的话，基本上会意识到这个问题，因此也还有一定的安全性。

*有可能会问，为什么通过 n 和 *p 输出的值不一样呢？这是因为 C++ 对常量的处理更像是编译时期的`#define`，是一个值替换的过程，代码中所有使用 n 的地方在编译期间就被替换成了 100。换句话说，“cout<<"n = "<<n<<endl;” 行代码被修改成了下面的形式：*

`cout<<"n = "<<100<<endl;`

这样以来，即使程序在运行期间修改 n 的值，也不会影响 cout 语句了。



**reinterpret_cast 关键字**

reinterpret 是重新解释的意思。顾名思义，reinterpret_cast这种转换仅仅是对二进制位的重新解释，不会借助已有的转换规则对数据进行调整，非常的简单粗暴，所以风险很高。

reinterpret_cast 可以认为是 static_cast 的一种补充，一些 static_cast 不能完成的转换，就可以用 reinterpret_cast 来完成，例如两个具体类型指针之间的转换、int 和指针之间的转换（有些编译器只允许 int 转指针，不允许反过来）。



**dynamic_cast 关键字**

dynamic_cast 用于在类的继承层次之间进行类型转换，它既允许向上转型（Upcasting），也允许向下转型（Downcasting）。向上转型是无条件的，不会进行任何检测，所以都能成功；向下转型的前提必须是安全的，要借助 RTTI 进行检测，所有只有一部分能成功。

dynamic_cast 与 static_cast 是相对的，dynamic_cast 是“**动态转换**”的意思，static_cast 是“**静态转换**”的意思。dynamic_cast 会在程序运行期间借助 RTTI 进行类型转换，这就要求基类必须包含虚函数；static_cast 在编译期间完成类型转换，能够更加及时地发现错误。



### 4. static_cast和reinterpret_cast 它们的区别知道吗？

static_cast指向和来自void*的指针保留地址。也就是说，在下面a,b和c都指向同一个地址：

```c++
int* a = new int();
void* b = static_cast<void*>(a);
int* c = static_cast<int*>(b);
```

reinterpret_cast保证只有当指针转换为不同的类型，然后将其reinterpret_cast 恢复为原始类型，你将获得原始值。

```c++
int* a = new int();
void* b = reinterpret_cast(void*)(a);
int* c = reinterpret_cast<int*>(b);
```



a和c都包含相同的值，但未指定b的值。

总结：对于void*的转换，应该首选static_cast。对于模糊类型的转换，应该使用reinterpret_cast.



### 5.说一下c++指针和引用的区别？

**相同点**

都是地址的概念； 指针指向一块内存，它的内容是所指内存的地址；引用是某块内存的别名

**不同点**

   1.指针是一个实体，而引用仅是个别名；

2. 引用使用时无需解引用（*），指针需要解引用；
3. 引用只能在定义时被初始化一次，之后不可变；指针可变；

4. 引用没有 const，指针有 const，const 的指针不可变；
5. 引用不能为空，指针可以为空；
6. “sizeof 引用”得到的是所指向的变量（对象）的大小，而“sizeof 指针”得到的是指针本身（所指向的变量或对象的地址）的大小；

  typeid（T） == typeid（T&） 恒为真，sizeof（T） == sizeof（T&） 恒为真，但是当引用作为成员时，其占用空间与指针相同（没找到标准的规定）。

7. 指针和引用的自增（++）运算意义不一样；



### 6.讲一讲int* p[n] 和 int(* p)[n] 以及int* p()和int（* p）()的区别

tips:牢记”[]”的优先级比较高，所以要用优先级更高的"()"做区分



int *p[n] ：由于“[]”的优先级比较高，所以先看p[],说明p是一个数组，再看剩下的部分得知，p这个数组里面的元素类型为int型指针。

int（*p）[n] :  由于“()”的优先级比较高，所以先看 * p ，说明p是一个指针，再看剩下的部分，得知p指针指向的是 一个长度为n的一维数组。

int* p( ) : 表示p为函数，范围值为int*

int (*p)() 表示p为函数指针，函数原型int func()



### 7.volatile关键字有什么用？

C++中的volatile和const对应。表示变量可以被编译器未知的因素所更改，比如操作系统、硬件或者其它线程。遇到volatile，编译器就不再进行优化，从而提供对特殊地址的稳定访问。



### 8.C++的智能指针用过吗？怎么样？

C++的智能指针均位于<memory>库里，有四种：shared_ptr/unque_ptr/weak_ptr/auto_ptr

**shared——ptr**

共享指针，（指向同一块内存）只有最后一个共享指针被释放，资源会自动销毁。（tips：当有裸指针混用时，只要共享指针都被释放，内存就会被销毁，裸指针就会悬空）

原理：无非是利用一个计数器，当发生使用赋值拷贝构造函数，运算符重载，作为函数返回值，或者作为一个参数传递给另一个参数，计数+1，当shared_ptr赋新值或者销毁，计数-1，直到计数为0，调用析构函数释放对象。

缺点是存在循环引用问题，见9；

```c++
std::shared_ptr<int> sptr = std::make_shared<int>(...); //初始化方法一 效率更高
std::shared_ptr<int> sptr(new int(...)); //初始化方法二

sptr.reset(); //指针释放
sptr.rest(new int) //原共享指针组-1，指向新内存

```

**unique_ptr**

独占式的指针，离开unique_ptr对象的作用域时，会自动释放资源。

由于指针或引用在离开作用域是不会调用析构函数的，但对象在离开作用域会调用析构函数。unique_ptr本质是一个类，将**复制析构函数**和**赋值析构函数**声明为delete

```c++
std::unique_ptr<int> uptr = std::make_unique<int>(...);//初始化方法一 效率更高
std::unique_ptr<int> uptr(new int(...));//初始化方法二

uptr.release();//接触uptr与资源的绑定，并返回一个指向资源的裸指针
delete uptr;
uptr = nullptr;

unique_ptr<int> uptr1 = make_unique<int>(100);
unique_ptr<int> uptr2(uptr1.release()); //接触资源与uptr1的绑定，并直接与uptr2绑定
```

**weak_ptr**

与shared_ptr一起使用，作为资源的观察者，不影响对象的引用计数。

**auto_ptr**

已被弃用

## 9.什么是shared_ptr 的循环引用问题，如何解决？

<img src="C:\Users\Luc\Pictures\Camera Roll\循环引用示意图.png" style="zoom: 25%;" />

```c++
struct box
{
    int data;
    shared_ptr<box> ptr;
    box(int data):_data(data){};
    ~box(){cout<<"~box()"<<endl;}
};

int main()
{
    shared_ptr<box> boxA = make_shared<box>(1);
    shared_ptr<box> boxB = make_shared<box>(2);
    
    cout << boxA.use_count() << endl;//1
    cout << boxA.use_count() << endl;//1
    
    boxA->ptr = boxB;
    boxB->ptr = boxA;
   
    cout << boxA.use_count() << endl;//2
    cout << boxA.use_count() << endl;//2
           
    return 0;
}
```

最一般的情况是，对象A内有一个shared_ptr指向对象B，对象B内也有一个shared_ptr指向A，另外，又有两个单独的shared_ptr分别指向A/B;

A与B分别有两个shared_ptr指向他们，所以他们引用计数均为2，造成了循环引用，谁也不会被释放，因为始终有另外一个指针指向它。

解决方法：1.当剩下最后一个引用时，需要收到打破循环引用释放对象；

​		   2.当A的生存周期超过B的生存周期，B改为一个普通指针指向A；

​		   3.将共享指针改为弱指针weak_ptr;

一般采用第三种方法，原理是弱指针的指针不会增加shared 引用数量。

```c++
struct box{   
    int data;   
    weak_ptr<box> ptr;   
    box(int data):_data(data){};  
    ~box(){cout<<"~box()"<<endl;}
};
    int main(){    
        shared_ptr<box> boxA = make_shared<box>(1); 
        shared_ptr<box> boxB = make_shared<box>(2);  
        cout << boxA.use_count() << endl;//1 
        cout << boxA.use_count() << endl;//1    
        boxA->ptr = boxB; 
        boxB->ptr = boxA;   
        cout << boxA.use_count() << endl;//1  
        cout << boxA.use_count() << endl;//1   
        return 0;
    }
```

### 10.unique指针如何在函数作为参数传递/unique指针在函数间的使用方法？

**改变unique所指的资源**

首先unique指针是独占的，不能赋值（unique_ptr不是不能赋值，而是不能赋左值，可以赋右值），直接作为函数参数传递显然是不行的。

以下是错误用法

```c++
void pass_up(unique_ptr<int> up) //错误用法
{
    cout<<"In pass_up"<<*up<<endl;
}

int main()
{
    auto up = make_unique<int>(123);
    pass_up(up);           //错误用法
}
```

其实，我们不需要传递unique_ptr本身，我们只需要传递unique_ptr所指向的资源。

方法一

```c++
void pass_up(int& value) //参数是 int 的引用
{
    cout<<"In pass_up"<<*up<<endl;
}

int main()
{
    auto up = make_unique<int>(123);
    pass_up(*up);           
}
```

方法二

```c++
void pass_up(int* p)  
{
    cout<<"In pass_up"<<*up<<endl;
}

int main()
{
    auto up = make_unique<int>(123);
    pass_up(up.get());       //up.get() 返回的是资源的裸指针
}
```

**改变unique本身**

```c++
void pass_up(unique_ptr<int> up)  
{
    cout<<"In pass_up"<<*up<<endl;
}

int main()
{
    auto up = make_unique<int>(123);
    pass_up(move(up));//move()函数作用是release()类似，解除up与资源的绑定，可以转移给另外一个unique_ptr
    if(up == nullptr) cout<<"up is moved."<<endl;//up指针不再指向原资源，变成空指针了
}
```

**unique可以作为函数返回值**



```c++
unique_ptr return_uptr(int value)  
{
   	unique_ptr<int> up = make_unique<int>(value);
    return up;
}

int main()
{
    unique_ptr<int> up = return_uptr(321);//unique_ptr不是不能赋值，而是不能赋左值，可以赋右值 而return的东西已经是右值了
    cout<<"up:"<<*up<<endl;
}
```

### 11.数组与指针的区别？指针数组和数组指针？

数组存放一组元素，而指针指向某一个对象。从底层实现上看，数组也是由base指针和各维度长度等组成，数组元素曾放在连续地址上。

指针数组是保存指针的数组，比如int* a[10],而数组指针是指向数组的指针，比如：

```c++
int var[10];
int *ptr = var;
int *ptr = &var[10];//与上面等价
```

在C++中，数组名代表数组中第一个元素（即序号为0的元素）的地址。如果是二维数组，则可以通过* (* (arr+i)+j)来访问arr[i] [j]

### 12.你知道函数指针吗？讲一讲

函数指针是指向函数的指针，是早期c的项目中经常能看到。

作用1：通过函数指针调用函数 

```c++
int foo(){return -1;}

int (*ptrfoo)() = foo;//不要写成foo()
ptrfoo();//使用方式1
(*ptrfoo)();//使用方式2

```

作用2：回调函数（以函数指针为参数的函数）

```c++
int Max(int a, int b){
    return a>b?a:b;
}
int Min(int a, int b){
    return a<b?a:b;
}

void printData(int (*p)(int,int),int a,int b)  //充当回调函数
{
    cout<<p(a,b)<<endl;
}

int main()
{
    void (*pp)(int (*)(int,int),int,int) = printData; //指向printData的函数指针
    pp(Max,1,2);
    pp(Min,1,2);
}

```

### 13.什么是注册函数？什么是回调函数？

**回调函数**无非是对函数指针的应用，用函数指针来**调用一个函数**，而**注册函数**则是将**函数指针**作为参数传进去，便于其他函数调用。



### 14.C++从源文件到可执行文件需要经历哪些步骤？

**预处理阶段(preprocessing)-》编译阶段(compilation)-》汇编阶段(assembly)-》链接阶段(linking)**

**预处理阶段**：编译器对文件包含关系进行检查（头文件和宏），将其做相应替换，生成.i文件；

**编译阶段：**将预处理的生成文件转化为汇编文件.s;

**汇编阶段：**将汇编文件转化为二进制机器码，对应后缀是.o(Linux),.obj(Windows);

**链接阶段：**将多个目标文件及所需要的库连接成可执行文件.out(Linux) .exe(Windows);
