[toc]

# vim杂记

* ``:%!xxd`` 转换成二进制模式
* ```:%!grep xxx``从vim中筛选
* vim - 管道输出给vim
* a.out | head -n 10000

## NORMAL模式下

```sh
:w test.txt // 保存文件
0 // 行首，含空格
^ // 行首，不含空格
$ // 行尾
W、B、E // 无视其他符号，只看空格

viw // 选中当前单词
ctrl+u ctrl+d ctrl+y ctrl+e // 滚动
ctrl+o ctrl+i // 历史跳转
gf // 头文件跳转，然后ctrl+o回来

// 操作基本都可以加上数字来重复
5i3 // 插入5个3
vi)
va)
% // 符号之间跳转
{}/() // 段落、函数之间跳转

// 删除
bvex // 可视化删除单词
bdw // 删除单词
D等于d$ // 删至末尾
C等于c$
bvec // 可视化删除并插入

// x直接删除，d后面要接选择

// 查找
停在一个单词(b)按shift+*或shift+#
f+字母（用分号查找下一个）
F+字母 	// 反向查找
t+字母  // 找到字符的前一个位置
cf+字母 // 组合技

// 编辑
I、A

// 批量操作
利用大写V选择一些行，然后按冒号:(冒号后面的是标记，代表选择的行，可以自行更改行号)，输入norm，然后输入命令如0x，按回车，批量执行0x。
// 利用宏
按q开始，接着按u，将命令存入u寄存器中，然后输入0xj，最后按q结束。使用的时候按@u，4@u 5@@

// 替换
:s/old/new/g(g是选项) // 单行替换
:%s/old/new/g  // 全局替换
// 若替换的字符有/， 可以用反斜杠\来转义

// 快速删除行尾空格
选中行，然后:s/\s*$//  // \s匹配任何空白字符
r + 字符 // 替换当前字符

// 缩进
选中行，然后按= // 将没有缩进的行搞成缩进
>> // 添加缩进
>i{ // 当前括号下的段落缩进
<< // 减少缩进

// 复制
在外面复制的代码，在插入模式下，按ctrl+v
p P
x p 交换两个字符的位置

// 执行命令
:sh // 进入临时的shell
exit 或 ctrl+d // 退出shell
:! + 命令 // %号可代替文件名

// 其他
:ls // 打开缓冲区
:b+编号 // 跳转不同缓冲区
:bn :bp // 上一个/下一个缓冲区
home // 跳转命令首字符
```

## 配置

```
set ts=4 // tab大小为4
set sw=4 // 软件生成的大小
set et // 将tab由空格组成
```



# GDB杂记

* starti 从第一条命令开始
* 打印进程内存 info proc {mappings,...}

```
n // 不进入函数
s // 进入函数
d + 编号 // 删除断点

b 打断点，如b Myclass::func 
info b // 查看断点
b xxx if xxxxx // 条件断点
tb  // 临时断点
save breakpoints xx.gdb // 保存断点
source xx.gdb // 加载断点

watch + 变量 // 变量被修改时，会中断
watch xx if xxx
awatch xx // 读取也能被中断
rwatch xx // 只读取被中断

p + 要打印的变量或函数(会执行)
p get_size() // 会执行get_size()
p *this 打印类的成员变量
p name.operator=("name!!!") // 修改变量

k // 杀死调试进程

r -xx -xx // 带参数运行

// 调试python
1. gdb python
2. r xx.py

// 另一种启动
1.gdb
2.file xxx

u main.cpp:33 // 运行到33行

// 调试运行中的程序
1.先找到pid, pidof xxx 或 ps |grep
2.sudo gdb
3.attach pid

backward-cpp库 可自动打印出错时的栈信息

// 多线程
set scheduler-locking on // 设置多线程调试时，step只执行单个线程
info threads
thread x
```

## GCC

* gcc -E 展开宏
* gcc -c 只编译文件，不链接，可通过ld来链接
* strace -f gcc xxx.c
