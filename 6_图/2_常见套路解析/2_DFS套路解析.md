

#DFS 模板（图遍历+连通块）

##核心思想-撞到南墙然后回退

沿着一条路径一直往下走（递归/栈模拟），**直到不能再走为止，然后`回退`并探索其他路径**。

##从DFS的角度来理解递归的工作模式

##DFS能够解决什么样子的问题？

| 应用场景                         | 描述                             |
| -------------------------------- | -------------------------------- |
| 图遍历                           | 访问图中所有节点（连通块）       |
| 拓扑排序                         | 有向无环图（DAG）中的排序问题    |
| 连通分量（Connected Components） | 统计图中联通区域数量             |
| 检测环路（Cycle Detection）      | 判断图中是否存在环（有向或无向） |
| 二分图判定                       | 染色法判断是否可以分为两组       |
| 岛屿数量 / 迷宫路径搜索          | 网格类图问题的通用解法           |
| 剪枝搜索 / 状态空间图探索        | 回溯、排列组合等问题             |

## DFS的核心思想 - 一路走到黑，不撞南墙不回头

想象你在走一个迷宫，为了找到出口，你采用以下策略：

- 在某个路口，随便选择一条**没走过**的路，然后一直走下去。
- 每经过一个路口，你都在墙上做一个**标记**，表示“这里我来过了”。
- 当你走到一个死胡同（前面没有路可走），或者前方的路都是你**已经标记过**的，你就**退回到上一个路口**。
- 在上一个路口，你再尝试选择一条**其他没走过的路**继续前进。
- 重复这个过程，直到你探索完所有从起点能到达的路口。

这个策略就是深度优先。

它会沿着一条路径不断深入，直到无法再前进时，才会**回溯**（Backtracking）到上一个节点，去探索其他的可能性。

与它对应的是广度优先搜索（BFS），BFS 像是在水面上扔一块石头，波纹从中心开始**一层一层向外扩散**，它会先访问完所有离起点最近的节点，然后再访问次近的节点。

## DFS和BFS的实现比较

| 特性           | 深度优先搜索 (DFS)                       | 广度优先搜索 (BFS)                   |
| -------------- | ---------------------------------------- | ------------------------------------ |
| **探索顺序**   | 深入探索一个分支，直到尽头再回溯         | 逐层探索，先探索完所有邻居节点       |
| **数据结构**   | 栈 (Stack) - 无论是显式还是隐式          | 队列 (Queue)                         |
| **应用**       | 拓扑排序、寻找路径、检测环、寻找连通分量 | 寻找最短路径（无权图）、社交网络分析 |
| **空间复杂度** | O(H)，H是最大深度（递归）                | O(W)，W是最大宽度                    |

## DFS 的关键要素

- `访问标记`: 防止重复访问
- `递归/栈结构`：控制访问顺序
- `回溯机制`： 深入一个分支到底，然后退回再探索别的路径

### 图的表示

#### 邻接表

最常用、最高效的表示方式。

它使用一个数组或哈希表，每个索引对应一个节点，其值是一个列表，包含了该节点所有相邻的节点。

对于稀疏图（边的数量远小于节点数量的平方）来说，空间和时间效率都很高。

####邻接矩阵

一个二维数组，`matrix[i][j] = 1` 表示节点 `i` 和 `j` 之间有边。

对于稠密图很有效，但空间复杂度较高 ($O(V^2)$，V是顶点数)。

#### 隐式图

在某些问题中，图不是显式给出的，比如棋盘、迷宫或数学问题。

节点和边的关系由问题的规则定义。

### 栈 - 递归栈或者显式栈

#### 递归栈

函数调用本身就是一个栈（调用栈，Call Stack）。

每次递归调用一个函数，就是将该函数的上下文（参数、局部变量等）压入栈中。

当函数返回时，就是出栈。

这是最自然、最简洁的实现方式。

#### 迭代实现 -显式栈

手动创建一个栈（如 `stack` 数据结构），通过循环来模拟递归的压栈和出栈过程。

当图的深度非常大时，可以避免递归导致的栈溢出。

### 已访问记录 - 至关重要的要素

这是 **DFS 中至关重要**的元素，主要有两个目的：

#### 避免死循环

在包含环的图中，如果没有“已访问”记录，DFS 会在环里无限循环下去，永远无法结束。

#### 避免重复处理

确保每个节点只被访问和处理一次，提高算法效率。

 通常可以使用一个布尔数组 `visited[node]` 或一个哈希集合 `visited_set` 来记录。

## DFS的实现原理和步骤

### 基于递归的过程

#### 实现过程步骤

1. 创建一个 `visited` 集合或数组，用于记录已访问的节点。

2. 从一个起始节点 `start_node` 开始。

3. 定义一个 `dfs(node)` 函数： 

   a. 将当前 `node` 标记为已访问 (`visited[node] = true`)。

   b. **处理当前节点**（例如，打印节点值，将其添加到结果列表中等）。这被称为“前序遍历”处理。

    c. 遍历当前 `node` 的所有邻居 `neighbor`。 

   d. 如果 `neighbor` **没有被访问过**，则对 `neighbor` 递归调用 `dfs(neighbor)`。

