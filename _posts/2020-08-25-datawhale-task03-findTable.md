---
layout: post
title: 'DataWhale Task03：查找之 查找表'
subtitle: '查找表刷题笔记'
date: 2020-08-25
author: Dave
tags: Java刷题
---

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/img/datawhale-ps.png)

>记录DataWhale 相对比较随意




**查找表的基本三类结构以及数据结构**

- **第一类:只查找有无-set**
    - 元素a是否存在,通常用set集合
    - se只存储键,而不需要对应其相应的值。
    - se中的键不允许重复
- **第二类:需查找重复次数**
    - 元素a出现了几次的结果返回:**推荐 LinkedList ArrayDeque**

        ```java
        //LinkedList ArrayDeque有此方法
        //ArrayList 无此方法
        removeFirstOccurrence(Object o) //删除第一次出现的指定元素
        removeLastOccurrence(Object o) //删除最后一次出现的指定元素
        ```
    - 元素a出现了几次的次数返回:**推荐 HashMap**
        ```java
        //key存放元素
        //value存放重复次数
        ```
        有时也可以使用**数组**
        ```java
        //index存放元素
        //value存放重复次数
        ```
- **第三类:需要改变映射关系-map**
    - 通过将原有序列的关系映射统表示为其他
    - 比如：字符串的字符按频次升序排列


### [349. 两个数组的交集](https://leetcode-cn.com/problems/intersection-of-two-arrays/)

- 两个set 两次遍历 只记录 有无 不记录 重复几次
```java
class Solution {
    public int[] intersection(int[] nums1, int[] nums2) {
        HashSet<Integer> set1=new HashSet<>();
        HashSet<Integer> ans=new HashSet<Integer>();     
        for(int i:nums1){
            set1.add((Integer)i);
        }
        for(int i:nums2){
            if(set1.contains(i)){
                //todo
                ans.add(i);
            }
        }
        int[] res=new int[ans.size()];
        int index=0;
        for(Integer i:ans){
            res[index++]=i;
        }
        return res;
    }//method end
}//class end
```
### [350. 两个数组的交集 II](https://leetcode-cn.com/problems/intersection-of-two-arrays-ii/)

- 在上一题基础上记录重复次数

     ```java
        //leetcode上大家都喜欢用hashmap，可能是参考官方解法比较多，但是这里用LinkedList/ArrayDeque 明显简洁很多

        //LinkedList ArrayDeque都有此方法
        //注意：ArrayList 无此方法
        removeFirstOccurrence(Object o) //删除第一次出现的指定元素
        removeLastOccurrence(Object o) //删除最后一次出现的指定元素
    ```

```java
class Solution {
    public int[] intersect(int[] nums1, int[] nums2) {
        LinkedList<Integer> arr1=new LinkedList<Integer>();
        ArrayList<Integer> ans=new ArrayList<Integer>();
        for(int i:nums1){
            arr1.add(i);
        }
        for(int i:nums2){
            if(arr1.contains(i)){
                //todo
                ans.add(i);
                arr1.removeFirstOccurrence(i);
            }
        }
        int[] res=new int[ans.size()];
        int index=0;
        for(Integer i:ans){
            res[index++]=i;
        }
        return res;
 
    }//method end
}//class end

```
### [242. 有效的字母异位词](https://leetcode-cn.com/problems/valid-anagram/)

- 两次遍历统计每个字符的个数即可

- 这里我使用**数组**，也可以使用hashmap

    ```java
    //index存放元素
    //value存放重复次数
    ```
```java
class Solution {
    public boolean isAnagram(String s, String t) {
        /*
        说明:你可以假设字符串只包含小写字母。
        a~z ANSII值97~122

        进阶:如果输入字符串包含 unicode 字符怎么办？你能否调整你的解法来应对这种情况？

    判断异位词即判断变换位置后的字符串和原来是否相同，那么不仅需要存储元素，还需要记录元素的个数。
    数组 的index和value 具备ANSII映射特性
        */
        if(s.length() != t.length()) return false;

        int len=s.length();
        boolean res=true;
        int[] sc=new int[123];
        int[] tc=new int[123];
        for(int i=0;i<len;i++){
            sc[s.charAt(i)]++;
            tc[t.charAt(i)]++;
        }

        for(int i=0;i<123;i++){
            if(sc[i]!=tc[i])
            return false;
        }

        return res;
    }
}
```

- 其实，还可以排序 比较是否相同
```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) {
        return false;
    }
    char[] str1 = s.toCharArray();
    char[] str2 = t.toCharArray();
    Arrays.sort(str1);
    Arrays.sort(str2);
    return Arrays.equals(str1, str2);
}
```
### [202. 快乐数](https://leetcode-cn.com/problems/happy-number/)

