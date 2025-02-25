[TOC]



## 一、str2str 用法

### 1、简介

从数据流中输入数据，并将其分割和输出到多个数据流中，输入流可以是串行、tcp 客户端、tcp 服务器、ntrip 客户端或文件。输出流可以是串行、tcp 客户端、tcp 服务器、ntrip 服务器或文件。str2str 是常驻应用程序。要停止它：

* 如果运行在前台，则在控制台中键入 ctr-c；
* 如果运行在后台，则向后台进程发送 SIGINT 信号。

如果输入流和输出流都遵循 #format 输入信息的格式将被转换为输出格式。要指定输出使用 -msg 选项。如果省略选项 -in 或 -out，则输入为 stdin，"... "输出为 stdout、输入使用stdin，输出使用 stdout。如果选项 -in 或 -out 中的流为空，也会使用 stdin 或 stdout





### 2、命令行参数

使用方法：`str2str [-in stream] [-out stream [-out stream...]] [options]`



数据流路径 stream path：

```yaml
serial       : serial://port[:brate[:bsize[:parity[:stopb[:fctr]]]]]
tcp server   : tcpsvr://:port
tcp client   : tcpcli://addr[:port]
ntrip client : ntrip://[user[:passwd]@]addr[:port][/mntpnt]
ntrip server : ntrips://[:passwd@]addr[:port]/mntpnt[:str] (only out)
ntrip caster : ntripc://[user:passwd@][:port]/mntpnt[:srctbl] (only out)
file         : [file://]path[::T][::+start][::xseppd][::S=swap]
```

数据格式 format：

```yaml
rtcm2        : RTCM 2 (only in)
rtcm3        : RTCM 3
nov          : NovAtel OEMV/4/6,OEMStar (only in)
oem3         : NovAtel OEM3 (only in)
ubx          : ublox LEA-4T/5T/6T (only in)
ss2          : NovAtel Superstar II (only in)
hemis        : Hemisphere Eclipse/Crescent (only in)
stq          : SkyTraq S1315F (only in)
javad        : Javad (only in)
nvs          : NVS BINR (only in)
binex        : BINEX (only in)
rt17         : Trimble RT17 (only in)
sbf          : Septentrio SBF (only in)
```

选项 option：

```yaml
-sta sta          station id
-opt opt          receiver dependent options
-s  msec          timeout time (ms) [10000]
-r  msec          reconnect interval (ms) [10000]
-n  msec          nmea request cycle (m) [0]
-f  sec           file swap margin (s) [30]
-c  file          input commands file [no]
-c1 file          output 1 commands file [no]
-c2 file          output 2 commands file [no]
-c3 file          output 3 commands file [no]
-c4 file          output 4 commands file [no]
-p  lat lon hgt   station position (latitude/longitude/height) (deg,m)
-px x y z         station position (x/y/z-ecef) (m)
-a  antinfo       antenna info (separated by ,)
-i  rcvinfo       receiver info (separated by ,)
-o  e n u         antenna offset (e,n,u) (m)
-l  local_dir     ftp/http local directory []
-x  proxy_addr    http/ntrip proxy address [no]
-b  str_no        relay back messages from output str to input str [no]
-t  level         trace level [0]
-fl file          log file [str2str.trace]
-h                print help
```







### 3、配置文件



## 二、GNSS 数据流概念

### 1、Linux 进程线程

#### 1. 多任务处理

多任务处理是指用户可以在同一时间内运行多个应用程序，每个正在执行的应用程序被称为一个任务。Linux就是一个支持多任务的操作系统，比起单任务系统它的功能增强了许多。多任务操作系统使用某种调度策略支持多个任务并发执行。事实上，处理器(单核)在某一时刻只能执行一个任务。每个任务创建时被分配时间片(ms级)，任务执行(占用CPU)时，时间片递减。操作系统会在当前任务的时间片用完时调度执行其他任务。由于任务会频繁地切换执行，因此给用户多个任务同时运行的感觉。

