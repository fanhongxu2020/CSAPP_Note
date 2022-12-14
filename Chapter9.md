# 虚拟内存
## 9.1 物理和虚拟寻址
主存是由M个以字节为单位的元素组成的数组，每个字节都有一个唯一的*物理地址（Physical Address）*。  
全部的地址可以表示为：*{0, 1, 2 ... M-1}*，通过物理地址寻找数据的方式叫做*物理寻址*。  
虚拟寻址是CPU通过地址翻译的方式将虚拟地址翻译成物理地址，并最终取得数据的方式。
## 9.2 地址空间
## 9.3 虚拟内存的作用是缓存
概念上，保存在磁盘上的N个连续的以字节为单位的元素组成了一个数组，这个数组被称之为虚拟内存。  
每个字节都以一个唯一的虚拟地址作为数组的索引。磁盘上的数组数据被缓存到主存中。  

虚拟内存被分割为多个固定大小的*虚拟页（Virtual Page）*，每个虚拟页的大小是P=2^p字节。  
同样的，物理内存被分割为*物理页（Physical Page）*，大小也是P字节。
任意时刻，虚拟页的集合都分为三个不相交的子集：  
- **未分配的**：还未创建的页，因为没有任何数据和它们相关联，所以不占用磁盘空间。
- **已缓存的**：已缓存在物理内存中的已分配页。
- **未缓存的**：未缓存在物理内存中的已分配页。
### 9.3.2 页表
页表是*页表条目（Page Table Entry，PTE）* 的数组。  
每个页（虚拟页或物理页）都有一个PTE与之对应，为了方便，我们假设PTE由一个标记位和地址位组成。  
根据PTE的内容分为三种情况：
1. 标记位为1，则地址位指向物理页的起始地址。
2. 标记位为0，并且地址位为空，则表示虚拟页还没有被分配。
3. 标记位为0，地址位不为空，则地址位指向磁盘中的虚拟页的起始地址。
### 9.3.3 页命中
通过虚拟地址定位到PTE，并且PTE标记位有效，通过地址位找到物理地址的过程称为页命中。
### 9.3.4 缺页
在虚拟内存术语中，DRAM缓存不命中被称为缺页。  
假设CPU引用了VP3的数据，但VP3没有在DRAM缓存，以下描述说明了该过程。  

    地址翻译硬件从主存中读取PTE3，推断出VP3并没有被缓存到DRAM，从而触发了缺页异常。  
    在内核中，缺页异常会触发异常回调，回调从DRAM中找出一个牺牲页用于保存VP3。
    牺牲页之前保存的是VP4的数据，假如VP4的数据被修改，那么替换VP3时会把VP4的数据写回硬盘。
    内核修改页表中的条目VP4不再缓存与DRAM。
    下一步，把VP3从硬盘复制到DRAM的PP3，更新PTE3后返回。当回调返回后，重启导致缺页的指令，
    但这时，VP3已经缓存在DRAM了，直接触发页命中的流程。

## 9.4 虚拟内存作为内存管理的工具
操作系统为每个进程都提供了独立的页表，因而每个进程也就有了独立的虚拟地址空间。  
注意：多个虚拟页面可以映射到同一个物理页面。  
虚拟内存简化了**链接**、**加载**、**代码共享**、**数据共享**和**内存分配**。

## 9.5 虚拟内存作为内存保护的工具
通过在PTE上添加一些额外的许可位可以用来控制每个虚拟页面的访问权限。  
运行在内核模式下的进程可以访问任何页面。  
SUP位表示进程必须运行在内核模式（超级用户）下才可以访问该页面。  
READ和WRITE分别控制读访问和写访问。  
如果进程违反了这些规则，会触发一般保护故障，将控制传递给内核异常处理程序，linux shell一般将这种异常报为**段错误（segmentation fault）**。

## 9.6 地址翻译
从形式上来说，地址翻译是指从虚拟地址空间到物理地址空间的一个映射。  
CPU的一个控制寄存器：**页表基址寄存器（Page Table Base Register）**，这个寄存器指向了当前页表。  
