# File-System Interface

# File Concept

- File: a logical storage unit created by OS
    - v.s. physical storage unit in disk (sector, track)
    - 硬碟圖:
        
        ![Untitled](File-System%20Interface%20fe26425da2e6450ab45322885298de43/Untitled.png)
        
- File attributes
    - Identifier(ID): non-human-readable name, unique
    - Name
    - Type
    - Location
    - Size
    - Protection
    - Last-access time, Last-updated time

## File Operations

- File operations include
    - Creating a file
    - Writing a file
    - Reading a file
    - Repositioning within a file (i.e. file seek)
    - Deleting a file
    - Truncating a file
- Process: open-file table 儲存 process 開啟的檔案資訊
    - Process存取File時, 會將File存在該Process的open-file table
- OS: system-wide table 儲存 OS 系統所有開啟的檔案資訊
    - OS則是存在system-wide table, 這個table只有一個

## Open-File Tables

- Per-process table
    - 紀錄所有被這個 process 開啟的檔案
    - 每一個開啟檔案的指標
    - 檔案的存取權以及 Accounting information (CPU數量，真實使用的時間, 時間限制, 帳戶數量, 行程數量)
- System-wide table
    - open-file tables 所有檔案指向 system-wide table
    - 還會儲存獨立於 process 的資訊，如 disk location, access dates 還有 file size
    - Open count
        - 紀錄各個檔案被指向的次數，代表目前有多少 user 在存取該檔案
        - 若不同 process 存取同一個檔案，只會有一份檔案，確保資料的一致性

![Untitled](File-System%20Interface%20fe26425da2e6450ab45322885298de43/Untitled%201.png)

## Open File Attributes

- Open-file attributes (metadata,描述資料屬性（property）的資訊)
    - File pointer (per-process)
    - File open count (system table)
    - Disk location (system table)
    - Access rights (per-process)
- File types (副檔名)
    - .exe, .com, .obj, .cc, .mov, etc
    - Hint for OS to operate file in a reasonable way
        - 對OS來說, 副檔名就只是一個提示, 讓OS先嘗試用副檔名對應的應用程式開啟

# Access Method

## Sequential Access

- 最常用的方法
- 檔案可以視為 Array of bytes，Sequential access 在讀檔時是連續讀取的，因此需要的資訊就是目前的位置以及要讀取多少長度
- Read/write next (block)
- Reset: repositioning the file pointer to the beginning
- Skip/rewind n records
- 這是最快的讀取方式, 因為硬碟讀取的時候是透過讀寫頭去取得資料, 如果是連續的資料, 讀寫頭就不需要移動位置

![Untitled](File-System%20Interface%20fe26425da2e6450ab45322885298de43/Untitled%202.png)

## Direct (relative) Access

- 直接告訴系統要讀取檔案什麼位置的資料
- Access an element at an arbitrary position in a sequence
- File operations include the block # as parameter
- Often use **random access** to refer the access pattern from direct access
- 檔案由多個大小固定的區塊所組成，可直接將檔案指標移動到某個特定的區塊，並一次讀取整個區塊

![Untitled](File-System%20Interface%20fe26425da2e6450ab45322885298de43/Untitled%203.png)

## Index Access Methods

- Index Access是一種Random Access, 最常用在Database
- Index: 檔案區塊的指標
- 為了找到檔案中的 record:
    - 尋找 index file --> 找到指標
    - 利用指標去直接存取 record
- With a large file → index could become too large

![Untitled](File-System%20Interface%20fe26425da2e6450ab45322885298de43/Untitled%204.png)

- 硬體設備執行 random access 類型的操作效率較低，如果要支援 random access，通常要有 memory cache 才會夠快

# Directory Structure (目錄結構)

## Partition, Volume & Directory

- A **partition** 硬碟分割(formatted or raw, 有格式化或沒格式化)
    - raw partition (no file system): UNIX swap space, database
        - database不需要格式化的partition, 因為他不用file system
    - Formatted(格式化) partition with file system is called **volume**
        - volumn: 槽, 例如C槽/D槽
        - partition格式化有了檔案系統之後的磁碟區域才叫做volume
    - a partition can be a portion of a disk or group of multiple disks (distributed file system)
    - Some storage devices (e.g.: 3.5 磁碟片) does not and cannot have partition
- **Directories** are used by file system to store the information about the files in the partition

## File-System Organization

![Untitled](File-System%20Interface%20fe26425da2e6450ab45322885298de43/Untitled%205.png)

## Directory vs. File

- Directory: 為一群包含所有檔案資訊的節點
    - Directory structure 跟 files 都位於硬碟中
        
        ![Untitled](File-System%20Interface%20fe26425da2e6450ab45322885298de43/Untitled%206.png)
        

## Directory Operations

