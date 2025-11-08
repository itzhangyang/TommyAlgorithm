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

#### Skiplist的搜索路径 - 总是从最高层的head节点开始

搜索总是从最高层的 `head` 节点开始。

在当前层（比如 `i` 层），向右（`forward[i]`）遍历，直到**下一个节点**（`next`）的值 $\ge$ `target` 或者到达`null`。

然后，从**当前节点**(`curr`)下降到下一层（`i-1` 层），重复此过程。

####多层结构： 在$L_i$出现，则必定在$L_j(j \lt i)$ 出现

一个节点如果在第 `k` 层出现，那么它一定也会在所有 `j < k` 的层出现。

####随机性 - 抛硬币决定是否放入$L_i(k=1...k)$层

一个节点有多高（即它会出现在多少层）是在插入时**随机决定**的。

我们定义一个概率 `P`（通常是 0.5 或 0.25）。

新节点插入 Level 0后，抛硬币，如果正面朝上（概率 `P`），则将其也插入 Level 1；

再抛一次，如果还是正面，插入 Level 2... 直到反面朝上或达到最大层数限制。

#### 平衡性

正是这种随机性，使得跳表在动态插入和删除后，能够**在概率上**保持 $O(\log n)$ 的结构平衡，而不需要像BST那样进行复杂的确定性旋转。

### 核心思想和套路

####核心思想： “地铁快慢线”的比喻

你就把**跳表（Skiplist）想象成一个城市的地铁系统**。

我们假设从某个起点到某个终点之间有很多趟不同的列车，有的慢，有的快。

最慢的是每一站都会停靠。

##### 慢车：每一站都停

- **Level 0 (最底层)**：这是**“慢车”**，或者叫“站站停”。它连接了城市里的**每一个**站点（也就是你存储的 5, 10, 20, 30...所有数据）。

#####快车：车越快，停的站点越少

- **Level 1, 2, 3... (高层)**：这些是**“快车”**。
  - Level 1 可能只停 10, 30, 40...
  - Level 2 可能只停 30...
  - **层数越高，车越快，停的站越少。**

##### 这个系统的目的 - 跳过大量无关站点，快速到达目的地



**就是为了让你能“跳过”大量你不关心的站，快速到达目的地。**

我们假设现在快慢系统是这样子规划的：

`L0` (慢车，站站停): `head -> 5 -> 10 -> 20 -> 30 -> 35 -> 40 -> 50`

`L1` (快车): `head -> 10 -> 20 -> 30 -> 40`

`L2` (特快): `head -> 10 -> 30`

`L3` (火箭线): `head -> 30`



##### 怎么保证快车比慢车经停的站点数量少呢？- 概率因子

我们需要在这个系统中设置一个**概率因子**$p$。

比如我们设定$p = 0.5$。

那么每次新增加一个站点的时候，我们需要通过这个概率因子来决定从$L_{[1....k]}$中的每一趟车子是否应该经停这个站点。 

真正的逻辑是TMD**“串联”**的！是**“条件概率”**！你TMD要上 L(k+1)，**前提是你TMD已经上了 Lk**！

我们从 $L_0$ (低保) 开始算：

1. **L0 (站站停)**
   - 概率是多少？**TMD是 100% ！**
   - 只要你 `add`，L0 就TMD必须有！
2. **L1 (第一层快线)**
   - 概率是多少？
   - 是你TMD**在 L0 的基础上**（概率100%），**再**抛一次硬币（概率0.5）。
   - 总概率：`1 * 0.5` = **50%**
3. **L2 (第二层快线)**
   - 概率是多少？
   - 是你TMD**已经上了 L1**（概率50%），**再**抛一次硬币（概率0.5）。
   - 总概率：`0.5 * 0.5` = **25%**
4. **L3 (第三层快线)**
   - 概率是多少？
   - 是你TMD**已经上了 L2**（概率25%），**再**抛一次硬币（概率0.5）。
   - 总概率：`0.25 * 0.5` = **12.5%**
