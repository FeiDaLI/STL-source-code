阅读材料：[https://blog.csdn.net/qq_41453285/category_9587198.html](https://blog.csdn.net/qq_41453285/category_9587198.html)（STL专栏,可以用来复习用）



<h2 id="ce87533f">第一章</h2>
<h5 id="3314ed5e">为什么要有STL？ STL是什么？</h5>
STL是C++中数据结构和算法的标准库，STL提供了一套容器(vector,list,map等)和算法（如sort、search等）供程序员使用。

STL基于泛型编程思想构建，它的组件可以应用于多种数据类型,无需修改代码。通过使用模板，STL的组件能够适应不同的数据类型。

STL引入了迭代器的概念，它们充当了容器和算法之间的桥梁，允许算法独立于容器类型工作。

STL定义了迭代器萃取类标准，确保了组件之间的解耦(留着以后补充)。

此外，STL还使用了适配器模式来拓展组件，比如将单向迭代器转换为双向迭代器(?)，将红黑树转换为map和set，将deque转换为queue和stack。

<h5 id="90389732">STL组成部分是什么？</h5>
1. **容器**  
容器是用来存储数据的对象，它们提供了各种数据结构，如`vector`（动态数组）、`list`（双向链表）、`deque`（双端队列）、`set`（集合）、`map`（映射）等。容器的设计遵循通用原则，允许它们容纳不同类型的元素，并且可以使用统一的接口进行访问和操作。
2. **算法**  
算法是STL中一系列通用的操作，如排序（`sort`）、查找（`search`）、复制（`copy`）、删除（`erase`）等。这些算法是函数模板，可以作用于任何满足一定条件的容器或序列，提供了强大的数据处理能力。
3. **迭代器**  
迭代器充当了容器和算法之间的桥梁，它们提供了一种统一的方式来访问容器中的元素。迭代器分为五种基本类型：输入迭代器、输出迭代器、前向迭代器、双向迭代器和随机访问迭代器，每种类型支持不同程度的导航和操作。
4. **仿函数**  
仿函数是一种可以像函数那样调用的对象，通常用于定制算法的行为。它们通过重载`operator()`实现，可以作为算法的参数，代表某种策略或计算规则。
5. **配接器（Adapters）**  
配接器用于修改容器、仿函数或迭代器的接口，从而扩展或改变其功能。例如，`queue`和`stack`实际上是基于`deque`的配接器，它们只暴露有限的操作集。配接器可以改变容器的访问方式，调整仿函数的行为，或者转换迭代器的类型。
6. **配置器（Allocators）**  
配置器负责内存的分配和管理，是STL中较低级别的组件，它们允许用户自定义如何分配和释放内存，这对于资源管理和性能优化至关重要。

<h2 id="fccb0eec">第二章 空间配置器(allocators)</h2>
关联知识点:new/delete，operator new/delete，placement new/delete，malloc/free

拓展知识点：伙伴系统，内存池，C++17中引入的空间配置器，FreeRTOS中的内存分配策略的比较，linux内存管理(伙伴系统.....空闲链表，位图法等等)

<h5 id="9f2aee24">为什么说空间配置器而不是内存配置器？</h5>
因为空间不一定是内存，也可以是磁盘或者其他存储介质。

<h5 id="aa055ffb">什么是 std::allocator?</h5>
std::allocator是C++标准库中的分配器。std::allocator的实现比较简单，只是对 opeartor new和和opeartor delete进行包装。

+ `std::allocator` 是 C++ 标准库的一部分，定义在 `<memory>` 头文件中，并被广泛使用。
+ `std::alloc` 是 SGI STL 的一个内部实现类，未成为 C++ 标准的一部分，但可以用来学习STL 内存管理。

std::allocator的操作：

内存分配和释放：allocate和deallocate 封装了opeartor new 和opeartor delete。

构造和析构：construct和 destroy

<h5 id="78bdffec">什么是SGI std::alloc?</h5>
`std::alloc` 是 SGI STL 中用于内存分配的一个实现，具有历史意义，但未成为 C++ 标准的一部分。

<h6 id="b19f92bb">std::alloc construct</h6>
调用placement new

```cpp
template <class _T1, class _T2>
inline void _Construct(_T1* __p, const _T2& __value) {
    new ((void*) __p) _T1(__value);  // placement new; ����T1::T1(value)
}
template <class _T1>
inline void _Construct(_T1* __p) {
    new ((void*) __p) _T1();
}
```

<h6 id="cf6847b6">std::alloc destory</h6>
destory中使用了萃取技术：根据type_tritis判断类型是否有非平凡的析构函数，根据true_type和false_type 选择不同的重载函数。

如果是非平凡的显示调用对象的析构函数，如果是平凡的不做任何操作。

<font style="background-color:#e6faeb;">🌍</font>

<font style="background-color:#e6faeb;">调用链：</font>

<font style="background-color:#e6faeb;">__destroy(_ForwardIterator __first, _ForwardIterator __last, _Tp*)</font>

<font style="background-color:#e6faeb;">调用 __destroy_aux(__first, __last, true_type(false_type));</font>

<font style="background-color:#e6faeb;">调用 _Destroy(_Tp* __pointer)</font>

```cpp
template <class _ForwardIterator, class _Tp>
inline void 
__destroy(_ForwardIterator __first, _ForwardIterator __last, _Tp*)
{
    typedef typename __type_traits<_Tp>::has_trivial_destructor
    _Trivial_destructor;
    __destroy_aux(__first, __last, _Trivial_destructor());
}

template <class _ForwardIterator>
void __destroy_aux(_ForwardIterator __first, _ForwardIterator __last, __false_type)
{
    for ( ; __first != __last; ++__first)
        destroy(&*__first);
}

template <class _ForwardIterator> 
inline void __destroy_aux(_ForwardIterator, _ForwardIterator, __true_type) {}


template <class _Tp>
inline void _Destroy(_Tp* __pointer) {
    __pointer->~_Tp();  
}
```

疑问:value_traits如何获取一个类型是否具有非平凡的函数？

+ `std::is_trivially_constructible<T>`: 检查平凡构造函数
+ `std::is_trivially_copyable<T>`: 检查是否可以平凡拷贝
+ `std::is_trivially_destructible<T>`: 检查平凡析构函数
+ `std::is_trivially_copy_assignable<T>`: 检查平凡赋值操作符

（原理不用搞懂）

<h6 id="m1u0B">`std::alloc 内存分配`</h6>
**SGI的双层级内存配置器策略**

<h3 id="6c603d3c">一级空间配置器</h3>
处理大于128字节的内存的申请与释放。

+ **内存管理**：
    - 封装了malloc和free，并处理内存不足（OOM，Out Of Memory）的情况。
    - 当内存申请失败时，会调用用户自定义的OOM处理函数（使用函数指针）。如果未设置处理函数，则会抛出`BAD_ALLOC`异常来终止程序。
+ **默认OOM处理**：
    - 在OOM处理函数`oom_malloc`和`oom_realloc`中，使用死循环不断尝试重新申请内存，直到成功或者系统无法继续分配内存。

<h3 id="40bd3d1e">二级空间配置器</h3>
处理小于128字节的内存块，使用了内存池技术。

+ **内存池和哈希桶**：维护一个大块内存池，使用16个空闲链表管理空闲块，每个链表负责管理特定大小（以8字节为单位）的内存块。
+ **内存分配**：申请内存时，根据请求的字节数找到对应的链表。如果链表中有空闲块，则直接从链表中取出。如果链表为空，则从内存池中获取新的内存块，并将其分割成适当大小的块添加到链表中。
+ **内存回收**：当内存释放时，将释放的块返回到对应的链表中。
+ **内存池补充**：如果内存池空间不足，会从系统堆中申请更多内存块来补充。

<h3 id="36fcf6c2">为什么SGI二级空间配置器会被历史淘汰?</h3>
现在为容器分配内存默认使用std::allocater而不是std::alloc，因为malloc的内存管理策略已经足以应对使用。如果使用std::alloc会多次一举。

<u>结合ptmalloc中fast bins说明和unsorted bin说明。</u>

glibc中使用4种空闲链表管理内存，fast bins，unsorted bin,small bins,large bins。

每种bin管理不同大小的空闲块，并且有不同的策略，这样二级空间配置器策略不再具有优势，内存分配器（如ptmalloc）提供了比SGI二级空间配置器更好的内存管理策略，在分配小块内存时，ptmalloc使用10个fast bin，分配和释放的内存块的时候不会分割也不会合并，这样提高了分配小块内存的效率，所以没有必要再多此一举，小于128字节的时候使用二级配置器，这样反而会有更大的开销。

<h3 id="f528758c">内存基本处理工具</h3>
<h6 id="__uninitialized_copy">__uninitialized_copy</h6>
从一个输入迭代器范围`[__first,_last) `复制构造元素到另一个由`_result`指向的未初始化区域。

它首先检查元素类型是否为POD类型 （标量类型或者传统的C结构类型)；

如果是，则调用STL的`copy`操作；

如果不是，则通过逐个调用构造函数来完成复制。

对于特化的字符串类型，直接使用memmove()函数进行拷贝。

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742810341187-6883e6a0-17c4-4561-900b-94b46c2df6b5.png)

**为什么char*直接使用memmove进行拷贝效率更高？**

```cpp
void *memmove(void *dest, const void *src, size_t n);
```

src指针指向的数据拷贝到dest指针指向的位置。

`memmove`直接处理原始内存，而不需要调用任何构造或析构函数。对于POD类型（如`char`），避免了额外的函数调用开销。

<h6 id="uninitialized_fill">uninitialized_fill</h6>
![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742810342054-ac057f19-a6eb-4c3e-906b-ce092fe214b6.png)

`uninitialized_fill` 函数用于将一个未初始化的内存区域填充为特定值。它接受三个参数：开始迭代器、结束迭代器和要填充的值。该函数为每个未初始化的位置调用构造函数，放置指定的值。

类型萃取后，对于POD类型，`uninitialized_fill` 可以STL中的`fill`函数来填充；

非POD类型则需要逐一调用构造函数。

<h6 id="uninitialized_fill_n">uninitialized_fill_n</h6>
![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742810341481-606aa688-9dac-4e52-8b32-5f8337185283.png)

与uninitialized_fill类似。

<h4 id="ARZ5z"></h4>
<h3 id="Ydoq3"><font style="background-color:#FBDE28;">再读源码</font></h3>
`**<font style="background-color:rgb(252, 252, 252);">__gnu_cxx::__alloc_traits</font>**`

`__gnu_cxx::__alloc_traits` 是 GNU C++ 标准库中用于封装内存分配器特性的模板类，属于 STL（标准模板库）的底层实现机制。它的主要作用是对标准分配器 `std::allocator` 或自定义分配器进行特性萃取，提供统一的接口以访问分配器的内存管理功能（如分配、释放、构造、析构等）。

**核心功能与特性**

1. **封装与继承**  
`__alloc_traits` 继承自 `std::allocator_traits`，并对其功能进行扩展和适配。例如，它通过 `using` 关键字继承了 `allocate`、`deallocate`、`construct` 等核心函数，并针对自定义指针类型（如非标准指针）提供了重载支持。
2. **类型萃取与重绑定**  
通过 `rebind` 结构体，`__alloc_traits` 可以将当前分配器类型（如 `allocator<T>`）转换为其他类型（如 `allocator<U>`），这在容器实现中用于动态调整内存分配器的类型。例如，`vector` 在重新分配内存时会通过 `rebind` 获取新的分配器类型。
3. **兼容性与扩展性**  
它兼容 C++11 及更高版本的标准，支持 `std::construct_at` 和 `std::destroy_at` 等新特性，同时保留对旧版本 C++ 的兼容性（如通过 `placement new` 实现构造和析构）。



在 STL 容器（如 `vector`、`list`）中，`__alloc_traits` 被用于管理内存分配和对象生命周期。例如，`vector` 的底层实现通过 `_Tp_alloc_type`（即 `__alloc_traits` 的别名）调用分配器的 `allocate` 和 `construct` 函数完成元素的存储。用户自定义容器或分配器时，也可通过继承或组合 `__alloc_traits` 实现高效的内存管理。



```cpp
// 自定义分配器示例
template<typename T>
class MyAllocator {
public:
    using value_type = T;
    T* allocate(std::size_t n) { /* 分配逻辑 */ }
    void deallocate(T* p, std::size_t n) { /* 释放逻辑 */ }
};

// 使用 __alloc_traits 萃取特性
using traits = __gnu_cxx::__alloc_traits<MyAllocator<int>>;
auto* ptr = traits::allocate(allocator_instance, 10); // 分配内存
traits::construct(allocator_instance, ptr, 42);      // 构造对象
traits::destroy(allocator_instance, ptr);          // 析构对象
traits::deallocate(allocator_instance, ptr, 10);   // 释放内存
```



`__gnu_cxx::__alloc_traits` 是 GNU C++ 标准库中实现内存管理的关键组件，通过封装分配器特性、支持类型重绑定和兼容多版本标准，为 STL 容器及自定义内存管理提供了高效、灵活的底层支持。

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1743339108966-cf88110a-0279-4dda-9db9-1244c2719a55.png)



```cpp
   using __allocator_base = __gnu_cxx::malloc_allocator<_Tp>;
```







<h4 id="LHjbQ">malloc_allocator.hh</h4>
这个头文件主要封装了malloc和free，加上一些必要的辅助函数。

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1743340007905-028e336d-1774-47a0-9edf-9cf727c31a2d.png)

1. 类型定义：
+ value_type: 存储的数据类型
+ size_type: 用于表示大小的无符号整数类型
+ difference_type: 用于表示指针差值的有符号整数类型
1. C++17及之前的类型定义：
+ pointer: T类型的指针
+ const_pointer: T类型的常量指针
+ reference: T类型的引用
+ const_reference: T类型的常量引用

内存管理函数：

allocate(size_type __n, const void* = 0):

+ 分配能够存储n个T类型对象的内存
+ 使用malloc实现
+ 处理内存对齐要求
+ 检查溢出和分配失败情况

deallocate(T* __p, size_type):

+ 释放之前分配的内存
+ 使用free实现
+ 不使用size参数（C++标准要求提供但不使用）

辅助函数：

+ address(reference __x): 返回对象的地址
+ max_size(): 返回最大可分配的对象数量
+ _M_max_size(): 内部实现，计算最大可分配大小
+ construct(pointer __p, const T& __val): 在已分配内存上构造对象
+ destroy(pointer __p): 析构已构造的对象

C++11特性：

+ propagate_on_container_move_assignment:
+ 类型特征，表示容器移动赋值时分配器行为
+ 设置为true_type表示支持移动语义

比较操作符：

+ operator==: 判断两个分配器是否等价(总是返回true)。operator!=: 判断两个分配器是否不等价(总是返回false)

<h5 id="Hx0tk">propagate_on_container_move_assignment</h5>
[https://github.com/gcc-mirror/gcc/blob/e0886d8ad4c51919c349d0b31f2bec3acbc79e14/libstdc%2B%2B-v3/include/bits/alloc_traits.h#L101](https://github.com/gcc-mirror/gcc/blob/e0886d8ad4c51919c349d0b31f2bec3acbc79e14/libstdc%2B%2B-v3/include/bits/alloc_traits.h#L101)

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1743341189528-69f8dce1-60b1-465c-b3b1-b65fe3cbde16.png)







<h4 id="HHjuU">allocator_traits.hh</h4>
![](https://cdn.nlark.com/yuque/0/2025/svg/45533403/1743342878516-d986697c-67c5-432a-9d8c-5c009fd009ab.svg)



`__allocator_traits_base`（基础分配器特征类）：

+ 这是一个内部实现类，提供基础的类型定义和检测功能
+ 主要包含各种类型别名的定义，用于检测分配器是否具有特定的类型成员

`allocator_traits<_Alloc>`（通用分配器特征类）：

+ 这是主要的分配器特征类，为所有分配器提供统一的接口
+ 重要成员类型：
    - `allocator_type`: 分配器类型
    - `value_type`: 被分配的对象类型
    - `pointer`: 指向对象的指针类型
    - `const_pointer`: 指向常量对象的指针类型
    - `void_pointer`: void指针类型
    - `const_void_pointer`: 常量void指针类型

重要成员函数：

+ `allocate`: 分配内存
+ `deallocate`: 释放内存
+ `construct`: 在已分配内存上构造对象
+ `destroy`: 销毁对象
+ `max_size`: 获取最大可分配大小
+ `select_on_container_copy_construction`: 容器复制时的分配器选择策略

`allocator_traits<allocator<_Tp>>`（标准分配器特化版本）：

+ 针对标准分配器的特化版本
+ 提供更简单和优化的实现
+ 所有指针类型都使用原生指针
+ 定义了固定的传播策略（如移动赋值时传播分配器）

`allocator_traits<allocator<void>>`（void特化版本）：

+ 专门用于void类型的特化版本
+ 不提供实际的内存分配功能
+ 主要用于类型转换和类型萃取

这些类之间的关系：

1. `__allocator_traits_base` 是基础类，提供共同的功能
2. `allocator_traits` 继承自 `__allocator_traits_base`，提供主要实现
3. 特化版本（`allocator<_Tp>` 和 `allocator<void>`）是 `allocator_traits` 的特殊形式



成员函数功能：

1. `allocate`: 分配指定大小的内存块
2. `deallocate`: 释放之前分配的内存
3. `construct`: 在已分配内存上构造对象
4. `destroy`: 调用对象的析构函数
5. `max_size`: 返回理论上可分配的最大大小
6. `select_on_container_copy_construction`: 决定容器复制时如何处理分配器



头文件实现了C++标准库中分配器特征的核心功能，为不同类型的分配器提供了统一的接口，使得容器和算法可以通过这个统一的接口来使用不同的分配器。



让我详细解释这些设计的原因和考虑：

1. **为什么要分成两个类？**
+ `__allocator_traits_base` 是一个实现细节类（注意双下划线前缀），主要负责类型检测和基础设施
+ `allocator_traits` 是面向用户的公共接口类，提供标准化的分配器接口

这种分离设计有以下好处：

1. **关注点分离**：
    - `__allocator_traits_base`专注于类型检测和底层实现
    - `allocator_traits`专注于提供清晰的用户接口
2. **实现细节隐藏**：
    - 用户不需要也不应该直接使用`__allocator_traits_base`
    - 将实现细节封装在基类中，降低代码复杂度
3. **为什么需要这些成员？**

```cpp
// __allocator_traits_base中的成员
template<typename _Tp> using __pointer = typename _Tp::pointer;
template<typename _Tp> using __c_pointer = typename _Tp::const_pointer;
template<typename _Tp> using __v_pointer = typename _Tp::void_pointer;
template<typename _Tp> using __cv_pointer = typename _Tp::const_void_pointer;
```

这些类型别名用于：

1. **类型检测**：检查分配器是否提供了必要的类型定义
2. **类型萃取**：从分配器中提取相关类型信息
3. **类型转换**：在不同指针类型间进行安全转换

```cpp
// allocator_traits中的成员
using allocator_type = _Alloc;
using value_type = typename _Alloc::value_type;
using pointer = __detected_or_t<value_type*, __pointer, _Alloc>;
using const_pointer = /*...*/;
using void_pointer = /*...*/;
using const_void_pointer = /*...*/;
```

这些类型定义是必要的，因为：

1. **标准要求**：C++标准要求分配器必须提供这些类型定义
2. **类型安全**：提供类型安全的内存分配和对象构造
3. **泛型编程**：支持容器和算法的泛型编程
4. **不定义可以吗？**

不可以，因为：

1. **标准符合性**：这些类型定义是C++标准要求的
2. **类型安全**：没有这些定义将失去类型安全保证
3. **功能完整性**：容器和算法需要这些类型信息
4. **这些成员占用内存空间吗？**

不占用，因为：

1. **类型别名**：这些都是类型定义，在编译时处理，不占运行时内存
2. **静态成员**：类中的函数都是静态的，不占实例内存
3. **空基类优化**：由于没有非静态数据成员，可以利用空基类优化

示例说明：

```cpp
// 验证大小
static_assert(sizeof(allocator_traits<some_allocator>) == 1, 
              "allocator_traits is empty");
```

5. **设计的精妙之处**：

```cpp
// 类型检测和回退机制
using pointer = __detected_or_t<value_type*, __pointer, _Alloc>;
```

这行代码展示了设计的智慧：

1. **优先使用自定义**：如果分配器定义了pointer类型，就使用它
2. **默认回退**：否则回退到value_type*
3. **编译时多态**：通过模板和类型特征实现编译时多态

总结：

1. 这种设计提供了：
    - 类型安全
    - 标准符合性
    - 实现灵活性
    - 零运行时开销
    - 良好的代码组织
2. 不能省略这些定义，因为：
    - 它们是标准要求的一部分
    - 提供了必要的类型信息
    - 支持泛型编程
    - 保证类型安全
3. 这种设计是现代C++的典范：
    - 零开销抽象
    - 类型安全
    - 编译时多态
    - 良好的代码组织













<h4 id="AZr0O">深入理解C++标准模板库（STL）的内存管理：探究分配器组件</h4>
这个框架的一个基本方面是分配器的概念，分配器负责管理容器所使用内存的分配与释放。在这个内存管理系统中，关键组件包括`std::allocator`、`std::allocator_traits`和`__alloc_traits`。

**解析**`**std::allocator_traits**`

`std::allocator_traits`类模板是用于访问分配器类型各种属性的标准化接口。



充当一个中介，只要`allocator_traits`能够提供必要的信息和功能，就可以让标准库组件（如容器）与任何分配器类型进行交互。



这种设计提供了高度的灵活性，使得开发者能够将自定义分配器集成到STL容器中，而无需对容器的实现进行重大修改。(为什么？这个流程是上面？和iterator_traits是一样的吗？)



标准容器仅通过`allocator_traits`这一层来访问分配器，即使是接口非常简单的分配器也能被有效利用。



用户自定义的`std::allocator_traits`被认为是格式错误的。这样的特化通常没有什么益处，因为它们通常会复制标准行为。后续标准中的这一限制简化了分配器模型，促进了不同编译器实现之间的一致行为。



`std::allocator_traits`定义了几个关键的成员类型，类型提供了与关联分配器（`Alloc`）相关的信息。



这些成员类型包括`allocator_type`（即`Alloc`本身）、`value_type`、`pointer`、`const_pointer`、`size_type`和`difference_type`。



这些类型通常从分配器本身派生（例如，`Alloc::value_type`、`Alloc::pointer`）。



此外，`std::allocator_traits`提供了一组<font style="background-color:#FBDE28;">静态成员函数</font>，为常见的分配器操作提供默认实现。



这些函数包括`allocate`、`deallocate`、`construct`、`destroy`、`max_size`和`select_on_container_copy_construction`。

```cpp
//
_GLIBCXX_NODISCARD static _GLIBCXX20_CONSTEXPR pointer
      allocate(_Alloc& __a, size_type __n, const_void_pointer __hint)
      {
  if constexpr (__has_allocate_hint<_Alloc, size_type, const_void_pointer>)
    return __a.allocate(__n, __hint);
  else
    return __a.allocate(__n);
      }

      /**
       *  @brief  Deallocate memory.
       *  @param  __a  An allocator.
       *  @param  __p  Pointer to the memory to deallocate.
       *  @param  __n  The number of objects space was allocated for.
       *  Calls <tt> a.deallocate(p, n) </tt>
      */
      static _GLIBCXX20_CONSTEXPR void
      deallocate(_Alloc& __a, pointer __p, size_type __n)
      { __a.deallocate(__p, __n); }
```



这些静态函数通常会委托给作为参数传递的分配器对象的相应成员函数。例如，调用`allocator_traits<A>::allocate(a, n)`通常会导致调用`a.allocate(n)`。值得注意的是，在分配器类型本身没有提供`construct`和`destroy`等函数的情况下，`allocator_traits`会提供默认实现。例如，默认的`construct`函数会退而使用placement new。这些默认实现极大地减轻了编写自定义分配器的负担，因为开发者主要需要专注于实现内存分配和释放的核心逻辑。这种设计也有助于向前兼容性。未来的C++标准可能会为分配器引入新的要求，但这些要求可能由编译器供应商对`allocator_traits`的更新来处理，从而最大限度地减少对现有自定义分配器进行更改的需求。



**详细分析**`**std::allocator**`**：默认分配器**

`std::allocator`的源代码揭示了其对核心内存管理函数的实现。

`allocate`方法使用全局的`::operator new`来分配一块请求大小的原始内存。

相反，`deallocate`方法使用全局的`::operator delete`来释放先前分配的内存。

对于对象构造，`std::allocator`的`construct`方法使用placement new（`::new((void *)__p) _Up(std::forward<_Args>(__args)...)`）在特定的内存位置初始化一个对象。最后，`destroy`方法直接调用所提供指针指向的对象的析构函数（`__p->~_Up()`）。



`std::allocator`还定义了一个名为`rebind`的嵌套模板结构体。这种机制允许从现有的分配器（`allocator<_Tp>`）创建一个用于不同类型（`_Tp1`）的分配器。在`rebind`结构体中，`other`类型定义解析为`allocator<_Tp1>`。这种能力对于STL容器至关重要，因为在内部操作（如移动元素或管理内部数据结构）期间，STL容器通常需要为与元素类型不同的类型分配内存。`std::allocator`中对这种`rebind`机制的显式定义，确保了默认分配器可以根据STL容器的需要适配为与各种类型一起工作。



**协同操作：这些组件如何协同工作**

当STL容器需要为特定类型的元素分配内存时，它通常通过`std::allocator_traits`与其关联的分配器进行交互。



然后调用`std::allocator_traits`的`allocate`方法，该方法又会调用底层分配器对象的`allocate`方法。对于默认分配器`std::allocator<T>`，这个`allocate`调用会转发给全局的`::operator new`。

当需要在分配的内存中构造一个对象时，容器使用`std::allocator_traits<Alloc>::construct`，然后使用placement new，要么调用分配器自己的`construct`方法，要么使用`allocator_traits`提供的默认实现。类似地，释放和销毁过程也通过`std::allocator_traits`进行中介，它会调用分配器的`deallocate`和`destroy`方法（或它们的默认实现）。



**自定义分配器与**`**allocator_traits**`**的使用**





C++中如何自定义一个分配器：

以下是一个完整的自定义分配器示例，结合`std::allocator_traits`实现最小化接口：

必须定义    value_type，using pointer，using const_pointer，using size_type。

```cpp
#include <iostream>
#include <vector>
#include <memory>

// 自定义分配器类
template <typename T>
class my_allocator {
public:
    // 必须定义的嵌套类型
    using value_type = T;
    using pointer = T*;
    using const_pointer = const T*;
    using size_type = std::size_t;
    // using difference_type = std::ptrdiff_t;
    // using propagate_on_container_move_assignment = std::true_type;
    // 构造/析构函数
    my_allocator() noexcept = default;
    template <typename U>
    my_allocator(const my_allocator<U>&) noexcept {}
    // 核心内存管理方法
    pointer allocate(size_type n) {
        if (n == 0) return nullptr;
        std::cout << "Allocate " << n << " objects of size " << sizeof(T) << "\n";
        return static_cast<pointer>(::operator new(n * sizeof(T)));
    }
    void deallocate(pointer p, size_type n) noexcept {
        if (p) {
            std::cout << "Deallocate " << n << " objects\n";
            ::operator delete(p);
        }
    }
    // rebind机制
    template <typename U>
    struct rebind {
        using other = my_allocator<U>;
    };
};

//测试自定义分配器
int main() {
    // 使用自定义分配器的vector
    std::vector<int, my_allocator<int>> vec;
    // 分配内存并构造对象
    vec.push_back(42);
    vec.push_back(100);
    // 显示内容
    for (auto& x : vec) {
        std::cout << x << " ";
    }
    std::cout << "\n";
    // 内存释放
    vec.clear();
    return 0;
}
```

编译运行输出示例：

```bash
Allocate 1 objects of size 4
    Allocate 2 objects of size 4
    Deallocate 1 objects
42 100 
    Deallocate 2 objects
=== 代码执行成功 ===
```



**1.必须定义的嵌套类型**：

+ `value_type`: 分配器管理的类型
+ `pointer`: 原始指针类型
+ `size_type`: 内存大小类型
+ `propagate_on_container_move_assignment`: 可选扩展，控制移动赋值时的分配器传播

**2.**`allocate()`: 使用全局`operator new`分配原始内存。`deallocate()`: 使用全局`operator delete`释放内存。



`std::allocator_traits`自动提供：

+ `construct()`: 使用placement new初始化对象
+ `destroy()`: 调用对象析构函数
+ `max_size()`: 返回最大可分配大小
+ `select_on_container_copy_construction()`: 容器复制时的分配器选择策略

my_allocator 仅需实现核心的`allocate`/`deallocate`方法

+ 其他方法`std::allocator_traits`自动提供
+ 完全兼容STL容器（这里使用`std::vector`)
+ 自动处理对象的构造/析构生命周期

