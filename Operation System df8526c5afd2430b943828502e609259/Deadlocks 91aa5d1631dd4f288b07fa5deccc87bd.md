# Deadlocks

[Chap7  Deadlocks ＿.pdf](Deadlocks%2091aa5d1631dd4f288b07fa5deccc87bd/Chap7__Deadlocks_%EF%BC%BF.pdf)

## **Deadlock Problem**

Deadlock 意思是系統中存在一組 process 陷入互相等待對方所擁有的資源的情況，造成所有的 process 無法往下執行，使得 CPU 利用度大幅降低。

![https://2.bp.blogspot.com/-9H3Dx8fA9kk/VLarbHhQq-I/AAAAAAAAlGI/qvSZuPdfGU8/s400/%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7%2B2014-12-06%2B%E4%B8%8B%E5%8D%886.24.13-17.png](https://2.bp.blogspot.com/-9H3Dx8fA9kk/VLarbHhQq-I/AAAAAAAAlGI/qvSZuPdfGU8/s400/%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7%2B2014-12-06%2B%E4%B8%8B%E5%8D%886.24.13-17.png)

Deadlock 發生須符合以下四項充要條件 :

- Mutual exclusion：某些資源在同一個時間點最多只能被一個 process 使用
- Hold and wait：某 process 持有(hold)部分資源，並等待(wait)其他 process 正在持有的資源
- No preemption：process 不可以強奪其他 process 所持有的資源
- Circular waiting：系統中存在一組 processes P=P0,P1,…,Pn，其中 P0 等待 P1 所持有的資源 ...  Pn等待 P0 所持有的資源，形成循環式等待。(因此，deadlock不會存在於single process環境中)

![Untitled](Synchronization%20d50e658a5b3a41568f48689acfcdcbe6/Untitled%2017.png)

## System Model

- Resources types R1, R2, …, Rm
    - E.g. CPU, memory pages, I/O devices
- Each resource type Ri has Wi instances
    - E.g. a computer has 2 CPUs
- Each process utilizes a resource as follows:
    - Request → use → release

## Deadlock Characterization

### Resource-Allocation Graph

• 3 processes, P1 ~ P3

• 4 resources, R1 ~ R4

• R1 and R3 each has one instance

• R2 has two instances

• R4 has three instances

• Request edges:

• P1→R1: P1 requests R1

• Assignment edges:

• R2→P1: One instance of R2 is allocated to P1

→ P1 is hold on an instance of R2 and waiting for an instance of R1

![Untitled](Deadlocks%2091aa5d1631dd4f288b07fa5deccc87bd/Untitled.png)

### Deadlock Detection

- If graph contains no cycle → no deadlock
    - Circular wait cannot be held
- If graph contains a cycle:
    - if one instance per resource type → deadlock
    - if multiple instances per resource type → **possibility** of deadlock

## Handling Deadlocks

三大處理deadlocks的方法: 

1. Ensure the system will **never** enter a **deadlock state**
    - deadlock **prevention**: ensure that at least one of the 4 necessary conditions cannot hold
        - 設計系統時避免deadlocks
    - deadlock **avoidance**: dynamically examines the resource-allocation state before allocation
        - 分配資源前，Run-time 動態地檢查目前資源分配的狀況
2. Allow to enter a deadlock state and **then recover**
    - deadlock **detection**
    - deadlock **recovery**
3. Ignore the problem and pretend that deadlocks never occur in the system
    - 假裝deadlocks不會發生, 為了去避免deadlocks要花費太多資源了
    - 大多數作業系統都是這樣處理, 包含 Linux and Windows

## Deadlock Prevention

- Mutual exclusion (ME): do not require ME on sharable resources
    - read-only 的資料不需要 mutual exclusion
    - Some resources are not shareable, however (e.g. printer)
- Hold & Wait:
    - When a process requests a resource, it does not hold any resource
        - 如果需要wait, 就全部資源都release掉, 從頭來過
    - Pre-allocate all resources before executing
    - resource utilization is low; starvation is possible
- No preemption
    - When a process is waiting on a resource, all its holding resources are preempted
        - e.g. P1 request R1, which is allocated to P2, which in turn is waiting on R2. (P1 → R1 → P2 → R2)
        - R1 can be preempted and reallocated to P1
        - 要存Resource的state, 因此增加cost
    - 適合用在容易紀錄及恢復狀態的資源
        - e.g. CPU registers & memory
    - It cannot easily be applied to other resources
        - e.g. printers & tape drives
- Circular wait
    - impose a total ordering of all resources types
        - 取resource只有一個方向性
    - a process requests resources in an increasing order
        - Let R={R0, R1, …, RN} be the set of resource types
        - When request Rk, should release all Ri, i ≥ k
    - Example:
        - F(tape drive) = 1, F(disk drive) = 5, F(printer) = 12
        - A process must request tape and disk drive before printer
            - tape → disk → printer
    - proof: counter-example does not exist
        
        ![Untitled](Deadlocks%2091aa5d1631dd4f288b07fa5deccc87bd/Untitled%201.png)
        
        - `Pn(Rn) → R0` 這個condition不會成立, Pn不能去要R0
    - 缺點: 排序越後面的resource越難拿到

## Deadlock Avoidance

若無法做 Preventon，則必須在 run time 期間進行動態檢查

### Avoidance Algorithms

- Single instance of a resource type
    - 用圖來看是否有可能deadlocks
    - **resource-allocation graph (RAG)** algorithm based on circle detection
- Multiple instances of a resource type
    - **banker’s algorithm** based on safe sequence detection

### Resource-Allocation Graph (RAG) Algorithm

- **Request edge**: Pi→Rj
    - Pi 在等待Rj 的資源
- **Assignment edge**: Rj→Pi
    - Rj 被 Pi所持有
- **Claim edge**: Pi→Rj
    - process Pi may request Rj
    in the future
    - worst case會成為request edge

![Untitled](Deadlocks%2091aa5d1631dd4f288b07fa5deccc87bd/Untitled%202.png)

- Claim edge converts to request edge when a resource is requested by process
- Assignment edge converts to a claim edge when a resource is released by a process

- Resources must be claimed a priori in the system
- Grant a request only if NO cycle created
- Check for safety using a cycle-detection algorithm, O($n^2$)
- Example: R2 cannot be allocated to P2

![Untitled](Deadlocks%2091aa5d1631dd4f288b07fa5deccc87bd/Untitled%203.png)

### Safe State

- safe state: a system is in a safe state if there exists a sequence of allocations to satisfy requests by all processes
    - This sequence of allocations is called safe sequence
    - 只要能夠依序執行process就不會有deadlock
- safe state → no deadlock
- unsafe state → possibility of deadlock
- deadlock avoidance → ensure that a system never enters an unsafe state

![Untitled](Deadlocks%2091aa5d1631dd4f288b07fa5deccc87bd/Untitled%204.png)

- A request is only granted if the allocation leaves the system in a safe state

### Banker's Algorithm

- 找出一個process sequence讓worst case仍能依序將各process都成功執行完
- Use for multiple instances of each resource type
- Banker algorithm:
    - Use a general safety algorithm to pre-determine if any safe sequence exists after allocation
    - Only proceed the allocation if safe sequence exists
- Safety algorithm:
    
    1. Assume processes need maximum resources
    2. Find a process that can be satisfied by free resources
    3. Free the resource usage of the process
    4. Repeat to step 2 until all processes are satisfied
    

## Deadlock Detection

- Single instance of each resource type
    - convert request/assignment edges into wait-for graph
        - 可以濾掉Resource, 直接看Process間的關係來判斷是否有circle
    - deadlock exists if there is a cycle in the wait-for graph
    
    ![Untitled](Deadlocks%2091aa5d1631dd4f288b07fa5deccc87bd/Untitled%205.png)
    
    ## Deadlock Recovery
    
    - **Process termination**
        - abort all deadlocked processes, process有deadlock就kill掉
        - abort 1 process at a time until the deadlock cycle is eliminated
            - which process should we abort first?
    - **Resource preemption**
        - select a victim: which one to preempt?
        - rollback: partial rollback or total rollback?
        - starvation: can the same process be preempted always?