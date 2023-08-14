# Memory Management

- CPU可以直接access的storage只有memory和registers
- Processes都必須放在memory才能被executed
- 控制管理多個程式各別需要多少memory

## 程式執行流程

![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled.png)

### compile time:

- 程式compile將source code轉成executable code

### load time:

- 將程式放進memory的過程
- linkage editor 包含static linking
- load time包含static loading

### execution time(run time):

- 程式正在run
- 包含dynamic loading, dynamic linking

## Address Binding

### Address Binding - Compile Time

![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%201.png)

- 在compile time就先決定了變數的記憶體位置, 也稱absolute code
- 記憶體位置如果改變就要重新compile

### Address Binding - Load Time

![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%202.png)

- load time開始時先不決定變數記憶體位置, 在load time結束前再找有空位的記憶體位置, 也稱relocatable code
- run time時才想換位置的話, 仍然需要relocate, 但至少不用再compile

### Address Binding - Execution Time

![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%203.png)

- 到run time的變數記憶體位置仍然是virtual的
- 也稱為logical-address code 或 virtual-address code
- CPU將記憶體位置(例如0x18)送到Memory的中間, 會透過一個特別的hardware叫做MMU去做記憶體位址的轉換(變成0x2018)
- 目前絕大多數OS都是用這個方法

### Memory-Management Unit (MMU)

![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%204.png)

- mapping virtual address to physical address

### Logical vs Physical Address

- Logical address - CPU產生的, a.k.a. virtual address
- Physical address - seen by the memory module
- compile-time & load-time address biding → logical addr = physical addr
- Execution-time address binding → logical addr ≠ physical addr
- The user program deals with logical addr; it never sees the real physical addr

### Dynamic Loading

- A routine(function call) is loaded into memory when it is called
- 有較好記憶體使用, 沒用到的routine就不會loaded

![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%205.png)

### Dynamic Linking

- Linking postponed until execution time
- 同樣的code在memory只有一份, 其他function要這段code的話再link到這個code, 而不是每個function都各自有一份同樣的code
- 每個routine上都有一個stub, stub是一小段code, 功能是去問OS是否memory有同樣的code, 若沒有就load這個routine進memory
- Dynamically linked library(DLL) 只有在程式呼叫到DLL裡面的function時, 才將function載入記憶體
- 缺點: 因為runtime才要透過system call去找function, 所以效能會慢一點

## Swapping

- 有些情況例如memory如果不夠用時, 會將一些process swap out到disk的某個特定空間(backing store), 之後再swap in到memory
- backing store: disk的特定區域, 在灌系統時我們會需要決定backing store要多大空間, 這個區域獨立於file system, 由MMU管理
- 要和execution time的address binding共用效果才會好
- Process必須idle時才可以swap

![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%206.png)

### Why Swap a process

- 記憶體空間不夠
- lower-priority process可能會等很久才會跑, 所以就先swap到disk

## Contiguous Memory Allocation (連續記憶體分配)

### Fixed-partition allocation

每個process都load固定大小的partition

### Variable-size partition

有些process不需要那麼大的partition, 會有些洞(hole)可以給其它process塞進去

![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%207.png)

塞洞的方法有以下幾種

- First-fit: 找到的第一個塞的下的洞就塞進去, 最多人用的方法
- Best-fit: 找到最小的剛好可以塞的洞再塞進去
- Worst-fit: 找最大的洞來塞

## Fragmentation (碎片化)

### External fragmentation

- 記憶體在各處零碎的使用, 導致中間有很多洞
- 只會出現在variable-size allocation

### Internal fragmentation

- 在partition內沒有被使用的記憶體空間
- 只會出現在fixed-partition allocation

### Solution for External fragmentation: compaction(壓實)

- 將被使用的記憶體集中在一個block
- 只能夠用在address binding - execution time

![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%208.png)

![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%209.png)

## Paging (Non-Contiguous Memory Allocation)

- fixed- partition allocation中使用
- frames: divide **physical** memory到fixed-sized block
- pages: divide **logical** address space, 每個page block都同樣size
- 若program有n pages, 就要找到n個free frames去load這個program
- 需持續追蹤 free frames
- 設置page table去translate logical to physical addresses, page table就是logical和physical address的mapping關係
    
    ![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%2010.png)
    
- 每個process都有自己的page table, OS會為每個process maintain一個page table

### Logical address可分為2個part

- Page number(p): page table的index, 可以找到對應的base address
- Page offset(d): 在該base address後面第幾個byte
- Physical addr = page base addr + page offset
    
    ![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%2011.png)
    
- page size都是2的次方數, 這是硬體的限制
- 32位元的電腦只有32 bits logical address, 所以program memory的最大值= $2^{32}$=4GB
- page size設4KB or 8KB 是commonly used, 因為設越大表示空間浪費越多
- 查目前電腦page size(單位是byte):

```bash
getconf PAGESIZE
```

## Implementation of Page Table

- Page table存在memory
- Page-table base register(PTBR): page table的physical memory address
- PTBR的值存在PCB (Process Control Block)
- 在Context-switch時改變PTBR的值
- 但是透過PTBR去讀memory, 會導致需要2次memory read, 一次找page table, 第二次才找到正確記憶體位址
- 為了解決2-access問題, 透過**Translation Look-aside Buffers(TLB)** (HW) 解決

