# 核心模式：O(logN) 动态有序 (平衡结构)



**挑战：** 数据流是动态的，需要 O(logN) 增删，并快速查询有序信息（如中位数、区间）。 **应对策略：** 双堆、`TreeMap`、`SkipList`。

##**LC 295. Find Median from Data Stream (数据流的中位数)**

- **简介：** `addNum(num)` (O(logN)), `findMedian()` (O(1))。
- **设计：** **`MaxHeap` (存较小一半) + `MinHeap` (存较大一半)**。始终保持两个堆的大小平衡（相差不超过1）。中位数即为 `(maxHeap.peek() + minHeap.peek()) / 2` 或 `maxHeap.peek()`。

##**LC 352. Data Stream as Disjoint Intervals (将数据流变为多个不相交区间)**

- **简介：** `addNum(val)` (O(logN)), `getIntervals()` (O(N))。
- **设计：** 使用 `TreeMap<Start, End>`。`addNum(val)` 时，检查 `val` 能否与 `TreeMap` 中已有的区间（通过 `floorKey` 和 `ceilingKey` 查找）合并。

##**LC 715. Range Module (Range 模块)**

- **简介：** `addRange`, `removeRange`, `queryRange` 均为 O(logN)。
- **设计：** `TreeMap<Start, End>`。这是 `LC 352` 的高级版，`add/remove` 时涉及更复杂的区间合并与分裂。

##**LC 1206. Design Skiplist (设计跳表)**

- **简介：** 实现 `search`, `add`, `erase` 平均 O(logN)。
- **设计：** `Skiplist` (跳表) 本身。一种概率性数据结构，用多层链表实现 O(logN) 查找。面试中作为 `TreeMap` 的替代方案提出会非常亮眼。

