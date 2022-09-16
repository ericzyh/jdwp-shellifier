# jdwp 命令测试脚本


### 脚本执行命令

    % python ./jdwp-shellifier.py -h
        usage: jdwp-shellifier.py [-h] -t IP [-p PORT] [--break-on JAVA_METHOD]
                            [--cmd COMMAND]

        Universal exploitation script for JDWP by @_hugsy_

        optional arguments:
        -h, --help            show this help message and exit
        -t IP, --target IP    Remote target IP (default: None)
        -p PORT, --port PORT  Remote target port (default: 8000)
        --break-on JAVA_METHOD
        Specify full path to method to break on (default:
            java.net.ServerSocket.accept)
            --cmd COMMAND         Specify full path to method to break on (default:
                None)


### jdwp 发送包和回复包的结构

    1. 发送包头

        #######################################################################################################################
        #  4字节(头部长度11+数据位长度)  #        4字节(命令序号ID)       #  1字节(发送为0) #   1字节(命令分组) #    1字节(命令id)  #
        #######################################################################################################################
 
    2. 回复包头

        #######################################################################################################################
        #  4字节(头部长度11+数据位长度)  #        4字节(命令序号ID)       #  1字节(发送为\0x80) #        2字节(error code)        #
        #######################################################################################################################

### 流程

1 . java程序启动的时候加以下vm参数，5050为vm开启的端口，transaction表示使用socket通讯，我们的脚本就可以通过5050端口调用vm提供的命令接口 

    % -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5050

2 . 握手

    连接vm端口，发送 "JDWP-Handshake" 握手, VM回复 "JDWP-Handshake" ,表示握手成功，就可以调用jdwp命令了


写交互指令调用命令接口

### jdwp命令列表

#### (1) VirtualMachine Command Set (虚拟机命令分组)

| 命令编号| 命令 | 描述   |
|------|--------|--------|
| 1 | Version | 获取版本信息 | 
| 2 | ClassesBySignature | 根据描述符搜索class信息 (如 Ljava/lang/String) | 
| 3 | AllClasses | 获取所有vm已经加载的类 |
| 4 | AllClasses | 获取所有的线程ID， 包含由vm直接创建的线程和vm通过jni创建的线程id， 不包含未开始或者已经结束的线程|
| 5 | TopLevelThreadGroups | 获取所有没有parent的线程, 这个命令可以用在创建thread树的第一步 |
| 6 | Dispose | 断开本次链接， 其他debugger可以继续链接了， 基于本次链接的所有事件请求全部取消, 被resume暂停的线程继续运行|
| 7 | IDSizes | 获取各种类型id的大小，主要是为了方便后续解析结果和组装请求 |
| 8 | Suspend | 暂停所有线程执行 |
| 9 | Resume | 恢复线程执行，需要注意的是suspend了几次，就需要调用Resume命令几次 |
| 10 | Exit | 结束虚拟机的运行| 
| 11 | CreateString | 创建一个String类型的对象让后返回对象的ID |
| 12 | Capabilities | 查看VM支持那些功能， 如是可以发送监控字段的变化的事件|
| 13 | ClassPaths | 返回classpath和bootclasspath的路径 ?? 两个的区别是啥|
| 14 | DisposeObjects | |
| 15 | HoldEvents | |
| 16 |ReleaseEvents | |
| 17 |CapabilitiesNew | |
| 18 |RedefineClasses | |
| 19 |SetDefaultStratum | |
| 20 |AllClassesWithGeneric | |
| 21 |InstanceCounts | |

#### (2) ReferenceType Command Set (ReferenceType 命令分组)

| 命令编号| 命令 | 描述   |
|------|--------|--------|
| 1 | Signature  | |
| 2 | ClassLoader | |
| 3 | Modifiers | | 
| 4 | Fields | |
| 5 | Methods  | |
| 6 | GetValues  | |
| 7 | SourceFile  | |
| 8 | NestedTypes | |
| 9 | Status  | |
| 10 | Interfaces  | |
| 11 | ClassObject  | |
| 12 | SourceDebugExtension | |
| 13 | SignatureWithGeneric  | |
| 14 | FieldsWithGeneric  | |
| 15 | MethodsWithGeneric  | |
| 16 | Instances  | |
| 17 | ClassFileVersion  | |
| 18 | ConstantPool  | |

