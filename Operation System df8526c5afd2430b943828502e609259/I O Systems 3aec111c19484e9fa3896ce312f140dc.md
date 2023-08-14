# I/O Systems

## **Overview**

- 電腦的兩個主要工作為 I/O & 計算
- I/O 設備: tape，HD，mouse，joystick搖桿，network card， screen，flash disks，etc
- I/O subsystem: 控制所有 I/O 設備的方法
- 兩個要解決的問題
    - 標準化的 HW/SW 介面
    - 如何處理不同種類的 I/O 設備
- Device drivers: 讓 I/O subsystem 可取得設備控制的介面
    - 類似 apps & OS 之間的系統呼叫
- Device categories
    - Storage devices: disks, tapes
    - Transmission devices: network cards, modems
    - Human-interface devices: keyboard, screen, mouse
    - Specialized devices: joystick, touchpad

## **I/O Hardware**

- **Port**
    - Host 與 I/O devices 之間的連接點，如 USB ports
- **Bus**
    - 傳輸管道，透過 wires(線路)和資料傳輸的 protocol，讓訊息在wires間傳輸，如 PCI bus
- **Controller**
    - 控制 port, bus 或 device 的電子系統
    - controller 可以有自己的 processor, memory 等 (如 SCSI controller)
- 傳統的 PC Bus 結構
    - 北橋晶片(Northbridge)控制比較快的 controller，bw、freq 高的
    - 南橋晶片控制比較慢的controller, 例如IDE、SATA、USB

