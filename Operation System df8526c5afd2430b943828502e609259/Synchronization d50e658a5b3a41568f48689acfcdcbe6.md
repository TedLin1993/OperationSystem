# Synchronization

## Background

- Concurrent access to shared data may result in data inconsistency
    - concurrent有可能會導致data存取不一致而出錯
- Maintaining data consistency requires mechanism to ensure the orderly execution of cooperating processes
    - 確保process執行順序一致, 這樣data才不會出錯

### Consumer & Producer Problem

- Determine whether buffer is empty or full
    - Previously: use in, out position
    - Now: use count value

producer: counter++

![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled.png)

consumer: counter- -

![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%201.png)

- Assume counter is initially 5. One interleaving of statement is:

![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%202.png)

最後結果應該是5, 卻變成4

### Race Condition

- 定義: the situation where several processes access and manipulate shared data concurrently. The final value of the shared data depends upon which process finishes last
    - 多個process同時存取同一塊data, 導致data的value取決於哪個process最後執行
- To prevent race condition, concurrent processes must be synchronized
    - On a single-processor machine, we could disable interrupt or use non-preemptive CPU scheduling ⇒實務上不太會這麼做
    - Commonly described as critical section problem
        - 有race condition的code就稱做critical section

## The Critical-Section Problem

- Purpose: a protocol for processes to cooperate
- Problem description:
    - N processes are competing to use some shared data
    - Each process has a code segment, called critical section, in which the shared data is accessed
    - Ensure that when one process is executing in its critical section, no other process is allowed to execute in its critical section → mutually exclusive
- General code section structure
    - Only one process can be in a critical section ,在critical section的部分, 一次只會有一個人進去

![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%203.png)

### Critical Section Requirements

1. Mutual Exclusion: if process P is executing in its CS, no other processes can be executing in their CS
    - 在critical section的部分, 一次只會有一個人進去
2. Progress: if no process is executing in its CS and there exist some processes that wish to enter their CS, these processes cannot be postponed indefinitely
    - 只要critical section是空的, 那process就一定可以進去
3. Bounded Waiting: A bound must exist on the number of times that other processes are allowed to enter their CS after a process has made a request to enter its CS
    - 外面等待的時間必須是有限的, critical section不能永久都被占用

### Algorithm for Two Processes

- Only 2 processes, P0 and P1
- Shared variables
    - int turn; //initially turn = 0
    - turn = i ⇒ Pi can enter its critical section
    
    ![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%204.png)
    
    - turn 不是1就是0, 透過turn這個token, 在兩個process之間交互run
    - 但是他可能P0跑好幾次(但進不去critical section), 但P1只跑1次而已
    - 所以以Critical Section Requirements來說, Mutual Exclusion有達到, Bounded Waiting有達到, 但Progress沒有達到

### Peterson’s Solution for Two Processes

- 確保兩個process能夠輪流執行
- Shared variables
    - int turn; //initially turn = 0
    - turn = i ⇒ Pi can enter its critical section
    - boolean flag[2]; //initially flag [0] = flag [1] = false
    - flag [i] = true ⇒ Pi ready to enter its critical section
    
    ![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%205.png)
    
- 增加一個flag決定要不要進去, 若不想進去就不會卡到別人, 以符合Progress
- 進去之前將turn這個token交出來, 這樣才能夠輪流run不同的process

![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%206.png)

- Critical section越小越好, 盡量只做會有race condition的部分, 不然performance會很差

### Bakery Algorithm (n processes)

- Before enter its CS, each process receives a #
    - 抽號碼牌
- Holder of the smallest # enters CS
    - 號碼越小就越先進CS
- The numbering scheme always generates # in non-decreasing order; i.e., 1,2,3,3,4,5,5,5
- If processes Pi and Pj receive the same #, if i < j, then Pi is served first
    - 如果號碼相同, 讓PID較小的process先run
- Notation:
    - (a, b) < (c, d) if a < c or if a == c && b < d, 先比號碼再比PID

### Pthread Lock/Mutex Routines

- To use mutex, it must be declared as of type `pthread_mutex_t`and initialized with `pthread_mutex_init()`
- A mutex is destroyed with `pthread_mutex_destory()`
- A critical section can then be protected using `pthread_mutex_lock()` and `pthread_mutex_unlock()`
- Example:
    
    ![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%207.png)
    

### Condition Variables (CV)

- CV represent some condition that a thread can:
    - Wait on, until the condition occurs; or
    - Notify other waiting threads that the condition has occurred
- Three operations on condition variables:
    - wait() --- Block until another thread calls signal() or broadcast() on the CV
    - signal() --- Wake up one thread waiting on the CV
    - broadcast() --- Wake up all threads waiting on the CV
- Wait()
    1. Put the thread into sleep & releases the lock
    2. Waked up, but the thread is locked
    3. Re-acquire lock and resume execution
        
        ![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%208.png)
        
        - Pthread的coding style要求CV一定要放在critical section裡面, 也就是mutex locked住才能操作CV

### Hardware Support

- The CS problem occurs because the modification of a shared variable may be interrupted
    - 透過硬體支援讓Critical Section不會被interrupt
