### 计算机系统漫游

<img width="1336" height="823" alt="Image" src="https://github.com/user-attachments/assets/1c51207d-bc7a-4acd-ad91-693fc2382555" />

文件是对I/O设备的抽象表示，虚拟内存是对主存和磁盘I/O设备的抽象表示，进程是对处理器、主存和I/O设备的抽象表示；虚拟机是对整个计算机的抽象，包括操作系统、处理器和程序。
- 操作系统提供一种假象，程序看上去是独占地使用处理器、主存和I/O设设备。

内核是操作系统代码常驻主存的部分。

### 链接

#### 7. 4可重定位目标文件

<img width="502" height="592" alt="Image" src="https://github.com/user-attachments/assets/5cbab20f-592c-423d-aeb0-23c16b8909ca" />

ELF 头(ELF header)以一个 16 字节的序列开始，这个序列描述了生成该文件的系统的字的大小和字节顺序。 ELF 头剩下的部分包含帮助链接器语法分析和解释目标文件的信息。其中包括 ELF 头的大小、目标文件的类型（如可重定位、可执行或者共享的）、机器类（如 x86-64）、节头部表(section header table) 的文件偏移，以及节头部表中条目的大小和数最。不同节的位置和大小是由节头部表描述的，其中目标文件中每个节都有一个固定大小的条目(entry) 。



## 参考资料

CSAPP书籍
[网站](https://hansimov.gitbook.io/csapp)



