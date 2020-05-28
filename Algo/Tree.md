[TOC]



![](e://pic/搜狗截图20200329191409.png)



# 树

## 94.二叉树的中序遍历

> 给定一个二叉树，返回它的中序 遍历。
>
> 示例:
>
> 输入: [1,null,2,3]
>    1
>     \
>      2
>     /
>    3
>
> 输出: [1,3,2]

### 1.递归法

我们知道二叉树的遍历有前序遍历 中序遍历 后序遍历。

前序遍历 :根左右

中序遍历:左根右

后序遍历:左右根

时间复杂度:O(n)。递归函数 T(n) = 2*T(n/2)+1 

空间辅助度:最坏情况下需要空间O(n),平均情况为O(log N)

```java
 public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList();
        helper(root,result);
        return result;
    }

    public static void helper(TreeNode root,List<Integer> result){
        if(root!=null){
            //左
            if(root.left!=null){
                helper(root.left,result);
            }
            //中
            result.add(root.val);
            //右
            if(root.right!=null){
                helper(root.right,result);
            }
        }
    }
```

### 2.基于栈的遍历

用一个栈接收访问的路径 因为栈是先进后出 所以最后访问的元素就是最先遍历的元素。先查找左节点 左节点完毕后 输出，查找右节点。

时间复杂度:O(n)

空间复杂度:O(n)

```java
 public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList();
        Stack<TreeNode> stack = new Stack();
        //获取根节点 
        TreeNode curr = root;
        //如果当前节点不为null or 栈不为null 说明还有节点没有遍历完
        while(curr != null || !stack.empty()){
            //先将左节点push 进
            if(curr!=null){
                stack.push(curr);
                curr = curr.left;
            }
            //后push的 先输出
            curr = stack.pop();
            result.add(curr.val);
            //查看当前节点的右节点 
            curr = curr.right;
        }

        return result;
    }
```



## 144.二叉树的前序遍历

> 给定一个二叉树，返回它的 前序 遍历。
>
>  示例:
>
> 输入: [1,null,2,3]  
>    1
>     \
>      2
>     /
>    3 
>
> 输出: [1,2,3]
>

### 1.递归法

```java
public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> result = new ArrayList();
        helper(root,result);
        return result;
    }

    public static void helper(TreeNode root,List<Integer> result){
        if(root!=null){
            //根
            result.add(root.val);
            //左
            if(root.left != null){
                helper(root.left,result);
            }
            //右
            if(root.right != null){
                helper(root.right,result);
            }
        }
    }
```

### 2.基于栈的遍历

其实我们使用递归去输出二叉树的遍历，计算机也是利用栈进行操作，我们可以模拟栈进行操作、

栈是先进后出的线性结构。因此 前序遍历是根左右  应该先push rootNode 输出。剩下就是左右节点的遍历，应该先push右节点，然后在push左节点，这样在pop的时候 顺序就是左右。

时间复杂度:O(n) 相当于树进行了一次loop操作

空间复杂度:O(n) 用栈做临时空间存储。

```java
public List<Integer> preorderTraversal(TreeNode root) {
        if(root == null){
            return new ArrayList();
        }

        List<Integer> result = new ArrayList();
        Stack<TreeNode> stack = new Stack();

        //先将头结点条件进去
        stack.push(root);

        while(!stack.isEmpty()){
            TreeNode node = stack.pop();
            result.add(node.val);
            //添加右节点 因为栈是先进后出 而前序遍历是根左右 所以右节点陷进去最后出来
            if(node.right != null){
                stack.push(node.right);
            }
            //添加左节点 
            if(node.left != null){
                stack.push(node.left);
            }

        }
        return result;
    }
```

## 590.N叉数的后序遍历

> #### [590. N叉树的后序遍历](https://leetcode-cn.com/problems/n-ary-tree-postorder-traversal/)
>
> 难度简单58
>
> 给定一个 N 叉树，返回其节点值的*后序遍历*。
>
> 例如，给定一个 `3叉树` :
>
> ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/12/narytreeexample.png)
>
>  
>
> 返回其后序遍历: `[5,6,3,2,4,1]`.
>
>  

### 1.递归

时间复杂度:O(n)

空间复杂度:O(n)

```java
LinkedList<Integer> linkedList = new LinkedList();
    //1.一个LikedList链表存储数值
    //2.递归调用
    // a.如果根节点为null 返回
    // b.依次遍历根节点的孩子节点 
    public List<Integer> postorder(Node root) {
        recur(root);
        return linkedList;
    }

    public void recur(Node root){
        if(root == null) return;
        for(Node node : root.children){
            recur(node);
        }
        linkedList.add(root.val);
    }
```

### 2.栈迭代

时间复杂度:O(n)

空间复杂度:O(n)

