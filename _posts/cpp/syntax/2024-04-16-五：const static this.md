---
title: 五：const static this
date: 2024-04-16 00:00:00 +0800
last_modified_at: 2024-04-22 00:00:00 +0800
categories: [cpp, syntax]
tags: [cpp]
author: author1
---

### 前文

本章更新cpp代码中常见的一些关键字。
本章节涉及到的变量和内存模型在**六：C++内存模型**中有更详细的说明。

### 正文

#### const

`const`意味着不能修改，当然这只是一个语法，并不意味着真不能修改`const`修饰的变量，而且C++的语法是很自由的，放在不同的地方有不同的效果。

简单的语法是`const 类型 变量名 = 值`，一般常量的变量名为全大写。注意这里必须初始化，`const`意味着不能修改，
先声明在其他地方初始化是不被允许的，如果类型是指针，可以先定义，但不表示`*变量名`能修改。

有趣的来了，前面说语法是自由的，可以放在各种位置，**例1**：
```
int age = 10;
const int* p = &age;
//*p = 20;
age = 20;
```

显然第三行会报错，因为`*p`是`const`，不能被修改，但是修改`age`没问题，或者说这里确实做到了不能修改`*p`，但修改`age`好像又使得`*p`这个值不能被修改失效了，有意思的是，这里只是修饰了`*p`，这不意味着`p`不能被修改，假设接下来是`int age2 = 30; p = &age2;`，打印`*p`就会发现，`*p`的值还是被修改了。

这也是开头说的，**`const`更是且只是一个约定俗成的语法，编译器对`const`做了一些限制，又没有完全限制**。其次，这里const也可以换位置，`int const *p = &age;`，const的位置是自由的。

**例2**：
```
int age = 10;
int* const p = &age;
//p = NULL;
*p = 20;
```

是的，`const`还可以换位置，同上，这里意味着`p`不能被修改，如第三行会报错，但`*p`又可以修改。

当然编译器也不是没考虑到一些情况，**例3**：
```
const int age = 10;
const int* p = &age;
const int age2 = 20;
int* p2 = &age2;
```

第一二行是允许编译通过的，三四不行，这是因为`age2`是常量，`p2`若指向其地址，则`p2`可以修改其值，但是在vs中，编译不通过的理由是`age2`的类型是`const int`，`p2`的类型是`int*`，类型不一致，所以操作来了，完全可以使用`int* p2 = (int *)&age2;`，通过编译，但这也不意味着能通过`*p2`修改`age2`的值，事实上，`*p2 = 100;`，而此时打印`p2`和`&age2`地址是一样的，但`*p2`和`age2`的值分别是`100`和`20`。所以请不要写这么奇怪的代码。

所以，const可以用多次，放多个位置，**例4**：
```
const int* const p = NULL;
```

这意味着`p`和`*p`都不再支持被修改。

同样的，const也可以用来修饰函数中的形参，**例5**：
```
void plusOne(const int x) {...}
```

这意味着`plusOne`的函数体无法改变`x`的值。

const也可以放在类的成员函数函数体前，**例6**：
```
class MyClass {
public:
    int x = 10;
    int getX() const {
        // x = 20;
        return x;
    }
};
```

其中`x = 20;`是错误的，这意味着`getX`只能使用但不能修改类中的变量，这样也会导致无法修改所有的类成员，如果某个类成员是需要被更改的，可以使用`mutable`关键字。

#### static

和`const`更像一个约定的语法不同的是，`static`有明确的作用，在一些问题如不修改已有变量的权限修饰情况下更改其生命周期中发挥作用，即改变变量的存储方式和作用域。

这个存储方式意思是说编译器将内存划分了一些区域：如栈、堆、数据区（静态区、全局区、常量区）等，不同存储区的生命周期是不一样的。

情况1，修饰局部变量：
- 变量的存储由栈变为静态区
- 变量的生命周期由函数调用结束变为程序运行结束
- 变量的**作用域没变**

注意这里作用域没变，或者说相对于每次调用函数在栈上分配空间创建变量，这里只在静态区分配了一次，因此**只会初始化一次**。

情况2，修饰全局变量：
- 变量的存储由全局区变为静态区
- 变量的生命周期仍为程序结束
- 变量的**作用域为当前文件**

注意这里全局变量的作用域变小了，也就是说全局变量本来可以在其他文件被使用，即可以被外部链接，但此时不行了，而后其存储仍在全局变量区，只是在这其中又开辟了一片静态区。
这也意味着不同文件可以有相同名称的静态全局变量。

情况3：修饰函数
- 函数的作用域为当前文件

情况4：修饰类的成员变量、函数
- 修饰的成员存在静态存储区

注意，类中的成员变量修饰后处在静态区，这意味在类中不能被初始化，因为类是处在栈中的，栈在调用后会被销毁，因此这里**初始化需要在类的外部进行初始化**，当然这不意味着静态成员函数的具体定义定义需要在类外实现（语法上方便了写代码），其次，**作用域没变，还是访问修饰符怎么定义就怎么样**，`static`修饰更主要的是换个存储区域，改变生命周期。

这意味着不需要实例，因为这个变量始终只有一份，只需要`类名.变量名`的语法即可使用，这种语法包含了这种意思，即成员属于类而不是属于某个实例。

但需注意问题，访问修饰符怎么定义就怎么样，意味着**外部初始化无法访问private修饰的成员变量**，所以要利用static修饰成员函数（static修饰后成员函数只能使用静态成员变量），然后使用此静态成员函数操作静态成员变量，注意此时这个静态成员函数是公有的。意思就是说这里还是符合封装概念最佳。

总结：

总的来说：一般程序把新产生的动态数据存放在堆区，函数内部的自动变量存放在栈区。自动变量一般会随着函数的退出而释放空间，静态数据（即使是函数内部的静态局部变量）也存放在数据区。数据区的数据并不会因为函数的退出而释放空间。

#### this

`this`关键字从下例开始：
```
class MyClass {
public:
    int x, y;
    // 1
    void setX(int x) {
        // this->x = x;
    }
    // 2
    void setX2(int x) const {
        // (*this).x = x;
    }
    int getX() {
        return x;
    }
}; // 别忘记类后面的分号
```

假设你有如上一个类，如何使用`setX`去给类的`x`赋值呢，毕竟在`setX`中，由于就近原则，使用的`x`肯定是形参的`x`，于是这个时候可以使用`this`关键字，`this`关键字是什么呢，其实在vs中，在`setX`中输入一个`this`，vs会给出提示，如在第1个`setX`中，提示是`MyClass *this`，在第2个用`const`修饰的函数体中，提示是`const MyClass *this`，也就是说，这个`this`是`MyClass`类型的一个指针。当然第2个是错的，因为`const`不能修改类的变量。

注意：
- this只能在成员函数中使用

接着假设现在有：
```
MyClass cls1;
Myclass cls2;
cls1.setX(10);
cls2.setX(20);
cls1.getX();
cls2.getX();
```

看起来好像很直观，`cls1`、`cls2`是两个对象，所以能设置不同的`x`的值并且获取不同的`x`值，其实这里隐藏了一个，即`setX`、`getX`会传入一个隐藏的地址进去，即`&cls1`，`&cls2`，别忘了成员函数中有`this`，此时根据传进来的地址，`return x;`相当于`return this->x`，也相当于`(&cls1)->x`（注意运算符的优先级）。
