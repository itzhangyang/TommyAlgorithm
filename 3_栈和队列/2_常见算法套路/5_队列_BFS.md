### 队列+BFS（广度优先遍历）

#### 核心思想

使用队列从起点开始一层层扩散，适合**最短路径**、**状态转移**等问题。

#### 实现原理

- 初始化遍历起点
- 每轮遍历当前队列所有元素（**层**）
- 将遍历节点的**邻居节点** 或者 **子节点**加入队列

#### 代码套路

```java
Queue<TreeNode> queue = new LinkedList<>();
queue.offer(root);

while (!queue.isEmpty()){
  int size = queue.size();
  for (int i =0; i<size;i++){
    TreeNode node = queue.poll();
    // 处理当前节点
    // ...
  	List<TreeNode> children = node.children();
    //将子节点加入队列
    for (TreeNode child : children){
      queue.offer(child);
    }
  }
}
```

#### 适用场景

- 图的**最短路径**
- 树的**层次遍历**
- **状态扩展**类问题

#### Leetcode 102: Binary Tree Level Order Traversal

给你二叉树的根节点 `root` ，返回其节点值的 **层序遍历** 。 （即逐层地，从左到右访问所有节点）。

##### 核心思想:BFS+确保每一层的节点放在一个list当中

##### 基于Queue的BFS套路

```java
Queue<Node> queue = new LinkedList<>();
queue.offer(root);

while (!queue.isEmpty()){
	Node node = queue.poll();
  //process the node
  if (node.left != null) queue.offer(node.left);
  if (node.right != null) queue.offer(node.right);
}
```

##### 怎样确保每次都遍历一整层的节点？保存`queue.size()`.

```java
while (!queue.isEmpty()) {
            //确保每一次
            int size = queue.size();
            List<Integer> nodes = new ArrayList<>();
            for (int i=0;i<size;i++){
                TreeNode node = queue.poll();
                nodes.add(node.val);
                if (node.left != null) queue.offer(node.left);
                if(node.right != null) queue.offer(node.right);
            }
            res.add(nodes);
        }
```

##### BFS 套路模板

```java
public List<List<Integer>> levelOrder(TreeNode root) {
        if (root == null) return new ArrayList<>();
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);

        List<List<Integer>> res = new ArrayList<>();

        while (!queue.isEmpty()) {
            //确保每一次
            int size = queue.size();
            List<Integer> nodes = new ArrayList<>();
            for (int i=0;i<size;i++){
                TreeNode node = queue.poll();
                nodes.add(node.val);
                if (node.left != null) queue.offer(node.left);
                if(node.right != null) queue.offer(node.right);
            }
            res.add(nodes);
        }

        return res;
    }
```



#### Leetcode 111: Minimum Depth of Binary Tree

##### 代码实现

```java
// 找到一叶子节点就停止遍历
    public int minDepth(TreeNode root) {
        if( root == null) return 0;
        Queue<TreeNode> queue = new LinkedList<>();
        int depth = 0;
        queue.offer(root);

        while(!queue.isEmpty()){
          // 注意在什么时刻执行depth += 1 操作
            depth++;
            int size = queue.size();
            for (int i=0; i<size; i++){
                TreeNode node = queue.poll();
                if (node.left == null && node.right == null){
                    return depth;
                }
                if (node.left != null){
                    queue.offer(node.left);
                }
                if (node.right != null ){
                    queue.offer(node.right);
                }
            }
        }
    return depth;
    }
```



#### Leetcode 994 Rotting Oranges

- 0 代表空单元格；
- 1 代表新鲜橘子；
- 2 代表腐烂橘子。

**每分钟**，腐烂橘子会使其上下左右四个方向上的新鲜橘子变成腐烂橘子。

**目标：** 求使所有新鲜橘子腐烂的最少分钟数；如果无法全部腐烂，返回 -1。

##### 问题本质

**最短传播路径问题：**属于**多源广度优先搜索**

- **起点**：所有初始状态为腐烂(`2`)的橘子
- **扩散方式：** 每分钟向四周新鲜橘子扩散
- **终点：** 所有橘子都被腐烂

##### 核心思想- BFS+多源并行传播+层数统计

从所有腐烂的橘子出发，按分钟向外**一层一层**扩散

每轮扩散即一分钟，直到没有新鲜橘子为止

##### 实现原理

