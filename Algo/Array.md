# Array

[TOC]

高频面试题

![](e://pic/7430894_1569830591035_8361FA23DFD51FBD74ED3E8B998C124D.png)

## 283.移动零

> 给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的相对顺序。
>
> **示例:**
>
> ```
> 输入: [0,1,0,3,12]
> 输出: [1,3,12,0,0]
> ```
>
> **说明**:
>
> 1. 必须在原数组上操作，不能拷贝额外的数组。
> 2. 尽量减少操作次数。

### -1.空间最优，操作局部优化（双指针）

```java
//主要思路  loop 2 次  第一次将所有非0元素移动到应该在的地方。
//第二次将后面的全部设置为0 
public void moveZeroes(int[] nums) {
        if (nums.length == 0){
            return;
        }
        int modifyIndex = 0;
        for(int i = 0;i<nums.length;i++){
            if(nums[i]!=0){
                nums[modifyIndex++] = nums[i];
            }
        }

        for(;modifyIndex<nums.length;modifyIndex++){
            nums[modifyIndex] = 0;
        }
    }
```

时间复杂度为O(n)

### 2.一次loop

```java
   /***
   * 思路:快排的思想找到一个中间点的数 比如 -21 -23 4 5 0 找到一个midNum数 作为标尺 来计算
     * 因此 在本题中找出所有0 用0作为一个标尺 0左边的都是非0 右边是0  
     * 通过一个loop就可以了。
     * @param nums
     */
    public void moveZeroes(int[] nums) {
        if (nums.length == 0){
            return;
        }

        int j = 0;
        for (int i = 0; i < nums.length ; i++) {
            if (nums[i] != 0){
                int temp = nums[i];
                nums[i] = nums[j];
                nums[j++] = temp;
            }
        }
    }
```

时间复杂度 O(n)

### 3.滚雪球

```java
/***
     * 滚雪球
     *  一次loop 记录0的元素，如果不为0 与前面的元素进行交换  直到最后。
     * @param nums
     */
    public void moveZeroes(int[] nums) {
        if (nums.length == 0){
            return;
        }

        int ballSize = 0;
        for (int i = 0; i < nums.length; i++) {
            if (nums[i] == 0){
                ballSize++;
            }else if (ballSize>0){
                nums[i-ballSize] = nums[i];
                nums[i] = 0;
            }
        }
    }
```

时间复杂度:O(n)

## 70.爬楼梯

> 假设你正在爬楼梯。需要 n 阶你才能到达楼顶。
>
> 每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？
>
> 注意：给定 n 是一个正整数。
>
> 示例 1：
>
> 输入： 2
> 输出： 2
> 解释： 有两种方法可以爬到楼顶。
>
> 1. 1 阶 + 1 阶
> 2. 2 阶

### 1.暴力破解

思路:我们可以将问题简单化，一个2个台阶可以分成是走2步和走1步结果的和。依次调用这个函数。进行细化求解。

```java
public static int climbStairs(int n) {
        if ( n == 0){
            return 0;
        }

        if ( n== 1){
            return 1;
        }
        return climbStairs(n-1)+climbStairs(n-2);
    }
```

时间复杂度：O(2^n)  由于时间过程 提交失败 

空间复杂度：O(n)

### -2.动态规划

不难发现，这个问题可以被分解为一些包含最优子结构的子问题，即它的最优解可以从其子问题的最优解来有效地构建，我们可以使用动态规划来解决这一问题。

dp[i]=dp[i−1]+dp[i−2]

```java
 public int climbStairs(int n) {
       if(n == 1){
           return 1;
       }

       int [] num = new int [n+1];

       num[0] = 1;
       num[1] = 1;

       for(int i=2;i<num.length;i++){
           num[i] = num[i-1]+num[i-2];
       }
       return num[n];
    }
```

## 11.container-with-most-water

> 给你 n 个非负整数 a1，a2，...，an，每个数代表坐标中的一个点 (i, ai) 。在坐标内画 n 条垂直线，垂直线 i 的两个端点分别为 (i, ai) 和 (i, 0)。找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。
>
> 说明：你不能倾斜容器，且 n 的值至少为 2。
>
> ```
> 输入：[1,8,6,2,5,4,8,3,7]
> 输出：49
> ```

### 1.枚举

思路:枚举 类似冒泡一样 遍历每个元素与另一个元素的大小 **下标值为长   数组的值为宽**

时间复杂度:O(n^2)  

```java
public int maxArea(int[] height) {
        int maxArea = 0;
        for (int i = 0; i < height.length-1 ; i++) {
            for (int j = i+1; j < height.length; j++) {
                int area = (j-i)*Math.min(height[i],height[j]);//长*宽
                maxArea = Math.max(maxArea,area);
            }
        }
        return maxArea;
    }
```

### -2.左右夹逼

思路:通过依次loop i从最前面开始 向右移动 j从最后面向前移动 每次比较 如果较小，继续寻找更大的。

时间复杂度:O(n)

```
public int maxArea(int[] height) {
        int maxArea = 0;
        for (int i = 0,j = height.length-1; i < j ; ) {
            //计算出最小的高
            int minHeight = height[i] <height[j] ? height[i++] : height[j--];
            int area = (j-i+1)*minHeight;
            maxArea = Math.max(area,maxArea);
        }
        return maxArea;
    }
```

## 27.[移除元素](https://leetcode-cn.com/problems/remove-element/)

> ```
> 给定一个数组 nums 和一个值 val，你需要原地移除所有数值等于 val 的元素，返回移除后数组的新长度。
> * 不要使用额外的数组空间，你必须在原地修改输入数组并在使用 O(1) 额外空间的条件下完成。
> * 元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。
> * 示例 1:
> * 给定 nums = [3,2,2,3], val = 3,
> * 函数应该返回新的长度 2, 并且 nums 中的前两个元素均为 2。
> * 你不需要考虑数组中超出新长度后面的元素。
> ```

### 1.拷贝覆盖

主要思路:定义一个变量，对不重复的值计数。for循环完成。

存在的问题 就是值可以计算出当前不相同值，数组的值是不符合要求的。

**这种思路在移除元素较多时更适合使用，最极端的情况是全部元素都需要移除，遍历一遍结束即可**

**时间复杂度:O(n) 空间复杂度O(1)**

```java
public static int removeElement3(int [] nums,int val){
        int count = 0;
        for (int num:nums) {
            if (num!=val){
                nums[count++] = num;
            }
        }
        return count;
    }
```

### 2.交换移除

主要思路:用一个变量去记录数组的长度，然后处理两种情况，一种是当前的值等于val 就将数组中最后的值赋值给当前相同的值，m 用来不断减少数组的长度。否则的话 是另一种情况 如果不相等 直接i++ 判断下一个就可以。

**这种思路在移除元素较少时更适合使用，最极端的情况是没有元素需要移除，遍历一遍结束即可**

时间复杂度：O(n)，空间复杂度：O(1)

```java
public static int removeElement4(int [] nums,int val){
        int m = nums.length;
        for (int i = 0; i < m; ) {//注意for 这里不能i++ i是用来确认哪些数据不是要寻找的
            if (nums[i] == val){
                nums[i] = nums[m-1];
                m--;
            }else {
                i++;
            }
        }
        return m;
    }
```

### 3.双指针

实现思路:一个快指针 一个慢指针。用快指针来判断是否相等 不相等赋值给慢指针，慢指针的作用是来定位前面如果有重复的直接跳过去。

时间复杂度:o(n)

空间复杂度:o(1)

```java
public static int removeElement5(int [] nums,int val){
        int i = 0;
        for (int j = 0; j < nums.length; j++) {
            if (nums[j]!=val){
                nums[i] = nums[j];
                i++;//加1
            }
            //如果相等 不做 等待下标i来进行覆盖
        }
        return i;
    }
```

### 4.双指针-要删除的元素很少时

实现思路:通过快慢指针 在一种比较极端的情况下。nums[1,2,3,5,4] val = 4 在最后才可以找到需要删除的元素，虽然nums中的值没有改变，但是通过m-- 停止了循环 i=4。

当我们遇到 nums[i] = valnums[i]=val 时，我们可以将当前元素与最后一个元素进行交换，并释放最后一个元素。这实际上使数组的大小减少了 1。

时间复杂度:O(n)

空间复杂度:O(1)

```java
public static int removeElement6(int [] nums,int val){
        int i = 0;
        int m = nums.length;
        while (i<m){
            if (nums[i] == val){//相当的话 直接将快指针元素赋值给慢指针
                nums[i] = nums[m-1];
                m--;
            }else {
                i++;//不相等 直接跳过
            }
        }
        return i++;
    }
```

## 35.搜索插入位置

> 给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。
>
> 你可以假设数组中无重复元素。
>
> 示例 1:
>
> 输入: [1,3,5,6], 5
> 输出: 2

### 1.二分查找

实现思路: 使用二分查找就可以了。如果使用for循环的话，时间复杂度为O(n) 但是使用二分查找 时间复杂度就下降了很多，因此 注意边界的控制。

1.当target与查询的mid值相等 返回

2.当mid的值大于target max = mid-1;

3.当mid的值小于target min = mid+1

时间复杂度:o(logn)

注意边界的控制。

```java
public int searchInsert(int[] nums, int target) {
        int min = 0;
        int max = nums.length-1;
        int mid;
        int index = 0;
        while (min<=max){
            //如果找到
            mid = (min+max)/2;
            if (nums[mid] == target){
                return mid;
            }else if (nums[mid]<target){
                min = mid+1;
            }else {
                max = mid-1;
            }
        }
        return min;
    }
```

## 1.两数之和

> 给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。
>
> 你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。
>
> 示例:
>
> 给定 nums = [2, 7, 11, 15], target = 9
>
> 因为 nums[0] + nums[1] = 2 + 7 = 9
> 所以返回 [0, 1]

### 1.暴力破解

thinking:通过两次loop 就可以找到，但是时间复杂度为O(n^2)

```java
public int[] twoSum(int[] nums, int target) {
        for (int i = 0; i < nums.length; i++) {
            for (int j = i+1; j < nums.length; j++) {
                if (target-nums[i] == nums[j]){
                    return new int []{i,j};
                }
            }
        }
        throw  new IllegalArgumentException("no find two sum !");
    }
```

### -2.哈希表

thiking：将数组中元素值作为key 存储到hashmap中 然后取寻找。遍历就可以了。

时间复杂度:O(n)

```java
 public int[] twoSum(int[] nums, int target) {
        Map<Integer,Integer> hashMap = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            hashMap.put(nums[i],i);//将值作为key 可以保证数据不会出现重复
        }

        for (int i = 0; i < nums.length ; i++) {
            int num = target - nums[i];
            if (hashMap.containsKey(num) && hashMap.get(num)!=i){
                return new int []{i,hashMap.get(num)};
            }
        }
        throw  new IllegalArgumentException("no find!");
    }
```

## 15.三数之和

> 给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有满足条件且不重复的三元组。
>
> 注意：答案中不可以包含重复的三元组。
>
> 示例：
>
> 给定数组 nums = [-1, 0, 1, 2, -1, -4]，
>
> 满足要求的三元组集合为：
> [
>   [-1, 0, 1],
>   [-1, -1, 2]
> ]
>

### 1.暴力求解

thinking:通过三层loop  这里如果使用ArrryList 是有序 可重复，不能避免元素重复问题。但是使用

LikedHashSet就可以**避免元素重复问题**。

时间复杂度:O(n^3)

```java
public List<List<Integer>> threeSum(int[] nums) {
       if(nums == null || nums.length <=2){
           return Collections.emptyList();
       }

       Arrays.sort(nums);

       Set<List<Integer>> result = new LinkedHashSet<>();
       for(int i=0;i<nums.length;i++){
           for(int j=i+1;j<nums.length;j++){
               for(int k = j+1;k<nums.length;k++){
                   if(nums[i]+nums[j]+nums[k] == 0){
                       List<Integer> list = Arrays.asList(nums[i],nums[j],nums[k]);
                       result.add(list);
                   }
               }
           }
       }
       return new ArrayList(result);
}
```

### 2.哈希表

```java
/**
    * hash slow
    * 1406 ms	46 MB
    *
    * @param nums
    * @return
    */
private List<List<Integer>> hashSolution(int[] nums) {
    if (nums == null || nums.length <= 2) {
        return Collections.emptyList();
    }
    Set<List<Integer>> result = new LinkedHashSet<>();

    for (int i = 0; i < nums.length - 2; i++) {
        int target = -nums[i];
        Map<Integer, Integer> hashMap = new HashMap<>(nums.length - i);
        for (int j = i + 1; j < nums.length; j++) {
            int v = target - nums[j];
            Integer exist = hashMap.get(v);
            if (exist != null) {
                List<Integer> list = Arrays.asList(nums[i], exist, nums[j]);
                list.sort(Comparator.naturalOrder());
                result.add(list);
            } else {
                hashMap.put(nums[j], nums[j]);
            }
        }
    }

    return new ArrayList<>(result);
}

```

### 3.夹逼法

thingking:左右夹逼，三数求和 a+b+c = 0  等价于 a+b=c   将数组进行排序，将c固定 a设置到c的下一个问题，也就是head  b设置到最后一个位置 tail位置。  

第一次遍历c固定 head 和tail 分别向中间移动，判断如果-sum < target 说明值大 因为是负数。所以tail 左移动，否则就是head右移动。如此一轮后 如果找不到，接着c++ head 和tail接着判断。

就可以找到。时间复杂度相对于上面两种是比较低的。

时间复杂度:O(n)

```java
public List<List<Integer>> threeSum(int[] nums) {
        if (nums.length == 0 || nums.length <=2){
            return Collections.emptyList();
        }

        //元素去重
        Set<List<Integer>> result = new LinkedHashSet<>();

        //排序
        Arrays.sort(nums);

        for (int i = 0; i < nums.length-2; i++) {
            int head = i+1;//左端
            int tail = nums.length-1;//右端
            while (head<tail){
                int sum = -(nums[head]+nums[tail]);//取负值
                if (sum == nums[i]){
                    List<Integer> list = 			Arrays.asList(nums[i],nums[head],nums[tail]);
                    result.add(list);
                }
                if(sum<=nums[i]){
                    tail--;
                }else {
                    head++;
                }
            }
        }
        return new ArrayList<>(result);
    }
```

## 26.删除排序数组中的重复项

> 给定一个排序数组，你需要在 原地 删除重复出现的元素，使得每个元素只出现一次，返回移除后数组的新长度。
>
> 不要使用额外的数组空间，你必须在 原地 修改输入数组 并在使用 O(1) 额外空间的条件下完成。
>
>  
>
> 示例 1:
>
> 给定数组 nums = [1,1,2], 
>
> 函数应该返回新的长度 2, 并且原数组 nums 的前两个元素被修改为 1, 2。 
>
> 你不需要考虑数组中超出新长度后面的元素。

### 1.双指针法

th: 一个快指针 一个慢指针 慢指针记录不重复的个数以及下标的作用，而快指针如果和慢指针中元素中出现相等的情况 不用添加直接跳过。

时间复杂度:O(n)

空间复杂度：只是用到了 临时变量 所以O(1) 

```java
 public int removeDuplicates(int[] nums) {
   if(nums.length == 0 || nums.length <2){
     return 0;
   }
        //双指针法
        int i =0;
        for(int j=1;j<nums.length;j++){
            if(nums[j] != nums[i]){  
                i++;
                nums[i] = nums[j];
            }
        }
        return i+1;//因为i从零 所以实际上不重复的个数为i+1
    }
```

##189.旋转数组

> 给定一个数组，将数组中的元素向右移动 k 个位置，其中 k 是非负数。
>
> 示例 1:
>
> 输入: [1,2,3,4,5,6,7] 和 k = 3
> 输出: [5,6,7,1,2,3,4]
> 解释:
> 向右旋转 1 步: [7,1,2,3,4,5,6]
> 向右旋转 2 步: [6,7,1,2,3,4,5]
> 向右旋转 3 步: [5,6,7,1,2,3,4]
> 示例 2:
>
> 输入: [-1,-100,3,99] 和 k = 2
> 输出: [3,99,-1,-100]
> 解释: 
> 向右旋转 1 步: [99,-1,-100,3]
> 向右旋转 2 步: [3,99,-1,-100]

### 1.暴力求解

th:k次决定了移动的次数。每移动一次就动全部元素，因为元素在变化 所有取最后一个元素就可以了。

time : O(k*n) 移动n个元素  k次

space:额外空间o(1)

```java
 //暴力
    public void rotate(int[] nums, int k) {
        int previous=0,temp = 0;
        for(int i=0;i<k;i++){
            previous = nums[nums.length-1];
            for(int j = 0;j<nums.length;j++){
                temp = nums[j];
                nums[j] = previous;
                previous = temp;
            }
        }
    }
```

### 2.使用额外的数组

创建一个新的数组，把移动的元素的位置固定好，关系表达式为 (k+i)%nums.length  

time:O(n)

space:O(n)

```java
//使用额外的数组
    public void rotate(int[] nums, int k) {
       int [] newArr = new int [nums.length];
       for(int i=0;i<nums.length;i++){
           newArr[(i+k)%nums.length] = nums[i];
       }
       
       for(int i=0;i<nums.length;i++){
           nums[i] = newArr[i];
       }
    }
```

还有两个答案 有经历看看

<https://leetcode-cn.com/problems/rotate-array/solution/xuan-zhuan-shu-zu-by-leetcode/>

## 88. 合并两个有序数组

> 给你两个有序整数数组 nums1 和 nums2，请你将 nums2 合并到 nums1 中，使 num1 成为一个有序数组。
>
>  
>
> 说明:
>
> 初始化 nums1 和 nums2 的元素数量分别为 m 和 n 。
> 你可以假设 nums1 有足够的空间（空间大小大于或等于 m + n）来保存 nums2 中的元素。
>
>
> 示例:
>
> 输入:
> nums1 = [1,2,3,0,0,0], m = 3
> nums2 = [2,5,6],       n = 3
>
> 输出: [1,2,2,3,5,6]

### 1.合并后在排序

时间复杂度:O((m+n)log(m+n))

空间复杂度：O(1)

```java
public void merge(int[] nums1, int m, int[] nums2, int n) {
	// nums2 要拷贝的数组 0 开始下标  nums1 拷贝的位置 m 个位置开始 长度为n
    System.arraycopy(nums2, 0, nums1, m, n);
    Arrays.sort(nums1);
  }
```

### 2.双指针/从前往后

时间复杂度:O(m+n)

空间复杂度:O(m)

```java
//双指针
    public void merge(int[] nums1, int m, int[] nums2, int n) {
         // Make a copy of nums1.
    int [] nums1_copy = new int[m];
    System.arraycopy(nums1, 0, nums1_copy, 0, m);

    // Two get pointers for nums1_copy and nums2.
    int p1 = 0;
    int p2 = 0;

    // Set pointer for nums1
    int p = 0;

    // Compare elements from nums1_copy and nums2
    // and add the smallest one into nums1.
    while ((p1 < m) && (p2 < n))
      nums1[p++] = (nums1_copy[p1] < nums2[p2]) ? nums1_copy[p1++] : nums2[p2++];

    // if there are still elements to add
    if (p1 < m)
      System.arraycopy(nums1_copy, p1, nums1, p1 + p2, m + n - p1 - p2);
    if (p2 < n)
      System.arraycopy(nums2, p2, nums1, p1 + p2, m + n - p1 - p2);

  }
```

### 3.双指针/从后往前

time:O(m+n)

space:O(1)

```java
class Solution {
  public void merge(int[] nums1, int m, int[] nums2, int n) {
    // two get pointers for nums1 and nums2
    int p1 = m - 1;
    int p2 = n - 1;
    // set pointer for nums1
    int p = m + n - 1;

    // while there are still elements to compare
    while ((p1 >= 0) && (p2 >= 0))
      // compare two elements from nums1 and nums2 
      // and add the largest one in nums1 
      nums1[p--] = (nums1[p1] < nums2[p2]) ? nums2[p2--] : nums1[p1--];

    // add missing elements from nums2
    System.arraycopy(nums2, 0, nums1, 0, p2 + 1);
  }
}
```

## 66.加1

> 给定一个由整数组成的非空数组所表示的非负整数，在该数的基础上加一。
>
> 最高位数字存放在数组的首位， 数组中每个元素只存储单个数字。
>
> 你可以假设除了整数 0 之外，这个整数不会以零开头。
>
> 示例 1:
>
> 输入: [1,2,3]
> 输出: [1,2,4]
> 解释: 输入数组表示数字 123。
> 示例 2:
>
> 输入: [4,3,2,1]
> 输出: [4,3,2,2]
> 解释: 输入数组表示数字 4321。

th:

一共三种情况 

​    1.不存在进位  123  -》 124 

​    2.数字长度不变  199  -> 200  

​    3.数字长度加1   99  -》 100

​    而前两种情况就在for循环中就可以解决了

​    第三种 创建一个新数组，长度加1  默认为0  第一个元素赋值0 

time:O(n)

```java
 /*
    一共三种情况 
    1.不存在进位  123  -》 124 
    2.数字长度不变  199  -> 200  
    3.数字长度加1   99  -》 100
    而前两种情况就在for循环中就可以解决了
    第三种 创建一个新数组，长度加1  默认为0  第一个元素赋值0  
    */
    public int[] plusOne(int[] digits) {
        for(int i = digits.length-1;i>=0;i--){
            digits[i]++;
            digits[i] = digits[i] %10;
            if(digits[i]!=0){
                return digits;
            }
        }
        int [] newArr = new int [digits.length+1];
        newArr[0] = 1;
        return newArr;
    }
```

第二种答案 思想和上面的一致

```java
public int[] plusOne(int[] digits) {
        int length = digits.length;
        for (int i = length-1; i >= 0 ; i--) {
            if ((digits[i]++)<9){
                return digits;
            }
            digits[i] = 0;
        }
        digits = new int [length+1];
        digits[0] = 1;
        return digits;
    }
```

