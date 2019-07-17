+ 仿函数
> 仿函数是重载了函数调用运算符的类。用来搭配标准库算法，泛化算法功能。比如sort()版本二允许用户指定操作，排序后的相邻元素会使该操作为true。
再如，accumulate()的操作默认是累加，我们可以通过使用仿函数使其变为累乘。

> 函数指针，lambda表达式是与仿函数类似功能的配件，虽然可以实现与算法的互动，但是不具有可配接性（Adaptable），
为了拥有配接能力，每一个仿函数必须定义自己的相应型别，就像迭代器如果要融入STL，需要定义5个型别一样。相应型别需要由
配接器取出。相应型别只是一些typedef，所有必要操作在编译期就全部完成了，对程序的执行效率没有影响。

> 仿函数的相应型别主要是参数型别和传回值型别，为了方便起见，标准库定义了两个class，分别代表一元仿函数和二元仿函数，
其中没有任何的data members或member functions，唯有一些型别定义。任何仿函数只要选择一个继承，便拥有了相应型别，也
就拥有了配接能力。

```c++
//一元仿函数
template<class Arg,class Result>
class unary_function{
typedef Arg argument_type;
typedef Result result_type;
};

//二元仿函数
template<class Arg1,class Arg2, class Result>
class binary_function{
typedef Arg1 first_argument_type;
typedef Arg2 second_argument_type;
typedef Redult result_type;
};

```

+ 仿函数的类型

STL内建的仿函数在头文件functional中
> + 算术类仿函数:plus<T> minus<T> multiplies<T> divides<T> modulus<T> negate<T>

```c++
template<class T>
class plus:public binary_function<T,T,T>{
T operator()(const T&a,const T&b)const {return a+b;}
};

template<class T>
class negate:public unary_function<T,T>{
T operator()(const T&a)const{return -a;}
};
```

accumulate(v1.begin(),v1.end(),multiplies<int>());表示累乘

> + 关系运算类仿函数：equal_to<T> not_equal_to<T> greater<T> greater_equal<T> less<T> less_equal<T>

```c++
template<class T>
class greater:public binary_function<T,T,T>{
bool operator()(const T& a,constT &b)const{return a>b;}
};

```

sort(v1.begin(),v2.end(),greater<int>());

> + 逻辑运算类仿函数: logical_and<T> logical_or<T> logical_not<T>

```c++
template<class T>
class logical_and:public binary_function{
bool operator()(const T& a,const T&b)const{return a&&b;}
};
```
> + 证同(identity)，选择(select)，投射(project)
>
> 证同返回本身。选择接受一个pair返回第一或第二参数。投射接受两个参数，传回某个参数，忽略参数。