1. **初始状态：** 用队列存储当前轮的所有腐烂橘子
2. 每次从队列中取出一个腐烂橘子，向上下左右检查是否有新鲜橘子
3. 若发现有新鲜橘子，腐烂它并加入队列（下一轮传播）
4. 统计传播的分钟数（层数），直到队列为空
5. 最后检查还有没有新鲜的橘子存在，决定是否返回-1'

##### 数据结构以及定义

| 数据结构         | 含义                                     |
| ---------------- | ---------------------------------------- |
| `Queue<int[]>`   | BFS队列，存储当前腐烂橘子的位置 `[x, y]` |
| `int[][] grid`   | 原始网格，表示橘子状态                   |
| `int freshCount` | 当前新鲜橘子的数量，用于判断是否全部腐烂 |
| `int minute`     | 扩散所需的分钟数，最终结果               |

##### 实现步骤

###### 初始化队列和新鲜橘子计数器

- 遍历整个网络，把所有腐烂橘子加入队列
- 同时统计新鲜橘子的数量

###### BFS扩散过程

- 每一轮队列大小为一层
- 将当前腐烂橘子向四周扩散，队新鲜橘子腐烂，加入新队列
- 每轮扩散后分钟数+1

###### 返回结果

- 如果所有橘子都腐烂完，返回所需分钟数
- 否则返回`-1`

##### 代码实现

```java
    class Node {
        int i;
        int j;
        int val;
        public Node(int i, int j,int val){
            this.i=i;
            this.j = j;
            this.val = val;
        }
    }

    // bfs思路，最小分钟数是bfs遍历所有
    public int orangesRotting(int[][] grid) {
        int rows = grid.length;
        int cols = grid[0].length;

        int[][] directions = {{1,0},{0,1},{-1,0},{0,-1}};

        int minutes = 0;
        int freshOranges = 0;

        Queue<Node> queue = new LinkedList<>();
        
        for (int i=0; i < rows; i++){
            for (int j = 0; j < cols; j++){
                if (grid[i][j]==2){
                    queue.offer(new Node(i, j, grid[i][j]));
                }else if (grid[i][j] == 1){
                    freshOranges++;
                }
            }
        }

        while (!queue.isEmpty()){
            int size = queue.size();
          	
            boolean hasNewRot = false;
            for (int i=0; i < size; i++){
                Node node = queue.poll();
                for (int[] d: directions){
                    int r = node.i+d[0];
                    int c = node.j+d[1];
                    if (r >=0 && r<rows && c >=0 && c < cols && grid[r][c] == 1){
                        grid[r][c] = 2;
                        queue.offer(new Node(r, c, 2));
                        freshOranges--;
                        hasNewRot=true;
                    }
                }
            }
          //因为第一轮腐烂橘子，不需要时间，因此从第二轮开始增加时间
            if (hasNewRot){
                    minutes++;
            }
        }

        return freshOranges == 0 ? minutes : -1;
    }
```

##### BFS计算层数+1的时候，应该在读取队列之前还是之后？

###### 如果需要计算第一层，则是在之前

在Leetcode 111中，需要遍历二叉树的最小深度，则需要计算第一层，所以层数+1操作应该在读取本层队列之前

###### 如果不需要计算第一层，则是在之后

在本题目中，已经腐烂的橘子在第一轮，不需要时间，因此不需要计算第一层，所以层数+1操作应该在读取本层队列之后

##### 代码实现

```java
    class Node {
        int i;
        int j;
        int val;
        public Node(int i, int j,int val){
            this.i=i;
            this.j = j;
            this.val = val;
        }
    }

    // bfs思路，最小分钟数是bfs遍历所有
    public int orangesRotting(int[][] grid) {
        int rows = grid.length;
        int cols = grid[0].length;

        int[][] directions = {{1,0},{0,1},{-1,0},{0,-1}};

        int minutes = 0;
        int freshOranges = 0;

        Queue<Node> queue = new LinkedList<>();
        
        for (int i=0; i < rows; i++){
            for (int j = 0; j < cols; j++){
                if (grid[i][j]==2){
                    queue.offer(new Node(i, j, grid[i][j]));
                }else if (grid[i][j] == 1){
                    freshOranges++;
                }
            }
        }

        while (!queue.isEmpty()){
            int size = queue.size();
            boolean hasNewRot = false;
            for (int i=0; i < size; i++){
                Node node = queue.poll();
                for (int[] d: directions){
                    int r = node.i+d[0];
                    int c = node.j+d[1];
                    if (r >=0 && r<rows && c >=0 && c < cols && grid[r][c] == 1){
                        grid[r][c] = 2;
                        queue.offer(new Node(r, c, 2));
                        freshOranges--;
                        hasNewRot=true;
                    }
                }
            }
            if (hasNewRot){
                    minutes++;
            }
        }

        return freshOranges == 0 ? minutes : -1;
    }
```

