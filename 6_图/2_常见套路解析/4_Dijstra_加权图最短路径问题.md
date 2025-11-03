# Dijkstra 最短路径 - 加权图中某点到其他每个节点的最短路径

>  LeetCode 图论题中，约 6%~10% 属于典型 Dijkstra 题
>
>  图论题中排第 3，属于“需要掌握”的高频算法

| 方面            | 说明                                     |
| --------------- | ---------------------------------------- |
| 面试频率        | 图论题中排第 3，属于“需要掌握”的高频算法 |
| LeetCode 覆盖度 | 图论题中约 6~10% 是典型 Dijkstra 题      |
| 特征关键词      | **最短路径** + **权重** + **非负边权**   |
| 能力要求        | 对优先队列（heap）、邻接表、松弛操作熟练 |

##理解Dijkstra算法究竟是解决什么问题的-加权图中某节点到其他所有节点的最短路径

Dijkstra（迪杰斯特拉）算法是一种经典的图算法，用于在**加权图**中查找**从一个节点（源节点）到所有其他节点的最短路径**。

它由荷兰计算机科学家艾兹赫尔·迪杰斯特拉（Edsger W. Dijkstra）于1956年提出，是解决单源最短路径问题的有效方法。

## Dijstra算法解决了什么问题？ - 单源最短路径问题

### 解决了什么问题？- 单源最短路径问题

Dijkstra算法主要解决的是**带非负权重的有向图或无向图**中的**单源最短路径问题**。

具体来说，就是给定一个图和一个源顶点，找出从源顶点到图中所有其他顶点的最短路径。

### 注意限制 ！- 不能处理带有负权重边的图

需要注意的是，标准的Dijkstra算法**不能**处理带有负权重边的图。

因为一旦出现负权边，算法的“贪心”策略（即总是选择当前看起来最近的节点）就会失效，无法保证找到全局最优解。

##Dijkstra算法的核心思想： 贪心算法 + BFS

每次从**`当前未处理的节点`中**选择距离起点start`最近`的节点，确定它的最短路径，并用它来更新`它的邻接点的距离`

它将图中的节点分为两类：

- **已确定最短路径的节点集合（S）**

- **未确定最短路径的节点集合（U）**

###什么是`未处理的节点`？

| 状态                 | 解释                                                     |
| -------------------- | -------------------------------------------------------- |
| 已处理节点           | 它的最短路径值已确定，再也不会被更短路径更新             |
| 未处理节点           | 虽然当前知道它到起点的距离，但我们不确定这是否是最短路径 |
| 未访问节点（不可达） | 当前甚至都不知道怎么到达它（距离仍是无穷大）             |

###算法的逻辑流程

算法从源节点开始，逐步将未确定集合U中的节点，按照距离源节点由近到远的顺序，逐个移动到已确定集合S中。其基本逻辑是：

- 总是选择当前距离源节点最近的未访问节点。

- 基于这个**新加入的“最近”节点**，更新其所有邻居节点到源节点的距离。

  如果通过这个新节点到达其邻居的路径比之前记录的路径更短，则更新邻居节点的距离（这个过程称为“松弛”操作）。

- 重复这个过程，直到所有节点都被访问过。

由于每次都选择全局最短的路径进行扩展，保证了当一个节点被加入到集合S时，其到源点的距离已经是最终的最短距离。

## Dijstra算法的要素

### 图的表示 - 邻接表或者矩阵

用于表示图的结构，存储节点之间的连接关系和边的权重。

### 距离数组（`dist`）- 从源节点到节点 `i` 的当前已知最短距离

`dist[i]` 存储从源节点到节点 `i` 的当前已知最短距离。

初始时，源节点的距离为0，其他所有节点的距离为无穷大。

### 已访问节点集合

一个布尔数组或集合，用于记录哪些节点已经找到了最短路径。

### 优先队列 - 快速找出距离源节点最近的节点

这是优化Dijkstra算法的关键数据结构。它用于高效地存储未访问的节点，并能快速地找出其中距离源节点最近的节点。

### 优先队列的存储元素 (`Pair`)

我们定义一个Pair元素，用于存储当前**每一个节点到源节点最近的距离**。

