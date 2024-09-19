# GC 日志

## PrintGCDetails

`-XX:+PrintGCDetails` 在每次GC事件发生时提供详细的GC日志信息，包括堆内存区域的前后大小、所花费的时间等。

结果：
```java
[GC (Allocation Failure) [PSYoungGen: 55416K->7257K(55808K)] 55440K->7289K(182784K), 0.0065462 secs] [Times: user=0.01 sys=0.01, real=0.01 secs] 
[GC (Allocation Failure) [PSYoungGen: 55385K->7537K(103936K)] 55417K->7577K(230912K), 0.0070950 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
```

## PrintGCDateStamps

`-XX:+PrintGCDateStamps` 在GC日志中为每条消息添加实际的时间戳

```java
2024-09-18T14:11:41.635+0800: [GC (Allocation Failure) [PSYoungGen: 55376K->7416K(55808K)] 55392K->7440K(182784K), 0.0059414 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
2024-09-18T14:11:41.859+0800: [GC (Allocation Failure) [PSYoungGen: 55544K->7374K(55808K)] 55568K->7398K(182784K), 0.0057323 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 
```
## PrintTenuringDistribution 

`-XX:+PrintTenuringDistribution ` 打印出对象晋升到老年代的年龄分布信息

```Java
2024-09-18T14:13:29.426+0800: [GC (Allocation Failure) 
Desired survivor size 7864320 bytes, new threshold 7 (max 15)
[PSYoungGen: 55258K->7084K(55808K)] 55282K->7116K(182784K), 0.0169202 secs] [Times: user=0.04 sys=0.00, real=0.02 secs] 
2024-09-18T14:13:29.650+0800: [GC (Allocation Failure) 
Desired survivor size 7864320 bytes, new threshold 7 (max 15)
[PSYoungGen: 55212K->7132K(103936K)] 55244K->7172K(230912K), 0.0093651 secs] [Times: user=0.02 sys=0.00, real=0.01 secs] 
```

## PrintHeapAtGC

`-XX:+PrintHeapAtGC` 在GC前后打印出堆的布局和大小信息

```java
2024-09-18T14:23:42.741+0800: [Full GC (Ergonomics) [PSYoungGen: 19456K->0K(826368K)] [ParOldGen: 61886K->62393K(158720K)] 81342K->62393K(985088K), [Metaspace: 30360K->30360K(1077248K)], 0.1569436 secs] [Times: user=0.38 sys=0.00, real=0.15 secs] 
Heap after GC invocations=14 (full 2):
 PSYoungGen      total 826368K, used 0K [0x0000000782300000, 0x00000007be080000, 0x00000007c0000000)
  eden space 806912K, 0% used [0x0000000782300000,0x0000000782300000,0x00000007b3700000)
  from space 19456K, 0% used [0x00000007b5780000,0x00000007b5780000,0x00000007b6a80000)
  to   space 33280K, 0% used [0x00000007b3700000,0x00000007b3700000,0x00000007b5780000)
 ParOldGen       total 158720K, used 62393K [0x0000000706800000, 0x0000000710300000, 0x0000000782300000)
  object space 158720K, 39% used [0x0000000706800000,0x000000070a4ee758,0x0000000710300000)
 Metaspace       used 30360K, capacity 31460K, committed 31616K, reserved 1077248K
  class space    used 3888K, capacity 4118K, committed 4224K, reserved 1048576K
}
{Heap before GC invocations=15 (full 2):
 PSYoungGen      total 826368K, used 806912K [0x0000000782300000, 0x00000007be080000, 0x00000007c0000000)
  eden space 806912K, 100% used [0x0000000782300000,0x00000007b3700000,0x00000007b3700000)
  from space 19456K, 0% used [0x00000007b5780000,0x00000007b5780000,0x00000007b6a80000)
  to   space 33280K, 0% used [0x00000007b3700000,0x00000007b3700000,0x00000007b5780000)
 ParOldGen       total 158720K, used 62393K [0x0000000706800000, 0x0000000710300000, 0x0000000782300000)
  object space 158720K, 39% used [0x0000000706800000,0x000000070a4ee758,0x0000000710300000)
 Metaspace       used 40394K, capacity 41570K, committed 41728K, reserved 1085440K
  class space    used 4787K, capacity 5045K, committed 5120K, reserved 1048576K
```

## PrintReferenceGC

`-XX:+PrintReferenceGC` 打印出与GC相关的软引用、弱引用、虚引用和终结器引用的处理情况