```java
 	// 1.N叉树的后序遍历 左右根 
    // 2.用一个链表存储数据 每次添加首节点 依次往后移动
    // 3.将root节点push到栈中 遍历左右子节点。如果不为null 继续
    public List<Integer> postorder(Node root) {
        if(root == null) return new ArrayList();
        LinkedList<Integer> linkedList = new LinkedList();
        Stack<Node> stack = new Stack();
        stack.push(root);
        while(!stack.isEmpty()){
            root = stack.pop();
            linkedList.offerFirst(root.val);//先进排在最后
            for(Node node : root.children){
                stack.push(node);
            }
        }
        return linkedList;
    }
```

## 589.N叉树的前序遍历

> #### [589. N叉树的前序遍历](https://leetcode-cn.com/problems/n-ary-tree-preorder-traversal/)
>
> 难度简单71
>
> 给定一个 N 叉树，返回其节点值的*前序遍历*。
>
> 例如，给定一个 `3叉树` :
>
>  
>
> ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/12/narytreeexample.png)
>
>  
>
> 返回其前序遍历: `[1,3,5,6,2,4]`。

### 1.递归

```java
//前序遍历  根左右  递归调用即可。
    //先将节点的值存储到linkedList中

    LinkedList<Integer> linkedList = null;
    public List<Integer> preorder(Node root) {
        linkedList = new LinkedList();
        return root == null ? new ArrayList() : recur(root);
    }

    public LinkedList<Integer> recur(Node root){
        if(root == null)  return null;
        linkedList.add(root.val);
        for(Node node : root.children){
            recur(node);
        }
        return linkedList;
    }
```

### 2.栈迭代

```java
	//迭代
    //前序遍历 根-左-右
    //1.将根节点存储到stack中
    //2.逆序将子节点添加到栈中 比如 push的顺序为 2 3 4  则输出的顺序为 4 3 2 
    public List<Integer> preorder(Node root) {
        if(root == null) return new ArrayList();
        List<Integer> list = new ArrayList();
        Stack<Node> stack = new Stack();

        stack.push(root);

        while(!stack.isEmpty()){
            root = stack.pop();
            list.add(root.val);
            for(int i=root.children.size()-1;i>=0;i--){
                stack.push(root.children.get(i));
            }
        }
        return list;
    }
```

## 429.N叉数的层序遍历

> #### [429. N叉树的层序遍历](https://leetcode-cn.com/problems/n-ary-tree-level-order-traversal/)
>
> 难度中等76
>
> 给定一个 N 叉树，返回其节点值的*层序遍历*。 (即从左到右，逐层遍历)。
>
> 例如，给定一个 `3叉树` :
>
> ![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/10/12/narytreeexample.png)
>
>  
>
> 返回其层序遍历:
>
> ```
> [
>      [1],
>      [3,2,4],
>      [5,6]
> ]
> ```

### 1.队列实现广度优先搜索

time:O(n)

space:O(n)

```java
	//1.用队列实现广度优先搜索
    //2.一个list 一个queue  list存储节点的值  queue存储每一层遍历的节点。
    //3.当queue != null 的时候，遍历当前层。
    public List<List<Integer>> levelOrder(Node root) {
        if(root == null) return new ArrayList();
        List<List<Integer>> list = new ArrayList<>();
        Queue<Node> queue = new LinkedList();//存储每一层的结点值
        queue.add(root);
        while(!queue.isEmpty()){
            List<Integer> level = new ArrayList<>();
            int size = queue.size();
            for(int i=0;i<size;i++){
                Node node = queue.poll();
                level.add(node.val);
                queue.addAll(node.children);
            }
            list.add(level);
        }
        return list;
    }
```



## [222. 完全二叉树的节点个数](https://leetcode-cn.com/problems/count-complete-tree-nodes/)

> #### [222. 完全二叉树的节点个数](https://leetcode-cn.com/problems/count-complete-tree-nodes/)
>
> 难度中等156
>
> 给出一个**完全二叉树**，求出该树的节点个数。
>
> **说明：**
>
> [完全二叉树](https://baike.baidu.com/item/%E5%AE%8C%E5%85%A8%E4%BA%8C%E5%8F%89%E6%A0%91/7773232?fr=aladdin)的定义如下：在完全二叉树中，除了最底层节点可能没填满外，其余每层节点数都达到最大值，并且最下面一层的节点都集中在该层最左边的若干位置。若最底层为第 h 层，则该层包含 1~ 2h 个节点。
>
> **示例:**
>
> ```
> 输入: 
>     1
>    / \
>   2   3
>  / \  /
> 4  5 6
>
> 输出: 6
> ```

```java
public int countNodes(TreeNode root) {
        if(root == null){
            return 0;
        }

        int left = countHigh(root.left);
        int right = countHigh(root.right);
        if(left == right){
            return countNodes(root.right) + (1<<left);
        }else{
            return countNodes(root.left) +(1<<right);
        }
    }

    //向左查找树的深度
    public int countHigh(TreeNode root){
        int count = 0;
        while(root!=null){
            count++;
            root = root.left;
        }
        return count;
    }
```

