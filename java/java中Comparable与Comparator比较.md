###java常用的排序算法

参考文章：[http://www.importnew.com/16266.html](http://www.importnew.com/16266.html)

分类：

- 插入排序（直接插入排序、希尔排序）
- 交换排序（冒泡排序、快速排序）
- 选择排序（直接选择排序、堆排序）
- 归并排序
- 分配排序（技术排序）
- 所需辅助空间最多：归并排序
- 所需辅助空间最少：堆排序
- 平均速度最快：快速排序
- 不稳定：快速排序，希尔排序，堆排序

![8种排序之间的关系](http://img.my.csdn.net/uploads/201209/07/1347008904_9606.jpg)

> 直接插入排序

基本思想：在要排序的一组数中，假设前面(n-1)(n>=2)个数已经是排号顺序，现在要把第n个数插入到前面的有序数中，使得这n个数也是排好顺序的，如此反复循环，直到全部排好顺序。

![图示](http://ww2.sinaimg.cn/mw690/b254dc71gw1eu26znoqp3j20f5066wew.jpg)

```
代码实现

public insertSort(){
	int a[] = {2,4,1,6,34,89,45,67};
	int temp = 0;
	for(int i=1;i<a.length;i++){
		int j = i-1;
		temp = a[i];
		for(;j>=0&&temp<a[j];j--){
			a[j+1] = a[j];
		}
		a[j+1] = temp;
	}
	
}
```

> 希尔排序

