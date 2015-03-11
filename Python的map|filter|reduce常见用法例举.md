Python的map/filter/reduce常见用法例举
================

**摘要**：熟练掌握Python的map、filter、 reduce、 lambda以及list comprehension的用法，可以让你写出简洁的代码。

<h4>1. map</h4>

大多数的for循环可以用map来代替。`map(func,seq)`自动地将`func`施加到`seq`的每一个元素，并返回执行结果。

**例1** 返回list `array=[3,5,7,12,18]`每个元素的立方组成的新list。

用for循环的解法

```python
cube_array = []

def cube(x):
    return x**3
    
for a in array:
    cube_array.append(cube(a))
    
```

或者简单一点：

```python
cube_array = []

for a in array:
    cube_array.append(a**3)
```

用map只需要一行。

```python
cube_array = map(lambda a:a**3,array)
```

刚才我们说，map的第二个参数是一个sequence，这里的sequence是我们的array，而map的意思，是对这个array进行一种操作。什么操作呢？这个操作由第一个参数来定义。

这里的第一个参数，是一个lambda函数。lambda在Python里面称为匿名函数，就是说它是个函数，却没有名称。lambda关键词之后的冒号前面用来定义参数，冒号后面用来定义我们要返回的东西。在这里，我们传入的参数是一个a，返回a的立方。而a是什么呢？a是array里面的每一个元素。

如果map和lambda的交叉使用一下子让你招架不住的话，可以看看下面这个例子：

```python
def cube(a):
    return a**3

cube_array = map(cube,array)
```

这里map的第一个参数是一个函数名cube，我们另外定义了这个函数。但由于这个函数实在太简单，我们没有必要按照惯常的方法，给它一个名字，郑重其事地出来，而是用lambda这样一种取巧的方法。

什么时候可能用lambda呢？函数特别简单的时候可能可以用，但注意，这里这个函数最好是我们从头到尾只会调用一次的。如果是那种简单但却会被反复调用的函数模块，还是给它一个名字比较好。不然每次都要写出lambda函数也是很麻烦。