```Java
2024-09-18T14:29:01.951+0800: [GC (Allocation Failure) 2024-09-18T14:29:01.978+0800: [SoftReference, 0 refs, 0.0001386 secs]2024-09-18T14:29:01.978+0800: [WeakReference, 887 refs, 0.0001785 secs]2024-09-18T14:29:01.978+0800: [FinalReference, 71948 refs, 0.0786627 secs]2024-09-18T14:29:02.057+0800: [PhantomReference, 0 refs, 0.0000366 secs]2024-09-18T14:29:02.057+0800: [JNI Weak Reference, 0.0000483 secs][PSYoungGen: 624608K->18944K(770560K)] 657300K->77171K(853504K), 0.1070115 secs] [Times: user=0.11 sys=0.03, real=0.11 secs]

2024-09-18T14:29:02.825+0800: [GC (Metadata GC Threshold) 2024-09-18T14:29:02.840+0800: [SoftReference, 0 refs, 0.0000446 secs]2024-09-18T14:29:02.840+0800: [WeakReference, 81 refs, 0.0000197 secs]2024-09-18T14:29:02.840+0800: [FinalReference, 91401 refs, 0.0933406 secs]2024-09-18T14:29:02.933+0800: [PhantomReference, 0 refs, 0.0000171 secs]2024-09-18T14:29:02.933+0800: [JNI Weak Reference, 0.0000572 secs][PSYoungGen: 466673K->30720K(782336K)] 524901K->102531K(865280K), 0.1094530 secs] [Times: user=0.10 sys=0.03, real=0.11 secs] 

2024-09-18T14:29:02.935+0800: [Full GC (Metadata GC Threshold) 2024-09-18T14:29:02.978+0800: [SoftReference, 0 refs, 0.0000944 secs]2024-09-18T14:29:02.978+0800: [WeakReference, 706 refs, 0.0001051 secs]2024-09-18T14:29:02.978+0800: [FinalReference, 81 refs, 0.0000601 secs]2024-09-18T14:29:02.978+0800: [PhantomReference, 1 refs, 0.0000114 secs]2024-09-18T14:29:02.978+0800: [JNI Weak Reference, 0.0000512 secs][PSYoungGen: 30720K->0K(782336K)] [ParOldGen: 71811K->69049K(170496K)] 102531K->69049K(952832K), [Metaspace: 34116K->34116K(1081344K)], 0.1547474 secs] [Times: user=0.41 sys=0.00, real=0.16 secs] 

```

## PrintGCApplicationStoppedTime

`-XX:+PrintGCApplicationStoppedTime` 会打印出应用程序因为GC而停顿的时间

```Java
2024-09-18T14:32:08.815+0800: [GC (Allocation Failure) [PSYoungGen: 624608K->19456K(821760K)] 658330K->81797K(900608K), 0.1122155 secs] [Times: user=0.14 sys=0.02, real=0.11 secs] 
2024-09-18T14:32:08.927+0800: [Full GC (Ergonomics) [PSYoungGen: 19456K->0K(821760K)] [ParOldGen: 62341K->62150K(159232K)] 81797K->62150K(980992K), [Metaspace: 30325K->30325K(1077248K)], 0.1259942 secs] [Times: user=0.35 sys=0.01, real=0.13 secs] 
2024-09-18T14:32:09.053+0800: Total time for which application threads were stopped: 0.2384900 seconds, Stopping threads took: 0.0000507 seconds
2024-09-18T14:32:09.055+0800: Total time for which application threads were stopped: 0.0000804 seconds, Stopping threads took: 0.0000251 seconds
2024-09-18T14:32:09.055+0800: Total time for which application threads were stopped: 0.0002314 seconds, Stopping threads took: 0.0001917 seconds

```

## GC 文件

GC日志输出相关参数:

`-Xloggc:/path/to/gc-%t.log`: 指定GC日志的输出文件路径。%t 是一个占位符，它会被替换成日志文件创建的时间戳。

`-XX:+UseGCLogFileRotation`: 启用此选项后，JVM会在GC日志文件达到一定大小时进行日志文件的轮换，而不是无限制地增长单个日志文件。

`-XX:NumberOfGCLogFiles=14`: 指定轮换的日志文件的最大数量。在这个例子中，当第15个日志文件被创建时，JVM会从第一个文件开始覆盖旧的日志文件。

`-XX:GCLogFileSize=100M`: 设置每个GC日志文件的最大大小。在这个例子中，当文件大小达到100MB时，会触发日志文件的轮换。