<h2 id="f2cabe01">第三章 迭代器</h2>
⭐关联知识点：[智能指针](https://kdocs.cn/l/ckBesnvyv9HT?linkname=j9KozDAsgR)、迭代器模式

<h4 id="Hub1D">为什么要使用迭代器？不使用行不行？</h4>
**<font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(252, 252, 252);">引入迭代器是为了解耦算法与容器</font>**

<font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(252, 252, 252);">如果没有迭代器，算法通常需要针对不同容器类型进行定制化实现（STL算法是指模版函数）。例如，遍历数组和链表需要不同的代码逻辑。而STL引入迭代器抽象了容器的内部结构，使得算法可以通过统一的接口操作任意容器，无需关心容器的具体实现细节</font>

**<font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(252, 252, 252);">所以由此得出迭代器的作用</font>**<font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(252, 252, 252);">：迭代器是一种抽象的“指针”，可以用来遍历和访问容器的元素，所以迭代器最重要的工作就是重载（如</font>`<font style="background-color:rgb(252, 252, 252);">*</font>`<font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(252, 252, 252);">解引用、->访问,</font>`<font style="background-color:rgb(252, 252, 252);">++</font>`<font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(252, 252, 252);">递增 --递减），使得算法可以通过迭代器访问容器元素，而无需直接依赖容器的成员函数或内部结构，</font><font style="background-color:#fbf5b3;">无需关系容器的实现细节</font><font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(252, 252, 252);">。</font>

:::info
代器可以看成是一种智能指针，封装了原生指针(如vector容器的迭代器就是封装了原始指针)或者节点的指针（list，map），迭代器是一种行为类似指针的对象，用来用来访问和遍历容器的元素。unique_ptr,shared_ptr也是封装了原生的指针，但是重点在析构函数中释放对象的内存。

:::

<h4 id="hsF9Y">为什么迭代器有关联类型？</h4>
<font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(252, 252, 252);">迭代器的关联类型是为了描述迭代器所操作的数据类型。比如可以在算法中声明一个迭代器所指类型的变量，这时就需要知道迭代器关联对象的类型是什么。</font>

<font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(252, 252, 252);">STL中定义了迭代器关联的对象类型有5个：value_type(值类型),refernce_type(引用类型)，pointer_type(指针类型)，difference_type(表示两个迭代器之间的距离)，iterator_type(迭代器类型)，迭代器类型又分为输入迭代器，输出迭代器，前向迭代器，双向迭代器，随机迭代器。</font>

<font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(252, 252, 252);">输入迭代器只能读取元素，不能修改。每个元素只能读取一次。</font>输出迭代器<font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(252, 252, 252);">只能写入元素，不能读取。每个位置只能写入一次。</font>

<h4 id="3a85c582">为什么要有迭代器萃取类?</h4>


待整理。



迭代器萃取类来获取迭代器特定类型信息，如值类型(value type)、差值类型(difference type)、指针类型(pointer)、引用类型(reference)和迭代器类别(iterator category)。

通过获取这些信息可以编写能处理各种不同迭代器的通用代码。



例如对char*类型的iterator执行copy操作，那么copy函数就可以直接使用memmove来完成操作，而不是遍历容器调用构造函数，这样可以提高算法的效率。

迭代器萃取的核心是`iterator_traits`结构体模板，它可以识别并使用迭代器内嵌的类型定义。

对于原生指针和指向常量的指针，定义`iterator_traits`的特化版本来确保正确性。



<h4 id="9da1dcbf">什么是萃取技术？</h4>
3.7节

💡

关联知识点：静态多态，函数重载。

萃取技术，就是当函数，类这种一些封装的通用算法中的某些部分会因为类型不同而导致处理的逻辑不同，然而我们又不想因为类型的差异修改算法本身的封装，所以采用萃取机制来解决这一问题。因此在STL中为了**提供通用的操作而又不想损失效率**就采用traits编程技巧。

原文链接：[https://blog.csdn.net/xixihaha331/article/details/51362327](https://blog.csdn.net/xixihaha331/article/details/51362327)

在`type_traits`中，主要关注以下几个类型特性：

是否具有平凡的默认构造函数，拷贝构造函数，赋值运算符，析构函数（平凡的XX函数是指编译器自动生成的函数）。如果是的话返回一个true_type，不是的话返回一个false_type。由于`true_type`和`false_type`是不同的类型而不是bool值，所以可以他们进行函数重载，以便根据类型的特性选择不同的函数实现。

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742810341433-20f69a2f-d29d-42e1-a8ec-83a660aa8a0e.png)

比如如果我们准备对一个元素型别未知的数组进行拷贝操作，如果我们知道这个元素型别是否有一个拷贝构造函数,以便我们决定是否使用快速的memcpy(),还是使用慢速的拷贝构造函数。

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1743349673604-06b248b2-a64f-4cea-a261-546f608ec886.png)



![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1743349900063-e78671eb-3c57-4666-8711-4b3afa952ea5.png)









<h2 id="e99d0560">第四章 序列型容器</h2>
<h3 id="vector">vector</h3>
💡

关联知识点：struct末尾char[1]动态扩容；array

阅读材料：[https://blog.csdn.net/qq_21438461/article/details/131506807?ops_request_misc=&request_id=&biz_id=102&utm_term=vector%20resize%E5%92%8Creserve&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-1-131506807.142^v100^pc_search_result_base4&spm=1018.2226.3001.4187](https://blog.csdn.net/qq_21438461/article/details/131506807?ops_request_misc=&request_id=&biz_id=102&utm_term=vector%20resize%E5%92%8Creserve&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-1-131506807.142^v100^pc_search_result_base4&spm=1018.2226.3001.4187)（写的很详细）

vector是动态数组，插入元素后如果内存不够，vector内部机制会自行扩充空间以容纳新的元素。

<h4 id="47aeb1a0">vector迭代器</h4>
vector支持随机存取，所以vector的迭代器类型是随机迭代器，所以使用原生指针(T *)就可以实现迭代器的所有功能。

如vector<int>中迭代器就是int *。

<h4 id="0d72b852">**vector数据结构**</h4>
vector采用线性连续空间，start和finish指向数组的头部和尾部(前开后闭)，并以迭代器end_of_storage指向已分配空间的尾端。

vector大小是[start,finish)，容量是[start，end_of_storage)。

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742810343172-8d1ed1f7-5f63-4f5d-b613-55311333b318.png)

<h4 id="b7d2113d">vector构造函数</h4>
```cpp
vector():start(0),finish(0),end_of_storage(0){}
vector(size_type n, const T& value) {fill_initialize(n,value);}
vector(int n, const T& value){fill_initialize(n,value);}
vector(long n, const T& value){fill_initialize(n,value);}
```

```cpp
void fill_initialize(size_type n, const T& value) {
    start = allocate_and_fill(n, value);
    finish = start + n;
    end_of_storage = finish;
}
```

所以vector初始化使用的是allocate_and_fill(n，value)。

```cpp
iterator allocate_and_fill(size_type n, const T& x){
    iterator result = data_allocator::allocate(n);
    uninitialized_fill_n(result, n, x);
    return result;
}
```

使用空间配置器分配内存，然后填充未初始化的内存。

在填充位初始化的内存的时候，使用类型萃取，如果vector中存的是POD类型(标量或者传统的C struct结构体)，就使用STL 中的fill算法。对于非POD类型，遍历容器然后调用构造函数。uninitialized_fill_n见第二章。类型萃取后，对于POD类型，`uninitialized_fill` 可以STL中的`fill`函数来填充，非POD类型则需要逐一调用构造函数。

<h4 id="0ef75086">vector中insert</h4>
<h4 id="cb197cd6">vector中迭代器失效</h4>
insert迭代器失效分为两类：

+ 扩容导致野指针
+ 迭代器<font style="background-color:#fbf5b3;">指向的位置意义发生改变</font>

阅读材料：[https://blog.csdn.net/weixin_45031801/article/details/138410868?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522172403171016800207010981%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=172403171016800207010981&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-3-138410868-null-null.142^v100^pc_search_result_base4&utm_term=%E8%BF%AD%E4%BB%A3%E5%99%A8%E5%A4%B1%E6%95%88&spm=1018.2226.3001.4187](https://blog.csdn.net/weixin_45031801/article/details/138410868?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522172403171016800207010981%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=172403171016800207010981&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-3-138410868-null-null.142^v100^pc_search_result_base4&utm_term=%E8%BF%AD%E4%BB%A3%E5%99%A8%E5%A4%B1%E6%95%88&spm=1018.2226.3001.4187)

1. **插入元素**

`vector` 插入元素时，可能会导致内部内存扩容，并释放原来的内存，只要是释放原来的内存就会。

重新分配会导致所有迭代器失效，包括对 `vector` 的所有元素的迭代器。

如果插入不导致扩容，会导致<font style="background-color:#fbf5b3;">插入位置后面的迭代器指向位置的意义发生改变</font>。

```cpp
std::vector<int> vec = {1, 2, 3};
cout<<vec.capacity();//3
auto it = vec.begin(); //it指向原来内存的第一个整数
vec.push_back(4);//插入后,vector内存扩容，it迭代器失效
cout<<vec.capacity();//6
std::cout << "First element: " << *it << std::endl; //迭代器失效(未知数)
std::cout << "Last element: " << vec.back() << std::endl;
```

2. **删除元素**

删除元素同样会影响迭代器：

+ erase的失效都是<font style="background-color:#fbf5b3;">迭代器指向的位置意义发生改变</font>，或者<font style="background-color:#fbf5b3;">不在有效访问数据的有效范围</font>内
+ erase一般不会使用缩容的方案，那么也就导致erase的失效一般<font style="background-color:#fbf5b3;">不存在野指针失效</font>**。**

3. **调整大小和容量**

+ `**resize()**`：增加 `vector` 的大小可能会触发重新分配，从而使所有现有迭代器失效。减少 `vector` 的大小不会导致迭代器失效，但会丢弃多余的元素。
+ `**reserve()**`：reserve如果容量大于当前容量，会分配一块新的内存，指向原有内存的迭代器全部失效。

4. **其他操作**

+ `**clear()**`：清空 `vector` 会使所有迭代器失效，因为所有元素都被移除，迭代器不再指向有效的元素。

<h4 id="84881111">常用操作</h4>
<h5 id="insert">insert</h5>
调用insert_aux。如果空间够，就把插入位置的元素整体后移(copy_backward)，然后插入元素。

如果空间不够使用空间配置器分配原来2倍大小的空间(allocate)。

然后移动数据，(见第二章) 如果经过类型萃取后是POD类型，就调用STL的copy算法，否则遍历调用构造函数。释放原空间,更新迭代器。

allocate->uninitialized_copy -> destroy(begin(), end());deallocate();->start = new_start; finish = new_finish

```cpp
template <class T, class Alloc>
void insert_aux(iterator position, const T& x){
    if(finish != end_of_storage) // 还有备用空间
    { 
        //在备用空间起始处构造一个元素，并以vector最后一个元素值为其初值
        construct(finish, *(finish - 1));
        ++finish;
        T x_copy = x;
        copy_backward(position, finish - 2, finish - 1);
        *position = x_copy;
    }
    else
        //已无备用空间
    {
        const size_type old_size = size();
        const size_type len = old_size != 0 ? 2 * old_size : 1;
        //以上配置元素：如果大小为0，则配置1，如果大小不为0，则配置原来大小的两倍,前半段用来放置原数据，后半段准备用来放置新数据
        iterator new_start = data_allocator::allocate(len);  // 实际配置
        iterator new_finish = new_start;
        // 将内存重新配置
        try{
            //uninitialized_copy()的第一个参数指向输入端的起始位置
            //第二个参数指向输入端的结束位置（前闭后开的区间)
            //第三个参数指向输出端(欲初始化空间)的起始处
            //将原vector的安插点以前的内容拷贝到新vector
            new_finish = uninitialized_copy(start, position, new_start);
            //为新元素设定初值 x
            construct(new_finish, x);
            // 调整已使用迭代器的位置
            ++new_finish;
            // 将安插点以后的原内容也拷贝过来
            new_finish = uninitialized_copy(position, finish, new_finish);
        }
        catch(...){
            // 回滚操作
            destroy(new_start, new_finish);
            data_allocator::deallocate(new_start, len);
            throw;
        }
        // 析构并释放原vector
        destroy(begin(), end());
        deallocate();
        // 调整迭代器，指向新vector
        start = new_start;
        finish = new_finish;
        end_of_storage = new_start + len;
    }
}
```

`**push_back的底层实现**`

push_back(x) -> insert_aux(end(),x)；

`**emplace_back的底层实现**`

push_back() 向容器尾部添加元素时，首先会创建这个元素，然后再将这个元素拷贝或者移动到容器中（如果是拷贝的话，事后会自行销毁先前创建的这个元素）；而 emplace_back() 在实现时，则是直接在容器尾部创建这个元素，省去了拷贝或移动元素的过程。（看不懂!）

**reserve**

reserve操作是用于预分配Vector的容量。在Vector中存储大量的元素时，可以使用Reserve来预先分配足够的内存。这样可以避免在添加元素时频繁地重新分配内存，从而提高程序的性能。Reserve操作只是预分配内存，并不会改变Vector的大小。

如果预分配的内存容量大于当前容量，会分配一块新的内存，然后把当前元素拷贝到新的内存，然后释放原有的内存，这样会使原有的迭代器失效。如果预分配的内存容量小于当前容量，不会做任何事情。

reserve不会改变vector的size（size是由finsh和start距离决定的）

<h5 id="resize">resize</h5>
resize函数实质是改变vector中的元素个数。

如果大小比原有大小 大，添加元素(可能会触发扩容机制，导致迭代器失效)然后进行初始化（finish和start的距离变大）。

如果大小比原有大小 小， 删除末尾元素。

<h4 id="45c015ea">vector的动态扩容</h4>
（代码见insert_aux）

当备用空间不足时，vector做了以下的工作：

（1）重新分配空间：若原来的空间大小为0，则扩充空间为１，否则扩充为原来的两倍。

（2）移动数据，释放原空间，更新迭代器，当调用默认构造函数构造vector时，其空间大小为0；但当我们push_back一个元素到vector尾端时，vector就进行空间扩展，大小为１，以后每当备用空间用完了，就将空间大小扩展为原来的两倍。

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742810342063-dd34670b-f250-4804-a73c-6ea4d339f921.png)

动态增加大小，并不是在原空间之后接续新空间，（因为无法保证原空间之后上有可供分配的空间），而是以原大小的两倍来另外分配一块较大空间，因此，一旦空间重新分配，指向原vector的所有迭代器就会失。

STL容器的线程安全问题及解决办法：

