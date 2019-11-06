
任意一个**进程**，默认有一个**主线程**
- 线程是负责执行二进制指令的，它会根据项目执行计划书(代码指令)，一行一行执行下去。
- 进程要比线程管的宽多了，除了执行指令之外，内存、文件系统等等都要它来管

进程并行执行任务
- 项目组是独立的，会议室是独立的，很多事情就不受你控制了，例如一旦有了两个项目组，就会有沟通问题
- 第一，创建进程占用资源太多；第二，进程之间的通信需要数据在不同的内存空间传来传去，无法共享

**进程**的执行是需要项目执行计划书的，那**线程**是一个项目小组，这个小组**也应该**有自己的**项目执行计划书**，也就是一个**函数**。我们将要执行的子任务放在这个函数里面

<img src="http://tvax1.sinaimg.cn/large/006gDTsUgy1g8o5bshi60j31u71mlan3.jpg" alt="e38c28b0972581d009ef16f1ebdee2bd" width="2383" data-width="2383" data-height="2109">

-----

<img src="http://tva4.sinaimg.cn/large/006gDTsUgy1g8o5f7ghqkj31sj1vlqcs.jpg" alt="0ccf37aafa2b287363399e130b2726be" width="2323" data-width="2323" data-height="2433">

-----

<img src="http://tva1.sinaimg.cn/large/006gDTsUgy1g8o5fpbe8kj32v52xz14g.jpg" alt="1d4e17fdb1860f7ca7f23bbe682d93f7" width="3713" data-width="3713" data-height="3815">


-----

<img src="http://tvax1.sinaimg.cn/large/006gDTsUgy1g8o5fur5t6j31s31uxn9u.jpg" alt="02a774d7c0f83bb69fec4662622d6d58" width="2307" data-width="2307" data-height="2409">