![在这里插入图片描述](https://pic-bed-1316053657.cos.ap-nanjing.myqcloud.com/img/20201208214244384.png)

#### 2. 进程

进程是指一个具有独立功能的程序在某个数据集合上的一次动态执行过程，它是操作系统进行资源分配和调度的基本单元，进程 = 程序 + 内核的PCB。一次任务的运行可以激活多个进程，这些进程相互合作来完成该任务的一个最终目标。进程具有并发性、动态性、交互性和独立性。Linux系统中主要包括以下三种类型的进程：

* **交互式进程**：这类进程经常与用户进行交互，需要等待用户的输入
* **批处理进程**：这类进行不必与用户进行交互，通常在后台运行
* **守护进程**：这类进程一直在后台运行，和任何终端都不关联

Linux操作系统采用虚拟内存管理技术，使得每个进程都有独立的地址空间，该地址空间是大小为4GB的线性虚拟空间。该技术不但更安全（用户不能直接访问物理内存），而且用户程序可以使用比实际物理内存更大的地址空间。

进程不但包括程序的指令和数据，而且包括程序计数器和处理器的所有寄存器以及存储临时数据的进程堆栈。内核将所有进程存放在双向循环链表（进程链表）中，链表的每一项都是 task_struct，称为进程控制块（PCB）的结构。task_struct 结构中最为重要的两个域为：state（进程状态）和 pid（进程标识符）



#### 3. 线程

因为进程拥有独立的堆栈空间和数据段，所以每当启动一个新的进程必须分配给它独立的地址空间，建立众多的数据表来维护它的代码段、堆栈段和数据段，这对于多进程来说十分“奢侈”，系统开销比较大。所以引入线程，线程拥有独立的堆栈空间，但是共享数据段，它们彼此之间使用相同的地址空间，共享大部分数据，比进程更节俭，开销比较小，切换速度也比进程快，效率高，但是正由于进程之间独立的特点，使得进程安全性比较高，也因为进程有独立的地址空间，一个进程崩溃后，在保护模式下不会对其它进程产生影响，而线程只是一个进程中的不同执行路径。

* 线程是**轻量级进程**(light-weight process)，线程的开销比进程小。
* 线程是最小的执行单位，进程是最小的资源分配单位。
* 创建进程的 fork，还是创建线程的 pthread_create，底层实现都是调用同一个内核函数 clone。Linux 内核是不区分进程和线程的, 只在用户层面上进行区分。
  *  如果复制对方的地址空间，那么就产出一个“进程”。
  *  如果共享对方的地址空间，就产生一个“线程”。





#### 4. Pid、Uid、Tid

* **PID**：进程id，一个pid对应一个进程，每次杀死进程，再重新启动程序，系统都会赋予一个新的pid，一般情况下一个应用程序对应一个pid，但一个应用程序也可以有多个pid。
* **PPID**：父进程PID。
* **UID**：用户身份证明(User Identification)的缩写。 UID用户在注册后，系统会自动的给你一个UID的数值。意思就是给这名用户编个号。 
* **TID**：线程id





#### 5. 进程间通信

Linux环境下，进程地址空间相互独立，每个进程各自有不同的用户地址空间。任何一个进程的全局变量在另一个进程中都看不到，所以进程和进程之间不能相互访问，要交换数据必须通过内核，在内核中开辟一块缓冲区，进程1把数据从用户空间拷到内核缓冲区，进程2再从内核缓冲区把数据读走，内核提供的这种机制称为进程间通信（IPC，InterProcess Communication）。 

常用通信方式：**管道** (使用最简单)、**信号** (开销最小)、**共享映射区** (无血缘关系)、**本地套接字** (最稳定)：







#### 6. 信号

信号是进程之间相互传递消息的一种方法，全称为**软中断信号**，也有人称作软中断，从它的命名可以看出，它的实质和使用很像中断。进程A给进程B发送信号，进程B收到信号之前执行自己的代码，**收到信号后，不管执行到程序的什么位置，都要暂停运行，去处理信号，处理完毕后再继续执行**。与硬件中断类似。



#### 7. 锁



#### 8. 信号量







### 2、Linux 网络编程

#### 1. IP 地址

IP地址：指互联网协议地址（Internet Protocol Address)，俗称IP。**IP地址用来给一个网络中的计算机设备做唯一的编号**。IP地址为32（IPV4）或者128位（IPV6），每一个数据包都必须携带目的地址IP和源IP地址，路由器依靠此信息为数据包选择最优路由（路线）。（IP地址就有点像是几栋楼中的楼的号码）

