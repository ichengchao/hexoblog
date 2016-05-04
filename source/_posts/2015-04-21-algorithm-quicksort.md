---
layout: post
title: 算法:快速排序
date: 2015-04-21 10:41
tags:
  - java
  - tech
  - algorithm
  - sorting
  
---

### 排序算法介绍 ###
[维基百科的解释](http://en.wikipedia.org/wiki/Sorting_algorithm)  
里面介绍了各种各样的排序算法.  
其中的快速排序有一个非常经典的动态图.  
![quickSort](http://chengchao.name/resource-container/image/sorting_quicksort.gif)

### 代码示例 ###

```java
public class QuickSort {

    public static void main(String[] args) {
        int[] array = { 8, 2, 5, 10, 7, 6, 15, 9, 12 };
        for (int i : array) {
            System.out.print(i);
            System.out.print(",");
        }
        System.out.println();
        sort(array, 0, array.length - 1);
        for (int i : array) {
            System.out.print(i);
            System.out.print(",");
        }

    }

    /**
     * 快速排序,递归
     * 
     * @param array
     * @param low
     * @param high
     */
    public static void sort(int[] array, int low, int high) {
        if (low < high) {
            int p = partion(array, low, high);
            sort(array, low, p - 1);
            sort(array, p + 1, high);
        }
    }

    /**
     * 将数据分区,比中轴小的放在左边,大的放在右边
     * 
     * @param array
     * @param low
     * @param high
     * @return
     */
    public static int partion(int[] array, int low, int high) {
        // 取最低位为中轴,并把该位置当做临时位进行交换
        int middle = array[low];
        while (low < high) {
            // 先从右往左找一个比中轴小的交换到交换位
            while (low < high && array[high] > middle) {
                high--;
            }
            array[low] = array[high];
            // 先从左往右找一个比中轴大的交换到交换位
            while (low < high && array[low] < middle) {
                low++;
            }
            array[high] = array[low];
        }
        // 整个交换完成后将中轴放回到交换位
        array[low] = middle;
        return low;
    }
    
    
    // *****************************以下这种实现跟动态图更为接近**************************************

    public static int partion2(int[] array, int low, int high) {
        int middle = array[low];
        int index = low;
        low++;
        while (low < high) {
            while (low < high && array[high] > middle) {
                high--;
            }
            while (low < high && array[low] < middle) {
                low++;
            }
            if (low == high) {
                break;
            } else {
                exchangeElements(array, low, high);
            }

        }
        // 放置中轴
        exchangeElements(array, low, index);
        return low;
    }

    /**
     * 把array中的index1和index2的内容互换
     * 
     * @param array
     * @param index1
     * @param index2
     */
    public static void exchangeElements(int[] array, int index1, int index2) {
        if (array == null || array.length < 1) {
            throw new IllegalArgumentException("array can not be null or empty!");
        }
        if (index1 < 0 || index2 < 0 || index1 > (array.length - 1) || index2 > (array.length - 1)) {
            throw new IllegalArgumentException("index must >0 and < array's size");
        }
        if (index1 == index2) {
            return;
        }
        int temp = array[index1];
        array[index1] = array[index2];
        array[index2] = temp;
    }

}

```