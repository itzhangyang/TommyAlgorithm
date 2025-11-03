#树形DP-图+动态规划

树形动态规划（Tree DP）是在**树**这种数据结构上进行动态规划的一种方法。

与在数组或字符串等线性结构上的DP不同，树形DP的状态转移主要依赖于树的**父子和兄弟关系**。

它通常通过**深度优先搜索（DFS）**来完成。

在递归的**回溯（后序遍历）**过程中，子节点将计算好的信息**汇报**给**父节点**。

父节点利用这些信息来计算**自身的状态**，最终得到整个问题的解。

## 核心思想： 自底向上

树形DP的核心思想是 **“子树分解”** 和 **“自底向上”** 的信息汇总。

### 定义子问题

对于树中的任意一个节点 `u`，我们把以它为根的整个子树看作一个独立的子问题。

### 依赖子节点

节点 `u` 的最优解，通常可以通过它所有直接子节点 `v` 的最优解组合而成。

### 信息汇总（后序遍历）

通过一次DFS遍历，当一个节点 `u` 的所有子树都已经被访问和计算完毕后（即DFS的后序位置），我们就可以利用从子节点那里收集到的信息来计算当前节点 `u` 的状态。

### 无后效性

一旦一个子树的状态计算完成，我们就不再关心其内部的具体计算过程，父节点只需要这个最终的状态结果即可。

## 实现原理与通用套路： DFS后序遍历

树形DP的实现与**深度优先搜索（DFS）**的递归结构天然契合。

### 实现原理 - 利用DFS递归调用栈

当DFS深入到叶子节点时，可以直接计算出其初始状态。

当DFS从一个子节点的递归中返回时，就意味着这个子问题的解（状态）已经被计算出来。

在一个节点的所有子节点递归都结束后，该节点就拥有了计算自身状态所需的所有信息。

### 通用解体套路

#### 问题识别 - 是否将将父节点的解通过子节点推导出来？

分析问题是否能将**以u为根的子树**的解，通过其**子节点v的子树**的解来推导。

如果可以，基本就是树形DP问题。

#### 状态定义：你需要从子节点中获取什么信息？

思考为了计算节点 `u` 的状态，你需要从它的子节点 `v` 那里获得**什么信息**？

这个“信息”就是DP状态。

在Java中，这个状态通常是一个数组（如 `int[]` 或 `long[]`）或一个自定义的`State`对象，由DFS函数返回。

#### 状态转移方程

明确如何利用所有子节点 `v` 返回的状态，结合节点 `u` 自身的信息，来计算出节点 `u` 的状态。

#### 编码实现

- 使用**邻接表**（`List<Integer>[]` 或 `Map<Integer, List<Integer>>`）来存树。
- 编写一个DFS函数，通常签名是 `dfs(int u, int parent)`，`parent` 参数用于在无向图中防止走回头路。
- DFS函数通过 `return` 语句返回**当前子树的状态**（一个数组或对象）。
- 在DFS函数内部，首先递归调用所有子节点，然后利用返回的结果进行状态转移计算。

## 代码套路模板

我们通常使用DFS函数返回一个包含状态信息的数组，这种方式代码简洁高效。

