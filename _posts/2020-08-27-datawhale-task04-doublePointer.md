---
layout: post
title: 'DataWhale Task04：查找之 双指针'
subtitle: '双指针刷题笔记'
date: 2020-08-27
author: Dave
tags: Java刷题
---

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/img/datawhale-ps.png)

>记录DataWhale 相对比较随意




**查找表 双指针**

- **第一类:前后指针**
    - 

查找表问题，大多是通过构建一个查找表，而避免了在查找中再内层嵌套循环，从而降低了时间
复杂度。那么在这一部分，主要是利用双指针来优化时间复杂度，虽然好几个地方还是在用hashmap。

    


### [1. 两数之和](https://leetcode-cn.com/problems/two-sum/)

- 第一个先上了个笨办法 时间复杂度 很不好 显然不是我们想要的

    时间复杂度：两层 for 循环，O（n²）

    空间复杂度：O（1）

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        //假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍
        int[] res=new int[2];
       
        for(int i=0;i<nums.length-1;i++){
            for(int j=nums.length-1;i<j;j--){
                if(target==nums[i]+nums[j]){
                    res[0]=i;
                    res[1]=j;
                    break;
                }
            }//forj end     
        }//fori end
        return res;
    }//method end
}//class end
```

- hash table 优化

    把数组的每个元素保存为 hash 的 key，下标保存为 hash 的 value 。

    这样只需判断 sub=target-nums[i] 在不在 hash 的 key 里就可以了，而此时的时间复杂度仅为 O（1） 缩减了内循环的时间复杂度

    时间复杂度：比解法一少了一个 for 循环，降为 O（n）

    空间复杂度：所谓的空间换时间， 开辟了一个 hash table ，空间复杂度变为 O（n）

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        //假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍
        Map<Integer,Integer> map=new HashMap<>();
        for(int i=0;i<nums.length;i++){
            map.put(nums[i],i);
        }
        for(int i=0;i<nums.length;i++){
            int sub=target-nums[i];
            if(map.containsKey(sub)&&map.get(sub)!=i){
                return new int[]{i,map.get(sub)};
            }
        }
    throw new IllegalArgumentException("No two sum solution");
    }//method end
}//class end
```

- hash table 优化 简化写法

    两个 for 循环，他们长的一样，我们当然可以把它合起来。复杂度上不会带来什么变化，变化仅仅是不需要判断是不是当前元素了，因为当前元素还没有添加进 hash 里。

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer,Integer> map=new HashMap<>();
        for(int i=0;i<nums.length;i++){
            int sub=target-nums[i];
            if(map.containsKey(sub)){
                return new int[]{i,map.get(sub)};
            }
            map.put(nums[i], i);
        }
        throw new IllegalArgumentException("No two sum solution");
    }//method end
}//class end
```
### [15. 三数之和](https://leetcode-cn.com/problems/3sum/)

```
给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有满足条件且不重复的三元组。

