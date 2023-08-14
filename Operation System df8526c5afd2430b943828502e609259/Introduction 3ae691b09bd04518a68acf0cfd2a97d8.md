# Introduction

### I/O operation

1. CPU 決定要read or write
2. Read: devices → controller buffers → memory
    
    Write: memory → controller buffers → devices
    
3. Interrupt 去告訴CPU做完了

但每次read/write 都要interrupt的話, 在high-speed device會花太多時間, 例如每次r/w要花4ms, 一次interrupt要2ms, 那就有一半的時間浪費在interrupt上了

### DMA(direct memory access 直接記憶體存取)

![Untitled](Introduction%203ae691b09bd04518a68acf0cfd2a97d8/Untitled.png)

- device可以透過DMA的技術獨立地直接讀寫系統記憶體而不需CPU介入處理, 這樣CPU可以專心在運算上而不被interrupt打擾
- 例如: 硬碟控制器、繪圖顯示卡、網路卡和音效卡

### Time-sharing

- 計算機科學中對資源的一種共享方式，利用多道程式與多工處理使多個使用者可以同時使用一台電腦。

### **System Call**

根據[維基百科](https://zh.wikipedia.org/zh-tw/%E7%B3%BB%E7%B5%B1%E8%AA%BF%E7%94%A8)，系統呼叫 (system call，簡稱為 syscall)，是指運行在 user space 的程式向作業系統核心請求需要更高權限運行的服務。系統呼叫提供 user space 和作業系統之間的介面。

簡單來說，system call 是 process 和 OS 之間的介面，當使用者程式需要 OS 的服務時，使用者程式便去呼叫 system call

System call 的種類大致分為

- Process Control
    - 對 process 做控制，如
        - process 的啟動與停止
        - 給予 process 記憶體空間
        - 讀取或設定 process 的屬性 (attributes)
        - 讓 process 作等待
    - e.g. `fork()`, `wait4()`
- File Management
    - 對檔案做控制，如
        - 檔案的建立與刪除
        - 檔案的開啟與關閉
        - 檔案的讀取與寫入
        - 讀取或設定檔案的屬性 (attributes)
    - e.g. `open()`, `read()`, `write()`, `close()`
- Device Management
    - 電腦系統各項硬體資源與周邊設備均為 device，當 process 在執行時會去要求取得資源或存取資料，如
        - Device 的要求與釋放
        - Device 資料的存取
        - 讀取或設定 device 的屬性 (attributes)
        - 邏輯連接或斷開 (Logically attach / detach)
    - e.g. `ioctl()`
- Information Maintenance
    - 對系統資料的更新與維護，如
        - 設定日期與時間
        - 存取系統資料
        - 存取 process、file、device 的屬性 (attributes)
    - e.g. `getpid()`, `getppid()`, `getpgid()`, `setpgid()`
- Communication
    - 當裝置以連線連通，執行訊息傳遞等 I/O 存取的行為，如
        - 建立或中斷連線
        - 輸入輸出網路資料
        - 狀態資訊的轉換 (Transfer status information)
        - 使用遠端裝置
    - e.g. `pipe()`, `socket()`

## Interrupt

CPU在執行程式過程中，遇到外部或內部的緊急事件(event)須優先處理，因此暫停執行當前的程式，轉而服務突發的事件，直到服務完畢，再回到原先的暫停處(為一記憶體地址)繼續執行原本尚未完成的程式。為事件服務的程式稱之為中斷服務程式或中斷處理程式。另外，和中斷類似的是trap或exceptions，通常因錯誤而發生(如不正確的記憶體存取，分頁錯誤)。

![https://i.imgur.com/9BCnUBK.png](https://i.imgur.com/9BCnUBK.png)

以上圖為例，在使用者對kernel發出system call運行process時，kernel即開始執行該程序，假如中間有突發的中斷產生，會從原先執行的process中的中斷點暫時跳出，進而對其進行服務，直到完成該中斷服務後再回到中斷點繼續執行原先的程序。

### External Interrupt(**外部中斷)**

外部中斷一般是指由計算機外設發出的中斷請求，如：鍵盤中斷、印表機中斷、定時器中斷等。外部中斷是可以遮蔽的中斷，也就是說，利用中斷控制器可以遮蔽這些外部裝置的中斷請求。

### Internal interrupt(**內部中斷)**

內部中斷是指因硬體出錯（如突然掉電、奇偶校驗錯等）或運算出錯（除數為零、運算溢位、單步中斷等）所引起的中斷。內部中斷是不可遮蔽的中斷。

### Software Interrupt(**軟體中斷)**

執行程式的過程中，當其需要OS提供服務時，程式會去呼叫某些system call以完成一些服務。

### Context Switch

讓多個 process 可以分享單一個 CPU 資源的計算過程。當 CPU Context Switch 到另一個 process，系統會儲存舊 process 的狀態並載入新 process 的狀態。Context switch 的時間是 overhead，耗時根據硬體配備而不同。

context指的是CPU registers的內容

交換的時機有以下幾種：

- 多工
- 中斷處理
- 用戶態或者內核態的交換 (可能)

### Storage

![Untitled](Introduction%203ae691b09bd04518a68acf0cfd2a97d8/Untitled%201.png)

![Untitled](Introduction%203ae691b09bd04518a68acf0cfd2a97d8/Untitled%202.png)

- 傳統cache都是由hardware控制, 現代開始有software控制的cache
- 上面那層是下面那層的cache, 下面那層是上面那層的backup, 例如 register是cache的cache, cache是register的backup

### OS Type

Closed-source OS: Windows, iOS

- 不開放Source code, 要錢, 可能比較有保障

Open-Source OS: GNU/Linux, BSD UNIX, and Solaris

- 開放Source code, 通常不要錢, 大家都可以改所以有可能比較有保證也可能沒有
- 要錢的代表: RedHat, 雖然開放source code, 但是提供服務幫人解決問題
- Solaris: Unix的分支, 原本是open source code, 但2009年被買走之後就不開放source code了