```Java
import java.util.*;

class TreeDP {
    // 使用邻接表表示树
    private List<Integer>[] graph;
    private int N;
    // 如果需要全局存储结果（例如树的直径），可以定义一个成员变量
    private int globalMax;

    public void solve(int n, int[][] edges) {
        this.N = n;
        this.graph = new ArrayList[N];
        for (int i = 0; i < N; i++) {
            graph[i] = new ArrayList<>();
        }
        // 建图
        for (int[] edge : edges) {
            graph[edge[0]].add(edge[1]);
            graph[edge[1]].add(edge[0]);
        }
        
        // 初始化全局变量
        globalMax = 0;

        // 从任意根节点（例如 0）开始DFS
        dfs(0, -1); 

        // 最终答案可能是 globalMax，或者是 dfs(0, -1) 的某个返回值
        // System.out.println("The answer is: " + globalMax);
    }

    /**
     * DFS函数处理以 u 为根的子树。
     * @param u 当前节点
     * @param parent u 的父节点，防止走回头路
     * @return 返回一个包含状态信息的数组。例如：new int[]{state1, state2}
     */
    private int[] dfs(int u, int parent) {
        // 1. 初始化当前节点 u 的状态
        //    例如：uState[0] = 0, uState[1] = value[u]
        int[] uStates = new int[2]; // 假设需要两个状态

        // 2. 遍历 u 的所有邻居 v
        for (int v : graph[u]) {
            if (v == parent) {
                continue; // 不走回头路
            }

            // 3. 递归调用，获取子节点 v 的状态信息
            int[] childStates = dfs(v, u);

            // 4. 【核心】状态转移：
            //    根据子节点 v 返回的状态 childStates，更新当前节点 u 的状态。
            //    这是问题的核心逻辑所在。
            //    uStates[0] = f(uStates[0], childStates[0], childStates[1], ...);
            //    uStates[1] = g(uStates[1], childStates[0], childStates[1], ...);
            
            //    同时，也可能在这里结合子节点信息和当前节点已有的信息来更新全局答案
            //    globalMax = Math.max(globalMax, combine(uStates, childStates));
        }

        // 5. 返回当前节点 u 的最终状态，供其父节点使用
        return uStates;
    }
}
```

##场景示例-权值和最大的不相邻节点

给定一棵树，每个节点有一个权值，选择一些**不相邻**的节点，使得选中节点的权值和最大。

“**不相邻的节点**”指的是：在你选择的节点集合中，**不能包含一对直接相连的节点**。

这个就是著名的**树上选点问题**（也称为“树上最大独立集”）。

- 相邻：指的是在树上有一条边直接连接的两个节点；
- 不相邻：指的是节点之间没有直接边连接；
- 限制“不能选相邻的节点”是树形 DP 中很多经典题目的前提，比如“最大独立集”。

####状态定义-`dp[u][0/1]`表示选/不选择节点`u`的最优值

`dp[u][0/1]` 表示不选/选节点 `u` 的最优值

- 我们用 `dp[u][0]` 表示 **不选**节点 `u` 时，以 `u` 为根的子树能获得的最大权值和。
- 我们用 `dp[u][1]` 表示 **选中**节点 `u` 时，以 `u` 为根的子树能获得的最大权值和。

#### 状态转移方程

对于每个节点`u`的**每一个子节点**`v`

- **如果不选u**:

  那么我们可以自由选择是否选`v`,即：

  - 不选择`v`: `dp[v][0]`
  - 选择`v`:`dp[v][1]`

  ```java
  for(Node v : u.neighbors){
  	dp[u][0] = dp[u][0]+max(dp[v][0],dp[v][1]);
  }
  ```

- **如果选择了u**:

  那不能选子节点 `v`，即：

  ```java
  dp[u][1]=dp[u][1]+dp[v][0]
  ```

#### 代码实现- 图的DFS遍历

##### 将树转化为图

- 首先将树转化为图的`邻接表List<Integer>[] grap`, `graph[i]` 表示节点`i`的邻接节点列表，每个节点的邻接点构成：

  - 父节点，只有一个
  - 子节点，多个

##### 利用DFS更新DP值

######复习- DFS的套路

- 利用图的`dfs`过程，复习一下dfs套路`dfs(graph,visited,node)`

  - 定义变量
    - Graph为图
    - node为遍历的起始节点
    - visited为已访问节点集合
  - DFS过程
    1. 将当前节点加入`visited`
    2. // 判定当前节点是否满足条件
    3. 获得当前节点的所有邻居节点，对于每一个邻居节点(`neighbor`)：
       1. 检查邻居节点(`neighbor`)是否已经在`visited`当中
       2. 如果邻居节点(`neighbor`)不在`visited`中，递归调用`dfs(graph, visited,neighbor)`

- 算法过程解析

  本质上是利用dsf的递归特性，从底部到顶部的计算。

  - 初始化：

    - 初始化`dp[u][0]=0`

    - 初始化`dp[u][1]=weight[u]`

  - 调用下一层递归计算 `dp[v][0]`以及`dp[v][1]`的值

  - 将`dp[v][x]`添加到`dp[u][x]`

    - `dp[u][0] +=Math.max(dp[v][0],dp[v][1]);`
    - `dp[u][1] += dp[v][0];`

