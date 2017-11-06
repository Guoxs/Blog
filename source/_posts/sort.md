---
title: 排序算法总结
date: 2016-09-03 20:16:43
tags: 算法与数据结构
---

参考文章：http://blog.csdn.net/amazing7/article/details/51603682

排序算法总结：
![对比分析图][1]
<!--more-->

## 冒泡排序
　重复走访要排序的数列，一次比较两个元素，如果顺序错误则交换，直到没有再需要交换的元素为止。每一轮都有一个最小的数字在正确的位置上。
```java
public static void bubble_sort(int[] arr){
	Boolean flag = true;
	int len = arr.length;
	while(flag){
		flag = false;
		for (int i = 0; i < len - 1; i++) {
			for (int j = 0; j < len - i - 1; j++) {
				if(arr[j] > arr[j+1]){
					swap(arr, j, j+1);
					flag = true;
				}
			}
		}
		len--;
	}		
}
```
## 选择排序

### 简单选择排序
所排序序列的记录个数为n，i取1,2,…,n-1。从所有n-i+1个记录（Ri,Ri+1,…,Rn）中找出排序码最小（或最大）的记录，与第i个记录交换。执行n-1趟后就完成了记录序列的排序。
```java
public static void select_sort(int[] arr){
	int len = arr.length;
	for (int i = 0; i < len; i++) {
		int min = i;
		for (int j = i+1; j < len; j++) {
			if(arr[j] < arr[min]){
				min = j;
			}
		}
		if(min != i){
			swap(arr,min,i);
		}
	}
}
```
在简单选择排序过程中，所需移动记录的次数比较少。最好情况下，即待排序记录初始状态就已经是**正序**排列了，则不需要移动记录。

最坏情况下，即待排序记录初始状态是按**第一条记录最大，之后的记录从小到大顺序排列**，则需要移动记录的次数最多为3（n-1）。

简单选择排序过程中需要进行的**比较次数**与初始状态下待排序的记录序列的排列情况无关。

当i=1时，需进行n-1次比较；当i=2时，需进行n-2次比较；依次类推，共需要进行的比较次数是 n(n-1)/2，即进行比较操作的时间复杂度为O(n^2)，进行移动操作的时间复杂度为O(n)。　

简单选择排序是不稳定排序。
### 堆排序
堆排序是指利用**堆**这种数据结构所设计的一种排序算法。可以利用数组的特点快速定位指定索引的元素。堆分为大根堆和小根堆，是完全二叉树。大根堆的要求是每个节点的值都不大于其父节点的值。
堆每次只能取得根节点元素，以大顶堆为例，第一次0号为最大的元素，取出0号元素，最后一个元素补在0号位置上，重建大顶堆。
```java
public class HeapSort {
	public static void main(String[] args) {
		int[] number = {95,45,15,78,-84,51,24,12,6,0,-1,88,4,145};
		buildMaxHeapify(number);
		heap_sort(number);
		print(number);
	}
	private static void buildMaxHeapify(int[] data){
		int startIndex = getParentIndex(data.length -1);
		for(int i = startIndex; i >= 0; i--){
			maxHeapify(data,data.length,i);
		}
	}
    /**
     * 排序，最大值放在末尾，data虽然是最大堆，在排序后就成了递增的
     * @param data
     */
    private static void heap_sort(int[] data) {
        //末尾与头交换，交换后调整最大堆
        for (int i = data.length - 1; i > 0; i--) {
            int temp = data[0];
            data[0] = data[i];
            data[i] = temp;
            maxHeapify(data, i, 0);
        }
    }	
	
	/**
     * 创建最大堆
     * @param data
     * @param heapSize需要创建最大堆的大小
     * @param index当前需要创建最大堆的位置
     */
    private static void maxHeapify(int[] data, int heapSize, int index){
        // 当前点与左右子节点比较
        int left = getChildLeftIndex(index);
        int right = getChildRightIndex(index);

        int largest = index;
        if (left < heapSize && data[index] < data[left]) {
            largest = left;
        }
        if (right < heapSize && data[largest] < data[right]) {
            largest = right;
        }
        //得到最大值后可能需要交换，如果交换了，其子节点可能就不是最大堆了，需要重新调整
        if (largest != index) {
            int temp = data[index];
            data[index] = data[largest];
            data[largest] = temp;
            maxHeapify(data, heapSize, largest);
        }
    }
	
	/**
     * 父节点位置
     * @param current
     * @return
     */
    private static int getParentIndex(int current){
        return (current - 1) >> 1;
    }
    
    /**
     * 左子节点position注意括号，加法优先级更高
     * @param current
     * @return
     */
    private static int getChildLeftIndex(int current){
        return (current << 1) + 1;
    }

    /**
     * 右子节点position
     * @param current
     * @return
     */
    private static int getChildRightIndex(int current){
        return (current << 1) + 2;
    }
}
```

