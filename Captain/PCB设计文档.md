# 进程控制块(PCB)设计文档

## 1. PCB概述

进程控制块(Process Control Block, PCB)是操作系统中表示进程的核心数据结构，包含了操作系统管理进程所需的所有信息。每个进程在系统中都有唯一对应的PCB。

### 1.1 PCB在操作系统中的角色

PCB是操作系统管理进程的关键数据结构，主要用于：

1. **进程标识**：通过唯一的进程ID(PID)标识系统中的每个进程
2. **状态管理**：记录进程当前状态(新建、就绪、运行、阻塞、终止)
3. **CPU现场保存**：保存进程运行时的寄存器值和程序计数器
4. **资源记录**：跟踪进程占用的系统资源
5. **调度管理**：记录进程优先级和调度相关信息
6. **进程统计**：收集进程运行的统计数据(CPU使用时间、等待时间等)

### 1.2 PCB在系统中的生命周期

```
    +-------+     +---------+     +---------+     +------------+
    | 创建  | --> |  就绪   | --> |  运行   | --> |   终止     |
    +-------+     +---------+     +---------+     +------------+
                      ^               |
                      |               v
                      |          +---------+
                      +----------+  阻塞   |
                                 +---------+
```

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

### 2.5 消息类

```java
public class Message {
    private int messageId;             // 消息ID
    private int senderPid;             // 发送进程ID
    private int receiverPid;           // 接收进程ID
    private MessageType type;          // 消息类型
    private byte[] data;               // 消息内容
    private long timestamp;            // 时间戳
    
    // 构造函数、getter和setter方法
    // ...
}

public enum MessageType {
    SYSTEM,                 // 系统消息
    USER,                   // 用户消息
    SIGNAL,                 // 信号
    DATA                    // 数据传输
}
```

### 2.6 文件句柄类

```java
public class FileHandle {
    private int fileDescriptor;        // 文件描述符
    private String fileName;           // 文件名
    private FileMode mode;             // 打开模式
    private long position;             // 当前位置
    private boolean isOpen;            // 是否打开
    
    // 构造函数、getter和setter方法
    // ...
}

public enum FileMode {
    READ,                   // 只读
    WRITE,                  // 只写
    READ_WRITE,             // 读写
    APPEND                  // 追加
}
```

### 2.7 PCB类图

```
             +---------------+
             |     PCB       |
             +---------------+
             | -pid: int     |
             | -state: State |
             | -priority: int|
             +---------------+
             | +methods()    |
             +---------------+
                   ▲
       +----+------+------+----+
       |    |      |      |    |
+------------+ +-------+  +-------+ +-------+
|ProcessInfo | |Memory |  |Resource| |Message|
+------------+ +-------+  +-------+ +-------+
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
    
    // 获取进程优先级
    int getPriority(int pid);
    
    // 设置进程优先级
    boolean setPriority(int pid, int priority);
    
    // 获取进程资源
    List<Resource> getProcessResources(int pid);
    
    // 分配资源给进程
    boolean allocateResource(int pid, Resource resource);
    
    // 从进程释放资源
    boolean releaseResource(int pid, int resourceId);
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
    
    // createPCB方法实现
    @Override
    public PCB createPCB(String processName, int priority) {
        // 生成唯一PID
        int pid = pidGenerator.getAndIncrement();
        
        // 创建新PCB
        PCB pcb = new PCB();
        pcb.setPid(pid);
        pcb.setProcessName(processName);
        pcb.setState(ProcessState.NEW);
        pcb.setPriority(priority);
        pcb.setCreationTime(System.currentTimeMillis());
        
        // 添加到PCB表
        pcbTable.put(pid, pcb);
        stateBasedPCBs.get(ProcessState.NEW).add(pcb);
        
        return pcb;
    }
    
    // 其他接口方法实现
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

详细时序图：
```
  用户   ProcessManager   PCBManager   MemoryManager   
   |          |              |              |
   |--创建进程->|              |              |
   |          |--创建PCB----->|              |
   |          |              |--生成PID-----|
   |          |              |--初始化PCB---|
   |          |              |<-PCB创建完成-|
   |          |--申请内存----------->|       |
   |          |<-内存分配完成--------|       |
   |          |--设置内存信息->|              |
   |          |--更新PCB状态->|              |
   |<-进程创建成功|              |              |
