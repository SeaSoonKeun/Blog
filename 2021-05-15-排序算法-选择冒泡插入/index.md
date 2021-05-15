# 排序算法-选择，冒泡，插入


排序算法：

## 1. 选择排序

```java
package Class01;

import org.w3c.dom.ls.LSOutput;

import java.util.Arrays;
import java.util.Random;

/**
 * @Auther: xucg
 * @Date: 2021/5/15 - 05 - 15 - 下午4:02
 * @Description: Class01
 */
/* 选择排序思路
从小到大排序情况：
每次选择数组中最小的数 与数组第一个数进行交换位置，
然后选择前n-1个数值进行再次排序，然后再选择其中最小的数与第二个数进行交换
 */
public class SelectionSort {
    public static void selectionSort(int arr[]){
        for (int i = 0; i < arr.length; i++) {
            int minindex = i;
            for (int j = i + 1; j < arr.length; j++) {
                minindex = arr[j] < arr[minindex] ? j : minindex;
            }
            // 因为采用异或的方式进行置换元素，所以需要保证两个数不一样
            if( i != minindex){
                swap(arr, i, minindex);
            }
        }
    }
// 交换两个数组元素的位置
    public static void swap(int arr[], int i, int j){
        arr[i] = arr[i] ^ arr[j];
        arr[j] = arr[i] ^ arr[j];
        arr[i] = arr[i] ^ arr[j];
//        int tmp = arr[i];
//        arr[i] = arr[j];
//        arr[j] = tmp;
    }
// 随机生成数组
    public static int[] generateRadomArray(int maxSize, int maxValue){
        int[] arr = new int[(int)((maxSize + 1) * Math.random())];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = (int)((maxValue + 1) * Math.random()) - (int)((maxValue + 1) * Math.random());
        }
        return arr;
    }
// 对数器
    public static void comparator(int[] arr){
        Arrays.sort(arr);
    }
// 比较两个数数组是否一致
    public static boolean isEqual(int[] arr1, int[] arr2 ){
        if (arr1.length != arr2.length){
            return false;
        }
        if (arr1 == null && arr2 == null){
            return true;
        }
        if ((arr1 == null && arr2 != null) || (arr2 == null && arr1 != null)){
            return false;
        }
        for (int i = 0; i < arr1.length; i++){
            if(arr1[i] != arr2[i]){
                return false;
            }
        }
        return true;
    }
// 拷贝数组至一个新的数组
    public static int[] copyArray(int[] arr1 ){
        int[] arr2 = new int[arr1.length];
        for (int i = 0; i < arr1.length; i++) {
            arr2[i] = arr1[i];
        }
        return arr2;
    }
// 输出数组
    public static void printArray(int[] arr){
        for (int i = 0; i < arr.length; i++) {
            System.out.print(arr[i] + " ");
        }
        System.out.println();
    }
    public static void main(String[] args) {
        int testTime = 100;
        int maxSize = 10;
        int maxValue = 20;
        for (int i = 0; i < testTime; i++) {
            int[] arr1 = generateRadomArray(maxSize, maxValue);
            int[] arr2 = copyArray(arr1);
            selectionSort(arr1);
            printArray(arr1);
            comparator(arr2);
            printArray(arr2);
            System.out.println(isEqual(arr1,arr2) ? "success":"failed");
        }
    }
}

```



## 2.冒泡排序

