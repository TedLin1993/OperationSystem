# Mass Storage System

## **Disk Structure**

- 硬碟被 addressed 為一維的 logical blocks 陣列
    - logical block 為最小的轉換單位，一個 block 有一個或多個 **sector**
- Logical block 以 sequential 的方式mapping(映射)到硬碟當中
    - Sector 0 為最外圈 1st track 的 1st sector
    - 從最外圈到最內圈
        
        ![https://i.imgur.com/N5zLZWu.png](https://i.imgur.com/N5zLZWu.png)
        

### **Sectors per Track**

- **Constant linear velocity** (CLV)
    - 每一個 track bits 的密度相等
    - 外圈有更多的 sectors
    - 保持相同的 data rate, 然而內圈的資料量較小
        
        → 所以內圈的旋轉速度要加快
        
    - 應用： CD-ROM & DVD-ROM
- **Constant angular velocity** (CAV)
    - 保持相同的旋轉速度
    - 內圈的 bit density 較高
    - 保持相同的 data rate
    - 應用：硬碟

### **Disk I/O**

- 硬碟用 I/O bus 跟電腦連接
    - EIDE, ATA, SATA (Serial ATA), USB, SCSI 等等
    - I/O bus 利用 controller 來控制
        - Host controller (computer end)
        - Disk controller (disk drive 內建)

## **Disk Scheduling**

### **Introduction**

- Disk-access 時間主要分成三個部份
    - **seek time**: 將讀寫頭移到需求的cylinder(環)上 (因為移動磁頭是機械式的行為，seek time 是影響硬碟讀取時間最主要的成份，scheduling 要盡可能讓磁頭在讀取多個檔案時能沿著一個方向移動)
    - **rotational latency**: 旋轉讀寫頭到需求的 sector 上, 硬碟7200轉, 所以很快, 不須優化
    - **read time**: 資料轉移的時間, 很快不須優化
- Disk bandwidth
    - # of bytes transferred/(complete time of last req – start time of first req)

![Untitled](Mass%20Storage%20System%20e2a6e70bebab432f904b6b0c108daad5/Untitled.png)

### **Disk Scheduling**

- 最小化 seek time
    - Seek time ~= seek distance
- 處理硬碟 I/O 請求的算法
    - FCFS（first-come, first-served）
    - SSTF（shortest-seek-time-first）
        - 普遍使用，但不是最佳的算法，且可能有 starvation 的問題
        - 適合 loading 輕的情況
    - SCAN
        - 在高負載的情況下表現佳
        - 沒有 starvation 的問題
    - C-SCAN（circular SCAN）
        - SCAN 的延伸，讓等待時間更為平均
    - LOOK and C-LOOK

### **FCFS (First-Come-First-Served)**

- 對磁頭移動來說，只關心 track 的 ID
- 假設現在的 request queue 為 98, 183, 37, 122, 14, 124, 65, 67，Head pointer 為 53，總移動量為 640
- 每次pointer的位置都離很遠的話就會非常耗時
    
    ![https://i.imgur.com/UDlFFEG.png](https://i.imgur.com/UDlFFEG.png)
    

### **SSTF (Shortest-Seek-Time-First)**

- SSTF 排程為 SJF 的一種，可能導致 starvation 的問題
- 總移動量為 236
- Greedy的作法, 移動距離不一定是最短的, 有可能一直來回擺動
    
    ![https://i.imgur.com/pn21WJd.png](https://i.imgur.com/pn21WJd.png)
    

### **SCAN Scheduling**

- 磁頭先往一邊走再往另一邊走
- A.K.A elevator algorithm
- 各個請求被讀取的時間容易不平均 (ex: 若磁頭往左邊移動但新請求剛好再最右側)
- 總移動量為 236
- request等待時間有可能很久, 若磁頭往右走的時候剛好新的request在最左邊, 那就必須等到磁頭移動到最右邊再回來才能讀到
    
    ![https://i.imgur.com/9rZE9O7.png](https://i.imgur.com/9rZE9O7.png)
    

### **C-SCAN Scheduling**

- 磁頭只往一個方向移動
- 同方向讀到底之後，會直接回到另一頭 (Round-Robin)
    - 例如磁頭讀到199後, 直接先回到0再開始讀
    - 從最右邊到最左邊, 因為中間沒有讀取, 所以不用很久
- 各個讀取時間比 SCAN 算法更為平均
- 總移動量 382
    
    ![https://i.imgur.com/mTyM3s2.png](https://i.imgur.com/mTyM3s2.png)
    

### **C-Look**

- 會看是否有新的請求與目前磁頭移動方向同向，有的話繼續延該方向移動
- 若同方向沒有新的請求，則磁頭不會回到最開頭，會從另一個方向最靠近目前位置的請求開始
- 若決定好移動位置，但此時有個介於之間的請求進來 (如 53 -> 65，決定了之後 64 才進來)，必須要等到下一次才會被讀取
- Scan會將磁頭走到底再回頭, Look只會走到最遠的request就會回頭
    
    ![https://i.imgur.com/WEM6flX.png](https://i.imgur.com/WEM6flX.png)
    

### Selecting Disk-Scheduling Algorithm

- SSTF
    - common and has a natural appeal, but not optimal
- SCAN
    - perform better for disks with heavy load
    - 沒有 starvation 問題
- C-SCAN
    - 等待時間較平均
- 效能會受到 file-allocation method 所影響
    - Contiguous: 較少磁頭移動
    - Indexed & linked: 較多磁頭移動

## **Disk Management**

### **Disk Formatting**

- Low-level formatting (or physical formatting):
    - 硬碟出廠時，已完成格式化作業
    - 將 disk (magnetic recording material)切分成數個 dist controller 可讀寫的 **sectors**
    - 每個 sector = header + data area + trailer
        - header & trailer 存放 sector 的數量及 ECC (error-correcting code)
        - ECC 藉由所有在 data area 的 bytes 計算，用 trailer 的 ECC 檢查硬碟有無壞軌
        - sector size通常都直接當作是data area的size
        - data area size 通常為 512B, 1KB or 4KB
- OS 接著做以下兩個步驟來使用硬碟
    - 將硬碟 partition 成一個或多個 groups of cylnders
    - logical formatting (建立 file system)

### **Boot Block**

- Bootstrap program
    - 初始化 CPU, registers, device, controllers, memory 並且啟動 OS
    - 第一階段 **bootstrap code** 存放在主機板的 ROM (Read Only Memory)
    - 完整的 bootstrap 放在boot disk (系統碟)的 boot block
- Booting from a Disk in Windows 2000
    1. 執行 ROM 中的 bootstrap code
    2. 讀取 MBR (Master boot record) 的 boot code
    3. 利用 partition table 找到 boot 的 partition 位置
    4. 讀取 boot sector/block 並繼續開機作業
        
        ![https://i.imgur.com/tje8kZk.png](https://i.imgur.com/tje8kZk.png)
        

### Bad Blocks

- Simple disks like IDE disks
    - 人工執行格式化程式來標記 bad block 對應的 FAT entry
    - bad blocks(壞軌的區域)不允許用來 allocation
    - 他不管上層軟體怎樣, 反正壞軌就是不給用, 所以可能會導致開不了機
- Sophisticated disks like SCSI disks
    - disk controller 會維護一個 bad blocks list
    - List 在硬碟的生命週期會持續更新
- **Sector sparing** (forwarding): 將壞軌區的資料 map 到備用的區域
    - 可能會影響到 disk-scheduling 的效能 (Physical 位置不連續)
    - 在硬碟 formatting 時，每個 cylinder 會留少數備用的 sectors
- **Sector slipping**: 若要避免 physical 位置不連續的問題，則必需將整個資料做搬移

### Swap-Space Management

- Swap-space: 虛擬記憶體使用硬碟空間 (swap-space) 作為主記憶體的延伸
- UNIX: 允許使用多個 swap spaces
- Location
    - 一般檔案系統的一部分 (e.g. NT)
        - 效率較差
    - 獨立的disk partition (raw partition)
        - 效率較好
        - 但 Size 固定
    - 允許上面兩種方式 (e.g. Linux)

### Swap Space Allocation

- 1st version: process 創建時放在記憶體，並複製一份完整的內容到連續的硬碟空間。主要問題是所有內容都複製，造成空間浪費
- 2nd version: 只複製要被 swap 的 pages 到 swap space
    - Solaris 1:
        - text segments(程式碼) 可以從檔案系統讀取，所以當 swap out 的時候直接捨棄
        - 只有 anonymous memory (stack, heap, etc) 這些動態配置的記憶體空間，在虛擬記憶體創建時，會存到 swap space
    - Solaris 2:
        - 更進一步只有當動態配置的記憶體空間需要被 swap out 時，才動態配置 swap-space，而非虛擬記憶體創建時就建立副本
            
            ![https://i.imgur.com/9nU7iRq.png](https://i.imgur.com/9nU7iRq.png)
            

## **RAID**

- RAID (Redundant Arrays of Inexpensive Disks)
    - 用多個便宜的硬碟組成
    - 利用 redundancy 提昇可靠度
    - 藉由 parallelism 提昇效能
- RAID 不同 levels的作法
    - Striping: 將檔案切割放到多個硬碟上
    - Mirror (Replication): 備份
    - Error-correcting code (ECC) & Parity bit: 產生 redundant 的 code 來做 Error detection 甚至檔案修補

### **RAID 0 & RAID 1**

- RAID 0: non-redudant striping
    - 透過平行讀取增加效率
    - I/O 的 bandwidth 與 striping 的數量成正比
        - 若有 N 顆硬碟，則讀寫的頻寬提昇 N 倍
            
            ![https://i.imgur.com/oF5LbvZ.png](https://i.imgur.com/oF5LbvZ.png)
            
- RAID 1: **Mirrored** disks
    - 藉由 redundancy 提昇可靠度
        - 也就是說一半的空間是拿來做備份的
        - 讀取的頻寬與硬碟數量成正比
        - 寫入的頻寬保持不變，甚至比單顆硬碟還差一些

![https://i.imgur.com/KLZBskv.png](https://i.imgur.com/KLZBskv.png)

### **RAID 2: Hamming code**

- 以 Hamming code 將資料編碼後分割成獨立的位元
- 在資料中加入錯誤修正碼

- e.g.: Hamming code(7, 4)
    - 4 data bits (在 4 個 disks 上) + 3 paritu bits (在 3 個 disks 上)
    - 每一個 parity bits 為 3 個 data bits 的線性編碼
- 當只有一個 disk 失效時，可以復原資料
    - 可以偵測到兩個 disks(bits) 的錯誤
    - 但只能還原 1 bit 的錯誤

![https://i.imgur.com/Tt2nQng.png](https://i.imgur.com/Tt2nQng.png)

- 比 RAID 1 更好的空間使用率 (RAID 1 100% redundancy v.s. RAID 2 75 % redundancy)
    
    ![https://i.imgur.com/JIQOR5j.png](https://i.imgur.com/JIQOR5j.png)
    

### **RAID 3 & 4: Parity Bit**

- 雖然 RAID 2 可以偵測硬碟錯誤，但 disk controller 就能夠偵測 sector 的讀取是否正確
    - 因為 low level 的 controller 會提供哪個 sector 壞掉，所以 1 個 parity bit 就足夠修正 1 個 disk 的失效
- RAID 3: Bit-level striping; RAID 4: Block-level striping
    - 優點是更好的空間使用率 (33% overhead)
    - 缺點是計算與儲存 parity bit 的 cost
- RAID 4: 因為 controller 不需要從多顆硬碟重建 block，有更高的 throughput
    - RAID3 一個 byte 會跨多顆硬碟，速度取決於最慢的那個 bit; 而 RAID 4 可以一次從單顆硬碟讀取多個 bits 的資料

![https://i.imgur.com/mjNRVzx.png](https://i.imgur.com/mjNRVzx.png)

### **RAID 5: Distributed Parity**

- 最常用的RAID
- RAID 2~4 都把 parity 集中在單獨的一群硬碟中，對使用者來說這些硬碟等於不存在
- RAID 5 則將資料及 parity 放在一起
- 可以避免過度使用單一顆硬碟
- RAID 3 & 4 讀的時候可以平行處理，但寫的時候因為 parity bits 集中擺放，等於只有一顆硬碟的效率
    
    ![https://i.imgur.com/cx3DbN9.png](https://i.imgur.com/cx3DbN9.png)
    
- RAID 5 讀取的 BW 為 N 倍, RAID3/4則只有 (N-1)倍, 因為RAID3/4將parity放在獨立一顆Disk
- RAID 5 寫入的 BW
    - 法一:
        - 1. 讀取所有未變動的資料 (N-2) bits
        - 2. 重新計算 parity bit
        - 3. 將變動及 parity bit 寫回硬碟
        - BW = N / ((N-2) + 2) = 1 倍 --> 和一顆硬碟的BW相同
    - 法二:
        - 1. 只讀取變動及 parity 的 bit
        - 2. 利用更新前後的差異計算新的 partiy
        - 3. 將變動及 parity bit 寫回硬碟
        - BW = N / (2+2) = N/4 倍 --> 有加速效果
            
            ![https://i.imgur.com/VeVvw9E.png](https://i.imgur.com/VeVvw9E.png)
            

### **RAID 6: P+Q Dual Parity Redundancy**

- 類似 RAID 5, 但儲存更多的 redundant 資訊來防止多顆硬碟失效的情況
- 使用 ECE code (i.e. Error Correction Code) 而非 single parity bit
- Parity bits 通常也 strips 到多顆硬碟中
- 複雜度及 overhead 高，除非資料真的很重要，一般不會採用這種實作
    
    ![https://i.imgur.com/hhklA5G.png](https://i.imgur.com/hhklA5G.png)
    

### **Hybrid RAID**

- RAID 0+1: Stripe then replicate
- RAID 1+0: Replicate then stripe
    
    ![https://i.imgur.com/PMY4edy.png](https://i.imgur.com/PMY4edy.png)