## 希尔排序
希尔排序法(缩小增量法) 属于插入类排序，是将整个无序列分割成若干小的子序列分别进行插入排序的方法。
把记录按下标的一定增量分组，对每组使用直接插入排序算法排序；随着增量逐渐减少，每组包含的关键词越来越多，当增量减至1时，整个文件恰被分成一组，算法便终止。
```java
public static void shell_sort(int[] arr){
	int gap = 1;	//记录步长
	int len = arr.length;
	int temp;		//插入排序交换值的暂存
	
	while(gap < len / 3){
		gap = gap * 3 + 1;
	}
	
	//循环遍历步长，最后必为1
	for(; gap > 0; gap /= 3){
		int i,j;
		for (i = gap; i < len; i++) {
			temp = arr[i];
			//每一列中在a[i]上面且比a[i]大的元素依次向下移动
			for (j = i - gap; j >= 0 && arr[j] > temp; j -= gap) {
				arr[j + gap] = arr[j];
			}
			arr[j + gap] = temp;
		}
	}		
}
```
希尔排序是一个不稳定的排序，其时间复杂度受**步长**（增量）的影响。

## 归并排序
归并排序采用分治法，各层递归可以同时进行。
归并排序速度**仅次于快速排序**，为稳定排序算法，一般用于对总体无序，但是各子项相对有序的数列。归并排序比较占用内存。

### 归并排序的非递归算法
① 申请空间，使其大小为两个已经排序序列之和，该空间用来存放合并后的序列
② 设定两个指针，最初位置分别为两个已经排序序列的起始位置
③ 比较两个指针所指向的元素，选择相对小的元素放入到合并空间，并移动指针到下一位置
④ 重复步骤③直到某一指针到达序列尾
⑤ 将另一序列剩下的所有元素直接复制到合并序列尾
```java
public static void merge_sort_iterator(int[] arr){
	int len = arr.length;
	
	int[] result = new int[len];
	int block,start;
	
	//两两合并后块大小变大两倍 (注意最后一次block等于len)
	for(block = 1; block <= len; block *= 2){
		 //把整个数组分成很多个块，每次合并处理两个块
		for(start = 0; start < len; start += 2*block){
			int low = start;
			int mid = (start + block) < len ? (start + block) : len;
			int high = (start + 2*block) < len ? (start + 2*block) : len;
			 //两个块的起始下标及结束下标
            int start1 = low, end1 = mid;
            int start2 = mid, end2 = high;
            //开始对两个block进行归并排序
            while (start1 < end1 && start2 < end2) {
                result[low++] = arr[start1] < arr[start2] ? arr[start1++] : arr[start2++];
            }
            while(start1 < end1) {
                result[low++] = arr[start1++];
            }
            while(start2 < end2) {
                result[low++] = arr[start2++];
            }       
		}
		//每次归并后把结果result存入arr中，以便进行下次归并
        int[] temp = arr;
        arr = result;
        result = temp;
	}
}
```
### 归并排序的递归实现