因为我们使用小顶堆，因此小顶堆的堆顶的Pair元素的距离一定是当前距离源点最小的。

 ```Java
class Pair{
  int node;
  int distance;
}
 ```



##算法流程

###初始化

####初始化距离数组

#####内容和保存结构dist[node] - start到node的最短距离 

- `dist[node] `表示从start到节点node的最短距离

#####初始化状态 - 自己为0，其他为最大值

- 初始化start到每一个节点的最小距离

  - start到start的距离为0

  - start到其他节点的距离为`MAX_VALUE`

    > - 求最大值的时候，初始值一般设置为`MIN_VALUE`
    > - 求最小值的时候，初始值一般设置为`MAX_VALUE`

####初始化`未确定最短路径`的所有节点-小顶堆

#####内容和保存结构-start到每个节点的最小距离

####定义最小堆 - 维护

定义一个小顶堆（优先队列`PriorityQueue`）维护`还没有确定最短路径`的节点，小顶堆的顶部存放当前`距离start最近`的节点。

```java
PriorityQueue<Pair> pq = new PriorityQueue<>((a,b)->a.distance - b.distance);
```

- 优先队列存放的元素为`start到每一个节点的距离`:
  - Node: 表示终点节点
  - Distance：表示start到终点node的距离
- 优先队列的排序顺序为按照距离distance从小到大排序，堆顶的元素始终为距离start节点最近的节点

#####初始化状态 - 将远点（start，0） 加入小顶堆

- 将start节点放入优先队列中（node=start, distance=0）

  ```java
  pq.offer(new Pair(start,0));
  ```


###计算流程 - 当堆不为空时

####从优先队列堆顶取出`距离start最小的节点`（u）

按照小顶堆的属性，**堆顶的节点是距离start最小的节点**，直接取出该节点。

####重新计算start到u的所有的邻接节点(v)的距离

- 从**优先队列**中取出一个`distance`元素, 假设元素中的值如下：

  - 节点为`u`
  - 距离为`d`

- 如果`d` > `dis[u]`， 说明`u`的最短路径不是这个，直接**剪枝** //Todo a question

- 获取所有的`u`的**邻接节点**，对于每一个邻接节点`v`:

  - 如果当前`v的最小距离`（`dist[v]`）大于`通过start通过u到达v的最小距离`(`dist[u]+w(u,v`)), 那么

    - 将v的最小距离更新为start通过u到达v的最小距离

      ```java
      dist[v]=dist[u]+w(u,v)
      ```


    - 将v加入邻接队列（因为v的邻接点还没有处理）(node=v,distance=dist[v])
    
      ```java
      pq.offer(new Distance(v,dist[v]));
      ```


​      

####重复上述两个流程，直至优先队列为空

## Dijstra的代码套路模板

```java
//这个类用来表示什么？
//表示start到每一个节点（node）的距离(dist[ance])
class Pair{
  int node,dist;
  Pair(int n, int d) { node = n; dist = d; }
}

public void dijkstra(List<int[]> graph, int start){
// 建立一个小顶堆，按照每一个节点距离start的距离从小到大排序
PriorityQueue<Pair> pq = new PriorityQueue<>((a,b)->a.dist-b.dist);
// start到每一个节点的距离 dist[i]表示start到节点i的距离
// 这是最终输出的结果，表示start到每一个节点的最小距离
int[] dist = new int [n];
//start 到每一个节点的默认距离都被设置为MAX_VALUE
Arrays.fill(dist, Integer.MAX_VALUE);
dist[start] = 0;
//将start Pair 加入队列
pq.offer(new Pair(start, 0));

while (!pq.isEmpty()){
  Pair currentPair = pq.poll();
  //如果当前Pair中的距离大于dist中的距离，什么也不做
  if(currentPair.dist > dist[currentPair.node]){
    continue;
  }
  // 如果当前Pair中的距离小于dist[node]中的距离
  // 找到与node相连的每一条边
  for(int[] edge: graph.get(currentPair.node)){
    int neigbor=edge[0],weight = edge[1];
    // 如果说start到node的距离+node到neigbor的距离 小于 start到neigbor的距离，则更新dist[neigbor]的距离
    // 将neigbor信息加入到队列当中
    if (dist[currentPair.node]+weight < dist[neigbor]){
      dist[neighbor] = dist[currentPair.node] + weight;
      pq.offer(new Pair(neighbor, dist[neighbor]));
    }
  }
}
}

```

