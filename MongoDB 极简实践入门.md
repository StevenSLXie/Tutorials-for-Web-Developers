MongoDB 极简实践入门
==========================

<h4>1. 为什么用MongoDB？</h4>
传统的计算机应用大多使用关系型数据库来存储数据，比如大家可能熟悉的MySql, Sqlite等等，它的特点是数据以表格(table)的形式储存起来的。数据库由一张张排列整齐的表格构成，就好像一个Excel表单一样，每个表格会有若干列，比如一个学生信息表，可能包含学号、姓名、性别、入学年份、高考成绩、籍贯等等。而表格的每一排，则是一个个学生的具体信息。在企业级应用和前互联网时代，关系型数据库几乎是不二选择。关系型数据库的特点是有整齐划一的组织，很方便对数据进行描述、插入、搜索。

想象有一个传统的网上服装商店吧，它的主要的数据可能是储存在一张叫products的表单里，表单可能包含这些列：商品编号(ID)、名称(Name)、商家(brand)、主目录(cate)、子目录(sub-cat)、零售价(price)、是否促销(promotion)等等。如果有一个用户想要查找所有价格低于300元的正在促销的鞋子的编号和名称，则可以执行类似于以下的SQL语句：
```
SELECT ID, name FROM products WHERE cate='shoes' AND price<300 and AND promotion=true;
```

SQL具备了强大了的深度查询能力，能满足各式各样的查询要求。而如果要对数据进行添加和删除，成本也是非常低的。这些是SQL的优势之一， 但随着互联网的兴起以及数据形式的多样化，四平八稳的SQL表单在一些领域渐渐显现出它的劣势。让我们通过一个例子来说明。考虑一个博客后台系统，如果我们用关系型数据库为每篇博客(article)建一个表单的话，这个表单大概会包括以下这些列：

| ID | Title  | Description       | Author | Content  | Likes |
|----|:-----: |:-----------------:|:------:|:-------: |:-----:|
| A_1  | Title1 | Political Article | Joe    | Content 1| 12    |
| A_2  | Title2 | Humorous Story    | Sam    | Content 2| 50    |

这时候用SQL数据库来存储是非常方便的，但假如我们要位每篇文章添加评论功能，会发现每篇文章可能要多篇评论，而且这个数目是动态变化的，而且每篇评论还包括好几项内容：评论的人、评论的时间、以及评论内容。这时候要将这些内容都塞进上述的那个表，就显得很困难。通常的做法是为评论(comment)单独建一个表：

| ID | Author  | Time   | Content  | Article |
|----|:-----: |:-----------------:|:------:|:------:|
| C_1  | Anna | 2014-12-26 08:23 | Really good articles! | A_1 | 
| C_2  | David | 2014-12-25 09:30     | I like it! |  A_1 |

类似地，每篇文章可能会有若干标签(tags)。标签本身又是一个表单：

| ID | Category  | Tags   | Content  | Article |
|----|:-----: |:-----------------:|:------:|:------:|
| T_1  | Anna | 2014-12-26 08:23 | Really good articles!|  A_1 |
| T_2  | David | 2014-12-25 09:30     | I like it!| A_2 |

而博客的表格则要通过foreign key跟这些相关联的表格联系起来(可能还包括作者、出版社等其它表格)。这样一来，当我们做查询的时候，比如说，“找出评论数不少于3的标签为‘政治评论’的作者为Sam的文章”，就会涉及到复杂的跨表查询，需要大量使用`join`语句。这种跨表查询不仅降低了查询速度，而且这些语句写起来也不简单。

那么，如果用MongoDB数据库来实现，可以如何设计数据模型呢？很简单，像下面这样<a href="http://www.tutorialspoint.com/mongodb/mongodb_data_modeling.htm">[1]</a>：

```
 _id: POST_ID
   title: TITLE_OF_POST, 
   description: POST_DESCRIPTION,
   author: POST_BY,
   tags: [TAG1, TAG2, TAG3],
   likes: TOTAL_LIKES, 
   comments: [	
      {
         user:'COMMENT_BY',
         message: TEXT,
         dateCreated: DATE_TIME,
      },
      {
         user:'COMMENT_BY',
         message: TEXT,
         dateCreated: DATE_TIME,
      }
   ]
```
在MongoDB里，每篇博客文章以一个文件(document)的形式保存起来，而文件内部包含了很多项目，比如`title tags`等，每一个项目都是`key-value`的形式，即有一个项目的名字，比如`title`，以及它的值`TITLE_OF_POST`。而重要的是，一个`key`可以有多个`values`，他们用`[]`括起来。

