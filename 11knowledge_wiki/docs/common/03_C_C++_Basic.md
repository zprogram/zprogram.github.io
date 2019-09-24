
#### 1.1.1、C++基础

- 整数编码与存储
- 二进制、八进制、十六进制与转换
- 变量的类型、作用域、存储空间与生命周期
- 字符串的定义以及常见操作
- 函数的传参：传值、传指针、传引用
- 指针，指针定义，*与&，指针加减运算，指针与数组关系，常量指针与指针常量，数组指针与指针数组，函数指针与指针函数，二级指针
- 内存布局，分配与泄漏
- 结构体，自然对齐与sizeof计算大小，结构体赋值（浅拷贝，深拷贝）
- 位运算与应用
- 数据打印格式
- printf引起的格式化串漏洞

##### 1）数据结构基础与算法

- 链表常规操作：创建，遍历，查找，插入，修改，删除
- 队列：先进先出，出队，入队，队空判断，链队与数组队列实现
- 栈：后进先出，堆栈，入栈，栈空判断，链栈和数组栈实现
- 树：二叉树定义，遍历（先序，中序，后序），二叉排序树，平衡二叉树
- 排序：插入（希尔排序），选择（堆排序），交换排序（冒泡排序，快速排序）思想，实现与复杂度
- 查找：折半查找，hash查找，平衡二叉树查找

##### 2）产出：基础项目

- https://leetcode-cn.com/problemset/all/
- C语言通讯录
- 课程管理系统

#### 1.1.2、Windows API编程