## Dijstra的代码套路模板 - 基于Visited数组

```Java
import java.util.*;

public class DijkstraPureVisited {

    /**
     * 仅使用显式的 visited 数组作为剪枝依据的 Dijkstra 算法
     * @param n 节点数量
     * @param edges 边的信息 [[u, v, weight], ...]
     * @param src 源点
     * @return 从源点到所有节点的最短距离数组
     */
    public int[] dijkstra(int n, int[][] edges, int src) {
        // 1. 构建邻接表
        List<int[]>[] adj = new ArrayList[n];
        for (int i = 0; i < n; i++) {
            adj[i] = new ArrayList<>();
        }
        for (int[] edge : edges) {
            adj[edge[0]].add(new int[]{edge[1], edge[2]});
        }

        // 2. 初始化 dist 和 visited 数组
        int[] dist = new int[n];
        Arrays.fill(dist, Integer.MAX_VALUE);
        boolean[] visited = new boolean[n]; // 唯一的剪枝依据
        dist[src] = 0;

        // 3. 初始化优先队列
        PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a[0]));
        pq.offer(new int[]{0, src});

        while (!pq.isEmpty()) {
            // 4. 取出当前距离最小的节点
            int[] curr = pq.poll();
            // 我们不再关心取出的距离 currentDist，只关心节点 u
            int u = curr[1];

            // 5. 【核心】如果节点的最短路径已经被确定，则跳过
            // 任何后续再次遇到 u 的情况都代表是一条更长（或相等）的路径。
            if (visited[u]) {
                continue;
            }
            
            // 6. 确定 u 的最短路径，并标记为“已访问”
            visited[u] = true;

            // 7. 遍历邻居，进行松弛操作
            for (int[] neighbor : adj[u]) {
                int v = neighbor[0];
                int weight = neighbor[1];
                
                // 我们使用 dist[u] (已确定的最短距离) 来更新邻居
                if (dist[u] + weight < dist[v]) {
                    dist[v] = dist[u] + weight;
                    pq.offer(new int[]{dist[v], v});
                }
            }
        }

        return dist;
    }
}
```

## 注意事项

### Dijstra的重要特征 - 一旦确定了某个节点的最近距离，就不会再次访问该节点

#### 怎样确定某个节点的最近距离的？- 优先队列

从**最小堆** 中取出来的**第一个关于某个节点的记录`Pair`中的距离**，就是**源点到该节点的最小距离**。

这是因为最小堆的特性所决定的。

在第一次取出该节点的记录时，最小堆中还可能有其他的关于该节点的记录，但是都不是最小距离。

因此从队列中取出某一个节点的记录`Pair`时，我们有两种方法来确定某个节点的最小距离是否已经确定了：

- 比较取出的`Pair`中的距离和`dist`中该节点的距离，如果`Pair`中的更大，我们就认为该节点是已经确定过的
- 设置一个`visited`数组，第一次从最小堆中取出该节点时，将该节点的`visited[i]`设置为`true`，后续再次遇到该节点时，通过检查`visited`判断该节点的最短距离是否已经确定

#### 某个节点的距离已经确定之后，Dijstra会剪枝跳过

根据上面的逻辑，如果某个节点的最短距离是已经确定的，那么Dijstra就会跳过该节点，继续从队列中获取下一个记录。







##Leetcode 743 网络延迟时间

给你一个列表 `times`，表示信号经过 **有向** 边的传递时间。 `times[i] = (ui, vi, wi)`，其中 `ui` 是源节点，`vi` 是目标节点， `wi` 是一个信号从源节点传递到目标节点的时间。

现在，从某个节点 `K` 发出一个信号。需要多久才能使所有节点都收到信号？如果不能使所有节点收到信号，返回 `-1` 。

###核心思想

>该题的本质是 **单源最短路径问题**，即从一个节点（`K`）出发，计算到所有其他节点的最短时间。

