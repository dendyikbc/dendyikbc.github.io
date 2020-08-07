---
layout: post
title: '剑指offer中的二叉树 (1)'
subtitle: '关于二叉树的层序遍历'
date: 2020-08-01
author: Dave
tags: Java 
---

![](https://raw.githubusercontent.com/dendyikbc/PicGoBed/master/img/binary-tree- traverce.jpg)
## 关于二叉树的层序遍历

## 二叉树遍历模板
- DFS遍历 

DFS递归模板
```java
void dfs(TreeNode root) {
    if (root == null) {
        return;
    }
    dfs(root.left);
    dfs(root.right);
}
```

- BFS遍历 使用队列数据结构
    - 层序遍历

    - 无权最短路径问题
        >注：带权最短路径问题用Dijkstra 算法

>个人喜好用ArrayDeque实现

BFS模板
```java
void bfs(TreeNode root) {
    Queue<TreeNode> queue = new ArrayDeque<>();
    
    queue.add(root);//root需要判定是否为null，只是模板在这里简写了
    while (!queue.isEmpty()) {
        TreeNode node = queue.poll(); // 等同于pollFirst和removeFirst()
        if (node.left != null) {
            queue.add(node.left);
        }
        if (node.right != null) {
            queue.add(node.right);
        }
    }
}

```


## 题型应用

## 从上往下打印二叉树 
>原始版本 一维List
- 剑指offer 22

- 题目描述：
    ```
    从上往下打印出二叉树的每个节点，同层节点从左至右打印。
    ```


**BFS 的ArrayDeque实现**

```java
//直接套用BFS模板
//一维List输出  不需要n分层计数
import java.util.ArrayList;
import java.util.Queue;
import java.util.ArrayDeque;
public class Solution {
    public ArrayList<Integer> PrintFromTopToBottom(TreeNode root) {
        if(root == null) return new ArrayList<Integer>();
        ArrayList<Integer> lists = new ArrayList<>();
        Queue<TreeNode> queue = new ArrayDeque<>();

        queue.add(root);//如果没有第一句  判断root是否为null 此处需要if判断
        while(!queue.isEmpty()){
            //peek（访问队头元素）、poll（访问队头元素并移除）
            TreeNode node = queue.poll();

            lists.add(node.val);//ArrayDeque允许null值 所以这里 弹出的一定不是null 即存在node.val
            //ArrayDeque不能添加null值，不然会报空指针  类实现 源码有写
            if (node.left != null) {
                queue.add(node.left);
            }
            if (node.right != null) {
                queue.add(node.right);
            }
        }

        return lists;
    }
}
```
**BFS 的LinkedList实现**

LinkedList的实现 可以完全照搬ArrayDeque；也可以根据LinkedList允许null元素，在入队和出队上修整
```java
//缩减一点点 Queue接口由Deque接口实现，Deque也可由LinkedList类实现，这里用LinkedList类实现
//一维List输出  不需要n分层计数
import java.util.ArrayList;
import java.util.Queue;
import java.util.LinkedList;
public class Solution {
    public ArrayList<Integer> PrintFromTopToBottom(TreeNode root) {
        if(root == null) return new ArrayList<Integer>();//使用LinkedList类实现时，因允许null元素，所以此步其实可以省略
        ArrayList<Integer> lists = new ArrayList<>();
        Queue<TreeNode> queue = new LinkedList<>();
        queue.add(root);//LinkedList 内部链表实现 允许元素为null，可以直接add 在输出时 判定null即可

        while(!queue.isEmpty()){
            //peek（访问队头元素）、poll（访问队头元素并移除）
            TreeNode node = queue.poll();

            if(node != null){//需要先判定弹出的是否为null值，正因如此，弹出时判空再执行操作，所以进队列时我们可以不管是否为null
                lists.add(node.val);
                //LinkedList 内部链表实现 允许元素为null 所以这里 node.left是否为null均可直接添加 
                //主要是在执行添加前 有判null与否 这里就可以无需判断
                queue.add(node.left);
                queue.add(node.right);
            }
        }

        return lists;
    }
}
```

**DFS 的递归实现**

```java

先分层递归后修整成一个维度

其实 一维的层序遍历 不建议dfs 不如bfs来得爽快

```



## 把二叉树打印成多行  
>层序遍历 分层打印 二维List

- 剑指offer 60

- 题目描述：

  ```
  从上到下按层打印二叉树，同一层结点从左至右输出。每一层输出一行。
  ```
思路：典型的层序遍历 分层打印 在原有BFS层序遍历的基础之上进行二维构造 输出二维List 每层需要记录元素个数

**BFS 的ArrayDeque实现**

```java
import java.util.ArrayDeque;
import java.util.ArrayList;
import java.util.Queue;
/*
public class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;

    }

}
*/
public class Solution {
    ArrayList<ArrayList<Integer> > Print(TreeNode pRoot) {
        ArrayList<ArrayList<Integer>> res = new ArrayList<>();
        Queue<TreeNode> queue = new ArrayDeque<>();

        if (pRoot != null) {
                queue.add(pRoot);
        }

        while (!queue.isEmpty()) {
            int n = queue.size();//记录每层元素个数
            ArrayList<Integer> level = new ArrayList<>();
            for (int i = 0; i < n; i++) { //处理这一层元素
                TreeNode node = queue.poll();//弹出
                level.add(node.val);
                if (node.left != null) {
                    queue.add(node.left);
                }
                if (node.right != null) {
                    queue.add(node.right);
                }
            }//for end  这一层元素处理完毕，此时level才没有层次交错元素混杂
            res.add(level);
        }//while end
        return res;
    }
       
}

```


**DFS 的递归实现**

```java
import java.util.ArrayList;

public class Solution {
 ArrayList<ArrayList<Integer>> nodesList = new ArrayList<ArrayList<Integer>>();
    public ArrayList<ArrayList<Integer>> Print(TreeNode pRoot) {
        if(pRoot == null) return nodesList;
        int level = 0;
        dfs(pRoot,level);
        return nodesList;

    }

    private void dfs(TreeNode pRoot, int level) {
        if(pRoot == null) return;

        if(nodesList.size() == level){
            nodesList.add(new ArrayList<Integer>());
        }
        nodesList.get(level).add(pRoot.val);
        dfs(pRoot.left,level+1);
        dfs(pRoot.right,level+1);
    }
    
}
```

## 按之字形顺序打印二叉树 
 
>层序遍历 分层打印 二维List 偶数层逆序

- 剑指offer 59

- 题目描述：

  ```
  请实现一个函数按照之字形打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右至左的顺序打印，第三行按照从左到右的顺序打印，其他行以此类推。
  ```

**DFS 的递归实现**

直接在DFS递归中，利用**Collections.reverse()**函数反转行内顺序
```java
import java.util.ArrayList;
import java.util.Collections;

public class Solution {
 ArrayList<ArrayList<Integer>> nodesList = new ArrayList<ArrayList<Integer>>();
    public ArrayList<ArrayList<Integer>> Print(TreeNode pRoot) {
        if(pRoot == null) return nodesList;
        int level = 0;//
        dfs(pRoot,level);
        for (int i = 0; i < nodesList.size(); i++) {
            if(i % 2 == 1){//偶数行（索引为奇数）逆序
                Collections.reverse(nodesList.get(i));
            }
        }
        return nodesList;

    }

    private void dfs(TreeNode pRoot, int level) {
        if(pRoot == null) return;

        if(nodesList.size() == level){
            nodesList.add(new ArrayList<Integer>());
        }
        nodesList.get(level).add(pRoot.val);
        dfs(pRoot.left,level+1);
        dfs(pRoot.right,level+1);
    }
    
}
```

**BFS 的LinkedList实现**

层序遍历 按行打印 利用LinkedList的双向链表特性 偶数层倒序添加 

解析可参考
- [LeetCode面试题32 - III. 从上到下打印二叉树 III（层序遍历 BFS / 双端队列，清晰图解）](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-iii-lcof/solution/mian-shi-ti-32-iii-cong-shang-dao-xia-da-yin-er--3/)

```java
//这里用的LinkedList实现队列，但是写法向BFS模板中的ArrayDeque靠齐  可以针对null 适当改写
import java.util.ArrayList;
import java.util.LinkedList;
import java.util.Queue;
class Solution {
    public ArrayList<ArrayList<Integer>> Print(TreeNode pRoot) {
        Queue<TreeNode> queue = new LinkedList<>();
        ArrayList<ArrayList<Integer>> res = new ArrayList<>();
        if(pRoot != null) queue.add(pRoot);
        while(!queue.isEmpty()) {
            LinkedList<Integer> tmp = new LinkedList<>();
            
            for(int i = queue.size(); i > 0; i--) {
                
                TreeNode node = queue.poll();//出队 队列首部 LinkedList的remove 等同于  poll
                if(res.size() % 2 == 0) 
                    tmp.addLast(node.val); // addLast 等同于add 顺序  第0层 开始 这里 为偶数索引 实际为奇数层
                else 
                    tmp.addFirst(node.val); // 倒序了啦，这里就
                
                if(node.left != null) //LinkedList 实现的queue 也可以在 弹出时 判断null 这里是在 入队时判断null
                    queue.add(node.left);
                if(node.right != null) 
                    queue.add(node.right);
            }
            
            ArrayList<Integer> arrayTmp = new ArrayList(tmp);//LinkedList转ArrayList
            res.add(arrayTmp);
        }
        return res;
    }
}

```