- If disable interrupts when in CS…
    - not feasible in multiprocessor machine
    - clock interrupts cannot fire in any machine
- HW support solution: atomic instructions
    - atomic: as one uninterruptible unit

### Mutual exclusion(Mutex) 互斥鎖

- 一種用於多執行緒編程中，防止兩條執行緒同時對同一公共資源（比如全域變數）進行讀寫的機制。該目的通過將代碼切片成一個一個的臨界區域（critical section）達成。
- 臨界區域指的是一塊對公共資源進行存取的代碼，並非一種機制或是演算法。一個程式、行程、執行緒可以擁有多個臨界區域，但是並不一定會應用互斥鎖。

![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%209.png)

## Semaphores

- A tool to generalize the synchronization problem (easier to solve, but no guarantee for correctness)
- More specifically…
    - a record of how many units of a particular resource are available
        - If #record = 1 → binary semaphore, mutex lock
        - If #record > 1 → counting semaphore
    - accessed only through 2 atomic ops: wait & signal
        - S = #record
        - Wait和signal都必須一次做完(atomic), 中間不能interrupt, 可以用hardware support或OS disable interrupt方式處理
- Spinlock implementation:
    - Semaphore is an integer variable
    - busy waiting, CPU會一直在while迴圈跑, 這樣不太好但很容易implement
        
        ![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%2010.png)
        

### POSIX Semaphore

- Semaphore is part of POSIX standard BUT it is not belonged to Pthread
    - It can be used with or without thread
    - 不一定用在thread, process之間也可以使用
- POSIX Semaphore routines:
    - sem_init(sem_t *sem, int pshared, unsigned int value)
    - sem_wait(sem_t *sem)
    - sem_post(sem_t *sem)
    - sem_getvalue(sem_t *sem, int *valptr)
    - sem_destory(sem_t *sem)
- Example:
    
    ![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%2011.png)
    

### Non-busy waiting Implementation

- Semaphore is data struct with a queue
    - may use any queuing strategy (FIFO, FILO, etc)
    
    ```cpp
    typedef struct {
    	int value; // init to 0 
    	struct process *L;
    	// “PCB” queue
    } semaphore;
    ```
    
    ![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%2012.png)
    
- wait() and signal()
    - use system calls: sleep() and wakeup()
    - must be executed **atomically**
        
        ![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%2013.png)
        
- 相較於busy waiting(spinlock), non-busy waiting 不會讓CPU一直在做白工, 但是這種作法會需要system calls, system call是非常消耗資源的行為, 所以若等待的時間很短的話, 還是用busy waiting較好

### Semaphore with Critical Section

![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%2014.png)

- sleep()和wakeup()不能包在CS裡面

### Cooperation Synchronization

- Process 2 的執行一定要在 Process 1之後
    
    ![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%2015.png)
    
- 複雜的例子

![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%2016.png)

## Deadlocks & Starvation

### **Deadlock**

Deadlock 意思是系統中存在一組 process 陷入互相等待對方所擁有的資源的情況，造成所有的 process 無法往下執行，使得 CPU 利用度大幅降低。