#### 实现代码套路模板

```Java
function DFS(graph, start_node):
    visited = new set() // 1. 创建 visited 集合

    function dfs_recursive(u):
        visited.add(u) // a. 标记为已访问
        print(u) // b. 处理当前节点

        for each neighbor v of u in graph: // c. 遍历邻居
            if v is not in visited: // d. 如果邻居未被访问
                dfs_recursive(v) //    递归调用,注意并不加入visited

     dfs_recursive(start_node) // 2. 从起始节点开始
```

### 基于迭代和显式栈

使用一个显式的栈来模拟递归过程。

#### 实现过程步骤

1. 创建一个 `visited` 集合或数组。

2. 创建一个栈 `stack`。

3. 将起始节点 `start_node` 压入栈中。

4. 当栈不为空时，循环执行以下操作： 

   a. 从栈顶弹出一个节点 `u`。 

   b. **如果 `u` 已经被访问过，则跳过**（这步很重要，因为一个节点可能被多次压入栈中）。

    c. 将 `u` 标记为已访问。

    d. **处理当前节点 `u`**。 

   e. 遍历 `u` 的所有邻居 `neighbor`，将**未被访问过的** `neighbor` 压入栈中。

#### 实现代码套路模板

```Java
function DFS_iterative(graph, start_node):
    visited = new set()
    stack = new Stack() // 2. 创建栈

    stack.push(start_node) // 3. 起始节点入栈,注意并不加入visited

    while stack is not empty: // 4. 循环
        u = stack.pop() // a. 弹出节点

        if u in visited: // b. 如果已访问，跳过
            continue

        visited.add(u) // c. 标记为已访问
        print(u) // d. 处理当前节点

        // e. 将邻居压入栈
        // 注意：通常反向遍历邻居，可以使迭代的访问顺序与递归版本一致
        for each neighbor v of u in reverse(graph.neighbors(u)):
             if v is not in visited:
                 stack.push(v)
```

## 使用 DFS的注意事项

### 处理环路和避免死循环 (Handling Cycles) - Visited

`visited` 集合是防止在有环图中无限循环的关键。

每次访问新节点前，**必须**检查它是否已经被访问过。忘记检查是初学者最常犯的致命错误。

### 处理非连通图

#### DFS只能遍历当前联通分量

从单个节点开始的 DFS 只会遍历该节点所在的**连通分量**。如果图有多个互不相连的部分，你需要一个额外的循环来确保所有节点都被访问。

#### 解决方法：遍历每一个未被访问的点并DFS

遍历图中的每一个节点，如果该节点尚未被访问，就以它为起点开始一次新的 DFS 遍历。

####代码套路模板

```Python
visited = set()
for node in all_nodes:
    if node not in visited:
        dfs(node) # 开始一次新的 DFS
```

### 递归深度和栈溢出

####**问题**：图太深导致递归栈溢出

当图非常深（例如，一条很长的链）时，递归实现可能会导致函数调用栈过深，从而引发“栈溢出”错误 (Stack Overflow Error)。

####**解决方案** - 迭代+显式栈：

在这种情况下，必须使用迭代实现。迭代实现用堆内存（手动创建的 `stack` 对象）来代替调用栈，避免了系统调用栈大小的限制。

### 节点的处理顺序： 前序 VS 后序

#### 前序处理：进入函数就将当前节点标记visited

在进入一个节点（即刚标记为 visited）后，立即处理它，然后再去探索它的邻居。上面给出的标准实现就是前序处理。

#### 后序处理： 邻居都已经被探索完毕再标记

在一个节点的所有邻居都已经被探索完毕，即将回溯离开该节点时，再处理它。这在递归中很容易实现，只需将处理语句放在 for 循环之后。

```Java
function dfs_post_order(u):
    visited.add(u)
    for each neighbor v of u:
        if v not in visited:
            dfs_post_order(v)
    // 所有子节点都已访问完毕，现在处理 u
    print(u) // 这就是后序处理
```

### DFS 配合回溯的使用

#### DFS 是许多回溯算法的基础

在解决组合、排列、子集、棋盘游戏（如N皇后、数独）等问题时，DFS 的“探索-回溯”模式被用来系统地搜索所有可能的解。

#### 回溯中的关键要素：Path

在回溯问题中，除了 `visited` 状态，通常还需要维护一个 `path` 或 `current_state`。

进入下一层递归时更新状态，回溯（函数返回）时需要**撤销**对状态的修改，以便探索其他分支。

## 图的DFS和二叉树的DFS过程比较

### 相同点：递归过程都需要传入当前节点

不管是哪一种DFS，都需要传入当前节点。

### 不同点

#### 起始节点选择

- 二叉树的DFS必须从跟节点开始
- 图的DFS过程可以从任意节点开始

#### 实现复杂度

- **二叉树：** 无需处理环路，递归实现简单，代码逻辑清晰
- **图：**需要额外维护访问标记数组或者集合，避免无限循环递归，实现更复杂

## 先检查解还是先检查剪枝？

这需要根据DFS的风格，本轮递归处理的是当前节点还是当前节点的邻居节点？

### 风格一： 本轮递归处理当前节点 - 先剪枝再检查解

