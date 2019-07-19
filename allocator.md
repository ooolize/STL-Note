### 空间配置器

STL空间配置器在头文件<memory>中，其中主要是allocator类起作用，

```c++ allocator<T> alloc```

+ alloc.allocate(n) 分配n个类型为T元素的原始空间。
+ alloc.construct(p,value) 在原始内存指针p处构建元素,将arg传递给构造函数。
+ alloc.destory(p) 将p指针所指的元素T销毁
+ alloc.deallocate(p,n) 释放从p开始的n个T元素的内存空间,p须是allocate返回的原始内存指针，n也是当时指定的大小。
+ uninitialized_copy(b,e,b1) 将b,e区间内元素赋值到原始指针b1开始的地方，b1以后需保证足够大。
+ uninitialized_fill(b,e,t) 原始内存区间b,e内全部赋值为t。

```c++
allocator<int> a;
auto p=a.allocate(vi.size()*2);
auto q=uninitialized_copy(v1.begin(),v1.end(),p);
auto m=uninitialized_fill(q,v1.size(),72);
while(m!=p)
destory(--m);
```

STL默认使用alloc作为空间配置器，注意它不是template的，无须指定类型

```c++ 
template<class T,class Alloc=alloc>
class vector{...};
```

1. construct()

构造函数很简单，调用operator new运算符，它有三种形式，这里使用placement new，它接受一个指针，在ptr所指地址上
构建一个对象。
```c++
void construct(T1*p,const T2&value){//①
new(p)T1(value);
}
```
2. destory()

销毁函数有两个版本，版本一使用销毁指针所指元素

```c++
template<class T>
void destory(T* pointer){
  pointer->~T();
}
```

版本二使用一对指针，销毁区间内所有元素。采用type_traits技法判断元素的析构函数是否是trivial的，以分别处理提高效率
想要使用该方法还要获得元素的类型，这就又增加的一层间接性。
//destory只是一个接口函数
```c++
template<class ForwardIterator>
void destory(ForwardIterator first,ForwardIterator last){
 __destory(first,last,value_type(first));
}
```
//__destory判断析构函数是否是trivial的

```c++
template<class ForwardIterator,class T>
void __destory(ForwardIterator first,ForwardIterator last,T*){
typedef tyname __type_triats<T>::has_trivial_destructor trivial_desturctor;
__destory_aux(first,last,trivial_desturctor());
}
```
//__destory_aux具体执行操作

```c++
template<class ForwardIterator>
void __destory_aux(ForwardIterator first,ForwardIterator last,__false_type){
for(;first!=last,++first)
    destory(&*first);//哦豁 是T*哦，first可不是指针
}
```

```c++
template<class ForwardIterator>
void __destory_aux(ForwardIterator first,ForwardIterator last,__true_type){}
```
---
### Todo
+ 找时间研究一下operator new
+ ①存疑 为什么要使用两个类型?
