# File-System Implementation

# File-System Structure

- 記憶體跟硬碟之間資料傳輸 I/O 的單位為 **block**
    - one block is one or more **sectors**
    - one sector is usually 512 bytes, 一個block大約是512bytes~4KB
- 一個作業系統可以使用多個檔案系統
    - 常見的file system: NTFS, FAT32
    - 主要差異在disk allocation和file search implement
- 檔案系統設計的兩個主要的問題
    - user programs(使用者程序)的介面 (上層)
    - disk(硬碟)的介面 (下層)

## Layered File System

- File System 實作上通常由以下三個部份組成
    - manages metadata: 找到 file 並提供 file handler
    - logical <–> physical mapping: 將程序要讀取內容的位置轉換成 block 的位置
    - read d1, c73…: 轉換後 physical 的 hard-drive 及 block 位置

![Untitled](File-System%20Implementation%20115927b509084f5aa5eba93b168546f4/Untitled.png)

- 不同File system最大的差異是在file-organization module, 也就是怎麼分配logic和physical mapping關係
- Multiple File System:
    - 對使用者而言，只有一個檔案系統的 Interface (FS manager) 存在，而 FS manager 會去呼叫檔案對應的檔案系統來執行
        
        ![Untitled](File-System%20Implementation%20115927b509084f5aa5eba93b168546f4/Untitled%201.png)
        

# File System Implementation

## On-Disk Structure

- Structrue 要永久存在，所以一定要存在disk 上
- **Boot control block** (per partition): 存放啟動作業系統相關資訊的 partition
    - 開機用的系統槽，需要開機用到的code和data，讓他變成一個可開機的partition
    - 通常為 partition 的第一個 block (若是空的代表沒有 OS)
    - 不同File system命名不同
        - UFS (Unix File Sys.): **boot block**
        - NTFS: partition boot sector
- **Partition control block** (per partition): partition details
    - hard drive成為一個partition之後會有一個partition control block，OS看這個control block才會知道怎麼access這個partition
    - details: # of blocks, block size, free-block-list, free FCB pointers, etc
    - 不同File system命名不同
        - UFS: **superblock**
        - NTFS: Master File Table
- **File control block** (per file): 跟檔案相關的 details
    - 當format後變成file system，這時候會需要file的control block
    - details: 權限, 大小, data blocks 的位置 (physical 的位置)
    - UFS: **inode**, NTFS: stored in MFT (relational database)
- **Directory structure** (per file system): organize files

![Untitled](File-System%20Implementation%20115927b509084f5aa5eba93b168546f4/Untitled%202.png)

- Data Blocks: 存放file的地方

## In-Memory Structure

- **in-memory partition table**: information about each mounted partition
    - mount: 將partition table load到memory
- **in-memory directory structure**: information of recently accessed directories
- **system-wide open-file table**: contain a copy of each opened file’s FCB
    - 存File Control Block
- **per-process open-file table**: pointer (file handler/descriptor) to the corresponding entry in the above table

## File-Open & File-Read

![Untitled](File-System%20Implementation%20115927b509084f5aa5eba93b168546f4/Untitled%203.png)

- 開檔時，會先去 directory structure 看檔案是否在記憶體中，若不存在則去硬碟把檔案的 directory structure 資訊讀進來，找到我們要的節點並指向 file control block，這樣就有了檔案的所有資訊
- 讀檔時，File handler 為 per-process open-file table 的某一個 entry，拿到 handler 確保檔案的確 open 之後，必須從 system-wide table 取得檔案的實際位置，才能從硬碟存取檔案內容

## File Creation Procedure

1. OS allocates a new **FCB**

2. Update **directory structure**

1. OS reads in the corresponding directory structure into memory

- 確認directory存在

2. Updates the dir structure with the new file name and the FCB

- 將新的檔案名稱以及 FCB 更新進 directory structure

3. (在檔案關閉後) OS 將 directory structure 寫回硬碟中

3. The file appears in user’s dir command

