**冒泡排序的改进**

1. 加一个标志位，当某一趟冒泡排序没有元素交换时，则冒泡结束，元素已经有序，可以有效的减少冒泡次数。

```java
/** 
  * 引入标志位，默认为true 
  * 如果前后数据进行了交换，则为true，否则为false。如果没有数据交换，则排序完成。 
  */  
public static int[] bubbleSort(int[] arr){  
        boolean flag = true;  
        int n = arr.length;  
        while(flag){  
            flag = false;  
            for(int j=0;j<n-1;j++){  
                if(arr[j] >arr[j+1]){  
                    //数据交换  
                    int temp = arr[j];  
                    arr[j] = arr[j+1];  
                    arr[j+1] = temp;  
                    //设置标志位  
                    flag = true;  
                }  
            }  
            n--;  
        }  
        return arr;  
}
```

    2. 记录每一次元素交换的位置，当元素交换的位置在第0个元素时，则排序结束