这种“宽松”的数据存储形式非常灵活，MongoDB不限制每个`key`对应的`values`的数目。比如有的文章没有评论，则它的值就是一个空集，完全没有问题；有的文章评论很多，也可以无限制地插入。更灵活的是，MongoDB不要求同一个集合(collection，相当于SQL的table)里面的不同document有相同的key，比如除了上述这种文件组织，有的文件所代表的文章可能没有likes这个项目，再比如有的文章可能有更多的项目，比如可能还有dislikes等等。这些不同的文件都可以灵活地存储在同一个集合下，而且查询起来也异常简单，因为都在一个文件里，不用进行各种跨文件查询。而这种MongoDB式的存储也方便了数据的维护，对于一篇博客文章来说，所有的相关数据都在这个document里面，不用去考虑一个数据操作需要involve多少个表格。

当然，除了上述的优点，MongoDB还有不少别的优势，比如MongoDB的数据是用JSON(Javascript Object Notation)存储的(就是上面的这种key-value的形式)，而几乎所有的web应用都是基于Javascript的。因此，存储的数据和应用的数据的格式是高度一致的，不需经过转换。更多的优点可以查看：<a href="http://www.tutorialspoint.com/mongodb/mongodb_advantages.htm">[2]</a>。

<h4>2. 关于这篇文章</h4>

这个极简教程，或者说笔记，并不是一个覆盖MongoDB方方面面的教程。所谓极简的意思，就是只选取那些最重要、最常用的内容进行基于实例的介绍，从而让读者能够在最短的时间内快速上手，并且能顺利地进行后续的纵深的学习。

具体地说，这个教程的特点是：
<ul>
<li>不求全面，只求实用。只覆盖最核心的部分；</li>
<li>以大量例子为导向；</li>
<li>一边阅读一边动手操作的话，大约只需要2小时的时间；</li>
</ul>

阅读这篇文章不需要有特别的基础，但最好知道数据库的基本概念，如果本身熟悉SQL那就更好啦。

<h4>3. 安装与环境搭建</h4>

MongoDB可以在Windows、Linux、Mac OS X等主流平台运行，而且下载和安装非常简单，非常友好。这篇文档的例子均在OS X测试过，有充足的理由相信，在其它平台也能顺利运行。

Windows的安装和设置可以参考：http://www.w3cschool.cc/mongodb/mongodb-window-install.html；

Linux的安装和设置可以参考：http://www.w3cschool.cc/mongodb/mongodb-linux-install.html；

Mac OS X下的安装和设置：

<ul>
<li>1. 在https://www.mongodb.org/ 下载适合你的Mac的MongoDb;</li>
<li>2. 下载得到的文件是一个zip文件，解压，然后放到你想到的文件夹，比如/Users/Steven/MongoDB;</li>
<li>3. 创建一个你喜欢的文件夹来存储你的数据，比如/User/Steven/myData;</li>
<li>4. 打开Terminal，cd到2里面那个文件夹/Users/Steven/MongoDB，再cd bin;</li>
<li>5. 输入./mongod --dbpath /User/Steven/myData,等到出现类似“waiting for connections on port 27017”，说明MongoDB服务器已架设好，而数据将储存在myData里面；</li>
<li>6. 新打开一个Terminal, cd /Users/Steven/MongoDB/bin,然后运行./mongo;顺利的话它将出现一个interactive shell让你进行各种操作，而你的数据将储存在myData里</li>
</ul>

如果以上的各个步骤都运行顺利，就可以跳到下一节啦。

<h4>3. 创建集合和删除集合</h4>
在上一节执行完步骤6后，你会看到命令行里显示：`connecting to: test`，这里的`test`是默认的数据库。这里我们可以新建一个数据库。在命令行里打入：

```
use tutorial
```

这样就新建了一个叫做`tutorial`的数据库。你可以执行

```
show databases
```

来显示当前的数据库。不过这时候由于我们的新数据库是空的，所以会显示类似这样的：

```
admin  (empty)
local  0.078GB
```

我们试着往我们的数据库里添加一个集合(collection)，MongoDB里的集合和SQL里面的表格是类似的：

```
db.createCollection('author')
```

顺利的话会显示：

```
{ "ok" : 1 }
```

表示创建成功。

你可以再回头执行：

```
show databases
```

这时候我们的tutorial集合已经位列其中。你可以再执行

```
show collections
```

可以看到创建的集合author也在其中。

我们暂时不需要author这个集合，所以我们可以通过执行：

```
db.author.drop()
```

来将其删除。这时候你再执行`show collections`，就再也看不到我们的author了。

这一节要记住的点主要只有一个：集合(collection)类似于SQL的表格(table)，类似于Excel的一个个表格。

<h4>4. 插入</h4>

想象一个精简版的“豆瓣电影”。我们需要创建一个数据库，来存储每部电影的信息，电影的信息包括：

<ul>
<li>电影名字</li>
<li>导演</li>
<li>主演(可能多个)</li>
<li>类型标签(可能多个)</li>
<li>上映日期</li>
<li>票房</li>
<li>喜欢人数</li>
<li>不喜欢人数</li>
<li>用户评论(可能多个)</li>
</ul>

<h4>5. 查询</h4>
<h4>6. 更新</h4>
<h4>7. 条件操作</h4>
<h4>8. 排序和索引</h4>
<h4>9. 待续</h4>