注意：答案中不可以包含重复的三元组。
```

三层for再去重复的暴力笨办法 这里就不写了

- 排序 + 双指针 前后指针 

    返回的是元素数组 ，先排序再结合前后双指针 

    遍历数组，然后再找两个数使得和为 0

    原本遍历需要 O（n），再找两个数使得和为0需要 O（n²）的复杂度，还是需要 O（n³）

    但是 先对数组进行排序后再用两个指针，一个指向头，一个指向尾，去找这两个数字，这样的话，找另外两个数时间复杂度就会从 O（n²），降到 O（n）。

    **保证不加入重复的 list 呢？**

    - 在nums 已经有序后，只需要找到一组之后，当前指针要移到和当前元素不同的地方。
    - 在遍历数组的时候，如果和上个数字相同，也要继续后移。
    - 指针移动时，如果和上个数字相同，也要继续后移。

    时间复杂度：O（n²），n 指的是 num

    空间复杂度：O（N），最坏情况，即 N 是指 n 个元素选3的排列组合个数，用来保存结果

```java
class Solution {
    public static List<List<Integer>> threeSum(int[] nums) {
        int len = nums.length;
        List<List<Integer>> ans = new ArrayList();
        if(nums == null || len < 3) return ans;

        Arrays.sort(nums); // 排序

        for (int i = 0; i < len-2 ; i++) {//遍历数组 其实要留两个指针余量 所以len-2
            if(nums[i] > 0) break; // 如果当前数字大于0，则三数之和一定大于0，所以结束循环
            if(i > 0 && nums[i] == nums[i-1]) continue; // 去重
            int L = i+1;
            int R = len-1;
            while(L < R){
                int sum = nums[i] + nums[L] + nums[R];
                if(sum == 0){
                    ans.add(Arrays.asList(nums[i],nums[L],nums[R]));
                    while (L<R && nums[L] == nums[L+1]) L++; // 去重
                    while (L<R && nums[R] == nums[R-1]) R--; // 去重
                    L++;
                    R--;
                }//if end
                else if (sum < 0) L++;
                else if (sum > 0) R--;
            }//while end
        }//fori end        
        return ans;
    }//method end
}// class end
```
- 同上，另一种写法 单独设置独立变量sum = 0 - nums[i] 便于书写和理解

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
    int len = nums.length;
    List<List<Integer>> res = new LinkedList<>(); 
    if(nums == null || len < 3) return res;

    Arrays.sort(nums); //排序

    for (int i = 0; i < len-2; i++) {
        //为了保证不加入重复的 list,因为是有序的，所以如果和前一个元素相同，只需要继续后移就可以
        if (i == 0 || (i > 0 && nums[i] != nums[i-1])) {
            //两个指针,并且头指针从i + 1开始，防止加入重复的元素
            int lo = i+1, hi = len-1, sum = 0 - nums[i];
            while (lo < hi) {
                if (nums[lo] + nums[hi] == sum) {
                    res.add(Arrays.asList(nums[i], nums[lo], nums[hi]));
                    //元素相同要后移，防止加入重复的 list
                    while (lo < hi && nums[lo] == nums[lo+1]) lo++;
                    while (lo < hi && nums[hi] == nums[hi-1]) hi--;
                    lo++; hi--;
                } else if (nums[lo] + nums[hi] < sum) lo++;
                else hi--;
           }//while end
        }//if i end
    }//for i end
    return res;
    }//method end
}//class end
```
### [18. 四数之和](https://leetcode-cn.com/problems/4sum/)

- 在三数之和上升级 叠加一层循环

    时间复杂度：O（n³）。

    空间复杂度：O（N），最坏情况，即 N 是指 n 个元素选4的排列组合个数，用来保存结果

```java
class Solution {
    public List<List<Integer>> fourSum(int[] nums,int target){
        int len = nums.length;
        List<List<Integer>> res = new LinkedList<>();
        if(nums == null || len < 4) return res;

        Arrays.sort(nums); // 排序

        //多加了层循环
        for (int j = 0; j < len - 3; j++) {
            //防止重复的
            if (j == 0 || (j > 0 && nums[j] != nums[j - 1]))//nums[j] != nums[j - 1]去重复
                for (int i = j + 1; i < len - 2; i++) {
                    //防止重复的，不再是 i == 0 ，因为 i 从 j + 1 开始
                    if (i == j + 1 || nums[i] != nums[i - 1]) {//num[i] != num[i - 1] 去重复
                        int L = i+1;
                        int R = len-1;
                        int  sum = target - nums[j] - nums[i];
                        while (L < R) {
                            if (nums[L] + nums[R] == sum) {
                                res.add(Arrays.asList(nums[j], nums[i], nums[L], nums[R]));
                                while (L < R && nums[L] == nums[L + 1])
                                    L++;//去重复
                                while (L < R && nums[R] == nums[R - 1])
                                    R--;//去重复
                                L++;
                                R--;
                            } else if (nums[L] + nums[R] < sum)
                                L++;
                            else
                                R--;
                        }//while end
                    }//if i end
                }//fori end && if j end
        }// forj end 
        return res;
    }//method end
}//class end
```
- 另一个注解版本 方法一致