```java
public static void merge_sort_recursive(int[] arr){
	int len = arr.length;
	int[] reg = new int[len];
	binary_sort(arr,reg,0,len-1);
}

public static void binary_sort(int[] arr, int[] reg, int start, int end){
	if(start >= end){
		return;
	}
	int len = end - start, mid = (len >> 1) + start;
	int start1 = start, end1 = mid;
	int start2 = mid + 1, end2 = end;
	//递归到子序列只有一个数的时候，开始逐个返回
	binary_sort(arr, reg, start1, end1);
	binary_sort(arr, reg, start2, end2);
	//-------合并操作，必须在递归之后（子序列有序的基础上）-------
	int k = start;
	while (start1 <= end1 && start2 <= end2) {
		reg[k++] = arr[start1] < arr[start2] ? arr[start1++] : arr[start2++];
    }
    while(start1 <= end1) {
    	reg[k++] = arr[start1++];
    }
    while(start2 <= end2) {
    	reg[k++] = arr[start2++];
    }
    //借用reg数组做合并，然后把数据存回arr中
    for (k = start; k <= end; k++){
    	arr[k] = reg[k];
    }            
}
```
## 快速排序
快速排序是对冒泡排序的一种改进，又称划分交换排序。快速排序使用分治法策略来把一个序列（list）分为两个子序列（sub-lists）。

基本步骤为：
① 从数列中挑出一个元素，称为**基准**（pivot）
② 重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面（相同的数可以到任一边）。在这个分区结束之后，该基准就处于数列的中间位置。这个称为**分区**（partition）操作。
③ 递归地把小于基准值元素的子数列和大于基准值元素的子数列排序

>在平均状况下，排序n个项目要**Ο(nlogn)**次比较。在最坏状况下则需要Ο(n^2)次比较，但这种状况并不常见。事实上，快速排序通常明显比其他Ο(nlogn)算法更快，因为它的**内部循环（inner loop）可以在大部分的架构上很有效率地被实现出来**。

```java
public static void quick_sort(int[] arr, int low, int high){
	if(low < high){
		int mid = getMiddle(arr,low,high);
		quick_sort(arr, low, mid - 1);
		quick_sort(arr, mid + 1, high);
	}
}

public static int getMiddle(int[] arr, int low, int high){
	int temp = arr[low];
	while(low < high){
		while(low < high && arr[high] >= temp){
			high --;
		}
		arr[low] = arr[high];
		while(low < high && arr[low] <= temp){
			low ++;
		}
		arr[high] = arr[low];
	}
	arr[low] = temp;
	return low;
}
```
## 桶排序
输入是由一个随机过程产生的[0,1)区间上均匀分布的实数。将区间[0,1)划分为n个大小相等的子区间（桶），每桶大小1/n：[0, 1/n)，[1/n,2/n)，[2/n,3/n)，…，[k/n,(k+1)/n)，…将n个输入元素分配到这些桶中，对桶中元素进行排序，然后依次连接桶输入0 ≤A[1..n]< 1辅助数组B[0..n-1]是一指针数组，指向桶（链表）。

![桶排序][2]
平均情况下桶排序以线性时间运行，桶排序是稳定的，排序非常快,但是同时也非常耗空间,基本上是**最耗空间**的一种排序算法。

提高桶排序的效率：
>① 映射函数f(k)能够将N个数据平均的分配到M个桶中，这样每个桶就有[N/M]个数据量。　 
>②尽量的增大桶的数量。极限情况下每个桶只能得到一个数据，这样就完全避开了桶内数据的“比较”排序操作。 当然，做到这一点很不容易，数据量巨大的情况下，f(k)函数会使得桶集合的数量巨大，空间浪费严重。这就是一个时间代价和空间代价的权衡问题了。