#### 先检查剪枝，再检查解

这种风格的函数是 `dfs(currentNode, ...)`，它的任务是处理 `currentNode` 本身。

典型的例子是走迷宫，`dfs(row, col)` 要处理 `(row, col)` 这个格子。

在这种模式下，顺序是：**先检查剪枝，再检查解。**

####**为什么？**

 因为你必须先确保你踏入的 `currentNode` 是一个**有效位置**（没越界、不是障碍物），然后才能判断这个有效位置是不是终点（解）。你不可能在一个无效的位置上找到解。

#### 代码示例

```Java
void dfs(Node currentNode, Path path) {

    // 1. 剪枝 (Pruning)
    // 检查当前节点本身是否合法。如果不合法，谈不上是解，直接返回。
    if (isOutOfBounds(currentNode) || isObstacle(currentNode) || isVisited(currentNode)) {
        return; 
    }

    // 既然节点有效，就将它加入路径（处理当前节点）
    path.add(currentNode);
    markAsVisited(currentNode);

    // 2. 检查是否找到解 (Solution Check)
    // 在节点合法的前提下，再判断它是不是我们要找的终点。
    if (isDestination(currentNode)) {
        processSolution(path);
        // 根据是否要找所有解，决定是否 return
        // 注意：这里需要回溯，以便找到其他路径
    }

    // 3. 向邻居递归
    for (Node neighbor : currentNode.getNeighbors()) {
        dfs(neighbor, path);
    }
    
    // 4. 回溯
    path.removeLast();
    markAsUnvisited(currentNode);
}
```

### 风格二： 本轮递归处理当前节点的邻居节点 - 先检查解，再检查剪枝

#### **先检查解，再检查剪枝。**

这种风格的函数是 `dfs(path, ...)`，它的任务是基于**已经合法的 `path`**，去探索下一步该怎么走。

典型的例子是组合总和，`dfs(currentSum, currentList)` 是基于当前的和与列表，去添加下一个数字。

在这种模式下，顺序是：**先检查解，再检查剪枝。**

#### 为什么？

因为传入函数的 `path` 本身是合法的，所以你首先要判断这个**已经合法的路径**是不是已经满足了题目要求（是不是一个解）。

如果它已经是解了，就没必要再往下走了（剪枝）。

如果还不是解，你再判断这条路还有没有继续走下去的必要（剪枝）。

#### 代码套路

```Java
void dfs(Path path, Target target) {
    
    // 1. 检查是否找到解 (Solution Check)
    // 传入的 path 是合法的，所以先看它是不是已经构成了一个完整的解。
    if (isSolution(path, target)) {
        processSolution(path);
        return; // 找到了，当前分支结束。
    }

    // 2. 剪枝 (Pruning)
    // 如果还没找到解，再判断当前路径是否已经不可能通向解了。
    // 例如，和已经超了，或者长度已经超了。
    if (isImpossible(path, target)) {
        return; // 此路不通，当前分支结束。
    }

    // 3. 循环处理“邻居”，并向下一层递归
    for (Candidate nextMove : getNextValidMoves(path)) {
        path.add(nextMove);
        dfs(path, updatedTarget);
        path.removeLast(); // 回溯
    }
}
```



## 怎样使用DFS解决计数的问题？

### 套路一： DFS返回累计所有可行路径的数量

这是最常见的计数模式。

DFS函数不再是 `void` 类型，而是返回一个数字（通常是 `int` 或 `long`），表示**从当前状态出发，能够达成目标的路径总数**。

#### 核心思想： 节点的解 = 所有子节点的解之和

- 求路径数量的情况下：当前节点的解 = 所有子节点的解之和

- 求最大路径长度的情况下：当前节点的解 = `max(所有子节点的解)` + 1

#### 标准流程和套路

##### 函数定义：`int dfs(currState,...)`

定义一个返回`int`类型的结果，代表从当前节点出发所能找到的解的数量。

##### 递归终止条件

###### 找到一个解

如果当前状态满足了题目要求的最终条件，说明找到了一条完整的路径，返回`1`

###### 遇到无效状态

如果当前状态是思路、越界、或者步满足约束条件，说明从这个节点出发不可能找到解，返回0；

##### 递归与回溯

- 初始化一个计数器`count = 0`
- 遍历当前状态下所有可能的下一步或者邻居
- 对于每一个下一步的状态，调用递归函数`count += dfs(curr,....)`
- 将所有子路径返回的结果都累加到`count`中

##### 返回值

返回累加后的 `count`。

#### 示例： 机器人走方格

##### 问题要点

一个机器人在 `m x n` 网格的左上角 `(0, 0)`，要走到右下角 `(m-1, n-1)`。它每次只能向下或向右移动。问有多少条不同的路径？

##### 思路分析

- `dfs(row, col)` 的返回值：从 `(row, col)` 走到终点的路径数。
- Base Case (找到解): 如果 `row == m-1 && col == n-1`，返回 `1`。
- Base Case (剪枝): 如果 `row >= m || col >= n`（越界），返回 `0`。
- 递归关系: `dfs(row, col) = dfs(row+1, col) + dfs(row, col+1)`。

##### 代码套路模板

