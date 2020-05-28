# DFS&&BFS

## 102.二叉树的层序遍历

> #### [102. 二叉树的层序遍历](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)
>
> 难度中等460
>
> 给你一个二叉树，请你返回其按 **层序遍历** 得到的节点值。 （即逐层地，从左到右访问所有节点）。
>
>  
>
> **示例：**
> 二叉树：`[3,9,20,null,null,15,7]`,
>
> ```
>     3
>    / \
>   9  20
>     /  \
>    15   7
>
> ```
>
> 返回其层次遍历结果：
>
> ```
> [
>   [3],
>   [9,20],
>   [15,7]
> ]
> ```

### 1.DFS

```java
	List<List<Integer>> result = new ArrayList<>();
    //1.DFS 将每一次用k记录 添加到对应的层级
	// time :O(n) 
    public List<List<Integer>> levelOrder(TreeNode root) {
        if(root == null)    return result;
        recur(root,0);
        return result;
    }

    private void recur(TreeNode root,int k){
        if(root != null){
            if(result.size()<=k){
                result.add(new ArrayList<>());
            }
            result.get(k).add(root.val);
            recur(root.left,k+1);
            recur(root.right,k+1);
        }
    }	
```

### 2.BFS

```java
//2.BFS 层序遍历 将每层节点添加到queue中 
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> result = new ArrayList<>();
        if(root == null)    return result;
        Queue<TreeNode> queue = new LinkedList<>();

        //1.添加首节点
        queue.add(root);
        while(!queue.isEmpty()){
            int cnt = queue.size();
            List<Integer> list = new ArrayList<>();
            while(cnt-->0){
                TreeNode node = queue.poll();
                if(node == null){
                    continue;
                }
                list.add(node.val);
                queue.add(node.left);
                queue.add(node.right);
            }
            if(queue.size()!=0){
                result.add(list);
            }
        }
        return result;
    }
```

## [200. 岛屿数量](https://leetcode-cn.com/problems/number-of-islands/)

> #### [200. 岛屿数量](https://leetcode-cn.com/problems/number-of-islands/)
>
> 难度中等574
>
> 给你一个由 `'1'`（陆地）和 `'0'`（水）组成的的二维网格，请你计算网格中岛屿的数量。
>
> 岛屿总是被水包围，并且每座岛屿只能由水平方向或竖直方向上相邻的陆地连接形成。
>
> 此外，你可以假设该网格的四条边均被水包围。
>
>  
>
> **示例 1:**
>
> ```
> 输入:
> 11110
> 11010
> 11000
> 00000
> 输出: 1
> ```
>
> **示例 2:**
>
> ```
> 输入:
> 11000
> 11000
> 00100
> 00011
> 输出: 3
> 解释: 每座岛屿只能由水平和/或竖直方向上相邻的陆地连接而成。
> ```

### dfs

```java
/*
     遍历岛屿 
     a.当遇到1 将岛屿变成2 依次将连着的岛屿 上下左右都改变、
     b.判断上下界限就可以了。
     time : O(M*N)
     space O(1)
    */
    public int numIslands(char[][] grid) {
        int res = 0;
        for(int i=0;i<grid.length;i++){
            for(int j=0;j<grid[0].length;j++){
                if(grid[i][j] == '1'){
                    res++;
                    dfs(grid,i,j);
                }
            }
        }
        return res;
    }

    private void dfs(char [][] grid,int i,int j){
        if(i < 0 || j < 0 || i >= grid.length || j >= grid[0].length || grid[i][j] != '1') return;
        grid[i][j] = '2';
        dfs(grid,i-1,j);
        dfs(grid,i+1,j);
        dfs(grid,i,j-1);
        dfs(grid,i,j+1);
    }
```

