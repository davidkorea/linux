# 从BIOS到bootloader

按下计算机的启动按钮时，你的主板就加上电，这时候你的 CPU 应该开始执行指令了。这个时候没有操作系统，内存也是空的，一穷二白。CPU 该怎么办呢？

# 1. ROM中的BIOS程序
计算机系统也早有计划。在主板上，有一个东西叫ROM（Read Only Memory，只读存储器）。ROM 是主板上的单独一块存储空间，不属于内存的一部分，不在内存里面。ROM是只读的，上面早就固化了一些**初始化的程序**，也就是BIOS（Basic Input and Output System，基本输入输出系统）。BIOS就是一套程序。

1. BIOS 要检查一下系统的硬件是不是都好着呢
2. 提供简单服务，处理简单请求
    - 要建立一个中断向量表和中断服务程序，因为现在你还要用键盘和鼠标，这些都要通过中断进行的。
    - 这个时期也要给客户输出一些结果，在内存空间映射显存的空间，在显示器上显示一些字符。


1M 的内存地址空间，这个空间非常有限，你需要好好利用才行。

在 x86 系统中，将 1M 空间最上面的 0xF0000 到 0xFFFFF 这 64K 映射给 ROM，也就是说，到这部分地址访问的时候，会访问 ROM。当电脑刚加电的时候，会做一些重置的工作，将 CS 设置为 0xFFFF，将 IP 设置为 0x0000，所以第一条指令就会指向 0xFFFF0，正是在 ROM 的范围内。在这里，有一个 JMP 命令会跳到 ROM 中做初始化工作的代码，于是，BIOS 开始进行初始化的工作。