```java
class Solution {
    public List<List<Integer>> fourSum(int[] nums,int target){
        /*定义一个返回值*/
        List<List<Integer>> result=new ArrayList<>();
        /*当数组为null或元素小于4个时，直接返回*/
        if(nums==null||nums.length<4){
            return result;
        }
        /*对数组进行从小到大排序*/
        Arrays.sort(nums);
        /*数组长度*/
        int length=nums.length;
        /*定义4个指针k，i，j，h  k从0开始遍历，i从k+1开始遍历，留下j和h，j指向i+1，h指向数组最大值*/
        for(int k=0;k<length-3;k++){
            /*当k的值与前面的值相等时忽略*/
            if(k>0&&nums[k]==nums[k-1]){
                continue;
            }
            /*获取当前最小值，如果最小值比目标值大，说明后面越来越大的值根本没戏*/
            int min1=nums[k]+nums[k+1]+nums[k+2]+nums[k+3];
            if(min1>target){
                break;
            }
            /*获取当前最大值，如果最大值比目标值小，说明后面越来越小的值根本没戏，忽略*/
            int max1=nums[k]+nums[length-1]+nums[length-2]+nums[length-3];
            if(max1<target){
                continue;
            }
            /*第二层循环i，初始值指向k+1*/
            for(int i=k+1;i<length-2;i++){
                /*当i的值与前面的值相等时忽略*/
                if(i>k+1&&nums[i]==nums[i-1]){
                    continue;
                }
                /*定义指针j指向i+1*/
                int j=i+1;
                /*定义指针h指向数组末尾*/
                int h=length-1;
                /*获取当前最小值，如果最小值比目标值大，说明后面越来越大的值根本没戏，忽略*/
                int min=nums[k]+nums[i]+nums[j]+nums[j+1];
                if(min>target){
                    continue;
                }
                /*获取当前最大值，如果最大值比目标值小，说明后面越来越小的值根本没戏，忽略*/
                int max=nums[k]+nums[i]+nums[h]+nums[h-1];
                if(max<target){
                    continue;
                }
                /*开始j指针和h指针的表演，计算当前和，如果等于目标值，j++并去重，h--并去重，当当前和大于目标值时h--，当当前和小于目标值时j++*/
                while (j<h){
                    int curr=nums[k]+nums[i]+nums[j]+nums[h];
                    if(curr==target){
                        result.add(Arrays.asList(nums[k],nums[i],nums[j],nums[h]));
                        j++;
                        while(j<h&&nums[j]==nums[j-1]){
                            j++;
                        }
                        h--;
                        while(j<h&&i<h&&nums[h]==nums[h+1]){
                            h--;
                        }
                    }else if(curr>target){
                        h--;
                    }else {
                       j++;
                    }
                }
            }
        }
        return result;
    }
}
```
### [16. 最接近的三数之和](https://leetcode-cn.com/problems/3sum-closest/)

- 笨办法 暴力 注意：返回值与三数之和 不一样 这里 只要求返回最接近的三数之和的值 不要求给出具体元素

    遍历所有的情况，然后求出三个数的和，和目标值进行比较，选取差值最小的即可。

    时间复杂度：O（n³），三层循环。

    空间复杂度：O（1），常数个。