要解决这个问题，有以下关键要求：

- 有向图；
- 每条边有权重（时间）；
- 要求最短路径；
- 如果有节点无法到达，返回 `-1`。

###关键数据结构

####邻接表

```Java
Map<Integer,List<int[]>> graph;
```

存储图结构，每个节点对应一个列表，列表里面是[neighbor, time]

```
List<int[]> times = graph.get(node1) //获取所有于node1邻接的节点以及距离
int time = time.get(node2)//获取从node1到node2的time
```

####最小堆/优先队列

```
PriorityQueue<int[]> pq = new PriorityQueue<>((a,b) -> a[1]-b[1]);
```

- 保存未处理的节点，堆顶是`距离原点距离最小`的节点

- 每次选择当前距离原点最近的节点进行扩展

###最短路径数组（或Map）

```
Map<Integer,Integer> dist;
```

- 记录从原点出发到每个节点的距离

###原理说明

####将图转化为邻接表表示方法

```java
Map<Integer,List<int[]>> graph = new HashMap<>();
for (int[] edge: times){
  int u = eadge[0],v=edge[1],w=edge[2];
  graph.computeIfAbsent(u,x-> new ArrayList<>().add(new int[]{v,w}));
}

```

####初始化最短距离数组

```
Map<Integer, Integer> dist = new HashMap<>();
```

####初始化优先队列

```
//优先队列中存放的元素同样是一个数组a[i]=x, 表示从k到i的距离是x（不一定是最小距离，没有经过确认）
PriorityQueue<int[]> pq = new PriorityQueue<>((a,b)->a[1]-b[1]);

//将起点元素本人添加至队列头
pq.offer(new int[]{k,0})
```

####遍历和操作优先队列

```java
//会进行不断的出队和入队的操作，一直到所有的元素都经历过队列
//每个过程其实都是取出队列头部的node, 然后更新其next nodes的值
 while (!pq.isEmpty()) {
            Distance distance = pq.poll();
            int node = distance.end;
   					// 为了减少重复访问，做个检查
            if (distance.distance > dist[distance.end]){
                continue;
            }

            List<int[]> nextNodes = graph.get(node);
            // 在有向图中，可能会存在一个node没有next 的情况，也可能会存在一个node有多个next的情况
            if (nextNodes ==null) continue;
						//
            for (int[] nextNode:nextNodes){
                int next = nextNodes[0];
                int time = nextNodes[1];
                if (dist[node]+time < dist[next]){
                    dist[next] = dist[node]+time;
                    pq.offer(new Distance(k, next, dist[next]));
                }
            }
        }
```

###### 查找最大时间值

```
for (int distance: dist){
            if (distance>maxDistance && distance != Integer.MAX_VALUE){
                maxDistance=distance;
            }
        }
        return maxDistance == 0 ?-1:maxDistance;
```

###规律和实践总结

####Dijkstra有两种存储dist的方法：dist[node]=min distance和Map<node,min distance >

#####Map方式：每一个节点只会被更新一次

```java
        int maxTime = 0;
        for (int i = 1; i <= n; i++) {
            if (dist[i] == Integer.MAX_VALUE) return -1;
            maxTime = Math.max(maxTime, dist[i]);
        }

        return maxTime;
```

- 一旦一个节点node被poll()出来，我们就立刻`dist.put(node,time)`;
- 后续即便有更短路径，我们也不会再次更新了

这里需要注意的是和传统的Dijkstra算法不一样，dist中的`每一个节点的值只会被更新一次`.

#####Dist[]方式：**每一个节点可能会被更新多次**

```java
while (!pq.isEmpty()){
  int current = pq.poll();
  for(each neighbor v of current[0]){
    if(dist[v]>dist[u]+w){
      dist[v]=dist[u]+v;
      pq.offer(v, dist[v]);
    }
  }
}
```



- 不管某个节点是不是已经访问过了，都允许不断入堆；

- 每次只要找到更短路径，就更新`dist[v]`;

- 同一个节点可能会被更新多次，直到最终路径收敛；

- **为了避免废路径访问，可以在`poll()`时加一个`剪枝`**;

  ```java
  if (time >dist[u]) continue;
  ```

  