- 用 HashSet 检测循环
```java
/*
这道题目思路很明显，当n不等于1时就循环，每次循环时，将其最后一位到第一位的数依次平方求和，比较求和是否为1。

难点在于，什么时候跳出循环？

假设循环个100次，还没得出结果就false，但是我们想要的肯定不是如此。

在算无限循环小数时有一个特征，就是当除的数中，和之前历史的得到的数有重合时，这时就是无限循环小数。
那么这里也可以按此判断，因为只需要判断有或无，不需要记录次数，故用set的数据结构。每次对求和的数进行append，当新一次求和的值存在于set中时，就return false.
*/
class Solution {
    public boolean isHappy(int n) {
        int tmp=getNextN(n);
        HashSet<Integer> set=new HashSet<>();
        boolean res=true;
        while(tmp!=1){
            if(set.contains(tmp)){
                res=false;
                break;
            }
            set.add(tmp);
            tmp=getNextN(tmp);

        }
        return res;
        
    }//method end
    public int getNextN(int n) {
        int tmp=n;
        int len=0;
        while(tmp!=0){
            tmp=tmp/10;
            len++;
        }

        int[] nn=new int[len];
        for(int i=len-1;i>=0;i--){
            nn[i]=n%10;
            n=(n/10);
        }
        int sum=0;
        for(int i:nn){
            sum+=i*i;
        }
        return sum;
        
    }//method end

}//class end
```

- 不断调用getNextN的过程 一直循环形成隐式带环链表
    链表成环 怎么搞 快慢指针大法
    
```java
class Solution {

     public int getNext(int n) {
        int totalSum = 0;
        while (n > 0) {
            int d = n % 10;
            n = n / 10;
            totalSum += d * d;
        }
        return totalSum;
    }

    public boolean isHappy(int n) {
        int slowRunner = n;
        int fastRunner = getNext(n);
        while (fastRunner != 1 && slowRunner != fastRunner) {
            slowRunner = getNext(slowRunner);
            fastRunner = getNext(getNext(fastRunner));
        }
        return fastRunner == 1;
    }
}
```
### [290. 单词规律](https://leetcode-cn.com/problems/word-pattern/)

- hashmap

