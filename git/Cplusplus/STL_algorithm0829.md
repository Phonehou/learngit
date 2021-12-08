## 质变算法 mutating algorithms——会改变操作对象之值
所有的STL算法都作用在由迭代器[first, last)所标示出来的区间上，所谓“质变算法”，是指运算过程中会更改区间内（迭代器所指）的元素内容，诸如**拷贝（copy）、互换（swap）、替换（replace）、填写（fill)、删除(remove)、排列组合(permutation)、分割(partition)、随机重排(random shuffling)、排序(sort)等等**。

通常提供两个版本：
* in-place（就地进行）版，例如replace()
* copy(另地进行)版,例如replace_copy()

所有的数值(numeric)算法，包括adjacent_difference()，accumulate()，inner_product(), partial_sum()等等，都实现于SGI<stl_numeric.h>之中，这是个内部文件，STL规定用户必须包含的是上层的<numeric>。
其他STL算法都实现于SGI的<stl_algo.h>和<stl_algobase.h>文件中，也都是内部文件，用户使用必须先包含上层相关头文件<algorithm>。

## 非质变算法 nonmutating algorithms——不改变操作对象之值
是指运算过程中不会更改区间内（迭代器所指）的元素内容，诸如查找(find)、匹配(search)、计数(count)、巡访(for_each)、比较(equal,mismatch)、寻找极值(max,min)等算法。

## 算法的泛化过程
"最后元素的下一位置"称为end
让find()接受两个指针作为参数，标示出一个操作区间：
迭代器是一种行为类似指针的对象
```c++
template<class Iterator, class T>
Iterator find(Iterator begin, Iterator end, const T& value)
{
    while(begin!=end&&*begin!=value)
        ++begin;
    //以下返回操作会引发copy行为
    return begin;
}
```

###  accumulate

### adjacent_difference

### inner_product
```c++
template <class InputIterator1, class InputIterator2, class T>
T inner_product(InputIterator1 first1, InputIterator1 last1, InputIterator2 first2, T init)
{
    //以第一序列之元素个数为据，将两个序列都走一遍
    for(;first1 != last1; ++first1, ++first2)
        init = init + (*first1 * *first2);  
    return init;
}
//提供仿函数
template <class InputIterator1, class InputIterator2, class T,class BinaryOperation1, class BinaryOperation2>
T inner_product(InputIterator1 first1, InputIterator1 last1, InputIterator2 first2, T init, BinaryOperation1 binary_op1, BinaryOperation2 binary_op2){
    for(;first1 != last1; ++first1, ++first2)
        init = binary_op1(init, binary_op2(*first1 * *first2));  
    return init;
}
```
### partial_sum

### power
```c++
//版本一：乘幂
template<class T, class Integer>
inline T power(T x, Integer n){
    return power(x, n, multiplies<T>());
}

//版本二：幂次方
template<class T, class Integer, class MonoidOperation>
T power(T x, Integer n, MonoidOperation op){
    if(n == 0)
        return identity_element(op);
    else{
        while((n & 1) == 0){
            n >>= 1;
            x = op(x, x);
        }

        T result = x;
        n >>= 1;
        while(n! = 0){
            x = op(x, x);
            if((n&1)!=0)
                result = op(result, x);
            n >>= 1;
        }
        return result;
    }
}
```
### iota
用来设定某个区间的内容，使其内的每一个元素从指定的value值开始，呈现递增状态。

## <stl_algobase.h>头文件中的所有算法
### equal
如果两个序列在[first, last)区间内相等，equal()返回true
### fill
将[first, last)内的所有元素改填新值
### fill_n
将[first, last)内的前n个元素改填新值，返回的迭代器指向被填入的最后一个元素的下一位置
### iter_swap
将两个ForwardIterators所指的对象对调
### lexicographical_compare

### max

### min

### mismatch
用来平行比较两个序列，指出两者之间的第一个不匹配点，返回一对迭代器。