```Java
public int dfs(int row, int col){
  //检查解的情况
  if(row == m-1 && col == n-1){
    return 1;
  }
  
  //越界的情况，剪枝
  if(row < 0 || row >= m){
    return 0;
  }
  if(col < 0 || col >= n){
    return 0;
  }
  
  //累加从所有邻居节点出发的路径数量
  int count = 0;
  for(int[] dir : dirs){
    int nextRow = row + dir[0];
    int nextCol = col + dir[1];
    
    count += dfs(nextRow,nextCol);
  }
  
  return count;
}
```

#### 记忆化优化之后的DFS计数套路

##### 核心思想：定义一个数组保存每个节点到终点的路径数量

记忆化优化的核心思想可以用一句话概括：**“记录并重用之前计算过的结果，避免重复劳动。”**

在矩阵的例子中，我们将会定义一个二维数组`dp[row][col]`，表示节点`[row,col]`到终点之间的路径的数量。

这个二维数组同时也会起到`visited`的作用，避免我们重复访问和计算。

想象一个场景：你的一个朋友是数学天才，你经常问他一些复杂的计算题。

- **没有记忆化**：你每次问他“5的阶乘是多少？”，他都从 `5*4*3*2*1` 重新算一遍。过了一会儿，你又问他“5的阶乘是多少？”，他又傻乎乎地从头再算一遍。
- **有记忆化**：第一次问他“5的阶乘是多少？”，他计算出结果是120。然后他非常聪明地**在一张草稿纸上写下：“5! = 120”**。下次你再问他同样的问题，他看一眼草稿纸，立即告诉你答案，省去了所有计算时间。

这张“草稿纸”就是计算机科学中的**“缓存（Cache）”**或**“备忘录（Memo）”**。我们通过消耗一点点存储空间（草稿纸），来节约大量的计算时间。这就是典型的“空间换时间”。

##### 记忆化优化之后的DFS计数

```Java
//定义一个全局的（通常可能会被称为memo,但是本质上就是动态规划）
int[][] dp = new int[m][n];
public int dfs(int row, int col){
  if(dp[row][col] > 0) return dp[row][col];
  if(row == m-1 && col == n-1){
    return 1;
  }
  
  if(row < 0 || row >= m){
    return 0;
  }
  if(col <0 || col >= n){
    return 0;
  }
  
  int count = 0;
  for(int[] dir : dirs){
    int nextRow = row + dir[0];
    int nextCol = col + dir[1];
    
    count += dfs(row,col);
  }
  dp[row][col] = count;
  
  return count;
}
```

### 套路二： 使用全局变量计数，返回值为`void`

当你只需要找到所有符合条件的**完整解**的数量时（例如，组合、排列问题），有时使用一个全局变量或成员变量来计数会更直观。

#### 核心思想 - 探索到完整的解时+1

DFS只负责探索，当探索到一个完整的解时，计数器加一。

#### 标准套路流程

##### 定义void DFS:`void dfs(curr)`

`void dfs(currentState, ...)`

##### 定义全局计数器

`int count = 0`

##### 递归终止条件

###### 找到一个解时更新计数器

如果当前状态是一个完整的解，则 `count++`，然后 `return`。

###### 无效状态剪枝

如果当前状态不合法，直接 `return`。

##### 递归和回溯

遍历所有可能的“下一步”。

修改状态，然后调用 `dfs(nextState, ...)`。

回溯状态。

#### 代码套路模板 - N皇后问题

在 `N x N` 的棋盘上放 `N` 个皇后，要求所有皇后不能互相攻击。问有多少种不同的摆法？

##### 问题分析

- 我们一行一行地放皇后。`dfs(row, ...)` 的任务是在第 `row` 行找一个合法的位置放皇后。
- Base Case (找到解): 如果 `row == N`，说明 `N` 个皇后都成功放下了，这是一个完整的解，`count++`。
- 剪枝: 在为第 `row` 行选择列的时候，要检查这个位置是否会被之前已放置的皇后攻击。

##### 代码套路模板

```Java
public class NQueensCounter {

    private int count; // 成员变量，用于全局计数
    private boolean[] cols; // 记录列是否被占用
    private boolean[] diag1; // 记录主对角线是否被占用 (r - c)
    private boolean[] diag2; // 记录副对角线是否被占用 (r + c)

    public int totalNQueens(int n) {
        count = 0;
        if (n <= 0) return 0;
        
        cols = new boolean[n];
        diag1 = new boolean[2 * n - 1];
        diag2 = new boolean[2 * n - 1];
        
        dfs(0, n);
        return count;
    }

    private void dfs(int row, int n) {
        // 1. 递归终止条件 - 找到解
        if (row == n) {
            count++;
            return;
        }

        // 2. 遍历所有下一步（在当前行的每一列尝试放皇后）
        for (int col = 0; col < n; col++) {
            // 3. 剪枝检查
            int d1 = row - col + n - 1;
            int d2 = row + col;
            if (cols[col] || diag1[d1] || diag2[d2]) {
                continue; // 此位置不合法
            }

            // 做出选择
            cols[col] = true;
            diag1[d1] = true;
            diag2[d2] = true;

            // 递归到下一行
            dfs(row + 1, n);

            // 回溯，撤销选择
            cols[col] = false;
            diag1[d1] = false;
            diag2[d2] = false;
        }
    }

    public static void main(String[] args) {
        NQueensCounter solver = new NQueensCounter();
        System.out.println("Number of solutions for 4-Queens: " + solver.totalNQueens(4)); // Output: 2
        System.out.println("Number of solutions for 8-Queens: " + solver.totalNQueens(8)); // Output: 92
    }
}
```



