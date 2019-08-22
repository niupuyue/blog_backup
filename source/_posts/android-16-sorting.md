---
title: 排序算法总结
date: 2016-09-30 15:03:50
tags:
  - 排序算法
---
排序算法
<!--more-->
|排序算法|平均时间复杂度|
|-----|-----|
|冒泡排序|O(n2)|
|选择排序|	O(n2)|
|插入排序|	O(n2)|
|希尔排序|	O(n1.5)|
|快速排序|	O(N*logN)|
|归并排序|	O(N*logN)|
|堆排序|	O(N*logN)|
|基数排序|	O(d(n+r))|

# 冒泡排序
## 思路
两个数字比大小，大的往下，小的往上
## 过程
1. 比较相邻的两个数据，如果第二个数小，就交换位置。
2. 从后向前两两比较，一直到比较最前两个数据。最终最小数被交换到起始的位置，这样第一个最小数的位置就排好了
3. 继续重复上述过程，依次将第2.3...n-1个最小数排好位置。

## java代码实现
```
public static void BubbleSort(int [] arr){
        
        int temp;//临时变量
        for(int i=0; i<arr.length-1; i++){   //表示趟数，一共arr.length-1次。
            for(int j=arr.length-1; j>i; j--){
                
                if(arr[j] < arr[j-1]){
                    temp = arr[j];
                    arr[j] = arr[j-1];
                    arr[j-1] = temp;
                }
            }
        }
    }
```
## 优化
> 问题：数据的顺序排好之后，冒泡算法仍然会继续进行下一轮的比较，直到arr.length-1次，后面的比较没有意义的。
解决
设置标志位flag，如果发生了交换flag设置为true；如果没有交换就设置为false。
这样当一轮比较结束后如果flag仍为false，即：这一轮没有发生交换，说明数据的顺序已经排好，没有必要继续进行下去
```
public static void BubbleSort1(int [] arr){
        
        int temp;//临时变量
        boolean flag;//是否交换的标志
        for(int i=0; i<arr.length-1; i++){   //表示趟数，一共arr.length-1次。
            
            flag = false;
            for(int j=arr.length-1; j>i; j--){
                
                if(arr[j] < arr[j-1]){
                    temp = arr[j];
                    arr[j] = arr[j-1];
                    arr[j-1] = temp;
                    flag = true;
                }
            }
            if(!flag) break;
        }
    }
```
# 选择排序
## 思路
在长度为n的无需数组中，第一次遍历n-1个数字，找到里面最小的那个，然后和第一个进行交换，第二次遍历n-2个数字，找到最小额元素和第二个进行交换。。。第n-1次遍历找到最小的数值与第n-1个元素进行交换，排序完成
## 过程
![sort01](/assets/sort/sort01.png)
## 代码实现
```
public static void select_sort(int array[],int lenth){
      
      for(int i=0;i<lenth-1;i++){
          
          int minIndex = i;
          for(int j=i+1;j<lenth;j++){
             if(array[j]<array[minIndex]){
                 minIndex = j;
             }
          }
          if(minIndex != i){
              int temp = array[i];
              array[i] = array[minIndex];
              array[minIndex] = temp;
          }
      }
  }
```
# 插入排序
## 思路
在一组需要排序的数组中，假设n-1个数字已经排好了，现在将第n个数字插入到前面的有序数列中，使得这个n个数字也是排好的顺序，循环往复
## 过程
![sort02](/assets/sort/sort02.png)
## 代码实现
```
public static void  insert_sort(int array[],int lenth){
      
      int temp;
      
      for(int i=0;i<lenth-1;i++){
          for(int j=i+1;j>0;j--){
              if(array[j] < array[j-1]){
                  temp = array[j-1];
                  array[j-1] = array[j];
                  array[j] = temp;
              }else{         //不需要交换
                  break;
              }
          }
      }
  }
```
# 希尔排序
## 思路
在要排序的一组数中，根据某一增量分为若干子序列，并对子序列分别进行插入排序。
然后逐渐将增量减小,并重复上述过程。直至增量为1,此时数据序列基本有序,最后进行插入排序
## 过程
![sort03](/assets/sort/sort03.png)
## 代码实现
```
public static void shell_sort(int array[],int lenth){
      
      int temp = 0;
      int incre = lenth;
      
      while(true){
          incre = incre/2;
          
          for(int k = 0;k<incre;k++){    //根据增量分为若干子序列
              
              for(int i=k+incre;i<lenth;i+=incre){
                  
                  for(int j=i;j>k;j-=incre){
                      if(array[j]<array[j-incre]){
                          temp = array[j-incre];
                          array[j-incre] = array[j];
                          array[j] = temp;
                      }else{
                          break;
                      }
                  }
              }
          }
          
          if(incre == 1){
              break;
          }
      }
  }
```
# 快速排序
## 思路
1. 先从数列中取出一个数作为key值；
2. 将比这个数小的数全部放在它的左边，大于或等于它的数全部放在它的右边；
3. 对左右两个小数列重复第二步，直至各区间只有1个数。
## 理解
[挖坑填数](http://blog.csdn.net/morewindows/article/details/6684558)
初始时 i = 0; j = 9; key=72
由于已经将a[0]中的数保存到key中，可以理解成在数组a[0]上挖了个坑，可以将其它数据填充到这来。
从j开始向前找一个比key小的数。当j=8，符合条件，a[0] = a[8] ; i++ ; 将a[8]挖出再填到上一个坑a[0]中。
这样一个坑a[0]就被搞定了，但又形成了一个新坑a[8]，这怎么办了？简单，再找数字来填a[8]这个坑。
这次从i开始向后找一个大于key的数，当i=3，符合条件，a[8] = a[3] ; j-- ; 将a[3]挖出再填到上一个坑中。
```
数组：72 - 6 - 57 - 88 - 60 - 42 - 83 - 73 - 48 - 85
      0   1   2    3    4    5    6    7    8    9
```
此时 i = 3; j = 7; key=72
再重复上面的步骤，先从后向前找，再从前向后找。
从j开始向前找，当j=5，符合条件，将a[5]挖出填到上一个坑中，a[3] = a[5]; i++;
从i开始向后找，当i=5时，由于i==j退出。
此时，i = j = 5，而a[5]刚好又是上次挖的坑，因此将key填入a[5]。
```
数组：48 - 6 - 57 - 88 - 60 - 42 - 83 - 73 - 88 - 85
      0   1   2    3    4    5    6    7    8    9
```
可以看出a[5]前面的数字都小于它，a[5]后面的数字都大于它。因此再对a[0…4]和a[6…9]这二个子区间重复上述步骤就可以了
```
数组：48 - 6 - 57 - 42 - 60 - 72 - 83 - 73 - 88 - 85
      0   1   2    3    4    5    6    7    8    9
```
key值的选取可以有多种形式，例如中间数或者随机数，分别会对算法的复杂度产生不同的影响。

##  归并排序
## 思路

[归并排序](http://blog.csdn.net/morewindows/article/details/6678165")
归并排序是建立在归并操作上的一种有效的排序算法。该算法是采用分治法的一个非常典型的应用。
首先考虑下如何将2个有序数列合并。这个非常简单，只要从比较2个数列的第一个数，谁小就先取谁，取了后就在对应数列中删除这个数。然后再进行比较，如果有数列为空，那直接将另一个数列的数据依次取出即可。
```
//将有序数组a[]和b[]合并到c[]中
void MemeryArray(int a[], int n, int b[], int m, int c[])
{
    int i, j, k;

    i = j = k = 0;
    while (i < n && j < m)
    {
        if (a[i] < b[j])
            c[k++] = a[i++];
        else
            c[k++] = b[j++]; 
    }

    while (i < n)
        c[k++] = a[i++];

    while (j < m)
        c[k++] = b[j++];
}
```
解决了上面的合并有序数列问题，再来看归并排序，其的基本思路就是将数组分成2组A，B，如果这2组组内的数据都是有序的，那么就可以很方便的将这2组数据进行排序。如何让这2组组内数据有序了？
可以将A，B组各自再分成2组。依次类推，当分出来的小组只有1个数据时，可以认为这个小组组内已经达到了有序，然后再合并相邻的2个小组就可以了。这样通过先递归的分解数列，再合并数列就完成了归并排序

## 代码实现
```
public static void merge_sort(int a[],int first,int last,int temp[]){
     
     if(first < last){
         int middle = (first + last)/2;
         merge_sort(a,first,middle,temp);//左半部分排好序
         merge_sort(a,middle+1,last,temp);//右半部分排好序
         mergeArray(a,first,middle,last,temp); //合并左右部分
     }
 }
```
```
//合并 ：将两个序列a[first-middle],a[middle+1-end]合并
 public static void mergeArray(int a[],int first,int middle,int end,int temp[]){     
     int i = first;
     int m = middle;
     int j = middle+1;
     int n = end;
     int k = 0; 
     while(i<=m && j<=n){
         if(a[i] <= a[j]){
             temp[k] = a[i];
             k++;
             i++;
         }else{
             temp[k] = a[j];
             k++;
             j++;
         }
     }   
     while(i<=m){
         temp[k] = a[i];
         k++;
         i++;
     }   
     while(j<=n){
         temp[k] = a[j];
         k++;
         j++; 
     }
     
     for(int ii=0;ii<k;ii++){
         a[first + ii] = temp[ii];
     }
 }
```
> 剩下的两个真的还没有看懂，先这么着吧

# 参考资料
[排序算法总结](https://www.jianshu.com/p/ae97c3ceea8d)