### copy
将输入区间[first, last)内的元素复制到输出区间[result, result+(last-first))内。也就是会执行赋值操作*result = *first, *(result+1) = *(first+1),...依次类推，返回一个迭代器result+(last-first))。
```c++
//完全泛化版本
template<class InputIterator, class OutputIterator>
inline OutputIterator copy(InputIterator first, InputIterator last, OutputIterator result)
{
    return __copy_dispatch<InputIterator, OutputIterator>()(first, last, result);
}
//特殊版本（1）。重载形式
inline char* copy(const char* first, const char* last, char* result){
    memmove(result, first, last - first);
    return result + (last - first);
}
//特殊版本（2）。重载形式
inline wchar_t* copy(const wchar_t* first, const wchar_T* last, wchar_t* result){
    memmove(result, first, sizeof(wchar_t) * (last - first));
    return result + (last - first);
}
//__copy_dispatch()完全泛化版本
template<class InputIterator, class OutputIterator>
struct __copy_dispatch
{
    OutputIterator operator()(InputIterator first, InputIterator last, OutputIterator result){
        return __copy(first, last, result, iterator_category(first));
    }
};
//偏特化版本（1）
template<class T>
struct __copy_dispatch<T*, T*>
{
    T* operator()(T* first, T* last, T* result){
        typedef typename __type_traits<T>::has_trivial_assignment_operator t;
        return __copy(first, last, result, t();
    }
};
//偏特化版本（2）
template<class T>
struct __copy_dispatch<const T*, T*>
{
    T* operator()(const T* first, const T* last, T* result){
        typedef typename __type_traits<T>::has_trivial_assignment_operator t;
        return __copy(first, last, result, t();
    }
};
//以下版本适用于“指针所指之对象具备trivial assignment operator”
template <class T>
inline T*__copy_t(const T* first, const T* last, T* result, __true_type){
    memmove(result, first, sizeof(T)*(last - first));
    return result + (last - first);
}
```

### copy_backward
将[first, last)区间内的每一个元素，以逆行的方向复制到以result-1为起点，方向亦为逆行的区间上。

## set相关算法
并集(union)、交集(intersection)、差集(difference)、对称差集(symmetric difference)。
### set_union
是一种稳定(stable)操作，意思是输入区间内的每个元素的相对顺序都不会改变
```c++
template<class InputIterator1, class InputIterator2, class OutputIterator>
OutputIterator set_union(InputIterator1 first1, InputIterator last1, InputIterator2 first2, InputIterator2 last2, OutputIterator result){
    //当两个区间都尚未到达尾端时，执行以下操作
    while(first1 != last1 && first2 != last2){
        if(*first1 < *first2){
            *result = *first1;
            ++first1;
        }
        else if(*first2 < *first1>){
            *result = *first2;
            ++first2;
        }
        else{
            *result = *first1;
            ++first1;
            ++first2;
        }
        ++result;   
    }
    return copy(first2, last2, copy(first1, last1, result));
}
```
### set_symmetric_difference
构造出集合(S1-S2)U(S2-S1),此集合内含“出现于S1但不出现于S2”以及“出现于S2但不出现于S1”的每一个元素。

### rotate
将[first, middle)内的元素和[middle, last)内的元素互换，middle所指的元素会成为容器的第一个元素
```c++
//分派函数(dispatch function)
template <class ForwardIterator>
inline void rotate(ForwardIterator first, ForwardIterator middle, ForwardIterator last){
    if(first == middle || middle == last) return;
    __rotate(first, middle, last, distance_type(first), iterator_category(first));
}
```

### lower_bound(应用于有序区间)
如果没找到元素，返回一个指向第一个“不小于value”的元素的迭代器，如果value大于[first,last)内的任何一个元素，则返回last。返回“假设这样的元素存在时应该出现的位置”

### upper_bound
返回value可被插入的“最后一个”合适的位置，如果value存在，那么它返回的迭代器将指向valu的下一位置。

### binary_search
如果[first, last)内有等同于value的元素，便返回true,否则返回false

### next_permutation
```c++
template<class BidirectionalIterator>
bool next_permutation(BidirectionalIterator first, BidirectionalIterator last){
    if(first == last) return false;  //空区间
    BidrectionalIterator i = first;
    ++i;
    if(i == last) return false;  //只有一个元素
    i = last;  //i指向尾端
    --i;
    for(;;){
        BidirectionalIterator ii = i;
        --i;
    }
    if(*i < *ii){ //如果前一个元素小于后一个元素
        BidirectionalIterator j = last;  //令j指向尾端
        while(!(*i < *--j));  //由尾端往前找，直到遇上比*i大的元素
        iter_swap(i,j);
        reverse(ii, last); //将ii之后的元素全部逆向重排
        return true;
    }
    if(i == first){
        //进行到最前面
        reverse(first, last);
        return false;
    }
}
```

### random_shuffle

### partial_sort/partial_sort_copy

## sort
### Insertion Sort
```c++
template<class RandomAccessIterator>
void __insertion_sort(RandomAccessIterator first, RandomAccessIterator last){
    if(first == last) return;
    for(RandomAccessIterator i = first + 1; i != last; ++i)
        __linear_insert(first, i, value_type(first));
}
```
### Quick Sort
Median-of-Three(三点中值)
```c++
template<class T>
inline const T& __median(const T& a, const T& b, const T& c){
    if(a < b)
        if(b < c)
            return b;
        else if (a < c)
            return c;
        else
            return a;
    else if (a < c)
        return a;
    else if (b < c)
        return c;
    else
        return b;
}
```