```java
/** 
 * 对arr进行桶排序，排序结果仍放在arr中 
 */  
@SuppressWarnings({ "unchecked", "rawtypes" })
public static void bucketSort(double arr[]){  
	//分桶     
    int n = arr.length;  
    //存放桶的链表
	ArrayList bucketList[] = new ArrayList [n];

    //每个桶是一个list，存放此桶的元素   
    for(int i = 0; i < n; i++){  
        //下取等
        int temp = (int) Math.floor(n*arr[i]);  
        //若不存在该桶，就新建一个桶并加入到桶链表中
        if(bucketList[temp] == null)  
            bucketList[temp] = new ArrayList();  
        //把当前元素加入到对应桶中
        bucketList[temp].add(arr[i]);            
    }  
    //桶内排序  
    //对每个桶中的数进行插入排序   
    for(int i = 0; i < n; i++){  
        if(bucketList[i] != null)  
            insert(bucketList[i]);  
    }  
    //合并桶内数据
    //把各个桶的排序结果合并   
    int count = 0; 
    for(int i = 0; i < n; i++){  
        if(bucketList[i] != null){  
            Iterator iter = bucketList[i].iterator();  
            while(iter.hasNext()){  
                Double d = (Double)iter.next();  
                arr[count] = d;  
                count++;  
            }  
        }  
    }  
}  

/** 
 * 用插入排序对每个桶进行排序 
 * 从小到大排序
 */  
@SuppressWarnings({ "rawtypes", "unchecked"})
public static void insert(ArrayList list){  
    if(list.size() > 1){  
        for(int i = 1; i < list.size(); i++){  
            if((Double)list.get(i) < (Double)list.get(i-1)){  
                double temp = (Double) list.get(i);  
                int j = i-1;  
                for(; j >= 0 && ((Double)list.get(j) > (Double)list.get(j+1)); j--){
                	list.set(j+1, list.get(j));  //后移
                }                        
                list.set(j+1, temp);  
            }  
        }  
    } 
}
```
## 基数排序
基数排序（Radix sort）是一种非比较型整数排序算法，其原理是将整数按位数切割成不同的数字，然后按每个位数分别比较。由于整数也可以表达字符串（比如名字或日期）和特定格式的浮点数，所以基数排序也不是只能使用于整数。

将所有待比较数值（正整数）统一为同样的数位长度，数位较短的数前面补零。然后，从最低位开始，依次进行一次排序。这样从最低位排序一直到最高位排序完成以后，数列就变成一个有序序列。

```java
public static void radix_sort(int[] arr){
	//确定排序趟数
	int max = arr[0];
	for (int i = 1; i < arr.length; i++) {
		if(arr[i] > max){
			max = arr[i];
		}
	}
	int time = 0;
	while(max > 0){
		max /= 10;
		time ++;
	}
	
	//初始化10个链表
	List<List<Integer>> list = new ArrayList<>();
	for (int i = 0; i < 10; i++) {
		List<Integer> item = new ArrayList<>();
		list.add(item);
	}
	
	//进行time次分配与收集
	for (int i = 0; i < time; i++) {
		//分配元素
		for (int j = 0; j < arr.length; j++) {
			int index = arr[j] % (int)Math.pow(10, i+1) / (int)Math.pow(10, i);
			list.get(index).add(arr[j]);
		}
		//收集元素
		int count = 0;
		for (int k = 0; k < 10; k++) {
			if(list.get(k).size() > 0){
				for (int a : list.get(k)) {
					arr[count] = a;
					count ++;
				}
				//清除数据，以便下次收集
				list.get(k).clear();
			}
		}
	}
}
```
## 插入排序
将一个数据插入到已经排好序的有序数据中，从而得到一个新的、个数加一的有序数据，算法适用于少量数据的排序，是稳定的排序方法。
插入排序又分为　**直接插入排序**　和 **折半插入排序**。
### 直接插入排序
把待排序的纪录按其关键码值的大小逐个插入到一个已经排好序的有序序列中，直到所有的纪录插入完为止，得到一个新的有序序列。
```java
public static void simple_insert_sort(int[] arr){
	int i;		//当前要插入的位置
	int pre;	//j前一个位置
	int key;	//后移时来暂存要插入的值
	int len = arr.length;
	
	for (i = 1; i < len; i++) {
		key = arr[i];
		pre = i - 1;
		while(pre >= 0 && arr[pre] > key){
			//留出来一个空白位置来实现依次后移（不会造成数据丢失问题）
			arr[pre+1] = arr[pre];
			pre--;
		}
		arr[pre+1] = key;
	}
}
```
>平均时间复杂度O(n^2)
>最差情况：反序，需要移动n*(n-1)/2个元素 ，运行时间为O(n^2)。 
>最好情况：正序，不需要移动元素，运行时间为O(n)

