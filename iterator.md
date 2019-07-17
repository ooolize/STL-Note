+ STL的迭代器
> 迭代器提供一种方法，使之能依序寻访某个容器的各个元素，而又无需暴露该容器的内部结构。

每个容器都有特定的迭代器，在我们使用它的时候,特别是将迭代器所指元素作为传回值时,为了
让编译器知道传回值的类型，我们需要“内嵌型别”+typename来实现。如下示例:
```c++
template<class T>
class Iter{
typedef *T value_type;
T* p;
Iter(T* q=0):p(q){}
T& operator*()const{return *p};

};

template<class I>
typename I::value_type 
func(I iter){
return *iter; 
}
```

但这种方法存在问题，当迭代器为原生指针时便无法使用内嵌型别，这时我们需要针对一般化情况的特
殊化处理，采用模板偏特化。

具体实现为：迭代器内部进行型别声明。原本使用返回值使用typename,加上类模板，便增加了一层
间接性，可以依靠偏特化处理特殊情况。

1. 型别声明
> 根据经验，常用的型别有五个需要声明：
> + value_type:迭代器所指对象的型别，任何一个与STL完美搭配的class，都应该定义自己的value_type内嵌型别。
> + difference_type: 可以表示两个迭代器之间的距离，容器的最大容量，为泛型算法提供计数类型（count），对于
原生指针，c++以内建的ptrdiff_t作为difference_type
> + reference_type：对于迭代器iter，它的value_type是T。*iter的型别就是iter的reference_type，如果iter是左值指针，那么*iter
就应该是T&，如果iter是右值指针，那么*iter就是const T&.
> + pointer_type: 对于迭代器iter，它的value_type是T。T*的型别就是iter的reference_type。
> + iterator_category:见algorithm文件中我们已经整理了五种类型。

代码如下：
```c++
template<class T>
class iterator:public iterator{
typedef T value_type;
typedef ptrdiff_t difference_type;
typedef T& reference_type;
typedef T* pointer_type;
typedef forward_iterator_tag iterator_category;
};
```

2. 类模板
> iterator_traits扮演的一个萃取机，用来提取某个迭代器的型别。要保证traits有效运作，每个迭代器必须遵守约
定，自行以内嵌型别定义的方式给出相应型别，以兼容整个stl。对于原生指针我们亦可以使用偏特化。示例如下：

```c++
template< class I>
class iterator_traits{
typedef typename I::value_type value_type;
typedef typename I::difference_type difference_type;
typedef typename I::reference_type reference_type;
typedef typename I::pointer_type pointer_type;
typedef typename I::category_type category_type;
};

template<class T>
class iterator_traits<T*>{
typedef T value_type;
...
};
```
3. 使用
> 当我们需要使用型别时，像标准库算法中根据不同的迭代器类型设计不同的操作，我们便可以用iterator_traits判定到底采用哪个子函数。

eg:  anvancd(iter,n)

```c++
template<class I>
void
advance(I iter,typename I::difference_type n){
__advancd(iter,n,iterator_traits<I>::category_type category());
}

template<class >
void
__advance(I iter,typename I::difference_type n,Forward_Iterator_tag){
while(n--)
iter++;
}

template<class >
void
__advance(I iter,typename I::difference_type n,Forward_Iterator_tag){
iter+n;
}
```

总结：
> 设计适当的相应型别，是迭代器的责任，设计适当的
迭代器，则是容器的责任，唯有容器本身，才知道设计
怎样的迭代器遍历自己，并规定迭代器相应的行为。迭代器的型别通过traits技法获得，这一手法在STL中广泛存在