#### 2. 端口号

网络的通信，本质上是两个进程（应用程序）的通信。如果说IP地址可以唯一标识网络中的设备，那么**端口号就可以唯一标识设备中的进程（应用程序）编号**。端口提供了一种访问通道，服务器一般都是通过知名的端口来识别的。例如，对于每个 TCP/IP 的实现来说，FTP 服务的端口号是21，每个 Telnet 服务器的 TCP 端口号都是 23，每个 TFTP (简单文件传送协议)服务器的UDP端口号都是 69。

端口号是用两个字节表示的整数，它的取值范围是 0~65535。其中 0~1023 之间的端口号用于一些知名的网络服务和应用，普通的应用程序需要使用 1024 以上的端口号。如果端口号被另外一个服务或应用所占用，会导致当前程序启动失败。

#### 3. 字节序

字节序：是指多字节数据的存储顺序，在设计计算机系统的时候，有两种处理内存中数据的方法：即**大端格式**、**小端格式**。

* **小端格式(Little-Endian)**：将**低位字节数据**存储在**低地址**。
* **大端格式(Big-Endian)**：将**高位字节数据**存储在**低地址**。

网络字节序的定义：将收到的第一个字节的数据当做高位来看待，这就要求发送端的发送的第一个字节应该是高位。而在发送端发送数据时，发送的第一个字节是该数字在内存中起始地址对应的字节。可见多字节数值在发送前，在内存中数值应该以大端法存放。

所以，网络协议指定了通讯字节序：大端。只有在多字节数据处理时才需要考虑字节序，运行在同一台计算机上的进程相互通信时，一般不用考虑字节序，异构计算机之间进行通讯时，需要将自己的字节序转换为网络字节序。

**C 语言有关字节序转换的函数：**

* 主机字节序转换网络字节序：htons、htonl
* 将网络字节序转换主机字节序：ntohs、ntohl
* 点分十进制IP地址转换为二进制IP地址：inet_aton、inet_pton
* 二进制IP地址转换成点分十进制的IP地址：inet_ntoa、inet_ntop

#### 4. socket 套接字

Socket 的英文原义是“孔”或“插座”。在网络编程中，网络上的两个程序通过一个双向的通信连接实现数据的交换，这个连接的一端称为一个 Socket。Socket 套接字是通信的基石，是支持TCP/IP协议的网络通信的基本操作单元。它是网络通信过程中端点的抽象表示，包含进行网络通信必须的五种信息：连接使用的**协议**，本地主机的 **IP 地址**，本地进程的协议**端口**，远地主机的 **IP 地址**，远地进程的协议**端口**。

Socket 实质上提供了进程通信的端点。进程通信之前，双方首先必须各自创建一个端点，否则是没有办法建立联系并相互通信的。正如打电话之前，双方必须各自拥有一台电话机一样。套接字之间的连接过程可以分为三个步骤：

* **服务器监听：**是服务器端套接字并不定位具体的客户端套接字，而是处于等待连接的状态，实时监控网络状态。
* **客户端请求：**是指由客户端的套接字提出连接请求，要连接的目标是服务器端的套接字。为此，客户端的套接字必须首先描述它要连接的服务器的套接字，指出服务器端套接字的地址和端口号，然后就向服务器端套接字提出连接请求。
* **连接确认：**是指当服务器端套接字监听到或者说接收到客户端套接字的连接请求，它就响应客户端套接字的请求，建立一个新的线程，把服务器端套接字的描述发给客户端，一旦客户端确认了此描述，连接就建立好了。而服务器端套接字继续处于监听状态，继续接收其他客户端套接字的连接请求。