**添加新的 key 的时候判断一下相应的 value 是否已经用过了**
- [详细通俗的思路分析，多解法](https://leetcode-cn.com/problems/word-pattern/solution/xiang-xi-tong-su-de-si-lu-fen-xi-duo-jie-fa-by--53/)

```java
class Solution {
    public boolean wordPattern(String pattern, String str) {
    HashMap<Character, String> map = new HashMap<>();
    HashSet<String> set = new HashSet<>();
    String[] array = str.split(" ");
    if (pattern.length() != array.length) {
        return false;
    }
    for (int i = 0; i < pattern.length(); i++) {
        char key = pattern.charAt(i);
        if (map.containsKey(key)) {
            if (!map.get(key).equals(array[i])) {
                return false;
            }
        } else {
            // 判断 value 中是否存在
            if (set.contains(array[i])) {
                return false;
            }
            map.put(key, array[i]);
            set.add(array[i]);
        }
    }
    return true;
}

}
```

```java
class Solution {
    public boolean wordPattern(String pattern, String str) {
    String[] words = str.split(" ");
    if (words.length != pattern.length())
        return false;
    Map index = new HashMap();
    for (Integer i = 0; i < words.length; ++i)
        if (index.put(pattern.charAt(i), i) != index.put(words[i], i))
            return false;
    return true;
    }
}
```
### [205. 同构字符串](https://leetcode-cn.com/problems/isomorphic-strings/)

- 通配规则翻译

第一次出现某个字符的索引 来标记该字符 形成字符结构 

距离：egg    变为 011 或者 abb

遗憾的是 容器没有直接的方法 可以给到 直接求出第一次出现某元素时的索引

```java
class Solution {
    public boolean isIsomorphic(String s, String t) {
        return isIsomorphicHelper(s).equals(isIsomorphicHelper(t));
    }

    private String isIsomorphicHelper(String s) {
        int[] map = new int[128];
        int n = s.length();
        StringBuilder sb = new StringBuilder();
        int count = 1;
        for (int i = 0; i < n; i++) {
            char c = s.charAt(i);
            //当前字母第一次出现,赋值
            if (map[c] == 0) {
                map[c] = count;
                count++;
            }
            sb.append(map[c]);
        }
        return sb.toString();
}

}
```

- map映射 + 双向验证

```java
/*

我们可以利用一个 map 来处理映射。对于 s 到 t 的映射，我们同时遍历 s 和 t ，假设当前遇到的字母分别是 c1 和 c2 。

如果 map[c1] 不存在，那么就将 c1 映射到 c2 ，即 map[c1] = c2。

如果 map[c1] 存在，那么就判断 map[c1] 是否等于 c2，也就是验证之前的映射和当前的字母是否相同。


对于这里，我们需要验证 s - > t 和 t -> s 两个方向。如果验证一个方向，是不可以的。

举个例子，s = ab, t = cc，如果单看 s -> t ，那么 a -> c, b -> c 是没有问题的。

必须再验证 t -> s，此时，c -> a, c -> b，一个字母对应了多个字母，所以不是同构的。

代码的话，只需要调用两次上边的代码即可。

*/
public boolean isIsomorphic(String s, String t) {
    return isIsomorphicHelper(s, t) && isIsomorphicHelper(t, s);

}

private boolean isIsomorphicHelper(String s, String t) {
    int n = s.length();
    HashMap<Character, Character> map = new HashMap<>();
    for (int i = 0; i < n; i++) {
        char c1 = s.charAt(i);
        char c2 = t.charAt(i);
        if (map.containsKey(c1)) {
            if (map.get(c1) != c2) {
                return false;
            }
        } else {
            map.put(c1, c2);
        }
    }
    return true;
}
```
### [451. 根据字符出现频率排序](https://leetcode-cn.com/problems/sort-characters-by-frequency/)

- hashmap

对于相同频次的字母，顺序任意，需要考虑大小写，返回的是字符串。
统计每个字符频率，按照频率倒叙对value进行输出

```java
/*
通过 int 数组记录 每个字符出现的次数
通过 HashMap 将出现次数相同的字符进行拼接
将 出现次数 排序, 倒序取出所有字符进行拼接
*/
class Solution {
    public String frequencySort(String s) {
        HashMap<Integer, String> map = new HashMap<>();

        int[] freq = new int[256];
        for (char c : s.toCharArray()) {
            freq[c]++;
        }

        for (int i = 0; i < freq.length; i++) {
            if (freq[i] != 0) {
                char ch = (char) i;

                String str = map.get(freq[i]);
                // 字符出现次数相同, 进行拼接
                if (str != null) {
                    String strNew = str.concat(build(ch, freq[i]));
                    map.put(freq[i], strNew);
                }else {
                    map.put(freq[i], build(ch, freq[i]));
                }
            }
        }

        Integer[] keys = map.keySet().toArray(new Integer[]{});
        Arrays.sort(keys);
        StringBuilder sbl = new StringBuilder();
        for (int i = keys.length - 1; i >= 0; i--) {
            sbl.append(map.get(keys[i]));
        }

        return sbl.toString();
    }

    private String build(char ch, int times) {
        String string = Character.toString(ch);
        StringBuilder res = new StringBuilder(string);
        int t = 1;
        while (t < times) {
            res.append(string);
            t++;
        }

        return res.toString();
    }
}
```

- 大顶堆 借助于compartor


```java
/*
用哈希表存储每个字符的出现次数，再通过一个大顶堆（根据出现次数排序），不断取出堆顶元素，使用StringBuilder不断append即可。
*/
class Solution {
    public String frequencySort(String s) {
        int[] letters = new int[128];
        for (char c : s.toCharArray()) {
            letters[c]++;
        }
        PriorityQueue<Character> heap = new PriorityQueue<>(128, (a, b) -> Integer.compare(letters[b], letters[a]));
        StringBuilder res = new StringBuilder();

        for (int i = 0; i < letters.length; ++i) {
            if (letters[i] != 0) heap.offer((char)i);
        }

        while (!heap.isEmpty()) {
            char c = heap.poll();
            while (letters[c]-- > 0)
                res.append(c);
        }
        return res.toString();
    }
}

```

**车轮滚滚 时间**

今天用错方法了  length，length()，size()

- 获取数组的长度使用length属性；

- 获取字符串的长度使用length()方法；

- 获取集合中元素个数使用size()方法。

时间长了，有时可能会有些迟疑。
```java
    int[] array = new int[10];
	String str = "hello dave";
	List<String> list = new ArrayList<String>();
		
	System.out.println("数组的length属性：" + array.length);
	System.out.println("字符串的length()方法：" + str.length());
	System.out.println("集合的size()方法：" + list.size());
```

- [通过Value获取Map中的键值Key的四种方法](https://www.cnblogs.com/522-1997/p/11788201.html)
- [https://leetcode.wang/](https://leetcode.wang/)
