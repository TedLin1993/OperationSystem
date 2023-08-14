# Virtual Memory Management

### Virtual memory: 將logical memory從physical memory中分離出來

- 可以run比physical memory還大的process
- 提升CPU/resources使用率
- 簡化programming, 使用者只須注意在自己的code上就好, 不需要花太多力氣在控制記憶體上面
- program跑得更快

![Untitled](Virtual%20Memory%20Management%2024c331dc081a4b74973c36fd030c64c4/Untitled.png)

## Demand Paging

- demand: 需要的page才load到memory, 而不是一次全部load到memory
    - 較少的I/O → 較快的response
    - 較少的memory使用 -> More users
- pure demand paging
    - process開始時沒有page
    - 一直到process需要的時候才將page load到memory
- swapper (midterm scheduler)是將整個process放到disk, 而pager是以page為單位放到disk
- Hardware support: 在page table加上 valid-invalid bit
    - 1: page in memory
    - 0: page not in memory
    - 一開始所有bit都設成0
- Secondary memory (swap space, backing store): high-speed disk

### Page Fault

- 一開始讀page是invalid的, 就trap到OS, 所以也稱page-fault trap
1. OS檢查internal table(in PCB)
    - Invalid reference → abort
    - Valid, just not in memory → continue
2. 找到一個empty frame (O(1)), 如果沒有empty frame就進Page Replacement
3. Swap the page from disk (swap space) 到 frame
4. Reset page table, valid-invalid bit = 1
5. Restart instruction

![Untitled](Virtual%20Memory%20Management%2024c331dc081a4b74973c36fd030c64c4/Untitled%201.png)

### Page Replacement

- 當page fault時沒有free frame的時候發生
- Swap一個frame到backing store
- Swap 原本要的page到frame
- 有不同的page replacement algorithms

### Demand Paging Performance

- Effective Access Time (EAT): $(1-p)*ma+p*pft$
    - $p$: page fault rate, $ma$: memory access time, $pft$: page fault time
- Examle: $ma$: 200ns, $pft$: 8ms
    - EAT = $(1-p)*200ns+p*8ms$ = $200ns+7999800ns*p$
- 如果 p = 0.001, EAT也會慢40倍, 所以page fault rate要非常非常低
- Program傾向locality, 所以可以有效降低page fault rate
    - locality: program的memory address會靠得很近
    - 一次page fault可以帶進4KB到memory, 也有效降低page fault rate

## Copy-on-Write

- parent process 和 child process 分享相同的frames
- 只有在process修改frame的時候, frame才會被copy
- COW可以更有效率的process creation (例如fork())
- fork 例子: 一開始parent 和 child 共用相同的frame
    
    ![Untitled](Virtual%20Memory%20Management%2024c331dc081a4b74973c36fd030c64c4/Untitled%202.png)
    
    接著parent 修改了frame
    
    ![Untitled](Virtual%20Memory%20Management%2024c331dc081a4b74973c36fd030c64c4/Untitled%203.png)
    

## Memory-Mapped Files

- Approach:
    - MMF讓file 在disk block mapping到memory frame, 所以從使用者角度上, file I/O 可以被當作一般的memory access
- Benefit:
    - Faster file access, 因為by pass File System, 直接去讀取硬碟
    - 可以同時讓多個process share 相同的file
- Concerns:
    - Security (access control)
    - data lost (斷電的話記憶體資訊會遺失)
    - more programming efforts (要自己判斷file內的格式, 不透過file system幫忙判斷)
    
    ![Untitled](Virtual%20Memory%20Management%2024c331dc081a4b74973c36fd030c64c4/Untitled%204.png)
    

## Page Replacement

- 當page fault時沒有free frame, 有兩種做法
    - swap一個記憶體內的process到disk
    - page replacement: 挑沒用到的page swap到disk
        - dirty bit: 判斷load到memory的page是否有被修改過, 只有被修改過的page才寫到disk
- 解決demand paging的兩大問題
    - frame-allocation algorithm: 決定多少frame要被allocated到一個process
    - page-replacement algorithm: 決定哪個frame要被replaced

### Page-replacement algorithm

- Goal: 最小化page-fault rate
- 常見的algorithms:
    - FIFO algorithm
    - Optimal algorithm
    - LRU algorithm
    - Counting algorithm: LFU, MFU
- 以下的Reference string均是: 1,2,3,4,1,2,5,1,2,3,4,5

### First-In-First-Out (FIFO) Algorithm

- 最舊的page被replace
- 例子:
    - Reference string: 1,2,3,4,1,2,5,1,2,3,4,5
    - 3 frames (available memory frames = 3)