5. **... Lk ...**
   - 总概率：**$(0.5)^k$**

我们TMD不是靠“运气”去“保证”它递减。

我们是TMD通过这个“串联”的、**概率TMD逐层递减**的**『设计』**，使得整个跳表在**『统计期望』**上，TMD**必然**是越往上层节点越稀疏（越少）！



##### 怎么决定最多设置多少趟列车？- 取决于站点数量

**这是你TMD自己定的『安全上限』。**

 你是盖楼的，你TMD总得有个章程，是盖 32 层封顶，还是 64 层封顶。

因为 LeetCode 的测试数据量（比如 20000 次操作），你TMD就算把 `MAX_LEVEL` 设成 16，甚至 10 (如果 `P=0.5` 的话，$\log_2(20000)$ 大概是 14.x)，都TMD**绰绰有余**了！

你设 32，是TMD为了“理论上”应付 $2^{32}$ 规模的数据，但 LeetCode 根本TMD考不到那个量级。

**你只要设一个TMD“合理”的值（比如 32），能通过它的测试就行了。** 你设 100 也行，就是TMD浪费点 `head` 节点的内存。



####Search (查找)：去 35 号站

现在我们的目标是去35号站点。

#####**如果只有慢车 (Level 0)**：

你必须从 `head` (总站)出发，坐慢车，一站一站地数：5, 10, 20, 30, 35... 太慢了，这就是 $O(n)$。

#####**有了快慢线系统 (跳表)**：先乘坐最快的车到距离最近的站点

这里的核心思想是先在$L_k$中一直乘坐到最后一个$\le target$  的站点$curr$。

如果最后这个站点就是$target$，那么就到站了。

如果这个站点不是$target$， 那么就在站点$curr$换成到列车$L_{k-1}$，并且继续前行到下一个$\le target$的最大站点。

######乘坐当前最快的车($L_k$)到距离最近的站点 

1. 你从 `head` (总站)出发，**永远先坐最快的车** (比如 Level 3)， 这个最快的车，就是所谓**当前最高的层级**。

2. L3 的下一站是 30。

   再下一站是 `null`。

   你发现 30 < 35，所以你坐到 30。

###### 在距离最近的站点切换到$L_{k-1}$

1. 在 30 号站，你**下车“换乘”**，换到下一级（Level 2），**注意这个时候你还在站点30**。

2. L2 的下一站是 `null`。你还在 30。

   

3. 在 30 号站，**下车“换乘”**（Level 1）。

4. L1 的下一站是 40。你发现 **40 > 35 (过站了！)**。所以你**不能坐这趟车**，你得留在 30 号站。

5. 在 30 号站，**下车“换乘”**（Level 0，慢车）。

###### 经过多次切换最终达到终点

1. L0 的下一站是 35。**找到了！**

**核心套路（查）**：

从**最高层快线**出发，一路向右，只要下一站没“过”（`<= target`），就一直坐。

如果下一站“过”了（`> target`），就从**当前站**下车，换乘到**下一层**（更慢的线），然后重复这个过程。



#### 2. Add (插入)：新建一个 25 号站

这就像要在 20 和 30 之间加一个新站“25”。

#####**步骤 1：找到施工点（找到每一层的前一站）**：先在每一层中找到前驱`prev[i]`

我们需要找到每一个车次中找到$\lt$ 25的**最后一个站点**，也就是说找在每一层中找到25的**前驱节点**。

- 你必须先弄清楚 25 号站该建在哪里，以及它在**每一层**应该**连在谁的后面**。
- 你怎么找？你假装在“查找 25 号站”，用上面 `search` 的方法走一遍。
- 你一路“下车换乘”时，把**你每次下车的站**记在一个小本本上：
  - 在 L3，你从 `head` 出发，下一站 30 > 25，你没动。**记下：L3 的前一站是 `head`**。
  - 在 L2，你从 `head` 出发，下一站 30 > 25，你没动。**记下：L2 的前一站是 `head`**。
  - 在 L1，你从 `head` 坐到 10，又坐到 20。下一站 30 > 25。**记下：L1 的前一站是 `20`**。
  - 在 L0，你从 20 出发。下一站 30 > 25。**记下：L0 的前一站是 `20`**。
