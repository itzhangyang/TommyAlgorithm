# BFS 模板：一个或者多个起点一层一层地`向外扩展`的搜索方式。

>  根二叉树的BFS遍历类似

##核心思想

> BFS 是一层一层地`向外扩展`的搜索方式。

- 从起点开始，先访问所有距离为1的节点，再访问所有距离为2的节点....
- 类似于`波浪` 一样，一圈一圈扩散出去
- 通常用于找`最短路径`，`层级遍历`, `范围扩散/感染`

> 找最短路径用 BFS！用 DFS 无法保证走到终点时就是最短路径。

##原理

1. 使用队列实现先进先出

   > 确保`先被访问的节点，线扩展`

2. 记录访问状态（`visited`或者`grid原地修改`）

   > 防止重复访问

3. 从起点出发，循环以下过程

   - 拉出队列头部的节点(node)
   - 遍历它的邻居(`上下左右的邻居`)
   - 如果邻居满足以下条件，就入对并且标记访问
     - 未访问
     - 合法
     - 满足条件

##代码实现模板

```java
Queue<int[]> queue = new LinkedList<>();
int[][] dirs = {{0,1}, {1,0}, {0,-1}, {-1,0}};

// 从某个起点开始
queue.offer(new int[]{x, y});
while (!queue.isEmpty()) {
    int[] cur = queue.poll();
    for (int[] d : dirs) {
        int nx = cur[0] + d[0];
        int ny = cur[1] + d[1];
        if (合法 && 没访问过 && 满足条件) {
            queue.offer(new int[]{nx, ny});
           // 标记为已访问;
        }
    }
}
```

##记录遍历层次数的做法 - Queue.Size

- 定义一个`steps`

- 在遍历`Queue` 的时候保证每次while循环会访问一整层的节点，这样下一次while循环开始的时候queue里面也正好是一整层节点

  ```java
  while (!queue.isEmpty()) {
      int size = queue.size();//
      for (int k=0;k<size;k++){//保证一次拿出k个节点
        int[] cur = queue.poll();
      	for (int[] d : dirs) {
          int nx = cur[0] + d[0];
          int ny = cur[1] + d[1];
          if (合法 && 没访问过 && 满足条件) {
              queue.offer(new int[]{nx, ny});
             // 标记为已访问;
          }
      }
      }
  }
  ```

  > “BFS 层数控，先 size 再 for 中；
  > 一轮走完加一步，不慌不乱最短通。”

##Leetcode 909: 蛇梯棋-BFS实现

> 你玩的是一个蛇梯棋游戏，有一个大小为 `n x n` 的棋盘（`n` 是偶数），编号从 1 到 n²，编号方式是 **“蛇形”** 编号，如下：
>
> ```
> 示例：n = 6，编号方式如下（蛇形，从左往右再右往左交替）：
> 
> 36 35 34 33 32 31
> 25 26 27 28 29 30
> 24 23 22 21 20 19
> 13 14 15 16 17 18
> 12 11 10  9  8  7
> 1  2  3  4  5  6
> ```
> 
>棋盘中可能出现蛇或梯子，棋盘用一个二维数组 `board` 表示，`board[i][j]`：
> 
>- `-1` 表示该格子是普通格；
> - 否则是某个数字 `x`，表示当你到达这个格子时，会直接移动到编号为 `x` 的格子（蛇或梯子）。
> 
>你每次投骰子，可以掷出 1 到 6 的点数，代表可以从当前格子往后走 1~6 步。
> 
>你从编号 1 的格子出发，目标是走到编号 `n*n` 的格子，返回最少需要的步数。
> 
>如果无法到达终点，返回 -1。

###核心思想-BFS

将题目中的棋盘以及蛇和梯子转化为一个图,然后使用`bfs`来寻找节点1～n*n最短路径。

####从节点1出发，是否真的可能到达$n^2$?

如果运气好的话，每次投骰子的点数合理的话，是可能会到达的。

所以我们需要计算每次投骰子，可能投到的所有的情况。



####怎么将棋盘和梯子和蛇转化为图？

##### 怎样互相映射矩阵索引和编号？

二维矩阵的索引就是`row`和`col`,值的范围都是`[0...n-1]`。

编号就是题目中规定的$[1...n^2]$， 但是其规则是比较特殊的。

对于我而言，这个题目最大的挑战在于如何总结归纳**矩阵元素二维索引和编号之间的映射关系**。

###### 怎样将二维索引映射到编号？

首先我们知道其计算顺序是**从下向上**的。