Socket本质是编程接口(API)，对TCP/IP的封装，TCP/IP也要提供可供程序员做网络开发所用的接口，这就是Socket编程接口；HTTP是轿车，提供了封装或者显示数据的具体形式；Socket是发动机，提供了网络通信的能力。常用函数如下：

* `socket()`：**创建** socket、同时指定协议和类型。
* `bind()`：用于**绑定** IP 地址和端口号到 socket。

* `listen()`：**监听**被绑定的端口、设置能处理的最大连接数。它不会主动的要求与某个进程连接，只是一直监听是否有其他客户进程与之连接，然后响应该连接请求，并对它做出处理，一个服务进程可以同时处理多个客户进程的连接。
* `accept()`：**接收连接请求**。accept 函数由 TCP 服务器调用，用于从已完成连接队列对头返回下一个已完成连接，如果已完成连接队列为空，那么进程被投入睡眠。
* `connect()`：**发送连接请求**，用于绑定之后的client端（客户端），与服务器建立连接。
* `send()`、`recv()`：**TCP 收发信息**。
* `sendto()`、`recvfrom()`：**UDP 收发信息**。

#### 5. TCP 协议

**传输控制协议** (Transmission Control Protocol)。TCP协议是面向连接的通信协议，即传输数据之前，在发送端和接收端建立逻辑连接，然后再传输数据，它提供了两台计算机之间可靠无差错的数据传输。流程如下：

