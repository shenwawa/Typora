# Block相关记录



## block的本质

`block`其本身也是一个OC对象，它内部也有一个isa指针。`block`对象内部封装了函数的调用以及函数调用的环境，也可以说成是封装了函数以及函数上下文。在编译的时候，编译器会将block的表达式以及block变量一起编译成一般C的结构体、函数，因为block的调用就是函数指针的调用。block表达式中使用局部变量时，编译生成的结构体`__main_block_impl_0`会持有变量，也就是对变量进行捕获，该结构体会带着默认值初始化结构实例。

下面我通过下图来具体了解下`block`的底层结

<img src="/Users/taeja/Documents/Typora/imgs/block_struct.png" alt="block_struct" style="zoom:75%;" />

底层代码如下：

```objective-c
struct __block_impl {
    void *isa;
    int Flags;
    int Reserved;
  	//指向调用函数的地址
    void *FuncPtr;
};

struct __main_block_impl_0 {
  struct __block_impl impl;
  // block的相关描述信息
  struct __main_block_desc_0* Desc;
  // 构造函数（类似于OC的init方法），返回结构体对象
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

// 封装了block执行逻辑的函数
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {

            NSLog((NSString *)&__NSConstantStringImpl__var_folders_2r__m13fp2x2n9dvlr8d68yry500000gn_T_main_c60393_mi_0);
        }

static struct __main_block_desc_0 {
  size_t reserved;
  //block对象的内存大小
  size_t Block_size;
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};
int main(int argc, const char * argv[]) {
    /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool;
        // 定义block变量
        void (*block)(void) = &__main_block_impl_0(
                                                   __main_block_func_0,
                                                   &__main_block_desc_0_DATA
                                                   );

        // 执行block内部的代码
        block->FuncPtr(block);
    }
    return 0;
}
```



## Block捕获变量

首先我们来看下面代码，然后判断下输出结果，代码如下：

```objective-c
// 示例1
int age = 10;
void (^Block)(void) = ^ {
  NSLog(@"age:%d", age);
};
age = 20;
Block();
```

通过我们上面对`block`源码的分析，我们可以很清楚的知道，当我们在block内部应用`age`的时候，此时block已将`age`这个局部变量进行了捕获，存放到了内部的`variables`当中，也就是另存了一份，当我们在外部修改值得时候，block内部存的变量值是不会被修改的，所以输出结果是age: 10



好我们再继续看下面这个例子，然后判断看看输出结果是什么，代码如下：

```objective-c
//auto关键字，自动变量关键字，OC中所有基本数据类型的局部定义都默认有这个关键字，OC中被隐藏掉了。在C和C++依然在使用。
auto int age = 10;
static int num = 25;
void (^Block)(void) = ^ {
  NSLog(@"age: %d, num: %d", age, num);
};
age = 20;
num = 11;
Block();
```

我们结合block的特性，可以分析吃，age为局部变量，会被block进行捕获，将其深复制一份存放遭variables中，所以age: 10输出值是不变得。但是num是个静态变量（更准确的说是静态局部变量）, block在捕获该变量时，时将其浅拷贝一份存放到variables中，所以，修改num的的值是会影响block中的捕获值的，所以num：11为输出结果。

说道着，我们还有作用范围的变量，叫做全局变量，全局变量是不会被block捕获的。下面是一个参考表，可以简要记忆三种作用域的变量和block之间的捕获关系。表格如下：

<img src="/Users/taeja/Documents/Typora/imgs/block_capture_excel.png" alt="block_capture_excel" style="zoom:60%;" />

当然还有其他的一些引用对象也会block捕获到，比如在block内部通过self进行函数调用，在调用函数的时候self指向的是被调用者，所以也会对self进行捕获，所以通常我们会通过``__weak typeof(self) weakSelf;``这种语法来定义一个self的弱一样来避免捕获self时候出现的循环引用。同样所有的所有的属性，也会被block捕获。实际上我们在访问属性的时候，会调用getter方法，所以block会先捕获self，再捕获属性。



## Block的类型

通常block的类型取决它内部的isa指针，可以通过调用class方法或访问isa指针来查看block的具体类型，这些类型都是继承与`NSBlock`。一般block有以下几种子类型。

**_NSGlobalBlock_ (_NSConcreteGlobalBlock)**

**_NSStackBlock_ (_NSConcreteGlobalBlock)**

**_NSMallocBlock_ (_NSConcreteGlobalBlock)**

这三种block子类的局别在于他们的内存分配

**_NSGlobalBlock_ (_NSConcreteGlobalBlock) ** --> 内存分配在数据区

**_NSStackBlock_ (_NSConcreteStackBlock) ** 	-->  内存分配在栈区，自动进行内存分配，自动回收。

**_NSMallocBlock_ (_NSConcreteMallocBlock) ** --> 内存分配在堆区，动态内存分配，需手动管理。

