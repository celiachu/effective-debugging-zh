# 改变运行和副作用

调试器通常用来观察一个被跟踪的进程的状态。它也可以改变debugee的状态，从改变它原本要执行的正常运行。这个方法创新性的使用是无限的。比如，验证当内存消耗完的时候程序的错误错误，调试器可以简单地设置函数`malloc`的返回值为`NULL`。它是快速和实惠的方式检查一些难以或者昂贵地模拟的特别案例。



调试器提供了一些改变程序运行的方式。最简单的是设置变量一个新的值。调试器通过调试符号弄清楚一个变量的内存地址，接着通过内核的帮助如`ptrace`方法覆盖目标进程的内存。下面的命令赋予变量`gFlags`值5。

```
(gdb)set var gFlags=5
```

改变线程的上下文肯定会改变程序的运行。比如，程序计数器，也就是下一条运行的指令，可以设置为另外一个指令地址。这个特性一个常用情形是重新运行一段已经运行过的代码，用来更近地查看发生了什么。如果重新运行的代码包含了一个断点，它会再次被碰到。下面的命令设置当前的线程恢复到文件`foo.c`第123行运行。
```
(gdb)jump foo.c:123
```

上面的命令仅仅改变线程的程序计数器。剩下的线程上下文不变。当前函数的栈帧仍然是线程栈的顶上帧。如果用户在上面的`jump`命令提供了另外一个函数的地址，后果是不可预测地，取决于两个函数的参数和局部变量是如何布局的。除非你知道函数调用习惯的所有细节，跳到另外一个函数不是一个好的主意。

当被调试进程已经停止，你可在调试器内调用任何函数。调试器在当前线程的最里层的帧为调用的函数创建一个新的栈帧。注意，调用C++类方式有点特别，因为它“秘密地”把`this`指针当成了被调用函数的第一个参数。另外一方面，一个C函数更直观。下面的例子调用了函数`malloc`来分配8字节内存块。调试器打印出返回的内存块地址。

```
(gdb) print /x malloc(8)
$1 = 0x501010

```
如果调用的函数有副作用，那么它会暗地里改变程序的行为。比如，下面的条件监测点开启一个拼凑的追踪和日志。每次变量`sum`改变的时候，gdb命令会调用函数`Logme`。

```
(gdb)watch sum if Logme(sum) > 0
```

