#枚举所有路径：DFS+回溯

##场景描述

在一个DAG中，找出从起点到终点的所有路径，用于任务调度路径枚举、流程引擎中圈路径遍历、路线分析（如从A地出发所有合法旅游路线）。

##通用应对策略：DFS+回溯

使用DFS+回溯构造路径，终点加入结果。

##核心算法思想

通过回溯不断构造路径，**在达到目标时复制当前路径加入答案**

##### DFS+回溯输入：`起点+终点+Graph`+path

##### DFS+回溯的输出：所有从起点到终点的路径列表

##实现代码模板套路

##### 注意使用node自定义类的时候，需要定义equals和hashcode函数

##### DFS+回溯代码套路模板

```java
public void dfs(Map<Node, List<Node>> graph,Node current, Node target, Set<Node> visited, List<Node> path){
  //处理逻辑
  if (/*满足条件*/){
    res.add(new ArrayList(path));
    return;
  }
  
  List<Node> neighbors = graph.getOrDefault(node,new ArrayList<>());
  for (Node neighbor: neighbors){
    if (visited.contains(node)) continue;
    
    visited.add(neighbor);
    path.add(neighbor);
    
    dfs(graph,neighbor,target,visited,path);
    
    path.remove(path.size()-1);
    visited.remove(node);
  }
}
```

## DFS的两种回溯模式

### 以当前节点为中心

#### 检查当前节点是否合法

 不合法则`return`

#### 将当前节点加入path

- 加入path
- 加入visited

#### 检查当前path是否是一个解

如果是一个解，则加入解集合。

//检查Path是否还能继续，不能继续则`return`

#### 遍历邻居节点递归DFS

#### 将当前节点从path中移除

### 以邻居节点为中心

#### 检查path是否是一个解

如果是解，加入解集合

#### 检查当前path是否合法，不合法剪枝

#### 遍历邻居节点

##### 将邻居节点加入path

- 加入path
- 加入`visited`

##### 递归DFS

##### 将邻居节点从path中移除

##Leetcode 797 所有可能路径

给你一个有 `n` 个节点的 **有向无环图（DAG）**，请你找出所有从节点 `0` 到节点 `n-1` 的路径并输出（**不要求按特定顺序**）

 `graph[i]` 是一个从节点 `i` 可以访问的所有节点的列表（即从节点 `i` 到节点 `graph[i][j]`存在一条有向边）。

###问题的本质-遍历起点到终点的所有路径-适用于回溯+DFS

####注意图的表示形式`graph[node index][neigbor list]`

这种图的表示形式并不算很常见，遇到的时候需要注意分析。

####遍历优先无环图不需要使用`visited`

####需要使用`visited`的典型场景

当图中可能存在环的时候，为了避免循环或者重复遍历，必须使用`visited`记录访问状态。

- **无向图：**如果不添加`visited`，DFS可能会**反复来回走**
- **有向图中可能有环：**比如在图中寻找是否存在环，就一定要`visited`
- **走迷宫类问题：**如(leetcode 79,200) ： 为了避免走重复同一个个字，通常会加`visited`

####有向无环图中不可以使用visited

要找出**有向无环图**中从节点`0`到`n-1`的路径，由于是DAG，不存在环，所以路径不回重复访问某个节点，不会形成死循环，所以不需要使用`visited`

####在DAG中使用`visited`会导致什么问题？-错误的剪掉合法路径

假设有如下Graph:

```
graph = [[1,2],[3],[3],[]]
```

合法路径有两条：

- [0,1,3]
- [0,2,3]

如果使用`visited`，则在访问完第一个分支以后，会将`3`标记为`未访问`，但是实际上可能已经阻止了一些本该可能的路径

####来看看“错误使用 `visited`”时发生了什么：

你写的代码里，如果访问了节点 `3`，你会标记为 `visited[3] = true`，然后再从另一条路径也想访问 `3`，就会被 `continue` 掉！

也就是说，**`visited` 限制了同一个节点在不同路径中被访问多次，这是不对的！**

> ❌ 错误理解是：“节点一旦访问过，别的路径就不能用了。”
> ✅ 正确理解是：“节点不能在**同一条路径**中重复访问（避免环），但可以在**不同路径**中多次访问。”

