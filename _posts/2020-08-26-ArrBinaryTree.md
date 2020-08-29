---
layout: post
title: '数组二叉树记录'
subtitle: '数组实现顺序二叉树'
date: 2020-08-26
author: Dave
cover: ''
tags: Java 
---


### 数组二叉树

```java
An e-commerce company has launched some new products To promote 
these newly launched products the company adds people from 
elsewhere in the company to their marketing teams. The people 
are added in a hierarchical manner. A person can add at most 
two subordinates under him/her. A new person can be added 
under one person only. The person who starts the chain is at 
the top of the hierarchy. Every person in the hierarchy is 
given a unique ID The marketing head is of the new team at 
the top of the hierarchy and is given the ID O. Every person 
in the hierarchy is also assigned the count(sales value) of 
products sold by him/her in the previous year. The company 
wants to assign new sales targets for the upcoming year to 
each employee. For this purpose they arrange the sales values 
of the employees by traversing the hierarchy in an inorder manner. 
The inorder traversal starts with the left subordinate then 
proceeds to the person who added him, then it moves to the 
right subordinate of the person. The new sales value is the 
sum of the current sales of the person to the left and the 
person to the right in the inorder traversal Write an algorithm 
to help the company find the new sales value assigned to 
each employee. The first line of the input consists of an 
integer -num, representing the number of nodes(N) The second 
line consists of N space-separated integers representing the 
subordinates, where the ith person has left and right subordinates 
at 2i+1 and 2i+2 positions, respectively Output Print space-separated 
integers representing the new sales values assigned to the employees 
in an inorder manner. If the value in the hierarchy is-1 then there 
is no need to calculate a new sales value as this person does not exist. 
The value -1 is never considered in the calculation of the new sales value for any person


The inorder traversal results are [5, 2, 1, 7, 9, 6]
The new sales value for the person with the current sale value l is 9(2+7)
The new sales value for the person with the current sale value 5 is 2(0+2). 
This person has no person to his left in inorder traversal,so the value is 
considered as o for the nonperson to the left.
The new sales values for the remaining people are similarly calculated 
Arranging the new values in inorder manner, the output is [2, 6, 9, 10, 13, 9]

```

>其实就是一个数组二叉树，-1代表此处没有节点,中序遍历后的输出进行处理，处理过程为：

顺序二叉树的满足条件：

1.一般指完全二叉树

2.第n个元素的左子树为2*n+1；

3.第n个元素的右子树为2*n+2；

4.第n个子树的父节点为（n-1）/2;

注意：n为数组下标所以是从0开始

```java
import java.util.*;

public class Main {
    public static void main(String[] args) throws Exception{
        int n=7;
        int[] in=new int[]{1,2,9,5,-1,7,6};
        System.out.print("input["+in.length+"]:");
        for(int i:in){
            System.out.print(i+" ");
        }
        System.out.println(" ");
        System.out.println("=======================");

        //preOrder(in,0);

        ArrayList<Integer> list=new ArrayList<Integer>();
        infixOrder(in,0,list);

        System.out.println(" ");
        System.out.println("=======================");

        int len=list.size();
        int[] res=new int[len];
        res[0]=list.get(1);
        for (int i = 1; i < len-1; i++) {
            res[i]=list.get(i-1)+list.get(i+1);
        }
        res[len-1]=list.get(len-2);
        System.out.print("res["+len+"]:");
        for(int i:res){
            System.out.print(i+" ");
        }
    }

    public static void infixOrder(int[] arr,int index,ArrayList<Integer> list) {
        if (arr == null && arr.length == 0) {
            System.out.println("数组为空");
            return;
        }
        //先遍历左子树 （index*2+1）
        if ((index * 2 + 1) < arr.length) {
            infixOrder(arr,index * 2 + 1,list);
        }
        if (index < arr.length && arr[index]!= -1) {
            System.out.print(arr[index] + "\t");
            list.add(arr[index]);
        }

        if ((index * 2 + 2) < arr.length) {
            infixOrder(arr,index * 2 + 2,list);
        }
    }

}

```


**车轮滚滚 时间**

- **数组二叉树的各种遍历**

```java
    package binaryTree;

/*
 * @ClassName : ArrBinaryTree
 * @Description : 数组实现的顺序二叉树
 * @Author : Dave
 * @Date: 2020-08-30 02:38
 **/
public class ArrBinaryTree {
    public static void main(String[] args) {
        int[] arr = {1, 2, 3, 4, 5, 6, 7};
        BinaryTree arrBinaryTree = new BinaryTree(arr);
        BinaryTree.postOrder(0);
    }
}

class BinaryTree {
    private static int[] arr;

    public BinaryTree(int[] arr) {
        this.arr = arr;
    }

    //前序遍历数组
    /*
    * @Param [index] 数组索引下标
    * @description
    * @author Dave
    * @date 2020/8/30 2:39
    * @return void
    * @throws
    **/
    public static void preOrder(int index) {
        if (arr == null && arr.length == 0) {
            System.out.println("数组为空");
            return;
        }
        System.out.println(arr[index]);
        if ((index * 2 + 1) < arr.length) {
            preOrder(index * 2 + 1);
        }
        if ((index * 2 + 2) < arr.length) {
            preOrder(index * 2 + 2);
        }


    }

    //中序遍历
    public static void infixOrder(int index) {
        if (arr == null && arr.length == 0) {
            System.out.println("数组为空");
            return;
        }
        //先遍历左子树 （index*2+1）
        if ((index * 2 + 1) < arr.length) {
            infixOrder(index * 2 + 1);
        }
        if (index < arr.length) {
            System.out.print(arr[index] + "\t");
        }

        if ((index * 2 + 2) < arr.length) {
            infixOrder(index * 2 + 2);
        }
    }

    //后序遍历
    public static void postOrder(int index) {
        if (arr == null && arr.length == 0) {
            System.out.println("数组为空");
            return;
        }
        if ((index * 2 + 1) < arr.length) {
            postOrder(index * 2 + 1);
        }
        if ((index * 2 + 2) < arr.length) {
            postOrder(index * 2 + 2);
        }
        if (index < arr.length) {
            System.out.println(arr[index]+"\t");
        }
    }

}

```
