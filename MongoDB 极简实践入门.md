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
在MongoDB里，每篇博客文章以一个文档(document)的形式保存起来，而文档内部包含了很多项目，比如`title tags`等，每一个项目都是`key-value`的形式，即有一个项目的名字，比如`title`，以及它的值`TITLE_OF_POST`。而重要的是，一个`key`可以有多个`values`，他们用`[]`括起来。

这种“宽松”的数据存储形式非常灵活，MongoDB不限制每个`key`对应的`values`的数目。比如有的文章没有评论，则它的值就是一个空集，完全没有问题；有的文章评论很多，也可以无限制地插入。更灵活的是，MongoDB不要求同一个集合(collection，相当于SQL的table)里面的不同document有相同的key，比如除了上述这种文档组织，有的文档所代表的文章可能没有likes这个项目，再比如有的文章可能有更多的项目，比如可能还有dislikes等等。这些不同的文档都可以灵活地存储在同一个集合下，而且查询起来也异常简单，因为都在一个文档里，不用进行各种跨文档查询。而这种MongoDB式的存储也方便了数据的维护，对于一篇博客文章来说，所有的相关数据都在这个document里面，不用去考虑一个数据操作需要involve多少个表格。

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

<h4>3. 安装与环境</h4>

MongoDB可以在Windows、Linux、Mac OS X等主流平台运行，而且下载和安装非常简单，非常友好。这篇文档的例子采用MongoDB 2.6版本，均在OS X测试过，有充足的理由相信，在其它平台也能顺利运行。

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

<h4>4. 创建集合和删除集合</h4>
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

<h4>5. 插入</h4>

想象一个精简版的“豆瓣电影”。我们需要创建一个数据库，来存储每部电影的信息，电影的信息包括：

<ul>
<li>电影名字</li>
<li>导演</li>
<li>主演(可能多个)</li>
<li>类型标签(可能多个)</li>
<li>上映日期</li>
<li>喜欢人数</li>
<li>不喜欢人数</li>
<li>用户评论(可能多个)</li>
</ul>

显然我们需要先创建一个叫电影的集合：

```
db.createCollection('movie')
```

然后，我们就可以插入数据了：

```
db.movie.insert(
 {
   title: 'Forrest Gump', 
   directed_by: 'Robert Zemeckis',
   stars: ['Tom Hanks', 'Robin Wright', 'Gary Sinise'],
   tags: ['drama', 'romance'],
   debut: new Date(1994,7,6,0,0),
   likes: 864367,
   dislikes: 30127,
   comments: [	
      {
         user:'user1',
         message: 'My first comment',
         dateCreated: new Date(2013,11,10,2,35),
         like: 0 
      },
      {
         user:'user2',
         message: 'My first comment too!',
         dateCreated: new Date(2013,11,11,6,20),
         like: 0 
      }
   ]
}
)
```

请注意，这里插入数据之前，我们并不需要先声明movie这个集合里面有哪些项目。我们直接插入就可以了~这一点和SQL不一样，SQL必须先声明一个table里面有哪些列，而MongoDB不需要。

把上面的例子复制进命令行应该可以顺利运行，但我强烈建议你手动打一下，或者输入一部你自己喜欢的电影。`insert`操作有几点需要注意：
<ul>
<li>1. 不同key-value需要用逗号隔开，而key:value中间是用冒号；</li>
<li>2. 如果一个key有多个value，value要用[]。哪怕当前只有一个value，也加上[]以备后续的添加；</li>
<li>3. 整个“数据块”要用{}括起来；</li>
</ul>

如果你在`insert`之后看到`WriteResult({ "nInserted" : 1 })`，说明写入成功。

这个时候你可以用查询的方式来返回数据库中的数据：
```
db.movie.find().pretty()
```

这里`find()`里面是空的，说明我们不做限制和筛选，类似于SQL没有`WHERE`语句一样。而`pretty()`输出的是经格式美化后的数据，你可以自己试试没有`pretty()`会怎么样。

仔细观察`find()`的结果，你会发现多了一个叫`'_id'`的东西，这是数据库自动创建的一个ID号，在同一个数据库里，每个文档的ID号都是不同的。

