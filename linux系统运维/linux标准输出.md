# linxu 标准输出 #

我们在Linux下经常会碰到nohup command>/dev/null 2>&1 &这样形式的命令。

我们把这条命令大概分解下。

首先就是一个nohup表示当前用户和系统的会话下的进程忽略响应HUP消息。&是把该命令以后台的job的形式运行。

那么就剩下command>/dev/null 2>&1，command>/dev/null较好理解，/dev/null表示一个空设备，就是说吧command的执行结果重定向到空设备中，说白了就是不显示任何信息。那么2>&1又是什么含义?

## 几个基本符号及其含义 ##

- /dev/null 表示空设备文件
- 0 表示stdin标准输入
- 1 表示stdout标准输出
- 2 表示stderr标准错误

## 从command>/dev/null说起 ##

其实这条命令是一个缩写版，对于一个重定向命令，肯定是a > b这种形式，那么command > /dev/null难道是command充当a的角色，/dev/null充当b的角色。

其实一条命令肯定是充当不了a，肯定是command执行产生的输出来充当a，其实就是标准输出stdout。所以command > /dev/null相当于执行了command 1 > /dev/null。**执行command产生了标准输出stdout(用1表示)，重定向到/dev/null的设备文件中。**

## 说说2>&1 ##

通过上面command > /dev/null等价于command 1 > /dev/null,那么对于2>&1也就好理解了，2就是标准错误，1是标准输出，那么这条命令不就是相当于把标准错误重定向到标准输出么。等等，是&1而不是1，这里&是什么？这里&相当于等效于标准输出。这里有点不好理解，先看下面。

## command>a 2>a 与 command>a 2>&1的区别 ##

通过上面的分析，对于command>a 2>&1这条命令，等价于command 1>a 2>&1可以理解为执行command产生的标准输出重定向到文件a中，标准错误也重定向到文件a中。

那么是否就说command 1>a 2>&1等价于command 1>a 2>a呢。其实不是，command 1>a 2>&1与command 1>a 2>a还是有区别的，区别就在于前者只打开一次文件a，后者会打开文件两次，并导致stdout被stderr覆盖。

&1的含义就可以理解为标准输出的引用，引用的就是重定向标准输出产生打开的a。从IO效率上来讲，command 1>a 2>&1比command 1>a 2>a的效率更高。