[https://www.zhihu.com/question/29987589](https://www.zhihu.com/question/29987589)



<h4 id="YkhnQ"><font style="background-color:#FBDE28;">源码再阅读</font></h4>
[https://github.com/gcc-mirror/gcc/blob/master/libstdc%2B%2B-v3/include/std/vector](https://github.com/gcc-mirror/gcc/blob/master/libstdc%2B%2B-v3/include/std/vector)

stl_vector.hh



![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1743336606150-9f16b73e-6b62-4f59-8028-f111c60f4ca1.png)

1. vector 继承自 _Vector_base

2. _Vector_base 包含 _Vector_impl

3. _Vector_impl 继承自 _Vector_impl_data

4. _Guard_alloc 和 _Temporary_value 是辅助类，与 vector 和 _Vector_base 有依赖关系

主要类的职责：

-_Vector_impl_data ：存储向量的核心数据成员（起始指针、结束指针、存储结束指针）

-_Vector_impl ：提供内存管理的实现

-_Vector_base ：处理底层内存分配和释放

-vector ：提供完整的向量容器功能

-_Guard_alloc ：用于异常安全的资源管理

-_Temporary_value ：用于临时对象的管理

<h5 id="DdMEK">_Vector_base类</h5>


```cpp
typedef typename __gnu_cxx::__alloc_traits<_Alloc>::template
    rebind<_Tp>::other _Tp_alloc_type;
```

这段代码的作用是：将用户提供的分配器类型转换为能够分配目标类型的分配器。



```cpp
typedef typename __gnu_cxx::__alloc_traits<_Tp_alloc_type>::pointer pointer;
```

1. _Tp_alloc_type 是通过 _Alloc 分配器重新绑定到 _Tp 类型后的分配器类型

2. __gnu_cxx::__alloc_traits 是一个分配器特性类，用于获取分配器的相关类型信息

3. __alloc_traits<_Tp_alloc_type>::pointer 获取了 _Tp_alloc_type 分配器定义的指针类型

4. 最后， pointer 就是这个指针类型的别名

简单来说， pointer 就是指向 _Tp 类型元素的指针。例如，如果 _Tp 是 int ，那么 pointer 就是 int* 。

这个指针类型主要用于管理 vector 内部存储的元素，包括：

- 指向第一个元素的指针 ( _M_start )

- 指向最后一个元素之后的指针 ( _M_finish )

- 指向分配内存末尾的指针 ( _M_end_of_storage )

这种设计使得 vector 能够高效地管理动态数组的内存分配和元素访问。

为什么vector要采用这么多的层次结构？

STL vector的设计采用这种多层次的结构是经过深思熟虑的。

1. **为什么需要 _Vector_base?**

```cpp
template<typename _Tp, typename _Alloc>
struct _Vector_base {
    typedef typename __gnu_cxx::__alloc_traits<_Alloc>::template
    rebind<_Tp>::other _Tp_alloc_type;
    
    _Vector_impl _M_impl;
    
    // 处理内存分配的基础操作
    pointer _M_allocate(size_t __n) {
        return __n != 0 ? _Tr::allocate(_M_impl, __n) : pointer();
    }
    void _M_deallocate(pointer __p, size_t __n) {
        if (__p) _Tr::deallocate(_M_impl, __p, __n);
    }
}
```

+ _Vector_base 主要负责**内存管理**的职责
+ 实现了RAII模式，确保在异常发生时正确释放内存
+ 提供了基本的内存分配和释放操作
+ 将内存管理与vector的其他功能分离，实现了关注点分离
2. **为什么需要 _Vector_impl?**

```cpp
struct _Vector_impl 
    : public _Tp_alloc_type,    // 继承自分配器
      public _Vector_impl_data   // 继承自数据结构
{
    _Vector_impl() _GLIBCXX_NOEXCEPT : _Tp_alloc_type() { }
    _Vector_impl(_Tp_alloc_type const& __a) _GLIBCXX_NOEXCEPT 
        : _Tp_alloc_type(__a) { }
}
```

+ **空基类优化(Empty Base Optimization, EBO)**：通过继承而不是包含来实现
    - 如果分配器是空类(大多数标准分配器都是)，通过继承可以避免占用额外的内存
    - 如果使用成员变量而不是继承，即使是空的分配器也会占用至少1字节
+ **状态管理**：_Vector_impl_data 包含了vector的核心状态(_M_start, _M_finish, _M_end_of_storage)
+ **分配器访问**：通过继承分配器，可以直接访问分配器的功能，无需额外的间接层
3. **为什么需要 _Vector_impl_data?**

```cpp
struct _Vector_impl_data {
    pointer _M_start;           // 指向数组起始位置
    pointer _M_finish;          // 指向最后一个实际元素之后
    pointer _M_end_of_storage;  // 指向分配的内存末尾
}
```

+ 封装了vector的**核心数据成员**
+ 提供了数据的**移动和交换操作**
+ 将数据成员组织在一起，便于整体管理和操作
1. **异常安全**

```cpp
~_Vector_base() _GLIBCXX_NOEXCEPT {
    _M_deallocate(_M_impl._M_start,
                  _M_impl._M_end_of_storage - _M_impl._M_start);
}
```

+ _Vector_base 的析构函数确保内存总是被正确释放
+ 分离内存管理和元素管理，使得异常处理更清晰
2. **性能优化**

```cpp
// 通过EBO优化，避免空分配器占用空间
struct _Vector_impl : public _Tp_alloc_type, public _Vector_impl_data
```

+ 空基类优化减少内存占用
+ 直接访问分配器功能，没有额外的间接开销
3. **功能分离**
+ 内存管理(_Vector_base)
+ 分配器功能(_Vector_impl 继承 _Tp_alloc_type)
+ 数据存储(_Vector_impl_data)
+ 容器操作(vector)
4. **代码复用**

```cpp
// 其他容器也可以复用这些基础设施
template<typename _Tp, typename _Alloc = std::allocator<_Tp> >
class deque {
    // 可以复用相似的内存管理策略
};
```

这种设计虽然看起来复杂，但每一层都有其特定的职责：

1. _Vector_base 处理内存管理和RAII
2. _Vector_impl 通过继承实现空基类优化和分配器功能
3. _Vector_impl_data 管理核心数据成员
4. vector 类专注于提供容器的公共接口和操作

这种分层设计使得代码更容易维护、测试和优化，同时提供了最佳的性能和异常安全性。这也是为什么STL的实现虽然看起来复杂，但能够提供高效、安全和灵活的容器实现。

<h5 id="j1LM4">_Vector_base每个成员函数的功能是什么？</h5>
1. **_M_get_Tp_allocator()**

```cpp
_Tp_alloc_type& _M_get_Tp_allocator() _GLIBCXX_NOEXCEPT
{ return this->_M_impl; }
```

+ 获取类型化的分配器引用返回内部 _M_impl 中的分配器对象
+ 用于访问分配器的各种操作（如分配、构造等）
2. **get_allocator()**

```cpp
allocator_type get_allocator() const _GLIBCXX_NOEXCEPT
{ return allocator_type(_M_get_Tp_allocator()); }
```

+ 获取分配器的副本
+ 将内部分配器转换为用户可见的分配器类型
+ 这是一个公共接口，供vector用户使用
3. **_M_allocate(size_t)**

```cpp
pointer _M_allocate(size_t __n)
{
    return __n != 0 ? _Tr::allocate(_M_impl, __n) : pointer();
}
```

+ 分配指定数量的未初始化存储空间
+ 如果请求的大小为0，返回空指针
+ 内部使用分配器的 allocate 函数进行实际的内存分配
4. **_M_deallocate(pointer, size_t)**

```cpp
void _M_deallocate(pointer __p, size_t __n)
{
    if (__p)
        _Tr::deallocate(_M_impl, __p, __n);
}
```

+ 释放之前分配的存储空间
+ 只有当指针非空时才进行释放
+ 使用分配器的 deallocate 函数进行实际的内存释放
5. **_M_create_storage(size_t)**

```cpp
void _M_create_storage(size_t __n)
{
    this->_M_impl._M_start = this->_M_allocate(__n);
    this->_M_impl._M_finish = this->_M_impl._M_start;
    this->_M_impl._M_end_of_storage = this->_M_impl._M_start + __n;
}
```

+ 创建指定大小的存储空间
+ 初始化 _M_impl 中的三个指针
+ 设置好vector的初始状态（空但有容量）
6. **_S_relocate 相关函数**

```cpp
static pointer _S_do_relocate(pointer __first, pointer __last,
                             pointer __result, _Tp_alloc_type& __alloc,
                             true_type) noexcept
{
    return std::__relocate_a(__first, __last, __result, __alloc);
}
```

+ `_S_do_relocate`: 实现元素的重定位（移动）操作
+ `_S_relocate`: 根据类型特性选择适当的重定位策略
+ 用于vector需要重新分配内存时，高效地移动现有元素
1. **内存管理的封装**
+ 所有内存操作都通过分配器进行。提供了统一的内存分配和释放接口。确保内存操作的一致性和安全性。
2. ** 性能优化**
+ `_S_relocate` 系列函数提供了移动语义支持
+ 避免不必要的拷贝操作
+ 针对不同类型特性提供优化的实现
4. **资源管理**

```cpp
// RAII 模式的应用
~_Vector_base() {
    _M_deallocate(_M_impl._M_start,
                  _M_impl._M_end_of_storage - _M_impl._M_start);
}
```

函数共同构成了vector的内存管理基础设施：

+ 提供了安全的内存管理
+ 支持异常安全保证
+ 实现了高效的资源利用
+ 为vector的高层操作提供了可靠的基础。



**vector_base的内部布局是什么？**

1. 基本内存布局

```cpp
struct _Vector_base {
    struct _Vector_impl_data {
        pointer _M_start;
        pointer _M_finish;
        pointer _M_end_of_storage;
    };

    struct _Vector_impl 
        : public _Tp_alloc_type,    // 基类1
        public _Vector_impl_data     // 基类2
    {
        // 没有额外的成员变量
    };

    _Vector_impl _M_impl;  // _Vector_base的成员
};
```

**2. 实际的内存布局分析**

1. **_Vector_impl_data 的布局**

```cpp
struct _Vector_impl_data {
    pointer _M_start;            // 假设在64位系统上，指针是8字节
    pointer _M_finish;           // 8字节
    pointer _M_end_of_storage;   // 8字节
};  // 总大小 = 24字节
```

2. **_Tp_alloc_type（分配器）的布局**

```cpp
// 大多数标准分配器是空类
class _Tp_alloc_type {
    // 没有成员变量，只有成员函数
};  // 理论上大小是1字节，但由于EBO可能是0字节
```

3. **_Vector_impl 的布局**

```cpp
struct _Vector_impl 
    : public _Tp_alloc_type,    // 由于EBO，可能不占用空间
    public _Vector_impl_data     // 24字节
{
    // 没有额外的成员变量
};  // 总大小 = 24字节（在启用EBO的情况下）
```

3. 空基类优化（Empty Base Optimization, EBO）

```cpp
// 没有EBO时的理论布局
class WithoutEBO {
    _Tp_alloc_type alloc;  // 1字节（最小对象大小）
    pointer _M_start;      // 8字节
    pointer _M_finish;     // 8字节
    pointer _M_end_of_storage;  // 8字节
};  // 总大小 = 32字节（包含对齐）

// 有EBO时的实际布局
class WithEBO 
    : public _Tp_alloc_type  // 0字节（被优化掉）
{
    pointer _M_start;      // 8字节
    pointer _M_finish;     // 8字节
    pointer _M_end_of_storage;  // 8字节
};  // 总大小 = 24字节
```

<h3 id="WG8cR">5. 内存布局示意图</h3>
```cpp
_Vector_base::_Vector_impl 的内存布局:

    +------------------------+  <-- _M_impl起始地址
    |     _Tp_alloc_type    |      (由于EBO，实际上不占空间)
    +------------------------+
    |       _M_start        |      8字节
    +------------------------+
    |       _M_finish       |      8字节
    +------------------------+
    |   _M_end_of_storage   |      8字节
    +------------------------+
```

<h3 id="Rc92q">6. 重要说明</h3>
1. **不是连续的定义，而是继承关系**

```cpp
// 这些结构体的定义是嵌套的，但它们的实例不一定是连续的
struct _Vector_base {
    struct _Vector_impl_data { /*...*/ };  // 只是类型定义
    struct _Vector_impl { /*...*/ };       // 只是类型定义
    _Vector_impl _M_impl;                  // 实际的内存分配在这里
};
```

2. **实际使用的是单个实例**

```cpp
_Vector_base {
    _Vector_impl _M_impl;  // 只有这一个实例占用实际内存
};
```

3. **内存优化**

```cpp
// 通过继承和EBO实现内存优化
struct _Vector_impl 
    : public _Tp_alloc_type,    // 可能被优化掉
    public _Vector_impl_data     // 保持原有大小
{};
```

所以总结一下：

1. 这些结构体的定义是嵌套的，但这只是为了作用域和访问控制
2. 实际的内存只由 _Vector_base 中的 _M_impl 成员占用
3. 通过EBO，实际内存布局被优化得很紧凑
4. 指针成员在内存中是连续的，这保证了高效的访问

这种设计既保证了封装性，又实现了最优的内存布局。













<h3 id="list">List</h3>
📌 关联知识点：linux内核链表，可以用链表表示的对象(?)，空闲链表/glibc，....

<h4 id="c59865d5">list的数据结构</h4>
STL中list是使用环状双向链表实现的。

**结点结构定义**

前一个节点的指针，后一个节点的指针和数据域(使用模板，所以是通用链表)。

```cpp
template <classT>
struct__list_node {
typedefvoid* void_pointer;
void_pointer next;
void_pointer prev;
T data;
};
```

链表最后使用一个指针指向环形链表的空白节点，空白节点指向头节点，这样形成了一个环。

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742810342307-edf5ec2b-d194-49f7-939c-29ff30c17d6d.png)

```cpp
template<classT,classAlloc = alloc>{  
protected :  
typedef __list_node<T> list_node ;  
public  :  
typedef list_node* link_type ;  
protected :  
link_type node ; //只要一个指针，便可以表示整个环状双向链表  
...
};
```

node是指向list节点的一个指针，可以使用这个指针表示整个环状双向链表。

如果指针node指向置于尾端的一个空白节点，node就能符合stl对于前闭后开区间的要求，这样以下函数便能轻易完成。

```cpp
iterator begin(){return (link_type)((*node).next)); }
iterator end(){return node;}
boolempty()const{ return node->next == node;}
size_type size()const{
    size_type result = 0;
    distance(begin(), end(), result);//SGI里面的distance函数作用就是遍历链表return result;
}
reference front(){ return *begin(); }
reference back(){ return *(--end()); }
```

<h4 id="8643a824">list的迭代器</h4>
list是一个双向循环链表，元素在内存中不需要连续存放。

`vector`在内存中连续存放，所以可以使用原生指针作为迭代器。

但是，`list`不连续存储，所以不能使用普通指针作为迭代器，因为它需要特殊的迭代器，list的迭代器封装了List节点指针(list_node *)，它是一个**双向迭代器**，允许前移和后移操作。

**list迭代器失效（被删除节点的迭代器）**

`list`删除操作，只有指向被删除元素的迭代器会失效，其他迭代器仍然有效。插入不会使任何的迭代器失效。

```cpp
template<classT,classRef，classPtr>
struct_list_iterator{
typedef _list_iterator<T,T&,T*> iterator;
typedef _list_iterator<T,T&,T*> iterator;
typedef bidirectional_iterator_tag iterator_category;
typedef T value_type;
typedef Ptr pointer;
typedef Ref reference;
typedef _list_node<T>* link_type;
typedefsize_t size_type;
typedefptrdiff_t difference_type;
link_type node; // list的迭代器封装了List节点指针(list_node *).
_list_iterator(link_type x):node(x){}
_list_iterator(){}
_list_iterator(const iterator& x):node(x.node){}
booloperator==(const self& x) const {return node==x.node;}
booloperator!=(const self& x) const {return node!=x.node;}
reference operator*() const {return (*node).data;}
reference operator->() const {return &(operator*());}      
self& operator++(){
    node=(link_type)((*node).next);
    return *this;
}
self operator++(int){
    self tmp=*this;
    ++*this;
    return tmp;
}
self& operator--(){
    node=(link_type)((*node).prev);
    return *this;
}
self operator--(int){
    self tmp=*this;
    --*this;
    return tmp;
}
}
```

<h4 id="4785cdfd">list节点的构造和释放</h4>
使用std::allocator构造节点和释放节点的内存，然后初始化节点的指针和值。

然后对节点进行操作。

<h4 id="de86b91d">list操作</h4>
insert：类似双向链表的插入。

```cpp
terator insert(iterator position, const T& x){
    link_type tmp = create_node(x);   // 产生一个节点,调整双向指针,使tmp插入.
    tmp->next = position.node;
    tmp->prev = position.node->prev;
    (link_type(position.node->prev))->next = tmp;
    position.node->prev = tmp;
    return tmp;
}
```

erase：类似双向链表的删除。

```cpp
iterator erase(iterator position){  
    link_type next_node=link_type(position.node->next);  
    link_type prev_node=link_type(position.node->prev_nodext);  
    prev_node->next=next_node;  
    next_node->prev=prev_node;  
    destroy_node(position.node); //会删除节点的内存不会内存泄漏
    returniterator(next_node);  
}
```

push_front(),push_back(),pop_front(),pop_back() 在 insert 和 erase 的基础上实现。



**slice操作**

用于高效移动链表元素或子链表的成员函数，其核心功能是将一个 `std::list` 中的元素或子链表移动到另一个 `std::list` 的指定位置，且操作过程中**不会复制或移动元素本身**，仅通过调整指针实现，因此时间复杂度为常数级别 O(1)。

1. **移动整个链表**  
将另一个链表的所有元素插入到当前链表的指定位置之前。  

```cpp
std::list<int> list1 = {10, 20, 30};
std::list<int> list2 = {40, 50, 60};
list1.splice(list1.begin(), list2); // list1变为{40,50,60,10,20,30}，list2为空
```

2. **移动单个元素**  
将另一个链表中的指定元素插入到当前链表的指定位置之前。  

```cpp
std::list<int> list1 = {10, 20, 30};
std::list<int> list2 = {40, 50, 60};
auto it = list2.begin();
++it; // 指向50
list1.splice(list1.begin(), list2, it); // list1变为{50,10,20,30}，list2变为{40,60}
```

3. **移动元素范围**  
将另一个链表中指定范围的元素插入到当前链表的指定位置之前。  

```cpp
std::list<int> list1 = {10, 20, 30};
std::list<int> list2 = {40, 50, 60, 70};
auto first = list2.begin();
auto last = list2.end();
--last; // 指向70
list1.splice(list1.begin(), list2, first, last); // list1变为{40,50,60,10,20,30}，list2变为{70}
```

• **高效性**：由于仅调整指针，`splice` 的时间复杂度为 O(1)，适用于频繁插入/删除的场景。  
• **迭代器失效**：被移动元素的迭代器在原链表中失效，但指向目标位置的迭代器（如 `pos`）仍有效。  
• **合并有序链表**：常与 `merge` 函数配合使用，实现有序链表的合并。

（链表排序算法题需要整理）

**动态重组链表**：在复杂逻辑中灵活调整链表结构。

<h3 id="deque">deque</h3>
🔔

关联知识点：二级指针，空基类优化

阅读材料：[https://zhuanlan.zhihu.com/p/644990261](https://zhuanlan.zhihu.com/p/644990261)

deque在常数时间在头尾两端分别做元素的插入和删除。虽然vector也可以在首端进行元素的插入和删除（利用insert和erase），但效率差（涉及到整个数组的移动)，无法被接受。

deque是由<font style="background-color:#fbf5b3;">一段一段</font>连续内存空间组成。与vector 容器采用连续的线性空间不同，deque容器存储数据的空间是由一段一段等长的连续空间构成，各段空间之间不一定连续。

使用一个指针数组(<font style="background-color:#fbf5b3;">中控器)map</font>指向一段一段连续的线性空间(缓冲区，默认大小512字节)。

<font style="background-color:#fbf5b3;">如果当前段空间用完了，就添加一个新的空间并将它链接在map的头部或尾部。</font>

如果map的空间用完了，就配置一段空间给新map使用，并把原来的map元素拷贝过去。

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742810342454-98fef54a-9b24-43f8-b682-be9d5e889028.png)

在初始化 Map 内存时，根据所需的节点数量计算出映射表的大小，并为其分配内存，预留额外的空间，以便在未来插入元素时不需要重新分配映射表。

<h4 id="b8fd10ec">deque数据结构</h4>
![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742810342789-722792de-b44d-459e-a877-fec4ffd683e7.png)

<h4 id="3d3b3dbd">deque迭代器</h4>
由于 deque 容器底层将序列中的元素分别存储到了不同段的连续空间中，迭代器在遍历 deque 容器时，必须能够确认各个连续空间在数组中的位置，必须能够判断自己是否已经处于空间的边缘位置，前进后退需要考虑是否要跳跃到上一个或下一个连续空间中。

deque的迭代器内部包含 4 个指针；cur：指向当前正在遍历的元素。

first：指向当前连续空间的首地址；last：指向当前连续空间的末尾地址。

node：二级指针用于指向数组中存储的指向当前连续空间的指针。

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742810342818-7f184a03-f0a4-4133-b4c1-47bac92abfee.png)

虽然deque容器的迭代器也支持随机访问，但是访问元素的速度要低于vector。

指针数组 (map)： 是一个指针数组，维护指向缓冲区的指针。map 左右两边预留有空间，允许前后扩展。迭代器 (start, finish)：分别指向 deque 中第一个元素和最后一个元素之后的位置，通过迭代器可以快速访问或修改 deque 的元素。

map_size 表示 map 中指针的数量，即数据块的总数。

map 的左右两边留有剩余空间是 deque 设计一个点。

这些剩余空间允许快速地在 deque 的前端或后端添加新的分段，而无需重新分配整个 map。

这样，即使是在 deque 的两端插入或删除元素，操作的时间复杂度也能保持在常数时间内。	

erase 和 insert 函数实现了元素的删除和插入操作，它们采用了不同的策略来最小化元素移动的开销。具体策略取决于操作位置相对于 deque 中间位置的不同，以减少需要移动的元素数量。

当 deque 需要扩展其存储空间时，它会分配一个新的 map，这个新 map 有更多的指针，可以指向更多的缓冲区。然后，deque 会将现有的缓冲区指针从旧 map 复制到新 map 中，并适当地调整 start 和 finish 迭代器，以反映新的布局。这个过程使得 deque 能够在维持高效插入和删除操作的同时，提供对元素的快速随机访问。

当 deque 的一个分段被填满时，它会动态地添加一个新的分段来存储更多的元素。同样地，如果 deque 的大小减少，一些分段可能会被释放以节约空间。deque 动态调整其分段的能力，使得它在存储大量数据时非常灵活和高效。

<h4 id="ae29f962">**EBO(Empty Base Optimization)空基类优化**</h4>
在 `_Deque_impl` 结构体中，使用了 EBO 技术。空基类优化是指在派生类中，基类如果没有数据成员（即是空基类），编译器通常会将其优化掉，从而减少额外的内存开销。

`struct_Deque_impl`

`: public _Tp_alloc_type, public _Deque_impl_data`

`{ /* ... */ };`

在 `_Deque_impl` 中，`_Tp_alloc_type`（内存分配器）可能没有数据成员，因此其继承的实际内存开销为零，这样可以避免因为空基类而增加不必要的内存消耗。

（需要整理）

