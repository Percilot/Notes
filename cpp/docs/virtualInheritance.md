# 虚继承与虚基类

## 为什么需要虚继承？
由于C++中允许多继承，我们几乎必然会遇到菱形继承的问题：

```cpp
class Grandfather
{
	int m_a;   
};

class Father : public Grandfather
{
    int m_b;
};

class Mother : public Grandfather
{
    int m_c;
};

class Son : public Father, public Mother
{
    int m_d;
};
```

这种菱形继承会使得`Son`类实质上存储有两份`Grandfather`的数据，导致了浪费和歧义。

歧义：例如我们访问 `Son.m_a`，我们无法确定这个`m_a`究竟是来自`Father`类还是`Mother`类。

为了解决这个问题（其实就是给多继承打补丁），我们引入了虚继承。



## 虚继承的使用

虚继承通过关键字`virtual`实现，此时派生类中只会保留一份间接基类的成员。

所有声明为虚继承的类都会作出以下保证：它承诺共享它的基类（解决数据冗余），这一基类也因此得名虚基类。对于它的派生类，它将不再自己调用虚基类的构造函数，而是转交给**最终的派生类**进行唯一一次构造（解决命名冲突）。

需要注意的是，在最终的派生类进行构造时，编译器会先调用虚基类的构造函数，之后再调用直接基类的构造函数。
