# C++ Lambda Explained with Examples

## Right on the spot

Lambda expression is essentially a function object that can be inserted right into where it is used, which results in often more concise code and much better readability. 


Below is a simple example:

```c++
[](int i) {std::cout << i*i << " ";};

```
It takes a parameter `i` and prints out the results of `i*i`. Inside `()` is a parameter list, similar to a normal function. Inside `[]` is called capture clause. We will come to that later. The statements inside `{}` is the lambda body, where the main functionality of the lambda is implemented. 

Lambda is especially useful when combining with algorithm functions in STL. For example, if we want to compute and print the square of each element in a vector, we can make use of the lambda example above together with `for_each`

```c++
for_each(v.begin(), v.end(), [](int i) {std::cout << i*i << " ";});
```

Another example is to find the first even number in a vector.

```c++
auto it = std::find_if(v.begin(), v.end(), [](int i){return i%2 == 0;});
```

For the Lambda expression, note that in this case we do not need to specify the return type as it is automatically deduced from the context. There are also rules to explicitly specify the return type, which we will cover later.


By the way, in pre-C++11 era, we would need to write a full-fledged function for the same purpose:

```c++
bool IsOdd (int i) {
  return ((i%2)==1);
}

auto it = std::find_if (v.begin(), v.end(), IsOdd);
```

The advantage of lambda over functions lies not only in that 4 lines become 1 line, but that the readability is better. The program appears in a natural flow so that reader does not have to "jump" to another function to figure out what is done to the vector. Moreover, checks like `IsOdd` is often one-time-use utility and it is not so efficient to abstract it as a function since it will likely not be called elsewhere. 

lambda can also be assigned to a variable and assignment operation also works, as follows.

```c++
auto c = [](int i) mutable {std::cout << i*i << " ";};
auto c2 = c;
```

`auto` is used to deduce the type. 


## Capturing externals

The beauty of lambda also lies in that with the capture clause, it is able to access/capture variables from the surrounding scope. See the following examples

```c++
#include <iostream>
#include <vector>
#include <algorithm>

using std::vector;

int main(int argc, const char * argv[]) {
    
    
    int offset = 5;
    vector<int> v{1,2,3,4,5};
    for_each(v.begin(), v.end(), [offset](int i) {std::cout << offset+i << "" " ";});
}
```
By placing `offset` in the capture clause, it is passed by value to the lambda and can effectively be used there. 

It turns out that there are 2 forms of capture, namely by value or by reference. Those variables with `&` prefix is passed-by-reference while others are passed-by-value. Below shows various forms of capture

```c++
[=] // capture all variables in the surrounding scope by value
[&] // capture all variables in the surrounding scope by reference
[val] // capture val by value
[&val] // capture val by reference
```
However, capturing all variables at one go is generally not recommended and should be avoided in most cases. For a detailed discussion on this topic, one can refer to Item 31 of [Effective Modern C++](https://www.amazon.com/Effective-Modern-Specific-Ways-Improve/dp/1491903996) by Scott Meyers.

There are some distinct differences between by-value and by-reference captures. 

First, by-reference can be used to modify the variable outside while by-value cannot. Let's consider the following 2 cases

Case 1:

```c++
int offset = 5;
vector<int> v{1,2,3,4,5};
for_each(v.begin(), v.end(), [&offset](int i) {std::cout << ++offset+i << " ";});
std::cout << offset << std::endl;
```
`offset` is passed by reference and it is modified by `++` each time lambda is called. Such modification WILL affects the value of `offset` outside the lambda. When its value is printed in the last line, it outputs `10` instead of `5`

Case 2:

```c++
int offset = 5;
vector<int> v{1,2,3,4,5};
for_each(v.begin(), v.end(), [offset](int i) {std::cout << ++offset+i << " ";});
std::cout << offset << std::endl;
```
Case 2 won't compile! `offset` is passed by value and by default when passed-by-value is used, it cannot be modified by lambda.

To overcome this, lambda provides a `mutable` keyword that allows changes on value capture. Case 2 can be modified as follows to allow `offset` to be updated inside lambda

```c++
int offset = 5;
vector<int> v{1,2,3,4,5};
for_each(v.begin(), v.end(), [offset](int i) mutable {std::cout << ++offset+i << " ";});
std::cout << offset << std::endl;
```
It is important to note that such changes only take effect within the lambda. Because the value is copied to lambda body, it will not affect the `offset` variable outside lambda. In this case, the last line should print `5` instead of `10`.

Another thing to note is that one can well combine by-reference and by-value in the capture clause. For example, the following is perfectly fine

```c++
[&val1, val2]
```

## Generalized and generic lambda

In C++14, capture is extended by allowing new variables to be initialized in the capture clause. This is particularly useful when you want to capture move-only objects into lambda. For example, `std::unique_ptr` is noncopiable and neither by-reference nor by-value can achieve the purpose of capturing it into lambda. However, in C++14, you can do this by generalized capture:

```c++
p = make_unique<vector<int>>(nums);
// do thing on p
auto a = [ptr = move(p)](){
           // use ptr
        };
```

As a trivial example, one can move (instead of copy) a vector into lambda as follows.

```c++
vector<int> v{1,2,3,4,5};
auto func = [data = std::move(v)](int i){
        std::cout << *data.begin() + i << " ";
    };
vector<int> v2{2,3,4,5,6};
for_each(v2.begin(),v2.end(),func);
```

C++14 also allows the parameter list to use `auto` type and let compiler deduces the actual type. For example

```c++
auto size = [](const auto & m){return m.size();};
```

We do not have to specify the type of `m`. For some verbose expression, this feature, called generic lambda, could be very helpful.

Another example is simple addition, where any of the parameters could be generic so that it has better flexibility.

```c++
auto add_ = [](auto x, auto y){return x+y;};
```

## Return from lambda

Lambda can deduce the return type if there is only one return statement or if multiple return statements yield the same type. In the examples above, as there is only one return or no return, there is no need to explicitly specify the return type. However, when multiple statements yield different types, `->` has to be used after parameter list. An example is shown as follows.

```c++
auto l = [](int value) -> double {
        if (value < 0) {
            return 1;
        } else {
            return -1.0;
        }
    };
```

As there are two return statements and the data types are different, compiler is unable to deduce the return type and thus the return type has to be explicitly declared. 