[](https://github.com/zprogram/CodeRecord/tree/master/03Win_Program)

##### **1）Windows API**

- 数据类型
- 句柄
- 文件系统函数
- 特殊文件

##### **2）Windows 注册表**

- 注册表根键
- Regedit
- 自启动
- 常用注册表函数

#####  3）网络通信编程

- 网络通信模型基础知识

- - TCP

  Server：

```
  WSAStartup()
  socket()
  bind()
  linsten()
  accept()
  send/recv()
  closesocket()
  WSACleanup()
```

  Client:

```
  WSAStartup()
  socket()
  connect()
  recv/send()
  closesocket()
  WSACleanup()
```

- - UDP

  客户端A

```
  socket()
  bind()
  send()
  recv()
  close()
```

  客户端B

```
  bind()
  recv()
  send()
  close()
```

- 网络模型

- - WSAAsyncSelect模型

  创建窗口（CreateWindows）/对话框然后为该窗口提供一个窗口回调函数(WinProc)/对话框函数。

  通过调用WSAsyncSelect函数自动将套接字设置为非阻塞模式，并注册一个或多个感兴趣的网络事件。

- - WSAEventSelect模型

  WSAEventSelect模型是以事件的形式通知应用程序。

  1）创建事件对象，注册网络事件 WSACreateSelect()/WSAEventSelect()

  2）等待网络事件发生 WSAwaitForMultpleEvents()

  3）获取网络事件 WSAWaitForMultipleEvents()

  4）手动设置信号量和释放资源 WSAResetEvent()

- - 完成端口模型

  利用内核对象的调度，使用少量的几个线程来处理和客户端的所有通信，从而消除线程上下文切换问题。当有事件产生时CPU能保证有资源可用，然后将这些事件加入到一个公共消息队列中去，当前哪一个线程空闲就去处理公共消息队列里的事件，如果没有事件了，线程就空闲下来。

  1）初始化套接字组件 WSASocket()

  2）绑定和监听 bind()

  3）创建完成端口 CreateIoCompletionPort()

  4）创建服务线程 GetSystemInfo()

  5）连接客户端 accept()

  6）套接字与完成端口关联  CreateIoCompletionPort()

  7）将套接字与完成端口关联起来以后，应用程序调用发送数据/接收数据函数完成重叠IO操作

```
  WSASend()/WSASendTo()
  WSARecv()/WSARecvFrom()
```

  8）等待重叠I/O操作结果 GetQueuedCompletionStatus()

  9）投递完成通知 PostQueuedCompletionStatus()


##### **4）多线程**

- 原子

  某一个线程对于某一个资源做操作的时候能够保证没有其他的线程能够对此资源进行访问。

```
  Interlockedxxxxx
```

  - 缺点：只能解决某个变量的问题，只能使一个整型数据做简单算数运算的时候是原子的。

- 临界区

  临界区是使用EnterCriticalSection与LeaveCriticalSection形成一个保护区来保护代码，这一对函数保证，多个保护区的代码，同一时刻只能有一个保护区的代码在执行。

```
  EnterCriticalSection()
  LeaveCriticalSection()
```

  - 缺点：临界区是在一个进程内有效的，无法在多线程的环境下进行同步。

- 互斥体

  一个线程进入临界区，结果因为线程由于某些原因崩溃了临界区无法被释放，那么其他线程也无法进入临界区，全部被卡住。互斥体可以解决这些问题。

  1、互斥体有两个状态，激发态和非激发态
  2、有一个概念，叫做线程拥有权与临界区类似。
  3、等待函数等待互斥体的副作用，将互斥体的拥有者设置为本线程，将互斥体的状态设置为非激发态。

```
  CreateMutex()
  OpenMutex()
  WaitForSingleObject()
```

- 信号量

  信号量是一种同步机制，对于一个信号量来说，它可以被加上多把锁，当所有的锁孔都被锁上，才不允许其他线程访问被信号量锁住的区域。

```
  CreateSemaphore()
  OpenSemaphore()
  ReleaseSemaphore()
  CloseHandle()
```

- 事件

  可以设置等待函数对于此事件对象有没有副作用，也可以手动设置事件对象为激发态还是非激发态
  
```
  CreateEvent()
  OpenEvent()
  SetEvent()
  ResetEvent()
  PulseEvent()
  CloseHandle()
```

- 小结:

```
  原子操作：简单的同步机制，只能对4个字节的数据进行算数运算
  
  临界区：对一段代码实现保护操作，只能在一个进程中的不同线程使用。无法检测由于线程崩溃造成的临界区无法释放的问题。
  
  互斥体：是一个内核对象，可以在不同进程的线程中实现对于一段代码的保护。能够检测由于线程崩溃造成的互斥体释放问题。只能被拥有者线程释放，故而多线程间的不同回调函数的同步使用互斥体可能会造成问题。
  
  信号量：是一个内核对象，没有拥有者的概念，可以控制多个线程同时访问被保护的代码，并且给线程数量设置一个上限。
  
  事件：是一个内核对象，没有拥有者的概念，可以封装自己的同步机制。
```

#####  5）API编程

- 进程操作

**遍历进程**

```
- 方法1：CreateToolhelp32Snapshot()、Process32First()和Process32Next()
- 方法2：EnumProcesses()、EnumProcessModules()、GetModuleBaseName()
- 方法3：wtsapi32.dll的WTSOpenServer()、WTSEnumerateProcess()
- 方法4：Native Api的ZwQuerySystemInformation

```
**结束进程**
```
- 针对窗口发送消息：FindWindow、SendMessageCallback(hwnd,WM_CLOSE,0,0,NULL,0);
- 针对窗口模拟鼠标：FindWindow、keybd_event
- 常用API：TerminateProcess、ZwTerminateProcess、NtTerminateProcess、WinStationTerminateProcess
- VBS脚本的wmi对象：mo.InvokeMethod(observer, "Terminate", null);
- 作业对象：CreateJobObject()、AssignProcessToJobObject()、TerminateJobObject()
- 远程攻击线程：用SetWindowsHookEx广播消息在注入的动态库的DLL_PROCESS_ATTACH事件中判断被注入的进程名，调用ExitProcess(0)/TerminateProcess(GetCurrentProcess(), 0)/ PostQuitMessage(0)
结束自身进程。
- ThreadContext patch法：修改目标进程ThreadContext的EIP指向目标程序的kernel32.dll的ExitProcess地址，OpenThread()、SuspendThread()、GetThreadContext()、SetThreadContext()、ResumeThread()
- 句柄攻击法：FindWindow()、GetWindowThreadProcessId()、OpenProcess()、DuplicateHandle()、TerminateProcess()、CloseHandle()
- 内存攻击法：搜出NtUnmapViewOfSection(更底层的MiUnmapViewOfSection)等等,卸掉目标进程的内存
- 调试器攻击法：DebugActiveProcess-->DebugSetProcessKillOnExit（不用这个直接退出也可以）、
              ntsd -c q -p pid 
- ring0线程进程攻击法：NtQuerySystemInformation、PsTerminateSystemThread、PspExitThread、PspTerminateProcess
- 伪关机法：先提升权限得到19号关机特权，然后hook关机消息(hook NtShutdownSystem)，里面过滤
  掉除了目标进程以外的所有进程的消息。

```


- **注册表启动编程**

```
- RegCreateKeyEx：用来创建注册表键，如果该键已经存在，则打开它（注册表键不区分大小写）
- RegSetValueEx：有名称值的数据和类型时设置指定值的数据和类型。
- RegSetCloseKey：关闭注册表句柄。
```

- **服务木马编程**

```
- OpenSCManager：建立了一个到服务控制管理器的连接。
- CreateService：创建一个服务对象，并将其添加到指定的服务控制管理器数据库的函数。
- OpenService：打开一个已经存在的服务。
```

- **注入编程**

```
- 远程线程：CreateRemoteThread()
- 利用注册表注入：HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion \Windows\AppInit_DLLs
- 设置钩子：SetWindowsHookEx()
- DLL加载顺序挟持：HKML\System\CurrentSet\SessionManger\SafeDll\SafeDllSearchMode
```
- **下载者**

```
- URLDownloadToFile：指从指定URL地址读取内容并将读取到的内容保存到特定的文件里
- WinExec：运行指定的程序
- CreateProcess：运行指定的程序
```

- **僵尸网络**

- **脚本类**

  VBA：

  - https://github.com/zprogram/CodeRecord/blob/master/1_3_%E6%A0%B7%E6%9C%AC%E5%88%86%E6%9E%90/00ScritpVirus/01%204.1%E3%80%81Office%20Macor%E5%9F%BA%E7%A1%80.md
  - https://github.com/zprogram/CodeRecord/tree/master/1_3_%E6%A0%B7%E6%9C%AC%E5%88%86%E6%9E%90/00ScritpVirus/00-Office_Macor_Basic

  Powershell：

  - https://github.com/zprogram/CodeRecord/blob/master/1_3_%E6%A0%B7%E6%9C%AC%E5%88%86%E6%9E%90/00ScritpVirus/02%204.2%E3%80%81PowerShell%E5%9F%BA%E7%A1%80.md
  - https://github.com/zprogram/CodeRecord/tree/master/1_3_%E6%A0%B7%E6%9C%AC%E5%88%86%E6%9E%90/00ScritpVirus/01-PowerShell_Basic


#### 1.1.3、界面编程

#####  SDK编程

#####  MFC编程

#### 1.1.6、RootKit基础

- 安装内核调试

  **安装环境**

```
  VirtualKD+VMware+WinDbg
```

  **WinDbg**

```
  d  读取程序数据或堆栈等内存位置
  
  bp 设置断点
  
  lm 列举出加载到进程空间的所有模块，包括用户模式下的可执行模块
  
  x  允许利用通配符来搜索函数或符号
  
  dt 查看一个数据结构的类型信息
```

- 内核编程基础

**钩子技术**

- SSDT钩子

```
  系统服务调度表（System Services Descriptor Table，SSDT）可以基于系统调用编号索引以定位相应函数的内存地址。
  
  找到SSDT地址
  导入需要勾住的内核函数
  使SSDT所在的内存页可写
  替换SSDT中指定内核函数为我们自定义的函数
  如果需要摘除钩子，将被勾住的函数还原后将SSDT所在的内存设为初始状态。
```

- SYSENETR钩子

```
编写跳板函数
获取需要安装钩子的函数地址
修改内存属性后安装钩子，将函数执行流程转移到跳板函数（将安装钩子的函数头5个字节改成jmp 模块名）
如果有摘除钩子的需要，则摘除完毕后还原内存属性
```

- 内联钩子
- IRP钩子
- IDT钩子
- IOAPIC钩子
- LADDR钩子

**Anti Rootkit**

- 检查SSDT钩子与IDT钩子
- 检查IAT
- 检查IRP钩子
- 检查内联钩子

#### 1.1.7、产出：安全编程项目

- Windows安全卫士
- 远程控制

### 2.1、Linux编程基础

#### 2.1.1、LinuxC程序设计

- Linux命令
- VIM命令
- shell
- 进程间通信
- ELF文件格式
- Pcap文件格式
- Linux操作系统内核

#### 2.1.3、产出：取证分析项目

- 命令行自动化脚本
- 辅助分析平台