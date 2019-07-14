### 数值算法
---
+ 输出数据时注意格式（换行？空格？特殊行的要求？）
+ 数值算法保存在numeric头文件中，包括
> + accumulate,inner_product,
  + partial_sum,adjacent_difference,
  + power,iota,(各个的用法？)

注意：
1. accumalate(first,last,i) i是传值进入函数内部的，在函数内部改变的i不会影响到外部，要获得 【first,last)的和 利用accumulate的返回值即可。inner_product也是如此。

2. partial_sum和adjacent_different是可以是质变算法，当result不由输入流控制时，result=begin,那么就是in place的。

测试代码：
```c++
#include<iostream>
#include<algorithm>
#include<vector>
#include<numeric>
#include<sstream>
using namespace::std;

void  printf(vector<int>& v1){
	for (auto &p : v1)
		cout << p;
	cout << endl;
}
int main(){
	int ai[] = { 1, 2, 3, 4, 5 };
	int i = 0;
	vector<int> v1(ai, ai + sizeof(ai) / sizeof(int));
	printf(v1);

	cout<<accumulate(v1.begin(), v1.end(), i)<<endl;

	cout<<inner_product(v1.begin(), v1.end(), v1.begin(),i)<<endl;
	
	ostream_iterator<int> ote1(cout, " ");
	adjacent_difference(v1.begin(), v1.end(),v1.begin());
	printf(v1);
	partial_sum(v1.begin(), v1.end(), v1.begin());
	printf(v1);
	getchar();
}
```
测试结果:
![1](C:\Users\Administrator\Desktop\to\img\1.png)
###基本算法
---
+ 各个容器迭代器种类：InputIterator,OutputIterator,ForwardIterator,BidirectionalIterator,RandomAccessIterator

容器|迭代器类型
:---|:---|
vector|RandomAccessIterators|
list|BidirectionalIterators|
slist|ForwardIterators|
deque|RandomAccessIterator|
stack|无 |
queue| 无|
heap| 无|

+ 将一段元素插入一个空容器中：
  一是使用容器的成员函数insert.
> + v.insert(p,b,e): 将迭代器b,e)之间的元素插入到p之前,b和e不能指向容器v.返回指向第一个新加元素的迭代器。若b,e)为空，则返回p
> + v.insert(p,n,t): 将n个t插入到p之前，返回新添加的第一个元素的迭代器。

 二是使用插入迭代器:在头文件iterator中，接受容器参数，返回该容器的插入迭代器,实际上是调用容器的push_back(push_front)
```c++
 auto it=back_insert(v1);
 *it=1;//插入1
```
> + back_inserter 创建一个使用push_back的迭代器

 eg:c++ fill_n(back_inserter(v1),10,5)
 
 向空容器v1中插入10个5
> + front_inserter 创建一个使用push_front的迭代器
> + inserter 接受第二个参数是该容器的迭代器，将元素插入到该迭代器的前面。
  eg:  copy(v1.begin(),v1.end(),inserter(v2.begin())
  
  将v1序列复制到空容器v2中

注意： front_inserter与inserter虽然都是向元素前插入序列但是却有所不同。比如从5 6 7 的6前插入序列1 2 3 4

front_inserter是 5 4 3 2 1 6 7

inserter是      	 5 1 2 3 4 6 7  

+ 基本算法：
1. + fill(ForwardIterator first,ForwardIterator last,const T& value)
   
   该函数将容器内所有元素重置为value（不可对空容器使用）
   
   无返回值
   +  fill_n(**Outputiterator** first,const int& n,const T&value)（question:插入迭代器,流迭代器属于什么类）
   
   该函数对容器填入n个value
   
   无返回值
2. + copy(RandomAccessIterator first,RandomAccessIterator last,**Outputiterator** result)
   
   该函数将容器内的元素复制到result所指的位置（result可在【first,last)中，但不保证成功）
   
   返回一个Outputiterator它指向last（一般不用）
   + copy_backward(**Bidirectional**Iterator first,**Bidirectional**Iterator last,**Bidirectional**Iterator result)
   
   该函数将【first,last)内的序列逆序插入到以result-1为起点，方向亦为逆序的区间上。
   
   返回一个Bidirectionaliterator它指向last（一般不用）

3. equal(InputerIterator first1,InputerIterator last1,InputerIterator first2)
   
   该函数判断序列1与序列2是否相等（使用前需要先判断大小是否相等）
   
   返回判断结果相等为true,不等为false.
4. lexigraphical_compare(InputIterator first1,InputIterator last1,InputIterator first2,InputIterator last2)
   
   该函数判断序列一和序列二的字典序，字典序默认less_then并且版本二提供仿函数```comp(*first1,*first2)```供比较
   
   返回判断结果不小于为true,大于为false