lambda的更多注意事项可以参考[这篇](https://pythonconquerstheuniverse.wordpress.com/2011/08/29/lambda_tutorial/)。

这个例子也可以用Python强大的list comprehension来实现：
```python
cube_array = [x**3 for x in array]
```

同样是一行。至于哪一种更好，就看你自己了。


<h4>2. filter</h4>
filter顾名思义，是对一个sequence中的每一个元素进行过滤，返回符合条件的那些元素。用法是`filter(func,seq)`，和map很像。

**例2** 返回`array = [3,5,7,12,18]`中的所有奇数。

```python
print filter(lambda x:x%2, array)
```

这里是对2取余，返回结果为True的元素。那么什么情况下结果为True？Python里面不为`0`，`None`或者`null`都是True。所以结果就是，偶数是False，奇数是True，返回所有奇数。

**例3** 返回`s = ['Hi Steven!','Hi David','How are you?']`中含有'Hi'的字符串。

```python
print filter(lambda x:x.count('Hi'),s)
```

count计算'Hi'的出现次数，只要不是0的都会返回。

filter的功能也可以用list comprehension来实现。

```python
print [i for i in s if i.count('Hi')]
```

也是非常简洁。如果用普通的for循环，虽然也是简单，但可能就要多写几行了。

<h4>3. reduce</h4>
`reduce(func,seq)`则是对seq中的每个元素进行func操作，最后返回一个值。

**例4** 求`array=[7,9,13,4,5]`所有元素的乘积。

```python
print reduce(lambda x,y:x*y,array)
```

这里的lambda函数有两个参数x和y，reduce会先将array里面的头两个数分别作为x和y，求它们的乘积，然后把它的结果和第三个相乘，再把结果和第四个相乘，直到最后一个元素。

**例5** 找出`array=[7,9,13,4,5]`中的最大值。

```python
print reduce(lambda a,b:a if a>b else b,array)
```

这和built-in的`max`是一样的。甚至你也可以这样：
```python
print reduce(max,array)
```

当然这是多此一举了。

**例6** 求`array=[100,2,3,4,5]`的平方和。

估算一下结果应该是一万多，但如果你是这样写的：

```python
print reduce(lambda x,y:x**2+y**2,array)
```

会发现输出一个天文数字：`100320484445070891742619928410906`。

为什么？这里要注意，reduce总是先取前两个数，然后再将结果和第三个的平方相乘，也就是类似这样`square(square(suqare(x1,x2),x3),x4)`。所以上面的结果是不断把之前的结果平方，所以自然结果就很大。

所以正确的写法可能是下面这样？

```python
print reduce(lambda x,y:x+y**2,array)
```
但你会发现，结果是154，不是一万多。很显然，后面的平方和是正常的，但第一个数没有变平方，而是只把它自己本身加进去了。
所以为了避免这种情况，reduce允许你传入第三个参数，设定初始值，像下面这样：
```python
print reduce(lambda x,y:x+y**2,array,0)
```

这样第一次的时候，reduce会取`0+100^2`，然后再把结果和`2^2`加起来，一直到最后一个元素。这样结果就正确了。这里的初始值其实相当于在数组最前面加一个冗余的值，而假如你把初始值设为1，就等于你在计算array各个元素的平方和+1了。

**例7** 统计sentences中Sam出现的总次数。
(此题来自Mary Rose Cook的[A practical introduction to functional progarmming](http://maryrosecook.com/blog/post/a-practical-introduction-to-functional-programming))

```python
sentences = ['Mary read a story to Sam and Isla.',
             'Isla cuddled Sam.',
             'Sam chortled.']
```

```python
print reduce(lambda a,x:a+x.count('Sam'),sentences,0)
```

这里的初始值就是必须的。不然，sentences的第一个元素甚至都不是一个“次数”的量，而是字符串，那么当它和后面的整数相加时就会出错（即使不出错，结果也没有意义）。

<h4>4. 混合使用</h4>

**例8** 求`array=[14,7,26,19,25,74,33,10,35,28]`中能被7整除的数的立方和。

很显然，我们要过滤掉不是7的倍数的那些数，于是我们要用filter，然后我们要求和，要用到reduce。

```python
print reduce(lambda a,x:a+x**3,filter(lambda x:x%7==0,array),0)
```

map和filter也同样可以结合起来用。

**例9** 返回1-100的数中不能被2和13整除的数的平方。

```python
print map(lambda x:x*x,filter(lambda x:x%2!=0 and x%13!=0,range(1,101)))
```

<h4>勘误和交流</h4>
匆忙写下的一个笔记，出错简直是一定的。如果您发现了任何错误或者有关于本文的任何建议，麻烦发邮件给我（stevenslxie at gmail.com）或者在GitHub上直接交流，不胜感激。

<h4>转载声明</h4>
如果你喜欢这篇文章，可以随意转载。但请
<ul>
<li>标明原作者StevenSLXie;</li>
<li>标明原链接(https://github.com/StevenSLXie/Tutorials-for-Web-Developers/blob/master/Python%E7%9A%84map%7Cfilter%7Creduce%E5%B8%B8%E8%A7%81%E7%94%A8%E6%B3%95%E4%BE%8B%E4%B8%BE.md);</li>
<li>在可能的情况下请保持文本显示的美观。比如，请不要直接一键复制到博客之类，因为代码的显示效果可能非常糟糕;</li>
<li>请将这个转载声明包含进来；</li>
</ul>


<h4>还有一件事</h4>
如果你阅读完这篇文章，觉得很有收获。可以：
<ul>
<li>在GitHub上的这个repo打星，因为我之后还会陆续更新。这样可能方便一点，对我更是一种鼓励；</li>
<li>如果你是V2EX过来的，可以在那里表示感谢；</li>

Happy Coding!
</ul>