```java
/**
 * @param u 当前处理的节点编号
 * @param parent u 的父节点，用于防止回溯
 * @param tree 树的邻接表（n+1 个列表，下标从 1 开始）
 * @param weight 节点权值，weight[u] 表示 u 的权重
 * @param dp dp[u][0] 表示不选 u，dp[u][1] 表示选 u
 */
void dfs(int u, int parent, List<Integer>[] tree, int[] weight,int[][] dp){
  dp[u][0]=0;
  dp[u][1]=weight[u];
  
  for (int v: tree[u]){
    if (v==parent) continue;
    dfs(v,u,tree,weight,dp);
    //如果一时间考虑不出来，记下来
    dp[u][0] +=Math.max(dp[v][0],dp[v][1]);
    dp[u][1] += dp[v][0];
  }
}
```

##### 总结

###### 状态和变量

| 状态        | 含义                                 | 状态转移公式                                               | 说明                                        |
| ----------- | ------------------------------------ | ---------------------------------------------------------- | ------------------------------------------- |
| `dp[u][0]`  | 不选节点 `u` 时，以 `u` 为根的最大值 | `dp[u][0] += max(dp[v][0], dp[v][1])` for all children `v` | 不选 `u`，儿子 `v` 可以选也可以不选         |
| `dp[u][1]`  | 选节点 `u` 时，以 `u` 为根的最大值   | `dp[u][1] += dp[v][0]` for all children `v`                | 选了 `u`，儿子 `v` 就**不能选**（防止相邻） |
| `weight[u]` | 节点 `u` 的权值                      | `dp[u][1] = weight[u] + Σ dp[v][0]`                        | 初始值：选中节点 `u` 的收益                 |
| 终值        | 整棵树的最优解                       | `max(dp[root][0], dp[root][1])`                            | 根节点选或不选中较优的一种                  |

##Leetcode 337: 树形打家劫舍

可以使用**DFS**的方式进行计算。

###### 实现代码

```java
    public int rob(TreeNode root) {
        int[] res = dfs(root);

        return Math.max(res[0], res[1]);
        
    }

    private int[] dfs(TreeNode node){
        if(node == null){
            return new int[]{0,0};
        }
         int[] left = dfs(node.left);
         int[] right = dfs(node.right);

         int[] res = new int[2];
         // 不偷当前节点的情况下，值为max(不偷左节点，偷左节点) + max(不偷右节点，偷右节点)
         res[0] = Math.max(left[0], left[1]) + Math.max(right[0], right[1]);
         // 偷当前节点的情况下，左节点和右节点都不能偷
         res[1] = node.val + left[0] + right[0];

         return res;
    }
```

##Leetcode 124 二叉树中的最大路径

二叉树中的 **路径** 被定义为一条节点序列，序列中每对相邻节点之间都存在一条边。同一个节点在一条路径序列中 **至多出现一次** 。该路径 **至少包含一个** 节点，且不一定经过根节点。

**路径和** 是路径中各节点值的总和。

给你一个二叉树的根节点 `root` ，返回其 **最大路径和** 。

###问题分析

一条路径由：**左路径**+**根节点**+**右路径**

### 核心思想 - 计算最大深度和的时候同时更新结果



###实现代码

```java
    private int maxGain(TreeNode node){
        if (node == null) return 0;
				//注意0的情况
        int leftGain = Math.max(maxGain(node.left),0);
        int rightGain = Math.max(maxGain(node.right),0);
				// merge, 当前的最大值=左子树最大值+右子树最大值+当前节点值
        int nodeMaxSum = leftGain+rightGain+node.val;
				// 更新最大值
        maxSum = Math.max(nodeMaxSum, maxSum);
				// 返回左子树和右子树的最大值+当前节点值
        return Math.max(leftGain, rightGain)+node.val;
    }
```

##Leetcode 834 树中距离之和

我们需要计算一棵无根树中，每个节点到所有其他节点的距离总和。

由于输入是树（无环图、连通图），我们可以从任意节点设为根节点，利用**整棵树的结构信息**高效地推导出每个节点的答案。

##### 问题分析