### 折半插入排序
折半插入排序,使用使用**折半查找**的方式寻找插入点的位置, 可以减少比较的次数,但移动的次数不变, 时间复杂度和空间复杂度和直接插入排序一样，在元素较多的情况下能提高查找性能。
```java
public static void binary_insert_sort(int[] arr){
	int len = arr.length;
	for (int i = 1; i < len; i++) {
		int key = arr[i];		//暂存要插入的值
		int pre = 0;
		int last = i - 1;
		while(pre <= last){
			int mid = (pre + last)/2;
			if(key < arr[mid]){
				last = mid -1;
			}
			else{
				pre = mid + 1;
			}
		}
		//arr[i]已经取出来存放在key中，把下标从pre + 1到 i-1的元素依次后移
		for (int j = i; j >= pre + 1; j--) {
			arr[j] = arr[j-1];
		}
		arr[pre] = key;
	}
}
```
## 计数排序
**计数排序的原理**：设被排序的数组为A,排序后存储到B，C为临时数组。所谓计数，首先是通过一个数组C[i]计算大小等于i的元素个数，此过程只需要一次循环遍历就可以；在此基础上，计算小于或者等于i的元素个数，也是一重循环就完成。下一步是关键：逆序循环，从length[A]到1，将A[i]放到B中第C[A[i]]个位置上。原理是：C[A[i]]表示小于等于A[i]的元素个数，正好是A[i]排序后应该在的位置。而且从length[A]到1逆序循环，可以**保证相同元素间的相对顺序不变**，这也是计数排序稳定性的体现。在数组A有附件属性的时候，稳定性是非常重要的。

**计数排序的前提及适用范围**: A中的元素不能大于k，而且元素要作为数组的下标，所以元素应该为非负整数。而且如果A中有很大的元素，不能够分配足够大的空间。所以计数排序有很大局限性，其主要适用于元素个数多，但是普遍不太大而且总小于k的情况，这种情况下使用计数排序可以获得很高的效率。

![计数排序][3]

```java
public static int[] counting_sort(int[] arr){
	//建立C数组
	int max = 0;
	for (int i : arr) {
		if(i > max){
			max = i;
		}
	}
	int[] C = new int[max+1];
	
	//建立B数组
	int length = arr.length;
	int[] B = new int[length];
	
	//统计A中各元素个数，存入C数组
	for (int i = 0; i < length; i++) {
		C[arr[i]] ++;
	}

	//修改C数组
	int sum = 0;
	for (int i = 0; i < max+1; i++) {
		sum += C[i];
		C[i] = sum;
	}
	
	//遍历A数组，构造B数组
	for(int i = length - 1; i >= 0; i--){
		B[C[arr[i]] - 1] = arr[i];
		C[arr[i]]--;
	} 
	return B;
}
```


[1]: http://static.zybuluo.com/guoxs/f2hpk3xr8b8rz6zlbw4uk9l6/20160607144411150
[2]: http://static.zybuluo.com/guoxs/777ogd5g1fuhisrskgl2u2u6/1.jpg
[3]: http://static.zybuluo.com/guoxs/zh7jpkqigmlxm2eby46ve8vj/2.png