阅读材料：[https://blog.csdn.net/weixin_46645965/article/details/136779767?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522172424448416800178541517%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=172424448416800178541517&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~baidu_landing_v2~default-5-136779767-null-null.142^v100^pc_search_result_base4&utm_term=%E7%A9%BA%E5%9F%BA%E7%B1%BB%E4%BC%98%E5%8C%96&spm=1018.2226.3001.4187](https://blog.csdn.net/weixin_46645965/article/details/136779767?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522172424448416800178541517%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=172424448416800178541517&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~baidu_landing_v2~default-5-136779767-null-null.142^v100^pc_search_result_base4&utm_term=%E7%A9%BA%E5%9F%BA%E7%B1%BB%E4%BC%98%E5%8C%96&spm=1018.2226.3001.4187)

<h4 id="stack">stack</h4>
💡

关联知识点：适配器模式，函数栈帧。

stack是后进先出的数据结构，有压栈和出栈的操作。

STL中stack是一个容器适配器，它提供了接口，这些接口通过封装另一个底层容器（如 deque或 list）的功能实现的，这使用的是适配器模式。

stack<font style="background-color:#fbf5b3;">默认使用deque底层容器，</font>但用户可以通过模板参数选择其他容器，如 vector 或 list。

(这些容器支持 back(), push_back(), 和 pop_back() 操作）

这种封装提供了灵活性，因为底层容器可以很容易地被替换，而不影响 stack 类的公共接口。

<h4 id="queue">queue</h4>
🔔

关联知识点：适配器模式，任务调度，消息队列，FIFO和FILO的区别

queue是先进先出的数据结构，有压栈和出栈的操作。

STL中queue是一个容器适配器，它提供了接口，这些接口通过封装另一个底层容器（如 deque或 list）的功能实现的，使用的是适配器模式。

默认情况下，queue 使用 deque 作为其底层容器，因为 deque 支持高效的在两端插入和删除操作，提供的功能比queue多，所以可以使用deque实现queue。然而通过模板参数，用户可以指定其他类型的容器(如 list)，只要这个容器支持front()，back()，push_back()和 pop_front()操作。

<h3 id="heap">heap</h3>
🔔

关联知识点：[排序](https://kdocs.cn/l/cjN5ud2YmCis?linkname=RoiqJqutyO)

heap堆是一个完全二叉树，可以用数组实现，STL中堆使用vector表示数组，这样插入的时候可以对数组进行动态扩容，操作见堆排序。

自定义比较函数（...）

<h4 id="priority_queue">priority_queue</h4>
优先队列是在正常队列的基础上加了优先级，保证每次的队首元素都是优先级最大的。每次从优先队列中取出的元素都是队列中**优先级最大**的一个，它的底层是通过**堆**来实现的。

<h3 id="forward_list">forward_list</h3>
forward_list(c++ 11)是单向非循环链表，每个元素只包含指向下一个元素的指针。

forward_list的迭代器是前向迭代器，只能从前向后遍历。

list的迭代器是双向迭代器，所以slist的功能会受限,比如不能反向遍历链表，但是slist省去了一个指针的内存空间，更加轻量级。

**使用forwad_list的场景：**

（1）内存空间有限：当内存使用紧张时，可以使用forward_list。由于只存储单个指针，相比双向链表占用的空间更小。

（2）不需要双向遍历：如果应用场景不需要双向遍历元素，那么 std::forward_list 比 std::list 更加高效。

为什么会有双向遍历？

查找/删除/插入，如果链表是双向链表(如`list`)，可以从当前节点向前或向后查找，以便更快地找到目标元素;

浏览器的历史记录通常需要双向操作，因为用户可以向前和向后浏览页面：

+ **前进和后退**: 浏览器维护一个历史记录链表来支持用户在页面间导航。用户可以前进或后退，双向链表使得这些操作非常高效。

<h2 id="f3e73912">第五章</h2>
<h2 id="xRUQY"> 关联式容器</h2>
关联式容器分为map和set，底层实现可以使用红黑树和哈希表实现。

所以关联式容器分为map，set，unordred_map，unordred_set，

<h4 id="2023ebf7">什么是关联式容器？</h4>
关联式容器是C++标准库中的一种重要数据结构，它允许我们存储键值对（key-value pair）或单独的元素，并基于键（key）来快速访问或检索对应的值（value）或元素。

关联式容器通过内部的数据结构（如红黑树或哈希表）来保存元素。

如果使用红黑树保存，元素就是有序的；如果使用哈希表元素就是无序的。

**关联式容器与序列式容器区别：**

（1）序列式容器按照元素在容器中的位置来存储和访问元素，而关联式容器则是根据元素的键来存储和访问元素。

（2）序列式容器（如vector、list、deque等）底层结构为顺序表，通过索引或迭代器来访问元素，里面存储的是元素本身，提供对元素的顺序访问。关联式容器则通过使用平衡搜索树(红黑树或者哈希表)作为其底层数据结构，可以根据键来快速查找、插入和删除元素。

<h4 id="d0cb4593">STL红黑树实现</h4>
[https://mp.weixin.qq.com/s/Kt623awkLbUC4rJd2zX8Vg](https://mp.weixin.qq.com/s/Kt623awkLbUC4rJd2zX8Vg)stl_tree.h

![](https://cdn.nlark.com/yuque/0/2025/webp/45533403/1742810342994-211486f8-2113-4be2-b706-160cc4b81c5c.webp)

header节点也是红黑树节点， parent指向红黑树的根节点，left指向最左节点，right指向最右节点。

**红黑树节点基类**

```cpp
// 颜色标记
enum _Rb_tree_color { _S_red = false, _S_black = true };
// 基类
struct _Rb_tree_node_base{
// typedef重命名
typedef _Rb_tree_node_base* _Base_ptr;
// 颜色
_Rb_tree_color  _M_color;
// 指向父亲
_Base_ptr    _M_parent;
// 指向左孩子
_Base_ptr    _M_left;
// 指向右孩子
_Base_ptr    _M_right;
// 求红黑树的最小节点
static _Base_ptr
_S_minimum(_Base_ptr __x) _GLIBCXX_NOEXCEPT{
    while (__x->_M_left != 0) __x = __x->_M_left;
    return __x;
}
// 求红黑树最大节点
static _Base_ptr
_S_maximum(_Base_ptr __x) _GLIBCXX_NOEXCEPT
{
    while (__x->_M_right != 0) __x = __x->_M_right;
    return __x;
}
};
```

红黑树节点_Rb_tree_node继承自红黑树基类_Rb_tree_node_base。

由于STL红黑树是有序的，所以需要一个比较函数对象，该对象类型定义如下：

```cpp
template<typename _Key_compare>  
struct _Rb_tree_key_compare  {      
//成员变量就是我们提供给红黑树的仿函数对象 
_Key_compare    _M_key_compare;  
}
```

C++11之后的内存对齐？

**成员函数：**

    - `_M_valptr()` 是一个成员函数，返回 `_Val` 数据成员的地址。
        * 对于 C++11 之前的版本，它返回 `_M_value_field` 的地址。
        * 对于 C++11 及以后的版本，它返回 `_M_storage` 中 `_Val` 数据的地址。
    - 除了非 const 版本外，还有一个 const 版本的 `_M_valptr()`，用于 const 对象，返回 const `_Val*`。

<h4 id="set">set</h4>
[https://mp.weixin.qq.com/s/GX8rmnaTyAQSGz7_5eBxow](https://mp.weixin.qq.com/s/GX8rmnaTyAQSGz7_5eBxow)

<h4 id="hashtable">hashtable</h4>
<h5 id="62abcd08">hashtable 概述</h5>
哈希表是一种数据结构，它提供了快速的数据插入、删除和查找功能。它通过使用哈希函数将键（key）映射到表中的一个位置来实现这一点，这个位置称为哈希值或索引。哈希表使得这些操作的平均时间复杂度为常数时间，即O(1)。

哈希表使用哈希函数将键映射到一个固定大小的数组上。

**碰撞**

两个不同的键通过哈希函数得到了相同的索引。由于哈希表的大小是有限的，而键的数量可能非常多，所以碰撞是不可避免的。

**（1）线性探测**：当发生碰撞时，线性探测会在哈希表中按线性顺序搜索下一个空闲位置。

**（2）二次探测**：与线性探测类似，但是搜索下一个空闲位置时，使用的是二次函数而不是线性函数。

**（3）开链**：每个哈希表的槽位不直接存储元素，而是存储一个链表。当发生碰撞时，新元素会被添加到对应槽位的链表中。

<h5 id="9bbe2a53">hashtable实现</h5>
![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742810343182-93b99146-dd32-4558-8837-b9eaf645903c.png)

hash_table node

```cpp
template <class Value>
struct hashtable_node
{
_hashtable_node* next; 
Value val;
};
```

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742810343327-c006f550-c09f-4bf9-83e2-cfb93a865041.png)

<h5 id="83ea9572">hashtable 的迭代器</h5>
```cpp
// 定义哈希表迭代器模板结构体
template <
class Value, 
class Key,  class HashFcn,  class ExtractKey, 
class EqualKey, 
class Alloc
>
struct hashtable_iterator {
// 使用typedef定义相关类型别名，以简化代码
typedef hashtable<Value, Key, HashFcn, ExtractKey, EqualKey, Alloc> hashtable_type;
typedef hashtable_iterator<Value, Key, HashFcn, ExtractKey, EqualKey, Alloc> iterator_type;
typedef __hashtable_const_iterator<Value, Key, HashFcn, ExtractKey, EqualKey, Alloc> const_iterator_type;
typedef __hashtable_node<Value> node_type;
typedef std::forward_iterator_tag iterator_category;
typedef Value value_type;
typedef ptrdiff_t difference_type;
typedef std::size_t size_type;
typedef Value& reference;
typedef Value* pointer;
// 成员变量
node_type* cur; // 当前节点指针
hashtable_type* ht; // 指向哈希表的指针
// 构造函数
hashtable_iterator(node_type* n = nullptr, hashtable_type* tab = nullptr) : cur(n), ht(tab) {}
// 解引用操作符，返回当前节点的值
reference operator*() const { return cur->val; }
// 成员访问操作符，返回当前节点值的指针
pointer operator->() const { return &(operator*()); }
// 前置++
iterator_type& operator++() {
    // 此处应有逻辑来移动迭代器到下一个节点
    // ...
    return *this;
}
// 后置++
iterator_type operator++(int) {
    iterator_type temp = *this;
    ++(*this); // 使用前置++来实现
    return temp;
}
// 相等比较操作符
bool operator==(const iterator_type& it) const { return cur == it.cur; }

// 不等比较操作符
bool operator!=(const iterator_type& it) const { return !(*this == it); }
};
```

<h5 id="a02e4418">hashtable的数据结构</h5>
```cpp
#include <vector> // For std::vector
#include <cstddef> // For size_t

// 定义哈希表类模板
template <
class Value,class Key,class HashFcn,
class ExtractKey,class EqualKey,
class Alloc = Alloc // 注意：这里应该是具体的分配器类名，例如 std::allocator
>
class hashtable {
public:
// 为模板型别参数重新定义一个名称
typedef HashFcn hasher;
typedef EqualKey key_equal;
typedef size_t size_type;
private:
// 以下三者都是 function objects
hasher hash; // 哈希函数对象
key_equal equalg; // 键值比较函数对象
ExtractKey get_key; // 键提取函数对象
typedef hashtable_node<Value> node; // 节点类型定义
typedef simple_alloc<node, Alloc> node_allocator; // 节点分配器定义
std::vector<node*, Alloc> buckets; // 桶数组，使用 vector 完成
size_type num_elements; // 元素数量
public:
// 构造函数
hashtable():num_elements(0){}
// bucket 个数即 buckets vector 的大小
size_type bucket_count() const { return buckets.size(); }
// 其他成员函数声明...
};

// 注意：hashtable_node, simple_alloc 需要被定义
// 注意：Alloc 应该是一个具体的分配器类，例如 std::allocator
```

虽然开链法(separate chaining)并不要求表格大小必须为质数，但 SGI STL 仍然以质数来设计表格大小，并且先将28 个质数(逐渐呈现大约两倍的关系)计算好，以备随时访问，同时提供一个函数，用来查询在这 28 个质数之中,"最接近某数并大于某数"的质数。

<h5 id="c348e5b9">hashtable 的构造,插入，内存分配，拷贝和回收</h5>
hashtable类中定义了一个node_allocator类型

```cpp
typedef simple_alloc<node, Alloc> node_allocator;
node* new_node(const value_type& obj) {
    node* n = node_allocator::allocate();
    n->next = 0;
    STL_TRY{
        construct(&n->val, obj); return n;
    }
    STL_UNWIND(node_allocator::deallocate(n));
}
void delete_node(node* n) {
    destroy(&n->val);
    node_allocator::deallocate(n);
}
```

`new_node`函数分配内存并构造一个新节点，若构造失败则释放内存。

`delete_node`函数销毁节点的值并释放内存。

<h5 id="f984fc7f">哈希表构造</h5>
```cpp
hashtable<int, int, hash<int>, identity<int>, equal_to<int>, alloc>
iht(50, hash<int>(), equal_to<int>());
```

创建一个哈希表`iht`，初始容量为50，使用默认的哈希函数和比较函数。

<h5 id="d7cacb40">初始化桶</h5>
```cpp
void initialize_buckets(size_type n) {
    const size_type n_buckets = next_size(n);
    buckets.reserve(n_buckets);
    buckets.insert(buckets.end(), n_buckets, (node*)0);
    num_elements = 0;
}
```

`initialize_buckets`函数根据给定的大小`n`计算下一个质数作为桶的数量，并初始化桶为`null`指针。

<h5 id="abc51f0c">插入元素 和 重建 hash table</h5>
```cpp
pair<iterator, bool> insert_unique(const value_type& obj) {
    resize(num_elements + 1);
    return insert_unique_noresize(obj);
}
```

`insert_unique`调用`resize`检查是否需要扩展哈希表的容量。

```cpp
void resize(size_type num_elements_hi) {
    const size_type old_n = buckets.size();
    if (num_elements_hint > old_n) {
        const size_type n = next_size(num_elements_hint);
        if (n > old_n) {
            vector<node*, A> tmp(n, (node*)0);
            STL_TRY {
                for (size_type bucket = 0; bucket < old_n; ++bucket) {
                    node* first = buckets[bucket];
                    while (first) {
                        size_type new_bucket = bkt_num(first->val, n);
                        buckets[bucket] = first->next;
                        first->next = tmp[new_bucket];
                        tmp[new_bucket] = first;
                        first = buckets[bucket];
                    }
                }
            }
            buckets.swap(tmp);
        }
    }
}
```

`resize`函数决定是否需要扩展哈希表，如果当前元素数量超过桶的数量，则重新配置桶并将现有节点转移到新桶中。

<h6 id="b9d60a77">插入不重建</h6>
```cpp
template<class V, class K, class HF, class Ex, class Eq, class A>
typename hashtable<V, K, HF, Ex, Eq, A>::iterator
hashtable<V, K, HF, Ex, Eq, A>::insert_equal_noresize(const value_type& obj) {
    const size_type n = bkt_num(obj); // 确定 obj 应位于 #n 桶
    node* first = buckets[n]; // 指向桶对应的链表头部
//遍历链表，查找重复的键值
    for(node* cur = first;cur;cur=cur->next) {
        if (equals(get_key(cur->val), get_key(obj))) {
            // 如果找到重复键值，插入新节点
            node* tmp = new_node(obj); // 产生新节点
            tmp->next = cur->next; // 新节点指向当前节点的下一个节点
            cur->next = tmp; // 将新节点插入到当前节点后面
            ++num_elements; // 节点个数累加1
            return iterator(tmp, this); // 返回指向新节点的迭代器
        }
    }
//若没有找到重复的键值，插入到链表头部
    node* tmp = new_node(obj); // 产生新节点
    tmp->next = first; // 新节点指向当前链表的头部
    buckets[n] = tmp; // 更新桶指向新节点
    ++num_elements; // 节点个数累加1
    return iterator(tmp, this); // 返回指向新节点的迭代器
}
```

<h5 id="dfdf4ae2">`bkt_num`函数的功能</h5>
`bkt_num`系列函数负责将元素映射到哈希表的桶中，确保即使是无法直接对哈希表大小进行模运算的类型（如`const char*`），也能正确处理。

**版本 1：**接受实值和桶的数量

```cpp
size_type bkt_num(const value_type& obj, size_t n) const {
    return bkt_num_key(get_key(obj), n); // 调用版本 4
}
```

接收一个元素值和桶的数量，首先提取键值（`get_key`），然后调用`bkt_num_key`来计算桶的位置。

**版本 2：**只接受实值

```cpp
size_type bkt_num(const value_type& obj) const {
    return bkt_num_key(get_key(obj)); // 调用版本 3
}
```

只接受元素值，调用`bkt_num_key`，不需要提供桶的数量，默认为当前桶的大小。

**版本 3: **只接受键值

```cpp
size_type bkt_num_key(const key_type& key) const {
    return bkt_num_key(key, buckets.size()); // 调用版本 4
}
```

**版本 4: **接受键值和桶的数量

```cpp
size_type bkt_num_key(const key_type& key,size_t n) const {
    return hash(key)%n;//调用哈希函数并取模
}
```

最基础的版本，接受键值和桶的数量，通过哈希函数计算出键值的哈希值，并对桶的数量进行取模运算，得出最终的桶索引。

<h5 id="024d4765">哈希表的复制(copy_from)和整体删除(clear)</h5>
由于哈希表由向量和链表组成，这两个操作都涉及内存管理。

<h6 id="056df23b">整体删除 `clear`</h6>
遍历每个桶，逐个删除桶中的所有节点，确保调用`delete_node`来释放内存。最后将桶的内容设置为`null`指针，并将节点总数设置为0。注意向量`buckets`本身并没有释放空间，只是清空了其内容。

```cpp
template <class V, class K, class HF, class Ex, class Eq, class A>
void hashtable<V, K, HF, Ex, Eq, A>::clear() {
    // 针对每一个 bucket
    for (size_type i = 0; i < buckets.size(); ++i) {
        node* cur = buckets[i];
        // 删除 bucket list 中的每一个节点
        while (cur != 0) {
            node* next = cur->next;
            delete_node(cur); // 删除当前节点
            cur = next; // 移动到下一个节点
        }
        // 令 bucket 内容为 null 指针
        buckets[i] = 0;
    }
    num_elements = 0; // 令总节点个数为0
    // 注意，buckets vector 并未释放掉空间，仍保有原来大小
}
```

<h6 id="e41c533b">复制 `copy_from`</h6>
复制另一个哈希表的内容。首先清空当前哈希表的`buckets`向量。预留空间以匹配目标哈希表`ht`的桶数量。向`buckets`中插入`null`指针，初始化每个桶。使用`STL_TRY`块来处理复制过程，确保在发生异常时能够安全清理。遍历目标哈希表的每个桶，将节点的值复制到新创建的节点中，保持链表结构。最后，更新当前哈希表的节点总数为目标哈希表的节点数量。

```cpp
template <class V, class K, class HF, class Ex, class Eq, class A>
void hashtable<V, K, HF, Ex, Eq, A>::copy_from(const hashtable& ht) {
    // 先清除己方的 buckets vector
    buckets.clear(); // 清空当前哈希表的内容
    // 为己方的 buckets vector 保留空间，使与对方相同
    buckets.reserve(ht.buckets.size());
    // 插入 n 个 null 指针
    buckets.insert(buckets.end(), ht.buckets.size(), (node*)0);
    STL_TRY {
        // 针对 buckets vector
        for (size_type i = 0; i < ht.buckets.size(); ++i) {
            if (const node* cur = ht.buckets[i]) {
                node* copy = new_node(cur->val); // 复制第一个节点
                buckets[i] = copy; // 设置当前桶的头节点
                // 针对同一个 bucket list 复制每一个节点
           for (node* next = cur->next; next; cur = next, next = cur->next) {
                    copy->next = new_node(next->val); // 复制后续节点
                    copy = copy->next; // 移动到新节点
                }
            }
        }
    }
    num_elements = ht.num_elements; // 重新登录节点个数
    STL_UNWIND(clear()); // 异常时清理资源
}
```

<h5 id="44954d50">hashtable 运用实例</h5>
```cpp
// file: hashtable-test.cpp
// 注意：客户端程序不能直接包含 <stl_hashtable.h>，
// 应该包含有用到 hashtable 的容器头文件，例如 <hash_set.h> 或 <hash_map.h>
#include <hash_set> // for hashtable
#include <iostream>
using namespace std;
int main() {
    // 创建哈希表
    // <value, key, hash-func, extract-key, equal-key, allocator>
    // 注意：哈希表没有默认构造函数
    hashtable<int, int, hash<int>, identity<int>, equal_to<int>, alloc> iht(50, hash<int>(), equal_to<int>());
    // 输出哈希表的信息
    cout << iht.size() << endl; // 输出当前元素数量，初始为0
    cout << iht.bucket_count() << endl; // 输出桶数量，应该是53（第一个质数）
    cout << iht.max_bucket_count() << endl; // 输出最大桶数量，4294967291
    // 插入元素
    iht.insert_unique(59);
    iht.insert_unique(63);
    iht.insert_unique(108);
    iht.insert_unique(2);
    iht.insert_unique(53);
    iht.insert_unique(55);
    // 输出当前元素数量
    cout << iht.size() << endl; // 应为6
    // 声明一个哈希表迭代器
    hashtable<int, int, hash<int>, identity<int>, equal_to<int>, alloc>::iterator ite = iht.begin();
    // 以迭代器遍历哈希表，输出所有节点的值
    for (int i = 0; i < iht.size(); ++i, ++ite) {
        cout << *ite << ' '; // 输出所有节点的值
    }
    cout << endl;
    // 遍历所有桶，输出每个桶的节点个数
    for (int i = 0; i < iht.bucket_count(); ++i) {
        int n = iht.elems_in_bucket(i);
        if (n != 0) {
            cout << "bucket [" << i << "] has " << n << " elems." << endl;
        }
    }
    // 测试表格重建（rehashing）
    for (int i = 0; i <= 47; i++) {
        iht.insert_equal(i); // 插入54个元素
    }
    // 输出节点数量和新的桶数量
    cout << iht.size() << endl; // 应为54
    cout << iht.bucket_count() << endl; // 应为97
    // 再次遍历所有桶，输出节点个数
    for (int i = 0; i < iht.bucket_count(); ++i) {
        int n = iht.elems_in_bucket(i);
        if (n != 0) {
            cout << "bucket [" << i << "] has " << n << " elems." << endl;
        }
    }
    // 再次以迭代器遍历哈希表，输出所有节点的值
    ite = iht.begin();
    for (int i = 0; i < iht.size(); ++i, ++ite) {
        cout << *ite << ';'; // 输出所有节点的值
    }
    cout << endl;
    // 查找元素
    cout << *(iht.find(2)) << endl; // 查找并输出值为2的节点
    cout << iht.count(2) << endl; // 输出值为2的节点个数
    return 0;
}
```

<h5 id="332a150c">`find`函数 和 `count`函数</h5>
`find`函数

函数用于查找具有特定键值的元素，并返回一个迭代器指向该元素。首先，通过`bkt_num_key(key)`确定该元素应位于哪个桶(`bucket`)。接着从该桶的头节点开始循环遍历链表，使用`equals`函数比较每个节点的键值与目标键值。一旦找到匹配的键值，循环结束，返回一个指向该节点的迭代器。如果未找到匹配，返回的迭代器将指向空。

`**count**`**函数**:

函数用于计算特定键值在哈希表中的出现次数。同样地，首先通过`bkt_num_key(key)`确定该元素应位于哪个桶。然后，从该桶的头节点开始，循环遍历链表。对于每个节点，使用`equals`函数判断键值是否与目标键值相同。如果匹配，`result`计数器加1。最后返回`result`，即该键值的总出现次数。

```cpp
//查找元素的迭代器
iterator find(const key_type& key) {
    size_type n = bkt_num_key(key); // 首先找到元素应该落在哪一个桶内
    node* first;
    // 从桶的头部开始，依次比较每个元素的键值
    for (first = buckets[n]; first && !equals(get_key(first->val), key); first = first->next) {
        // 循环直到找到匹配的键值，或到达链表末尾
    }
    return iterator(first, this); // 返回迭代器，指向找到的节点或为空
}
//计算特定键值的出现次数
size_type count(const key_type& key) const {
    const size_type n = bkt_num_key(key); // 首先找到元素应该落在哪一个桶内
    size_type result = 0;
    // 从桶的头部开始，依次比较每个元素的键值
    for (const node* cur = buckets[n]; cur; cur = cur->next) {
        if (equals(get_key(cur->val), key)) {
            ++result; // 如果找到匹配，累加计数
        }
    }
    return result; // 返回特定键值的出现次数
}
```

<h5 id="hash-functions">hash functions</h5>
```cpp
//hash function 基础定义
template <class Key> struct hash { };
//字符串的哈希计算函数
inline size_t stl_hash_string(const char* s) {
    unsigned long h = 0;
    for (; *s; ++s) {
        h = 5 * h + *s; // 计算哈希值
    }
    return size_t(h);
}
//针对不同类型的哈希函数实现,对于char*类型的哈希函数
STL_TEMPLATE_NULL struct hash<char*> {
    size_t operator()(const char* s) const { return stl_hash_string(s); }
};
//对于 const char* 类型的哈希函数
STL_TEMPLATE_NULL struct hash<const char*> {
    size_t operator()(const char* s) const { return stl_hash_string(s); }
};
//对于单字符类型的哈希函数
STL_TEMPLATE_NULL struct hash<char> {
    size_t operator()(char x) const { return x; }
};
//对于无符号字符类型的哈希函数
STL_TEMPLATE_NULL struct hash<unsigned char> {
    size_t operator()(unsigned char x) const { return x; }
};
//对于有符号字符类型的哈希函数
STL_TEMPLATE_NULL struct hash<signed char> {
    size_t operator()(signed char x) const { return x; }
};
//对于短整型的哈希函数
STL_TEMPLATE_NULL struct hash<short> {
    size_t operator()(short x) const { return x; }
};
//对于无符号短整型的哈希函数
STL_TEMPLATE_NULL struct hash<unsigned short> {
    size_t operator()(unsigned short x) const { return x; }
};
//对于整型的哈希函数
STL_TEMPLATE_NULL struct hash<int> {
    size_t operator()(int x) const { return x; }
};
//对于无符号整型的哈希函数
STL_TEMPLATE_NULL struct hash<unsigned int> {
    size_t operator()(unsigned int x) const { return x; }
};
//对于长整型的哈希函数
STL_TEMPLATE_NULL struct hash<long> {
    size_t operator()(long x) const { return x; }
};
//对于无符号长整型的哈希函数
STL_TEMPLATE_NULL struct hash<unsigned long> {
    size_t operator()(unsigned long x) const { return x; }
};
```

对于整型和字符型数据，哈希函数通常直接返回原值。

对于字符串，调用 `stl_hash_string` 函数以计算哈希值（每位的ascii码值然后乘以5）。

SGI hashtable 无法处理上述所列各项型别以外的元素，例如string,double,float。欲处理这些型别，用户必须自行为它们定义 hash function。

<h4 id="hashset">hashset</h4>
hash_set键值对都是value，直接调用底层hashtable对应的方法。例如insert方法会调用hashtable的insert_unique方法来确保没有重复元素被添加到集合中。

需要注意的是，由于hash_set不提供自动排序的功能，所以如果需要有序的元素，应该使用set而不是hash_set。

```cpp
#include <stl_function.h>  // 假设 identity<> 定义在这里
template <class Value,class HashFcn = hash<Value>,
class EqualKey = equal_to<Value>,class Alloc = alloc>
class hash_set{
private:
    //使用identity<>将键值转换为实值，因为set的键值就是实值
    typedef hashtable<
        Value, Value,  // 键值类型与实值类型都是 Value
        HashFcn,       // 哈希函数
        identity<Value>,  // 身份转换器
        EqualKey,      // 比较相等的函数
        Alloc          // 分配器
    > ht;
    ht rep;  // 底层使用哈希表来存储数据
public:
    // 类型定义
    typedef typename ht::key_type key_type;
    typedef typename ht::value_type value_type;
    typedef typename ht::hasher hasher;
    typedef typename ht::key_equal key_equal;
    typedef typename ht::size_type size_type;
    typedef typename ht::difference_type difference_type;
    typedef typename ht::const_pointer pointer;
    typedef typename ht::const_pointer const_pointer;
    typedef typename ht::const_reference reference;
    typedef typename ht::const_reference const_reference;
    typedef typename ht::const_iterator iterator;
    typedef typename ht::const_iterator const_iterator;
    // 获取哈希函数和比较函数
    hasher hash_funct() const { return rep.hash_funct(); }
    key_equal key_eq() const { return rep.key_eq(); }
    // 构造函数
    hash_set() : rep(100, hasher(), key_equal()) {}
    explicit hash_set(size_type n) : rep(n, hasher(), key_equal()) {}
    hash_set(size_type n, const hasher& hf) : rep(n, hf, key_equal()) {}
    hash_set(size_type n, const hasher& hf, const key_equal& eql) : rep(n, hf, eql) {}
    // 从迭代器范围构造
    template <class InputIterator>
    hash_set(InputIterator f, InputIterator l)
        : rep(100, hasher(), key_equal()) { rep.insert_unique(f, l); }
    template <class InputIterator>
    hash_set(InputIterator f, InputIterator l, size_type n)
        : rep(n, hasher(), key_equal()) { rep.insert_unique(f, l); }
    template <class InputIterator>
    hash_set(InputIterator f, InputIterator l, size_type n, const hasher& hf)
        : rep(n, hf, key_equal()) { rep.insert_unique(f, l); }
    template <class InputIterator>
    hash_set(InputIterator f, InputIterator l, size_type n, const hasher& hf, const key_equal& eql)
        : rep(n, hf, eql) { rep.insert_unique(f, l); }
    // 其他成员函数
    size_type size() const { return rep.size(); }
    size_type max_size() const { return rep.max_size(); }
    bool empty() const { return rep.empty(); }
    void swap(hash_set& hs) { rep.swap(hs.rep); }
    friend bool operator== __STL_NULL_TMPL_ARGS (const hash_set&, const hash_set&);
    iterator begin() const { return rep.begin(); }
    iterator end() const { return rep.end(); }
    pair<iterator, bool> insert(const value_type& obj) {
        pair<typename ht::iterator, bool> p = rep.insert_unique(obj);
        return pair<iterator, bool>(p.first, p.second);
    }
    template <class InputIterator>
    void insert(InputIterator f, InputIterator l) { rep.insert_unique(f, l); }
    pair<iterator, bool> insert_noresize(const value_type& obj) {
        pair<typename ht::iterator, bool> p = rep.insert_unique_noresize(obj);
        return pair<iterator, bool>(p.first, p.second);
    }
    iterator find(const key_type& key) const { return rep.find(key); }
    size_type count(const key_type& key) const { return rep.count(key); }
    pair<iterator, iterator> equal_range(const key_type& key) const { return rep.equal_range(key); }
    size_type erase(const key_type& key) { return rep.erase(key); }
    void erase(iterator it) { rep.erase(it); }
    void erase(iterator f, iterator l) { rep.erase(f, l); }
    void clear() { rep.clear(); }
    void resize(size_type hint) { rep.resize(hint); }
    size_type bucket_count() const { return rep.bucket_count(); }
    size_type max_bucket_count() const { return rep.max_bucket_count(); }
    size_type elems_in_bucket(size_type n) const { return rep.elems_in_bucket(n); }
};
// 重载 == 运算符
template <class Value, class HashFcn, class EqualKey, class Alloc>
inline bool operator==(const hash_set<Value, HashFcn, EqualKey, Alloc>& hs1,
                       const hash_set<Value, HashFcn, EqualKey, Alloc>& hs2)
{
    return hs1.rep == hs2.rep;
}
```

`unordered_set` 是 C++11 标准库的一部分，属于 STL（标准模板库）。`hash_set` 是早期的实现，通常在某些编译器或库中提供（如 SGI STL），但不是标准的一部分。`unordered_set` 使用哈希表实现，提供平均常数时间复杂度的查找、插入和删除操作。`hash_set` 也使用哈希表，但实现细节和性能可能因库而异。

`unordered_map` 是 C++11 标准库的一部分，属于STL(标准模板库)。`hash_map` 是早期的实现，通常在某些编译器或库中提供（如 SGI STL），但不是标准的一部分，可能不在所有编译器上可用。

<h2 id="94b5c169">第六章 算法</h2>
<h4 id="0100b5a5">STL 算法总览</h4>
![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742810343399-66bac52b-0e35-49c9-bedc-d9dbb628a4b0.png)

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742810343462-a4f3ff8b-3845-4710-b819-e7385253edd3.png)

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742810344207-8bb920e4-a9b3-4e93-b01c-b97e009b2574.png)

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742810343715-fe78bf1a-ba6e-46f2-931e-c98ac05e5c25.png)

<h5 id="3eb88c80">质变算法 mutating algorithms—会改变操作对象之值</h5>
所有的 STL算法都作用在由迭代器(first,last)所标示出来的区间上。所谓“质变算法”,是指运算过程中会更改区间内(迭代器所指)的元素内容。诸如拷贝(copy)、互换(swap)、替换(replace)、填写(fill)、删除(remove)、排列组合(permutation)、分割(partition)、随机重排(random shuffling)、排序(sort)等算法，都属此类。

```cpp
int ia[] ={ 22,30,30,17,33,40,17,23,22,12,20 };
vector<int> iv(ia, ia+sizeof(ia)/sizeof(int));
vector<int>::const_iterator cite1 = iv.begin();
vector<int>::const_iterator cite2 = iv.end();
sort(cite1,cite2);
```

对常迭代器指向的元素进行sort 操作,编译器会报错。

<h5 id="117e39e5">非质变算法(不改变操作对象之值)</h5>
所有的STL算法都作用在由迭代器[first,last)所标示出来的区间上。非质变算法”,是指运算过程中不会更改迭代器所指的元素内容。查找(find),匹配(search),计数(count),巡访(for_each),比较,寻找极值(max, min)等算法。

<h5 id="156017d7">STL 算法的一般形式</h5>
所有泛型算法的前两个参数都是一对迭代器(iterators),通常称为 first 和last,用以标示算法的操作区间。STL 习惯采用前闭后开区间表示法。当 first==last时，上述所表现的便是一个空区间。

这个[first,last)区间的必要条件是，必须能够经由累加操作符的反复运用从 first到达 last。编译器本身无法强求这一点。如果这个条件不成立，会导致未可预期的结果。

质变算法(mutating algorithms,6.1.3节)通常提供两个版本：一个是 in-place (就地进行)版，就地改变其操作对象；另一个是 copy(另地进行)版，将操作 对象的内容复制一份副本，然后在副本上进行修改并返回该副本。

不是所有质变算法都有 copy 版，例如 sort()就没有。如果我们希望以这类“无 copy 版本”之质变算法施行于某一段区间元素的副本身上，我们必须自行制作并传递那一份副本。

<h4 id="5a42a577">算法的泛化过程</h4>
如何将算法独立于其所处理的数据结构之外，不受数据结构的羁绊，思想层面就不是那么简单了。如何设计一个算法，适用于大多数数据结构呢？只要把操作对象的型别加以抽象化，把操作对象的标示法和区间目标的移动行为抽象化，整个算法也就在一个抽象层面上工作了,整个过程称为算法的泛型化,简称泛化。如写一个find()函数，在 array 中寻找特定值。

```cpp
int* find(int* arrayHead,int arraySize, int value){
  for(int i=0;i<arraySize; ++i)
     if(arrayHead[i]==value)break;
  return &(arrayHead[i]);
}
```

"最后元素的下一位置"称为 end。返回end以表示"查找无结果"似乎是个可笑的做法。为什么不返回 null? 因为，一如稍后即将见到的，end 指针可以对其他种类的容器带来泛型效果，这是 null 所无法达到的。

```cpp
int* find(int* begin,int* end,int value){
  while(begin != end && *begin != value) ++begin;
   return begin;
}
```

这个函数在“前闭后开”区间[begin, end)内(end 指向 array最后元素的下一位置)查找 value，并返回一个指针，指向它所找到的第一个符合条件的元素；如果没有找到，就返回end。

```cpp
template<typename T>//关键词 typename 也可改为关键词 class
T* find(T* begin,T* end, const T& .value){
  //以下用到了 operator!=, operator*, operator++
  while(begin != end && *begin != value)++begin;
  return begin;
}
```

传统的`find`函数可能设计为接受原生指针作为参数,用于在数组或其他线性结构中寻找特定元素。然而，这限制了`find`只能用于那些可以直接用指针遍历的数据结构。  
 如果我们想要让`find`函数能够处理更复杂的数据结构，比如链表(链表是自定义的类，不能使用指针递增递减)，原生指针不能简单地通过递增(`++`)来指向下一个元素。

STL引入了迭代器，迭代器是一种行为类似于指针的对象，但它提供了丰富的接口来适应不同的数据结构。在链表中一个迭代器可以设计成当执行递增操作，会移动到链表中的下一个节点。迭代器比喻为"智能指针"，迭代器不仅模仿了指针的基本功能（如解引用和递增/递减），还加入了额外的功能。使用迭代器`find`函数就可以被设计为接受任何类型的迭代器，从而使得这个函数可以用来搜索不同种类的容器，包括但不限于数组,向量(vector)、列表(list)、集合(set)等。

<h4 id="53f2d082">数值算法</h4>
<h5 id="accumulate">accumulate</h5>
主要功能是将一个范围内的所有元素累加起来，并返回累加的结果。

```cpp
template <class InputIterator, class T>
T accumulate(InputIterator first, InputIterator last, T init);
template <class InputIterator, class T, class BinaryOperation>
T accumulate(InputIterator first, InputIterator last, T init, BinaryOperation binary_op);
```

<h5 id="adjacent_difference">adjacent_difference</h5>
```cpp
template <class InputIterator, class OutputIterator>
OutputIterator adjacent_difference(InputIterator first, InputIterator last, OutputIterator result);
template <class InputIterator, class OutputIterator, class BinaryOperation>
OutputIterator adjacent_difference(InputIterator first, InputIterator last, OutputIterator result, BinaryOperation binary_op);
```

`binary_op` 是一个二元操作符，它可以是你自定义的函数对象或lambda表达式，用来替代默认的减法操作。

<h5 id="inner_product">inner_product</h5>
```cpp
template <class InputIterator1, class InputIterator2, class T>
T inner_product(InputIterator1 first1, InputIterator1 last1,
                InputIterator2 first2, T init);
template <class InputIterator1, class InputIterator2, class T,
          class BinaryOperation1, class BinaryOperation2>
T inner_product(InputIterator1 first1, InputIterator1 last1,
                InputIterator2 first2, T init,
                BinaryOperation1 binary_op1, BinaryOperation2 binary_op2);
```

主要功能是计算两个序列的内积（点积）。内积通常用于向量运算，它是两个向量对应元素相乘后的和。

```cpp
std::vector<int>vec1={1, 2, 3};
std::vector<int>vec2={4, 5, 6};
//使用默认的乘法和加法操作
int product_sum = std::inner_product(vec1.begin(), vec1.end(), vec2.begin(), 0); // 结果是 1*4 + 2*5 + 3*6 = 32
```

<h5 id="partial_sum">partial_sum</h5>
对于输入序列 [a0, a1, a2, ..., an]，partial_sum 会生成一个新的序列 [a0, a0 + a1, a0 + a1 + a2, ..., a0 + a1 + ... + an]。

```cpp
template <class InputIterator,class OutputIterator>
OutputIterator partial_sum(InputIterator first, 
 InputIterator last, OutputIterator result);

template <class InputIterator,class OutputIterator,class BinaryOperation>
OutputIterator partial_sum(InputIterator first, 
InputIterator last, OutputIterator result,BinaryOperation binary_op);
```

<h4 id="fill">fill</h4>
将[first,last）内的所有元素改填新值。

```cpp
template <class ForwardIterator, class T>
void fill(ForwardIterator first,ForwardIterator last, const T& value) //迭代走过整个区间 
for(;first!=last; ++first) //设定新值 
   *first=value;
}
```

<h4 id="fill_n">fill_n</h4>
将[ first，last）内的前n个元素改填新值，返回的迭代器指向被填人的最后一个元素的下一位置。

```cpp
template <class OutputIterator, class Size, class T>
OutputIterator fil1_n(OutputIterator first, Size n, const T& value){ 
//经过n个元素,设定新值 
for(; n>0; --n,++first) *first = value;
  return first;
}
```

<h5 id="lexicographical_compare">lexicographical_compare</h5>
以“字典排列方式”对两个序列[first1,last1)和[first2,last2)进行比较。

<h5 id="max">max</h5>
取两个对象中的较大值。有两个版本，版本一使用对象型别T所提供的greater-than操作符来判断大小，版本二使用仿函数 comp来判断大小。

```cpp
template <class T>
inline const T& max(const T& a, const T& b)
return a < b ? b:a;
}
template <class T,class Compare>
inline const T& max(const T& a, const T& b, Compare comp)(
// 由 comp 决定“大小比较”标准 
return comp(a,b)? b:a;
}
```

<h5 id="min">min</h5>
取两个对象中的较小值。有两个版本，版本一使用对象型别 T 所提供的less-than 操作符来判断大小，版本二使用仿函数 comp来判断大小。

```cpp
template <class T>
inline const T& min(const TE a, const T& b)
return b< a ? b:a;
}
template <class T,class Compare>
inline const T& min(const T& a,const. T& b,Compare comp){
  return comp(b,a)? b:a; // 由 comp 决定“大小比较”标准
}
```

<h5 id="mismatch">mismatch</h5>
用来平行比较两个序列，指出两者之间的第一个不匹配点，返回一对迭代器,分别指向两序列中的不匹配点。

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742810343846-66c116ea-39e0-45b8-94ca-5f1db3575715.png)

<h5 id="swap">swap</h5>
该函用交换(对调)两的 容。

```cpp
template <class T>
inline void swap(T& a, T& b){
   T tmp=a; a=b; b=tmp;
}
```

<h4 id="copy">copy</h4>
阅读资料：[https://blog.csdn.net/Johnsonjjj/article/details/107743872?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-1-107743872-blog-135630724.235%5Ev43%5Epc_blog_bottom_relevance_base8&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-1-107743872-blog-135630724.235%5Ev43%5Epc_blog_bottom_relevance_base8&utm_relevant_index=2](https://blog.csdn.net/Johnsonjjj/article/details/107743872?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-1-107743872-blog-135630724.235%5Ev43%5Epc_blog_bottom_relevance_base8&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-1-107743872-blog-135630724.235%5Ev43%5Epc_blog_bottom_relevance_base8&utm_relevant_index=2)(写的比较详细附有源码)

copy函数原型

```cpp
template<class InputIterator, class OutputIterator>  
inline OutputIterator copy(  
      InputIterator first,   
      InputIterator last,   
      OutputIterator result  
);
//first last拷贝到result的位置
```

copy算法第一个参数和第二个迭代器是输入迭代器，第三个迭代器是输出迭代器。

copy函数针对不同类型的迭代器采取不同的策略。

对于字符串（`const char*` 和 `const wchar_t*`），使用<font style="background-color:#fbf5b3;">全特化版本</font>直接通过 `memmove` 复制。

一般的迭代器，使用泛化版本并通过`__copy_dispatch`结构体根据迭代器类型(类型萃取)选择到合适的实现，这使用函数重载实现。

对于随机访问迭代器（由 `std::random_access_iterator_tag` 表示），可以利用迭代器的随机访问特性进行优化。在 `__copy` 的随机访问迭代器版本中，使用 `last - first` 计算距离，并使用循环来减少迭代器的操作次数，从而提高效率。

(在copy的时候随机迭代器可以计算copy的长度，所以for循环的长度确定，可以使用循环展开进行优化)

阅读材料：[https://blog.csdn.net/weixin_39538031/article/details/135155700](https://blog.csdn.net/weixin_39538031/article/details/135155700)

对于其他迭代器类别（如输入迭代器、前向迭代器、双向迭代器), 使用通用的 `__copy` 实现。这些迭代器类别不支持随机访问，因此每次循环都需要显式地递增迭代器。

对于指针迭代器，`__copy_dispatch` 会检查指针指向的类型是否支持 trivial assignment，如果是，则使用 `memmove`；否则调用非 trivial 的复制逻辑，逐个元素进行拷贝。

这里涉及到模板全特化和偏特化：  
阅读资料：[https://harttle.land/2015/10/03/cpp-template.html(](https://harttle.land/2015/10/03/cpp-template.html()这个博客和博主非常强，可以向他学习。以后考虑做一个自己的博客，并多阅读，多参与开源项目扩大自己的影响力）

<h4 id="6152c232">set相关算法</h4>
STL 一共提供了四种与 set(集合)相关的算法，分别是并集(union)、交集(intersection)、差集(difference)、对称差集(symmetric difference)。

<h4 id="dc220b6a">heap 算法</h4>
make_heap(),pop_heap(),push_heap(),sort_heap()

<h4 id="963cf597">数据处理相关</h4>
<h5 id="-1">`adjacent_find`</h5>
查找范围内相邻且相等的元素对，返回指向第一个这样的元素的迭代器。

```cpp
template <class ForwardIterator> ForwardIterator 
  adjacent_find(ForwardIterator first, ForwardIterator last);
```

<h5 id="-2">`count`</h5>
计算范围内等于给定值的元素个数

```cpp
template <class InputIterator, class T> typename iterator_traits<InputIterator>::difference_type 
count(InputIterator first, InputIterator last, const T& value);
```

<h5 id="-3">`count_if`</h5>
计算范围内满足特定条件的元素个数

```cpp
template <class InputIterator, class Predicate> typename iterator_traits<InputIterator>::difference_type 
count_if(InputIterator first, InputIterator last, Predicate pred);
```

<h5 id="-4">`find`</h5>
查找范围内第一个等于给定值的元素

```cpp
template <class InputIterator, class T> InputIterator 
find(InputIterator first, InputIterator last, const T& value);
```

<h5 id="-5">`find_if`</h5>
查找范围内第一个满足特定条件的元素

```cpp
template <class InputIterator, class Predicate> InputIterator 
find_if(InputIterator first, InputIterator last, Predicate pred);
```

<h5 id="-6">`find_end`</h5>
查找一个范围在另一个范围内的最后一次出现位置

```cpp
template <class ForwardIterator1, class ForwardIterator2> ForwardIterator1 
find_end(ForwardIterator1 first1, ForwardIterator1 last1, 
ForwardIterator2 first2, ForwardIterator2 last2);
```

<h5 id="-7">`find_first_of`</h5>
查找一个范围在另一个范围内的第一次出现位置。

```cpp
template <class ForwardIterator1, class ForwardIterator2> ForwardIterator1 
find_first_of(ForwardIterator1 first1, ForwardIterator1 last1, 
              ForwardIterator2 first2, ForwardIterator2 last2);
```

<h5 id="-8">`for_each`</h5>
对范围内的每个元素应用一个函数或仿函数。

```cpp
template <class InputIterator, class Function> Function 
for_each(InputIterator first, InputIterator last, Function f);
```

<h5 id="-9">`generate`</h5>
使用生成器填充范围内的元素

```cpp
template <class ForwardIterator, class Generator> void 
generate(ForwardIterator first, ForwardIterator last, Generator gen);
```

<h5 id="-10">`generate_n`</h5>
使用生成器为指定数量的元素赋值

```cpp
template <class OutputIterator, class Size, class Generator> 
OutputIterator 
generate_n(OutputIterator first,Size n,Generator gen);
```

<h5 id="-11">`remove`</h5>
移除所有等于给定值的元素，但不调整容器大小。

```cpp
template <class ForwardIterator, class T> ForwardIterator 
remove(ForwardIterator first, ForwardIterator last, const T& value);
```

<h5 id="-12">`remove_copy`</h5>
将不等于给定值的元素复制到另一区间，并移除原区间中等于该值的元素。

```cpp
template <class InputIterator, class OutputIterator, class T> OutputIterator 
remove_copy(InputIterator first, InputIterator last, OutputIterator result, 
const T& value);
```

<h5 id="-13">`remove_if`</h5>
根据条件移除元素，但不调整容器大小

```cpp
template <class ForwardIterator, class Predicate>ForwardIterator 
remove_if(ForwardIterator first, ForwardIterator last, Predicate pred);
```

<h5 id="-14">`remove_copy_if`</h5>
根据条件将元素复制到另一区间，并移除原区间中满足条件的元素。

```cpp
template<class InputIterator, class OutputIterator, class Predicate> 
OutputIterator 
remove_copy_if(InputIterator first, InputIterator last, 
OutputIterator result, Predicate pred);
```

<h5 id="-15">`replace`</h5>
将范围内所有等于旧值的元素替换为新值。

```cpp
template <class ForwardIterator, class T1, class T2> void 
replace(ForwardIterator first, ForwardIterator last, 
const T1& old_value, const T2& new_value);
```

<h5 id="-16">`replace_copy`</h5>
将范围内所有等于旧值的元素替换为新值，并复制到另一区间。

```cpp
template <class InputIterator, class OutputIterator, class T1, class T2> 
OutputIterator replace_copy(InputIterator first, InputIterator last, 
OutputIterator result, const T1& old_value, const T2& new_value);
```

<h5 id="-17">`replace_if`</h5>
根据条件将元素替换为新值。

```cpp
template <class ForwardIterator, class Predicate, class T> void 
replace_if(ForwardIterator first, ForwardIterator last, 
Predicate pred, const T& new_value);
```

<h5 id="-18">`replace_copy_if`</h5>
根据条件将元素替换为新值，并复制到另一区间

```cpp
template <class InputIterator, class OutputIterator, class Predicate, class T> 
OutputIterator replace_copy_if(InputIterator first, InputIterator last, 
OutputIterator result, Predicate pred,const T& new_value);
```

<h5 id="-19">`reverse`</h5>
反转范围内的元素顺序

```cpp
template <class BidirectionalIterator> void 
reverse(BidirectionalIterator first, BidirectionalIterator last);
```

<h5 id="-20">`reverse_copy`</h5>
反转并复制范围内的元素到另一区间。

```cpp
template <class BidirectionalIterator, class OutputIterator> OutputIterator 
reverse_copy(BidirectionalIterator first, 
BidirectionalIterator last, OutputIterator result);
```

<h5 id="-21">`rotate`</h5>
循环移动范围内的元素

```cpp
template <class ForwardIterator> void 
rotate(ForwardIterator first,ForwardIterator middle,ForwardIterator last);
```

<h5 id="-22">`rotate_copy`</h5>
循环移动并复制范围内的元素到另一区间。

```cpp
template <class ForwardIterator, class OutputIterator> OutputIterator rotate_copy(ForwardIterator first, ForwardIterator middle, ForwardIterator last, OutputIterator result);
```

<h5 id="-23">`search`</h5>
查找一个范围在另一个范围内的第一次出现位置

```cpp
template <class ForwardIterator1, class ForwardIterator2> 
ForwardIterator1 
search(ForwardIterator1 first1,  ForwardIterator1 last1, 
ForwardIterator2 first2, ForwardIterator2 last2);
```

<h5 id="-24">`search_n`</h5>
查找连续出现给定次数的值。

```cpp
template <class ForwardIterator, class Size, class T> 
ForwardIterator 
search_n(ForwardIterator first, ForwardIterator last, 
Size count, const T& value);
```

<h5 id="-25">`swap_ranges`</h5>
交换两个相同长度区间的元素。

```cpp
template <class ForwardIterator1, class ForwardIterator2> 
ForwardIterator2 
swap_ranges(ForwardIterator1 first1,ForwardIterator1 last1, 
ForwardIterator2 first2);
```

<h5 id="-26">`transform`</h5>
对范围内的每个元素应用一元操作。

```cpp
template <class InputIterator, class OutputIterator, class UnaryOperation>
OutputIterator 
transform(InputIterator first1,InputIterator last1,
OutputIterator result,UnaryOperation op);
```

对两个范围内的元素应用二元操作，并将结果存储在输出迭代器所指向的位置。

```cpp
template <class InputIterator1, class InputIterator2, class OutputIterator, 
class BinaryOperation> 
OutputIterator 
transform(InputIterator1 first1, InputIterator1 last1, 
InputIterator2 first2, OutputIterator result, 
BinaryOperation binary_op);
```

<h5 id="-27">`max_element`</h5>
找出范围内最大元素的位置

```cpp
template <class ForwardIterator> ForwardIterator 
max_element(ForwardIterator first, ForwardIterator last);
```

<h5 id="min_element">min_element</h5>
找出范围内最小元素的位置

```cpp
template <class ForwardIterator> ForwardIterator 
min_element(ForwardIterator first, ForwardIterator last);
```

<h5 id="-28">`includes`</h5>
检查一个已排序的序列是否包含另一个已排序序列的所有元素。

```cpp
template <class InputIterator1, class InputIterator2> bool 
includes(InputIterator1 first1,InputIterator1 last1, 
InputIterator2 first2,InputIterator2 last2);
```

<h5 id="-29">`merge`</h5>
合并两个已排序的序列到一个有序序列

```cpp
template <class InputIterator1, class InputIterator2, class OutputIterator> 
OutputIterator 
merge(InputIterator1 first1, InputIterator1 last1, 
InputIterator2 first2,InputIterator2 last2,OutputIterator result);
```

<h5 id="-30">`partition`</h5>
重新排列范围内的元素，使得满足条件的元素位于不满足条件的元素之前。

```cpp
template <class BidirectionalIterator, class Predicate> 
BidirectionalIterator partition
(BidirectionalIterator first, BidirectionalIterator last,
Predicate pred);
```

<h5 id="-31">`unique`</h5>
移除相邻重复元素，但不调整容器大小

```cpp
template <class ForwardIterator> ForwardIterator 
unique(ForwardIterator first, ForwardIterator last);
```

<h5 id="-32">`unique_copy`</h5>
移除相邻重复元素，并将唯一元素复制到另一区间。

```cpp
template<class InputIterator,class OutputIterator>OutputIterator 
unique_copy(InputIterator first,InputIterator last,
OutputIterator result);
```

<h4 id="afcfa06c">lower_bound和upper_bound</h4>
lower_bound 返回的是“不小于”给定值的第一个位置。

upper_bound 返回的是“大于”给定值的第一个位置。

例如，在一个包含 {1, 2, 4, 4, 6, 7} 的有序数组中，对于值 4:

lower_bound 将返回指向第一个 4 的迭代器。

upper_bound 将返回指向第一个大于 4 的元素（即 6）的迭代器。

如果你想要找到等于给定值的所有元素的范围，你可以结合使用 lower_bound 和 upper_bound 来获取这个范围。比如上面的例子中，[lower_bound, upper_bound) 将给出所有 4 的范围。

<h4 id="random_shuffle">random_shuffle</h4>
将(first,last)的元素次序随机重排

<h4 id="a85cb289">partial_sort/partial_sort_copy</h4>
+ 首先,`partial_sort`使用`make_heap()`将区间 `[first, middle)` 构造成一个最大堆（max-heap）。
+ **筛选较小元素**：然后，它遍历 `[middle, last)` 中的每一个元素，将其与堆顶元素进行比较。如果新元素小于堆顶元素，则替换堆顶元素，并重新调整堆的结构以保持最大堆的性质。
+ **堆排序**：遍历结束后，`[first, middle)` 中已经包含了最小的 `N` 个元素，但顺序并未完全排序，因此最后调用 `sort_heap()` 对该区间进行排序，使其按升序排列。

`[first, last)`，`partial_sort` 会将这个序列中的前 `N` 个最小元素放入 `[first, middle)` 中，其中 `middle` 是一个指向序列中间位置的迭代器，通常定义为 `first + N`。排序后，`[first, middle)` 中的元素是按升序排列的最小元素，而 `[middle, last)` 中的元素则不保证有序。

```cpp
std::vector<int> data = {7, 3, 5, 8, 1, 9, 2, 6, 4, 0};
int N = 5; 
//希望找到并排序前5个最小的元素
//使用 partial_sort 将前 N 个最小的元素排好序    
std::partial_sort(data.begin(), data.begin() + N, data.end());
```

<h4 id="sort">sort</h4>
**快速排序（Quick Sort）**

    - `sort()` 的主要部分是基于快速排序的。快速排序是一种分治的排序算法，通过选择一个基准元素，然后将数据分区使得所有小于基准的元素放在它的左边，大于基准的元素放在它的右边。对于每一个分区，它递归地进行排序，直到所有元素都被排序完成。

**插入排序（Insertion Sort）**

    - 当递归排序的分区规模较小（通常在 STL 的实现中，这个阈值为 16 或更小）时，为了避免递归调用的开销，`sort()` 会切换到插入排序。
    - 插入排序在小规模数据上性能优异，因为它的比较和交换操作相对较少，特别是在数据接近有序的情况下。

**堆排序（Heap Sort）**：

    - 如果在执行快速排序时递归深度过大，导致效率下降（例如，数据接近有序的情况可能导致快速排序的最坏情况发生），`sort()` 会自动切换到堆排序。
    - 堆排序是一种时间复杂度为 O(n log n) 的算法，可以有效避免快速排序在最坏情况下的退化问题。它使用堆结构来排序数据，确保算法在最坏情况下仍然能保持较好的性能。

**为什么要求 RandomAccessIterator**

STL `sort()` 算法要求容器提供 `RandomAccessIterator`（随机访问迭代器），原因如下：

+ 快速排序和堆排序都需要频繁访问数据，并且会对数据进行大量的索引操作。随机访问迭代器能够在常数时间内访问任意位置的元素。
+ 如果迭代器不支持随机访问（如双向迭代器或前向迭代器），则每次访问元素时都可能需要线性时间，这会极大降低排序的效率。

**不适用 **`**sort()**`** 的情况**

+ **关联容器（Associative Containers）**：
    - 像 `set`, `map` 这类关联容器，底层是红黑树（RB-tree）实现，元素在插入时就会被自动排序，因此不需要使用 `sort()` 算法。
+ **序列容器**：
    - 对于 `vector` 和 `deque`，它们支持随机访问迭代器，因此可以使用 `sort()` 算法。
    - 对于 `list` 和 `slist`，它们的迭代器分别是双向迭代器（BidirectionalIterator）和单向迭代器（ForwardIterator），不支持随机访问。对于这种情况，应使用它们的成员函数 `sort()`，这些函数针对容器的特点进行了优化。

SGI STL 的sort()算法实现是一个混合排序算法，主要基于快速排序,并结合了堆排序和插入排序。这种排序算法被称 Introsort，它根据不同的情况自动选择最合适的排序算法，从而在性能和稳定性之间取得平衡。

下面是对这个sort()实现细节的解析：

1. sort()函数入口

```cpp
template <class RandomAccessIterator>
inline void sort(RandomAccessIterator first, RandomAccessIterator last) {
   if(first!=last){
       introsort_loop(first, last, value_type(first), lg(last - first) * 2);
       final_insertion_sort(first, last);
    }
}
```

sort() 通过调用 introsort_loop() 函数执行主要的排序逻辑。

lg(last - first) * 2计算出最大允许的递归层数。lg()计算的是二进制对数的值(即log2)，并乘以 2，这是控制分割深度的手段，以防止快速排序出现性能退化的情况。

final_insertion_sort()用于在排序完成后对较小规模的元素进行最终的插入排序。

2. lg()函数

```cpp
template <class Size>
inline Size __lg(Size n) {
    Size k;
    for (k = 0; n > 1; n >>= 1) ++k;
    return k;
}
```

lg()函数计算一个整数n 的二进制对数。它返回 n 的对数 k，其中2^k ≤ n。

这是为了限制快速排序的最大递归深度，避免出现最坏情况的 O(n²) 时间复杂度。

3. introsort_loop()函数

```cpp
template <class RandomAccessIterator, class T, class Size>
void introsort_loop(RandomAccessIterator first, RandomAccessIterator last, T*, Size depth_limit) {
    const int stl_threshold = 16;  // 定义阈值常量，16 个元素以下使用插入排序
    while (last - first > stl_threshold) {  // 如果元素个数大于阈值，继续快速排序
        if (depth_limit == 0) {  // 分割深度用尽时，改用堆排序
            partial_sort(first, last, last);  // 堆排序处理剩下的元素
            return;
        }
        --depth_limit;  // 递归深度减一
        // median-of-3 partition：选择三个元素的中位数作为基准
        RandomAccessIterator cut = unguarded_partition(
            first, last, T(median(*first, *(first + (last - first) / 2), *(last - 1)))
        );
        // 对右半部分递归排序
        introsort_loop(cut, last, value_type(first), depth_limit);
        // 现在回到 while 循环，准备对左半部分递归排序
        last = cut;
    }
}
```

**快速排序阶段**

当前区间的元素个数大于一个预设阈值（stl_threshold=16)，它会继续进行快速排序。使用 median-of-3 partition(三数取中法)来选择枢轴元素，以避免出现最坏情况(即所有元素几乎相同或完全有序）。median-of-3选择first、middle和 last - 1这三个元素的中位数作为枢轴元素，能较好地避免性能退化。

**递归深度限制**

depth_limit用于限制快速排序的递归深度。它的初始值是 lg(n) * 2，每次递归时递减。如果递归深度达到 depth_limit == 0,快速排序的递归过深，算法会改用堆排序（Heap Sort）来避免快速排序最坏情况下的性能退化。

```cpp
final_insertion_sort() 函数
template <class RandomAccessIterator>
void final_insertion_sort(RandomAccessIterator first, RandomAccessIterator last) {
    if (last - first > stl_threshold) {
        insertion_sort(first, first + stl_threshold);
        unguarded_insertion_sort(first + stl_threshold, last);
    } else {
        insertion_sort(first, last);
    }
}
```

final_insertion_sort()在快速排序阶段完成后用于处理剩下的较小区间（通常小于16 个元素）。当区间较小时，插入排序的效率较高。

Introsort算法：SGI STL 的 sort() 实现采用的是 Introsort 算法，它在处理大规模数据时结合了快速排序的高效性、堆排序的稳定性以及插入排序在小规模数据时的优势。

递归深度控制:通过depth_limit限制快速排序的递归深度，防止性能退化为 O(n²)，一旦递归过深，会自动切换到堆排序来保证 O(n log n) 的最坏情况性能。

混合排序:它根据数据的规模和当前的排序状态选择最合适的排序策略，既能保证效率，又能避免最坏情况的出现。

这种实现方式使得STL sort()能够高效处理不同规模和特性的输入数据,并提供稳定的O(nlogn)性能。`inplace_merge` 的作用是将两个相邻的、有序的序列合并成一个有序的序列。例如，假设有两个有序序列 [1, 3, 5] 和 [2, 4, 6]，它们排列在一起形成 [1, 3, 5, 2, 4, 6]。`inplace_merge` 可以将它们合并成 [1, 2, 3, 4, 5, 6]。

<h4 id="7de03803">inplace_merge(应用于有序区间)</h4>
`inplace_merge` 使用了一个辅助缓冲区来提高效率。当缓冲区不足时，它会采用递归分治的策略。

初始化和检查

算法首先会检查两个序列是否为空，如果其中任何一个为空，则不需要任何操作。

```cpp
if(first==middle||middle==last)return;
```

这一步判断两个区间是否为空，若是，则直接返回，无需合并。`inplace_merge` 调用辅助函数 `inplace_merge_aux` 来处理合并过程：

```cpp
inplace_merge_aux(first, middle, last, value_type(first), distance_type(first));
```

辅助函数根据序列长度和缓冲区的大小，选择不同的合并策略。

在`inplace_merge_aux`中，如果缓冲区的大小足够容纳其中一个序列，它会将其中一个序列拷贝到缓冲区中，然后用简单的合并算法完成整个合并过程：

```cpp
if (len1 <= len2 && len1 <= buffer_size) {
    Pointer end_buffer = copy(first, middle, buffer);
    merge(buffer, end_buffer, middle, last, first);
}else if(len2 <= buffer_size) {
    Pointer end_buffer = copy(middle, last, buffer);
    merge_backward(first, middle, buffer, end_buffer, last);
}
```

+ **Case 1: **如果缓冲区足够容纳序列一,则将序列一复制到缓冲区中，之后将缓冲区中的数据与序列二进行合并。
+ **Case 2: **如果缓冲区足够容纳序列二，则将序列二复制到缓冲区中，之后将序列一与缓冲区中的数据合并。

当缓冲区不足以容纳任何一个序列时，`inplace_merge` 采用递归分治的策略。它将较长的序列一分为两半，然后递归调用自己来继续处理。

```cpp
BidirectionalIterator first_cut = first;
BidirectionalIterator second_cut = middle;
Distance len11 = 0;
Distance len22 = 0;
// 序列二较长if (len2 > len1) {
    len22 = len2 / 2;
    advance(second_cut, len22);
    first_cut = upper_bound(first, middle, *second_cut);
    distance(first, first_cut, len11);
}
```

算法根据较长的序列找到分割点，然后对分割后的两部分递归地调用 `merge_adaptive`。

在找到分割点后，算法会调用 `rotate_adaptive` 将两个部分旋转合并。这使得下一个递归调用可以在原地继续合并操作。

`BidirectionalIterator new_middle = rotate_adaptive(first_cut, middle, second_cut, len1 - len11, len22, buffer, buffer_size);`

`rotate_adaptive` 是 `rotate` 的一种优化版本，它会尝试使用缓冲区，如果缓冲区不够大，则直接调用 `rotate`。

（ 这个地方看不懂 ）

<h4 id="nth_elem-ent">nth_elem ent</h4>
`nth_element`是找到一个第 n 大元素，并且重新排列序列，使得这个元素在正确的位置上，同时满足以下条件：

+ **左侧子区间**：所有元素都小于或等于这个“第 n 大”元素。
+ **右侧子区间**：所有元素都大于或等于这个“第 n 大”元素。
+ **次序无关**：它不会保证两个子区间内部的元素顺序，这与 `sort` 或 `partial_sort` 不同。

这使得 `nth_element` 更像是一个改进版的 `partition`，而不是 `sort`。

假设有一个序列：`{22, 30, 30, 17, 33, 40, 17, 23, 22, 12, 20}`

执行以下操作：

`nth_element(iv.begin(), iv.begin() + 5, iv.end());`

目标是将序列重新排列，使得第 5 个位置上的元素（即索引 5）是这个序列排序后应该在的位置。算法执行完成后，结果可能是：

`{20, 12, 22, 17, 17, 22, 23, 30, 30, 33, 40}`

`iv[5]` 上的元素是 `22`，并且它与完全排序后的序列中同一位置上的元素值相同。然而，左侧和右侧部分的元素顺序并没有完全排序，只保证满足“左侧所有元素小于等于 22，右侧所有元素大于等于 22”。

`nth_element` 的实现思路借鉴了快速排序中的**分区算法（partitioning）**，采用了**median-of-3 partitioning** 技术。具体来说：

1. **Median-of-3 partitioning**：算法从序列的开头、中间和结尾选取三个元素，计算它们的中值（即三者中居中的那个值），并将该中值作为“枢轴（pivot）”。
2. **分割序列**：以枢轴为中心，将序列分为左（小于枢轴）和右（大于枢轴）两部分。
3. **递归分治**：判断 `nth` 元素在左侧还是右侧，并递归地对相关部分继续执行相同操作，直到序列长度小于或等于 3。
4. **插入排序**：当序列长度减小到 3 或以下时，使用插入排序（insertion sort）完成排序，因为对于小规模的数据，插入排序更加高效。这种方法的效率高于完全排序，因为它只对涉及的部分进行重新排列。

<h4 id="merge-sort">merge sort</h4>
可以使用inplace_merge来实现merge sort

```cpp
template <class BidirectionalIter>
void mergesort(BidirectionalIter first, BidirectionalIter last) {
    // 计算序列长度
    typename iterator_traits<BidirectionalIter>::difference_type n = distance(first, last);
    // 如果序列长度为 0 或 1，则序列已经有序，无需进一步操作
    if (n <= 1)
        return;
    // 找到序列的中点，将序列对半分割
    BidirectionalIter mid = first + n / 2
    // 对左半部分递归进行归并排序
    mergesort(first, mid);
    // 对右半部分递归进行归并排序
    mergesort(mid, last);
    // 将两个已经排序的子序列合并成一个有序序列
    inplace_merge(first, mid, last);
}
```

归并排序和快速排序的时间复杂度都是 O(Nlog⁡N)，但是它们的性能在实际应用中会有所不同：

+ **空间复杂度**：归并排序通常需要额外的内存空间来存储合并过程中的中间结果，而快速排序可以在原地进行，不需要额外的存储。这使得快速排序在内存管理上更有优势。
+ **数据移动成本**：由于归并排序需要将元素复制到临时缓冲区中，这会增加数据移动的成本。而快速排序由于在原地进行，因此数据移动较少。

<h2 id="c19bb4bd">第七章 仿函数</h2>
**什么是仿函数(函数对象)？为什么不直接用函数指针？**

可以当函数使用的对象，一个类或结构体，重载圆括号操作符 "()"，可以像函数一样被调用。仿函数对象可以当成参数传入函数中。

```cpp
class/struct Obj{
  operator()(args){...}
};
Obj obj;
//方法1
obj(args);
//方法2
Obj()(args);
```

为什么重载()运算符可以调用？C++的规定

STL(标准模板库)中，使用仿函数而不是简单的函数指针：

1. **封装性：**仿函数可以封装状态，它们可以拥有数据成员。仿函数可以在多次调用之间保持一些内部状态。函数指针只能指向一个独立的函数，这个函数没有自己的状态。
2. **灵活性：**由于仿函数是一个类，可以面向对象编程的特性。比如可以定义多个仿函数类来完成不同的任务，或者通过继承来扩展已有的仿函数类，这样可以更容易地组合和复用代码。
3. STL中的许多组件（如容器、算法和迭代器）都是设计来协同工作的。仿函数可以更好地融入这种设计模式，尤其是当涉及到适配器（adapters）和其他高级特性时。仿函数可以被设计成STL组件的一部分，使得它们可以轻松地与其他STL组件交互。(需要理解)

**比较大小的仿函数std::greater和std::less**

```cpp
template<typename _Tp>
struct greater{
   bool operator()(const _Tp& __x, const _Tp& __y) const
   { return __x > __y; }
};
template<typename _Tp>
struct less{
     bool operator()(const _Tp& __x, const _Tp& __y) const
      {return __x < __y;}
 };
```

```cpp
greater<int>gt;
 cout<<gt(2,3); //0
```

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742810343984-9399760e-598d-4f72-910d-930bcb8e9140.png)

STL 仿函数的分类，若以操作数(operand)的个数划分，可分为一元和二元仿函数，若以功能划分，可分为算术运算(Arithmetic)、关系运算(Rational)、逻辑运算(Logical)三大类。任何应用程序欲使用 STL 内建的仿函数，都必须包含 <functional> 头文件，SGI 则将它们实际定义于<stl_function.h>文件中。

<h4 id="f396a296">可配接(Adaptable)的关键</h4>
仿函数的相应型别主要用来表现函数参数型别和传回值型别。为了方便起见,<stl_function.h>定义了两个 classes,分别代表一元仿函数和二元仿函数(STL不支持三元仿函数),其中没有任何数据成员和成员函数只有一些型别定义。

任何仿函数，只要选择继承其中一个 class,就拥有那些相应型别,也就自动拥有了配接能力。

<h5 id="ac3b1e42">一元仿函数：unary_function</h5>
unary_function用来呈现一元函数的参数型别和回返值型别。其定义非常简单：

```cpp
// STL规定，每一个 Adaptable Unary Function 都应该继承此类别。
template <class Arg,class Result>
struct unary_function {
  typedef Arg argument_type; //表示仿函数的参数类型.
  typedef Result result_type;//表示仿函数的返回类型.
}
```

```cpp
template <classT>structnegate : public unary_function<T, T> {
    T operator()(const T& x)const{ return -x; }
};
```

`negate`继承了`unary_function<T, T>`，说明它的参数类型和返回类型都是 `T`。`operator()` 是重载的函数调用运算符，使得这个结构体可以像函数一样被调用。它接受一个参数并返回其负值。

**仿函数适配器unary_negate**

```cpp
template <class Predicate>
class unary_negate {
public:
    bool operator()(const typename Predicate::argument_type& x) const {
        // 逻辑负值操作
    }
};
```

通过组合已有的仿函数来创建新的行为。在这种情况下，`unary_negate` 将一个仿函数的逻辑输出取反。这种方式可以避免重复编写逻辑负值的代码，而是通过组合来实现新功能。

<h5 id="55445a83">二元仿函数：binary_function</h5>
```cpp
template <class Arg1, class Arg2, class Result>
struct binary_function {
    typedef Arg1 first_argument_type;//二元仿函数的第一个参数类型
    typedef Arg2 second_argument_type;//二元仿函数的第二个参数类型
    typedef Result result_type;//二元仿函数的返回值类型
};
```

```cpp
template<class T>
struct plus : public binary_function<T, T, T> {
     T operator()(const T& x, const T& y) const { return x + y; }
};
```

`plus`继承了`binary_function<T, T, T>`，表示两个参数和返回值都是类型 `T`。`operator()` 是重载的函数调用运算符，使得 `plus`仿函数可以像普通函数一样使用，它接受两个参数并返回它们的和。

**仿函数适配器 **`**binder1st**`

```cpp
template <class Operation>
class binder1st {
protected:
    Operation op;
    typename Operation::first_argument_type value;
public:
    typename Operation::result_type
    operator()(const typename Operation::second_argument_type& x) const {
        return op(value, x);
    }
};
```

`binder1st` 适配器通过将二元仿函数的第一个参数绑定为固定值，将其转换为一元仿函数。

<h4 id="0afd78d6">算术类仿函数</h4>
STL 内建的“算术类仿函数”,支持加法、减法、乘法、除法、模数(余数,modulus)和否定(negation)运算。除了“否定”运算为一元运算，其它都是二元运算。

加法(plus<T>)，减法(minus<T>)，乘法(multiplies<T>)，除法(divides<T>)，模取(modulus<T>)，否定(negate<T>)

```cpp
template <class T>
struct plus : public binary_function<T, T, T> {
   T operator()(const T& x, const T& y) const { return x + y; }
};

template <class T>
struct minus : public binary_function<T, T, T> {
    T operator()(const T& x, const T& y) const { return x - y; }
};

template <class T>
struct multiplies : public binary_function<T, T, T> {
    T operator()(const T& x, const T& y) const { return x * y; }
};

template <class T>
struct divides : public binary_function<T, T, T> {
    T operator()(const T& x, const T& y) const { return x / y; }
};

template <class T>
struct modulus : public binary_function<T, T, T> {
    T operator()(const T& x, const T& y) const { return x % y; }
};

template <class T>
struct negate : public unary_function<T, T> {
    T operator()(const T& x) const { return -x; }
};
```

<h4 id="666ea9de">关系运算类仿函数</h4>
STL 内建的“关系运算类仿函数”支持了等于、不等于、大于、大于等于、小于、小于等于六种运算。每一个都是二元运算。

等于(equal_to<T>)，不等于(not_equal_to<T>)，大于(greater<T>)，大于或等于(greater_equal<T>)，小于less<T>，小于或等于less_equal<T>。

```cpp
template <class T>
struct equal_to : public binary_function<T, T, bool> {
    bool operator()(const T& x, const T& y) const { return x == y; }
};

template <class T>
struct not_equal_to : public binary_function<T, T, bool> {
    bool operator()(const T& x, const T& y) const { return x != y; }
};

template <class T>
struct greater : public binary_function<T, T, bool> {
    bool operator()(const T& x, const T& y) const { return x > y; }
};

template <class T>
struct less : public binary_function<T, T, bool> {
    bool operator()(const T& x, const T& y) const { return x < y; }
};

template <class T>
struct greater_equal : public binary_function<T, T, bool> {
    bool operator()(const T& x, const T& y) const { return x >= y; }
};
template <class T>
struct less_equal : public binary_function<T, T, bool> {
    bool operator()(const T& x, const T& y) const { return x <= y; }
};
```

<h4 id="14550353">证同(identity)、选择(select)、投射(project)</h4>
`**identity**`** 仿函数**

```cpp
struct identity : public unary_function<T, T> {
    const T& operator()(const T& x) const { return x; }
};
```

`identity` 仿函数直接返回其输入参数，不做任何修改。

`**select1st**`**仿函数**

```cpp
template <class Pair>
struct select1st : public unary_function<Pair, typename Pair::first_type> {
    const typename Pair::first_type& operator()(const Pair& x) const {
        return x.first;
    }
};
```

接受一个 `pair`，并返回其第一个元素（first）。

`**select2nd**`** 仿函数**

```cpp
template <class Pair>
struct select2nd : public unary_function<Pair, typename Pair::second_type> {
    const typename Pair::second_type& operator()(const Pair& x) const {
        return x.second;
    }
};
```

`**project1st**`** 仿函数**

```cpp
template <class Arg1, class Arg2>
struct project1st : public binary_function<Arg1, Arg2, Arg1> {
    Arg1 operator()(const Arg1& x, const Arg2&) const { return x; }
};
```

接受两个参数，只返回第一个参数，忽略第二个参数。

`**project2nd**`** 仿函数**

```cpp
template <class Pair>
struct select2nd : public unary_function<Pair, typename Pair::second_type> {
    const typename Pair::second_type& operator()(const Pair& x) const {
        return x.second;
    }
};
```

接受两个参数，只返回第二个参数，忽略第一个参数。

这些仿函数在泛型编程中提供了抽象化和间接性。

<h2 id="1279dfeb">第八章 适配器</h2>
[https://blog.csdn.net/jnu_simba/article/details/9530341?ops_request_misc=%257B%2522request%255Fid%2522%253A%25221F7B0931-69B8-4419-BC49-3DB473446D2B%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fcommercial.%2522%257D&request_id=1F7B0931-69B8-4419-BC49-3DB473446D2B&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~insert_commercial~default-3-9530341-null-null.142^v100^pc_search_result_base4&utm_term=%E5%87%BD%E6%95%B0%E9%80%82%E9%85%8D%E5%99%A8&spm=1018.2226.3001.4187](https://blog.csdn.net/jnu_simba/article/details/9530341?ops_request_misc=%257B%2522request%255Fid%2522%253A%25221F7B0931-69B8-4419-BC49-3DB473446D2B%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fcommercial.%2522%257D&request_id=1F7B0931-69B8-4419-BC49-3DB473446D2B&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~insert_commercial~default-3-9530341-null-null.142^v100^pc_search_result_base4&utm_term=%E5%87%BD%E6%95%B0%E9%80%82%E9%85%8D%E5%99%A8&spm=1018.2226.3001.4187)

适配器(adapter)是一种设计模式，将一个 class 的接口转换为另一个 class 的接口，使原本因接口不兼容而不能合作的对象可以一起运作。适配器之所以能够工作，是因为adapter内部持有对象的副本，副本可以是容器、迭代器、流或者一个仿函数。通过对这个对象的封装，适配器可以控制对象的行为，并对其输入和输出进行必要的调整。这样，适配器就能够将原本不兼容的接口或功能转化成一个统一的形式，使其能够在标准库算法中使用。

比如container adaper内部定义了一个指向container的指针，（逆向）迭代器适配器内部定义了一个迭代器对象。

IOStream adaper内部定义了一个指向stream对象的指针 

<h4 id="4fb6891f">配接器分类</h4>
<h5 id="5d43d8f0">容器适配器</h5>
用来扩展7种基本容器，利用基本容器（deque，heap）扩展形成了stack、queue和prioriory_queue。

<h5 id="1d949b6a">迭代器适配器</h5>
反向迭代器 reverse iterators，插入迭代器 insert iterators，IO流迭代器 iostream iterators)。

<h6 id="insert-iterators">**insert iterators**</h6>
阅读材料:[https://blog.csdn.net/qq_41453285/article/details/104375418?ops_request_misc=%257B%2522request%255Fid%2522%253A%25223A93E96C-958D-45A3-8711-AE6DE3863185%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=3A93E96C-958D-45A3-8711-AE6DE3863185&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-2-104375418-null-null.142^v100^pc_search_result_base4&utm_term=iterator%20adapters&spm=1018.2226.3001.4187](https://blog.csdn.net/qq_41453285/article/details/104375418?ops_request_misc=%257B%2522request%255Fid%2522%253A%25223A93E96C-958D-45A3-8711-AE6DE3863185%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=3A93E96C-958D-45A3-8711-AE6DE3863185&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-2-104375418-null-null.142^v100^pc_search_result_base4&utm_term=iterator%20adapters&spm=1018.2226.3001.4187)

insert iterators，可以将一般迭代器的赋值操作转换为插入操作 。<font style="background-color:#fbf5b3;">每一个insert iterators内部都维护一个容器(由用户指定)。容器有自己的迭代器，对insert iterators做赋值时，就在insert iterators中被转换为对该容器的迭代器做插入操作。在insert iterators的operator=操作符中调用底层容器的push_front()、push_back()或insert() 操作函数</font>，至于其他的迭代器行为例如:operator++,operator*,operator--,operator->都被关闭。

insert itertators的前进、后退、取值、成员取用等操作都是没有意义的。

insert itertators 专用于尾端插入操作的back_insert_iterator，专用于头端插入操作front_insert_iterator，在任意位置插入操作的insert_iterator。

由于上面三个iterator adapters的使用接口不是很直观，STL提供下图所示的三个函数。

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742810344051-1d8d3937-6e52-4072-8386-02233f7f1e4c.png)

**back_insert_iterator类、back_inserter()函数**

```cpp
// 这是一个迭代器配接器,用来将某个迭代器的赋值操作改为插入操作,改为从容器的尾端插入
template<class Container>
class back_insert_iterator{
protected:
    Container* container;//底层容器
public:
   typedef out_iterator_tag iterator_category;//注意类型
   typedef void             value_type;
   typedef void             difference_type;
   typedef void             pointer;
   typedef void             reference;
    //构造函数使back_insert_iterator与容器绑定起来
 explicit back_insert_iterator(Container& x):container(&x) {}
 back_insert_iterator<Container>& operator=(const typename Container::value_type& value){
        container->push_back(value);//这里是关键，直接调用push_back()
        return *this;
    }
    //下面的操作符对back_insert_iterator不起作用(关闭功能)
    //因此都返回自己
    back_insert_iterator<Container>& operator*() { return *this; }
    back_insert_iterator<Container>& operator++() { return *this; }
    back_insert_iterator<Container>& operator++(int) { return *this; }
};
```

在`back_insert_iterator`类中，`container`成员变量是指向容器的一个指针。

**front_insert_iterator类、front_inserter()函数**

```cpp
//一个迭代器配接器，用来将某个迭代器的赋值操作改为插入操作——改为从容器的头部插入
//注意，该容器不支持vector，因为vector没有提供push_front()函数。
template<class Container>
class front_insert_iterator
{
protected:
    Container* container; //底层容器
public:
    typedef out_iterator_tag iterator_category; //注意类型
    typedef void             value_type;
    typedef void             difference_type;
    typedef void             pointer;
    typedef void             reference;
    //构造函数使back_insert_iterator与容器绑定起来
    explicit front_insert_iterator(Container& x) :container(&x) {}
    front_insert_iterator<Container>& operator=(const typename Container::value_type& value)
    {
        container->push_front(value);//这里是关键，直接调用push_front()
        return *this;
    }
    //下面的操作符对front_insert_iterator不起作用(关闭功能)
    //因此都返回自己
    front_insert_iterator<Container>& operator*() { return *this; }
    front_insert_iterator<Container>& operator++() { return *this; }
    front_insert_iterator<Container>& operator++(int) { return *this; }
};
```

```cpp
//这是一个辅助函数，方便我们使用front_insert_iterator
template<class Container>
inline front_insert_iterator<Container> front_inserter(Container& x){
    return front_insert_iterator<Container>(x);
}
```

**insert_iterator类、inserter()函数**

```cpp
//这是一个迭代器配接器，用来将某个迭代器的赋值操作改为插入操作——在任意位置上插入
//并将迭代器右移一个位置——如此便可以方便地连续插入
//表面上是赋值操作，实际上是插入操作
template<class Container>
class insert_iterator{
protected:
     Container* container; //底层容器
     typename Container::iterator iter;
public:
    typedef out_iterator_tag iterator_category; //注意类型
    typedef void             value_type;
    typedef void             difference_type;
    typedef void             pointer;
    typedef void             reference;
    insert_iterator(Container& x, typename Container::iterator i) 
        :container(&x), iter(i) {}
    insert_iterator<Container>& operator=(const typename Container::value_type& value){
       iter = container->inserter(iter, value); //关键，直接调用insert()
       ++iter; //使insert iterator永远随其目标而移动
       return *this;
    } 
    //下面的操作符对insert_iterator不起作用(关闭功能)
    //因此都返回自己
    insert_iterator<Container>& operator*() { return *this; }
    insert_iterator<Container>& operator++() { return *this; }
    insert_iterator<Container>& operator++(int) { return *this; }
};
```

<h6 id="reverse-iterators">**reverse Iterators**</h6>
+ Reverse Iterators，就是将一般迭代器的前进方向逆转：
    - 使原本应该前进的operator++变为后退操作
    - 使原本应该后退的operator--编程前进操作
+ 如果STL算法接受的不是一般的迭代器，而是这种逆向迭代器，它就会从尾到头的方向来处理序列中的元素。

```cpp
使用迭代器模式理解这个过程：

//迭代器配接器，用来将某个迭代器逆反前进方向
template<class Iterator>
class reverse_iterator{
protected:
   Iterator current;  /记录对应的正向迭代器
public:
    //5中与迭代器相关的类型
 typedef typname iterator_traits<Iterator>::iterator_category iterator_category;
 typedef typname iterator_traits<Iterator>::value_type value_type;
    typedef typname iterator_traits<Iterator>::difference_type difference_type;
    typedef typname iterator_traits<Iterator>::pointer pointer;
    typedef typname iterator_traits<Iterator>::reference reference;
    typedef Iterator iterator_type;          //正向迭代器
    typedef reverse_iterator<Iterator> self; //逆向迭代器
public:
    reverse_iterator() {}
    //将reverse_iterator与某个迭代器x关联起来
   explicit reverse_iterator(iterator_type x) :current(x) {}
   reverse_iterator(const self& x) :current(x.current) {}
   iterator_type base()const { return current; }//取出对应的正向迭代器
   reference operator*()const {
        Iterator tmp = current;
        return  *--tmp;
        //以上关键在于。对逆向迭代器取值，“对应的正向迭代器”后退一位取值
    }
   pointer operator->()const { return &(operator*()); }//意义同上 
//前进变后退
   self& operator++() {
       --current; ++*this;
   }
   self operator++(int) {
       self tmp = *this; --current; return tmp;
   }
//后退变前进
   self& operator--() {
       ++current; ++*this;
   }
    self operator--(int) {
        self tmp = *this;
        ++current;
        return tmp;
    }
//前进与后退方向逆转
    self operator+(difference_type n)const {
        return self(current - n);
    }
    self& operator+=(difference_type n) {
        current -= n;
        return *this;
    }
    self operator-(difference_type n)const {
        return self(current + n);
    }
    self& operator-=(difference_type n) {
        current += n;
        return *this;
    }
    //下面第一个*和唯一一个+都会调用本类的operator*和operator+
    //第二个*则不会
    reference operator[](difference_type n)const { return*(*this + n); }
};
```

<h6 id="iostream-iterators">**IOStream Iterators**</h6>
IOStream Iterators将输入和输出流（如 `std::cin`、`std::cout`、文件流等）适配成符合标准迭代器接口的对象，使这些流可以与 C++ 标准库算法无缝地协同工作。

（1）绑定istream对象(如cin)的迭代器称为istream_iterator，istream_iterator将输入流适配成一个输入迭代器，使其可以像普通的迭代器一样通过递增操作（`++`）读取数据并通过解引用（`*`）访问当前值。

（2）绑定ostream(如cout)的迭代器称为ostream_iterator，将输出流适配成一个输出迭代器，使其可以像普通的输出迭代器一样通过赋值操作将数据写入流中，这些迭代器将它们对应的流当做一个特定类型的元素序列来处理。

**istream_iterator**

源码简化

```cpp
template <class T, class Distance = ptrdiff_t>
class istream_iterator {
    friend bool operator==(/* 参数 */); // 用于比较两个迭代器是否相等
protected:
    istream* stream;    // 指向输入流的指针
    T value;            // 当前读取的值
    bool end_marker;    // 标记是否到达输入的结束
    void read();        // 用于从输入流读取数据
public:
    // 定义了一些迭代器的类型
    typedef input_iterator_tag iterator_category;
    typedef T value_type;
    typedef Distance difference_type;
    typedef const T* pointer;
    typedef const T& reference;
    // 构造函数
    istream_iterator();
    istream_iterator(istream& s);
    // 迭代器的解引用操作
    reference operator*() const;
    pointer operator->() const;
    // 迭代器的前置和后置递增操作
    istream_iterator<T, Distance>& operator++();
    istream_iterator<T, Distance> operator++(int);
};
```

istresm iterator对istream对象做了封装，因为是STL迭代器，所以需要定义一些迭代器类型相关的变量（C++ 标准库对迭代器类型的要求。标准库中定义的迭代器类型通常会包含一组类型别名，用于描述迭代器的性质和行为），然后重写解引用，++等操作（调用istream对象)。

**创建与初始化**

创建一个输入流迭代器时，必须指定迭代器将要读取的数据的类型。因为istream_iterator使用>>读取流，因此创建istream_iterator对象时必须使用输入流对象初始化它。当然也可以默认初始化迭代器，这样创建一个可以当做尾后值使用的迭代器。

```cpp
#include <iostream>
#include <fstream>
#include <iterator>
#include <string>
int main() {
    // 从标准输入读取整数
    std::istream_iterator<int> int_it(std::cin); // 使用 cin 初始化读取整数的迭代器
    std::istream_iterator<int> int_eof;          // 尾后迭代器，用于检测输入结束
    std::cout << "Reading integers from standard input (type any non-integer to stop):\n";
    while (int_it != int_eof) {
        std::cout << "You entered: " << *int_it << std::endl;
        ++int_it; // 递增迭代器，读取下一个整数
    }
    // 从文件中读取字符串
    std::ifstream in("afile.txt");
    if (!in) {
        std::cerr << "Failed to open the file.\n";
        return 1;
    }
    std::istream_iterator<std::string> str_it(in);  // 使用文件流初始化读取字符串的迭代器
    std::istream_iterator<std::string> str_eof;     // 尾后迭代器，用于检测文件读取结束
    std::cout << "Reading strings from file 'afile.txt':\n";
    while (str_it != str_eof) {
        std::cout << "Read string: " << *str_it << std::endl;
        ++str_it; // 递增迭代器，读取下一个字符串
    }
    return 0;
}
```

尾后迭代器是指向容器末尾的一个位置的迭代器，这个位置并不存储任何实际的数据。它主要用于标识容器的结束，通常用于在循环或算法中检测是否已经遍历完所有元素。因为流迭代器类似于一个容器，保存了我们输入的数据,因此可以使用下面的方式对一个vec进行构造。

```cpp
//阻塞在此处，从cin中读取数据
std::istream_iterator<int> in_iter(std::cin);
std::istream_iterator<int> eof; 
//从迭代器范围构造vec
std::vector<int> vec(in_iter, eof);
for (auto val : vec) { std::cout << val << " "；}
```

我们可以在算法中使用istream_iterator迭代器，如accumulate()算法的参数1和参数2表示一个迭代器范围，参数3为初始值，此算法将迭代器所指的范围内的元素与初始值加在一起返回总和。

```cpp
//阻塞在此处，从cin中读取数据
std::istream_iterator<int> in_iter(std::cin);
std::istream_iterator<int> eof;
将in_iter, eof内的元素做总和，以0为初始值
std::cout << std::accumulate(in_iter, eof, 0) << std::endl;
```

**ostream_iterator**

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742810344541-d8b53b12-1315-4179-acd7-fadd8c6a2f40.png)

源码简化

```cpp
template <class T>
class ostream_iterator {
protected:
    ostream* stream;           // 指向输出流的指针
    const char* string;        // 每次输出后的间隔符号
public: 
//定义了迭代器的类型
    typedef output_iterator_tag iterator_category;
    typedef void value_type;
    typedef void difference_type;
    typedef void pointer;
    typedef void reference;
//构造函数
    ostream_iterator(ostream& s) : stream(&s), string(0) {}
    ostream_iterator(ostream& s, const char* c) : stream(&s), string(c) {}
//赋值操作符，用于将数据输出到流中
    ostream_iterator<T>& operator=(const T& value) {
        *stream << value;        // 输出数据到流
        if (string) *stream << string;  // 输出间隔符
        return *this;
    }
//以下操作符对输出迭代器没有实际效果，但需要提供以符合迭代器接口
    ostream_iterator<T>& operator*() { return *this; }
    ostream_iterator<T>& operator++() { return *this; }
    ostream_iterator<T>& operator++(int) { return *this; }
};
```

ostream iterator封装了ostream对象，然后根据STL对迭代器标准定义了一些类型变量，然后重载赋值运算符将数据输出到ostream中，*，++都没有效果。

**创建与初始化**

创建一个输出流迭代器时，必须指定迭代器将要输出的数据的类型。ostream_iterator使用<<输出数据，因此创建ostream_iterator对象时其参数1必须指定一个输出流对象，另外其还有一个参数2，这个参数是一个字符串类型（必须是C风格的字符串），在输出每个元素后都会打印此字符串。另外不允许有类似于istream_iterator的尾后迭代器，因此不允许有默认初始化。

```cpp
std::vector<int> vec{ 1,2,3 };
//输出流迭代器操作的元素类型为int，使用cout作为初始化
std::ostream_iterator<int> out_iter(std::cout, " ");
//for循环每执行一次，从vec中取出一个元素赋值给out_iter，然后out_iter进行输出
for (auto val : vec) {
    out_iter = val;
}
std::cout << std::endl;
```

可以在算法中使用，使用copy()算法将vector的元素拷贝进输出流迭代器对象中进行输出。

```cpp
std::vector<int> vec{ 1,2,3 };
std::ostream_iterator<int> out_iter(std::cout, " ");
std::copy(vec.begin(), vec.end(), out_iter);
std::cout << std::endl;
```

三种迭代器的综合使用：

```cpp
#include<iterator>  // for iterator adapters
#include<deque>
#include<algorithm> // for copy, find
#include<iostream>
using namespace std;
int main() {
    // 创建一个 ostream_iterator 绑定到 cout，每次输出元素后跟一个空格
    ostream_iterator<int> outite(cout, " ");
    int ia[] = {0, 1, 2, 3, 4, 5};
    deque<int> id(ia, ia + 6);
    // 将所有元素拷贝到 outite (即拷贝到 cout)
    copy(id.begin(), id.end(), outite); // 输出: 0 1 2 3 4 5
    cout << endl;
    // 使用 front_insert_iterator 将 ia 的部分元素拷贝到 id 内
    // 注意：front_insert_iterator 会将 assign 操作改为 push_front 操作
    copy(ia + 1, ia + 2, front_inserter(id));
    copy(id.begin(), id.end(), outite); // 输出: 1 0 1 2 3 4 5
    cout << endl;
    // 使用 back_insert_iterator 将 ia 的部分元素拷贝到 id 内
    copy(ia + 3, ia + 4, back_inserter(id));
    copy(id.begin(), id.end(), outite); // 输出: 1 0 1 2 3 4 5 3
    cout << endl;
    // 搜寻元素 5 所在位置
    deque<int>::iterator ite = find(id.begin(), id.end(), 5)；
    // 使用 insert_iterator 在找到的位置前插入 ia 的部分元素
    copy(ia + 0, ia + 3, inserter(id, ite));
    copy(id.begin(), id.end(), outite); // 输出: 1 0 1 2 3 4 0 1 2 5 3
    cout << endl;
    // 将所有元素逆向拷贝到 outite
    copy(id.rbegin(), id.rend(), outite); // 输出: 3 5 2 1 0 4 3 2 1 0 1
    cout << endl;
    // 创建一个 istream_iterator 绑定到 cin，直到遇到 end-of-stream
    istream_iterator<int> inite(cin), eos; // eos: end-of-stream
    // 从标准输入读取整数并插入到 id 的开始位置
    copy(inite, eos, front_inserter(id));
    // 输出最终的 deque 内容
    copy(id.begin(), id.end(), outite);
    cout << endl;
    return 0;
}
```

<h4 id="377a1944">函数适配器 </h4>
函数适配器能够将仿函数和另一个仿函数(或某个值或某个一般函数)结合起来。

```cpp
// 计算奇数元素的个数 这里的bind2nd将二元函数对象modulus转换为一元函数对象。
// bind2nd(op,value)(param) 相当于 op(param, value)
cout<<count_if(v.begin(), v.end(),bind2nd(modulus< int>(),  2))<< endl;
```

在STL(标准模板库)中,仿函数适配器(functor adapter)是一种特殊的类模板，可以修改现有的仿函数(functor)的行为，包括绑定参数、否定执行结果或组合多个函数等。

仿函数是重载了 operator() 的类对象，它可以像普通函数一样被调用。而仿函数适配器则是在原有仿函数的基础上添加了一层抽象，使得可以预设一些行为，预设的行为将在实际使用时影响仿函数的操作。

常见的仿函数适配器

Binder：std::bind1st 和 std::bind2nd 是两个用于绑定仿函数参数的适配器。它们允许将一个二元仿函数转换为一元仿函数，通过固定其中一个参数来达到目的。

例如,如果有一个比较两个整数的仿函数 less<int>(),可以使用bind2nd(less<int>(),12)来创建一个新的仿函数，这个新仿函数会检查其输入是否小于 12。

Negator：std::not1 和 std::not2 是用来对仿函数的结果取反的适配器。如果原始仿函数返回布尔值，那么使用这些适配器后，仿函数将会返回相反的布尔值。

Composer：std::unary_negate 和 std::binary_negate 可以用来构造新的仿函数，该仿函数会对原仿函数的结果进行逻辑非操作。另外，还有其他更复杂的组合方式，比如使用 std::compose1 和 std::compose2 来组合两个仿函数，形成一个新的复合仿函数。

Other Adapters：还有一些其他的适配器，如 std::ptr_fun 可以将普通的函数指针转换为仿函数，这样就可以让普通的C风格函数也能与STL算法一起工作。

当使用仿函数适配器时，实际上是在创建一个新的对象，这个对象内部持有一个指向原来仿函数的引用或副本。当这个适配器对象被调用时，它会按照预设的方式调用内部持有的仿函数，并根据需要处理参数和返回值。因此，虽然看起来像是在调用适配器对象，但实际上是在间接地调用并控制着原来的仿函数。

![](https://cdn.nlark.com/yuque/0/2025/png/45533403/1742810344477-b3525ceb-7bcc-49da-ae54-541cdcb6c702.png)

<h4 id="1f7f270e">对返回值进行逻辑否定：not1, not2</h4>
```cpp
// 以下配接器用来表示某个 Adaptable Predicate 的逻辑负值(logical negation)
template <class Predicate>
class unary_negate : public std::unary_function<typename Predicate::argument_type, bool> {
protected:
    Predicate pred; // 内部成员 Predicate
public:
    explicit unary_negate(const Predicate& x) : pred(x) {} // 构造函数
    bool operator()(const typename Predicate::argument_type& x) const { // 重载 () 运算符
        return !pred(x); // 将 pred 的运算结果加上否定(negate)运算
    }
};
// 辅助函数，使我们得以方便使用 unary_negate<Predicate>
template <class Predicate>
inline unary_negate<Predicate> not1(const Predicate& pred) {
    return unary_negate<Predicate>(pred);
}
// 以下配接器用来表示某个 Adaptable Binary Predicate 的逻辑负值
template <class Predicate>
class binary_negate : public std::binary_function<
typename Predicate::first_argument_type,typename Predicate::second_argument_type,
bool>
{
protected:
    Predicate pred; // 内部成员 Predicate
public:
    explicit binary_negate(const Predicate& x) : pred(x) {} // 构造函数
    bool operator()(
        const typename Predicate::first_argument_type& x,
        const typename Predicate::second_argument_type& y
    ) const { // 重载 () 运算符
        return !pred(x, y); // 将 pred 的运算结果加上否定(negate)运算
    }
};
//辅助函数，使我们得以方便使用 binary_negate<Predicate>
template <class Predicate>
inline binary_negate<Predicate> not2(const Predicate& pred) {
    return binary_negate<Predicate>(pred);
}
```

<h4 id="4d780f82">对参数进行绑定：bind1st，bind2nd</h4>
```cpp
template <class Operation>
class binder1st : public std::unary_function<
    typename Operation::second_argument_type,
    typename Operation::result_type>{
protected:
    Operation op; // 内部成员
    typename Operation::first_argument_type value; // 内部成员
public:
    // 构造函数
binder1st(const Operation& x, const typename Operation::first_argument_type& y)
        : op(x), value(y) {} // 将表达式和第一参数记录于内部成员
    // 重载 () 运算符
typename Operation::result_type operator()(
        const typename Operation::second_argument_type& x) const {
        return op(value, x); // 实际调用表达式，并将 value 绑定为第一参数
 }
};
// 辅助函数，使我们得以方便使用 binder1st<Operation>
template <class Operation, class T>
inline binder1st<Operation> bind1st(const Operation& op, const T& x) {
   typedef typename Operation::first_argument_type arg1_type;
   return binder1st<Operation>(op, arg1_type(x)); // 先把x转型为 op 的第一参数型别
}
// 以下配接器用来将某个 Adaptable Binary Function 转换为 Unary Function
template <class Operation>
class binder2nd : public std::unary_function<
    typename Operation::first_argument_type,
    typename Operation::result_type>{
protected:
    Operation op; // 内部成员
    typename Operation::second_argument_type value; // 内部成员
public:
    // 构造函数
    binder2nd(const Operation& x, const typename Operation::second_argument_type& y)
        : op(x), value(y) {} // 将表达式和第二参数记录于内部成员
    // 重载 () 运算符
    typename Operation::result_type operator()(
        const typename Operation::first_argument_type& x) const {
        return op(x, value); // 实际调用表达式，并将 value 绑定为第二参数
    }
};
// 辅助函数，使我们得以方便使用 binder2nd<Operation>
template <class Operation, class T>
inline binder2nd<Operation> bind2nd(const Operation& op, const T& x) {
    typedef typename Operation::second_argument_type arg2_type;
    return binder2nd<Operation>(op, arg2_type(x)); // 先把x转型为 op 的第二参数型别
}
```

<h4 id="e1cabfbb">用于函数合成：compose1，compose2</h4>
```cpp
// 已知两个 Adaptable Unary Functions f(),g(),以下配接器用来产生一个 h(),
// 使 h(x)= f(g(x))
template <class Operation1,class Operation2>
class unary_compose
: public unary_function<typename Operation2::argument_type,
typename Operation1::result_type>{
protected:
  // 内部成员 
   Operation1 op1;
  //内部成员 
   Operation2 op2;
public:
  //constructor
 unary_compose(const Operation1& x, const Operation2& y):op1(x),op2(y){}
 // 将两个表达式记录于内部成员
 typename Operation1::result_type operator()
  (const typename Operation2::argument_type& x) const {
 //函数合成 
    return opl(op2(x));
  }
};

// 辅助函数，让我们得以方便运用 unary_compose<0p1,Op2>
template <class Operation1,class operation2>
inline unary_compose<Operation1,Operation2>
 compose1(const Operation1& opl,const Operation2& op2){
   return unary_compose<0peration1,Operation2>(op1,op2);
}

// 已知一个 Adaptable Binary Function f 和
// 两个 Adaptable Unary Functions g1,g2,
//以下配接器用来产生一个 h,使 h(x)= f(g1(x),g2(x))
template <class Operation1,class Operation2,class Operation3>
class binary_compose：public unary_function<typename Operation2::argument_type,
typename Operationl::result_type> {
protected:
// 内部成员 
  Operation1 op1;
// 内部成员 
  Operation2 op2;
// 内部成员 
  Operation3 op3;
public:
//constructor,将三个表达式记录于内部成员
  binary_compose(const Operation1& x, const Operation2& y,const Operation3& z):
  op1(x),op2(y),op3(z){}
  typename Operation1::result_type
   operator()(const typename Operation2:sargument_type& x) const{
//函数合成 
   return opl(op2(x),op3(x));
}
};
// 辅助函数，让我们得以方便运用 binary_compose<0p1,0p2,0p3>
template <class Operation1,class Operation2,class Operation3>
inline binary_compose<Operation1, Operation2,operation3>
compose2(const Operation1& opl,const Operation2& op2,
const Operation3& op3){
return binary_compose<Operation1,Operation2,Operation3>
(op1,op2,op3)
}
```

**unary_compose类模板**

unary_compose 用来将两个一元函数(Adaptable Unary Functions)组合成一个新的函数。这个新的函数首先应用第二个函数(op2)，然后将结果传递给第一个函数（op1）。这样，新函数 h(x) 的行为是 h(x) = f(g(x))

`**binary_compose**`**类模板**

`binary_compose` 用来将一个二元函数和两个一元函数组合成一个新的函数。这个新的函数首先分别应用两个一元函数（`op2` 和 `op3`），然后将它们的结果作为参数传递给二元函数（`op1`）。这样，新函数 `h(x)` 的行为是 `h(x) = f(g1(x), g2(x))`。

<h4 id="f49791b0">用于函数指针：ptr_fun</h4>
函数指针不能直接传入STL算法的参数，需要使用函数适配器把他包装成仿函数使用。

```cpp
#include <functional> // 为了使用 std::unary_function 和 std::binary_function
// 以下配接器其实就是把一个一元函数指针包起来：
// 当仿函数被使用时，就调用该函数指针
template <class Arg, class Result>
class pointer_to_unary_function : public std::unary_function<Arg, Result> {
protected:
    Result (*ptr)(Arg); // 内部成员，一个函数指针
public:
    pointer_to_unary_function() : ptr(nullptr) {} // 默认构造函数
    // 以下 constructor 将函数指针记录于内部成员之中
    explicit pointer_to_unary_function(Result (*x)(Arg)) : ptr(x) {}
    // 以下，通过函数指针执行函数
    Result operator()(Arg x) const { return ptr(x); }
};
// 辅助函数，让我们得以方便运用 pointer_to_unary_function
template <class Arg, class Result>
inline pointer_to_unary_function<Arg, Result> ptr_fun(Result (*x)(Arg)) {
    return pointer_to_unary_function<Arg, Result>(x);
}
// 以下配接器其实就是把一个二元函数指针包起来；
// 当仿函数被使用时，就调用该函数指针
template <class Arg1, class Arg2, class Result>
class pointer_to_binary_function : public std::binary_function<Arg1, Arg2, Result> {
protected:
    Result (*ptr)(Arg1, Arg2); // 内部成员，一个函数指针
public:
    pointer_to_binary_function() : ptr(nullptr) {} // 默认构造函数
    // 以下 constructor 将函数指针记录于内部成员之中
    explicit pointer_to_binary_function(Result (*x)(Arg1, Arg2)) : ptr(x) {}
    // 以下，通过函数指针执行函数
    Result operator()(Arg1 x, Arg2 y) const { return ptr(x, y); }
};
// 辅助函数，让我们得以方便使用 pointer_to_binary_function
template <class Arg1, class Arg2, class Result>
inline pointer_to_binary_function<Arg1, Arg2, Result> ptr_fun(Result (*x)(Arg1, Arg2)) {
    return pointer_to_binary_function<Arg1, Arg2, Result>(x);
}
```

<h4 id="90d45827">用于成员函数指针：mem_fun,mem_fun_ref</h4>
```cpp
// 基类 Shape
class Shape {
public:
    virtual void display() = 0; // 纯虚函数
    virtual ~Shape() {} // 虚析构函数，确保派生类对象被正确销毁
};
// 派生类 Rect
class Rect : public Shape {
public:
    void display() override { cout << "Rect "; }
};
// 派生类 Circle
class Circle : public Shape {
public:
    void display() override { cout << "Circle "; }
};
// 派生类 Square，继承自 Rect
class Square : public Rect {
public:
    void display() override { cout << "Square "; }
};
int main() {
  vector<Shape*> V;
  V.push_back(new Rect);
  V.push_back(new Circle);
  V.push_back(new Square);
  V.push_back(new Circle);
  V.push_back(new Rect);
    // 使用多态显示形状
  for (int i = 0; i < V.size(); ++i) {
        V[i]->display();
    }
 cout << endl; // 输出: Rect Circle Square Circle Rect
   //使用 for_each 和 mem_fun 显示形状
 for_each(V.begin(), V.end(), mem_fun(&Shape::display));
cout << endl; // 输出: Rect Circle Square Circle Rect
    // 释放动态分配的内存
    for (auto* shape : V) {
        delete shape;
    }
    return 0;
}
```

语法层面不能写：for_each(V.begin(),V.end(),CShape::display);

也不能写：for_each(V.begin(),V.end(),Shape::display);

一定要以配接器 mem_fun 修饰 member function,才能被算法for_each 接受。

```cpp
#include <functional> // 为了使用 unary_function 和 binary_function
// “无任何参数”、“通过 pointer 调用”、“non-const 成员函数”
template <class S, class T>
class mem_fun_t : public std::unary_function<T*, S> {
public:
    explicit mem_fun_t(S (T::*pf)()) : f(pf) {} // 记录下来
    S operator()(T* p) const { return (p->*f)(); } // 转调用
private:
    S (T::*f)(); // 内部成员，pointer to member function
};
//“无任何参数”、“通过 pointer 调用”、“const 成员函数”
template <class S, class T>
class const_mem_fun_t : public std::unary_function<const T*, S> {
public:
    explicit const_mem_fun_t(S (T::*pf)() const) : f(pf) {}
    S operator()(const T* p) const { return (p->*f)(); }
private:
    S (T::*f)() const; // 内部成员，pointer to const member function
};
//“无任何参数”、“通过 reference 调用”、“non-const 成员函数”
template <class S, class T>
class mem_fun_ref_t : public std::unary_function<T, S> {
public:
    explicit mem_fun_ref_t(S (T::*pf)()) : f(pf) {} // 记录下来
    S operator()(T& r) const { return (r.*f)(); } // 转调用
private:
    S (T::*f)(); // 内部成员，pointer to member function
};
//“无任何参数”、“通过 reference 调用”、“const 成员函数”
template <class S, class T>
class const_mem_fun_ref_t : public std::unary_function<T, S> {
public:
    explicit const_mem_fun_ref_t(S (T::*pf)() const) : f(pf) {}
    S operator()(const T& r) const { return (r.*f)(); }
private:
    S (T::*f)() const; // 内部成员，pointer to const member function
};
//“有一个参数”、“通过 pointer 调用”、“non-const 成员函数”
template <class S, class T, class A>
class mem_fun1_t : public std::binary_function<T*, A, S> {
public:
    explicit mem_fun1_t(S (T::*pf)(A)) : f(pf) {} // 记录下来
    S operator()(T* p, A x) const { return (p->*f)(x); } // 转调用
private:
    S (T::*f)(A); // 内部成员，pointer to member function
};
//“有一个参数”、“通过 pointer 调用”、“const 成员函数”
template <class S, class T, class A>
class const_mem_fun1_t : public std::binary_function<const T*, A, S> {
public:
    explicit const_mem_fun1_t(S (T::*pf)(A) const) : f(pf) {}
    S operator()(const T* p, A x) const { return (p->*f)(x); }
private:
    S (T::*f)(A) const; // 内部成员，pointer to const member function
};
//“有一个参数”、“通过 reference 调用”、“non-const 成员函数”
template <class S, class T, class A>
class mem_fun1_ref_t : public std::binary_function<T, A, S> {
public:
    explicit mem_fun1_ref_t(S (T::*pf)(A)) : f(pf) {} // 记录下来
    S operator()(T& r, A x) const { return (r.*f)(x); } // 转调用
private:
    S (T::*f)(A); // 内部成员，pointer to member function
};
//“有一个参数”、“通过 reference 调用”、“const 成员函数”
template <class S, class T, class A>
class const_mem_fun1_ref_t : public std::binary_function<T, A, S> {
public:
    explicit const_mem_fun1_ref_t(S (T::*pf)(A) const) : f(pf) {}
    S operator()(const T& r, A x) const { return (r.*f)(x); }
private:
    S (T::*f)(A) const; // 内部成员，pointer to const member function
};
//mem_fun adapter 的辅助函数：mem_fun, mem_fun_ref
template <class S, class T>
inline mem_fun_t<S, T> mem_fun(S (T::*f)()) {
    return mem_fun_t<S, T>(f);
}
template <class S, class T>
inline const_mem_fun_t<S, T> mem_fun(S (T::*f)() const) {
    return const_mem_fun_t<S, T>(f);
}
template <class S, class T>
inline mem_fun_ref_t<S, T> mem_fun_ref(S (T::*f)()) {
    return mem_fun_ref_t<S, T>(f);
}
template <class S, class T>
inline const_mem_fun_ref_t<S, T> mem_fun_ref(S (T::*f)() const) {
    return const_mem_fun_ref_t<S, T>(f);
}
//具有一个参数的 mem_fun 和 mem_fun_ref 辅助函数
template <class S, class T, class A>
inline mem_fun1_t<S, T, A> mem_fun(S (T::*f)(A)) {
    return mem_fun1_t<S, T, A>(f);
}
template <class S, class T, class A>
inline const_mem_fun1_t<S, T, A> mem_fun(S (T::*f)(A) const) {
    return const_mem_fun1_t<S, T, A>(f);
}
template <class S, class T, class A>
inline mem_fun1_ref_t<S, T, A> mem_fun_ref(S (T::*f)(A)) {
    return mem_fun1_ref_t<S, T, A>(f);
}
template <class S, class T, class A>
inline const_mem_fun1_ref_t<S, T, A> mem_fun_ref(S (T::*f)(A) const) {
    return const_mem_fun1_ref_t<S, T, A>(f);
}
```