我们也可以同时输入多个数据：
```
db.movie.insert([
 {
   title: 'Fight Club', 
   directed_by: 'David Fincher',
   stars: ['Brad Pitt', 'Edward Norton', 'Helena Bonham Carter'],
   tags: 'drama',
   debut: new Date(1999,10,15,0,0),
   likes: 224360,
   dislikes: 40127,
   comments: [	
      {
         user:'user3',
         message: 'My first comment',
         dateCreated: new Date(2008,09,13,2,35),
         like: 0 
      },
      {
         user:'user2',
         message: 'My first comment too!',
         dateCreated: new Date(2003,10,11,6,20),
         like: 14 
      },
      {
         user:'user7',
         message: 'Good Movie!',
         dateCreated: new Date(2009,10,11,6,20),
         like: 2
      }
   ]
},
{
   title: 'Seven', 
   directed_by: 'David Fincher',
   stars: ['Morgan Freeman', 'Brad Pitt',  'Kevin Spacey'],
   tags: ['drama','mystery','thiller'],
   debut: new Date(1995,9,22,0,0),
   likes: 134370,
   dislikes: 1037,
   comments: [	
      {
         user:'user3',
         message: 'Love Kevin Spacey',
         dateCreated: new Date(2002,09,13,2,35),
         like: 0 
      },
      {
         user:'user2',
         message: 'Good works!',
         dateCreated: new Date(2013,10,21,6,20),
         like: 14 
      },
      {
         user:'user7',
         message: 'Good Movie!',
         dateCreated: new Date(2009,10,11,6,20),
         like: 2
      }
   ]
}
])
```

顺利的话会显示：
```
BulkWriteResult({
	"writeErrors" : [ ],
	"writeConcernErrors" : [ ],
	"nInserted" : 2,
	"nUpserted" : 0,
	"nMatched" : 0,
	"nModified" : 0,
	"nRemoved" : 0,
	"upserted" : [ ]
```

表面我们成功地插入了两个数据。注意批量插入的格式是这样的：`db.movie.insert([{ITEM1},{ITEM2}])`。几部电影的外面需要用[]括起来。

请注意，虽然collection的插入不需要先声明，但表达相同意思的key，名字要一样，比如，如果我们在一个文档里用`directed_by`来表示导演，则在其它文档也要保持同样的名字(而不是`director`之类的)。不同的名字不是不可以，技术上完全可行，但会给查询和更新带来困难。

好了，到这里，我们就有了一个叫tutorial的数据库，里面有一个叫movie的集合，而movie里面有三个记录。接下来我们就可以对其进行查询了。

<h4>6. 查询</h4>

在上一节我们已经接触到最简单的查询`db.movie.find().pretty()`。MongoDB支持各种各样的深度查询功能。先来一个最简单的例子，找出大卫芬奇(David Fincher)导演的所有电影：

```
db.movie.find({'directed_by':'David Fincher'}).pretty()
```

将返回《搏击俱乐部》和《七宗罪》两部电影。这种搜索和SQL的`WHERE`语句是很相似的。

也可以设置多个条件。比如找出大卫芬奇导演的, 摩根弗里曼主演的电影：

```
db.movie.find({'directed_by':'David Fincher', 'stars':'Morgan Freeman'}).pretty()
```

这里两个条件之间，是AND的关系，只有同时满足两个条件的电影才会被输出。同理，可以设置多个的条件，不赘述。

条件之间也可以是或的关系，比如找出罗宾怀特或摩根弗里曼主演的电影：

```
db.movie.find(
{
  $or: 
     [  {'stars':'Robin Wright'}, 
        {'stars':'Morgan Freeman'}
     ]
}).pretty()
```

注意这里面稍显复杂的各种括号。

还可以设置一个范围的搜索，比如找出50万人以上赞的电影：

```
db.movie.find({'likes':{$gt:500000}}).pretty()
```

同样要注意略复杂的括号。注意，在这些查询里，key的单引号都是可选的，也就是说，上述语句也可以写成：
```
db.movie.find({likes:{$gt:500000}}).pretty()
```

类似地，少于二十万人赞的电影：

