
---
title: cuda-- 问题

---

# cuda 问题
我遇到了cuda的warp级别的死锁问题 [Google](https://forums.developer.nvidia.com/t/concurrent-writing-to-a-global-variable/31628)
尽管接触贡献者提供了一些想法，但我感觉那些想法并不适合我，因为我的显卡时1060 3g版本，我不想将加锁和解锁的任务给warp中的某一个线程，这样的话，很可能导致我们的线程出现较多的问题。

所以我是用atomicAdd及其返回值，以及_syncthreads操作解决了对应的操作。
