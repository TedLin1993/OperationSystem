# CPU Scheduling

## Basic Concepts

- Process執行過程中, 不是在CPU burst就是在I/O burst

![Untitled](CPU%20Scheduling%201f11c50db8904406a24d6bae2aec3073/Untitled.png)

### I/O bound vs CPU bound

- I/O-bound process – spends more time doing I/O than computations, many short CPU bursts. ex : 等待鍵盤輸入的程式
- CPU-bound process – spends more time doing computations; few very long CPU bursts. ex: 壓縮程式

### Histogram of CPU-Burst durations

![Untitled](CPU%20Scheduling%201f11c50db8904406a24d6bae2aec3073/Untitled%201.png)

- 絕大部分情況都是很多短的CPU-burst與很少長的CPU-burst

### Preemptive vs. Non-preemptive

- CPU scheduling 會發生在process:
    1. Switches from running to waiting state (CPU做完)
    2. Switched from running to ready state (time sharing)
    3. Switches from waiting to ready (I/O做完)
    4. Terminates (Process 執行完)
- Non-preemptive scheduling
    - 只會在上面1和4的情況去scheduling, 一定要CPU burst完才做
    - E.g, Windows 3.x
- Preemptive scheduling
    - 所有case都會做scheduling, CPU burst到一半也可能會做
    - E.g, Windows 95與之後的版本, Mac OS X
- Preemptive Issues
    - CPU burst到一半做context switch會造成synchronization
    - 會影響OS kernel的設計, 需要做synchronization的處理
    - Unix solution: kernel process維持Preemptive, 不給打斷

### Dispatcher

- Scheduler負責選擇誰要被換, Dispatcher是另一個module, 負責換人這個動作
    - context switch就是dispatcher下面的動作
- Dispatch latency: dispatcher 從停止一個process到開始一個process之間的時間
    - Scheduling time
    - Interrupt re-enabling time
    - Context switch time

## Scheduling Algorithms

### Scheduling Criteria (衡量系統的方法)

- CPU utilization: CPU使用率
- Throughput: 每單位時間內完成的process數
- Turnaround time: 平均process執行時間
- Waiting time: Process在ready queue的時間
- Response time: 從submission到第一個response的時間, response: CPU開始burst

### FCFS Scheduling (First Come First Serve)

- Process (Burst Time) in arriving order:
    - P1 (24), P2 (3), P3 (3)
        
        ![Untitled](CPU%20Scheduling%201f11c50db8904406a24d6bae2aec3073/Untitled%202.png)
        
    - Waiting time: P1 = 0, P2 = 24, P3 = 27
    - Average Waiting Time (AWT): (0+24+27)/3=17
- Preemptive與Non-preemptive行為不變, 都是先進來的先run
- Convoy effect (護衛效應): 短process在等長process先run, 導致AWT變高

### Shortest-Job-First (SJF) Scheduling

- 最短的process先run, 最小化Waiting time
- Preemptive與Non-preemptive行為不同
    - Preemptive: 新來的process若有較短的burst, 就會preemptive
    - Non-preemptive: CPU在run一個process時不可以preempted, 一直到CPU run完才能換下一個process
- Example
    
    ![Untitled](CPU%20Scheduling%201f11c50db8904406a24d6bae2aec3073/Untitled%203.png)
    
    - Non-Preemptive
        
        ![Untitled](CPU%20Scheduling%201f11c50db8904406a24d6bae2aec3073/Untitled%204.png)
        
    - Preemptive
        
        ![Untitled](CPU%20Scheduling%201f11c50db8904406a24d6bae2aec3073/Untitled%205.png)
        

### Priority Scheduling

- 以Priority number來衡量process
- CPU 被allocate給最高的priority process
- SJF也是一種priority scheduling, 他的priority就是較短的burst time
- 問題: 會有starvation問題(low priority process永遠不被execute)
- 解法: aging (時間增加會讓priority增加)

### Round-Robin (RR) Scheduling

- 每個process輪流做一段cpu time(Time Quantum), 通常是10~100 ms
- 當TQ結束, process被preempted並被加到ready queue的最後
- Performance:
    - TQ large → FIFO
    - TQ small → overhead(context switch)增加
- 例子: TQ=20
    
    ![Untitled](CPU%20Scheduling%201f11c50db8904406a24d6bae2aec3073/Untitled%206.png)
    
- 通常turnaround time會比SJF還長, 但是response time比SJF還短

### Multilevel Queue Scheduling

- 實際OS其實會各種scheduling混在一起, 看情況選擇使用哪種方法
- 先定義幾種Queue, 並且這些Queue要用哪種Scheduling, 再隨機選擇要先執行哪一個queue
    
    ![Untitled](CPU%20Scheduling%201f11c50db8904406a24d6bae2aec3073/Untitled%207.png)
    

### Multilvel Feedback Queue Scheduling

- 有上下區分的Multilevel Queue Scheduling, 會先執行最上層的Queue的Process
- Process在Queue之間可以移動
- 可以讓越I/O-bound process移到上層, 因為有較短的CPU burst, 並且讓CPU-bound的process移到下層
- Example: 第一層的Process CPU time需≤ 8ms, 第二層CPU time需≤16ms, 第三層FCFS
    
    ![Untitled](CPU%20Scheduling%201f11c50db8904406a24d6bae2aec3073/Untitled%208.png)
    