```
db.movie.find({likes:{$lt:200000}}).pretty()
```

类似的运算符还有：`$lte`:小于或等于；`$gte`:大于或等于；`$ne`:不等于。

注意，对于包含多个值的key，同样可以用find来查询。比如：

```
db.movie.find({'tags':'romance'})
```

将返回《阿甘正传》，虽然其标签既有romance，又有drama，但只要符合一个就可以了。

如果你确切地知道返回的结果只有一个，也可以用`findOne`:

```
db.movie.findOne({'title':'Forrest Gump'})
```

如果有多个结果，则会按磁盘存储顺序返回第一个。请注意，`findOne()`自带pretty模式，所以不能再加`pretty()`，将报错。

如果结果很多而你只想显示其中一部分，可以用`limit()`和`skip()`，前者指明输出的个数，后者指明从第二个结果开始数。比如：

```
db.movie.find().limit(2).skip(1).pretty()
```

则跳过第一部，从第二部开始选取两部电影。

<h4>7. 局部查询</h4>

第五节的时候我们讲了`find`的用法，但对于符合条件的条目，我们都是返回整个JSON文件的。这类似于SQL里面的`SELECT *`。有的时候，我们需要的，仅仅是部分数据，这个时候，`find`的局部查询的功能就派上用场了。先来看一个例子，返回tags为drama的电影的名字和首映日期。

```
db.movie.find({'tags':'drama'},{'debut':1,'title':1}).pretty()
```

数据库将返回：

```
{
	"_id" : ObjectId("549cfb42f685c085f1dd47d4"),
	"title" : "Forrest Gump",
	"debut" : ISODate("1994-08-05T16:00:00Z")
}
{
	"_id" : ObjectId("549cff96f685c085f1dd47d6"),
	"title" : "Fight Club",
	"debut" : ISODate("1999-11-14T16:00:00Z")
}
{
	"_id" : ObjectId("549cff96f685c085f1dd47d7"),
	"title" : "Seven",
	"debut" : ISODate("1995-10-21T16:00:00Z")
}
```

这里find的第二个参数是用来控制输出的，1表示要返回，而0则表示不返回。默认值是0，但`_id`是例外，因此如果你不想输出`_id`，需要显式地声明：

```
db.movie.find({'tags':'drama'},{'debut':1,'title':1,'_id':0}).pretty()
```


<h4>8. 更新</h4>
很多情况下你需要更新你的数据库，比如有人对某部电影点了个赞，那么你需要更新相应的数据库。比如有人对《七宗罪》点了个赞，而它本来的赞的个数是134370，那么你需要更新到134371。可以这样操作：

```
db.movie.update({title:'Seven'}, {$set:{likes:134371}})
```

第一个大括号里表明要选取的对象，第二个表明要改动的数据。请注意上述的操作相当不现实，因为你首先要知道之前的数字是多少，然后加一，但通常你不读取数据库的话，是不会知道这个数(134370)的。MongoDB提供了一种简便的方法，可以对现有条目进行增量操作。假设又有人对《七宗罪》点了两个赞，则可以：

```
db.movie.update({title:'Seven'}, {$inc:{likes:2}})
```

如果你查询的话，会发现点赞数变为134373了，这里用的是`$inc`。除了增量更新，MongoDB还提供了很多灵活的更新选项，具体可以看：http://docs.mongodb.org/manual/reference/operator/update-field/ 。

注意如果有多部符合要求的电影。则默认只会更新第一个。如果要多个同时更新，要设置`{multi:true}`，像下面这样：
```
db.movie.update({}, {$inc:{likes:10}},{multi:true})
```

所有电影的赞数都多了10.

注意，以上的更新操作会替换掉原来的值，所以如果你是想在原有的值得基础上增加一个值的话，则应该用`$push`，比如，为《七宗罪》添加一个popular的tags。

```
db.movie.update({'title':'Seven'}, {$push:{'tags':'popular'}})
```
你会发现《七宗罪》现在有四个标签：

```
	"tags" : [
		"drama",
		"mystery",
		"thiller",
		"popular"
	],

```
<h4>9. 删除</h4>