```

### 4.2 PCB状态更新

进程状态转换需要同时更新以下信息：
1. 修改PCB中的状态字段
2. 更新状态相关的时间统计信息
3. 在相应的进程队列间移动PCB
4. 记录状态变化日志（便于调试和分析）

状态转换示意图：
```
      +-------------+          调度
      |    就绪     |------------------------+
      |   READY     |                        |
      +-------------+                        V
           ^                           +-------------+
           |                           |    运行     |
      I/O完成                          |   RUNNING   |
     资源获取                          +-------------+
           |                                 |
           |                                 | I/O请求
      +-------------+          CPU时间片到期 | 等待资源
      |    阻塞     |<------------------------+
      |   BLOCKED   |
      +-------------+
```

### 4.3 PCB销毁

PCB销毁流程包括：
1. 将进程状态设置为TERMINATED
2. 释放进程占用的所有资源
3. 更新进程统计信息
4. 通知相关子系统（如调度器）
5. 从系统PCB表中移除

详细时序图：
```
  用户   ProcessManager   PCBManager   MemoryManager   FileManager
   |          |              |              |              |
   |--终止进程->|              |              |              |
   |          |--查找PCB----->|              |              |
   |          |<-返回PCB------|              |              |
   |          |--释放内存----------->|       |              |
   |          |<-内存释放完成--------|       |              |
   |          |--关闭文件------------------------->|        |
   |          |<-文件关闭完成----------------------|        |
   |          |--设置终止状态->|              |              |
   |          |--更新统计信息->|              |              |
   |          |--从队列移除-->|              |              |
   |<-进程终止成功|              |              |              |
```

## 5. PCB队列管理

### 5.1 系统队列设计

系统中维护以下主要PCB队列：

1. **就绪队列**：包含所有READY状态的进程，按优先级组织
2. **阻塞队列**：包含所有BLOCKED状态的进程，按阻塞原因分类
3. **新建进程队列**：包含NEW状态的进程
4. **已终止进程队列**：包含TERMINATED状态的进程，等待资源回收

多级队列设计：
```
就绪队列:
  [高优先级] → 进程A → 进程B
  [中优先级] → 进程C → 进程D → 进程E
  [低优先级] → 进程F → 进程G

阻塞队列:
  [等待I/O] → 进程H → 进程I
  [等待资源] → 进程J → 进程K
  [等待事件] → 进程L
```

### 5.2 队列数据结构设计

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
    
    // 查找特定PID的进程
    public T findByPid(int pid) {
        lock.readLock().lock();
        try {
            for (T pcb : queue) {
                if (pcb.getPid() == pid) {
                    return pcb;
                }
            }
            return null;
        } finally {
            lock.readLock().unlock();
        }
    }
    
    // 从队列中移除特定进程
    public boolean remove(T pcb) {
        lock.writeLock().lock();
        try {
            return queue.remove(pcb);
        } finally {
            lock.writeLock().unlock();
        }
    }
    
    // 获取队列大小
    public int size() {
        lock.readLock().lock();
        try {
            return queue.size();
        } finally {
            lock.readLock().unlock();
        }
    }
    
    // 检查队列是否为空
    public boolean isEmpty() {
        lock.readLock().lock();
        try {
            return queue.isEmpty();
        } finally {
            lock.readLock().unlock();
        }
    }
}
```

## 6. 设计考量

### 6.1 线程安全

由于PCB是系统中共享的关键数据结构，必须确保线程安全：

1. 使用`ConcurrentHashMap`存储PCB表，确保高并发访问性能
2. 进程队列操作使用读写锁，优化并发读取性能
3. 通过原子操作更新进程状态，避免竞态条件
4. 使用不可变对象减少同步需求
5. 采用事务性更新方式处理复杂的状态变更

线程安全策略：
```
操作类型        |  并发控制策略
----------------|------------------
读取PCB信息     |  无锁/读锁
修改单个PCB属性 |  AtomicReference/CAS
状态转换        |  写锁/事务
队列操作        |  读写锁分离
资源分配/释放   |  2PL(两阶段锁)
```

### 6.2 性能优化