## **Virtual File System**

- VFS 提供物件導向的方法實作 file system
- 對於不同檔案系統，VFS 使用同一個 **system call interface**
- VFS 依 partition info 呼叫適當的 file system routines

![Untitled](File-System%20Implementation%20115927b509084f5aa5eba93b168546f4/Untitled%204.png)

- 在 Linux VFS 中有 4 個主要的物件類型
    - inode --> individual file單獨的檔案 (on disk)
    - file object --> 開啟的檔案 (檔案開啟後的操作物件, in memory)
    - superblock object --> 整個 file system (對應到 partition)
    - dentry object --> 單獨的 directory entry
- VFS 定義一系列需要實作的操作
    - int open(…) --> 開啟檔案
    - ssize_t read() --> 讀取檔案

## **Directory Implementation**

- Linear lists (Linked list)
    - 檔案名稱的串列，且有相應的指標指向 data blocks
    - 容易實作但 performance 差
        - insertion, deletion, searching
- Hash table --> linear list w/ hash data structure
    - O(1) 搜尋時間
    - 每一個 hash key 對應到一個 linked list，避免 hash collision
    - hash table 通常有固定數量的 entry 數量

# Disk Allocation Methods

## Outline

- file分配到disk blocks的分配方法
- Allocation strategy:
    - Contiguous allocation
    - Linked allocation
    - Indexed allocation

## **Contiguous allocation**

- 檔案在硬碟中的 block 擺放位置要連續
    - 不需要 traverse 整個硬碟，只要簡單的計算就知道檔案在哪一個 block → 磁頭轉動的數量最小化
    - Directory entry 只要存 start 跟 length，非常簡單
- 無論是 sequential 或 random 的存取都可以有效率地實作
- 問題:
    - 很容易有 External fragment  --> compaction
        - External fragment: file和file之間的空隙太多, disk的利用率很差
        - compaction: 磁碟重組
    - 檔案不能隨意地增長
        - 解法 --> extend-based FS (跟 linked list 結合)

![Untitled](File-System%20Implementation%20115927b509084f5aa5eba93b168546f4/Untitled%205.png)

### **Extent-Based File System**

