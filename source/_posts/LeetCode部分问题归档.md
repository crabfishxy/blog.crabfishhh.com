---
title: LeetCode部分问题归档
date: 2018-03-29 13:23:02
tags: 算法 OJ
description: 摘取部分LeetCode中遇到的对个人有启发作用的问题和解法
---

# Longest Valid Parentheses

## Description

Given a string containing just the characters '(' and ')', find the length of the longest valid (well-formed) parentheses substring.

For "(()", the longest valid parentheses substring is "()", which has length = 2.

Another example is ")()())", where the longest valid parentheses substring is "()()", which has length = 4.

## My Solution

我的方法比讨论区的要复杂一些，并且时间效率也不高，主要还是因为动态规划不熟练。

用了一个stack，一个和字符串长度相同的数组来解决。大体分为两步：

1. index和类型组成一个Map入栈，没当一个'('和一个')'匹配掉，将这对括号的index对应的数组置为1
2. 遍历完成后对数组进行遍历，找到最长的连续'1'即可

## My Code

```Java
import java.util.*;
class Solution {
    public int longestValidParentheses(String s) {
        Stack<Map.Entry<Integer, String>> stack = new Stack<>();
        List<Integer> list = new LinkedList<>();
        int f[] = new int[s.length()];
        int max = 0;
        for(int i = 0; i < s.length(); i ++){
            String c = "" + s.charAt(i);
            if(stack.size() == 0){
                stack.push(new AbstractMap.SimpleEntry<>(i, c));
                continue;
            }
            if(stack.peek().getValue().equals(")")){
                stack.clear();
                stack.push(new AbstractMap.SimpleEntry<>(i, c));
                continue;
            }
            if(stack.peek().getValue().equals("(") && c.equals(")")){
                f[i] = 1;
                f[stack.peek().getKey()] = 1;
                stack.pop();

                continue;
            }
            if(stack.peek().getValue().equals("(") && c.equals("(")){
                stack.push(new AbstractMap.SimpleEntry<>(i, c));
            }else{
                stack.push(new AbstractMap.SimpleEntry<>(i, c));
            }


        }
        for(int i =1 ; i < s.length(); i ++){
            if(f[i] != 0 ){
                f[i] = f[i-1] + 1;
                if(f[i] > max)max = f[i];
            }
        }
        return max/2*2;
    }
}
```

## Standard Solution

DP：

1. s[i] = ')' and s[i-1] = '('. dp[i] = dp[i-2] + 2

2. s[i] = ')' and s[i-1] = ')'

if s[i-dp[i-1]-1] = '(' then


dp[i]=dp[i−1]+dp[i−dp[i−1]−2]+2

-------

# Maximal Rectangle

## Description

Given a 2D binary matrix filled with 0's and 1's, find the largest rectangle containing only 1's and return its area.

For example, given the following matrix:

```
1 0 1 0 0
1 0 1 1 1
1 1 1 1 1
1 0 0 1 0
```

Return 6.

## Solution

Discuss区的被顶到最上面的方法真的非常简洁明了，虽然这道题的tag已经标明是用DP做，但在看到他的答案之前真的没有想到这么简洁的方法：

大致思路是先行后列，在遍历列的过程中记录该列这个点上方的高度（即1的数量），然后再得到左右高度均超过或等于这个值的界限，从而得到面积。

## code

```C++
class Solution {
    public:
int maximalRectangle(vector<vector<char> > &matrix) {
    if(matrix.empty()) return 0;
    const int m = matrix.size();
    const int n = matrix[0].size();
    int left[n], right[n], height[n];
    fill_n(left,n,0); fill_n(right,n,n); fill_n(height,n,0);
    int maxA = 0;
    for(int i=0; i<m; i++) {
        int cur_left=0, cur_right=n; 
        for(int j=0; j<n; j++) { // compute height (can do this from either side)
            if(matrix[i][j]=='1') height[j]++; 
            else height[j]=0;
        }
        for(int j=0; j<n; j++) { // compute left (from left to right)
            if(matrix[i][j]=='1') left[j]=max(left[j],cur_left);
            else {left[j]=0; cur_left=j+1;}
        }
        // compute right (from right to left)
        for(int j=n-1; j>=0; j--) {
            if(matrix[i][j]=='1') right[j]=min(right[j],cur_right);
            else {right[j]=n; cur_right=j;}    
        }
        // compute the area of rectangle (can do this from either side)
        for(int j=0; j<n; j++)
            maxA = max(maxA,(right[j]-left[j])*height[j]);
    }
    return maxA;
}
```

----------

# Single Number II

## Description

****