##Leetcode 133:克隆图-基于DFS实现

### 问题要点

**输入**: 一个连通无向图中的任意一个节点的引用 (`Node* node`)。

**输出**: 整个图的**深度拷贝 (Deep Copy)**。

**深度拷贝定义**:

- 必须创建一个全新的图，图中所有节点都必须是新创建的 (`new Node(...)`)。
- 新图中节点的 `val` 值必须与原图中对应节点的值相同。
- 新图的拓扑结构（即节点间的连接关系）必须与原图完全一致。 4_  **数据结构**: 节点 `Node` 包含一个整数 `val` 和一个邻居列表 `List<Node> neighbors`。

**核心挑战**: 图中可能包含**环 (Cycle)**。如果处理不当，遍历会陷入无限循环。

##### 问题的本质和分析

###### 图的遍历

为了复制整个图，我们必须从给定的起始节点出发，访问到图中的每一个节点和每一条边。

###### 维护映射关系

遍历原图并创建新节点时，我们需要一种机制来记录“原图中的某个节点”与“新图中对应的新建节点”之间的一一对应关系。

如果我们不维护这种映射关系，会遇到两个主要问题：

- **重复创建节点**: 当多条路径都指向同一个节点 `A` 时，我们可能会为 `A` 创建多个副本，这破坏了图的结构。
- **无法处理环**: 遇到环时，我们会沿着环路不停地遍历下去，导致无限递归（DFS）或无限循环（BFS），最终造成栈溢出或超时。

因此，最关键的解决方案是使用一个哈希表（`HashMap` 或 `Hashtable`）来存储这种映射关系：`Map<OriginalNode, ClonedNode>`。这个哈希表同时起到了“备忘录”和“已访问集合”的作用。

**备忘录**: 当我们需要一个原节点的克隆体时，先查表。如果表中已有，直接返回，无需重复创建。

**已访问集合**: 哈希表的 `keySet` 天然地记录了所有已经访问过并为其创建了克隆体的原图节点，从而避免了重复遍历和死循环。

##### 模式套路匹配

这个题目完美匹配了**图的遍历**算法模式，特别是应用于需要跟踪节点状态或对应关系的场景。

- **算法模式**: 图的遍历（Graph Traversal）。
- **具体算法**:
  - 深度优先搜索 (Depth-First Search, DFS)
  - 广度优先搜索 (Breadth-First Search, BFS)
- **辅助数据结构**: 哈希表 (Hash Map) 用于存储已访问节点和它们的克隆。