![在这里插入图片描述](https://pic-bed-1316053657.cos.ap-nanjing.myqcloud.com/img/57d8cf3042bb4a28808b1ea8bdeac793.png)

#### 6. UDP 协议

**用户数据报协议(**User Datagram Protocol)。UDP协议是一个面向无连接的协议。传输数据时，不需要建立连接，不管对方端服务是否启动，直接将数据、数据源和目的地都封装在数据包中，直接发送。每个数据包的大小限制在64k以内。它是不可靠协议，因为无连接，所以传输速度快，但是容易丢失数据。日常应用中,例如视频会议、QQ聊天等。流程如下：

![在这里插入图片描述](https://pic-bed-1316053657.cos.ap-nanjing.myqcloud.com/img/b026367ec7854f519e3e91429dd70c2f.png)

### 3、串口通信





### 4、RTCM 数据





### 5、NMEA 数据





### 6、SSR 改正

2007 年，IGS-RTPP（IGS Real-Time Pilot Project）项目正式运作，IGS-RTPP 项目可以收集全球数百个实时跟踪站的数据并且生成高精度的钟差与轨道产品；NTRIP 协议作为一种网络数据传输协议，可以实现将信息以 RTCM 的格式通过互联网向全世界播发。基于 NTRIP 协议，2013 年 4 月，IGS 正式开始向全球用户播发高精度的实时产品。 目前可以从 IGS 的各分析中心获取实时精密轨道和钟差产品，我国武汉大学等机构也能实现实时产品的播发。







### 7、NTRIP 协议

NTRIP 协议（Networked Transport of RTCM via Internet Protocol）是一个基于超文本传输议 (HTTP/1.1) 开发的非专有协议, 并且作为 CORS 系统的通用协议之一, 用于网络传输 GNSS 数据流的专业应用层协议，解决了 CORS 参考站和 RTCM 差分数据通讯标准问题 。移动定位用户可通过移动网络、无线或者 TCP/IP 网络，如 GSM, GPRS, EDGE, WiMax 或 UMTS 等, 实时获取 RTCM 差分数据进行改正定位信息。 NTRIP 协议包括以下四个系统软件或硬件组件：

* **NTRIP 客户端**（NtripClient）：登录 NtripCaster 后, 接收差分数据。

* **NTRIP 服务器**（NtripServer）：负责提供 GNSS 差分数据给 NtripCaster。
* **NTRIP 播发器**（NtripCaster）：差分数据中心，负责接收和发送差分数据。
* **NTRIP 源**（NtripSource）：用来产生差分数据并传输给 NTRIP 服务器 NtripServer。

Ntrip Client 是指接收 RTK 数据流的用户站设备，Ntrip Client 使用 Ntrip Caster 分配的 IP 地址通过互联网连接到 NtripCaster。Ntrip Server 这部分用于从 GPS 参考站网络得到 Ntrip Caster 传输的 RTK 数据。在CORS 系统中 Ntrip  服务器（硬件）通常是运行 CORS 系统管理软件的计算机。Ntrip 服务器给产生不同差分数据格式的数据源（Ntrip Source）分配一个节点名（mountpoint），Ntrip  处理中心就将多个节点名列表制成源列表(Source Table)  。Ntrip Client 访问请求 NtripCaster 分配的 IP 地址时就可以收到这张源列表，根据源列表的信息，客户可以自由选择自己需要的数据格式。

NtripSource 和 NtripServer 一般已经集成到一台GPS基准站内，GPS基准站产生差分数据（扮演着NtripSource的角色），然后再通过网络发送给NtripCaster（扮演着NtripServer的角色）

NtripSource 和 NtripServer也可以分开：GPS基准站产生差分数据，然后通过串口发送给一个程序，这个程序再把差分数据发送给NtripCaster。这里GPS基准站扮演着NtripSource的角色，程序扮演着NtripServer的角色。

NtripCaster一般就是一台固定IP地址的服务器，它负责接收、发送差分数据。**给NtripClient发送差分数据时有两种方案**：一是直接转发NtripSource产生的差分数据；二是通过解算多个NtripSource的差分数据，为NtripClient产生一个虚拟的基准站（即VRS）。

NtripClient一般就是GPS流动站。登录NtripCaster后，发送自身的坐标给NtripCaster。NtripCaster选择或产生差分数据，然后发送给NtripClient。这样GPS流动站即可实现高精度的差分定位。



## 三、stream.c













## 四、streamsvr.c

### 1、通用数据流 API

* `stropen()`：先根据选项赋值 stream 结构体，然后根据数据流类型调用对应的函数（openXXX），打开数据流。
* `strclose()`：调用对应的函数（closeXXX），关闭数据流。
* `strsync()`：用于带时间标签的重放文件，调用 syncfile() 根据时间标签同步时间。
* `strlock()`、`strunlock()`：读写数据流之前上锁，之后解锁。
* `strread()`：调用对应的函数（readXXX），读取数据流中高端数据到 buffer。
* `strwrite()`：调用对应的函数（writeXXX），写 buffer 数据到数据流。
* `strstat()`：调用对应的函数（stateXXX），获取数据流状态。
* `strstatx()`：调用对应的函数（statexXXX），获取扩展数据流状态。
* `strsum()`：获取数据流状态概要。
* `strsetopt()`：获取全局数据流选项，包括：非活动超时、重新连接的时间间隔、平均数据速率、接收/发送缓冲区大小、文件交换边缘时间。
* `strsettimeout()`：设置超时时间。
* `strsetdir()`：设置 ftp/http 下载到的本地目录路径。
* `strsetproxy()`：设置 http/ntrip 代理地址。
* `strgettime()`：获取数据流播放文件的当前时间或重放时间。
* `strsendnmea()`：调用 `outnmea_gga()`、`strwrite()`，向数据流发送 NMEA GPGGA 信息。
* `gen_hex()`：生成普通十六进制信息。
* `set_brate()`：生成十六进制信息。
* `set_brate()`：设置比特率。
* `strsendcmd()`：根据接收机品牌，发送接收机指令。





## 五、str2str.c


