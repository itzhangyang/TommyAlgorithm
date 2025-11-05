# 核心模式：O(logN) 动态有序 (平衡结构)



**挑战：** 数据流是动态的，需要 O(logN) 增删，并快速查询有序信息（如中位数、区间）。 **应对策略：** 双堆、`TreeMap`、`SkipList`。

##**LC 295. Find Median from Data Stream (数据流的中位数)**

- **简介：** `addNum(num)` (O(logN)), `findMedian()` (O(1))。
- **设计：** **`MaxHeap` (存较小一半) + `MinHeap` (存较大一半)**。始终保持两个堆的大小平衡（相差不超过1）。中位数即为 `(maxHeap.peek() + minHeap.peek()) / 2` 或 `maxHeap.peek()`。

##**LC 352. Data Stream as Disjoint Intervals (将数据流变为多个不相交区间)**

- **简介：** `addNum(val)` (O(logN)), `getIntervals()` (O(N))。
- **设计：** 使用 `TreeMap<Start, End>`。`addNum(val)` 时，检查 `val` 能否与 `TreeMap` 中已有的区间（通过 `floorKey` 和 `ceilingKey` 查找）合并。

### 问题要点 (Key Points)

- **数据流 (Data Stream):** 数字是一个一个按顺序给出的，你无法预知未来的数字。

- **动态维护 (Dynamic Maintenance):** 每添加一个数字 `val`，都需要**立即**更新当前的区间列表。

- **不相交区间 (Disjoint Intervals):** 维护的区间列表必须是互相不重叠的。

- **合并 (Merge):** 当新来的数字 `val` 恰好能连接两个已有的区间时（例如，已有 `[1, 3]` 和 `[5, 6]`，新来 `4`），需要将它们合并成一个大区间（`[1, 6]`）。

- **扩展 (Extend):** 当 `val` 恰好在某个区间的任一端点旁边时（例如，已有 `[1, 3]`，新来 `4`），需要扩展该区间（`[1, 4]`）。

- **查询 (Query):** `getIntervals()` 操作需要返回当前所有区间的**有序**列表。

### 问题本质和分析 - 动态区间合并

这道题的本质是 **“动态区间合并” (Dynamic Interval Merging)**。

它不同于 LeetCode 56 (Merge Intervals)，在第 56 题中，你一次性拿到所有区间，排序后线性扫描合并即可。

在这道题中，`addNum` 和 `getIntervals` 是交替调用的。

`addNum` 的调用次数可能非常多。

如果我们每次 `addNum` 都像第 56 题一样，把所有数字（或单点区间）拿出来重新排序合并一遍，效率会非常低。

因此，`addNum` 操作必须非常高效。它需要做到：

1. 快速定位新数字 `val` 应该在哪个位置。
2. 快速找到 `val` 的“左邻居”（即以 `val-1` 结尾的区间）和“右邻居”（即以 `val+1` 开始的区间）。
3. 高效地执行合并、扩展或插入新区间操作。

`getIntervals` 操作因为需要返回所有区间，其时间复杂度至少是 $O(N)$（其中 $N$ 是当前区间的数量）。

这暗示我们 `addNum` 的时间复杂度最好是 $O(\log N)$ 或 $O(1)$，而不是 $O(N)$ 或 $O(N \log N)$。

### 规律观察

####**情况 1：完全覆盖 (Already Covered)** - 已经落在一个存在的区间

- `val` 已经落在一个现有区间 `[a, b]` 内部（即 $a \le val \le b$）。
- **动作：** 什么都不做。

####**情况 2：左右合并 (Merge Both)** - 存在左右相邻区间

- 存在一个区间 `[a, val-1]` 并且存在另一个区间 `[val+1, b]`。
- **动作：** 将这两个区间和 `val` 合并成一个新区间 `[a, b]`。

####**情况 3：向左合并 (Merge Left)** - 存在左相邻区间

- 只存在一个区间 `[a, val-1]`。
- **动作：** 将该区间扩展为 `[a, val]`。

####**情况 4：向右合并 (Merge Right)** - 存在右相邻空间

- 只存在一个区间 `[val+1, b]`。
- **动作：** 将该区间扩展为 `[val, b]`。

####**情况 5：独立区间 (New Interval)** 

