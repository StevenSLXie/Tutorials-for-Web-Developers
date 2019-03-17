# C++ Tutorial: STL Vector Walk-through
This tutorial demonstrates the usages of `vector` defined in C++ standard template library (STL). It is the first part of this series that aims to cover the essential aspects of C++ STL. While not every member function is demonstrated, this document tries to introduce, using easy-to-understand examples, the most frequently used parts of the library.

## How to use this tutorial?
It is best that you read this while typing and running the examples using your favourite IDE. The headers needed to be included are as follows.

```c++
#include <iostream>
#include <vector>
```

Moreover, as `vector` is frequently used in the following, you can also write `using std::vector` to simplify the code.



## Why Vectors?
For those coming from standard C, `array` is often used to define a sequence of the same type. The same facility can still be used in C++, but compared with that, `vector` offers greater flexibility and functionality. For example, `vector` is able to change its size dynamically while the corresponding memory allocation is automatically taken care of by the container itself.

## Define a Vector

`vector` can be defined in various ways. The simplest one is to define a vector without specifying the elements, as follows.

```c++
vector<int> v1;
```

This creates an empty vector. Note that integer vector is used here. But in reality, virtually any type can be used as the element type of a vector, including user-define class.

A commonly used member function is `push_back`. You can use it to insert new elements to the end of the vector.

```c++
v1.push_back(10);
```

You can also define `vector` in an array-like way.

```c++
vector<int> v2 = {100,200,300};
```

If you would like to define a `vector` of the same elements, you can use 

```c++
vector<int> v2(5,200);
```
This effectively an array of 200 with a size of 5.

Alternatively, you can initialise a `vector` by copying another `vector`.

```c++
vector<int> v3(v2);
```

When copying elements from other objects, there are additional ways to do it.

```c++
vector<int> v4(v2.begin(),v2.end());
```

`begin()` and `end()` are two member functions of vector that returns iterators. Iterators are pointers that point to a specific memory address of a STL container. We will have more to say about iterators. But for now, just remember that `begin()` points to the first element of the vector while `end()` points to the position *after* the last element. Note that `end()` does not point to the last element, but the position after it. This turns out to facilitate referencing elements in a container in many situations.

For the example above, the whole v2 is copied to v4. The two arguments specify the start and end position in a `[a, b)` manner. Such facility allows us to copy only part of a vector. For example, 

```c++
vector<int> v4(v2.begin(),v2.begin()+2);
```

This initialises v4 by copying the first 2 elements from v2

Lastly, we can also initialise v5 by copying from (part of) an array

```c++
int a1[] = {10,20,30,40,50};
vector<int> v5(a1, a1+sizeof(a1)/sizeof(int));
```
## Access and Iterate Over a Vector

There are various ways to access and iterate over a vector. For element access, the simplest one is by the `[]` operator. Below shows how to use `[]` in a for-loop to iterate over a vector. Note that the member function `size` returns the number of elements in the vector.

```c++
for (int i = 0; i< v5.size(); ++i){
  std::cout << v5[i] << " ";
}
```
Another way is to the `at` function, such as `v5.at(i)`. Its main difference with the `[]` operator lies in that `at` checks whether `i` exceeds the size boundary and throws an exception while `[]` operator does not check the bound.

Another stream of element access is by iterators. Iterators, as the name suggest, are used for iterations within a container. `vector` and other containers defined in STL have a corresponding data type called `iterator`. An iterator is essentially a pointer that can points to each element in the vector. Below shows its basic use in a for-loop

```c++
for (vector<int>::iterator it = v5.begin(); it != v5.end(); ++it){
  std::cout << *it << " ";
}
```

This piece of code traverses from the first element of v5 towards the end of v5 and prints each element value the iterator points to. Note that an iterator essentially a pointer that stores an address (not a value) and `*` operator is required to retrieve the element value.

On a side note, you may notice that `vector<int>::iterator` appears very verbose. Good news is that from C++11, you can use the specifier `auto` as follows.

```c++
for (auto it = v5.begin(); it != v5.end(); ++it){
  std::cout << *it << " ";
}
```
This is possible because the type of `it` can be well deduced from `v5.begin()` and thus it is not a must to explicitly declare its type.
This piece of code will be called frequently when you deal with vector and it makes sense to wrap it up using a utility function.

```c++
void traverse_element(const vector<int> & v){
    for(auto it = v.begin(); it != v.end(); ++it){
        std::cout << *it << " ";
    }
    std::cout << std::endl;
}
```
Two things are worth mentioning. First, the argument `v` is of pass-by-reference rather than pass-by-value. This avoids copying the vector and is generally more memory-efficient. Second, the argument is a `const` type, which prevents `v` from being modified within the function. Putting a `const` is generally good practice when only read-only operations are performed (no write-in operation). Note that in this case, as a `const` object is passed, the corresponding iterator has to be of type `const_iterator`. We use `auto` to by-pass this issue. Otherwise, we would have to specify the iterator type, which would be `vector<int>::const_iterator` in this case. 