- $row = n - 1$ ，其值的范围是 $[1...n]$ ( $col + 1$)
  - $id = col + 1$
- $row = n-2$， 其编号的顺序是$[2n...n+1]$
  - $col = 0, id = 2n$
  - $col = 1, id = 2n -1$
  - $col = 2, id = 2n - 2$
  - $col = 3, id = 2n - 3$
  - ...
  - $col = n-1, id = 2n - (n-1) = n + 1$
  - 可以推导出来 $id = 2n - col$
- $row = n-3$，其编号的范围是$[2n+1...3n]$
  - $col = 0, id = 2n+1$
  - $col = 1, id = 2n+2$
  - $col = 2, id = 2n + 3$
  - $col = 3, id = 2n+4 $
  - $...$ 
  - $col = n-1, id = 2n +(n-1)+1 = 3n$
  - 可以推导出来：$id = 2n + col + 1$
- $row = n - 4$, 其编号范围是$[4n,3n+1]$
  - $col = 0, id = 4n $
  - $col = 1, id = 4n -1$
  - ....
  - $4n - col$
- $row = n - 5$， 其编号范围是$[4n+1..5n]$
  - $col = 0, id = 4n + 1 $
  - $col = 1, id = 4n +2$
  - $col = n-1, id = 5n$
  - 可以推导出：$id = 4n + n -1 +1 = 4n + col + 1$

**结论**

- 对于 $n - row$是是奇数的情况下：

  $id = ((n - row) - 1) n + col + 1$

- 对于$n - row$是偶数的情况下：

  $id = (n - row)*n - col$

不过，似乎我们并不需这个公式。

看上去我们需要的是将编号转化为二维索引的能力，就是反向映射。

###### 怎样将编号映射到二维索引？

现在我们尝试将$id$解析为$row$和$col$。

1. 寻找当前$id$所在的**自下而上**的$row$

   `rowFromBottom = (id - 1) % n`

2. 计算出从左往右数的列$col$

   `col = (id - 1) % n`
3. 计算出实际的行号（**从上往下数**第几行）

   `int r = (n-1) - rowFromBottom`
4. 根据从下往上数的行号的奇偶性，决定列号

   ```Java
   if(rowFromBottom % 2 == 0){
     c = col;
   }else{
     c = (n-1) - col;
   }
   ```
5. 返回最终计算的行号和列号

   ```Java
   return new int[]{r,c};
   ```


- 每一个二维数组中的元素可以转化为图中的一个节点
- 投骰子，可以有两种情形转化为一个边
  - 投骰子投到了一个普通节点`-1`,那就存在一个边`current-selectedNode`
  - 投骰子投到了一个特殊节点`x`,那就存在另外一个边 `current-x`

#####怎么获取每个节点的所有邻居？通过骰子的六个值

既然要生成图，我们就需要明白怎样获得当前节点的邻居节点？

首先先跳过传统矩阵类题目的常识，在本题中，当前节点的邻居节点并不是上下左右四个节点。

通过骰子的六个值可以计算出当前节点的**六个邻居节点**。

##### 算法原理

该算法的实现核心有两个非常需要注意的地方

- 图的构建过程，在生成边的时候需要考虑投骰子的情况
  - 骰子投到一般节点
  - 骰子投到跳跃节点
- 投骰子次数也就相当于是图的遍历层数，需要注意怎么计算遍历的层数

###### 从二维数组构建图

注意跳跃节点和一般节点的处理逻辑。

```java
    public Map<Integer,List<Integer>> getGraph(int[][] board){
        Map<Integer,List<Integer>> graph = new HashMap<>();
        int n = board.length;
        for (int s=1;s<=n*n;s++){
            List<Integer> neighbors = new ArrayList<>();
            // 投骰子，骰子的值从1点到6点
            for (int dice =1;dice<=6;dice++){
                //找到骰子所指向的节点
                int next = s+dice;
                //判断是否越界
                if (next > n * n) break;
                // 获得骰子所指向的节点具体索引[i,j]
                int[] position = getPosition(next, n);
                int i = position[0],j=position[1];
                //获得骰子所指向的节点的值
                int target = board[i][j];
                //如果节点是普通节点，将节点加入邻居节点列表
                if (target == -1){
                    neighbors.add(next);
                }else{
                //如果节点是跳跃节点，将跳跃目标节点加入邻居节点
                    neighbors.add(target);
                }
            //构建图的邻接表
            graph.put(s, neighbors);
            }
        }
        return graph;
    }
```

###### 使用BFS遍历图求得最小遍历层数

