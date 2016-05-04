---
layout: post
title: 算法:最长公共子序列
date: 2015-04-02 23:49
tags:
  - java
  - tech
  - algorithm
---

### 背景 ###
之前做项目,需要做一个文本文件的在线diff.其中用到的就是LCS算法.
今天心血来潮,打算用java写个示例

### LCS算法介绍 ###
[维基百科的解释](http://en.wikipedia.org/wiki/Longest_common_subsequence_problem)
  
网上的介绍比较多,不细讲  
就是用一个二维矩阵来记录两个字符串中所有位置的匹配情况,若是匹配则为1,否则为0.  
然后求出对角线最长的1序列，其对应的位置就是最长匹配子串的位置.  
当然如果扩展成一个文本文件的话,就是一行一个单位.  
这里就不贴图了,直接google,或者看wikipedia里面的图.

### 代码示例 ###

```java
package javaTest;

import junit.framework.Assert;

import org.junit.Test;

/**
 * LCS 最长公共子序列演示
 * 
 * @author charles-dell 2015年4月2日 下午11:32:31
 */
public class LCS {

    @Test
    public void test_lcs() {
        // case1
        String x = "abcdefg";
        String y = "fghijklmn";

        String lcs = getLCS(x, y);
        Assert.assertEquals("fg", lcs);

        // case2
        x = "aaaaaaa";
        y = "aaaaaaa";
        lcs = getLCS(x, y);
        Assert.assertEquals("aaaaaaa", lcs);

        // case3
        x = "abcde";
        y = "fghij";
        lcs = getLCS(x, y);
        Assert.assertEquals("nothing", lcs);

        // case4
        x = "asdfg";
        y = "asdjgfhjjhsdfg";
        lcs = getLCS(x, y);
        Assert.assertEquals("sdfg", lcs);
    }

    /**
     * 获取最长公共子序列
     * 
     * @param x
     * @param y
     * @return
     */
    public static String getLCS(String x, String y) {

        char[] xc = x.toCharArray();
        char[] yc = y.toCharArray();

        int xLen = x.length();
        int yLen = y.length();
        int[][] myArray = new int[xLen][yLen];

        int max = 0;
        int flag = 0;

        for (int i = 0; i < xLen; i++) {
            for (int j = 0; j < yLen; j++) {
                if (xc[i] == yc[j]) {
                    if ((i - 1) >= 0 && (j - 1) >= 0) {
                        myArray[i][j] = myArray[i - 1][j - 1] + 1;
                    } else {
                        myArray[i][j] = 1;
                    }

                    if (max < myArray[i][j]) {
                        max = myArray[i][j];
                        flag = j;
                    }
                } else {
                    myArray[i][j] = 0;
                }
            }
        }

        // 打印二维矩阵图
        System.out.println("**********************矩阵图************************");
        System.out.println();
        System.out.print("#");
        for (char c : yc) {
            System.out.print(" " + c);
        }
        System.out.println("");
        for (int i = 0; i < xLen; i++) {
            System.out.print(xc[i]);
            for (int j = 0; j < yLen; j++) {
                System.out.print(" " + myArray[i][j]);
            }
            System.out.println("");
        }

        // 如果有>=2的公共子序列,就打印出来
        if (max >= 2) {
            return (y.substring((flag + 1) - max, flag + 1));
        } else {
            return "nothing";
        }

    }

    /**
     * 打印没有增加的二维矩阵
     * 
     * @param x 字符串x
     * @param y 字符串y
     */
    public static void printMatrixNoIncrement(String x, String y) {

        char[] xc = x.toCharArray();
        char[] yc = y.toCharArray();

        int xLen = x.length();
        int yLen = y.length();
        int[][] myArray = new int[xLen][yLen];

        for (int i = 0; i < xLen; i++) {
            for (int j = 0; j < yLen; j++) {
                if (xc[i] == yc[j]) {
                    myArray[i][j] = 1;
                } else {
                    myArray[i][j] = 0;
                }
            }
        }

        System.out.print("#");
        for (char c : yc) {
            System.out.print(" " + c);
        }
        System.out.println("");
        for (int i = 0; i < xLen; i++) {
            System.out.print(xc[i]);
            for (int j = 0; j < yLen; j++) {
                System.out.print(" " + myArray[i][j]);
            }
            System.out.println("");
        }

    }

}
```
<!--
<script src="https://gist.github.com/ichengchao/517721eed0cc29e7c2a5.js"></script>
-->