这个 **“遍历 + 哈希表”** 的组合是解决涉及环路或共享节点的图、链表（例如：[LeetCode 138. 复制带随机指针的链表](https://leetcode.cn/problems/copy-list-with-random-pointer/)）等复杂数据结构复制问题的通用套路。



##### 核心思想 - HashMap记录原节点和克隆节点之间的映射

核心思想是利用一个 `Map<Node, Node> visited` 来打通原图和克隆图之间的桥梁。

- 遍历图中所有的节点，构建它们的副本
- 图中可能存在环或者重复访问的节点，需要用一个`HashMap<OriginalNode,ClonedNode>`来记录**已经克隆的节点**,同时这个map作为**visited**，避免死循环和重复克隆

**通用流程：**

1. 创建一个哈希表 `visited`，用于存储 `(原节点 -> 克隆节点)` 的映射。
2. 从给定的起始节点 `node` 开始遍历。
3. 在遍历过程中，第一次遇到一个原图节点 `originalNode` 时：
   -  a. 创建它的克隆节点 `clonedNode`，并将 `val` 复制过去。 
   -  b. 将这个映射关系存入哈希表：`visited.put(originalNode, clonedNode)`。 
   -  c. 继续处理 `originalNode` 的邻居。
4. 当需要建立克隆节点之间的连接时（例如，为 `clonedNodeA` 添加邻居 `clonedNodeB`），`clonedNodeB` 应该通过查找 `originalNodeB` 在 `visited` 哈希表中的映射来获得。如果 `originalNodeB` 对应的克隆还不存在，就递归或迭代地去创建它。



##### 回顾DFS套路

###### DFS：沿着一条路径走下去，直到不能走的时候回退

- 将当前节点标记为`visted`
- 对于当前节点的每一个邻居节点
  - 如果邻居节点没有被标记为`visited`, 递归调用DFS

```java
void dfs(int node,Set<Integer> visited, List<List<Integer>> graph){
  visited.add(node);
  for(int neigbor: graph.get(node)){
    if (!visited.contains(neigbor)){
      dfs(neigbor,visited,graph);
    }
  }
}
```

##### 实现原理和步骤

######方法一： 基于DFS 递归实现

**主函数 `cloneGraph(node)`**:

- 处理边界情况：如果输入的 `node` 为 `null`，直接返回 `null`。
- 创建一个全局（或通过参数传递）的哈希表 `visited`。
- 调用 DFS 辅助函数 `dfs(node, visited)` 并返回结果。

**辅助函数 `dfs(originalNode, visited)`**:

- **终止/查找条件**: 检查 `visited` 中是否已经包含了 `originalNode`。如果包含，说明该节点已经创建了克隆，直接返回 `visited.get(originalNode)`。这是处理环和避免重复创建的关键。

- **创建克隆**: 如果 `visited` 中没有 `originalNode`，说明是第一次访问。

  a. 创建一个新的节点 `clonedNode = new Node(originalNode.val)`。

  b. **（关键步骤）** 立刻将新创建的克隆节点存入 `visited` 中：`visited.put(originalNode, clonedNode)`。

  **这一步必须在递归调用其邻居之前完成**，否则在遇到环时，无法在 `visited` 中找到环的起始节点，导致无限递归。

- **递归构建邻居**: 

  a. 遍历 `originalNode` 的所有邻居 `neighbor`。

   b. 对每一个 `neighbor`，递归调用 `dfs(neighbor, visited)`。

  这将返回 `neighbor` 对应的克隆节点。 

  c. 将返回的克隆邻居节点添加到 `clonedNode` 的邻居列表中。

- **返回结果**: 返回当前创建的 `clonedNode`。

######方法二： 基于BFS实现

BFS 使用队列来逐层遍历图。

1. **主函数 `cloneGraph(node)`**:
   - 处理边界情况：如果输入的 `node` 为 `null`，返回 `null`。
   - 创建一个哈希表 `visited` 和一个队列 `queue`。
   - **初始化**: a. 克隆起始节点 `clonedStartNode = new Node(node.val)`。 b. 将起始节点和它的克隆放入 `visited`：`visited.put(node, clonedStartNode)`。 c. 将起始的原图节点 `node` 放入队列 `queue` 中，用于启动 BFS 过程。
2. **迭代过程**:
   - 当 `queue` 不为空时，循环执行： a. 从队列中取出一个原图节点 `currentNode = queue.poll()`。 b. 通过 `visited` 获取其对应的克隆节点 `clonedCurrentNode = visited.get(currentNode)`。 c. 遍历 `currentNode` 的所有邻居 `originalNeighbor`。 d. 对于每个 `originalNeighbor`: i.   检查 `visited` 中是否包含 `originalNeighbor`。 ii.  **如果未访问过**: * 创建一个新的克隆邻居 `clonedNeighbor = new Node(originalNeighbor.val)`。 * 将映射关系存入 `visited`：`visited.put(originalNeighbor, clonedNeighbor)`。 * 将这个 `originalNeighbor` 加入队列 `queue`，以便后续处理它的邻居。 iii. **建立连接**: 从 `visited` 中获取 `originalNeighbor` 对应的克隆节点，并将其添加到 `clonedCurrentNode` 的邻居列表中。`clonedCurrentNode.neighbors.add(visited.get(originalNeighbor))`。

##### 代码实现

###### 基于DFS实现

```Java
import java.util.HashMap;
import java.util.Map;
import java.util.List;
import java.util.ArrayList;

class Solution {
    // 使用一个 HashMap 来存储已经访问过的节点和它们对应的克隆节点
    // Key: 原始图中的节点, Value: 克隆图中的对应节点
    private Map<Node, Node> visited = new HashMap<>();

    public Node cloneGraph(Node node) {
        if (node == null) {
            return null;
        }

        // 如果节点已经被访问过（即已经为它创建了克隆）
        // 直接从 HashMap 返回克隆节点，避免无限循环
        if (visited.containsKey(node)) {
            return visited.get(node);
        }

        // 创建当前节点的克隆。注意，邻居列表此时为空
        Node clonedNode = new Node(node.val, new ArrayList<>());

        // 将原始节点和克隆节点放入 visited Map 中
        // 这一步至关重要，必须在递归其邻居之前完成
        visited.put(node, clonedNode);

        // 遍历原始节点的所有邻居
        for (Node neighbor : node.neighbors) {
            // 递归地克隆邻居节点，并将返回的克隆邻居添加到当前克隆节点的邻居列表中
            clonedNode.neighbors.add(cloneGraph(neighbor));
        }

        return clonedNode;
    }
}
```

###### 基于BFS实现

```Java
import java.util.HashMap;
import java.util.Map;
import java.util.Queue;
import java.util.LinkedList;
import java.util.List;
import java.util.ArrayList;

class Solution {
    public Node cloneGraph(Node node) {
        if (node == null) {
            return null;
        }

        Map<Node, Node> visited = new HashMap<>();
        Queue<Node> queue = new LinkedList<>();

        // 1. 克隆起始节点并初始化
        Node clonedStartNode = new Node(node.val, new ArrayList<>());
        visited.put(node, clonedStartNode);
        queue.add(node);

        // 2. BFS 遍历图
        while (!queue.isEmpty()) {
            // 取出原始图中的节点
            Node currentNode = queue.poll();
            
            // 遍历其所有邻居
            for (Node originalNeighbor : currentNode.neighbors) {
                // 如果邻居还没有被访问过（即还没为它创建克隆）
                if (!visited.containsKey(originalNeighbor)) {
                    // 创建克隆
                    Node clonedNeighbor = new Node(originalNeighbor.val, new ArrayList<>());
                    // 存储映射关系
                    visited.put(originalNeighbor, clonedNeighbor);
                    // 将原始邻居入队，以便后续处理
                    queue.add(originalNeighbor);
                }
                
                // 3. 建立克隆节点之间的连接
                // 从 visited map 中获取原始邻居对应的克隆邻居
                // 然后将其添加到当前克隆节点的邻居列表中
                visited.get(currentNode).neighbors.add(visited.get(originalNeighbor));
            }
        }

        return clonedStartNode;
    }
}
```

#####注意事项

**空图处理**: 第一个判断 `if (node == null)` 是必要的边界条件。

**哈希表的关键作用**: 必须深刻理解 `visited` 哈希表既是“备忘录”也是“已访问标记”。没有它，算法将失败。

**DFS 的关键点**: 在 DFS 递归实现中，`visited.put(originalNode, clonedNode)` 必须在递归调用 `cloneGraph(neighbor)` **之前**执行。这是为了在图中存在环 `A -> B -> A` 时，从 B 返回 A 时能立刻在 `visited` 中找到 A 的克隆体，从而中断递归，否则会无限递归。

**BFS 的关键点**: 在 BFS 中，节点入队时就应该创建克隆并放入 `visited` map，或者至少在第一次从邻居中看到它时这样做。这确保了每个节点只被处理和入队一次。

**深拷贝 vs 浅拷贝**: 要清楚地知道，`new Node(...)` 和 `new ArrayList<>()` 是必须的，不能直接复用原图的任何对象引用（除了 `val` 这种基本类型的值）。

##### 经验总结

**识别模式**: 遇到复制复杂数据结构（图、带随机指针的链表等）的问题时，第一时间想到 **“遍历 + 哈希表”** 的模式。哈希表用来解决节点共享和环路问题。

**选择遍历方式**: DFS 和 BFS 都可以解决这个问题，它们的时间和空间复杂度相同（O(N+E)，其中 N 是节点数，E 是边数）。

- **DFS (递归)**: 代码通常更简洁，逻辑更直观，但如果图的深度过大，可能导致栈溢出（在 LeetCode 场景下通常不会）。
- **BFS (迭代)**: 使用队列，没有栈溢出风险，对于寻找最短路径等问题是标准解法，代码结构性也很强。

**触类旁通**: 理解了这道题，就掌握了处理类似图问题的核心技巧。例如，你需要判断图中是否有环、寻找两个节点间是否存在路径、或者对图中的每个节点执行某种操作而又不能重复时，这个模式都是基础。它是图论算法的基石之一。

##Leetcode 399: 除法求值

### 问题要点

1. **输入**:
   - 一个字符串对的列表 `equations`，其中 `equations[i] = [Ai, Bi]` 表示一个方程。
   - 一个浮点数数组 `values`，其中 `values[i]` 是方程 `Ai / Bi` 的结果。
   - 一个字符串对的列表 `queries`，其中 `queries[j] = [Cj, Dj]` 是你需要求解的查询。
2. **输出**:
   - 一个浮点数数组 `ans`，其中 `ans[j]` 是查询 `Cj / Dj` 的结果。
3. **特殊情况**:
   - 如果查询的结果无法从已知方程中推导出来（例如，变量不存在或变量之间没有路径），则结果为 `-1.0`。
   - 输入保证合法，不会有除以零的情况，也不会有矛盾的方程。

###问题的本质和分析 - 求图中两个端点之间的路径的积

#### 模型转换和分析

从数学的角度，如果A/B=x, B/C=y, C/D=z， 那么A/D = `A/B  * B/C * C/D` 

- `equations[i] = [Ai, Bi]`
- `values[i]` = `equation[i][0]/equation[i][1]`
- 可以将equations和values 转化为一个`有向图`（注意每一条边都是双向的，并且权重互为倒数）
  - equations是图的**边集合**, `equations[Ai,Bi]` 表示一条从**点Ai到Bi的边**
  - values是边的**权重集合**,`values[i]`表示边`[Ai,Bi]`的权重
  - 加入某个边是`[a,b]`, 对应的value是`c`,那么边`[b,a]`对应的value就是`1/c`;
  - 求`Cj/Dj`，也就是求从`Cj到Dj之间的`所有的边的**乘积**

####如何使用DFS计算两个端点的路径权重累计值（累加/乘..）？

这是一个很重要的套路。

```Java
    // 你的 dfs 方法已经完全正确，无需修改
    private double dfs(Map<String, List<Edge>> graph, Set<String> visited, String curr, String target, double accProduct) {
        visited.add(curr);
        if (Objects.equals(curr, target)) {
            return accProduct;
        }
        List<Edge> edges = graph.getOrDefault(curr, new ArrayList<>());
        for (Edge edge : edges) {
            String next = edge.to;
            if (!visited.contains(next)) {
                double val = edge.val;
                double product = dfs(graph, visited, next, target, accProduct * val);
                if (product != -1.0) {
                    return product;
                }
            }
        }
        return -1.0;
    }
```



###核心思想和套路 - 图的DFS

####构建图：将equations 和 values转化为`有向图Map<num1,Map<num2,value>>`

- equations是图的边集合
- values是边的权重集合

需要注意的是每一个`equations[i]和values[i]`可以创建**两个有向边**

- `A -> B`: `val = values[i]`
- `B -> A`: `val = 1/values[i]`

####采用dfs的方式从计算每一个query[DFS 套路]

- 找到起始节点和结束节点
- 如果如果起始节点和输入节点是同一个节点，返回1.0
- 如果起始节点不是输入节点
  - 如果起始节点不能够达到结束节点，返回-1.0
  - 如果起始节点能够到达结束节点，返回中途所有边的**乘积**

###关键数据结构

####构建图（邻接表）`Map<current,Map<neighbor, weight>>`

将问题建模成为有向图的邻接表形式

```java
  
public Map<String,Map<String, Double>>  getGraph(List<List<String>> equations,double[] values){
           Map<String,Map<String,Double>> graph = new HashMap<>();
            for (int i=0;i<equations.size();i++){
               //对于每一个equation和value， 生成两个边
             
               List<String> equation = equations.get(i);
               String num1 = equation.get(0);
               String num2 = equation.get(1);
               Double value = values[i];
               // Edge 1 = [num1,num2], weight1 = value
               Map<String, Double> neighbors1 = graph.get(num1);
               if (neighbors1 == null){
                neighbors1 = new HashMap<>();
                graph.put(num1, neighbors1);
               }
               neighbors1.put(num2, value);
								
							 // Edge2=[num2,num1],Weight2 = 1/value
               Map<String,Double> neighbors2 = graph.get(num2);
               if (neighbors2 == null){
                neighbors2 = new HashMap<>();
                graph.put(num2, neighbors2);
               }
               neighbors2.put(num1, 1.0d/value);
            }
            return graph;
    }
```

####Visited Set

用来在dfs过程中记录**已经计算过的节点**，dfs标配数据结构

###算法实现原理

####**. 初始化**

- 创建一个图的数据结构 `Map<String, Map<String, Double>> graph`。

####构建图

- 遍历 `equations` 数组，设当前方程为 `equations[i] = [A, B]`，对应值为 `values[i] = k`。
- `graph.computeIfAbsent(A, key -> new HashMap<>()).put(B, k);`
- `graph.computeIfAbsent(B, key -> new HashMap<>()).put(A, 1.0 / k);`

####处理每个查询

- 创建一个结果数组 `results`，长度与 `queries` 相同。
- 遍历 `queries` 数组，设当前查询为 `queries[j] = [C, D]`。
- **预检查**:
  - 如果 `graph` 中不包含 `C` 或 `D`，说明变量未知，结果为 `-1.0`。
  - 如果 `C` 和 `D` 相同，结果为 `1.0`。
- **DFS 搜索**:
  - 创建一个 `HashSet<String> visited` 用于本次查询。
  - 调用 `dfs(C, D, 1.0, graph, visited)`。
  - 将 `dfs` 的返回值存入 `results[j]`。

####DFS 辅助函数 `dfs(String current, String target, double product, Map graph, Set visited)`

- **标记访问**: 将 `current` 节点加入 `visited` 集合。
- **获取邻居**: 从 `graph` 中获取 `current` 节点的所有邻居。
- **遍历邻居**:
  - 对于每一个邻居 `neighbor` 和对应的权重 `value` (`current / neighbor = value`)：
    - 如果 `neighbor` 就是 `target`，说明找到了路径。返回 `product * value`。
    - 如果 `neighbor` 未被访问过：
      - 递归调用 `dfs(neighbor, target, product * value, graph, visited)`。
      - 如果递归调用返回的不是 `-1.0`（说明找到了路径），则直接返回该结果。
- **失败返回**: 如果遍历完所有邻居都未能找到 `target`，返回 `-1.0`。


```java
/**
graph: 图的邻接表
visted:已经访问的节点集合
current：当前节点
target：目标节点
accProduct: 累计乘积值
**/
public int dfs(Map<String,Map<String,Double>> graph, Set<String> visited, String current,String target,double accProduct){
	// 添加当前节点到visited
  visited.add(current);
  // Base condition,找到了target节点，则返回累计乘积值
  if (current == target) return accProduct;
  // 没找到target节点，继续往下找
  // 逻辑处理
  Map<String,Double> neighbors = graph.get(current);
  for (Map.Entry<String,Double> neighbor: neighbors){
    // DFS套路，如果邻居节点没有visited
    if (!visited.contains(neighbor)){
      double currentAccProduct = accProduct*neighbor.value;
      // 继续计算后序的乘积值，这一步的理解很重要
      // 如果后续计算的累积值不是-1，说明在后面的路径中找到了target节点，返回即可
      double product = dfs(graph,visited,neighbor.key,target,currentAccProduct)
      if (product !=-1.0){
        return product;
      }
    }
  }
  //路径不存在则返回默认值-1
  return -1.0d;
}
```