- Search for a file
- Create a file
- Delete a file
- List a directory(對現代檔案系統常常是效能瓶頸處，因為可能資料很多、檔案的樹狀結構很深)
- Rename a file
- Traverse the file system

## Single-Level Directory

![Untitled](File-System%20Interface%20fe26425da2e6450ab45322885298de43/Untitled%207.png)

- All files in one directory
    - 沒有folder
    - Filename has to be unique
    - Poor efficiency in locating a file as number of files increases

## Two-Level Directory

![Untitled](File-System%20Interface%20fe26425da2e6450ab45322885298de43/Untitled%208.png)

- a separate dir for each user
- path = user name + file name
- single-level dir problems still exists per user

## Tree-Structured Directory

- 主流的directory structure
- **Absolute path**: 絕對路徑, starting from the root
- **Relative path**: 相對路徑, starting from a directory

![Untitled](File-System%20Interface%20fe26425da2e6450ab45322885298de43/Untitled%209.png)

## Acyclic(無環)-Graph Directory

- Use links to share files or directories
    - UNIX-like: **symbolic link** (ln -s /spell/count /dict/count)
- A file can have multiple absolute paths
- When does a file actually get deleted?
    - deleting the link but not the file
    - deleting the file but leaves the link → dangling pointer (懸空指針)
    - deleting the file when reference counters is 0
    - 真正保險的作法是刪除 link 但不刪除檔案，當檔案的 counter 歸零時才把檔案刪除

![Untitled](File-System%20Interface%20fe26425da2e6450ab45322885298de43/Untitled%2010.png)

## General-Graph Directory

![Untitled](File-System%20Interface%20fe26425da2e6450ab45322885298de43/Untitled%2011.png)

- May contain cycles
    - Reference count does not work any more
    - E.g. self-referencing file

![Untitled](File-System%20Interface%20fe26425da2e6450ab45322885298de43/Untitled%2012.png)

- How can we deal with cycles?
    - Garbage collection
        - First pass traverses the entire graph and marks accessible files or directories
        - Second pass collect and free everything that is un-marked
        - Poor performance on millions of files …
    - Use cycle-detection algorithm when a link is created

# File System Mounting

- 將檔案系統與樹狀目錄結合就稱為mounting (掛載)，檔案系統必須要先mounting到目錄系統的某個目錄後，才能使用檔案系統。
- A file system must be mounted before it can be accessed
- **Mount point**: File System 被掛載的root path (根路徑)
- Mount timing:
    - boot time
        - 一開機就mount
        - 例如volumn, 需要先mount才可以load data到記憶體執行
    - automatically at run-time
        - 例如USB, 插上去之後自動安裝driver, 使用者不用做任何事就可以使用
    - manually at run-time

## File System Mounting Example

![Untitled](File-System%20Interface%20fe26425da2e6450ab45322885298de43/Untitled%2013.png)

![Untitled](File-System%20Interface%20fe26425da2e6450ab45322885298de43/Untitled%2014.png)

- OS會存mount table去存partition與mount directory間的mapping

# File Sharing

## File Sharing on Multiple Users

- Each user: (userID, groupID)
    - ID is associated with every ops/process/thread the user issues
- Each file has 3 sets of attributes
    - **owner** (擁有者權限)
    - **group** (所屬 group 的權限)
    - **others** (非擁有者也非 group 成員的權限)
- 擁有者 attributes 描述擁有者對檔案操作的權限
    - group/others 也有對應的 attributes
    - group/others 的權限由 owner 或 root 決定

![Untitled](File-System%20Interface%20fe26425da2e6450ab45322885298de43/Untitled%2015.png)

## Access-Control List

- 以一個list去存各個使用者對各個檔案的權限
- We can create an **access-control** **list** (ACL) for each user
    - check requested file access against ACL
    - problem: unlimited # of users
        - 每個使用者都個別紀錄權限的話, list會太長
- 3 classes of users → 3 ACL (**RWX**) for each file
    - 每個file都將使用者分成3個權限class
        - R: read
        - W: write
        - X: execute
    - 用3個bit (RWX) 描述不同層級 user 對檔案的執行權限, 所以佔用空間很小
        - owner (e.g. 7 = RWX = 111)
        - group (e.g. 6 = RWX = 110)
        - public (others) (e.g. 4 = RWX = 100)
        - 如下圖第一行中的 -rw-rw-r–，分別代表 owner-group-others 的權限
            
            ![Untitled](File-System%20Interface%20fe26425da2e6450ab45322885298de43/Untitled%2016.png)
            

# Protection

- 分成兩種: 使用者的權限資訊安全和避免資料遺失
- 檔案的擁有者/創建者必須能控制
    - 能執行哪些操作
    - 誰能執行這些操作
    - Access control list (ACL)
- Files should be kept from
    - physical damage (reliability): i.e. **RAID**
    - improper access (protection): i.e. password