- **这个“小本本”，就是我之前说的 `prev` 数组！** 它就是个人话版的“施工图”，告诉你 25 号站在每一层的前一站（前驱）分别是谁。

#####**步骤 2：开始施工（抛硬币）**

###### 抛硬币的具体过程

我们假设最多有`max_level = k`层，那么现在除了$L_0$我们必须要添加这个站点之外。

其余的$L_{i = 1...k}$ 就需要通过根据**概率因子**抛硬币的方式来决定最终哪些层需要添加这个站点。

具体来说，从$L_1$开始抛第一次硬币。

如果硬币正面朝上，那么就为$L_2$抛硬币。

如果还是朝上，则继续为$L_3$抛硬币。

....

假设我们在为$L_m$ 抛硬币的时候，硬币抛出反面了。

那么OK，我们就在$L_{0...m-1}$上增加这个站点。

对于实际运营的列车数量，我们得看看这个$m$值是否已经超过了我们之前的值$m'$。

如果已经超过了，那么需要更新运营车辆数量为$levelCount = m$。

举一个例子

###### **Level 0 (慢车)**：你必须在 $L_0$ 上建 25。

- 你拿出“小本本”看 L0：`prev[0]` 是 `20`。

- 你把 `20` -> `30` 的轨道断开。
- 连上两条新轨道：`20` -> `25` 和 `25` -> `30`。
- L0 施工完毕。

######**决定 25 号站是否设为“快车站”？**

- **抛硬币！** (这就是 `randomLevel()`)
- **正面 (概率 P)**：好，L1 也设为快车站。
  - 看“小本本” L1：`update[1]` 是 `20`。
  - 把 L1 上的 `20` -> `30` 断开。
  - 连上 L1 的轨道：`20` -> `25` 和 `25` -> `30`。
- **再抛一次硬币！**
- **正面 (概率 P)**：好，L2 也设为快车站。
  - 看“小本本” L2：`update[2]` 是 `head`。
  - 把 L2 上的 `head` -> `30` 断开。
  - 连上 L2 的轨道：`head` -> `25` 和 `25` -> `30`。
- **再抛一次硬币！**
- **反面**：停！施工结束。

- 最后，这个新建的 25 号站，在 L0, L1, L2 都有停靠，但在 L3 没有。

**核心套路（增）**：先用“查找”的方式找到每一层的“前一站”并记在“小本本”（`update` 数组）上。然后从 L0 开始“接轨道”，接着“抛硬币”决定要不要在 L1 接、L2 接、L3 接...



#### Erase (删除)：拆除 30 号站

#####**步骤 1：找到 30 号站，并拿到它在每一层的“前一站”**(每一层的前驱)

- 和 `add` 一样，你先“查找 30”，把每一层的“前一站”都记在“小本本”（`update` 数组）上。
  - L3 的前一站是 `head`
  - L2 的前一站是 `10`
  - L1 的前一站是 `20`
  - L0 的前一站是 `20`

#####**步骤 2：拆轨道（“绕过”30 号站）**

我们假设要删除的节点为`node30`.

前面我们已经说了我们使用一个`prev[k]`数组，记录站点30在第$k$层的前驱站点。

那么我们在处理第$k$层的时候，只需要先找到站点30在第$k$层的前驱节点$prev[k]$, 然后将$prev[k]$的后继节点设置为`node30.next[k]` 即可。

- 你从 L0 开始，拿出“小本本”。
- **L0**：前一站是 `20`。`20` 本来指向 `30`，`30` 指向 `35`。
  - 你直接把 `20` 的轨道改成指向 `35` (<code>20.next[0] = 30.next[0]</code>)。
  - 30 号站在 L0 就被“绕”过去了。