我们需要计算一颗**无根树**中，每个节点到其他节点的距离总和。

由于输入是树（**无环图，连通图**）我们可以从任意节点设为根节点，利用整棵树内的结构信息高效的推导出每个节点的答案。

- **朴素思路：**对每一个节点跑一次DFS/BFS，统计到其他节点的所有距离，时间复杂度是`O(n^2)`会超时
- **优化方向：**我们可以借助树的**递归性质**和**换根技巧**，先以某个点为根求出距离和，然后递推出所有节点的距离和，达到`O(n)`的复杂度。

##### 核心思想：树形动态规划 + 换根技巧

通过两次**DFS:**

- 第一次**后序遍历（自底向上）：**从叶子节点向上聚合树信息
- 第二次**前序遍历（自顶向下）：**根据父节点的信息推导子节点的信息

##### 状态定义和初始化

###### 状态定义

- `count[i]`:以`i`为根节点的树中的节点的数量（包括节点`i`）
- `res[i]`:以`i`为根节点，`i`节点到其他节点的所有的距离和

###### 状态初始化

```java
int[] count = new int[n];
int[] res = new int[n];
```

##### 遍历逻辑

###### 第一次DFS:后序遍历`dfs1(int node,int parent)`

后序遍历是先遍历子节点，回溯，然后遍历父节点。

我们利用后序遍历，统计每一个父节点的**子节点数量**，计算树的**根节点**（`root`）到**其他所有节点**的距离之和。

注意这个过程我们**只能计算根节点的结果**，其他节点的结果我们无法计算。

- 统计每个节点的子树的大小`count[node]`
- 计算根节点到所有其他节点的距离和`res[0]`

###### 第二次DFS:前序遍历`dfs2(int node, int parent)`

前序遍历是**先处理父节点**，之后再利用父节点的结果，处理子节点。

- 将`res[parent]`推导到`res[child]`
- 基于`count`信息进行公式转换

##### 状态转移方程

###### 第一次DFS（后序）

对于**当前节点**的每一个子树`v`，他到所有的子孙后代的节点的距离的计算方式：

1. 计算`v`到其所有子节点的距离和
2. 因为**当前节点**和`v`之间的距离为`1`，也就是说它到`v`子树中的每一个子节点的距离又远了`1`， 距离和应当+`count[v]`
3. 因此当前节点到子树`v`中的每一个节点的距离和为`res[v] + count[v]`

对于每一个节节点`node`，到其他节点的路径和就是到其所有叶子节点的距离：



```Java
for(int child : children){
	res[node] += res[child]+count[child]
	count[node] += count[child]
}
```

###### 第二次DFS(前序 - 自下而上)

注意我们现在已经有了**根节点到所有其他节点的距离数据**，所以我们可以考虑从**根节点开始向下推导每一个节点的数据**。

现在我们的目的是：利用 `ans[parent]` 来计算 `ans[child]`。

我们已经定义了`ans[u]`: 节点 `u` 到树中所有其他节点的距离之和。



假设我们正在从父节点 `u` 访问子节点 `v`，并且我们已经计算出了 `ans[u]`。

现在我们假设我们已经计算了节点`u`的结果`ans[u]`， 现在对于`u`的每一个子节点`v`，我们需要计算`ans[v]`。

对于每一个子节点`v`， `ans[v]`中的距离和包括两个部分：

- **v到它的所有的子节点的距离和：**

  我们在第一轮**后序遍历**中已经计算到，`v`树中的节点数量是`count[v]`。

  对于`v`树中的每一个节点，其到`v`的距离比到`u`的距离少1,`count[v]`个节点的距离总和则少了`count[v]`。

- **v到其余的节点的距离和**

  我们已经知道`v`树中的节点数量为`count[v]`， 那么`v`树之外的节点数量就是`n - count[v]`。

  对于`v`树之外的每个节点，其到`v`的距离比到`u`的距离多了1，`n-count[v]`个节点的距离总和则多了`n-count[v]`。

根据以上的推导，我们得出节点`v`到其他所有节点的距离总和公式：

`ans[v] = ans[u] - count[v] + n - count[v]`

```
res[child] = res[node] - count[child] +(n-count[child])
```