1. 使用惰性初始化减少内存占用
2. 关键路径上避免不必要的对象创建
3. 为高频查询构建索引（如按状态、优先级的索引）
4. 使用缓存减少重复计算
5. 采用批处理模式进行批量PCB更新

性能关键点：
```
操作               |  性能考量
-------------------|------------------
进程创建           |  减少锁竞争，预分配资源
进程调度           |  优化队列查找，使用优先级堆
状态转换           |  最小化锁范围，减少上下文切换开销
PCB查找            |  哈希表查找，O(1)复杂度
资源分配/释放      |  批量处理，减少锁争用
```

### 6.3 可扩展性

1. 使用组合而非继承设计PCB
2. 核心PCB保持精简，通过扩展属性支持特殊需求
3. 提供观察者模式接口，允许其他模块监听PCB状态变化
4. 采用插件式设计支持自定义PCB属性和行为
5. 使用工厂模式创建不同类型的PCB

模块扩展接口：
```java
// PCB扩展接口
public interface PCBExtension {
    String getName();
    void initialize(PCB pcb);
    void onStateChange(PCB pcb, ProcessState oldState, ProcessState newState);
    void onResourceChange(PCB pcb, Resource resource, boolean isAllocation);
    void cleanup(PCB pcb);
}

// PCB观察者接口
public interface PCBObserver {
    void onPCBCreated(PCB pcb);
    void onPCBStateChanged(PCB pcb, ProcessState oldState, ProcessState newState);
    void onPCBTerminated(PCB pcb);
}
```

## 7. PCB与其他模块的交互

### 7.1 PCB与内存管理模块

PCB需要与内存管理模块交互，以管理进程的内存空间：

```java
// 内存管理接口
public interface MemoryManager {
    // 分配内存给进程
    MemoryAllocation allocateMemory(int pid, int size);
    
    // 释放进程占用的内存
    void freeMemory(int pid);
    
    // 获取进程内存使用情况
    MemoryInfo getMemoryInfo(int pid);
    
    // 检查内存访问权限
    boolean checkMemoryAccess(int pid, int address, int size, AccessType type);
    
    // 页面置换
    void swapPages(int pid);
}

// PCB-内存管理交互流程
public class PCBMemoryInteraction {
    private PCBManager pcbManager;
    private MemoryManager memoryManager;
    
    // 创建进程分配内存
    public boolean allocateMemoryForProcess(int pid, int size) {
        PCB pcb = pcbManager.findPCB(pid);
        if (pcb == null) return false;
        
        MemoryAllocation allocation = memoryManager.allocateMemory(pid, size);
        if (allocation != null) {
            MemoryInfo memInfo = new MemoryInfo();
            memInfo.setBaseAddress(allocation.getBaseAddress());
            memInfo.setMemorySize(allocation.getSize());
            pcb.setMemoryInfo(memInfo);
            return true;
        }
        return false;
    }
    
    // 进程终止释放内存
    public void releaseProcessMemory(int pid) {
        memoryManager.freeMemory(pid);
    }
}
```

### 7.2 PCB与调度器的交互

PCB与处理机调度模块交互，处理进程的调度和执行：

```java
// 处理机调度接口
public interface Scheduler {
    // 将进程添加到就绪队列
    void addToReadyQueue(PCB pcb);
    
    // 从就绪队列中选择下一个执行的进程
    PCB selectNextProcess();
    
    // 进程时间片到期
    void timeSliceExpired(PCB pcb);
    
    // 进程等待事件
    void waitForEvent(PCB pcb, Event event);
    
    // 进程被唤醒
    void wakeupProcess(PCB pcb);
}

// PCB-调度器交互流程
public class PCBSchedulerInteraction {
    private PCBManager pcbManager;
    private Scheduler scheduler;
    
    // 将新创建的进程加入就绪队列
    public void addNewProcessToReady(int pid) {
        PCB pcb = pcbManager.findPCB(pid);
        if (pcb != null && pcb.getState() == ProcessState.NEW) {
            pcbManager.updatePCBState(pid, ProcessState.READY);
            scheduler.addToReadyQueue(pcb);
        }
    }
    
    // 处理进程时间片到期
    public void handleTimeSliceExpired(int pid) {
        PCB pcb = pcbManager.findPCB(pid);
        if (pcb != null && pcb.getState() == ProcessState.RUNNING) {
            scheduler.timeSliceExpired(pcb);
            pcbManager.updatePCBState(pid, ProcessState.READY);
        }
    }
}
```