- **L1**：前一站是 `20`。`20` 本来指向 `30`，`30` 指向 `40`。
  - 你直接把 `20` 的轨道改成指向 `40`。L1 也绕过去了。
- **L2**：前一站是 `10`。`10` 本来指向 `30`，`30` 指向 `null`。
  - 你直接把 `10` 的轨道改成指向 `null`。L2 也绕过去了。
- **L3**：前一站是 `head`。`head` 本来指向 `30`，`30` 指向 `null`。
  - 你直接把 `head` 的轨道改成指向 `null`。L3 也绕过去了。
- 现在，30 号站虽然还在内存里，但没有任何“轨道”指向它了，它彻底从“地铁系统”中消失了。

**核心套路（删）**：

先用“查找”的方式找到每一层的“前一站”并记在“小本本”（`prev` 数组）上。

然后遍历这个“小本本”，在**每一层**都执行“绕过”操作（<code>前一站.next = 要删的站.next</code>）。



##### 步骤3: 拆完以后你得看看哪些列车需要下线了！

现在假设我们当前最快的线路是$L_x$。

现在我们某个站点给拆了。

###### 先去检查最快的线路$L_x$

**你的TMD第一个动作**：立刻去查**L5**这条线！

**你查什么？** 你TMD就查**“『总站』(head) 在 L5 的下一站TMD是谁？”**（<code>check head.forward[5]</code>）

**如果 `head.forward[5]` 指向 `null`**：

- **卧槽！** 这TMD说明 L5 这条快线，**TMD就只有“30号站”这一个站！**
- 你TMD把它拆了，L5 这条线就TMD废了！它变成了 `head` -> `null`！
- **调度中心（你）TMD立刻下令：“L5 列车下线！”**
- 你TMD马上把“告示牌”上的 5 划掉，改成 4。（<code>currentLevel--</code>）

**如果 `head.forward[5]` 指向 “40号站”**：

- **哦**，TMD原来 L5 上不止一个站。
- 你拆了 30，但 L5 这条线现在是 `head` -> `40` -> ...，TMD**还在跑！**
- **调度中心（你）TMD：“啥也别干！一切照旧！”**
- “告示牌”TMD**保持 `currentLevel = 5`**。你TMD收工。

###### 注意连锁反应！

你TMD千万记住，上面**第一种情况**（`head.forward[5]` 是 `null`）还没完！

你刚把告示牌从 5 改成 4。

**你TMD能下班吗？不能！**

你TMD必须**立刻、马上**，用TMD同样的逻辑**再查一遍 L4**！

1. **告示牌现在是 `currentLevel = 4`。**

2. **你TMD去查 L4**：“『总站』(head) 在 L4 的下一站TMD是谁？”（<code>check head.forward[4]</code>）

3. **如果 `head.forward[4]` 也TMD是 `null`**：

   - **操！** L4 也TMD废了！它TMD（可能）也只有“30号站”！
   - **调度中心（你）TMD：“L4 也TMD给老子下线！”**
   - 你TMD把“告示牌”上的 4 划掉，改成 3。（<code>currentLevel--</code>）

4. **TMD还没完！你TMD继续查 L3！**

5. **你TMD去查 L3**：“『总站』(head) 在 L3 的下一站TMD是谁？”（<code>check head.forward[3]</code>）

6. **如果 `head.forward[3]` 指向 “10号站”**：

   - **好了！** L3 TMD还在跑！

   - **调度中心（你）TMD：“TMD终于查完了！收工！”**

   - 告示牌TMD**最终停在 `currentLevel = 3`**。

     

### 实现原理和步骤

#### `search(target)`

##### 从最高层开始一直向右找到最后一个$\lt target$ 的数字

1. 从 `head` 节点和 `currentLevel`（当前最高层）开始。
2. 在第 `i` 层，向右遍历（`curr = curr.next[i]`），直到 `curr.next[i]` 为 `null` 或者 `curr.next[i].val >= target`。