- `val` 的两边（`val-1` 和 `val+1`）都没有连接到任何现有区间。
- **动作：** 创建一个新区间 `[val, val]`。

### 模式套路匹配 (Pattern Matching)

1. **问题类型：** 动态集合管理（插入、删除、查询）。
2. **核心需求：** 集合中的元素（区间）需要保持**有序**（按起始点排序）。
3. **高效操作：** 需要 $O(\log N)$ 级别的插入、删除和**查找**（查找 `val` 的邻居）

这个需求模式完美匹配 **平衡二叉搜索树 (Balanced BST)**。

在 Java 中，实现平衡 BST 的数据结构是：

- `TreeSet`: 仅存储 "键"。
- `TreeMap`: 存储 "键-值" 对。

#### 利用TreeMap的重要特性和功能

`TreeMap` 在 Java 中是 `SortedMap` 和 `NavigableMap` 接口的实现，它底层的数据结构是**红黑树 (Red-Black Tree)**。

红黑树是一种自平衡的二叉搜索树 (BST)。这意味着它在保持元素有序的同时，还能保证增、删、查操作的时间复杂度都稳定在 $O(\log N)$。



##### 自动排序

 `TreeMap` 会自动根据**键 (Key)** 的自然顺序（或者您在构造时传入的 `Comparator`）对所有条目 (Entries) 进行排序。

##### 高效的 $O(\log N)$ 增删查 (Efficient Operations)

作为一棵平衡树，`TreeMap` 的 `put(key, value)`, `get(key)`, `remove(key)` 操作的平均和最坏时间复杂度都是 $O(\log N)$。

##### 强大的导航 API (Powerful Navigation API)

这是 `TreeMap` 解决此问题的**最核心武器**。`TreeMap` 实现了 `NavigableMap` 接口，这为我们提供了强大的、基于"顺序"的查找能力。

在 `addNum(val)` 时，我们不是在找 `val` 是否存在，而是在找 `val` 的**"邻居"**。

以下是我们在解法中用到的关键 API：

######🚀 `floorKey(K key)`

返回 $\le key$ 的**最大键** (returns the greatest key $\le key$)。如果不存在，返回 `null`。

###### 🚀 `higherKey(K key)`

返回 $> key$ 的**最小键** (returns the least key $> key$)。如果不存在，返回 `null`。



### 核心思想和套路 (Core Idea and Techniques) - TreeMap

####**核心思想：** 使用 `TreeMap` 来存储和维护不相交的区间。



- **键 (Key):** 区间的 **起始点 (start)**。
- **值 (Value):** 区间本身，即一个数组 `int[start, end]`。

`TreeMap` 会自动根据 "键"（即区间的起始点）对所有区间进行排序。

这使得 `getIntervals()` 操作（$O(N)$）和 `addNum` 中的查找操作（$O(\log N)$）都非常高效。

#### `addNum(val)` 的核心套路：

#####检查是否存在覆盖区间

###### 找到左侧最靠近$val$的区间

- 使用 `map.floorKey(val)` 找到 $\le val$ 的最大起始点 `lKey`。

###### 检查左侧区间是否覆盖$val$

- 如果 `lKey` 存在，并其对应的区间 `map.get(lKey)` 的结束点 $\ge val$（即 `map.get(lKey)[1] >= val`），说明 `val` 已被覆盖，直接返回。

#####**检查邻居 (Cases 2, 3, 4):**

######检查左侧区间是否和$val$相邻？

获取左侧区间（即小于$val$的最大区间起点）$interval$， 并检查区间的终点是否和$val$相邻：$interval[end] == val - 1$。

如果区间和$val$是**相邻的**, 则可以合并左侧区间

- `boolean leftConnect = (lKey != null && map.get(lKey)[1] == val - 1);`

######检查右侧区间是否和$val$相邻？

如何找到右侧相邻区间？

我们的方法是找到起点$\gt {val}$的最小的区间

- `Integer hKey = map.higherKey(val);` (或 `map.ceilingKey(val+1)`)
- `boolean rightConnect = (hKey != null && hKey == val + 1);`

##### 执行操作 (Cases 2, 3, 4, 5):

###### 如果左右区间都相邻：合并三个区间

- 合并左右两个区间。`int[] left = map.get(lKey);` `int[] right = map.remove(hKey);`