![https://i.imgur.com/teHX0zK.png](https://i.imgur.com/teHX0zK.png)

## **I/O Method**

### **Basic I/O Method**

- 每一個 I/O port (device) 以unique的 port address 辨識
- 每一個 I/O port 包含四個 registers (1~4Bytes)
    - **Data-in register**: host 讀取資料輸入
    - **Data-out register**: host 寫入資料輸出
    - **Status register**: host 讀取確認 I/O 狀態
    - **Control register**: host 寫入控制設備
- Program 以特定的 I/O instruction 與 I/O port 互動
    - i/o instruction 裡一定有 port address
    
    ![https://i.imgur.com/atEqBPB.png](https://i.imgur.com/atEqBPB.png)
    

### **I/O Method Categorization**

- 如何取得設備 address 來分類
    - **Port-mapped I/O**
        - 使用與 memory 不同的address空間
        - 以特定的 I/O instruction 存取 (如 IN, OUT)
    - **Memory-mapped I/O**
        - memory 保留空間給 device
        - 將 device controller 的資料 map 到記憶體的某一塊區域，要事先註冊好
        - 以標準的 data-transfer instruction (e.g. MOV) 存取
        - 優點
            - 做大量資料傳輸時效率高 (CPU 只要把資料寫到記憶體即可，不用等待剩餘的 I/O 時間)(e.g. graphic card --> pinned memory)
        - 缺點
            - reliability 較低，檔案遺失的風險較高
            - 初始化的成本較高
- 如何與 device 互動
    - **Poll** (busy-waiting): processor 週期性地確認設備的 status register
    - **Interrupt**: 任務完成時，由設備通知 processor
- 誰來控制資料傳輸的動作
    - **Programmed I/O**: 由 CPU 控制資料搬移
        - CPU 會一直接收到 interrupt，速度變慢
    - **Direct memory access** (DMA) I/O: 由 DMA controller (一個獨立出來的 controller，不屬於任何一個 I/O device，專門負責資料傳輸)
        - 當資料量很大時使用
        - CPU 發出請求給 DMA 之後，DMA 開始傳輸資料， 完成時 interrupt 的方式通知 CPU
        - 通常搭配 memory-mapped I/O 和 interrupt I/O method
- 執行 DMA 的 6 個步驟 (以 read 為例)
    1. 設備的 driver 被告知希望從 disk 中搬移資料到 memory 中的 buffer X
    2. 設備的 driver 告知 disk controller 從 disk 中搬移 C bytes 的資料到 buffer X
    3. disk controller 初始化 DMA transfer
    4. disk controller 一個 byte 一個 byte 地將資料送給 DMA (這過程中有非常多 I/O request)
    5. DMA controller 將資料轉移到 buffer X
    6. 當資料搬移完成，DMA interrupt CPU 通知任務完成，整個過程CPU只會有一次interrupt
        
        ![https://i.imgur.com/wd3K0Yf.png](https://i.imgur.com/wd3K0Yf.png)
        

## **Kernel I/O Subsystem**

- **I/O Scheduling**: 透過 I/O queue 中任務的排序，提昇系統效能
    - e.g. disk I/O order scheduling
- **Buffering**: 資料在 I/O devices 之間傳輸時，將資料存在記憶體中
    - 使用 buffer 來調節不同 devices 的速度
        - devices 之間的傳輸速度不同 (e.g. disk to network card）
        - devices 之間傳輸的資料量不同
        - 支援 copy semantics
            - 多次的 data copy
- **Caching**: 持有一些複製資料的 fast memory
    - 持有的資料僅為原資料的複製
    - 為了減少資料搬運的 overhead，提昇存取的效能
- **Spooling**: 把設備輸出的資料 hold 住，等資料讀取完才執行動作
    - 將設備output的data先讀到 spooling 中，等待data都傳送到spooling之後，再送到設備
    - e.g. printer
- **Error handling** : 當 I/O error 發生時，如何處理並不讓系統崩潰
    - e.g. SCSI 設備回傳錯誤資訊
- **I/O protection**
    - Privileged instructions
        - 指為防止程式師編錯程式影響整個系統的工作，禁止一般應用設計者所寫的程式直接使用，僅允許作業系統與系統管理程式使用的一些指令。

### **UNIX I/O Kernel Data Structure**

- Unix 系統將所有的 I/O devices 視為檔案來管理
    
    ![https://i.imgur.com/wz8H4cq.png](https://i.imgur.com/wz8H4cq.png)
    

### Device-status Table

![Untitled](I%20O%20Systems%203aec111c19484e9fa3896ce312f140dc/Untitled.png)

### **Blocking and Nonblocking I/O**

- Blocking: process 等待 I/O 完成才繼續執行
    - 容易設計及理解
    - 但某些情況效率不足
    - use for **synchronous** communication & I/O
- NonBlocking: system call 之後會立即回傳
    - 透過 multi-threading 實作
    - **asynchronous** communication & I/O
    - 由 write, write callback 為一組執行 I/O 的動作，先 call write，完成時執行 write callback 告訴系統任務完成

### **Life Cycle of An I/O Request**

![https://i.imgur.com/2zP61B7.png](https://i.imgur.com/2zP61B7.png)

### **Performance**

- I/O 是影響系統效能的主要原因
    - I/O 的動作需要執行好幾層的 device driver code 來完成
    - 因為有好幾層的 code 要執行，每 call 一層 driver，其 subsystem 都是不同的process，造成大量的 context switches
    - 因為多層的呼叫，導致會多次的context switch，給CPU和hardware caches帶來壓力
    - 因為多層的呼叫，controller 跟 physical memory 之間有大量的資料複製
    - Interrupt handling 是一個相對成本較高的任務
        - 當 I/O 時間較小時，busy-waiting 可以比 interrupt 更有效率

### **Improving Performance**

- 減少 context switches 的次數
- 減少資料複製
- 小的 I/O 任務盡量不要使用 interrupt
- 使用 DMA
- 平衡 CPU, 記憶體, bus 及 I/O 效能達到最高的 throughput

## **Application I/O Interface**

### **A Kernel I/O Structure**

- **Device drivers** 讓 I/O subsystem 可取得設備控制的介面，並隱藏這些 device controllers 之間的差異
    
    ![https://i.imgur.com/ibmINMX.png](https://i.imgur.com/ibmINMX.png)
    

### Characteristics of I/O Devices

![Untitled](I%20O%20Systems%203aec111c19484e9fa3896ce312f140dc/Untitled%201.png)

### **I/O Device Class**

- Deivce class 在不同 OS 間通常是相同的
    - Block I/O
    - Char-stream I/O
    - Memory-mapped file access
    - Network sockets
    - Clock & timer interfaces
- Back-door interfaces (e.g. ioctl())，難以分類到以上類別
    - 呼叫 ioctl() 後，可以操作所有的 I/O devices
    - 讓 programmer 不用開發新的 system call，就可以取得任意 device driver 實作的函式
    - 這個後門也讓駭客容易入侵

### **Block & Char Devices**

- Block devices: disk drives
    - system calls: read(), write(), seek()，整個 memory buffer 的概念
    - Memory-mapped file 可以放在最上層
    - 以 interrupt 做 I/O
- Char-stream devices: mouse, keyboard, serial ports
    - system calls: get(), put()
    - Libraries layered on top allow line editing
    - 以 polling 做 I/O

### **Network Devices**

- 有很多種不同形式 (block, character…etc)，有自己的介面
    - system call: send(), recv(), select()
    - select() 回傳正在等待傳輸的 socket，因此可以不使用 busy waiting
- Many other approaches
    - pipes, FIFOS, STREAMS, message queues