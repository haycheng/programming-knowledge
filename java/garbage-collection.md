# JVM 垃圾收集（Garbage Collection，GC）笔记

#### Q：垃圾收集的基本步骤是什么？
A：JVM中的垃圾收集都是先标记（Mark），然後再清理（Sweep），最後为了防止碎片过多而无法进行下一次内存分配，还需要进行去碎片操作（Defrag），以获取更大的连续可用内存。也就是个“Mark, Sweep and Defrag Garbage Collection”的过程。

#### Q：Hotspot 虚拟机的堆分为哪几个区域？
A：分为新生代（Young Generation）、老年代（Old Generation，或 Tenured Generation）、永久代（Permanent Generation），其中新生代包括一个Eden区和两个Survivor区。

#### Q：GC的算法有哪些？
A：单代的GC算法有：标记-清除（Mark-Sweep）算法，复制（Copying）算法，标记-整理（Mark-Compact）算法。
- 标记-清除算法：在进行GC的内存中找出已死的对象後，将其所占内存标记为可使用；存在大量不连续的可用内存块。
- 复制算法：将进行GC的内存中存活的对象，复制到一块新的内存中；GC前需要准备一块与待GC区域大小相同的内存区域（最坏情况。通常只需要比待GC区域小得多的一块内存）。
- 标记-整理算法：将进行GC的内存中存活的对象，从左对齐紧凑排列。

通常会对不同代的内存，采用不同的GC算法，这叫分代回收（Generational Collection）。

#### Q：常见的垃圾收集器有哪些？
A：**不同代的内存区域，使用不同的垃圾收集器**。垃圾收集器都是仅限于特定内存区域的（G1除外，它可以作用于新生代和老年代）。

用于新生代的垃圾收集器：
- Serial：采用复制算法；单线程进行垃圾收集；垃圾收集时用户线程停止。
- ParNew：采用复制算法；多线程进行垃圾收集；垃圾收集时用户线程停止。
- Parallel Scavenge：与ParNew同，只是该收集器的目标是让CPU尽量用于运行用户的代码，适用于大量计算的任务。

由此可见，**新生代垃圾收集器基本都是使用复制算法的**。

用于老年代的垃圾收集器：
- Serial Old：Serial 的老年代版本；除了采用标记-整理算法，其他与 Serial 相同。
- Parallel Old：Parallel Scavenge 的老年代版本；除了采用标记-整理算法，其他与 Parallel Scavenge 相同。
- CMS（Concurrent Mark Sweep）：以获取最短停顿时间为目标，适用于Web服务端，提高响应用户的速度；分为初始标记（CMS initial mark）、并发标记（CMS concurrent mark）、重新标记（CMS remark）、并发清除（CMS concurrent sweep）4个步骤；初始标记和重新标记阶段都有STW(stop the world)，但时间较短；并发标记和并发清除阶段与用户线程同时运行。

与新生代不同，**老年代的垃圾收集器不会使用复制算法，而用标记-整理算法**。因为采用复制算法的话，要么用两块同样大小的内存，而老年代的内存块一般较大，这么做比较浪费；要么也分成类似的Eden和两个Survivor区块，这样就会存在存留对象多而To-Survivor存放不下的风险。所以老年代的垃圾收集采用标记-整理算法比较合适。

**注**：垃圾收集器中的 parallel 表示多个垃圾收集线程并发进行工作，concurrent 表示垃圾收集线程与用户线程并行运行。

#### Q：常用的垃圾收集器组合有哪些？
A：

#### Q：GC 类型有哪些？
A：新生代的GC叫做 Minor GC，老年代的GC叫做 Major GC，同时包括新生代和老年代的GC叫做 Full GC。Minor GC 含义比较明确，但 Major GC 和 Full GC 的关系比较微妙，而且後两者并没有官方的定义。

#### Q：什么时候会触发 Minor GC？
A：Eden区没有足够的空间用于分配新的对象时，就会产生 Minor GC。因此new 新对象的速度越快，则 Minor GC 频率越高。

#### Q：Minor GC 过程中有 STW 吗？
A：有，如 Serial 收集器的标记阶段。因为Eden区的对象绝大部分都会死掉，只有很少一部分需要复制到Survivor或Old区，所以其STW的时间很短，对应用的影响也很小（除非Eden中有大量的对象存活）。

#### Q：什么触发 Major GC？
A：Minor GC

#### Q：什么触发 Full GC？
A：Minor GC

### 参考文档：
1. [Garbage Collection in Java](https://plumbr.io/handbook/garbage-collection-in-java)
1. [Java Garbage Collection Basics](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)
1. [Major GC和Full GC的区别是什么？触发条件呢？](https://www.zhihu.com/question/41922036)
1. [聊聊JVM（四）深入理解Major GC, Full GC, CMS](https://blog.csdn.net/ITer_ZC/article/details/41825395)
1. [JVM参数MetaspaceSize的误解](https://www.jianshu.com/p/b448c21d2e71)
