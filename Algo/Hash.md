# Hash

![](e://pic/7430894_1569830476289_917532E4CCA96E15E5ABCCCBCA30C9F8.png)

## 242.有效的字母异位词

> #### [242. 有效的字母异位词](https://leetcode-cn.com/problems/valid-anagram/)
>
> 难度简单173收藏分享切换为英文关注反馈
>
> 给定两个字符串 *s* 和 *t* ，编写一个函数来判断 *t* 是否是 *s* 的字母异位词。
>
> **示例 1:**
>
> ```
> 输入: s = "anagram", t = "nagaram"
> 输出: true
>
> ```
>
> **示例 2:**
>
> ```
> 输入: s = "rat", t = "car"
> 输出: false
> ```

### 1.依次遍历比较

思路:异位次的含义是 一个字符串中字母的顺序改变，但是个数不变 就是异位词。

比较长度 如果长度不相等 返回false 然后将两个字符串进行排序，时间复杂度为O(logN) 

然后进行依次比较 如果不相等直接返回true  不好的一点是 排序使时间复杂度提高了。

time :O(NlogN)

space:O(1)

```java
//time O(NlogN)  space O(n)
    public boolean isAnagram(String s, String t) {
        //排序后比较
        if(s.length() != t.length()){
            return false;
        }

        char [] chs = s.toCharArray();
        char [] cht = t.toCharArray();

        Arrays.sort(chs);
        Arrays.sort(cht);

        return Arrays.equals(chs,cht);
    }
```

### 2.哈希表

思路：如果长度不同 返回false 定义一个数组 26  s.charAt(i)-'a' 存储进数组  对应的下标  t.charAt(i)-'a'  不存储对应的下标。前者++  后者-- 当最后数组为0是异位词，否则不是异位词。

time:O(n)

space:O(1)

```java
public boolean isAnagram(String s, String t) {
        //哈希表
        if(s.length() != t.length()){
            return false;
        }

        int [] alpha = new int [26];
        for(int i=0;i<s.length();i++){
            alpha[s.charAt(i) - 'a']++;
            alpha[t.charAt(i) - 'a']--;
        }

        for(int i=0;i<alpha.length;i++){
            if(alpha[i]!=0){
                return false;
            }
        }
        return true;
    }
```

## 49.字母异位词分组

> #### [49. 字母异位词分组](https://leetcode-cn.com/problems/group-anagrams/)
>
> 难度中等309
>
> 给定一个字符串数组，将字母异位词组合在一起。字母异位词指字母相同，但排列不同的字符串。
>
> **示例:**
>
> ```
> 输入: ["eat", "tea", "tan", "ate", "nat", "bat"],
> 输出:
> [
>   ["ate","eat","tea"],
>   ["nat","tan"],
>   ["bat"]
> ]
> ```



### 1.排序数组分类

时间复杂度:O(NK logK)

空间复杂度:O(NK)

```java
public List<List<String>> groupAnagrams(String[] strs) {
        if(strs.length == 0 || strs == null){
            return null;
        }
        //1.按照排序
        Map<String,List> ans = new HashMap<String,List>();
        for(String str : strs){
            char [] chs = str.toCharArray();
            Arrays.sort(chs);
            String key = String.valueOf(chs);
            if(!ans.containsKey(key))
                ans.put(key,new ArrayList());
            ans.get(key).add(str);
        }

        return new ArrayList(ans.values());
    }
```

### 