### 7.3 PCB与进程间通信

PCB支持进程间通信(IPC)机制，包括消息传递和共享内存：

```java
// 进程间通信接口
public interface IPC {
    // 发送消息
    boolean sendMessage(int senderPid, int receiverPid, Message message);
    
    // 接收消息
    Message receiveMessage(int receiverPid, boolean blocking);
    
    // 创建共享内存区
    SharedMemory createSharedMemory(int ownerPid, int size);
    
    // 附加到共享内存
    boolean attachSharedMemory(int pid, int sharedMemoryId);
    
    // 从共享内存分离
    boolean detachSharedMemory(int pid, int sharedMemoryId);
}

// PCB-IPC交互流程
public class PCBIPCInteraction {
    private PCBManager pcbManager;
    private IPC ipcManager;
    
    // 进程发送消息
    public boolean sendMessageBetweenProcesses(int senderPid, int receiverPid, String content) {
        PCB senderPCB = pcbManager.findPCB(senderPid);
        PCB receiverPCB = pcbManager.findPCB(receiverPid);
        
        if (senderPCB != null && receiverPCB != null) {
            Message message = new Message();
            message.setSenderPid(senderPid);
            message.setReceiverPid(receiverPid);
            message.setData(content.getBytes());
            message.setTimestamp(System.currentTimeMillis());
            
            return ipcManager.sendMessage(senderPid, receiverPid, message);
        }
        return false;
    }
    
    // 进程接收消息
    public Message receiveProcessMessage(int receiverPid, boolean blocking) {
        PCB receiverPCB = pcbManager.findPCB(receiverPid);
        
        if (receiverPCB != null) {
            if (blocking) {
                // 如果是阻塞接收，需要更新进程状态
                pcbManager.updatePCBState(receiverPid, ProcessState.BLOCKED);
            }
            
            Message message = ipcManager.receiveMessage(receiverPid, blocking);
            
            if (message != null && blocking) {
                // 接收到消息后，如果是阻塞式接收，恢复进程状态
                pcbManager.updatePCBState(receiverPid, ProcessState.READY);
            }
            
            return message;
        }
        return null;
    }
}
```

### 7.4 PCB与文件系统的交互

PCB需要跟踪进程打开的文件，与文件系统模块交互：

```java
// 文件系统接口
public interface FileSystem {
    // 打开文件
    FileHandle openFile(int pid, String path, FileMode mode);
    
    // 关闭文件
    boolean closeFile(int pid, int fileDescriptor);
    
    // 读文件
    byte[] readFile(int pid, int fileDescriptor, int offset, int length);
    
    // 写文件
    int writeFile(int pid, int fileDescriptor, byte[] data, int offset);
    
    // 获取进程打开的所有文件
    List<FileHandle> getOpenFiles(int pid);
}

// PCB-文件系统交互流程
public class PCBFileSystemInteraction {
    private PCBManager pcbManager;
    private FileSystem fileSystem;
    
    // 进程打开文件
    public FileHandle processOpenFile(int pid, String path, FileMode mode) {
        PCB pcb = pcbManager.findPCB(pid);
        
        if (pcb != null) {
            FileHandle handle = fileSystem.openFile(pid, path, mode);
            if (handle != null) {
                // 更新PCB中的打开文件列表
                pcb.getOpenFiles().add(handle);
            }
            return handle;
        }
        return null;
    }
    
    // 进程关闭所有文件（终止时）
    public void closeAllProcessFiles(int pid) {
        PCB pcb = pcbManager.findPCB(pid);
        
        if (pcb != null) {
            List<FileHandle> openFiles = pcb.getOpenFiles();
            for (FileHandle file : openFiles) {
                fileSystem.closeFile(pid, file.getFileDescriptor());
            }
            openFiles.clear();
        }
    }
}
```

## 8. PCB序列化与持久化

为了支持进程迁移和系统状态保存，PCB需要支持序列化与持久化：