图的**邻接表** 构建完成之后，使用**BFS层序遍历**，采用**逐层遍历**的方式，每次遍历一个层的节点。

这里每一层的节点表示投了一次骰子，因此第一次到达节点的时候所需要的层数也就是最少投骰子的次数。

求当前的遍历层数的最佳实践也在如下的代码中：

```java
    // 其实就相当于是图，蛇或者梯子表示图的边，b[i][j]=x, 表示b[i][j](id=?需要计算出来，i为偶数的情况下id=)和id=x之间的节点存在一条边
    public int snakesAndLadders(int[][] board) {
      //构建邻接表
        Map<Integer,List<Integer>> graph = getGraph(board);
        int n = board.length;
        Queue<Integer> queue = new LinkedList<>();
        int steps =0;
        queue.offer(1);
        Set<Integer> visited = new HashSet<>();
        
        while (!queue.isEmpty()) {
            //加上如下for循环，可保证每次while循环访问的是一整个层次的节点
            int size = queue.size();
            for (int k=0;k<size;k++){
            int node = queue.poll();
            //到达目标节点，返回层数
            if (node == n*n) return steps;
            //获取节点的邻居节点列表
            List<Integer> nextNodes = graph.getOrDefault(node,new ArrayList<>());
            for (Integer next: nextNodes){
                if (!visited.contains(next)){
                    visited.add(next);
                    queue.offer(next);
                }
            }
            }
            steps++;
        }
        return -1;
    }
```

###### 辅助函数getPosition的实现

数字和board映射之间的规则：

- 从下往上
- 如果行索引为偶数，从左到右
- 如果行索引为奇数，从右到左

**计算的规则：**

1. 计算出**从下往上**数数字所在的行树

   `int rowFromBottom = (num-1) % n`

2. 计算出现在该行中如果从左往右数，应该是第几个(`o-indexed`)

   `int col = (num - 1) % n`

3. 计算出实际的行号（**从上往下数**第几行）

   `int r = (n-1) - rowFromBottom`

4. 根据从下往上数的行号的奇偶性，决定列号

   ```Java
   if(rowFromBottom % 2 == 0){
     c = col;
   }else{
     c = (n-1) - col;
   }
   ```

5. 返回最终计算的行号和列号

   ```
   return new int[]{r,c};
   ```

   

#### 算法最佳实践

##### 明确建模方式是关键第一步

**原则**：**先建模，再写代码。**抽象出图结构后问题就变得清晰易解。

##### **最短步数问题优先考虑 BFS**

如果题目问“最少投骰子次数”、“最短路径”这类问题，优先考虑 **BFS（Breadth-First Search）** 而不是 DFS。

BFS 会一层层地扩展路径，保证第一次到达目标节点的路径就是最短路径。

**原则**：**最短路径 → BFS；全部路径 → DFS；最短加带权 → Dijkstra。**

#####  **图建完以后，能用 BFS 不要自己加 visited size 累加器**

一层层扩展 queue，每一轮 +1 步数，不要在 `for` 循环里对步数乱加。

**最佳实践**：用 `queue.size()` 控制 BFS 层级，一轮投一次骰子。

##### visited 控制访问

✅ 加入 queue 前先 visited.add

#### 使用BFS计算步数的时候的最佳实践

##### 理解BFS 的本质： 一层一层的向外扩展

**第 0 层**: 只有起始节点。

**第 1 层**: 所有与起始节点直接相连的节点（距离为 1）。

**第 2 层**: 所有与第 1 层节点直接相连，但尚未访问过的节点（距离为 2）。

...依此类推。

我们的 `steps` 变量，其实就是在记录当前正在处理的是**第几层**的节点，也就是记录**距离起点的步数**。

##### Step初始值应当为0

##### Step++操作应当在for循环之后

为了保证逻辑的清晰和结果的正确，请牢记这个标准模式：

1. **`int steps = 0;`** —— 到达起点需要 0 步。
2. **`while` 循环** —— 只要还有节点待处理就继续。
3. **`for (int i = 0; i < size; i++)`** —— 遍历当前层的所有节点。
4. **`steps++;`** —— 在 `for` 循环之后，处理下一层之前，步数加一。

这个模板适用于所有用 BFS 求解无权图中最短路径/最少步数的问题。希望这次的分析能帮你彻底搞清楚这个混淆点！



#### 实现细节注意事项

##### 编号和棋盘坐标的映射很常见

这个题的编号是蛇形递增的，映射过程是：

- 编号 → 坐标：用于判断跳跃；
- 坐标 → 编号：用于构图或调试。

##### 注意边界条件

//TBD