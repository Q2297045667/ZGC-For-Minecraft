# ZGC 用于 Minecraft
## 前言
请注意，通常认为 ZGC 必须针对你使用的机器进行精细调整。尽管对于 17 以下的 Java 版本来说确实如此，但如今这些设置已经动态化，因此无需再进行配置。
## ZGC 是什么？
ZGC 代表 Z 垃圾收集器，简单来说，垃圾收集器是一种用于在服务器内部释放内存以便进一步使用的工具。除了 ZGC，还有其他几种垃圾收集器，以及许多可以改变其运行方式的标志（选项）。Aikar 的标志是一组常用的垃圾收集标志，但它们也有局限性，因为尽管它们可能很好，但垃圾收集器仍然需要暂停服务器才能运行。然而，ZGC 通过主要与服务器并行运行（而不是每次运行时都暂停服务器）来解决这个问题。
## 谁应该使用 ZGC？
ZGC 需要分配大量内存，因此 ZGC 最适合分配了 *至少* 8GB 内存的服务器，这没关系，因为低于此配置的服务器无论如何都可以使用 G1GC 而不会出现问题。但请注意，ZGC 可能会导致内存使用量增加，而且尚未进行广泛测试，因此可能需要分配更多内存。
换句话说，只有大型服务器才应该使用 ZGC。
## 建议的 ZGC 标志：
```
java -Xms8G -Xmx8G -XX:+UnlockExperimentalVMOptions -XX:+UseZGC -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -XX:+PerfDisableSharedMem -XX:-ZUncommit -XX:+ParallelRefProcEnabled -jar server.jar --nogui
```
请记住，使用其他标志要么：
1. 增加总体 CPU 使用率，但垃圾收集性能提升有限
2. 降低总体 CPU 使用率，但垃圾收集器可能会不堪重负。

总体而言，除非你确切知道自己在做什么，并且有充分的理由更改它们，否则最好不要更改这些标志。唯一已知的例外是 `-XX:+UseLargePages` 标志，由于需要事先进行一些设置，因此未包含在内。

### 重要说明：
- 根据你希望分配给服务器的内存量（以 GB 为单位）更改 `-Xmx` 和 `-Xms`。
- 将 `server.jar` 更改为你正在使用的 jar 文件的名称。
- `+` 表示明确启用该标志，而 `-` 表示明确禁用该标志。

### 每个标志的原因：

| 标志 | 原因 |
|--------------|-------------|
| `-Xms` 和 `-Xmx` 匹配 | 防止因向操作系统请求额外内存而导致性能问题，并减少垃圾收集器在获取额外内存之前需要做的工作量。 |
| `-XX:+UnlockExperimentalVMOptions` | 是许多其他标志所必需的。 |
| `-XX:+UseZGC` | 启用 ZGC |
| `-XX:+DisableExplicitGC` | 防止插件和其他代码干扰垃圾收集器。 |
| `-XX:+AlwaysPreTouch` | 在启动时设置并保留内存。提高效率和访问速度。 |
| `-XX:+PerfDisableSharedMem` | 防止垃圾收集器写入文件系统，减少因 I/O 导致的延迟。详情见 [此处](https://www.evanjones.ca/jvm-mmap-pause.html) |
| `-XX:-ZUncommit` | 防止 ZGC 解除内存提交并将内存返回给操作系统。 |
| `-XX:+ParallelRefProcEnabled` | 按照 Aikar 的说法：“优化垃圾收集过程，使其使用多个线程进行弱引用检查。不清楚为什么这还不是默认设置……” |

## ZGC 的其他可用标志：

查看 [此处](https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html) 了解大多数通用标志。

| 标志 | 原因 |
|--------------|-------------|
| `-XX:+UseLargePages` | 确实可以提升性能且没有缺点。但需要进行一些设置。 |
| `-XX:+UseTransparentHugePages` | 按照 Aikar 的说法：“这是一个有争议的功能，如果你无法为宿主配置真正的 HugeTLBFS，可能会有用。我们尚未测量 THP 对 MC 的作用，或者它与 AlwaysPreTouch 的影响，因此这一部分是为那些想要尝试的高级用户准备的。” |
| `-XX:ConcGCThreads` | 垃圾收集器使用的线程数。默认情况下动态设置。 |
| `-XX:ZAllocationSpikeTolerance` | ZGC 的内存分配峰值容忍度，默认值为 2。值越大，内存分配率越高。 |
| `-XX:ZCollectionInterval` | ZGC 发生的时间间隔（以秒为单位）。默认值为 0。 |
| `-XX:ZFragmentationLimit` | 如果当前区域碎片化程度大于此值，则该区域将被回收（默认值为 25）。 |
| `-XX:ZMarkStackSpaceLimit` | 指定标记堆栈分配的最大字节数。默认值为 8589934592（8096M）。 |
| `-XX:ZProactive` | 是否启用主动回收。默认值为 true。 |
| `-XX:UseNUMA` | 启用 NUMA 支持。默认情况下根据机器自动设置。使用此标志强制启用/禁用。 |
| `-XX:ZUncommitDelay` | 设置堆内存在被解除提交之前必须空闲的时间（以秒为单位，需启用 ZUncommit）。默认值为 300。 |

## 主要来源：
[Purpur Discord](https://discord.gg/purpurmc-685683385313919172)

[Aikar 的标志](https://docs.papermc.io/paper/aikars-flags)

[Oracle 文档](https://docs.oracle.com/en/java/javase/17/gctuning/index.html) / [Oracle](https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html)

[OpenJDK Wiki](https://wiki.openjdk.org/display/zgc/Main)

[Dev.java](https://dev.java/learn/jvm/tool/garbage-collection/zgc-overview/)