```java
class Solution {
    public int threeSumClosest(int[] nums, int target) {
        int sub = Integer.MAX_VALUE; //保存和 target 的差值
        int sum = 0; //保存当前最接近 target 的三个数的和
        for (int i = 0; i < nums.length; i++) {
            for (int j = i + 1; j < nums.length; j++)
                for (int k = j + 1; k < nums.length; k++) {
                    if (Math.abs((nums[i] + nums[j] + nums[k] - target)) < sub) {
                        sum = nums[i] + nums[j] + nums[k];
                        sub = Math.abs(sum - target);
                    }
                }
        }
        return sum;
    }
}
```
- 同样的受三数之和启发 数组排序+ 前后指针 降低时间复杂度 这里甚至都不用管去重复问题 会变得很简单

    时间复杂度：如果是快速排序的 O(log_n) 再加上 O（n²），所以就是 O（n²）。

    空间复杂度：O（1），常数个。

```java
class Solution {
    public int threeSumClosest(int[] nums, int target) {
        int len=nums.length;
        int sum=0;
        int sub=Integer.MAX_VALUE;
        Arrays.sort(nums);
        for(int i=0;i<len-2;i++){
            int L=i+1,R=len-1;
            while(L < R){
                if(Math.abs(nums[i]+nums[L]+nums[R]-target)<sub){
                    sub=Math.abs(nums[i]+nums[L]+nums[R]-target);
                    sum=nums[i]+nums[L]+nums[R];
                }
                if(nums[L]+nums[R]+nums[i]>target){
                    R--;
                }else{
                    L++;
                }

            }//while end
        }//fori end
        return sum;
    }//method end
}//class end
```
### [454. 四数相加 II](https://leetcode-cn.com/problems/4sum-ii/)

- 四数之和 变种

```

思路：

1）固定的四个数相加，每个数从A组元中选择一个，树状图画下去 就可得到
 关于程序上的计算，直接三个for嵌套肯定可以实现，O(N^3)，但是不推荐，也不是我们想要的；

2) 四数之和 刚好 拆分两两相组  而且返回值只需要返回个数count
A + B = K; C+ D = -K;

拆分为子问题 A B两数组中 抽取a 和 b相加求和，得到的k 用hashmap存放 key为两数之和sum，value为sum的频次 
Map<Integer,Integer> getMapOf(int[] A,int[] B)

mapAB=getMapOf(A,B);
mapCD=getMapOf(C,D);

对两个map进行遍历，遇到k 和 -k匹配对的时候就对其value进行计数相加

最后返回 count


//另外的 上面过程简化
mapAB=getMapOf(A,B);
得到mapAB后，对CD进行操作时直接判断mapAB中是否包含-k即可进行计数操作
```

