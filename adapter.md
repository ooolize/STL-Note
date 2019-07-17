+ 配接器

> 配接器将一个class的接口转换为另一个class的接口。

> + 容器适配器：stack,queue修饰deuqe的接口而成另一种容器的风貌
> + 迭代器适配器：insert_iterator,istream_iterator,reverse_iterator。他们定义在iterator中
>> + insert_iterator:可以将一般迭代器的赋值操作变为插入操作。根据容器不同又分为后向插入的back
_inserter(c),前项插入的front_inserter(c),中间项插入的inserter(c,iter).
>> + reverse_itertor:将一般迭代器的行进方向逆转，重载了operator++为后退，operator--为前进。
>> + iostream_iterator:绑定在一个iostream对象上，拥有输入输出功能。
> + 仿函数适配器：通过系结(bind),否定(negate),组合(compose)仿函数以及对一般函数的修饰创造可
能的表达式

back_inserter内部维护一个容器，重载赋值操作，当用户赋值时，就调用push_back(push_front)。至于
其他operator++,operator* 等功能都被关闭。实现为一个class template如下：
```c++
template<class container>
class back_inserter_iterator{
typedef void value_type;
typedef void difference_type;
typedef void reference_type;
typedef void pointer_type;
typedef output_iterator_tag iterator_category;
protect:
container* c;
public:
back_inserter_iterator(container* c):c(&c){}
back_inserter_iterator<container>& operator=(const typename container::value_type& v){
c->push_back(v);
return *this;
}

back_inserter_iterator<container>& operator*(){return *this;}
back_inserter_iterator<container>& operator++(){return *this;}
back_inserter_iterator<container>& operator--(int){return *this;}

};
```
我们用一个辅助函数back_inserter调用这个类的赋值操作。
```c++
template<class container>
back_inserter_iterator<container>& back_inserter(container& c){
return back_inserter_iterator(c);
}
```

在这个函数中，我们并没有使用重载的operator+,而是仅仅多了一层间接性。它实现的是auto p=back_inserter(c);而不是* p=12;