- 更新左区间的结束点：`left[1] = right[1];`

###### 只是左区间相邻：合并左区间

- 扩展左区间。`int[] left = map.get(lKey);`

- 更新左区间的结束点：`left[1] = val;`

###### 只是右区间相邻

- 扩展右区间。`int[] right = map.remove(hKey);`

- 创建新区间 `[val, right[1]]` 并放入 map：`map.put(val, new int[]{val, right[1]});` (注意：Key 变了，必须 "先删后增")。

###### 左右区间都不相邻：创建新区间

- 创建新区间。`map.put(val, new int[]{val, val});`

### 实现原理和步骤 (Implementation Principle and Steps)

####成员变量:`TreeMap<start,<start,end>>`

- `private TreeMap<Integer, int[]> map;`
- `map` 的键是区间的 `start`，值是 `[start, end]` 数组。

####**构造函数 `SummaryRanges()`:**

- `map = new TreeMap<>();`

####**`addNum(int val)`:**

#####**步骤 1 (检查覆盖):**

- `Integer lKey = map.floorKey(val);`
- `if (lKey != null && map.get(lKey)[1] >= val) { return; }`

#####**步骤 2 (检查连接):**

- `boolean leftConnect = (lKey != null && map.get(lKey)[1] == val - 1);`
- `Integer hKey = map.higherKey(val);`
- `boolean rightConnect = (hKey != null && hKey == val + 1);`

#####**步骤 3 (执行合并/扩展/新增):**

- `if (leftConnect && rightConnect)`:
  - `int[] leftInterval = map.get(lKey);`
  - `int[] rightInterval = map.remove(hKey);` // 移除右区间
  - `leftInterval[1] = rightInterval[1];` // 更新左区间
- `else if (leftConnect)`:
  - `map.get(lKey)[1] = val;` // 扩展左区间
- `else if (rightConnect)`:
  - `int[] rightInterval = map.remove(hKey);` // 移除右区间
  - `map.put(val, new int[]{val, rightInterval[1]});` // 添加以 val 为 start 的新区间
- `else`:
  - `map.put(val, new int[]{val, val});` // 添加新区间

####**`getIntervals()`:**

- `int[][] result = new int[map.size()][2];`
- `int i = 0;`
- `for (int[] interval : map.values()) { result[i++] = interval; }`
- `return result;`

### 实现代码

```Java
class SummaryRanges {

    TreeMap<Integer,int[]> map;

    public SummaryRanges() {
        map = new TreeMap<>();

    }
    
    public void addNum(int value) {
        //找到小于当前值的最大起点
        Integer leftStart = map.floorKey(value);
        
        if(leftStart  != null &&  map.get(leftStart)[1] >= value){
            return;
        }
        boolean leftConnect = (leftStart != null && map.get(leftStart)[1] == value - 1);
        
        Integer rightStart = map.higherKey(value);
        boolean rightConnect = (rightStart != null && rightStart == value + 1);

        if(leftConnect && rightConnect){
            int[] leftInterval = map.get(leftStart);
            int[] rightInterval = map.remove(rightStart);
            leftInterval[1] = rightInterval[1];

        }else if(leftConnect){
            int[] leftInterval = map.get(leftStart);
            leftInterval[1] = value;

        }else if(rightConnect){
            int[] rightInterval = map.remove(rightStart);
            rightInterval[0] = value;
            map.put(value, rightInterval);

        }else{
            map.put(value, new int[]{value,value});
        }
    }
    
    public int[][] getIntervals() {
        List<int[]> list =  map.values().stream().toList();
        int[][] ans = new int[list.size()][2];
        for(int i = 0; i < list.size(); i++){
            ans[i] = list.get(i);
        }

        return ans;
    }
}
```



### 注意事项

**复杂度分析：**

- `addNum(val)`: $O(\log N)$。`TreeMap` 的 `floorKey`, `higherKey`, `put`, `remove` 操作都是 $O(\log N)$（$N$ 是当前区间的数量）。
- `getIntervals()`: $O(N)$。需要遍历 `TreeMap` 中的所有 $N$ 个条目来构建结果数组。

**空间复杂度：** $O(N)$，用于存储 $N$ 个区间。