![](http://tvax3.sinaimg.cn/large/006gDTsUgy1g8a8mamrcjj31dw0vwmzs.jpg)

> - 将内存的最上面64k空间 映射到ROM来使用。 并不是说rom是内存的一部分, 而是内存的一部分 用来加载rom中的信息
> - 而bios就烧录在rom中, 也即 内存最上面的64k加载bios来运行
> - 0x 十六进制，每一位可由四位二进制表示
>   - 0xF0000，5位十六进制 = 4*5=20位二进制，所以2^20=1024*1024=1M， 即 五位十六进制可以寻址1M
> - 由于刚上电 处于实模式real pattern所以地址总线20位, 数据总线16位
>   - CS 代码段起始地址 16位 0xFFFF
>   - IP 代码段offset 16位 0x0000
>   - 换算成20位 = 起始地址左移4位 + offset= 0xFFFF0 + 0x0000 = 0xFFFF0


# 2. MBR中的bootloader程序grub2 
- grub2
    - Boot.img
    - core.img
        - diskboot.img
        - lzma_decompress.img
        - kernel.img
    
## 2.1 MBR
BIOS 的界面上，你会看到一个启动盘的选项。启动盘有什么特点呢？ 它一般在第一个扇区，占 512 字节，而且以 0xAA55 结束。这是一个约定，当满足这个条件的时候，就说明这是一个启动盘，在 512 字节以内会启动相关的代码。
 
- 硬盘第一个扇区（sector），也叫做MBR master boot record 主引导记录/扇区，共512字节
- **MBR里面有启动相关的代码**，这些代码是在安装Linux系统时，由Grub2，全称 Grand Unified Bootloader Version 2这个程序写到硬盘第一个扇区的
    - 可以通过 `grub2-mkconfig -o /boot/grub2/grub.cfg` 来配置系统启动的选项
        ```bash        
        menuentry 'CentOS Linux (3.10.0-862.el7.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-862.el7.x86_64-advanced-b1aceb95-6b9e-464a-a589-bed66220ebee' {
          load_video
          set gfxpayload=keep
          insmod gzio
          insmod part_msdos
          insmod ext2
          set root='hd0,msdos1'
          if [ x$feature_platform_search_hint = xy ]; then
            search --no-floppy --fs-uuid --set=root --hint='hd0,msdos1'  b1aceb95-6b9e-464a-a589-bed66220ebee
          else
            search --no-floppy --fs-uuid --set=root b1aceb95-6b9e-464a-a589-bed66220ebee
          fi
          linux16 /boot/vmlinuz-3.10.0-862.el7.x86_64 root=UUID=b1aceb95-6b9e-464a-a589-bed66220ebee ro console=tty0 console=ttyS0,115200 crashkernel=auto net.ifnames=0 biosdevname=0 rhgb quiet 
          initrd16 /boot/initramfs-3.10.0-862.el7.x86_64.img
        }
        ```

## 2.2 grub2 - boot.img
grub2 第一个要安装的就是**boot.img**，它由 boot.S 编译而成，一**共512字节**，正式安装到启动盘的第一个扇区，这个扇区通常称为MBR。 

BIOS找到MBR后，会将boot.img从硬盘加载到内存中的 0x7c00 来运行。

## 2.3 grub2 - core.img（diskboot.img，lzma_decompress.img，kernel.img ）
由于 512 个字节实在有限，boot.img 做不了太多的事情。它能做的最重要的一个事情就是加载 grub2 的另一个镜像 core.img。

core.img 由 lzma_decompress.img、diskboot.img、kernel.img 和一系列的模块组成，功能比较丰富，能做很多事情。

### 2.3.1 core.img - diskboot.img
boot.img先加载的是 core.img的第一个扇区。如果从硬盘启动的话，第一个扇区里面是 diskboot.img，对应的代码是 diskboot.S。

### 2.3.2 core.img - lzma_decompress.img & kernel.img & modules
boot.img将控制权交给diskboot.img后，diskboot.img 的任务就是将 core.img 的其他部分加载进来。
- 先是 lzma_decompress.img，对应代码startup_raw.S，这是一个解压缩程序 
- 再往下是 kernel.img，是压缩过的，需要上面的程序将其解压
    - 它不是Linux 的内核，而是grub的内核
    - 加压kernel之前，先将real pattern的1M内存转换为proctect pattern的4G内存
- 最后是各个模块 module 对应的映像

![](http://tvax3.sinaimg.cn/large/006gDTsUgy1g8a9ylzhywj31x51680vs.jpg)

### 2.3.3 real_to_proc
解压grub2的kernel，数据量就很大了，real pttern的1M内存就不够了，所以grub2的kernel解压之前，lzma_decompress.img调用real_to_prot将实模式转换为保护模式。

- 第一项是启用分段
    - 内存中建立 段描述符表 存储真正的起始地址
    - CPU控制单元的CS和DS段寄存器 转换为 选择子 指向内存中的描述符
- 第二项是启动分页
    - 能够管理的内存变大了，就需要将内存分成相等大小的块

- 打开 Gate A20，也就是第 21 根地址线的控制线。
    - 在实模式8086下面，一共就 20 个地址线，可访问 1M 的地址空间。如果超过了这个限度怎么办呢？当然是绕回来了
    - 在保护模式下，第 21 根要起作用了，于是我们就需要打开 Gate A20
    - 函数 DATA32 call real_to_prot 会打开 Gate A20，也就是第 21 根地址线的控制线

接下来对压缩过的 kernel.img 进行解压缩，然后跳转到 kernel.img 开始运行。


### 2.3.4 kernel.img
- **kernel.img** 对应的代码是startup.S 以及一堆 c 文件，在startup.S中会调用grub kernel 的**主函数 grub_main**
- 主函数里面，**grub_load_config()** 开始解析上面写的那个 **grub.conf** 文件里的配置信息
- 若解析grub.conf正常
    - grub_main 会调用 **grub_command_execute (“normal”, 0, 0)**
    - 最终会调用 **grub_normal_execute()** ，在这个函数里面的 **grub_show_menu()** 会显示出让你选择的那个操作系统的列表
- 从上述列表中选择要启动某个操作系统，就要开始调用 **grub_menu_execute_entry()** ，开始解析并执行你选择的那一项

若选择启动linux
- grub_cmd_linux() 函数会被调用，它会首先读取 Linux 内核镜像头部的一些数据结构，放到内存中的数据结构来，进行检查。
- 如果检查通过，则会读取整个 Linux 内核镜像到内存
- 如果配置文件里面还有 initrd 命令，用于为即将启动的内核传递 init ramdisk 路径。于是 grub_cmd_initrd() 函数会被调用，将 initramfs 加载到内存中来。
- 当这些事情做完之后，grub_command_execute (“boot”, 0, 0) 才开始真正地启动内核。





