### **Translation Look-aside Buffers(TLB)**

- TLB使用Associative memory, 一般memory查找index需要全部遍歷一遍, 所以是O(n), 但他可以同時查全部的index, 所以是O(1)
    
    ![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%2012.png)
    
- TLB就是Page table的cache
- 時間比較: TLB search: 20 ns; memory access: 100 ns
- 結合原本的page, 結構會變這樣:
    
    ![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%2013.png)
    
- 若TLB miss就還是走page table這條路, 但是實際情況hit ratio常常會大於99%
- Context-switch到其他process的時候, 因為page number對應到的frame number不同, 所以通常TLB要flush清空, 這也是context-switch會讓效能變差的原因

## Memory Protection

- 每個page都會有一個protection bit, 最常拿來當作valid-invalid bit, 用來判斷page能否使用

### Valid-invalid bit

- Valid: 這個page/frame是process的logical address space
- Invalid: 這個page/frame不是process的logic address space, 可能是被swap到disk
- example: process的page只到page 5, 但page table 有7個row, 最後兩個就是invalid
    
    ![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%2014.png)
    
- 有可能page table設太大導致memory waste → 使用PTLR (page table length register)去紀錄process有幾個page, 然後page table只使用這個長度的entry

## Shared Pages

- paging share 相同的code, 這個code必須是reentrant
    
    ### Reentrant code (pure code)
    
    - It never change during execution
    - 例如text editors, compilers, web servers
- shared code在physical memory只有一份
- 多個virtual addresses都會mapping到同一個physical address
    
    ![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%2015.png)
    

## Page Table Memory Structure

- page table隨著時代變得越來越大, 而不容易去load
    - 4 GB($2^{32}$) logical address space with 4KB ($2^{12}$) page → 1百萬 ($2^{20}$) page table entry
    - 假設每個entry需要4bytes (32bit) → page table的Total size= 4MB
    - Page Table需要記憶體連續空間, 所以需要縮小
- Solutions:
    - Hierarchical Paging
    - Hash Page Tables
    - Inverted Page Table

### Hierarchical Paging

- two-level page table: 將page table切割成多個inner page table, 再用一個outer page table去紀錄各個inner page table的記憶體位址
    
    ![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%2016.png)
    
    - 可以當做array A[1000]轉成A[10][100]
- 將page table拆解, 就可以分散存在記憶體各處
- 64bit 的OS因為page table太大, 所以通常不會使用Hierarchical Paging
- 代價:
    - 3 memory access, 如果level更多層還會access更多次
    - inner page size固定, 會有memory waste的問題
- Examples:
    - SPARC (32-bit) and **Linux** use 3-level paging
    - Motorola 68030 (32-bit) use 4-level paging

### Hashed Page Table

- Commonly-used 在 address > 32bit
- Virtual page number被hashed到一個hash table, hash table的size不用先allocate, 所以不會有memory waste 的問題
- hash table的每個entry包含 (Virtual Page Number, Frame Number, Next Pointer) 等三個參數
- 會有linked list traverse waste time的問題
- Pointers waste memory
    
    ![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%2017.png)
    

### Inverted Page Table

- 不用Page table, 倒過來改用Frame Table, 所以亦稱做Frame Table
- 每個entry 在Frame Table包含 (PID, Page Number), 因為同樣的Frame可能在不同的Process中的Page使用, 是一對多的關係
- 可以減少原先Page table的memory 使用量過大的問題, 因為:
    - 每個Process都需要一個page table, 但使用frame table只需要保存一個table即可
    - 每個frame都幾乎會被用到, 所以不太會有memory waste的問題
- 缺點:
    - memory access time會增加, 因為從page讀到下一個page時, 需要找整個frame table去搜尋page
    - shared page/memory 會很困難
    
    ![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%2018.png)
    
- 圖中的page table就是frame table
- 主要是因為shared memory太困難, 所以Frame Table實際上很少被用到

## Segmentation (Non-Contiguous Memory Allocation)

- 和paging相同都是Non-Contiguous Memory Allocation, OS通常會同時有Segmentation與Paging
- 從user的角度去看memory-management, 依據code的內容區分成不同的部分就叫做segment
    
    ![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%2019.png)
    
- Logical address: (seg#, offset)
    - Offset和physical addr的長度相同
- Segmentation Table: 2 dimensional physical addresses; 每個table entry 有:
    - Base (4 bytes): the start physical addr
    - Limit (4 bytes): the length of the segment
- Segment-table base register (STBR): 紀錄segmentation table的physical addr
- Segment-table length register (STLR): 紀錄segment table的長度

### Segmentation Hardware

![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%2020.png)

- Limit register (圖中"<")是用來確定offset是否有超過Segmentation entry的limit, 若有超過就噴error
- MMU藉由assign segment的base address來allocate memory

### Example

![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%2021.png)

### Sharing of Segments

![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%2022.png)

- 在Segments做sharing是目前OS主要的做法, 如圖中editor指向同樣的記憶體位址, 但是存放的資料在不同的記憶體位址

## Segmentation with Paging

### Basic Concept

![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%2023.png)

- Segments是一個假想的空間, 介於logical address與physical address之間, 稱做linear address

### Address Translation

![Untitled](Memory%20Management%200b915bd7c9b04193a9ce2f96df99a026/Untitled%2024.png)

- Segmentation unit和paging unit都是MMU