**TreeMap 的 API:** `floorKey(k)`（$\le k$ 的最大键）和 `higherKey(k)`（$> k$ 的最小键）是解决此问题的关键。

**区间值的修改：**

- 当**向左合并**（情况3）或**左右合并**（情况2）时，我们是修改现有 `lKey` 对应的 `int[]` 数组。`TreeMap` 中的键 `lKey` 不变，所以可以直接修改值。
- 当**向右合并**（情况4）时，区间的 `start` 变了（从 `val+1` 变为 `val`）。由于 `start` 是 `TreeMap` 的键，我们**不能**直接修改键。必须 `remove` 旧的键值对，然后 `put` 一个新的键值对。

### 经验总结

LeetCode 352 是考察“动态”问题处理的典范。当你遇到需要**边插入边维护有序性**，并且涉及**区间、范围查找、查找前驱/后继**的问题时，`TreeMap` (或 `TreeSet`) 几乎总是最佳选择。

这个问题的核心套路可以归纳为：**"查邻居，定策略"**。

1. **查邻居：** 使用 `floorKey` 和 `higherKey` 快速定位新元素 `val` 的“左邻居”和“右邻居”。
2. **定策略：** 根据邻居的存在情况（是否为 null）和连接情况（`end == val-1` 或 `start == val+1`），来决定是执行合并、扩展还是新增操作。

##LC 1206. Design Skiplist (设计跳表)

- **简介：** 实现 `search`, `add`, `erase` 平均 O(logN)。
- **设计：** `Skiplist` (跳表) 本身。一种概率性数据结构，用多层链表实现 O(logN) 查找。面试中作为 `TreeMap` 的替代方案提出会非常亮眼。

### 问题要点

你需要设计并实现一个 `Skiplist` 类，该类应包含以下三个方法：

1. `search(int target)`: 判断一个 `target` 是否存在于跳表中。
2. `add(int num)`: 向跳表中插入一个元素 `num`。
3. `erase(int num)`: 从跳表中删除一个元素 `num`。如果 `num` 存在多个，**只删除其中一个**。如果 `num` 不存在，则什么也不做。

题目还额外说明，跳表中的元素可以重复。

所有操作的**平均时间复杂度**应为 $O(\log n)$，空间复杂度为 $O(n)$。

### 问题的本质和分析 - 支持高效增删改的有序集合

这个问题的本质是实现一个**有序集合**（Sorted Set）数据结构，它需要高效地支持查找、插入和删除。

我们来分析一下常见的备选方案：

####**有序数组**：添加和删除需要移动元素($O(N)$)

- `search`: $O(\log n)$ (使用二分查找)。
- `add/erase`: $O(n)$ (需要移动元素以保持有序)。- 不能达到要求

####**有序链表**：搜索和遍历均为$O(N)$

- `search`: $O(n)$ (需要遍历)。
- `add/erase`: $O(n)$ (首先需要 $O(n)$ 查找到位置，然后 $O(1)$ 插入/删除)。

####**平衡二叉搜索树 (BST)** (如红黑树、AVL树)：

- `search/add/erase`: 都能达到 $O(\log n)$。
- **缺点**：实现非常复杂，尤其是插入和删除时涉及的“旋转”和“再平衡”操作。



**跳表（Skiplist）** 正是为了解决这个矛盾而生的。

它是一种**概率性**数据结构，核心思想是在一个有序链表（Level 0）的基础上，增加多级“快速通道”（Level 1, Level 2, ...）。

- Level 0 是一个包含所有元素的标准有序链表。
- Level 1 是一个更稀疏的链表，它只包含 Level 0 中约 1/2 (或 1/p) 的节点。
- Level 2 是一个更稀疏的链表，它只包含 Level 1 中约 1/2 (或 1/p) 的节点。
- 以此类推...

> 跳表示意图： L3: head ----------------> 30 ----------------> null |                      | L2: head ----> 10 --------> 30 ----------------> null |          |           | L1: head ----> 10 -> 20 -> 30 ----> 40 --------> null |          |    |      |       | L0: head -> 5 -> 10 -> 20 -> 30 -> 35 -> 40 -> 50 -> null

###什么是跳表？

顾名思义，跳表是一种类似于链表的数据结构。更加准确地说，跳表是对有序链表的改进。

为方便讨论，后续所有有序链表默认为 **升序** 排序。

