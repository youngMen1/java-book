# 第04讲：Linux 三剑客（下）

本课时主要讲解 Linux 三剑客的更高级用法，也就是如何使用管道将三剑客连接使用？

## Shell 输入输出      

如果你想掌握本课时的内容，首先需要了解 Shell 的输入输出，在 Shell 里，每个指令其实都是一个进程，和我们的被测程序一样，这个进程也有输入和输出。 比如，你可以使用 read 读取输入，并赋值给变量，也可以使用 echo、print 输出变量。而在 Shell 输入输出指令里面有一个特别需要我们注意的功能叫作管道，用 | 表示，它可以将上一个指令的输出自动变成下一个指令的输入。

## 文件描述符

接下来，我们来看一下管道的具体用法。如图所示，Shell 下任何程序都有输入和输出，这里需要额外注意的是错误输出，比如我们输入 ls dddd 指令。

CgoB5l3czBiAQrPTAAJmF953Il8386.png

因为 dddd 文件是不存在的，所以会打印了一个独立显示的报错信息，我们就称之为错误输出。

CgotOV3czCaAPOigAAI94SOIP8Q284.png

我们现在输入 ls -l /tmp appium.log 指令，可以打印一个正确的输出。

CgotOV3czDOAQH4QAAJeXeuP_OE959.png

而输入是一个读取文件的过程，比如我们输入 grep "hogwarts" /tmp/hello.txt 指令便是从 hello.txt 文件中读取 hogwarts。