- Contiguous allocation的延伸應用
- 許多新的 file system 使用 modified contiguous allocation scheme
- extent 是一個硬碟的連續 blocks
    - 一個檔案可以有一個或多個 extents
    - 一個 extent 若用完了，系統會給檔案另一個 extent，利用 linked list 串起來
    - extent entry: (starting block #, length, pointer to next extent)
    - extents block 必須有一個 pointer 紀錄下一個 extent 的位置
- 缺點
    - Random access 的成本增加 (例如檔案在第三個 extent，必須要讀前兩個 extent 才知道第三個 extent 的位置，增加硬碟讀取的次數)
    - 仍然有 internal & external 的 fragment
        - 如果extent長度是固定的就會有internal fragment

## L**inked Allocation**

- 檔案由一個 linked list of blocks 所組成
    - 每一個 block 有 pointer 指向下一個 block
    - 因為需要空間放 pointer，所以 block 中實際存放的大小(data portion)為 block size - pointer size
- Directory 只需要存起始跟結束的 block id
- 建立新檔案時並不需同時就宣告檔案大小
- 缺點
    - 只適合 sequential-access
        - random access 需要遍歷 linked list
        - 因為 link pointer 存放在 block 中，每次存取 block 硬碟都要讀取一次
    - 需要 pointer 的空間 (4/512 = 0.78%)
        - 一個block 512 bytes, 需要4 bytes存pointer
        - 解法：unit = cluster of blocks (把很多 blocks 組成一個大的 blocks，減少遍歷的長度)
            - 會有 internal fragment 的問題
    - Reliability
        - 只要遺失一個 link 等於整份檔案遺失

![https://i.imgur.com/m5fuCw4.png](https://i.imgur.com/m5fuCw4.png)

### **FAT (File Allocation Table) file system**

- FAT32
    - Linked Allocation的延伸應用
    - 在 MS/DOS & OS/2 中使用
    - 將 links 存成一個 table (所有檔案的 links)
    - 每一個 table entry 大小為 32 bits
    - 放在 partition 的起始點

![https://i.imgur.com/UHnipmm.png](https://i.imgur.com/UHnipmm.png)

- FAT (table) 通常會 cache 在記憶體中
    - Random access 的效率改善
    - 硬碟只要讀取 FAT 就可以找到檔案 block 的位置

## **Indexed Allocation**

- Directory 擁有 file index block 的 address
- 每一個 file 有自己的 index block
- index block 儲存檔案資料的 block id
- 將所有 blocks 的 pointer 集中存在一個 block 中

![https://i.imgur.com/h7onJmv.png](https://i.imgur.com/h7onJmv.png)

- 優點
    - direct & random access 非常有效率
    - 沒有 external fragmentation
    - 很容易去建立檔案 (只要有 free list，就知道哪裡有空間放檔案)
- 缺點
    - 需要額外放置 index 的 blocks，這些 blocks 只能放 index，空間被浪費掉
    - 另一個問題是，當檔案太大，一個 index block 存不下時，那 index block 應該要多大？
        - **linked scheme**
        - **multilevel index**
        - **combined**（inode in BSD UNIX）

### **Linked Indexed Scheme**

- 將 index blocks 用 linked list 串起來
- 適合 small to medium size 的檔案
    
    ![https://i.imgur.com/66M6aXx.png](https://i.imgur.com/66M6aXx.png)
    

### **Multilevel Scheme (two-level)**

- 第一階的 index block 儲存第二階 index blocks 的指標
- 第二階 index blocks 的指標才指向 file blocks
- 適合 large size 的檔案
    
    ![https://i.imgur.com/HNuDtHa.png](https://i.imgur.com/HNuDtHa.png)
    

### **Combined Scheme: UNIX inode**

- File pointer: 4B (32 bits) --> 只能存放 4GB (2^32) 的檔案
- 可以讓不同大小的檔案，使用不同類型的 index scheme，提昇檔案讀取的效率
- 假設一個 block 的 entry 數量為 12，綠色因為每個 index blocks 還要存 2 個其他 index blokcs 的 pointer，所以檔案大小乘上 2^10
    
    ![https://i.imgur.com/BaStY7E.png](https://i.imgur.com/BaStY7E.png)
    

# Free-Space Management

- Free-Space list: 紀錄所有硬碟中 free 的 blocks
- Scheme
    - Bit vector
    - Linked list (same as linked allocation)
    - Grouping (same as linked index allocation)
    - Counting (same as contiguous allocation)
- 檔案系統通常用與管理檔案相同的方式管理 free space

## **Bit vector**

- Bit Vector (bitmap): 每個 block 以一個 bit 來表示，0 為 free; 1 為 occupied
- 優點
    - 簡單且快速 (硬體支援 bit-manipulation instruction)
- 缺點
    - 為了效能必須存在快取中，當檔案很大時 bitmap 會非常大，超過快取的空間
    - 例如一個 1TB(4KB block) 的硬碟需要 32MB 的 bitmap
        
        ![https://i.imgur.com/Ye3Suov.png](https://i.imgur.com/Ye3Suov.png)
        

## **Grouping & Counting**

- Linked List
    - 所有的 free blocks 利用 linked-list 串在一起
    - 將第一個 free block pointer 存在 disk 和快取中的特別位置
    - 遍歷 list 效率低
    - 可參考 FAT 的作法所有的 link-pointers 存在一個 table 當中
- Grouping
    - 與 linked-index allocation 一樣
    - 將 n free blocks 的 address 放在 1st block
    - 前（n-1） pointers 是 free blocks
    - 最後一個 pointers 指向另一個 grouping block
- Counting
    - 與 contiguous allocation 一樣
    - 儲存第一個 free block 的指標和 contiguous free blocks 的數量