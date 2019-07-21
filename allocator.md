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

destory只是一个接口函数
```c++
template<class ForwardIterator>
void destory(ForwardIterator first,ForwardIterator last){
 __destory(first,last,value_type(first));
}
```
__destory判断析构函数是否是trivial的

```c++
template<class ForwardIterator,class T>
void __destory(ForwardIterator first,ForwardIterator last,T*){
typedef tyname __type_triats<T>::has_trivial_destructor trivial_desturctor;
__destory_aux(first,last,trivial_desturctor());
}
```
__destory_aux具体执行操作

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

3. alloc:内存的分配与释放

为了解决内存破碎问题，内存管理使用双层配置器，如果__USE_MALLOC定义，那么仅使用第一层，反之，则仅使用第二层。
还设计了标准接口类simple_alloc,这仅是对已有函数的引用包装。

+ 一层配置器:__malloc__alloc_template

一层配置器直接使用malloc与realloc进行分配和释放
```c++
template<int init>
class __malloc__alloc_template{
public:
static void* oom_malloc(size_t n) //内存不足处理
void* allocate (size_t n){
void* result=malloc(n);
if(result==0){
   oom_malloc(n);
   return result;
  }
}
void deallocate(void* p){
    free(p);
}
};
```

+ 二层配置器有16个档位(0-128)的块大小，每个档位都有很多连续的单位区块，我们构建
一个自由链表(free list)存储每个档位的**首区块的地址**。我们分配(allocate),释放(realocate)
重新装填(refill)实际上都是简单的在链表中删除或增加元素。

在实际实现中，我们需要一些辅助函数
> + ROUND_UP将所需的大小调至8的整数倍
> + FREELIST_INDEX所需大小通过除8得到索引值(即档位)

以下是具体实现，为了节约成本，对自由链表的节点采用union，实现了allocate,deallocate,
refill,chunk_alloc函数

```c++
enum{__ALIGN =8};
enum{__MAX_BYTES=128};
enum{__NFREELIST=__MAX_BYTES/__ALIGN};
template<int init>
class __default_alloc_template{ 
private:
static size_t ROUND_UP(size_t bytes){
      return ((bytes)+__ALIGN-1)&~(__ALIGN-1));
}
static size_t FREELIST_INDEX(size_t bytes){
      return (bytes+__ALIGN-1)/__ALIGN-1;
}
union obj{
     union obj* free_list_link;
     chat client_data[1];
}
static obj* volatile free_list[__NFREELISTS];
static void *refill (size_t n);
static char* chunk_alloc(size_t n,size_t types);

static char* start_free;
static char* end_free;
static size_t heap_size;

public:
static void* allocate(size_t n);
static void deallocate(void* p,size_t n);
};
```
相当于在free_list中删除一个节点

```c++
tempalte<class init>
void* __default_malloc_template::allocate(size_t n){
     if(n>__MAX_BYTES) return malloc_alloc::allocate(n);
     obj* result;
     size_t index=FREELIST_INDEX(ROUND_UP(n));
     obj*volatile* my_free_list=free_list+index;
     result=my_free_list;
     *my_free_list=result->free_list_link;
     return result;
}
```
相当于在free_list中增加一个节点

```c++
template<class init>
void  __deafult_malloc_template::deallocate(void * p,size_t n){
     size_t index=FREELIST_INDEX(n);
     obj* q=(obj*)p  //why?
     obj* volatile* my_free_list=free_list+index;
     q->free_list_link=(*my_free_list);
     (*my_free_list)=q;
}
```
相当于在free_list中增加一系列节点，如果内存只够一块，直接返回，否则先返回第一个区块
后面的储存在自由链表中
```c++
template<class init>
void* __default_alloc_template::refill(size_t n){
     obj* result,*current_node,*next_node,*my_free_list;
     int  nobjs=20;
     my_free_list=free_list+FREELIST_INDEX(n);
     char* chunk=chunk_alloc(n,nobjs);//引用传递
     if(nobjs==1)return chunk;
     
     reault=chunk;
     *my_free_list=next_node=(obj*)(chunk+n);
     for(int i=0;;i++){//
        current_node=next_node;
        next_node=(obj*)(next_node+n);//找出新节点的位置
        if(i==nobjs-1){
            current_node->free_list_link=0;
            break;
        }
        else
            current_node->free_list_link=next_node;//将新节点和旧节点连起来
     }
     return result;
}
```
内存池控制
```c++
template<int init>
char* __default_alloc_template<init>::chunk_alloc(size_t n,int& nobjs){//返回值怎么是char*啊
    char* result;
    size_t total_bytes=n*nobjs;
    size_t bytes_left=end_free-start_free;
    
    if(bytes_left>=total_bytes){
        result=start_free;
        start+=total_bytes;
        return result;
    }
    else_if(bytes_left>=n){
        size_t size=bytes_left/n;
        size_t use_size=size*n;
        result=start_free+use_size;
        bytes_left=end_free-result;
        return result;
    }
    else{
        size_t bytes_to_get=2*total_bytes+ROUND_UP(heap_size>>4);//why?
        if(bytes_left>0){
            obj*volatile*my_free_list=free_list+FREELIST_INDEX(bytes_left);
            (obj*)start_free->free_list_link=*my_free_list;//start_free有下一个obj*吗？哪里定义了
            *my_free_list=(obj*)start_free;
        }    
    }
    
    ...
}
```
---
### Todo
+ 找时间研究一下operator new
+ ①存疑 为什么要使用两个类型?
+ 为什么是void* 
+ 内存不足的c++ new handle处理机制
+ union类型，enum类型
+ 全是static的？
+ free_list节点到底是怎样子的呢