##### 转移到$L_{i-1}$层

1. 下降到第 `i-1` 层，重复步骤 2。

   `curr = curr.next[i-1]`

2. 当到达 Level 0（最底层）时，`curr` 节点是 `target` 的前驱。

##### 检查$L_0$层：

1. 检查 `curr.next[0]` 是否存在，以及它的值是否等于 `target`。
2. 返回 `curr.next[0] != null && curr.next[0].val == target`。



#### `add(num)`

##### 找到每一层$L_i$中的前驱节点$prev[i]$

1. **找到前驱节点**：如“核心套路”中所述，遍历所有层，填充 `Node[] prev` 数组。

##### 更新跳表的高度

######生成随机层高：投硬币方法的具体实现

调用 `randomLevel()` 方法，得到一个新层高 `newLevel`（例如，范围 0 到 `MAX_LEVEL - 1`）。

在Java中， `random.nextDouble()`返回一个介于`[0...1)`之间的一个小小数，假设其返回的结果是$r$.

那么 $r < 0.5$ 和$r \ge 0.5$的概率是一样的，都是0.5($p=0.5$，也就是概率因子)。

我们使用这个函数来模拟抛金币，假设$r < 0.5$就是硬币的正面。

按照上面说的逻辑，我们通过抛金币获得应该添加钙站点的最大层：

```Java
private int randomLevel() {
        int lvl = 0;
        // random.nextDouble() 返回 [0.0, 1.0) 之间的小数
        // 只要小于 P 并且没达到最大层数，就增加一层
        while (random.nextDouble() < P) {
            lvl++;
            if(lvl == max_level - 1){
              break;
            }
        }
        return lvl;
    }
```

######更新跳表高度：
如果 `newLevel > currentLevel`，意味着新节点是目前最高的节点。
我们就需要初始化一下新增的层的相关数据了。

前面我们提到，我们在每一个节点上定义了一个$next[k]$数组，$node.next[i]$表示节点$node$在第$i$层的下一个节点。

我们假设$L_x(x = [currentLevel + 1... newLevel])$ 是一个新添加的层， 我们需要初始化$L_x$的`head`节点的`next`数组。

需要将 `head` 节点在 `[currentLevel+1...newLevel]` 这些新层上的 `next` 指针（`prev` 数组中的对应位置）设置为 `null`（因为这些新层上目前只有 `head` 和新节点）。

在填充 `prev` 数组时，`head` 会被自动存入这些新层，所以我们只需更新 `currentLevel = newLevel` 即可。

##### 创建和插入新节点

1. **创建新节点**：`Node newNode = new Node(num, newLevel + 1)` (注意层高和索引的关系)。
2. **插入节点（核心）**：遍历 `i` 从 0 到 `newLevel`：
   - `newNode.next[i] = prev[i].next[i]` (新节点指向前驱节点的原下一个节点)
   - `prev[i].next[i] = newNode` (前驱节点指向新节点)



#### `erase(num)`

##### **找到前驱节点**：

和 `add` 一样，填充 `Node[] prev` 数组。

#####**检查目标是否存在**：

- 在 Level 0，`Node nodeToDelete = prev[0].next[0]`。
- 如果 `nodeToDelete == null` 或者 `nodeToDelete.val != num`，说明 `num` 不存在，直接返回 `false`。

#####**执行删除（自底而上的顺序）**：遍历 `i` 从 0 到 `currentLevel`：

###### 从$L_0$开始搜索和删除节点

- 检查在第 `i` 层，`prev[i]` 的下一个节点是否确实是我们要删除的 `nodeToDelete`。
- `if (prev[i].next[i] == nodeToDelete)`:
  - `prev[i].next[i] = nodeToDelete.next[i]` (绕过 `nodeToDelete` 节点)
- `else`:
  - `break;` (如果`prev[i].next[i]` 不是 `nodeToDelete`，说明 `nodeToDelete` 在更高层不存在，无需继续向上删除)