```java
package Class01;

import javax.imageio.plugins.tiff.TIFFImageReadParam;
import java.util.Arrays;

/**
 * @Auther: xucg
 * @Date: 2021/5/15 - 05 - 15 - 下午8:23
 * @Description: Class01
 */

/*
解题思路：
从左到右，增序排列
第一层：
第一个和第二个比，大的冒泡
第二个和第三个比，大的冒泡，
……
直到最大的冒泡到最后面，利用swap交换
第二层：
依次第二大的冒泡到倒数第二个位置
……
 */
public class BubbleSort {
    public static void bubbleSort(int[] arr){
        // 只需要比到倒数第二个数，因为倒数第二个会和最后一个数比较
        for (int i = 0; i < arr.length - 1; i++) {
            //因为需要比较j+1，所以j需要在arrlength - i 基础上再减一
            for (int j = 0 ; j < arr.length - i - 1; j++) {
                if (arr[j] > arr[j + 1]){
                    swap(arr, j, j+1);
                }
            }
        }
    }
    // 随机生成数据
    public static int[] generatArray(int maxSize, int maxValue){
        int length = (int)((maxSize + 1) * Math.random());
        int[] arr = new int[length];
        for (int i = 0; i < length; i++) {
            int value = (int)((maxValue + 1) * Math.random()) - (int)((maxValue + 1) * Math.random());
            arr[i] = value;
        }
        return arr;
    }
    // 交换数组元素
    public static void swap(int[] arr, int a, int b){
        arr[a] = arr[a] ^ arr[b];
        arr[b] = arr[a] ^ arr[b];
        arr[a] = arr[a] ^ arr[b];
    }
    // Arrays工具对数器
    public static void comparator(int[] arr){
        Arrays.sort(arr);
    }
    // 复制数组
    public static int[] copyArray(int[] arr){
        int[] arr2 = new int[arr.length];
        for (int i = 0; i < arr.length; i++) {
            arr2[i] = arr[i];
        }
        return arr2;
    }
    // 打印数组
    public static void printArray(int[] arr){
        for (int i = 0; i < arr.length; i++) {
            System.out.print(arr[i] + " ");
        }
        System.out.println();
    }
    // 判断两个数组是否相等
    public static boolean isEqual (int[] arr1, int[] arr2){
        if (arr1.length != arr2.length){
            return false;
        }
        if (((arr1 == null)&& (arr2 != null))||((arr1 != null)&&(arr2 == null))){
            return false;
        }
        for (int i = 0; i < arr1.length; i++) {
            if (arr1[i] != arr2[i]){
                return false;
            }
        }
        return true;
    }
    public static void main(String[] args) {
        int testTime = 100;
        int maxSize = 10;
        int maxValue = 20;
        for (int i = 0; i < testTime; i++) {
            int[] arr1 = generatArray(maxSize, maxValue);
            //printArray(arr1);
            int[] arr2 = copyArray(arr1);
            bubbleSort(arr1);
            //printArray(arr1);
            comparator(arr2);
            //printArray(arr2);
            System.out.println(isEqual(arr1, arr2) ? "success" : "failed");
        }
    }
}

```

## 3.插入排序

```java
package Class01;

import java.util.Arrays;

/**
 * @Auther: xucg
 * @Date: 2021/5/15 - 05 - 15 - 下午9:37
 * @Description: Class01
 */
/*
解题思路：
一个一个取，然后插入到前面的数组中；移动过程有点类似与冒泡算法
 */
public class InsertSort {
    public static void insertSort(int[] arr){
        if(arr.length == 0 || arr.length == 1)
            return;
        // 从第二个数开始
        for (int i = 1; i < arr.length; i++) {
            // 注意此处判断条件的顺序，不可以反，否则会引起数组越界
            for (int j = i - 1; j >= 0 && arr[j + 1] < arr[j]; j--){
                //类似冒泡排序算法交换元素
                swap(arr, j, j+1);
            }
        }
    }
    public static int[] generatArray(int maxSize, int maxValue){
        int length = (int)((maxSize + 1) * Math.random());
        int[] arr = new int[length];
        for (int i = 0; i < length; i++) {
            int value = (int)((maxValue + 1) * Math.random()) - (int)((maxValue + 1) * Math.random());
            arr[i] = value;
        }
        return arr;
    }
    public static void swap(int[] arr, int a, int b){
        arr[a] = arr[a] ^ arr[b];
        arr[b] = arr[a] ^ arr[b];
        arr[a] = arr[a] ^ arr[b];
    }
    // Arrays工具对数器
    public static void comparator(int[] arr){
        Arrays.sort(arr);
    }
    // 复制数组
    public static int[] copyArray(int[] arr){
        int[] arr2 = new int[arr.length];
        for (int i = 0; i < arr.length; i++) {
            arr2[i] = arr[i];
        }
        return arr2;
    }
    // 打印数组
    public static void printArray(int[] arr){
        for (int i = 0; i < arr.length; i++) {
            System.out.print(arr[i] + " ");
        }
        System.out.println();
    }
    // 判断两个数组是否相等
    public static boolean isEqual (int[] arr1, int[] arr2){
        if (arr1.length != arr2.length){
            return false;
        }
        if (((arr1 == null)&& (arr2 != null))||((arr1 != null)&&(arr2 == null))){
            return false;
        }
        for (int i = 0; i < arr1.length; i++) {
            if (arr1[i] != arr2[i]){
                return false;
            }
        }
        return true;
    }
    public static void main(String[] args) {
        int testTime = 100;
        int maxSize = 10;
        int maxValue = 20;
        for (int i = 0; i < testTime; i++) {
            int[] arr1 = generatArray(maxSize, maxValue);
//            printArray(arr1);
            int[] arr2 = copyArray(arr1);
            insertSort(arr1);
//            printArray(arr1);
            comparator(arr2);
//            printArray(arr2);
            System.out.println(isEqual(arr1, arr2) ? "success" : "failed");
        }
    }
}

```