##### 需要注意的事项

- **边界判断**，需要确保邻居节点的下标不越界，包括`row`和`col`下标
- 只有新橘子被感染才需要+1分钟，否则不需要增加时间
- 初始状态下没有新橘子，直接返回0；
- 新鲜橘子无法接触腐烂橘子，最终返回-1；
- 广度优先搜索不能清空初始腐烂位置，需要用队列记录层级

#### Leetcode 752: Open the lock

你有一个带四个转轮的转盘锁，每个转轮上有数字 `'0'` 到 `'9'`。每次可以把一个轮子 forward（+1）或 backward（-1），形成一个新的状态。

- 初始状态是 `"0000"`。
- 给定一个 deadends（死亡锁）列表和一个 target（目标锁）字符串。
- 如果某个状态在 deadends 中，则该状态无法使用。
- 返回从 `"0000"` 到 target 的最小步数。如果无法到达，返回 -1。

##### 问题的本质-图的单源BFS最短路径问题

###### BFS最短路径是层数而不是节点数

本题目的本质是求源点`0000`到指定的`target`需要径路的最短路径（层数）

###### 图的搜索问题

 每个锁的状态是一个节点，每次拨动一个轮子可以生成最多 8 个相邻状态，相当于在图中进行**无权最短路径搜索**。

- 状态空间大小：$10^4$（0000~9999）
- 每个状态最多有 8 个相邻状态（每个轮子可以向前或向后转）

##### 核心思想

###### 使用BFS寻找最短路径

- BFS是逐层遍历，最先访问到的`target`一定是**最短路径**
- 每个状态最多有8个**相邻状态**

###### 使用队列维护访问的状态+当前步数

- `Queue<状态，当前步数>`

###### 使用 Set 记录访问过的状态

##### 实现原理

###### 状态转换和生成逻辑

从当前状态生成下一个状态的方式：

- 对于每一位数字，向上转`+1`和向下转`-1`，用字符串拼接成新状态

```java
"1234" → 转第一位： "2234", "0234"
"1234" → 转第二位： "1334", "1134"
... 以此类推
```

###### 终止条件

一旦弹出的状态是`target`，就返回**当前步数**

###### 剪枝条件

在`deadends`或者已经访问过的状态中直接跳过

##### 实现代码

```java
public int openLock(String[] deadends, String target){
  Set<String> deadSet = new HashSet<>(Arrays.asList(deadends));
  if (deadSet.contains('0000')) return -1;
  
  Queue<String> queue = new LinkedList<>();
  Set<String> visited = new HashSet<>();
  
  queue.offer("0000");
  visited.add("0000");//注意考虑visited应该在什么时候添加的问题
  
  int steps = 0;
  
  while (!queue.isEmpty()){
    int size = queue.size();
    
    for (int i=0; i<size; i++){
      String current = queue.poll();
      if (current.equals(target)) return steps;
      
      List<String> nextStates = getNextStates(current);
      
      for (String next: nextStates){
        if (!deadSet.contains(next) && !visited.contains(next)){
          queue.offer(next);
          visited.add(next);
        }
      }
      //初始状态并不计算步数，因此在读取队列之后+1
      steps++
    }
  }
  
  return -1;
}

private List<String> getNextStates(String state){
  List<String> result = new ArrayList<>();
  
  char[] chars = state.toCharArray();
  
  for (int i=0;i<4;i++){
    char orginal = chars[i];
    
    //向上转动
    chars[i] = original == '9' ? '0' : (char)(orginal+1);
    result.add(new String(chars));
    
    //向下转动
    chars[i] = orginal == '0' ? '9' : (char)(orginal-1);
    result.add(new String(chars));
    
    //复原
    chars[i] = orginal;
  }
  
  return result;
}
```



#### Leetcode 773: Sliding Puzzle