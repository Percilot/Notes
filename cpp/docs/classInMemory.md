

# 类的内存模型

在这一部分我们将描述C++中类在内存中的布局。



## 对象成员布局

C++对象成员可以分为两类：成员函数和成员属性。成员函数可以分为：构造函数、析构函数、运算符重载函数，虚成员函数，静态成员函数、普通成员函数、内联成员函数；成员属性可以分为：普通成员属性，静态成员属性。

接下来我们对于各项属性进行分析：

1. 成员函数：存放于整个程序的代码部分。这主要是由于成员函数与对象无关，仅与类相关。

2. 虚函数表指针：由于要实现运行时多态，每个对象需要维护虚函数表。

   - 每个对象都有一个指向虚表的指针，这个指针存放在对象起始地址处。

   - 由**同一个类**实例化的所有对象共用一个虚表
   - 虚函数表本体位于只读数据段

3. 静态成员属性：存放在静态存储区

4. 普通成员属性：存放在对象中，以字节对齐方式存放的。

   字节对齐规则如下：

   - 将各数据成员内存对齐。 对齐模数是【该数据成员所占内存】与【`#pragma pack`指定的数值】中的较小者。数据成员所占内存应该是该模数的整数倍；另外，如果对齐模数为 N ，存放的起始位置就是 N 的整数倍；
   - 将结构体整体内存对齐。 对齐模数是【结构体内部最大的基本数据类型成员】、【`#pragma pack`指定的数值】长度中数值较小者。结构体所占内存以及起始位置应该是该模数的整数倍；
   - 结构体嵌套结构体时，对齐模数是【子结构体中数据类型最大值】 和 【`#pragma pack`指定的数值】 中的较小值。结构体成员所占内存以及起始位置应该是该模数的整数倍。



## 复合中的内存布局

当类中包含其他的类时，这种情况比较简单，只需要类比内存对齐规则即可。我们首先需要将被包含的类内部成员变量按内存对齐规则排布后，再与其他成员按内存对齐规则排布。**复合中存在两次内存对齐。**



## 继承中的内存布局

### 普通继承

#### 无虚函数的单继承

这一部分的内存操作与复合中相似而又不同。在继承时，子类直接将父类的成员变量排布在子类的成员变量之前，之后**只使用一次内存对齐**。



#### 含虚函数的单继承

需要指出的是，虚函数表位于父类的成员变量之前，此时内存空间布局如下：

```
虚函数表指针		父类成员变量		子类成员变量
```

根据子类的不同属性和操作，虚函数表又会产生变化：

1. 如果子类中没有自己的虚函数，那么只需要将父类的虚函数表复制一份即可。
2. 如果子类中有自己的虚函数，那么不仅需要复制父类的虚函数表，还需要在父类的虚函数表**之后**添加上自己的虚函数地址，构成一份新的虚函数表。
3. 如果子类覆写了父类的虚函数，那么就需要修改复制来的父类的虚函数表，将对应项修改为子类的函数地址。



#### 多重继承

上面我们分析了二重继承，与之类似的我们可以分析多重继承。

多重继承的本质并没有什么变化，成员变量仍然按照父类到子类到孙类的顺序排布。同时由于虚函数表是可以继承的，所以仍然只存在一张虚函数表，只不过进行了多次如上文中所述的复制和修改过程。



#### 多继承

接下来我们考虑多继承的情况。这一部分的情况比较复杂，我们分情况讨论。

1. 最简单的情况则是两个父类中均不含虚函数，内存布局仅为按照声明顺序排布：

   ```
   第一个父类成员变量	第二个父类成员变量	子类成员变量
   ```

2. 当有且仅有一个父类中存在虚函数时，内存布局将必然按照如下的方式排布，而**与声明顺序无关**：

   ```
   虚函数表指针	有虚函数的父类成员变量	无虚函数的父类成员变量	子类成员变量
   ```

3. 当有多个父类存在虚函数时，内存布局将按照声明顺序排布，但需要注意的是，子类的虚函数将新增在第一个虚函数表中，而覆写可以发生在每一个虚函数表中。



### 虚继承

在C++中，所有虚继承而来的派生类会生成一个隐藏的虚基类表指针，这个虚基类指针往往位于虚函数表指针之后（如果没有虚函数表指针那么就将位于内存最前端）。虚基类表与虚函数表类似，存在多个条目。虚基类表的第一、第二、第三……条目以此为本类、最左虚继承父类、次左虚继承父类……起始地址与虚基类表指针之间的偏移值。



#### 无虚函数的单虚继承

这一部分的内存操作普通继承恰好相反。在继承时，子类直接将父类的成员变量排布在子类的成员变量之**后**。



#### 含虚函数的单虚继承

1. 若只有父类有虚函数，此时子类会继承父类的虚表，同时按照与普通继承相反的顺序放置成员变量：

   ```
   虚函数表指针		虚基类表指针		子类成员变量		父类成员变量
   ```

2. 若只有子类有虚函数，此时子类会新建一张虚表，内存布局如下：

   ```
   虚函数表指针		虚基类表指针		子类成员变量		父类成员变量
   ```

3. 若子类和父类均有虚函数，此时子类将会新建属于自己的虚函数表：

   ```
   子类虚函数表指针	虚基类表指针		子类成员变量
   父类虚函数表指针	父类成员变量
   ```

   子类会将自己新增的虚函数填写进子类虚函数表指针指向的虚函数表，若覆写了父类的虚函数，则直接修改对应父类虚函数表中的对应项。



#### 菱形继承

我们以最经典的菱形继承结尾：

```cpp
class Grandfather {

};

class Father : virtual public Grandfather {
    
};

class Mother : virtual public Grandfather {
    
};

class Son : public Father, public Mother {
    
};
```

- 对于`Son`类内存而言，基类出现的顺序是：`Father`（最左父类）、`Mother`（次左父类）、`Grandfather`（虚祖父类）。
- `Son`类本身的数据位于`Grandfather`类之前。
- 编译器并没有为`Son`类本身生成虚函数表，而是覆盖并拓展了最左父类`Father`的虚函数表。
- 共同虚基类`Grandfather`的内容放到了派生类`Son`对象内存的最后。

