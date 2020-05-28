# LinkedList

高频题

![](e://pic/7430894_1569830509086_8E72661D2242C40ECD146E2DB6D88051.png)



## [53. 最大子序和](https://leetcode-cn.com/problems/maximum-subarray/)？？

> 给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。
>
> 示例:
>
> 输入: [-2,1,-3,4,-1,2,1,-5,4],
> 输出: 6
> 解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。
> 进阶:
>
> 如果你已经实现复杂度为 O(n) 的解法，尝试使用更为精妙的分治法求解。
>
> 



```java
public int crossSum(int[] nums, int left, int right, int p) {
        if (left == right) return nums[left];

        int leftSubsum = Integer.MIN_VALUE;
        int currSum = 0;
        for (int i = p; i > left - 1; --i) {
            currSum += nums[i];
            leftSubsum = Math.max(leftSubsum, currSum);
        }

        int rightSubsum = Integer.MIN_VALUE;
        currSum = 0;
        for (int i = p + 1; i < right + 1; ++i) {
            currSum += nums[i];
            rightSubsum = Math.max(rightSubsum, currSum);
        }

        return leftSubsum + rightSubsum;
    }

    public int helper(int[] nums, int left, int right) {
        if (left == right) return nums[left];

        int p = (left + right) / 2;

        int leftSum = helper(nums, left, p);
        int rightSum = helper(nums, p + 1, right);
        int crossSum = crossSum(nums, left, right, p);

        return Math.max(Math.max(leftSum, rightSum), crossSum);
    }

    public int maxSubArray(int[] nums) {
        return helper(nums, 0, nums.length - 1);
    }
```



## 206.反转链表

> 反转一个单链表。
>
> 示例:
>
> 输入: 1->2->3->4->5->NULL
> 输出: 5->4->3->2->1->NULL
> 进阶:
> 你可以迭代或递归地反转链表。你能否用两种方法解决这道题？
>

```java
//实现思路 
//在遍历列表时，将当前节点的next指针改为指向前一个元素，由于节点没有引用其上一个节点，因此必须实现存储前一个元素，在更改引用之前，还需要另一个指针来存储一个节点，不要忘记在最后返回新的头引用
public ListNode reverseList(ListNode head) {
    ListNode prev = null;//设置一个null结点
    ListNode cur = head;
    while (cur!=null){
        ListNode next = cur.next; //记录ListNode节点
        cur.next = prev;//将cur.next 设置为null
        prev = cur; //当前节点赋值成prev
        cur = next; //当前节点设置成cur
    }
    return prev;
}
```

- 时间复杂度：O(n)O(n)，假设 n是列表的长度，时间复杂度是 O(n)O(n)。
- 空间复杂度：O(1)O(1)。


## 141.环形链表

> 给定一个链表，判断链表中是否有环。
>
> 为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。
>
> 示例 1：
>
> 输入：head = [3,2,0,-4], pos = 1
> 输出：true
> 解释：链表中有一个环，其尾部连接到第二个节点。
>

![](e:///pic/lc-快慢指针.png)

### 1.快慢指针	

可以类比成两个运动员 在同一个环形运动场跑步，一个跑的快的 和 一个跑的慢的 在一定的时间内 快的一定会和慢的相遇，这个时候 就说明 有环存在。

```java
//检查是否链表中有环
    //思路:定义一个快指针 和 一个慢指针 慢指针一次走一步 快指针一次走两步
    //如果有环 一定会相遇。多次循环
    public boolean hasCycle(ListNode head) {
        if (head == null){
            return false;
        }
        ListNode fastNode = head.next;
        ListNode slowNode = head;
        while (fastNode!=null && fastNode.next!=null){
            fastNode = fastNode.next.next;
            slowNode = slowNode.next;
            if (slowNode == fastNode)
                return true;
        }
        return false;
    }

```

分析:

时间复杂度:o(n)

1.第一种情况  链表不存在环 快慢指针分别到达尾部，其时间取决于列表的长度。O(n)

2.第二种情况  链表中存在环  在最糟糕的情形下，时间复杂度为 O(N+K)O(N+K)，也就是 O(n)

空间复杂度 o(1)

只是用了快慢指针两个结点。

### 2.哈希表

思路: 通过创建哈希表，遍历链表 将每次遍历的都存储到哈希表中 如果哈希表中有当前哈希值 说明 遍历到了环形处，说明是环形链表 否则的话 遍历完  说明不是环形链表

```java
public boolean hasCycle2(ListNode head) {
        Set nodeSet = new HashSet();
        while (head!=null){
            if (nodeSet.contains(head)){
                return true;
            }
            nodeSet.add(head);
            head = head.next;
        }
        return false;
    }
```

时间复杂度：o(n) 对于含有n个元素的链表 

空间复杂度:   o(n) 最坏情况下 全部添加 所有取决于哈希表中的元素数目

## 23.合并K个排序链表

> 合并 k 个排序链表，返回合并后的排序链表。请分析和描述算法的复杂度。
>
> 示例:
>
> 输入:
> [
>   1->4->5,
>   1->3->4,
>   2->6
> ]
> 输出: 1->1->2->3->4->4->5->6

### 1.不使用递归

基本思路：我们可以使用分治思想 来解决这个问题。当一个链表集合中需要合并并排序。我们可以假设只有2个 每次合并两个。两个合并一个。这样就可以获取到最终的结果。

第一次将list[0]与list[len-1]合并 2次list[1]与list[len-2] 经过 for循环一次后。可以等到一半的list链表结合。在依次len = len/2; 依次合并 就可以获取到下标为0的为头结点的这个链表。

```java
//合并两个有序的链表
    public ListNode mergeKLists(ListNode[] lists) {
        int len = lists.length;
        if (len == 0)
            return null;
        while (len > 1) {
            for (int i = 0; i < len / 2; i++) {
                lists[i] = merageKList(lists[i],lists[len-1-i]);
            }
            len = (len+1)/2;
        }
        return lists[0];
    }

    //合并两个有序的链表 
    private ListNode merageKList(ListNode l1, ListNode l2) {
        if (l1 == null)
            return null;
        if (l2 == null)
            return null;

        ListNode listNode = new ListNode(-1);
        ListNode head = listNode;

        while (l1 != null && l2 != null) {
            if (l1.data < l2.data) {
                head.next = l1;
                l1 = l1.next;
            } else {
                head.next = l2;
                l2 = l2.next;
            }
            head = head.next;
        }

        if (l1 != null){
            head.next = l1;
        }else{
            head.next = l2;
        }
        return listNode.next; //note 注意返回的不是head 返回head 会带有 -1这个结点的值 只需要后边的值 牛掰思想
    }
```

时间复杂度：整体时间复杂度为O(N*log(k)), k为链表个数，N为链表平均长度。

## 24.两两交换链表中的节点

```
//给定一个链表，两两交换其中相邻的节点，并返回交换后的链表。 
//
// 你不能只是单纯的改变节点内部的值，而是需要实际的进行节点交换。 
//
// 
//
// 示例: 
//
// 给定 1->2->3->4, 你应该返回 2->1->4->3.
// 
// Related Topics 链表
```

### 1.递归

thinking: 使用递归的方法  每一次递归都交换一对节点，用firstNode.next 记录上一次节点交换后的首节点。secondNode节点作为首节点。

```java
public ListNode swapPairs(ListNode head) {
        //if the list has no node or has only one node left
        if(head == null || head.next == null){
            return head;
        }

        //Nodes to be swapped
        ListNode firstNode = head;
        ListNode secondNode = head.next;

        //swapping
        firstNode.next = swapPairs(secondNode.next);
        secondNode.next = firstNode;

        //Now the head is the second node
        return secondNode;
    }
```

- 时间复杂度：O(N)    O(N)，其中 N 指的是链表的节点数量。
- 空间复杂度：O(N)    O(N)，递归过程使用的堆栈空间。

#### -递归简洁版

```java
 public ListNode swapPairs(ListNode head) {
        if ((head == null)||(head.next == null))
            return head;
        ListNode n = head.next;
        head.next = swapPairs(head.next.next);
        n.next = head;
        return n;
    }
```

### 2.迭代法

thinking：定义一个前驱节点，记录每次交换后的节点变化

head节点和pre节点每次交换完成后重置。

```java
/***
     *  迭代法
     *  1.定义一个前驱节点，用以记录每次交换后的元素的前置节点
     *  2.交换
     *  3.head 节点 和pre节点重置
     * @param head
     * @return
     */
    public ListNode swapPairs(ListNode head) {

        ListNode dummy = new ListNode(-1);
        dummy.next = head;

        ListNode pre = dummy;

        while ((head!=null) && (head.next !=null)){

            ListNode firstNode = head;
            ListNode secondNode = head.next;

            //swap
            pre.next = secondNode;//-1 -> 2
            firstNode.next = secondNode.next;// 1 -> 3
            secondNode.next = firstNode;

            pre = firstNode;
            head = firstNode.next;
        }
        return dummy.next;
    }
```

- 时间复杂度：O(N)，其中 N 指的是链表的节点数量。
- 空间复杂度：O(1)

## 25.K 个一组翻转链表

> 难度困难416
>
> 给你一个链表，每 *k *个节点一组进行翻转，请你返回翻转后的链表。
>
> *k *是一个正整数，它的值小于或等于链表的长度。
>
> 如果节点总数不是 *k *的整数倍，那么请将最后剩余的节点保持原有顺序。
>
>  
>
> **示例：**
>
> 给你这个链表：`1->2->3->4->5`
>
> 当 *k *= 2 时，应当返回: `2->1->4->3->5`
>
> 当 *k *= 3 时，应当返回: `3->2->1->4->5`
>
>  
>
> **说明：**
>
> - 你的算法只能使用常数的额外空间。
> - **你不能只是单纯的改变节点内部的值**，而是需要实际进行节点交换。



```java
/***
     * 迭代法
     * 1.链表分区已翻转 + 待翻转 + 未翻转部分
     * 2.翻转前确定翻转的范围 通过k来决定
     * 3.记录链表前驱和后继 方便翻转完成后，把已翻转和未翻转连接起来。
     * 4.初始化两个边路pre  end  pre >代表待翻转链表的前驱，end代表待翻转的末尾。
     * 5.经过k次 end到达链表末尾。  记录待翻转链表的后继 next = end.next
     * 6.翻转链表，将三部分连接起来，然后重置pre 和 end 指针 进入下一个循环
     * 7.特殊情况 翻转部分长度不足k时，在定位end 完成后，end = null 已经到达末尾。
     * 8.时间复杂度为 O(n*K) 最好的情况为 O(n) 最差的情况未 O(n^2)
     * 9.空间复杂度为 O(1)
     * @param head
     * @param k
     * @return
     */
    public ListNode reverseKGroup(ListNode head, int k) {
        ListNode dumy = new ListNode(0);
        dumy.next = head;

        ListNode pre = dumy;
        ListNode end = dumy;

        while (end.next != null) {
            //确定待翻转的范围
            for (int i = 0; i < k && end != null; i++) {
                end = end.next;
            }

            if (end == null) {
                break;
            }

            ListNode start = pre.next;//待翻转链表的开始位置
            ListNode next = end.next;//下一个待翻转链表的起始位置

            end.next = null;//和后继待翻转链表断开

            pre.next = reverse(start);
            start.next = next;
            pre = start;

            end = pre;
        }
        return dumy.next;
    }

    public static ListNode reverse(ListNode head) {
        ListNode pre = null;
        ListNode curr = head;
        //假设 1 -> 2 -> 3
        while (curr != null) {
            ListNode next = curr.next; // next = 2   // next = 3
            curr.next = pre;// 1.next = null        // 3.next = 1
            pre = curr;// pre = 1;                 //  1 = 3
            curr = next; // curr = 2              // 3  = null
        }
        return pre; //
    }
```

## 21. 合并两个有序链表

> 将两个升序链表合并为一个新的升序链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 
>
> **示例：**
>
> ```
> 输入：1->2->4, 1->3->4
> 输出：1->1->2->3->4->4
> ```
>
> 

### 1.迭代

th:主要是通过定义一个head节点 遍历两个链表就可以，最后一定会有一个链表没有遍历完 使用三目运算符进行算

**需要注意的点**  因为head节点一直next next下去 所以应该定义一个root节点 作为根节点 head节点 一直next下去。返回的是root节点

time:O(m+n)

space:O(1) 

```java
/*
    *思路 主要是通过定义一个head节点 遍历两个链表就可以，最后一定会有一个链表没有遍历完 使用三目运算符进行算
    notice 因为head节点一直next next下去 所以应该定义一个root节点 作为根节点 head节点 一直next下去。返回的是root节点
    */
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if(l1 == null || l2 == null){
            return null;
        }

        ListNode root = new ListNode(-1);

        ListNode head = root;

        while(l1!=null && l2!= null){
            if(l1.val <= l2.val){
                head.next = l1;
                l1 = l1.next; 
            }else{
                head.next = l2;
                l2 = l2.next;
            }
            head = head.next;
        }

        head.next = (l1 == null ? l2 : l1);

        return root.next;
    }
```

### 2.递归

th：其实如果一个方法可以迭代就可以递归解决，但是递归代码虽然简洁 理解上比较难一些，递归代码无非就是 三点需要注意：1 终止条件 2. 业务逻辑 3.解决下一个子问题 也就是 调用自身。

time:O(m+n)

space:调用 mergeTwoLists 退出时 l1 和 l2 中每个元素都一定已经被遍历过了，所以 n + mn+m 个栈帧会消耗 O(n + m)O(n+m) 的空间。

```java
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
       if(l1 == null ){
           return l2;
       }else if(l2 ==  null){
           return l1;
       }else if(l1.val < l2.val){
           l1.next = mergeTwoLists(l1.next,l2);
           return l1;
       }else {
           l2.next = mergeTwoLists(l1,l2.next);
           return l2;
       }
    }
```