#### (3) ClassType Command Set (3)

| 命令编号| 命令 | 描述   |
|------|--------|--------|
| 1 | Superclass  | | 
| 2 | SetValues  | |  
| 3 | InvokeMethod | |
| 4 | NewInstance  | |

#### (4) ArrayType Command Set (4)

| 命令编号| 命令 | 描述   |
|------|--------|--------|
| 1 | NewInstance | |

#### (5) InterfaceType Command Set (5)

| 命令编号| 命令 | 描述   |
|------|--------|--------|
| 1 | InvokeMethod | |


#### (6) Method Command Set (6)

| 命令编号| 命令 | 描述   |
|------|--------|--------|
| 1 | LineTable | |
| 2 | VariableTable | |
| 3 | Bytecodes | |
| 4 | IsObsolete  | | 
| 5 | VariableTableWithGeneric | |

#### (9) ObjectReference Command Set (9)

| 命令编号| 命令 | 描述   |
|------|--------|--------|
| 1 | ReferenceType | |
| 2 | GetValues | |
| 3 | SetValues | | 
| 5 | MonitorInfo  | |
| 6 | InvokeMethod | |
| 7 | DisableCollection  | |
| 8 | EnableCollection | | 
| 9 | IsCollected | |
| 10 | ReferringObjects | |

#### (10) StringReference Command Set (10)

| 命令编号| 命令 | 描述   |
|------|--------|--------|
| 1 | Value | | 

#### (11) ThreadReference Command Set 

| 命令编号| 命令 | 描述   |
|------|--------|--------|
| 1 | Name  | |
| 2 | Suspend  | |
| 3 | Resume | | 
| 4 | Status | | 
| 5 | ThreadGroup  | |
| 6 | Frames | |
| 7 | FrameCount  | |
| 8 | OwnedMonitors  | |
| 9 | CurrentContendedMonitor | |
| 10 | Stop | |
| 11 | Interrupt | |
| 12 | SuspendCount | |
| 13 | OwnedMonitorsStackDepthInfo | | 
| 14 | ForceEarlyReturn  | |

#### (12) ThreadGroupReference Command Set 

| 命令编号| 命令 | 描述   |
|------|--------|--------|
Name (1)
Parent (2)
Children (3)

#### (13) ArrayReference Command Set 

| 命令编号| 命令 | 描述   |
|------|--------|--------|
| 1 |  Length | 返回某个数组的长度 | 
| 2 |  GetValues | 获取某个数组的一部分成员 | 
| 3 |  SetValues | 设置取某个数组的一部分成员 | 

#### (14) ClassLoaderReference Command Set 

| 命令编号| 命令 | 描述   |
|------|--------|--------|
| 1 | 返回被某个classloader加载的所有类 |

#### (15) EventRequest Command Set 

| 命令编号| 命令 | 描述   |
|------|--------|--------|
| 1 | Set | 设置 EVENT REQUEST |
| 2 | Clear | 删除某个通过SET设置的EVENT REQUEST | 
| 3 | ClearAllBreakpoints | 清空所有断点 | 

#### (16) StackFrame Command Set 

| 命令编号| 命令 | 描述   |
|------|--------|--------|
| 1 | GetValues | 返回栈帧里某个字段的值 | 
| 2 | SetValues | 设置栈帧里某个字段的值 | 
| 3 | ThisObject | 返回这个栈帧对应的对象id(this)，如果是静态或本地方法返回null|
| 4 | PopFrames | 弹出顶部的栈帧，进程需要被挂起，需要用CapabilitiesNew看看vm支不支持这个命令,  |

#### (17) ClassObjectReference Command Set 

| 命令编号| 命令 | 描述   |
|------|--------|--------|
| 1 | ReflectedType | 获取一个classed对象的RefernceTypeId |

#### (64) Event Command Set

| 命令编号| 命令 | 描述   |
|------|--------|--------|
| 100 | Composite | |