######**更新跳表高度（收缩）**：

- 删除后，如果最高层 `currentLevel` 没有任何节点了（即 `head.next[currentLevel] == null`），则需要降低 `currentLevel`。
- 使用 `while (currentLevel > 0 && head.forward[currentLevel] == null)` 循环来递减 `currentLevel`。

######返回 `true`。

### 实现代码

```Java
class ListNode{
    int val;
    ListNode[] next;

    public ListNode(int val, int max){
        this.val = val;
        this.next = new ListNode[max];
    }
}
class Skiplist {

    private final double P_FACTOR = 0.5d;

    private final int MAX_LEVEL = 10;

    private ListNode head;

    private int currentLevel;

    private Random random;

    public Skiplist() {
        head = new ListNode(-1, MAX_LEVEL);
        currentLevel = 0;
        this.random = new Random();
    }
    
    public boolean search(int target) {
        // int level = currentLevel;
        ListNode curr = head;
        for(int level = currentLevel; level >= 0; level--){
            while(curr.next[level] != null && curr.next[level].val < target){
                curr = curr.next[level];
            }
        }
        curr = curr.next[0];
        return curr != null && curr.val == target;
    }
    
    public void add(int num) {
        //搜索没一层中的前驱节点
        ListNode[] prev = new ListNode[MAX_LEVEL];
        ListNode curr = this.head;
        for(int level = currentLevel; level >= 0; level--){
            while (curr.next[level] != null && curr.next[level].val < num) {
                curr = curr.next[level];
            }
            prev[level] = curr;
        }
        //生成新的层数
        //并且检查是否需要添加新的层
        int newLevel = randomLevel();
        if(newLevel > currentLevel){
            for(int i = currentLevel+1; i <= newLevel; i++){
                prev[i] = head;
            }
        }

        ListNode newNode = new ListNode(num, MAX_LEVEL);
        for(int i = 0; i <= newLevel; i++){
            newNode.next[i] = prev[i].next[i];
            prev[i].next[i] = newNode;
        }
    }
    
    public boolean erase(int num) {
         //搜索没一层中的前驱节点
        ListNode[] prev = new ListNode[MAX_LEVEL];
        ListNode curr = this.head;
        for(int level = currentLevel; level >= 0; level--){
            while (curr.next[level] != null && curr.next[level].val < num) {
                curr = curr.next[level];
            }
            prev[level] = curr;
        }

        ListNode nodeToDelete = prev[0].next[0];
        if(nodeToDelete == null || nodeToDelete.val != num){
            return false;
        }

        for(int i = 0; i <= currentLevel; i++){
            if(prev[i].next[i] == nodeToDelete){
                prev[i].next[i] = nodeToDelete.next[i];
            }else{
                break;
            }
        }

        while (currentLevel > 0 && head.next[currentLevel] == null) {
            currentLevel--;
        }

        return true;
    }

    private int randomLevel(){
        int level = 0;
        while(random.nextDouble() < P_FACTOR){
            level++;

            if(level == MAX_LEVEL - 1){
                break;
            }
        }

        return level;
    }
}
```

### 注意事项

1. **哨兵头节点 (`head`)**：这是实现跳表的关键技巧。它统一了所有操作的边界情况（如在列表最前面插入/删除），使代码大大简化。
2. **`prev` 数组**：`add` 和 `erase` 操作的灵魂。必须正确地找到并存储每一层的前驱节点。
3. **层高管理 (`currentLevel`)**：
   - `add` 时：如果随机层高 `newLevel` > `currentLevel`，需要**增加** `currentLevel`。
   - `erase` 时：如果删除的是最高层的唯一节点，需要**降低** `currentLevel`。
4. **`randomLevel()` 的实现**：`P` 值的选择会影响性能。`P=0.5` 时，空间复杂度平均为 $O(2n) = O(n)$。`P=0.25` 时，空间复杂度平均为 $O(1.33n) = O(n)$，但层高会更低，指针更少，查询常数可能略高。
5. **`erase` 的细节**：题目要求删除*一个*实例。我们的实现（找到第一个匹配的 `num` 并删除）符合这个要求。如果 `num` 有重复，`update` 数组会定位到*第一个* `num` 的前驱。

