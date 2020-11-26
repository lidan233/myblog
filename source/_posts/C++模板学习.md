---
title: C++ template 学习
---

# 模板学习

## 模板想法

### 参数推断

C++ 编译器会自动模板参数，但是函数调用过程中第一个模板参数必须给出后面的T，前面的模板typename可以自动推断，如果给出前面的，不给出后面的，会导致出现问题。

```C++
float data[1024] ;
template <typename T> T GetValue(int i)
{
    return static_cast<T>(data[i]) ;
}

float a = GetValue(0) ;
float b = GetValue(1) ;
```

编译器没有办法推断出模板的参数的，只有在特殊的情况才能。
特殊情况如下所示：

``` C++
template <typename DstT, typename SrcT> DstT c_style_cast(SrcT v)// 模板参数 DstT 需要人肉指定，放前面。
{
    return (DstT)(v);
}

int v = 0;
float i = c_style_cast<float>(v);  // 形象地说，DstT会先把你指定的参数吃掉，剩下的就交给编译器从函数参数列表中推导啦。
```

### 非类型参数

```C++
template <typename T,int size> struct Array{
    T data[size] ;
}

Array<int,16> arr ;
```

模板所有的参数必须在编译器能够确定，如果不能确定就会报错,而且目前仅仅支持int等**整型**类型的参数。

```C++
template <int i> class A {};

void foo()
{
    int x = 3;
    A<5> a; // 正确！
    A<x> b; // error C2971: '_1_3::A' : template parameter 'i' : 'x' : a local variable cannot be used as a non-type argument
}
```


