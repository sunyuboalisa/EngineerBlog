---
title: RISC-V
draft: false
tags:
---
 xv6-riscv-book[^1]
# 环境
Ubuntu24
```shell
sudo apt-get install git build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu

#验证安装
qemu-system-riscv64 --version
riscv64-linux-gnu-gcc --version
riscv64-unknown-elf-gcc --version
riscv64-unknown-linux-gnu-gcc --version

```
# 结构

| **模块 (Category)**                       | **源码文件 (File)** | **核心功能翻译与大白话解析 (Description)**                                                       |
| --------------------------------------- | --------------- | ------------------------------------------------------------------------------------ |
| **引导启动**<br><br>  <br><br>_(Boot)_      | `entry.S`       | **最开始的启动汇编指令**。计算机刚开机、按下电源键后执行的第一组机器码，用来初始化 CPU 状态并跳入 C 语言。                          |
|                                         | `main.c`        | **控制其他模块的初始化**。内核的 C 语言入口，负责挨个给内存、进程、中断、磁盘等模块“拉闸通电”。                                 |
|                                         | `start.c`       | **早期的机器模式（M Mode）启动代码**。在 RISC-V 特有的最高特权级下做安全配置，随后把 CPU 降级到 S Mode（内核态）并跳到 `main.c`。 |
| **进程管理**<br><br>  <br><br>_(Processes)_ | `exec.c`        | **`exec()` 系统调用的实现**。负责从磁盘上把可执行文件（比如 `cat`）加载到内存里，并替换掉当前进程的内存空间。                     |
|                                         | `proc.c`        | **进程管理与 CPU 调度**。负责进程的创建（`fork`）、销毁（`exit`）以及 CPU 核心怎么在多个运行的进程之间来回切换（上下文切换）。         |
|                                         | `swtch.S`       | **线程/上下文切换的底层汇编**。负责把当前 CPU 寄存器的状态存起来，并把下一个要运行的进程的寄存器状态加载进去。                         |
|                                         | `sysproc.c`     | **进程相关的系统调用安检口**。处理诸如 `sys_fork`、`sys_exit`、`sys_sbrk`（内存扩容）等和进程状态直接相关的系统调用。         |
| **陷入与中断**<br><br>  <br><br>_(Traps)_    | `kernelvec.S`   | **处理来自内核代码的陷阱/中断**。当 CPU 已经在内核态运行时，如果突然发生硬件中断（比如定时器到了），由这段汇编优先拦截。                    |
|                                         | `trampoline.S`  | **处理来自用户代码的陷阱/中断（跳板）**。用户态（U Mode）和内核态（S Mode）切换时的“安检通道”。`ecall` 冲进来后，首先在这里保存用户寄存器。  |
|                                         | `trap.c`        | **处理和返回陷阱与中断的 C 语言核心**。汇编把现场保存好后，交由这里的 C 代码来判断：这到底是一个系统调用、一个硬件中断，还是一个程序崩溃错误（如缺页异常）？  |
|                                         | `syscall.c`     | **系统调用分发器**。它拿着 `a7` 寄存器里的编号（如 `SYS_write`），去查系统调用函数指针表，把请求派发给对应的处理函数。               |
| **内存管理**<br><br>  <br><br>_(Memory)_    | `vm.c`          | **管理页表与虚拟地址空间**。控制 RISC-V 的硬件 MMU 虚拟内存映射，实现内核和每个进程独立的“内存虚拟世界”。                       |
|                                         | `kalloc.c`      | **物理内存分配器**。内核里大名鼎鼎的物理页管家，以 4096 字节（Page）为单位，负责物理内存的借出（`kalloc`）和回收（`kfree`）。        |
| **设备驱动**<br><br>  <br><br>_(Devices)_   | `console.c`     | **连接用户键盘与屏幕的控制台驱动**。处理输入（把键盘字符存入缓冲区）和输出（调用串口发送）。                                     |
|                                         | `plic.c`        | **RISC-V 中断控制器驱动（PLIC）**。管理主板上所有的外部硬件中断信号（如键盘、磁盘、串口），决定哪个 CPU 核心去处理哪个中断。             |
|                                         | `printf.c`      | **内核专用的格式化输出**。就是内核代码自己用的 `printf`，负责把内核调试日志通过控制台打出来。                                |
|                                         | `uart.c`        | **16550a 串口设备驱动**。 负责直接对物理内存地址 `0x10000000` 进行读写，把字符真正弹射给 QEMU 宿主机。                  |
|                                         | `virtio_disk.c` | **虚拟磁盘驱动**                                                                           |
| **文件系统**<br><br>  <br><br>_(FS)_        | `bio.c`         | **磁盘块缓存层（Buffer Cache）**。为了防止每次读写都去慢吞吞地砸磁盘，在内核里开辟一块内存作为磁盘块的缓存，并用同步锁保证多核安全。           |
|                                         | `file.c`        | **文件描述符（FD）多态抽象支持**。**（你刚才翻到的倒数第二个文件）** 把管道、设备、普通文件统一抽象成 `struct file`，实现“万物皆文件”。    |
|                                         | `fs.c`          | **传统文件系统核心**。管理磁盘上的 Inode、数据块（Data Block）、目录结构以及位图（Bitmap）。                          |
|                                         | `log.c`         | **文件系统日志与崩溃恢复**。Unix 系统的安全卫士。写磁盘前先写日志（Transaction），确保系统遭遇突然断电时磁盘数据绝不损坏。              |
|                                         | `sysfile.c`     | **文件相关的系统调用安检口**。**（你刚才翻到的第三个文件）** 负责接收用户态传来的 `open`、`read`、`write` 并在入口处做严格的安全检查。   |
|                                         | `pipe.c`        | **管道实现**。实现 Unix 标志性的进程间通信 `                                                         |
| **杂项库函数**<br><br>  <br><br>_(Misc)_     | `sleeplock.c`   | **睡眠锁（Sleep Lock）**。一种长睡眠锁。当一个线程拿不到锁时，它会主动躺下睡觉（放弃 CPU），等锁释放了再被唤醒。适用于耗时很长的磁盘 I/O 场景。  |
|                                         | `spinlock.c`    | **自旋锁（Spin lock）**。一种极度狂热的短锁。当拿不到锁时，CPU 会在原地死等（Spin）。适用于多核 CPU 修改内核核心全局变量的极短场景。      |
|                                         | `string.c`      | **C 语言字符串与字节数组库**。内核自己实现的 `memset`、`memmove`、`strlen` 等基础工具函数。                       |
# 相关基础知识
## RISC-V 核心寄存器与指令分类整理表
### 32个通用寄存器
在 RISC-V 中，共有 32 个通用寄存器（在 RV64 中每个寄存器都是 64 位宽）。为了防止程序员和编译器乱用，ABI（应用程序二进制接口）给它们规定了严格的**分工与别名**：

|**寄存器编号**|**ABI 别名**|**硬件/汇编名称**|**用途与职责说明**|
|---|---|---|---|
|**x0**|`zero`|零寄存器|**硬编码永远为 0**。任何写入它的操作都会被丢弃，读出来的永远是 0。|
|**x1**|`ra`|返回地址寄存器|**Return Address**。函数调用时，硬件自动把返回地址存在这里（`jalr` 指令依赖它）。|
|**x2**|`sp`|栈指针寄存器|**Stack Pointer**。永远指向当前线程或内核栈的栈顶。|
|**x3**|`gp`|全局指针寄存器|**Global Pointer**。用于快速寻址全局变量（通常在小内存模型中固定）。|
|**x4**|`tp`|线程指针寄存器|**Thread Pointer**。常用于存放线程本地存储（TLS）或在 xv6 中用来存当前 CPU 核心 ID。|
|**x5 - x7**|`t0 - t2`|临时寄存器 0-2|**Temporaries**。属于“草稿纸”，函数调用时不需要保护，可随意覆盖。|
|**x8**|`s0` / `fp`|保存/帧指针|**Saved / Frame Pointer**。既可当旧栈帧指针用，也属于需要被子函数严格保护的寄存器。|
|**x9**|`s1`|保存寄存器 1|**Saved Register**。子函数如果使用了它，必须在退出前恢复原样。|
|**x10 - x11**|`a0 - a1`|参数/返回值|**Arguments / Return Values**。用于传递前两个函数参数，或存放函数的返回值（`a0`）。|
|**x12 - x17**|`a2 - a7`|参数寄存器 2-7|**Arguments 2-7**。用于传递第 3 到第 8 个函数参数。|
|**x18 - x27**|`s2 - s11`|保存寄存器 2-11|**Saved Registers**。长期保存的变量，子函数修改必须恢复。|
|**x28 - x31**|`t3 - t6`|临时寄存器 3-6|**Temporaries 3-6**。额外的临时草稿纸寄存器。|

### 核心控制与状态寄存器
CSR 按特权级分类。以下是操作系统内核开发中最核心、最常碰到的 CSR：
#### 1. 机器态（Machine-mode）核心 CSR

|**寄存器名称**|**全称 / 含义**|**核心作用**|
|---|---|---|
|`mstatus`|Machine Status|全局控制面板，管理中断使能、上一个特权级（MPP）等。|
|`mstatush`|Machine Status High|32位系统下扩展的高位状态寄存器（64位系统通常合一）。|
|`misa`|Machine ISA and Extensions|查询芯片支持的指令集扩展（如 RV64GC 中的 G、C 等）。|
|`mie` / `mip`|Machine Interrupt Enable/Pending|机器态的中断使能位与中断挂起（触发）状态位。|
|`mtvec`|Machine Trap Vector|机器态异常/中断发生时的跳转入口地址。|
|`mscratch`|Machine Scratch|机器态的临时中转寄存器，常用于保存内核栈指针。|
|`mepc`|Machine Exception Program Counter|记录机器态下被异常打断的那条指令的地址。|
|`mcause`|Machine Cause|记录机器态下触发异常或中断的具体原因编号。|
|`mtval`|Machine Trap Value|记录触发异常时的附加信息（如非法的内存访问地址）。|
|`mhartid`|Machine Hardware Thread ID|**硬件线程（CPU核心）编号**（0, 1, 2...）。|
#### 2. 监督态（Supervisor-mode，操作系统内核态）核心 CSR

|**寄存器名称**|**全称 / 含义**|**核心作用**|
|---|---|---|
|`sstatus`|Supervisor Status|S 模式下的状态寄存器（`mstatus` 的受限子集）。|
|`sie` / `sip`|Supervisor Interrupt Enable/Pending|S 模式的中断使能与挂起状态寄存器。|
|`stvec`|Supervisor Trap Vector|**S 模式异常/中断入口地址**（如 xv6 的 `kernelvec`）。|
|`sscratch`|Supervisor Scratch|S 模式下的临时中转寄存器（常用于用户态与内核态切换时存寄存器）。|
|`sepc`|Supervisor Exception Program Counter|**记录 S 模式下被打断指令的地址**。|
|`scause`|Supervisor Cause|**记录 S 模式下中断/异常的原因**。|
|`stval`|Supervisor Trap Value|S 模式下异常的附加值（如出错的虚拟地址）。|
|`satp`|Supervisor Address Translation and Protection|**页表基地址与分页控制寄存器**（开启虚拟内存的开关）。|

## 三大特权级
RISC-V 架构定义了多个特权级别，操作系统必须利用它们来实现权限隔离：

- **U 模式（User, 32位编码 00）**：用户态。运行普通的应用程序（如 xv6 的 shell、cat 等），权限最低，不能直接访问硬件或修改关键寄存器。
    
- **S 模式（Supervisor, 32位编码 01）**：监督态/内核态。操作系统内核运行在这里，负责进程调度、虚拟内存管理和设备驱动。
    
- **M 模式（Machine, 32位编码 11）**：机器态。最高权限，直接掌控整颗芯片。通常用于最底层的固件（如 OpenSBI）或简单的引导桥接（如我们的 `start.c`）


## RISC-V 核心指令集

### 1. 基础与常用扩展模块
- **I (Integer)**：整数核心指令集（算术、逻辑、跳转、访存 `lw`/`sw`/`ld`/`sd`）。
    
- **M (Multiply/Divide)**：乘除法扩展（如硬件执行乘法 `mul`、除法 `div`）。
    
- **A (Atomic)**：原子操作扩展（用于多核并发锁、互斥量，如 `lr.d`、`sc.d`）。
    
- **F / D (Float / Double)**：单精度与双精度浮点数运算扩展。
    
- **C (Compressed)**：**压缩指令扩展**。允许将常用的 32 位指令压缩为 16 位，极大地节省代码体积（xv6 和大多数嵌入式系统必开）。
### 2. 特权与系统控制指令
| **指令**            | **类型** | **作用**                                         |
| ----------------- | ------ | ---------------------------------------------- |
| `csrr`            | CSR 操作 | 从指定的 CSR 读出数据到通用寄存器。                           |
| `csrw`            | CSR 操作 | 将通用寄存器的数据写入指定的 CSR。                            |
| `csrrw` / `csrrs` | CSR 操作 | 读写/置位 CSR 的组合原子指令。                             |
| `mret`            | 流程控制   | 从 **M 模式**异常/中断中返回，并根据 `mstatus` 切换特权级。        |
| `sret`            | 流程控制   | 从 **S 模式**异常/中断中返回，并恢复到用户态或前一个 S 模式。           |
| `ecall`           | 异常触发   | 环境调用（Environment Call）。用户态向内核发起**系统调用**的法定途径。  |
| `ebreak`          | 异常触发   | 断点指令。触发断言或通知 GDB 调试器暂停。                        |
| `wfi`             | 电源管理   | Wait For Interrupt。让 CPU 进入低功耗休眠，直到下一个硬件中断唤醒它。 |
# 引导启动
1. 定义`entry.S`
2. 配置设备地址 比如 UART，以及一些向屏幕输出信息的宏 （memlayout.h）
3. 实现c语言的打印函数
4. 定义入口函数 （main.c）
5. 编写 Makefile，将汇编源文件和c语言源文件编译，配置qemu启动

---
# 内存管理
## 寻址
## 页表
## 物理内存管理
## 虚拟内存管理

# 进程管理
# 文件系统
# 设备驱动


[^1]: https://pdos.csail.mit.edu/6.1810/2025/xv6/book-riscv-rev5.pdf
