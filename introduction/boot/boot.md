
- [bootsect.s](#bootsects)
- [setup.s](#setups)

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


# setup.s

下面要设置的是扩展内存的大小
```asm
mov	ah,#0x88
int	0x15
mov	[2],ax
```
调用中断是0x15， 功能号ah=0x88

结果是从0x100000(1M)之后的扩展内存的大小，单位是KB

由于这个时候的ds设置为0x9000, 所以[2] 代表0x90002, 因此扩展内存的大小就存放在0x90002地址处

该值在main函数中会被用到

```c
#define EXT_MEM_K (*(unsigned short *)0x90002)

memory_end = (1<<20) + (EXT_MEM_K<<10);     // 内存大小=1Mb + 扩展内存(k)* 1kb（1024 byte)
```

这里可以计算处内存的总大小。


下面这段则是获取硬盘参数
```asm
	mov	ax,#0x0000
	mov	ds,ax          !ds = 0
	lds	si,[4*0x41]    !这条指令将0x0000:0x0104单元存放的值赋值给si寄存器，将0x0000:0x0106单元存放的值赋给ds寄存器
	mov	ax,#INITSEG
	mov	es,ax
	mov	di,#0x0080     !es:di = 0x9000:0x0090
	mov	cx,#0x10       !循环16次
	rep
	movsb
```    
关于8086的中断向量表可以参考以下文档
https://wenku.baidu.com/view/b865a394c47da26925c52cc58bd63186bceb929f.html