![https://2.bp.blogspot.com/-9H3Dx8fA9kk/VLarbHhQq-I/AAAAAAAAlGI/qvSZuPdfGU8/s400/%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7%2B2014-12-06%2B%E4%B8%8B%E5%8D%886.24.13-17.png](https://2.bp.blogspot.com/-9H3Dx8fA9kk/VLarbHhQq-I/AAAAAAAAlGI/qvSZuPdfGU8/s400/%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7%2B2014-12-06%2B%E4%B8%8B%E5%8D%886.24.13-17.png)

Deadlock 發生須符合以下四項充要條件 :

- Mutual exclusion：某些資源在同一個時間點最多只能被一個 process 使用
- Hold and wait：某 process 持有部分資源，並等待其他 process 正在持有的資源
- No preemption：process 不可以任意強奪其他 process 所持有的資源
- Circular waiting：系統中存在一組 processes P=P0,P1,…,Pn，其中 P0 等待 P1 所持有的資源 ...  Pn等待 P0 所持有的資源，形成循環式等待。(因此，deadlock不會存在於single process環境中)

![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%2017.png)

Deadlock會導致Starvation

## Classical Synchronization Problems

### Listing & Purpose

- Purpose: used for testing newly proposed synchronization scheme
- Bounded-Buffer (Producer-Consumer) Problem
- Reader-Writers Problem
- Dining-Philosopher Problem

### Bounded-Buffer Problem

- A pool of n buffers, each capable of holding one item
- Producer: (buffer++)
    - grab an empty buffer
    - place an item into the buffer
    - waits if no empty buffer is available
- Consumer: (buffer-- )
    - grab a buffer and retracts the item
    - place the buffer back to the free pool
    - waits if all buffers are empty

### Readers-Writers Problem

通常對同一個檔案或同一份資料，不同的程序(process)會想對它做讀或寫的動作，

檔案可以接受同一時間被多個程序讀取，但寫入檔案一次只能由一個程序作，而且寫入時其他程序不能嘗試讀寫。

- A set of shared data objects
- A group of processes
    - reader processes (read shared objects)
    - writer processes (update shared objects)
    - a writer process has exclusive access to a shared object
- Different variations involving priority
    - first RW problem: no reader will be kept waiting unless a writer is updating a shared object
        - 如果檔案不是正在被寫入, reader都可以同時讀取
    - second RW problem: once a writer is ready, it performs the updates as soon as the shared object is released
        - writer如果要寫入, 他應該要ASAP得到檔案的存取權限
        - writer has higher priority than reader
        - once a writer is ready, no new reader may start reading
- First Reader-Writer Algorithm

![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%2018.png)

### Dining-Philosophers Problem

- 5 persons sitting on 5 chairs with 5 chopsticks

- A person is either thinking or eating
    - thinking: no interaction with the rest 4 persons
    - eating: need 2 chopsticks at hand
    - a person picks up 1 chopstick at a time
    - done eating: put down both chopsticks

![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%2019.png)

- deadlock problem
    - one chopstick as one semaphore
- starvation problem

## Monitor

### Motivation

- Although semaphores provide a convenient and effective synchronization mechanism, its correctness is depending on the programmer
    - All processes access a shared data object must execute wait() and signal() in the right order and right place
    - This may not be true because honest programming error or uncooperative programmer

### Monitor --- A high-level language construct

- 高階語言的synchronization tool, 以OOP的概念更方便操作
    - 將CV當作class內的物件, 若要存取CV就必須使用class所提供的method
    - 同時只可以有一個thread去執行method, 其他thread必須在queue排隊
- The representation of a monitor type consists of
    - declarations of variables whose values define the state of an instance of the type
    - Procedures/functions that implement operations on the type
- The monitor type is similar to a class in O.O. language
    - A procedure within a monitor can access only local variables and the formal parameters
    - The local variables of a monitor can be used only by the local procedures
- But, the monitor ensures that only one process at a time can be **active** within the monitor
- Similar idea is incorporated to many prog. language:
    - concurrent pascal, C# and Java
- High-level synchronization construct that allows the safe sharing of an abstract data type among concurrent processes

- Syntax
    
    ```cpp
    monitor monitor-name {
    	// shared variable declarations
    	procedure body P1 (…) {
    	 . . . 
    	}
    	procedure body P2 (…) {
    	 . . . 
    	}
    	procedure body Pn (…) {
    	 . . . 
    	}
    	initialization code {
    	}
    }
    ```
    

![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%2020.png)

### Monitor Condition Variables

- To allow a process to wait within the monitor, a condition variable must be declared, as condition x, y;
- Condition variable can only be used with the operations `wait()` and`signal()`
    - x.wait():
        - means that the process invoking this operation is suspended until another process invokes
        - 將thread enqueue到一個waiting queue
    - x.signal():
        - resumes exactly one suspended process. If no process is suspended, then the signal operation has no effect
            - 從waiting queue dequeue一個thread出來
        - In contrast, signal always change the state of a semaphore
            - semaphore的signal會+1, 而wait會-1

![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%2021.png)

### Dining Philosophers Example

- 將題目轉成monitor
    
    ![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%2022.png)
    
    - 只有5個人和5根筷子, 所以array length = 5
    - 須確保state不會產生race condition
    
    ![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%2023.png)
    
    - pickup(): 確認是否能拿到2根筷子, 若沒有就wait
    - putdown(): 確認左右兩邊是否能拿到兩根筷子
    - test(): 左右兩邊都沒有筷子, 且自己想吃東西 → 搶走左右兩邊的筷子來吃東西

## Atomic Transactions

### System Model

- **Transaction**: a collection of instructions (or instructions) that performs a single logic function
- **Atomic Transaction**: operations happen as a single logical unit of work, in its entirely, or not at all
    - transaction要馬整個作完, 要馬完全不做(all or nothing)
- Atomic transaction is particular a concern for database system
    - Strong interest to use DB techniques in OS

### File I/O Example

- Transaction is a series of read and write operations
- Terminated by commit (transaction successful) or abort (transaction failed) operation
- Aborted transaction must be rolled back to undo any changes it performed
    - It is part of the responsibility of the system to ensure this property

### Log-Based Recovery

- 有log才能夠rolled back
- Record to stable storage information about all modifications by a transaction
    - Stable storage: never lost its stored data
        - log存在stable storage確保系統crash的情況下, log仍能保留
- **Write-ahead logging**: Each log record describes single transaction write operation
    - Transaction name
    - Data item name
    - Old & new values
    - Special events: <Ti starts>, <Ti commits>
- Log is used to reconstruct the state of the data items modified by the transactions
-  Use undo (Ti), redo(Ti) to recover data
- 問題: log沒有時間概念, 從系統啟動到系統關掉或crash, 這樣如果要roll backed會從頭開始做

### Checkpoints

- 讓系統rolled back時可從checkpoints開始做
- When failure occurs, must consult the log to determine which transactions must be re-done
    - Searching process is time consuming
    - Redone may not be necessary for all transactions
- Use checkpoints to reduce the above overhead:
    - Output all log records to stable storage
    - Output all modified data to stable storage
    - Output a log record <checkpoint> to stable storage