## Multi-Processor Scheduling

### Asymmetric multiprocessing:

- 有一個master processor 主導, 決定process要run在哪個processor上
- processor就是CPU

### Symmetric multiprocessing (SMP):

- 所有process都在一個common ready queue給所有processor去搶要給誰run
- 需要synchronization mechanism
- 一般電腦都是SMP架構

### Processor affinity

- 定義: process綁定在特定的core上面, 只能給那個core run
    - 一個CPU可以有1到多個core
- 因為CPU L1 cache可以cache住這個process, 所以可以run得更快
- soft affinity: process有可能migrate到其他core去run, 例如core讓process等太久
- hard affinity: process不能migrate

## NUMA and CPU Scheduling

### NUMA (non-uniform memory access):

- 每個CPU都有自己用的memory
    
    ![Untitled](CPU%20Scheduling%201f11c50db8904406a24d6bae2aec3073/Untitled%209.png)
    

## Load-balancing

- 保持工作負載均勻分佈在所有processors上

### Two stragegies

- Push migration: loading高的 processor push process 到loading低的processor
- Pull migration: loading低的 processor pull process 從loading高的 processor
- 整體系統loading高的會選擇Pull migration, 這樣processor不會浪費時間去問其他誰有空; 反之整體系統loading低的會選擇Push migration

## Multi-core Processor Scheduling

### Multi-core Processor

- 更快且消耗更少
- memory stall: memory 的I/O太慢導致CPU/core在空轉

![Untitled](CPU%20Scheduling%201f11c50db8904406a24d6bae2aec3073/Untitled%2010.png)

### Multi-threaded multi-core systems

- 1個core同時執行多個hardware threads, 例如Intel Hyper-threading
- 一個thread在memory stall的時候可以執行另一個thread, 讓CPU使用率更高

![Untitled](CPU%20Scheduling%201f11c50db8904406a24d6bae2aec3073/Untitled%2011.png)

### Two ways to multithread a processor

- 當memory stall發生時, processor switch到其他thread有兩種做法
- coarse-grained: 會清空instruction pipeline所以cost較高
- fine-grained (interleaves): 硬體register紀錄instruction pipeline的state所以cost較低

### Scheduling for Multi-threaded multi-core systems

- 1st level: Choose which software thread to run on eachhardware thread (logical processor)
- 2nd level: How each core decides which hardware thread to run

## Real-Time Scheduling

- Real-time不是越快越好, 而是能在dealine前將事情做完
- Soft real-time: 盡量避免miss deadline, 但仍有可能發生, 例如media streaming
- Hard real-time: 完全不能miss deadline, 例如汽車/武器/核電廠

### Real-Time Scheduling Algorithms

- FCFS → 不是real time
- Rate-Monotonic (RM) algorithm: period越短, priority越高
- Earliest-Deadline-First (EDF) algorithm: deadline越早, priority越高

## Operating System Examples

### Solaris Scheduler

- Priority-based multilevel feedback queue scheduling
- Six classes of scheduling:
    - 將thread分class, thread不會在不同的class間切換
    - 分成: real-time, system, time sharing, interactive, fair share, fixed priority
- Each class has its own priorities and scheduling algorithm
- The scheduler converts the classspecific priorities into global priorities

![Untitled](CPU%20Scheduling%201f11c50db8904406a24d6bae2aec3073/Untitled%2012.png)

### Windows XP Scheduler

- 和Solaris 差不多, 只是priority 切割的數字有些不同
- Similar to Solaris: Multilevel feedback queue
- Scheduling: from the highest priority queue to lowest priority queue (priority level: 0 ~ 31)
    - The highest-priority thread always run
    - Round-robin in each priority queue
- Priority changes dynamically except for Real-Time class
    
    ![Untitled](CPU%20Scheduling%201f11c50db8904406a24d6bae2aec3073/Untitled%2013.png)
    

### Linux Scheduler

- Preemptive priority based scheduling
    - But allows only user mode processes to be preempted
    - Two separate process priority ranges
    - Lower values indicate higher priorities, 數字越小Priority越高
    - Higher priority with longer time quantum
- Real-time tasks: (priority range 0~99)
    - static priorities
- Other tasks: (priority range 100~140)
    - dynamic priorities based on task interactivity

![Untitled](CPU%20Scheduling%201f11c50db8904406a24d6bae2aec3073/Untitled%2014.png)

- Scheduling algorithm
    - A runnable task is eligible for execution as long as it has remaining time quantum
    - When a task exhausted its time quantum, it is considered expired and not eligible for execution, 也就是time quantum用完, 這個task就不能執行了
        - 這樣能讓priority低的也一定會run到
    - New priority and time quantum is given after a task is expired
    - active array清空之後才會把expired array的task移到active array
    
    ![Untitled](CPU%20Scheduling%201f11c50db8904406a24d6bae2aec3073/Untitled%2015.png)