```java
// PCB序列化接口
public interface PCBSerializer {
    // 将PCB序列化为字节数组
    byte[] serialize(PCB pcb);
    
    // 从字节数组反序列化PCB
    PCB deserialize(byte[] data);
    
    // 保存PCB到文件
    void savePCBToFile(PCB pcb, String filePath);
    
    // 从文件加载PCB
    PCB loadPCBFromFile(String filePath);
}

// PCB序列化实现示例
public class PCBSerializerImpl implements PCBSerializer {
    @Override
    public byte[] serialize(PCB pcb) {
        try (ByteArrayOutputStream bos = new ByteArrayOutputStream();
             ObjectOutputStream oos = new ObjectOutputStream(bos)) {
            oos.writeObject(pcb);
            return bos.toByteArray();
        } catch (IOException e) {
            // 处理异常
            return null;
        }
    }
    
    @Override
    public PCB deserialize(byte[] data) {
        try (ByteArrayInputStream bis = new ByteArrayInputStream(data);
             ObjectInputStream ois = new ObjectInputStream(bis)) {
            return (PCB) ois.readObject();
        } catch (IOException | ClassNotFoundException e) {
            // 处理异常
            return null;
        }
    }
    
    // 其他方法实现...
}
```

## 9. PCB数据结构测试与验证

PCB作为系统核心数据结构，需要全面的测试策略：

```java
// PCB单元测试示例
public class PCBTest {
    private PCBManager pcbManager;
    
    @Before
    public void setup() {
        pcbManager = new PCBManagerImpl();
    }
    
    @Test
    public void testCreatePCB() {
        PCB pcb = pcbManager.createPCB("TestProcess", 10);
        assertNotNull(pcb);
        assertEquals("TestProcess", pcb.getProcessName());
        assertEquals(10, pcb.getPriority());
        assertEquals(ProcessState.NEW, pcb.getState());
    }
    
    @Test
    public void testUpdatePCBState() {
        PCB pcb = pcbManager.createPCB("TestProcess", 10);
        int pid = pcb.getPid();
        
        boolean result = pcbManager.updatePCBState(pid, ProcessState.READY);
        assertTrue(result);
        
        PCB updatedPCB = pcbManager.findPCB(pid);
        assertEquals(ProcessState.READY, updatedPCB.getState());
    }
    
    // 其他测试方法...
}
```

## 10. 实现计划

### 10.1 第一阶段实现 (1-2周)

1. **核心PCB类**
   - 实现基本PCB结构
   - 添加必要的getter/setter方法
   - 确保属性的不可变性和线程安全

2. **PCB管理器**
   - 实现PCB创建、查找、更新和删除功能
   - 添加状态变更方法
   - 实现PCB索引机制

3. **进程队列**
   - 实现基本队列操作
   - 添加优先级支持
   - 确保线程安全

### 10.2 第二阶段实现 (3-4周)

1. **PCB与其他模块交互**
   - 实现与内存管理模块的接口
   - 实现与调度器的接口
   - 添加文件系统支持

2. **进程间通信**
   - 实现消息传递机制
   - 添加共享内存支持
   - 实现信号量机制

3. **PCB序列化**
   - 添加序列化支持
   - 实现状态保存和恢复
   - 添加进程迁移支持

### 10.3 第三阶段实现 (后续扩展)

1. **性能优化**
   - 实现PCB缓存机制
   - 优化队列操作性能
   - 减少锁竞争

2. **高级功能**
   - 添加进程克隆功能
   - 实现进程组管理
   - 支持进程优先级动态调整

3. **监控与调试**
   - 添加PCB状态监控机制
   - 实现进程执行跟踪
   - 添加性能统计与分析

## 参考资料

1. Abraham Silberschatz, Peter B. Galvin, Greg Gagne, 《操作系统概念》, 第9章 - 进程管理
2. Andrew S. Tanenbaum, 《现代操作系统》, 第2章 - 进程与线程
3. Brian Goetz等, 《Java并发编程实践》, 第5章 - 构建阻塞队列
4. Erich Gamma等, 《设计模式：可复用面向对象软件的基础》, 工厂模式与观察者模式
5. Java官方文档, java.util.concurrent包, https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/package-summary.html 