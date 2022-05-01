
- [bootsect.s](#bootsects)

# bootsect.s
BIOS会将磁盘的第一个扇区中的内容（512字节）放到内存的0x7c00的位置。

```asm
_start:
	mov	ax,#BOOTSEG
	mov	ds,ax                  !ds=0x07c0
	mov	ax,#INITSEG
	mov	es,ax                  !es=0x9000
	mov	cx,#256                !cx = 256
	sub	si,si                  !si = 0
	sub	di,di                  !di = 0
	rep
	movw
```	
前两句话
```asm
mov	ax,#BOOTSEG
mov	ds,ax            
```
令ax=0x07c0, ds=0x07c0， 实际上的目的就是将ds设置为0x07c0

接着,
```asm
mov	ax,#INITSEG
mov	es,ax
```
令ax=0x9000, ds=0x9000，  实际上的目的就是将es设置为0x9000

最后的五句话实现了内存的迁移，我们逐一拆解。
```asm
mov	cx,#256                !cx = 256
sub	si,si                  !si = 0
sub	di,di                  !di = 0
rep
movw
```
```mov	cx,#256```： 汇编语言中cx通常用于计数器，这里也是，暗示后面的操作会进行256次。

```
sub	si,si                  !si = 0
sub	di,di
```
这两句话就是简单的清零操作。


最后```rep movw```代表重复操作movw指令。
movw 代表复制一个字（2 bytes）的数据
源地址是ds:si， 目的地址是es:di，总共复制256次。

因此rep movw 实现的功能就是将0x07c0:0x0000为起点的512字节的数据迁移到0x9000:0x0000处。

总结起来， 整个操作系统的代码的第一段实现的功能就是将内存中的内容进行迁移。

接下来使用段间跳转指令jmpi实现PC的跳转
```asm
jmpi	go,INITSEG
```
这里跳转到0x9000的go标签处，目前setup.s的程序已经拷贝到了0x9000处， 所以这句话实际上就是跳转到setup.s中的go标签处执行

```asm
go:	mov	ax,cs                  !cs=0x9000
	mov	ds,ax                  !ds=0x9000
	mov	es,ax                  !es=0x9000
! put stack at 0x9ff00.
	mov	ss,ax
	mov	sp,#0xFF00		! arbitrary value >>512
```

ss 为栈段寄存器， ss被赋值0x9000, sp被赋值0xFF00， 因此选择的栈顶的位置ss:sp 就是0x9FF00。



```asm
load_setup:
	mov	dx,#0x0000		! drive 0, head 0
	mov	cx,#0x0002		! sector 2, track 0
	mov	bx,#0x0200		! address = 512, in INITSEG
	mov	ax,#0x0200+SETUPLEN	! service 2, nr of sectors
	int	0x13			! read it
	jnc	ok_load_setup		! ok - continue
	mov	dx,#0x0000
	mov	ax,#0x0000		! reset the diskette
	int	0x13
	j	load_setup
```
BIOS INT 0x13中断读取磁盘到内存。　

**功能号**：AH = 02H

**作用**： 读磁盘扇区：

**调用参数**：
- AL = 扇区数(要读取几个扇区)
- CX中的0~5位代表扇区号（从哪个扇区开始读），CX中的6~15位代表柱面号（其中，CL的6~7为柱面数的高两位，CH存低8位）
- DH/DL = 磁头号/驱动器号
- ES:BX = 数据缓冲区地址

**返回参数**：

- 读成功 ⇒ AH = 00H， AL = 读取的扇区数，CF = 0
- 读失败 ⇒ AH = 错误码

参考https://www.cnblogs.com/AmitX-moten/p/4823598.html 0x13号中断。

因此该段代码的意思就是将磁盘的第2-5个扇区拷贝到es:bx 处 即0x90200处。