Other than `begin()` and `end()`, `vector` has 2 other member functions `rbegin()` and `rend()` that returns iterators for reverse traversing. In particular, `rbegin()` points to the last element while `rend()` points to the theoretical position preceding the first element. So to traverse from the last element to the first, one can write 

```c++
for (auto it = v5.rbegin(); it != v5.rend(); ++it){
  std::cout << *it << " ";
}
```

## Insertion, Deletion, Modification

The `push_back()` we used previously is a common way to insert new elements. It appends new elements to the end of the vector. Correspondingly, there is a `pop_back` that pops out the last element of the vector. If we want to insert new elements to a random position of a vector, `insert` can be used. For example,

```c++
    vector<int> v5 = {5,6,7,8,9};
    auto it = v5.begin();
    v5.insert(it, 3); // insert 3 in the very beginning
    it = v5.begin();
    v5.insert(it+1,4); // insert 4 right after the first element. 
    it = v5.end();
    v5.insert(it, 3, 10);  // insert 3 copies of 10 to the end of the vector
    it = v5.begin();
    int a2[] = {1,2};
    v5.insert(it, a2, a2+sizeof(a2)/sizeof(int)); // insert elements from an array
```

This piece of code demonstrates various ways of using `insert()`. After 4 `insert` operations, vector v5 becomes `v5={1,2,3,4,5,6,7,8,9,10,10,10}`. For `insert`, the first argument is an iterator that specifies the position where new elements are inserted.

As for deleting elements, one can use `erase()`. `erase` also takes one or two iterators as its arguments. In the case of one iterator, `erase` deletes that particular element resided in the address that the iterator points to. In the case of two iterators, a range of elements specified by iterators are deleted.

It should be noted that for `insert` and `erase`, operating in positions other than end of the vector causes the vector to relocate its elements. This is generally inefficient.

## 2D Vector

Vectors can be well expanded to 2 dimensions to store, for example, a 2D matrix.
The following example shows how to define a 2D vector, how to insert new elements and how to iterate over it.

```c++
vector<vector<int> > vv1;
vector<int> row1(5,1); // define a new row
vector<int> row2(5,-1); // define another row
vv1.push_back(row1); // insert the first row
vv1.push_back(row2); // insert the second row
vv1[0].push_back(8); // append 8 to the first row.
```

It can also be constructed directly, 

```c++
vector<vector<int> >vv2 = {{1,2,3},
                           {4,5,6},
                           {7,8,9,10}};
```

It can be seen that there is great flexibility when defining a 2D vector: the number of elements in each row does not have to be the same.

Accessing a 2D vector is also simple:

```c++
for(int i = 0; i<vv2.size(); i++){
    for (int j=0;j<vv2[i].size(); j++){
        std::cout << vv2[i].at(j) << " ";
    }
    std::cout << std::endl;
}
```
## Memory management and resource allocation

The `vector` automatically manages the memory allocation. Internally, it uses contiguous storage location to store its elements, which means the elements using offsets on pointers. This is the same as array. What is different is that vector may change its size from time to time when new elements are inserted. When this happens, a new memory space will be created and all elements will be moved to it. This is in generally expensive and vector may thus allocate some extra space to accommodate future growth. As a result, the actual capacity of a vector may be larger than the actual size.

In addition to automatic allocation, one can also use `reserve` to pre-allocate space. For example, `v.reserve(1000)` increases the vector capacity to 1000 (if v had a capacity less than 1000).

In the event that the requested capacity is smaller than the current capacity, no action will be taken.

This function is helpful when one can foresee, in the program, how the vector will grow. He can then use `reserve` to set the capacity for once and avoid frequent re-allocation.

One can use `v.capacity()` to check the current capacity. Note that `reserve` does not change vector size. That is, before and after the `reserve` operation, `v.size()` should return the same value as `size()` only counts the number of elements.

In the event that the size of a vector is way smaller than the capacity, one can use `v.shrink_to_fit()` to request a reduction of capacity to fit the size. But this is a non-binding function, which means it is implementation-dependent and compiler can optimise otherwise such that the capacity may still be larger than the size.

Below is a code snippet that demonstrates the above and some possible (could be platform-dependent) output:

```c++
vector<double> v;
std::cout <<"After initialization, capacity is: " << v.capacity()<< std::endl;
v.reserve(1000);
for (int i = 0; i< 1000; i++){
    v.push_back(i);
}
std::cout <<"After reserve, capacity is: " << v.capacity()<< std::endl;
while(!v.empty()){
    v.pop_back();
}
std::cout <<"After empty the elements, capacity is: " << v.capacity()<< std::endl;
v.shrink_to_fit();
std::cout <<"After shrink_to_fit, capacity is: " << v.capacity()<< std::endl;
```

Possible output:

> After initialization, capacity is: 0

> After reserve, capacity is: 1000

> After empty the elements, capacity is: 1000

> After shrink_to_fit, capacity is: 0

## Summary

This tutorial covers the basic aspects of `vector` defined in STL. Next I will introduce more on other types of containers (e.g., map, list). Do note that the containers in STL are designed in a way that their interfaces are very similar, in spite of unique features that each container has. So understanding the interfaces introduced in this tutorial for `vector` will definitely help you understand other types of containers.

















