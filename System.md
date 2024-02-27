#### 计算机是如何启动的？

  参考：[https://www.jb51.net/article/208552.htm](https://www.jb51.net/article/208552.htm)

  计算机的启动可以分为两个阶段：引导阶段和加载内核阶段。

  - 先说说引导阶段，在按下开机键的时候，内存中什么数据也没有，因此需要借助某种方式，将操作系统加载到内存中，而完成这项任务的就是BIOS。BIOS是主板芯片上的一个程序，计算机通电后，第一件事情就是读取BIOS。BIOS首先会进行硬件检测，检测硬件是否满足计算机运行的基本条件，如果硬件出现问题，主板会发出不同的蜂鸣声，然后停止启动，如果没有问题，则会显示CPU内存硬盘等信息。
  - 接着，BIOS会读取存储设备的第一个扇区，从中读取一个叫“主引导记录”也就是MBR信息，MBR的主要作用是告诉计算机到哪个硬盘位置上去找操作系统。然后运行“管理启动器”boot loader，由用户选择启动哪个操作系统。
  - 用户选择操作系统后，就到加载内核阶段，操作系统的内核被载入内存中。以Linux为例，先载入/boot目录下的kernel，kernel加载完成后，运行第一个程序/sbin/init。init进程是Linux启动后的第一个进程，pid为1，其他的进程都是它的后代。init进程会加载系统的各个模块，比如窗口程序和网络程序等，直到界面显示登录信息，至此，整个系统启动完毕。


#### 说说Android的启动过程

  参考：[https://www.jb51.net/article/208552.htm](https://www.jb51.net/article/208552.htm)

  1. Android是基于Linux系统的。但是 它没有BIOS程序，取而代之的是BootLoader。BootLoader类似于BIOS，在系统加载前，用于初始化硬件设备。而Android中也没有硬盘，只有ROM，类似于硬盘存放操作系统，用户程序等。当按下电源键后，首先加载BootLoader，BootLoader去初始化硬件设备，然后读取ROM找到系统并将内核加载进RAM中。当内核启动后会初始化各种软硬件环境，加载驱动程序，挂载跟文件系统。最后阶段会启动执行第一个进程init进程。
  2. init进程是用户的第一个进程，pid为1。init进程会创建和挂载启动所需要的文件目录，对属性服务进行初始化，最后会解析init.rc文件。
  3. init.rc文件是Android系统的重要配置文件，主要功能是定义了系统启动时需要执行的一系列动作、设置环境变量和属性和执行特定的service。通过这个脚本启动了几个重要的服务：
      - service_manager：ServiceManager 是 Binder IPC 通信过程中的守护进程，本身也是一个 Binder 服务。ServiceManager 进程主要是启动 Binder，提供服务的查询和注册。
      - surface_flinger：SurfaceFlinger 负责图像绘制，合成所有 Surface 并渲染到显示设备。
      - media_server：MediaServer 进程主要是启动 AudioFlinger 音频服务，CameraService 相机服务。负责处理音频解析播放，相机相关的处理。
      - Zygote：Zygote 进程也叫孵化进程，它主要作用是启动system-server和作为守护进程监听处理“孵化新进程”的请求。
  4. Zygote启动了system-server进程，system-server会启动系统中的各种服务，像AMS，PMS，WMS，等等。
  5. 在启动的AMS服务中，又会去启动Launcher进程，到这里整个Android启动便完成。


#### 系统服务何时启动？如何启动？

  在SystemServer进程启动时，会分批启动所有系统服务。通过SystemServiceManager#startService来启动一个服务，服务启动后需要注册到ServiceManager中，其他进程访问ServiceManager来获取服务的代理对象。