### 经验总结

- 跳表是平衡二叉搜索树的一个出色替代品。它用**随机化**的简单实现，换取了**平均** $O(\log n)$ 的复杂度。

- 所有操作（`search`, `add`, `erase`）都遵循一个共同的模式：**从高层到低层，从左到右**进行搜索。

- `add` 和 `erase` 的核心是维护 `update` 数组（存储前驱节点）。

- 在面试中，能够清晰地解释跳表的原理（多层索引、随机化）、写出节点定义、以及 `search` 和 `add` 的核心逻辑，通常就足够了。`erase` 的层高收缩是锦上添花。



##**LC 715. Range Module (Range 模块)**

- **简介：** `addRange`, `removeRange`, `queryRange` 均为 O(logN)。
- **设计：** `TreeMap<Start, End>`。这是 `LC 352` 的高级版，`add/remove` 时涉及更复杂的区间合并与分裂。

### 问题要点

你需要设计并实现一个 `RangeModule` 类，它用来追踪和管理『半开区间』`[left, right)`（即包含 `left`，但不包含 `right`）。

这个类需要实现三个核心方法：

1. **`addRange(int left, int right)`**:
   - 添加区间 `[left, right)`。
   - 如果新区间与一个或多个已存在的区间重叠，**必须将它们合并 (Merge)** 成一个或多个连续的大区间。
2. **`queryRange(int left, int right)`**:
   - 查询 `[left, right)` 这个区间，是否**完完整整**地被当前模块中已追踪的区间所**完全覆盖 (Cover)**。
3. **`removeRange(int left, right)`**:
   - 移除 `[left, right)` 这个区间。
   - 这可能会导致一个已存在的区间被**缩短**（砍头或去尾），或者被**分裂 (Split)** 成两个较小的区间。

### 问题的本质和分析 - 动态区间管理

我们来分析一下挑战在哪里：

- **`addRange` (合并)**:
  - 假设已有 `[1, 5)` 和 `[10, 20)`。
  - 此时 `addRange(3, 12)`。
  - `[1, 5)` 与 `[3, 12)` 重叠。
  - `[10, 20)` 与 `[3, 12)` 也重叠。
  - 这三个区间必须被合并成一个**单一的大区间** `[1, 20)`。
- **`removeRange` (分裂)**:
  - 假设已有 `[1, 20)`。
  - 此时 `removeRange(5, 10)`。
  - 这个操作会把原来的区间“挖掉”一块，导致它**分裂**成两个新的区间：`[1, 5)` 和 `[10, 20)`。
- **`queryRange` (覆盖)**:
  - 假设已有 `[1, 5)` 和 `[10, 20)`。
  - `queryRange(2, 4)` -> `true` (被 `[1, 5)` 完全覆盖)。
  - `queryRange(12, 18)` -> `true` (被 `[10, 20)` 完全覆盖)。
  - `queryRange(4, 12)` -> `false` (因为 `[5, 10)` 这个区间是**空**的，没有被覆盖)。

性能瓶颈分析：

如果使用一个简单的 List<Interval> 来存储所有区间，那么每次 add 或 remove 时，你都可能需要遍历整个列表来查找所有重叠的区间。在最坏情况下，这会导致 $O(N)$ 的时间复杂度（其中 $N$ 是列表中区间的数量）。题目中操作次数高达 $10^4$ 次， $O(N)$ 的操作很快会导致超时 (TLE)。

解决方案：

我们需要一个能够快速定位相关区间的数据结构。所有操作都与区间的『端点』相关，并且我们需要维护区间的『顺序』。

这强烈地指向了**平衡二叉搜索树 (Balanced BST)**。在 Java 中，实现这一功能的完美数据结构就是 **`TreeMap`**。
