### 数值算法
---
+ 输出数据时注意格式（换行？空格？特殊行的要求？）
+ 数值算法保存在numeric头文件中，包括
> + accumulate,inner_product,
  + partial_sum,adjacent_difference,
  + power,iota,(各个的用法？)

注意：
1. accumalate(first,last,i) i是传值进入函数内部的，在函数内部改变的i不会影响到外部，要获得 [first,last)的和 利用accumulate的返回值即可。inner_product也是如此。

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