删除的句法和find很相似，比如，要删除标签为romance的电影，则：
```
db.movie.remove({'tags':'romance'})
```

考虑到我们数据库条目异常稀少，就不建议你执行这条命令了~

注意，上面的例子会删除所有标签包含romance的电影。如果你只想删除第一个，则
```
db.movie.remove({'tags':'romance'},1)
```

如果不加任何限制：

```
db.movie.remove()
```

会删除movie这个集合下的所有文档。


<h4>10. 索引和排序</h4>

为文档中的一些key加上索引(index)可以加快搜索速度。这一点不难理解，假如没有没有索引，我们要查找名字为Seven的电影，就必须在所有文档里逐个搜索。而如果对名字这个key加上索引值，则电影名这个字符串和数字建立了映射，这样在搜索的时候就会快很多。排序的时候也是如此，不赘述。MongoDB里面为某个key加上索引的方式很简单，比如我们要对导演这个key加索引，则可以：
```
db.movie.ensureIndex({directed_by:1})
```

这里的1是升序索引，如果要降序索引，用-1。

MongoDB支持对输出进行排序，比如按名字排序：
```
db.movie.find().sort({'title':1}).pretty()
```

同样地，1是升序，-1是降序。默认是1。

```
db.movie.getIndexes()
```

将返回所有索引，包括其名字。

而

```
db.movie.dropIndex('index_name')
```

将删除对应的索引。

<h4>11. 聚合</h4>

MongoDB支持类似于SQL里面的`GROUP BY`操作。比如当有一张学生成绩的明细表时，我们可以找出每个分数段的学生各有多少。为了实现这个操作，我们需要稍加改动我们的数据库。执行以下三条命令：

```
db.movie.update({title:'Seven'},{$set:{grade:1}})
db.movie.update({title:'Forrest Gump'},{$set:{grade:1}})
db.movie.update({title:'Fight Club'},{$set:{grade:2}})
```

这几条是给每部电影加一个虚拟的分级，前两部是归类是一级，后一部是二级。

这里你也可以看到MongoDB的强大之处：可以动态地后续添加各种新项目。

我们先通过聚合来找出总共有几种级别。

```
db.movie.aggregate([{$group:{_id:'$grade'}}])
```

输出：

```
{ "_id" : 2 }
{ "_id" : 1 }
```

注意这里的2和1是指级别，而不是每个级别的电影数。这个例子看得清楚些：

```
db.movie.aggregate([{$group:{_id:'$directed_by'}}])
```

这里按照导演名字进行聚合。输出：

```
{ "_id" : "David Fincher" }
{ "_id" : "Robert Zemeckis" }
```

接着我们要找出，每个导演的电影数分别有多少：

```
db.movie.aggregate([{$group:{_id:'$directed_by',num_movie:{$sum:1}}}])
```

将会输出：

```
{ "_id" : "David Fincher", "num_movie" : 2 }
{ "_id" : "Robert Zemeckis", "num_movie" : 1 }
```

注意$sum后面的1表示只是把电影数加起来，但我们也可以统计别的数据，比如两位导演谁的赞比较多：

```
 db.movie.aggregate([{$group:{_id:'$directed_by',num_likes:{$sum:'$likes'}}}])
```

输出：

```
{ "_id" : "David Fincher", "num_likes" : 358753 }
{ "_id" : "Robert Zemeckis", "num_likes" : 864377 }
```

注意这些数据都纯属虚构啊！

除了`$sum`，还有其它一些操作。比如：
```
db.movie.aggregate([{$group:{_id:'$directed_by',num_movie:{$avg:'$likes'}}}])
```

统计平均的赞。

```
db.movie.aggregate([{$group:{_id:'$directed_by',num_movie:{$first:'$likes'}}}])
```

返回每个导演的电影中的第一部的赞数。

其它各种操作可以参考：http://docs.mongodb.org/manual/reference/operator/aggregation/group/ 。

<h4>12. All or Nothing?</h4>

