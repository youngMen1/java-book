# 第03讲：Linux 三剑客（上）

在本课时我将讲解 Linux 三剑客，以及它们的具体使用语法。

## Linux 三剑客简介      

首先，我们来了解下 Linux 三剑客具体指什么？

1.第一个工具是 grep，grep 会根据正则表达式查找相关内容并打印对应的数据。

2.第二个工具是 awk，awk 的名字来源于三个作者的名字简称，它可以根据定位到的数据行处理其中的分段。

3.第三个工具是 sed，它是 stream editor 流式编辑器的简称，可以定位到数据行并对数据进行增删改查操作。

因为它们三个组合使用的功能非常的强大，几乎完美的应对了 Shell 中的数据分析场景，于是人们把这三个工具统称为 Linux 三剑客。

## Linux 三剑客价值      

接下来，我们通过对三剑客与 SQL 进行类比，来具体看看它们到底能做些什么？

* grep 相当于 SQL 的 select *from table，它可以进行数据的查找与定位。

* awk 相当于 SQL 的 select field from table，它可以进行数据切片。

* sed 相当于 SQL 的 update table set field=new where field=old，它可以对数据进行修改。

你可以发现，grep 和 awk 可以进行组合使用，来达到查找数据并对数据进行分割的目的，grep 也可以与 sed 组合使用，达到查找数据并修改的目的，它们三个还可以组合在一起使用来完成一系列的操作，就相当于大数据处理中的 Map-Reduce，我们接下来看如何具体的使用它们。

### grep      

首先，我们来看下如何使用 grep，grep 用于根据正则表达式查找相关内容并打印对应的数据，我们打开 Shell 环境，通过 vim/tmp/hello.txt 命令创建一个文件，并在文件内输入三条数据：

* hello from hogwarts

* hello from sevenriby

* hello from testerhome

![](/static/image/CgoB5l3XtNuAPoP1AACZeigKKy8997.png)

然后通过 grep hogwarts /tmp/hello.txt 指令查找数据，指令中间的参数是正则表达式，指令后面的参数是文件名。

![](/static/image/CgotOV3XtPiAXjErAADUYrh9hPI032.png)


你可以看到 grep 把 hello from hogwarts 从文件中提取出来。

 ![](/static/image/CgoB5l3XtQuAMGB7AAFQI4atTN4017.png)
 
你还可以通过 grep 把 testerhome 提取出来，通过 cat 指令可以看到在 hello.txt 中有三行数据。 
  
![](/static/image/CgotOV3XtRqAHiTCAAIU-z_0BFc527.png)    

 
如果我们输入 grep hello 指令，它会把三条数据都提取出来。这就是 grep 的第一个作用，根据指定的正则表达式查取对应的数据，我们上面的演示用的是简单的字符串。

![](/static/image/CgotOV3XtSmAVa4mAAKQLiKRwF0939.png)

接下来，我们学习如何使用正则表达式获取以字母 s 或 t 开头的后面跟任意两个字符的数据，输入 grep "[st].." /tmp/hello.tex 指令，其中 [] 表示正则表达式，..表示后面跟任意的两个字符，你可以看到输出了两条数据。

![](/static/image/CgoB5l3XtTeAfsKQAAJKMXmUvhU283.png)

我们还可以通过 -o 指令只打印匹配的内容，输入 grep -o "[st].." /tmp/hello.tex 指令，你可以看到只打印了匹配到的内容，而不是整条数据。

       

grep 还有一些其他的指令，比如 -i 可以忽略字符大小写。  

![](/static/image/CgoB5l3XtUWAK5t-AAIxEAoUhtM359.png)


-v 表示将匹配到的内容过滤掉，比如我们输入 grep -v "[st].." /tmp/hello.tex 指令，只显示了一条数据，过滤掉了匹配正则条件的内容。


-o 表示只打印匹配的数据，-E 表示支持使用扩展正则表达式，我们接下来详细了解下 pattern 正则表达式。

我把正则表达式分为两类，第一类称为基本表达式，基本表达式包括了典型的正则标识符。

* ^表示开头；

* $表示结尾；

* []表示任意匹配项；

* *表示0个或多个；

* .表示任意字符。

第二类是扩展表达式，它在基础表达式的基础上做了一些扩展，支持了更高级的语法和更复杂的条件。
* ？表示非贪婪匹配；

* + 表示一个或多个；

* () 表示分组；

* {} 表示一个范围的约束；

* | 表示匹配多个表达式中的任何一个。   


![](/static/image/CgoB5l3XtVaAJPZaAAJpe4YZD9c323.png)

我们举个例子，输入 grep -E "(hog|home)" /tmp/hello.tex 指令，输出结果分别匹配了 hog 与 home。

![](/static/image/CgotOV3XtWSAF9K-AAKOZuR4qrg333.png)

但如果你的指令中不含 -E，则指令不支持扩展正则，这个时候你会发现它什么都匹配不到。 

![](/static/image/CgoB5l3XtXGAQRR7AAKVDfReUHg948.png)

如果不使用 -E，我们可以使用 \ 转义符对匹配条件进行转义，也可以达到同样的效果