```java
/*
上面思路2的简化版本，O(N^2)，暂时找不到能优化下去的了
*/
class Solution {
    public int fourSumCount(int[] A, int[] B, int[] C, int[] D) {
        int res = 0;
        Map<Integer, Integer> map1 = new HashMap<>();
        for (int a : A) {
            for (int b : B) {
                int sum1 = a + b;
                map1.put(sum1, map1.getOrDefault(sum1, 0) + 1);
            }
        }
        for (int c : C) {
            for (int d : D) {
                int sum2 = c + d;
                if(map1.containsKey(-sum2))  
                    res += map1.get(-sum2);
            }
        }
        return res;
    }//method end
}//class end
```
[一个解析](https://leetcode-cn.com/problems/4sum-ii/solution/shuang-zhi-zhen-jie-si-shu-zhi-he-topi-by-a-fei-8/)

### [49. 字母异位词分组](https://leetcode-cn.com/problems/group-anagrams/)

-  字符排序 + 遍历 超时

字母异位词 排序后 一致

>我的第一个思路是hashmap，但是我们可以看看两次for 可以不 

 看看别人的 为什么不超时 [详细通俗的思路分析，多解法](https://leetcode-cn.com/problems/group-anagrams/solution/xiang-xi-tong-su-de-si-lu-fen-xi-duo-jie-fa-by--16/)

 更新：他的也超时了，他说的复杂度O(N*K)，其实也可以说是O(N*N)，始终会先进入循环再判断是否使用，按照后续操作数来看，可以看做O(N*K)；我发现，O(N*N) 就是会超时 我本以为是我我用字符串equals和 他用 hashmap做统计支持的差别 其实 都差不多超时 
```java
/*测试用例ok 超出时间限制 复杂度O(N*N) 还是老实用hashmap吧*/
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        //所有输入均为小写字母。a~z 97~122
        /*
        1）hashmap or arrays 统计字符个数 来判断 是否为一类异或词
        2）字符串 按照字符数组排序后异或词结果一致
        */
        List<List<String>> res=new ArrayList<List<String>>();
        
        int len=strs.length;
        int[] isUse=new int[len];
        for(int i=0;i<len;i++)
            isUse[i]=0;
        for(int i=0;i<len;i++){
            if(isUse[i]!=0) continue;//使用过了的索引 跳过
            List<String> tmp=new ArrayList<String>();//每一层的异或词list
            tmp.add(strs[i]);
            isUse[i]=1;
            for(int j=i+1;j<len;j++){
                if(isUse[j]!=0) continue;//使用过了的索引 跳过
                // todo
                if(getStrByOrder(strs[i]).equals(getStrByOrder(strs[j]))){//getStrByOrder(strs[i]).equals(getStrByOrder(strs[j]))
                    tmp.add(strs[j]);
                    isUse[j]=1;
                }
            }//forj end
            if (tmp != null) {
                res.add(tmp); 
            }    
        }//fori end

        return res;
    }//method end
    //用于给单词 按字典顺序排序
    public String getStrByOrder(String str) {
        char[] ch=str.toCharArray();
        Arrays.sort(ch);
        return String.valueOf(ch);
    }//method end

    //此方法来源于上方链接解释，用hashmap做统计支持 判定字符串的异或相等性
    private boolean equals(String string1, String string2) {
    Map<Character, Integer> hash = new HashMap<>();
    //记录第一个字符串每个字符出现的次数，进行累加
    for (int i = 0; i < string1.length(); i++) {
        if (hash.containsKey(string1.charAt(i))) {
            hash.put(string1.charAt(i), hash.get(string1.charAt(i)) + 1);
        } else {
            hash.put(string1.charAt(i), 1);
        }
    }
     //记录第一个字符串每个字符出现的次数，将之前的每次减 1
    for (int i = 0; i < string2.length(); i++) {
        if (hash.containsKey(string2.charAt(i))) {
            hash.put(string2.charAt(i), hash.get(string2.charAt(i)) - 1);
        } else {
            return false;
        }
    }
    //判断每个字符的次数是不是 0 ，不是的话直接返回 false
    Set<Character> set = hash.keySet();
    for (char c : set) {
        if (hash.get(c) != 0) {
            return false;
        }
    }
    return true;
    }//method end



}//class end
```
- 字符排序 + HashMap<String, List<String>>

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        HashMap<String, List<String>> hashMap = new HashMap<>();//根据String把元素放入对应的ListString
        List<String> anagramGroup;//组，存放字母一样的单词
        for (String curStr : strs) {
            String sortedStr = getStrByOrder(curStr);//每个单词按字母排序
            if (hashMap.containsKey(sortedStr)) {//存在该排序后单词的key，则取出value，将curStr插入
                anagramGroup = hashMap.get(sortedStr);
                anagramGroup.add(curStr);
            } else {//不存在该key 则新建一个list，将curStr add进去，在将该value即list 以sortedStr为key put进去
                anagramGroup = new ArrayList<>();
                anagramGroup.add(curStr);
                hashMap.put(sortedStr, anagramGroup);
            }
        }
        return new ArrayList<>(hashMap.values());
    }

    //用于给单词 按字典顺序排序
    public String getStrByOrder(String str) {
        char[] ch=str.toCharArray();
        Arrays.sort(ch);
        return String.valueOf(ch);
    }//method end
}

