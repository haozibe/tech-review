### 排序算法

1. 快排

   ~~~java
   public void quickSort(int[] n){
     sort(n,0,n.length-1);
   }
   
   public void sort(int[] n,int l,int r){
     if(l<r){
       int index = patition(n,l,r);
       sort(n,l,index-1);
       sort(n,index+1,r);
     }
   }
   
   public int patition(int[] n,int l,int r){
     int p = n[l];
     int i = l;
     int j = r;
     while(i<j){
       while(i<j && n[j] >= p){
         j--;
       }
       while(i<j && n[i] <= p){
         i++;
       }
       swap(n,i,j);
     }
     swap(n,l,i);
     return i;
   }
   
   public void swap(int[] n,int i,int j){
     int temp = n[i];
     n[i] = n[j];
     n[j] = temp;
   }
   ~~~

2. 插入排序

   ~~~java
   public  void insertionSort(int[] arr){
   	for(int i=0;i<arr.length;i++){
       int temp = arr[i];
       for(int j=i;j>0;j--){
         if(j>0&&temp<arr[j-1]){
           arr[j] = arr[j-1];
         }else{
           arr[j] = temp;
           break;
         }
       }
     }
   }
   ~~~

3. 选择排序

   ~~~java
   public void selectionSort(int[] arr){
     for(int i=0;i<arr.length;i++){
       int min = i;
       for(int j=i+1;j<arr.length;j++){
         if(arr[min] > arr[j]){
           min = j;
         }
       }
       if(min != i){
           int temp = arr[min];
           arr[min] = arr[i];
           arr[i] = temp;
       }
     }
   }
   ~~~

4. 希尔排序

   ~~~java
   public void shellSort(int[] n ){
     int gap = 1;
     int len = n.length;
     while(gap<len/3){
       gap = gap*3+1;
     }
     for(;gap>0;gap/=3){
       for(int i =gap;i<len;i++){
         temp = n[i];
         for(int j=i-gap;j>=0&&arr[j]>temp;j-=gap)
           arr[j+gap]=arr[j];
         arr[j+gap]=temp;
       }
     }
   }
   ~~~
   
5. 冒泡排序

   ~~~java
   public void bubbleSort(int[] arr){
     for(int i=arr.length;i>=0;i--){
       for(int j=0;j+1<i;j++){
         if(arr[j+1]<arr[j]){
           int temp = arr[j+1];
           arr[j+1] = arr[j];
           arr[j] = temp;
         }
       }
     }
   }
   ~~~

6. 堆排序

   ~~~java
   public void heapSort(int[] n){
     //从第一个非叶子节点开始，构建大顶堆
   	for(int i = n.length/2-1;i>0;i--){
       adjustHeap(n,i,length);
     }
     //2.调整堆结构+交换堆顶元素与末尾元素
    for(int j = n.length-1;j>0;j--){
      swap(n,0,j);
      adjustHeap(n,0,j);
    }
   }
   
   public void adjustHeap(int[] n, int i,int length){
     int temp = n[i];
     for(int k=i*2+1;k<length;k++){
       if(k+1<length&&n[k+1]>n[k] ){
         k++;
       }else if(arr[k] > temp){
         arr[i] = arr[k];
         i = k;
       }else{
         break;
       }
     }
     arr[i] = temp; 
   }
   
   ~~~

7. 归并排序

   ~~~java
   public void mergeSort(int[] n){
     int[] temp = new int[arr.length]
     sort(n,0,n,length-1);
   }
   public void sort(int[] n,l,r){
     if(l < r){
       mid = (l+r)>>>1;
       sort(arr,left,mid,temp);
       sort(arr,mid+1,right,temp);
       merge(arr,left,mid,right,temp);
     }
   }
   public static void merge(int[] arr,int l ,int m ,int r,temp){
     int i = l;
     int j = mid+1;
     int t = 0;
     while(i<mid&&j<right){
       if(arr[i]<=arr[j]){
         temp[t++] = arr[i++]
       }else{
         temp[t++] = arr[j++]
       }
       while(i<mid){
         temp[t++] = arr[i++];
       }
       while(j<right){
         temp[i++] = arr[j++];
       }
       t=0;
       while(l<r){
         arr[l++] = temp[t++];
       }
     }
   }
   ~~~

8. 

### 手写快排

~~~java
public void quickSort(int [] a){
  sort(a,0,a.length-1);
}
public void sort(int[] a,int l,int r){
  if(min < max){
    int index = partition(a,l,r);
    sort(a,l,index-1);
    sort(a,index+1,r);
  }
}

public int partition(int[] a,int l,int r){
  int temp = a[l];
  int j = l;
  int k = r;
  while(j < k){
    while(j < k && a[k] >= temp){
       k--;
    }
    while(j < k && a[j] <= temp){
      j++
    }
    swap(a,j,k)
  }
  swap(a,l,j)；
  return j;
}
~~~



### 二叉查找

~~~java
public int binarySearch(int[] a,int b){
  int max = a.length-1;
  int min = 0;
  int mid;
  while(min < max){
    mid = min + ((max - min) >>> 1);
		if(a[mid] < b){
      min = min + 1;
    }else if(a[mid] > b){
      max = max -1;
    }else{
			return mid;
    }
  }
  return -1;
}
~~~