| 图类型                | 是否可能死循环     | 是否需要 `visited` | `visited` 的作用                       | 是否会误伤（遗漏合法路径） |
| --------------------- | ------------------ | ------------------ | -------------------------------------- | -------------------------- |
| **有向无环图（DAG）** | ❌ 否               | ❌ 不需要           | 不需要剪枝；允许节点在不同路径重复出现 | ✅ 会（不该加）             |
| **无向无环图（树）**  | ❌ 否（但边是双向） | ✅ 需要             | 防止走回头路（避免伪环）               | ❌ 不会（该加）             |
| **有向有环图**        | ✅ 是               | ✅ 必须             | 防止死循环 / 检测环                    | ❌ 不会（该加）             |
| **无向有环图**        | ✅ 是               | ✅ 必须             | 防止回头 / 死循环                      | ❌ 不会（该加）             |

####不用visited的典型场景：DAG

当题目保证图是**有向无环图（DAG）**，就**不会有环**，也就不需要担心重复访问引起死循环。

- DAG的拓扑排序
- 找DAG中的所有路径
- 动态规划记忆化DFS

##### DFS+回溯中的path在调用前是否应该先添加起点到path中

一般来说有两种写法：

######写法一：在调用DFS前将起点加入Path(推荐)

```java
path.add(0);
dfs(graph,0,path);
```

###### 写法二：在DFS里面添加节点（更通用一些）

```java
dfs(current){
  path.add(current);
  ...
  path.remove(path.size-1);
}
```

这种写法比较通用，比如在Leetcode 46 全排列那种场景，但在路径搜索中可能需要**小心处理结束条件**。

###### 什么情况下应该用哪一种写法？

- 比如你的**路径是从某个明确起点开始**，比如`0 -> n-1`，一开始加上起点是更自然、更简洁的写法
- 如果你在DFS中进入某个节点再考虑它，那你就得保证逻辑统一（比如**每次进入时都`add(current)`，返回时都`remove()`**）

##### 实现原理

###### 遍历思想：深度优先搜索（DFS）

- 把图看作是一棵**多叉树**，每个节点都有若干子节点
- 从节点`0` 出发，一路DFS到`n-1`
- 每次到达终点时，将路径记录下来

######路径记录：使用`path`保存当前路径

- 每次进入一个节点，就加入`path`
- 每次回溯时，从`path`中移除**最后一个节点**
- 每当路径走到终点`n-1`时，将`path`加入到结果集

######不使用`visited`

- 因为题目保证图为有向无环图
- 所有路径中不会重复访问同一个节点，不存在死循环
- 使用`visited`会误伤，使节点**不能再多条路径中使用**
  - 比如路径`A-B-D`和`A-C-D`如果标记了D，就会漏掉其中一条

#####实现代码

```java
    List<List<Integer>> res = new ArrayList<>();
    int n; 
    // DFS+回溯
    public List<List<Integer>> allPathsSourceTarget(int[][] graph) {
        n = graph.length;
        List<Integer> path = new ArrayList<>();
        path.add(0);
        dfs(graph,0,path);
        return res;
    }

    private void dfs(int[][] graph, int current, List<Integer> path){
        if(current == n-1){
            res.add(new ArrayList<>(path));
            return;
        }
        int[] nexts = graph[current];

        for(int next : nexts){
            path.add(next);

            dfs(graph,next,path);

            path.remove(path.size()-1);
        } 
    }
```

#####实现中的注意事项

| 注意点                                           | 说明                                               |
| ------------------------------------------------ | -------------------------------------------------- |
| **1. 不要用 `visited`**                          | 图是 DAG，节点可以在不同路径重复出现               |
| **2. `path.add(0)` 要放在 DFS 前面**             | 因为路径必须从 0 开始，而 `dfs` 的递归只加后续节点 |
| **3. 使用 `new ArrayList<>(path)` 保存路径副本** | 否则最终结果会被 `path` 的回溯操作修改掉           |
| **4. 图是邻接表结构**                            | `graph[i]` 是一个数组，表示 i 能到达的所有节点     |