```
稍微简化一点代码行数
```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        HashMap<String, List<String>> hash = new HashMap<>();
        for (int i = 0; i < strs.length; i++) {
            char[] s_arr = strs[i].toCharArray();
            Arrays.sort(s_arr);
            String key = String.valueOf(s_arr); 
            if (hash.containsKey(key)) {
                hash.get(key).add(strs[i]);
            } else {
                List<String> temp = new ArrayList<String>();
                temp.add(strs[i]);
                hash.put(key, temp);
            }
        }//fori end
        return new ArrayList<List<String>>(hash.values()); 
    }//method end
}//class end
```
### [447. 回旋镖的数量](https://leetcode-cn.com/problems/number-of-boomerangs/)

- hashmap构建查找表

思路：
（1）遍历所有的点，让每个点作为两点距离的左端点。
（2）然后再遍历其他的点（作为右端点），统计左端点到所有其他点的距离出现的频次（用HashMap存储，Key为距离，Value为频次）
（3）如果Value大于1，则将Value带入 n(n-1) 计算结果并累加到result中

注意点：
（1）如果有一个点a，还有两个点 b 和 c ，如果 ab 和 ac 之间的距离相等，那么就有两种排列方法 abc 和 acb ；
如果有三个点b，c，d 都分别和 a 之间的距离相等，那么有六种排列方法，abc, acb, acd, adc, abd, adb；
如果有 n 个点和点 a 距离相等，那么排列方式为 n(n-1)。
（2）计算距离时不进行开根运算, 以保证精度。

复杂度都是O(N*N) 没想到啥能降低的 才疏学浅

```java
class Solution {
    public int numberOfBoomerangs(int[][] points) {
        int result=0;
        for(int i=0;i<points.length;i++){
            //存储点i到所有其他点的距离出现的频次（Key为距离，Value为频次）
            Map<Integer, Integer> map = new HashMap<>();
            for(int j=0;j<points.length;j++){
                if(i!=j){
                    int dis = dis(points[i], points[j]);
                    if(map.containsKey(dis))
                        map.put(dis,map.get(dis)+1);
                    else
                        map.put(dis,1);
                }
            }
            Set<Map.Entry<Integer, Integer>> entries = map.entrySet();
            for (Map.Entry<Integer, Integer> entry : entries) {
                if(entry.getValue()>1)
                    //如果存在距离相等的两点的数量大于一，则可以组队（回旋镖）
                    //如果存在2个则2种排列方法，存在3个则6种排列方法，存在n个则n（n-1）中排列方法
                    result=result+entry.getValue()*(entry.getValue()-1);
            }
        }
        return result;
    }
    //计算两元组距离
    public static int dis(int[] left, int[] right){
        return (int) (Math.pow(left[0]-right[0],2)+Math.pow(left[1]-right[1],2));
    }
}
```

```java
class Solution {
    public int numberOfBoomerangs(int[][] points) {
        int res = 0;
        //O(n^2)
        for(int i = 0; i < points.length;i++){
            Map<Integer, Integer> record = new HashMap<>();
            for(int j = 0; j < points.length; j ++){
                if( j != i )
                    if(record.containsKey(distance(points[i], points[j]))){
                        record.put(distance(points[i], points[j]),record.get(distance(points[i], points[j])) + 1);
                    }
                    else
                        record.put(distance(points[i], points[j]), 1);
            }//for  j end
            for(int k : record.values()){
                if(k >= 2)//这里其实可以不加这句，因为k=1或k=0，结果都是0，加上可以减少一定的计算量。
                    res += k * (k - 1);
            }      
        }//for i end
        return res;
    }//method end
    private int distance(int[] x, int[] y){
        return (x[0] - y[0]) * (x[0] - y[0]) + (x[1] - y[1]) * (x[1] - y[1]);
    }//method end
}
```
### [149. 直线上最多的点数](https://leetcode-cn.com/problems/max-points-on-a-line/)

题目把我看懵了 在我的认知中 两点确定一条直线 再判断其他点在直线上与否 这显然不能这样模拟解决 复杂度绕死

也不能像人一样 画出来 一眼看吧

用查找表的思想 至少也是两点确定一条直线 有N*(N-1)个两点确定的直线 再判断其他点是否在直线上和直线重合问题？ 

引发问题 谁是key，直线方程y=kx+b,计算得出k和b作为key吗？

所以key为二维数组double/float <k,b> 还是一个length=2的list，value存放直线上的点数 最后return max

- 两次遍历 计算N*(N-1)个 k b
- hashmap存储HashMap<>?
    >数组可以直接做hashmap的key吗？显然 不能

    >数组作为key，只是数组的地址引用的hashcode，Arrays.hashcode静态方法倒是能够根据数组的内容创建相应的hashcode，问题是hashmap用数组做key时用的是前者的hashcode。所以应避免使用数组为键。

    >list倒是可以的，这里也可以k b按照规则转String 比如 "k#b"做 key

    直线的表现形式除了y=kx+b 还可以 一个点A(x,y) 加上斜率k 

    所以可以 针对每个点 对剩余点进行遍历 求得k，用k做hashmap的key；然后对每个点进行上述操作，记录哪条直线上的点最多

    从 **两点确定一条直线的 直线出发**  或者 **从 一个点+斜率k 确定 一条直线出发** 

    **针对于斜率的计算，浮点数可能会bug，有关计算机精度问题，可以优化考虑为分数，化简至最简 比如3/5 用字符串"分子"+"/"+"分母" 构建一个String作为key**

- hashmap 查找表结构

```java
/*
1）斜率的求算中，有时会出现直线为垂直的情况，故需要对返回的结果进行判断，如果分母为0，则返回inf，但是这里还需要避免相同元素 时，不能直接计算
*/
class Solution {
    public int maxPoints(int[][] points) {
        int len=points.length;
        if (len < 3) return len;

        int res = 0;
        //遍历每个点
        for (int i = 0; i < len; i++) {
            int duplicate = 0;//考虑可能会有相同的点，duplicate记录重复次数
            int max = 0;//保存经过当前点的直线中，最多的点
            HashMap<String, Integer> map = new HashMap<>();
            for (int j = i + 1; j < len; j++) {
                //求出分子分母
                int x = points[j][0] - points[i][0];
                int y = points[j][1] - points[i][1];
                if (x == 0 && y == 0) {//重复计数
                    duplicate++;
                    continue;
                }
                //进行约分
                int gcd = gcd(x, y);
                x = x / gcd;
                y = y / gcd;
                String key = x + "#" + y;//可以考虑改StringBuilder
                map.put(key, map.getOrDefault(key, 0) + 1);
                //getOrDefault(key, 0)有key就用key对应的value,没有就使用defaultValue，这里的defaultValue传入值为0
                max = Math.max(max, map.get(key));
            }//forj end
            //1 代表当前考虑的点，duplicate 代表和当前的点重复的点
            res = Math.max(res, max + duplicate + 1);
        }//fori end
        return res;
    }//method end

    private int gcd(int a, int b) {//获取最大公约数
        while (b != 0) {
            int temp = a % b;
            a = b;
            b = temp;
    }
    return a;
    }//method end  

}//class end
```
**车轮滚滚 时间**

- 计算机精度 对除法运算的结果 有可能出现bug
- **hashmap的getOrDefault(key, defaultValue)方法**
    ```java
    map.put(key, map.getOrDefault(key, 0) + 1);
    //getOrDefault(key, 0)有key就用key对应的value,没有就使用defaultValue，这里的defaultValue传入值为0
    //以前没怎么注意到hashmap这个方法 其实挺好用的 

    //当Map集合中有这个key时，就使用这个key值，如果没有就使用默认值defaultValue
    default V getOrDefault(Object key, V defaultValue)
    //如果key值已存在，则不替换对应的value
    V putIfAbsent(K key, V value)
    ```

- [https://leetcode.wang/](https://leetcode.wang/)