#### 普通链表的查询：从头开始逐个比较($O(N)$)

一个有序链表的查找操作，就是从头部开始逐个比较，直到当前节点的值大于或者等于目标节点的值。

很明显，这个操作的复杂度是 𝑂(𝑛)![O(n)](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7)。

#### 跳表的整体结构： 分为多层，每层一个链表

跳表在有序链表的基础上，引入了 **分层** 的概念。

首先，跳表的每一层都是一个有序链表，特别地，最底层是初始的有序链表。

##### $L_0$层：包含所有元素的标准链表

 这是最底层的“国道”或“地面道路”。它是一个**标准、有序的链表**，包含了**所有**的数据节点。

##### $L_{1...k}$ （更高层）：索引层

 这些是“高速公路”或“快速通道”，我们称之为**索引层**。

##### 索引层是如何构建的？

 L1 是从 L0 中**随机**“提拔”一部分节点上来的。

L2 又是从 L1 中**随机**“提拔”一部分节点上来的，以此类推。

换言之，对于每一个$L_k[k=1...n]$，$L_k$中的元素都是从$L_{k-1}$中**随机提拔**上来的。

**结果：** 越往上，层级越高，节点越稀疏，跨度越大。这使得我们可以跳过大量节点进行快速查找。



#### 跳表中层与层之间的关系是怎样的？

##### 每一层内节点之间的关系：简单的有序链表

在**同一个层级**（例如 L2）内，节点之间的关系非常简单：

- 它们构成一个**有序的、稀疏的链表**。
- 每个节点都有一个指向**该层中**下一个节点的指针（`forward` 指针）。
- 例如，在 L2 上，节点 A 可能直接指向节点 D（跳过了 B 和 C），因为 B 和 C 在 L2 这一层不存在。

##### 层与层之间的节点关系： `forward`指针

- **“塔”结构：** 如果一个节点（比如节点 A）存在于高层（如 L2），那么它**必定**也存在于所有比它低的层（L1 和 L0）。

- **垂直连接：** $L_2$ 上的节点 A 和 $L_1$ 上的节点 A、L0 上的节点 A，实际上是**同一个数据节点**。

  这个节点只是拥有多个不同层级的 `forward` 指针（一个用于 $L_2$，一个用于 $L_1$，一个用于 $L_0$）。

- 您可以想象每个被“提拔”的节点都是一座“塔”，塔的高度就是它所在的最高层级。

#### 跳表的查询过程：先走高速，再下匝道

查询过程就是“**先走高速，再下匝道**”的过程。

假设我们要查找值为 `target` 的节点：

#####**从顶层出发：** 查询总是从**最高索引层**（比如 $L_k$）的**头部哨兵节点**开始。

#####**水平遍历（走高速）：** 在当前层（比如 $L_k$）向右遍历。

- 只要下一个节点的值**小于** `target`，就继续向右前进。

##### **寻找“匝道”（下降）：** 

当遇到以下两种情况时，停止向右：

- 下一个节点是 `NULL`（到头了）。
- 下一个节点的值**大于或等于** `target`（说明 `target` 如果存在，必定在当前节点和下一个节点之间）。

#####**垂直下降（下高速）：** 从当前节点**下降一层**（从 $L_k$ 到 $L_{k-1}$）。

##### **重复：** 

在 Lk-1 层，重复第 2 步和第 3 步（在该层继续向右遍历，直到需要下降）。

##### **到达底层：** 

不断重复这个“水平查找、再垂直下降”的过程，直到下降到最底层 L0。

##### **最终查找：** 

在 L0（包含所有节点的层）上执行最后的水平遍历。此时，您停下的位置的下一个节点，就是要找的 `target`（如果它存在的话）。

**总结：** 查询总是从**左上角**开始，尽量向**右**移动，当无法向右时（下一个节点太大或为空），则向**下**移动一层，最终在 L0 层找到目标。这个过程的平均时间复杂度为 $O(\log n)$。



### 规律观察



##**LC 715. Range Module (Range 模块)**

- **简介：** `addRange`, `removeRange`, `queryRange` 均为 O(logN)。
- **设计：** `TreeMap<Start, End>`。这是 `LC 352` 的高级版，`add/remove` 时涉及更复杂的区间合并与分裂。