MongoDB支持单个文档内的原子化操作(atomic operation)，这是说，可以将多条关于同一个文档的指令放到一起，他们要么一起执行，要么都不执行。而不会执行到一半。有些场合需要确保多条执行一起顺次执行。比如一个场景：一个电商网站，用户查询某种商品的剩余数量，以及用户购买该种商品，这两个操作，必须放在一起执行。不然的话，假定我们先执行剩余数量的查询，这是假定为1，用户接着购买，但假如这两个操作之间还加入了其它操作，比如另一个用户抢先购买了，那么原先购买用户的购买的行为就会造成数据库的错误，因为实际上这种商品以及没有存货了。但因为查询剩余数量和购买不是在一个“原子化操作”之内，因此会发生这样的错误<a href="http://www.tutorialspoint.com/mongodb/mongodb_atomic_operations.htm">[2]</a>。

MongoDB提供了`findAndModify`的方法来确保atomic operation。比如这样的：
```
db.movie.findAndModify(
			{
			query:{'title':'Forrest Gump'},
			update:{$inc:{likes:10}}
			}
		      )

```
query是查找出匹配的文档，和find是一样的，而update则是更新likes这个项目。注意由于MongoDB只支持单个文档的atomic operation，因此如果query出多于一个文档，则只会对第一个文档进行操作。

`findAndModify`还支持更多的操作，具体见：http://docs.mongodb.org/manual/reference/command/findAndModify/。


<h4>13. 文本搜索</h4>

除了前面介绍的各种深度查询功能，MongoDB还支持文本搜索。对文本搜索之前，我们需要先对要搜索的key建立一个text索引。假定我们要对标题进行文本搜索，我们可以先这样：

```
db.movie.ensureIndex({title:'text'})
```

接着我们就可以对标题进行文本搜索了，比如，查找带有"Gump"的标题：

```
db.movie.find({$text:{$search:"Gump"}}).pretty()
```

注意text和search前面的$符号。

这个例子里，文本搜索作用不是非常明显。但假设我们要搜索的key是一个长长的文档，这种text search的方便性就显现出来了。MongoDB目前支持15种语言的文本搜索。

<h4>14. 正则表达式</h4>

MongoDB还支持基于正则表达式的查询。如果不知道正则表达式是什么，可以参考<a href="http://en.wikipedia.org/wiki/Regular_expression">Wikipedia</a>。这里简单举几个例子。比如，查找标题以`b`结尾的电影信息：

```
db.movie.find({title:{$regex:'.*b$'}}).pretty()
```

也可以写成：

```
db.movie.find({title:/.*b$/}).pretty()
```

查找含有'Fight'标题的电影：

```
db.movie.find({title:/Fight/}).pretty()
```

注意以上匹配都是区分大小写的，如果你要让其不区分大小写，则可以：

```
db.movie.find({title:{$regex:'fight.*b',$options:'$i'}}).pretty()
```

`$i`是insensitive的意思。这样的话，即使是小写的fight，也能搜到了。

<h4>15. 后记</h4>

至此，MongoDB的最基本的内容就介绍得差不多了。如果有什么遗漏的以后我会补上來。如果你一路看到底完全了这个入门教程，恭喜你，你一定是一个有毅力的人。

把这个文档过一遍，不会让你变成一个MongoDB的专家(如果会那就太奇怪了)。但如果它能或多或少减少你上手的时间，或者让你意识到“咦，MongoDB其实没那么复杂”，那么这个教程的目的也就达到啦。

这个文档是匆忙写就的，出错简直是一定的。如果您发现了任何错误或者有关于本文的任何建议，麻烦发邮件给我（stevenslxie at gmail.com）或者在GitHub上直接交流，不胜感激。

<h4>转载声明</h4>
如果你喜欢这篇文章，可以随意转载。但请
<ul>
<li>标明原作者StevenSLXie;</li>
<li>标明原链接(https://github.com/StevenSLXie/Tutorials-for-Web-Developers/blob/master/MongoDB%20%E6%9E%81%E7%AE%80%E5%AE%9E%E8%B7%B5%E5%85%A5%E9%97%A8.md);</li>
<li>在可能的情况下请保持文本显示的美观。比如，请不要直接一键复制到博客之类，因为代码的显示效果可能非常糟糕;</li>
<li>请将这个转载声明包含进来；</li>
</ul>











