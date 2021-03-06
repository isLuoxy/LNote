# jvm 乱七八糟

## 1、java程序运行步骤

java文件经过 javac 编译器**编译**成带 .class 后缀的字节码文件**（这里的编译与一般的编译不同，java的编译是指把源代码编译成字节码，即把 .java 变成 .class 的类文件，这种类文件的特点是无关平台，不能直接运行，需要借助虚拟机 jvm ）**的字节码文件，然后通过jvm虚拟机加载字节码文件，并且逐行**解释**字节码文件（解释不形成中间文件，边解释、边执行），

## 2、jit（Just In Time）优化

在java中，有些方法或者代码块是高频率调用的，可以理解成热点代码，那么这样可以采用jit技术（运行时编译），提前将这些字节码**编译**成本地机器码（编译会形成中间文件），有点类似缓冲，运行时直接执行就行了，而不用先解释再执行。



## 3、指令重排序

处理器为了提高程序运行效率，可能会对输入代码进行优化，它不保证程序中各个语句的执行先后顺序同代码中的顺序一致，但它会保证程序最终运行结果和代码运行的结果是一致的。

在单线程中不会被重排序的逻辑

![](F:\一些笔记\JVM\pic\352511-20170814091904240-225568883.png)



## 4、happens-before原则（先行发生原则）

java内存模型具备一些先天的“有序性”，即不需要通过任何手段就能够得到保证的有序性，这个通常也称为happens-before原则，如果两个操作的执行次序无法从happens-before原则推导出来，那么他们就不能保证他们的有序性吗虚拟机就可以随意对它们进行重排序

**8条原则摘自《深入理解java虚拟机》**

- 程序次序规则：一个线程内按照代码顺序，书写在前面的操作先行发生于书写在后面的操作。
- 锁定规则：一个unlock操作先行发生与后面对一个锁额lock操作。
- volatile变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作。
- 传递规则：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生与操作C
- 线程启动规则：Thread对象的start（）方法先行发生与此线程的每个一个动作
- 线程中断规则：对线程interrupt（）方法的调用先行发生与被中断线程的代码检测到中断事件的发生
- 线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join（）方法结束、Thread.isAlive（）的返回值手段检测到线程已经终止执行
- 对象终结规则：一个对象的初始化完成先行发生与他的finalize（）方法的开始

