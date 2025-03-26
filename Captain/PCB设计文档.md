# 进程控制块(PCB)设计文档

## 1. PCB概述

进程控制块(Process Control Block, PCB)是操作系统中表示进程的核心数据结构，包含了操作系统管理进程所需的所有信息。每个进程在系统中都有唯一对应的PCB。

## 2. PCB数据结构

### 2.1 基本结构

```java
public class PCB {
    // 基本标识信息
    private final int pid;                      // 进程ID
    private String processName;                 // 进程名称
    private int parentPid;                      // 父进程ID
    
    // 状态信息
    private ProcessState state;                 // 进程状态
    private int priority;                       // 进程优先级
    
    // 调度信息
    private long creationTime;                  // 创建时间
    private long startTime;                     // 开始执行时间
    private long totalCpuTime;                  // 已使用CPU时间
    private long lastScheduledTime;             // 最近被调度时间
    private int timeSlice;                      // 分配的时间片
    
    // 程序执行信息
    private long programCounter;                // 程序计数器
    private List<Integer> registers;            // 寄存器值
    
    // 内存管理信息
    private MemoryInfo memoryInfo;              // 内存信息
    
    // 资源管理信息
    private List<Resource> allocatedResources;  // 已分配资源列表
    private List<Resource> requestedResources;  // 请求的资源列表
    
    // 进程间通信
    private List<Message> messageQueue;         // 消息队列
    
    // 文件管理
    private List<FileHandle> openFiles;         // 打开的文件列表
    
    // 统计信息
    private long waitingTime;                   // 等待时间
    private long responseTime;                  // 响应时间
    private long turnaroundTime;                // 周转时间
    
    // 构造函数、getter和setter方法
    // ...
}
```

### 2.2 进程状态枚举

```java
public enum ProcessState {
    NEW,        // 新建状态：进程刚被创建，尚未加入就绪队列
    READY,      // 就绪状态：等待被调度
    RUNNING,    // 运行状态：正在执行
    BLOCKED,    // 阻塞状态：等待某个事件发生
    TERMINATED  // 终止状态：进程执行结束
}
```

### 2.3 内存信息类

```java
public class MemoryInfo {
    private int baseAddress;           // 内存基址
    private int memorySize;            // 占用内存大小
    private int pageTableAddress;      // 页表地址(分页式内存管理)
    private List<MemorySegment> segments; // 内存段列表(分段式内存管理)
    
    // 构造函数、getter和setter方法
    // ...
}
```

### 2.4 资源类

```java
public class Resource {
    private int resourceId;            // 资源ID
    private String resourceName;       // 资源名称
    private ResourceType type;         // 资源类型
    private int quantity;              // 资源数量
    
    // 构造函数、getter和setter方法
    // ...
}

public enum ResourceType {
    CPU,                    // CPU时间
    MEMORY,                 // 内存
    FILE,                   // 文件
    IO_DEVICE,              // I/O设备
    SEMAPHORE,              // 信号量
    OTHER                   // 其他资源
}
```

## 3. PCB操作接口

### 3.1 PCB管理器接口

```java
public interface PCBManager {
    // 创建新PCB
    PCB createPCB(String processName, int priority);
    
    // 根据PID查找PCB
    PCB findPCB(int pid);
    
    // 更新PCB状态
    boolean updatePCBState(int pid, ProcessState newState);
    
    // 删除PCB
    boolean deletePCB(int pid);
    
    // 获取所有PCB
    List<PCB> getAllPCBs();
    
    // 获取特定状态的PCB
    List<PCB> getPCBsByState(ProcessState state);
}
```

### 3.2 PCB管理器实现

```java
public class PCBManagerImpl implements PCBManager {
    // 存储所有PCB的映射表
    private Map<Integer, PCB> pcbTable;
    
    // 按进程状态分类的PCB列表
    private Map<ProcessState, List<PCB>> stateBasedPCBs;
    
    // 进程ID生成器
    private AtomicInteger pidGenerator;
    
    // 构造函数
    public PCBManagerImpl() {
        pcbTable = new ConcurrentHashMap<>();
        stateBasedPCBs = new EnumMap<>(ProcessState.class);
        for (ProcessState state : ProcessState.values()) {
            stateBasedPCBs.put(state, new CopyOnWriteArrayList<>());
        }
        pidGenerator = new AtomicInteger(1); // 从1开始分配PID
    }
    
    // 接口方法实现
    // ...
}
```

## 4. PCB生命周期管理

### 4.1 PCB创建

PCB创建流程包括：
1. 生成唯一PID
2. 初始化PCB各项属性
3. 设置进程为NEW状态
4. 分配必要的系统资源
5. 将PCB添加到系统PCB表中

### 4.2 PCB状态更新

进程状态转换需要同时更新以下信息：
1. 修改PCB中的状态字段
2. 更新状态相关的时间统计信息
3. 在相应的进程队列间移动PCB
4. 记录状态变化日志（便于调试和分析）

### 4.3 PCB销毁

PCB销毁流程包括：
1. 将进程状态设置为TERMINATED
2. 释放进程占用的所有资源
3. 更新进程统计信息
4. 通知相关子系统（如调度器）
5. 从系统PCB表中移除

## 5. PCB队列管理

系统中维护以下主要PCB队列：

1. **就绪队列**：包含所有READY状态的进程，按优先级组织
2. **阻塞队列**：包含所有BLOCKED状态的进程，按阻塞原因分类
3. **新建进程队列**：包含NEW状态的进程
4. **已终止进程队列**：包含TERMINATED状态的进程，等待资源回收

队列数据结构设计：

```java
public class ProcessQueue<T extends PCB> {
    private PriorityQueue<T> queue;
    private ReentrantReadWriteLock lock;
    
    public ProcessQueue(Comparator<T> comparator) {
        queue = new PriorityQueue<>(comparator);
        lock = new ReentrantReadWriteLock();
    }
    
    // 添加进程到队列
    public void enqueue(T pcb) {
        lock.writeLock().lock();
        try {
            queue.offer(pcb);
        } finally {
            lock.writeLock().unlock();
        }
    }
    
    // 从队列中取出进程
    public T dequeue() {
        lock.writeLock().lock();
        try {
            return queue.poll();
        } finally {
            lock.writeLock().unlock();
        }
    }
    
    // 查看队头进程，不移除
    public T peek() {
        lock.readLock().lock();
        try {
            return queue.peek();
        } finally {
            lock.readLock().unlock();
        }
    }
    
    // 其他队列操作方法
    // ...
}
```

## 6. 设计考量

### 6.1 线程安全

由于PCB是系统中共享的关键数据结构，必须确保线程安全：

1. 使用`ConcurrentHashMap`存储PCB表，确保高并发访问性能
2. 进程队列操作使用读写锁，优化并发读取性能
3. 通过原子操作更新进程状态，避免竞态条件

### 6.2 性能优化

1. 使用惰性初始化减少内存占用
2. 关键路径上避免不必要的对象创建
3. 为高频查询构建索引（如按状态、优先级的索引）

### 6.3 可扩展性

1. 使用组合而非继承设计PCB
2. 核心PCB保持精简，通过扩展属性支持特殊需求
3. 提供观察者模式接口，允许其他模块监听PCB状态变化 