##### 实现代码

###### 第一次DFS： 后序遍历-计算所有根节点到叶子节点的距离

在后序遍历中，我们是先计算每个子节点的`res[child]`和`count[child]`的：

- `res[child]`是每个子节点的距离和
- `count[i]`是每一个子节点的树中所包含的节点数量

对于每一个**子树中的节点**，根节点到它的距离为**子节点到它的距离**+1，因此因为每一个子树中的节点数量是`count[child]`，所以根节点到子树中所有节点的距离`res[child]+count[i]`.

```java
private int postorderDfs(int node, int parent){
  count[node] = =1 ;//初始化其所在的树的节点的数量
  for(int child : graph.getOrDefault(node, new ArrayList<>()){
    if (child == parent) continue;
    postorderDfs(child,node);
    // 节点的路径和就是子树的路径和+子树节点的数量
    res[node] += res[child]+count[child];//为什么是这个
  }
}
```

###### 第二次DFS：前序遍历-计算所有叶子节点到其他叶子节点的距离

前序遍历中，我们先计算每一个**父节点到顶部节点的距离[父节点...祖父节点...曾祖节点...根节点]**

再第一轮**后序DFS**中，我们已经知道了如下数据：

- `res[i]`: 表示以节点 `i` 为根时，**i 到其他所有节点的距离和**
- `count[i]`: 以 `i` 为根的子树中包含的节点数（包括自己）
- `n`：树中总节点数

现在我们需要将**node换根到child:**

- 假设当前根是`node`，我们已经知道`res[node]`
- 现在我们要把根移动到它的某个子节点`child`， 求`res[child]`

现在`node`成了`child`的子树，那么`child`的子节点就包含了原来自己的子节点和`node`中的子节点了

| 类别         | 说明                      | 变化                                      |
| ------------ | ------------------------- | ----------------------------------------- |
| child 的子树 | `count[child]` 个节点     | 这些节点离新根 `child` 更近，距离减少 `1` |
| 其余节点     | `n - count[child]` 个节点 | 这些节点离新根 `child` 更远，距离增加 `1` |

```java
private int preorderDfs(int node, int parent,int n){
  for (int child : graph.getOrdefault(node, new ArrayList<>())){
    if (child == parent) continue;
    // 子树的路径和 = 父节点的路径和 - 子树节点数量（距离-1） + 子树之外的节点的数量
    res[child] = res[node] - count[child]+(n-count[child]);
    preorder(child, node,n);
  }
}
```

###### 调用上面的两次DFS进行计算

```java
int[] res;//每个节点到其他节点的距离和
int[] count[];//每个节点的子孙节点的数量
Map<Integer,List<Integer>> graph;
public int[] sumOfDistanceInTree(int n, int[][] edges){
  res = new int[n];
  count = new int[n];
  graph = getGraph(n, edges);
  
  postorderDfs(0,-1);//从根节点开始调用
  preorderDfs(0,-1,n);
  return res;
}
```

##### 注意事项

| 项             | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| 🚫 防止死循环   | DFS 时用 `parent` 参数防止回到上一个节点                     |
| 🧩 建图细节     | `Map<Integer, List<Integer>>` 比数组更灵活，适合处理稀疏图或非连续编号 |
| 🧮 count 数组   | 初始化为 0，DFS 中从叶子往上累加                             |
| ⏱️ 时间复杂度   | O(n)，适合处理 3 万规模的大树                                |
| 🧠 换根公式核心 | `res[child] = res[parent] - count[child] + (n - count[child])` 是这道题最关键的思想 |

##### 经验总结

| 维度       | 内容                                                     |
| ---------- | -------------------------------------------------------- |
| ✅ 本质     | 树形结构适合递归聚合子树信息（树形 DP）                  |
| 🔄 换根技巧 | 从一个点为根出发，推导所有节点作为根的结果，避免重复计算 |
| 🧱 构图方式 | 使用 `Map<Integer, List<Integer>>` 建图，结构清晰且灵活  |
| 🚀 复杂度   | O(n) 时间、O(n) 空间，解决大规模数据问题                 |
| 🔍 可推广   | 可用于重心查找、最远节点统计、带权距离计算等问题场景     |

