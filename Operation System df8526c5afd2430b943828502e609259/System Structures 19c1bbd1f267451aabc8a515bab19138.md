# System Structures

![Untitled](System%20Structures%2019c1bbd1f267451aabc8a515bab19138/Untitled.png)

UI透過system calls去呼叫OS的service

Services中的各個service會在後面章節介紹, 這裡介紹User Interfaces與Communication

## User Interface

主要分成CLI與GUI

### CLI(Command Line Interface)

- Fetches a command from user and executes it
- Shell: Command-line interpreter (CSHELL, **BASH**), CLI的Interface和CLI溝通的介面
    - Adjusted according to user behavior and preference

### GUI (Graphic User Interface)

- 圖形化介面, 例如windows

## Communication Models

兩種方法: message passing 或 shared memory

![Untitled](System%20Structures%2019c1bbd1f267451aabc8a515bab19138/Untitled%201.png)

### message passing

也就是memory copy, process A 必須透過kernel才能複製檔案到process B, 這是為了protect, 只有OS才有權利讀/寫process的資料

### shared momory

先透過system call向OS要出shared memory這個空間, 讓兩個process可共用這塊memory

如果是multi thread的話, OS default就會create shared memory了

## System Call vs API

![Untitled](System%20Structures%2019c1bbd1f267451aabc8a515bab19138/Untitled%202.png)

### system call

- 透過software interrupt 從user mode進到kernel mode
- 為了效能通常都是用assembly-language寫的

### API

- 使用者幾乎都是透過API去呼叫system call
- Commonly implemented by language libraries, e.g., C Library
- API不一定需要system call, 例如Math API function

## Common APIs

### Win32 API

- for Windows
- 64 bit version也叫做win32 API

### POSIX API

- POSIX: Portable Operating System Interface for Unix, 整合非windows陣營來對抗windows
- for POSIX-based systems, 包含UNIX, Linux, Mac OS

### Java API

- for Java virtual machine(JVM)

## System Structure

### Simple OS Architecture

![Untitled](System%20Structures%2019c1bbd1f267451aabc8a515bab19138/Untitled%203.png)

只有將OS和driver切開(only 2 levels of code), OS裡面完全沒有區分架構

- 不安全(容易被鑽漏洞)
- 難以維護

### Layered OS Architecture

![Untitled](System%20Structures%2019c1bbd1f267451aabc8a515bab19138/Untitled%204.png)

- 上層的layer才可以call下層layer的function, 所以memory可以call disk, 但disk不能call memory
- 最上層user interface如果要call hardware, 他需要從最上層一路call到最底層
- 容易debug, 也容易maintain
- 效能較差, 很難定義layer上下關係

## Microkernel OS

![Untitled](System%20Structures%2019c1bbd1f267451aabc8a515bab19138/Untitled%205.png)

- Kernel很小, 只負責和Process溝通
- 現在放在kernel的service都往上推到user space
- 開始有module的概念
- 優點: 非常好維護, 非常好延伸
- 缺點: 效能比Layered OS更差, 因為每一次溝通都是system call

### Modular OS Architecture

![Untitled](System%20Structures%2019c1bbd1f267451aabc8a515bab19138/Untitled%206.png)

保存了Microkernel的module這個部分, 但是將service放進kernel

### Virtual Machine

![Untitled](System%20Structures%2019c1bbd1f267451aabc8a515bab19138/Untitled%207.png)

virtual machine manager就是kernel, 而VM都是user space

好處:

- 各個VM彼此獨立, 對資源有很好的protection
- 可以兼容(compatibility) application所需的環境
- 可以develop OS, 或測試hack OS
- cloud computing

### Java Virtual Machine

![Untitled](System%20Structures%2019c1bbd1f267451aabc8a515bab19138/Untitled%208.png)

Java本身就是一個VM, 他會在自己的JVM編譯出bytecodes, 在translate到host system

因為是用VM, 儘管有很多優化, 效能還是會較差