![Untitled](Virtual%20Memory%20Management%2024c331dc081a4b74973c36fd030c64c4/Untitled%205.png)

                9次page fault

- frames數量與page fault數量的關係圖
    
    ![Untitled](Virtual%20Memory%20Management%2024c331dc081a4b74973c36fd030c64c4/Untitled%206.png)
    
- Belady's anomaly: 他是有可能frame增加反而page faults rate上升的

### Optimal (Belady) Algorithm

- Replace 未來最久沒被使用的page → 需要未來資訊
- 4frames: 1,2,3,4,1,2,5,1,2,3,4,5 → 6 page fault
    
    ![Untitled](Virtual%20Memory%20Management%2024c331dc081a4b74973c36fd030c64c4/Untitled%207.png)
    
- 現實上不會知道未來資訊, 所以此Algorithm只是拿來讓其他Algorithm比較而已

### LRU Algorithm (Least Recently Used)

- 近似optimal algorithm: Replace過去的最久沒被使用的page
- 這是最常用到的algorithm, 因為process有locality的傾向, 所以結果不錯
- Counter implementation
    - page referenced: time stamp
    - replacement: 移除最舊的 counter → linear search O(n)
- Stack implementation
    - page reference: 移到 double-linked list的top
    - replacement: 移除bottom page → O(1)
    - 例子: 4frames: 1,2,3,4,1,2,5,1,2,3,4,5
    
    ![Untitled](Virtual%20Memory%20Management%2024c331dc081a4b74973c36fd030c64c4/Untitled%208.png)
    

### Stack Algorithm

- optimal和LRU都屬於stack algorithm
- 定義: n frames的page table一定是n+1 frames的page table的subset
- Stack algorithm不會有Belady's anomaly, 也就是不會frames增加反而page fault增加的

### Counting Algorithm

- 每個counter都有一個page
- LFU Algorithm (least frequently used)
    - 頻率最低的page就被replace
- MFU Algorithm (Most frequently used)
    - 頻率最高的page就被replace, 有可能page之前太常被使用所以之後就不會被使用
- 這些做法不太會被使用, 因為implementation is expensive
- 結果也不是特別好

## Allocation of Frames

- 每個process都需要minimum數量的frames → 每個process要給多少frame?

### Frame Allocation

- Fixed allocation
    - Equal allocation: 有100 frame 和 5 process → 20 frames/process
    - Proportional allocation: 依照process的size去allocation frames
- Priority allocation
    - 使用Proportional allocation, 但是是基於priority而不是size
- Local allocation: 如果要replace, 都只replace自己process的frame
- Global allocation: replacement frame可以從其他process的frame去replace
    - 高piority process可以去搶low-priority process的frame
    - Common used, 表現很好
    - 每個process必須要有minimum數量的frame, 不然會有thrashing

## Thrashing

- 當一個process因為沒有足夠的frame而導致一直page fault, 導致一直I/O, 最後CPU使用率下降

![Untitled](Virtual%20Memory%20Management%2024c331dc081a4b74973c36fd030c64c4/Untitled%209.png)

- 定義: process花在paging的時間超過execution
- OS看到CPU使用率太低, 會嘗試更多的multiprogramming → 導致更多page fault和更多I/O, 惡性循環
- CPU使用率會直線下降, 甚至完全停住
- 為了避免Threshing, 有Working-set model與Page-fault frequency兩種方法

### Working-Set Model

- Locality model: process在executes時, 會從一個locality到另一個locality
- working-set window: parameter $\Delta$ (delta)
- working-set: 最近的$\Delta$ page內當作近似locality
- 例如$\Delta$ = 10:

![Untitled](Virtual%20Memory%20Management%2024c331dc081a4b74973c36fd030c64c4/Untitled%2010.png)

- 找出working-set size:
    - $WSS_i$: process i的working-set size
    - $D=\Sigma WSS_i$ (total demand frames)
    - 當 D > m (available frames) → thrashing
- OS需監控每個process的$WSS_i$, 並allocate足夠的frame
    - 若D << m, 增加MP的degree
    - 若D > m, 暫停process
- 缺點: too expensive for tracking, overhead太高

### Page Fault Frequency Scheme

- 建立page fault rate的upper bound和lower bound
- 當page fault rate 超過 upper bound → 從其他process allocate frame給他
- 當page fault rate 低於 lower bound → 這個process移除一些frame

![Untitled](Virtual%20Memory%20Management%2024c331dc081a4b74973c36fd030c64c4/Untitled%2011.png)