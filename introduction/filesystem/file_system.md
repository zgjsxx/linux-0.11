

- [file system](#file-system)
  - [disk](#disk)

# file system

## disk
![img](../introduction/filesystem/img/disk.jpg)

由上，可用（柱面号，盘面号，扇区号）来定位任意一个“磁盘块”。


老式的Linux设备号的命名规则

设备号 = 主设备号 * 256 + 次设备号

或者说
```
dev_no = (major << 8) + minor
```
这里的主设备号是事先定义好的（1-内存，2-磁盘，3-硬盘，4-ttyx，5-tty，6-并行口，7-非命名管道）。譬如对于硬盘，主设备号为3，因此3*256+0=0x300即为系统中第一个硬盘的设备号。更多的例子如下表：

|设备号|设备文件|对应的设备|
|--|--|---|
|0x300|/dev/hd0|系统中的第一个硬盘|
|0x301|/dev/hd1|系统中的第一个硬盘的第一个分区|
|0x302|/dev/hd2|系统中的第一个硬盘的第二个分区|
|0x303|/dev/hd3|系统中的第一个硬盘的第三个分区|
|0x304|/dev/hd4|系统中的第一个硬盘的第四个分区|
|0x305|/dev/hd5|系统中的第二个硬盘|
|0x306|/dev/hd6|系统中的第二个硬盘的第一个分区|
|0x307|/dev/hd7|系统中的第二个硬盘的第二个分区|
|0x308|/dev/hd8|系统中的第二个硬盘的第三个分区|
|0x309|/dev/hd9|系统中的第二个硬盘的第四个分区|

inode
https://blog.csdn.net/sunshine77_/article/details/120764986
https://github.com/sunym1993/flash-linux0.11-talk
https://koktlzz.github.io/posts/understand-linux-boot-process-by-installing-coreos/
https://blog.csdn.net/longintchar/article/details/106912096