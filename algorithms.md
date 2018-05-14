> *   __*交换运算切勿使用*__ `[arr[i], arr[j]] = [arr[j], arr[i]]` __*会消耗大量时间去运算*__
> *   __*整数转换时，使用*__ `Math.floor` __*比之*__ `Number.parseInt` __*要快很多*__

---
### 快速排序
> *   时间复杂度O(nlogn)~O(n^2), 平均为O(nlogn), 空间复杂度O(1), 不稳定
> *   双向进行比较, 运行时间更少

	function quickSort(arr, sub = 0, sup = arr.length - 1) {
		const pivot = arr[sub]
	    let left = sub, right = sup
	    if (sub < sup) {
	        while (left < right) {
	            while (arr[right] > pivot && left < right) {
	                right--
	            }
	            arr[left] = arr[right]
	            while (arr[left] < pivot && left < right) {
	                left++
	            }
	            arr[right] = arr[left]
	        }
	        arr[left] = pivot
	        if (sub < left - 1) {
	            quickSort(arr, sub, left - 1)
	        }
	        if (left < sup) {
	            quickSort(arr, left + 1 , sup)
	        }
	    }
	}

---
### 堆排序
> *   时间复杂度O(nlogn) 不稳定
> *   据说在大量数据时运行时间小于快排, 但在浏览器崩溃边缘测试依旧远远慢于快排

	/**
     * 按max heap调整堆
     * @params arr 传入数组
     * @params heapSize 当前堆的大小
     * @params index 需要调整的节点下标
     */
    function mutateHeap(arr, heapSize, index) {
        while (index < heapSize) {
            let left = 2 * index + 1, right = 2 * index + 2, largest = index
            if (left < heapSize && arr[index] < arr[left]) {
                largest = left
            }
            if (right < heapSize && arr[largest] < arr[right]) {
                largest = right
            }
            if (largest == index) {
                break
            }
            let temp = arr[index]
            arr[index] = arr[largest]
            arr[largest] = temp
            index = largest
        }
    }

    /**
     * 先初始化堆, 然后调整
     * @params arr 传入数组
     */
    function heapSort(arr) {
        // 初始化堆, originIndex为最后一个父节点
        const originIndex = Math.floor((arr.length - 1) / 2)
        for (let i = originIndex; i >= 0; i--) {
            mutateHeap(arr, arr.length, i)
        }
        // 调整堆
        for (let i = arr.length - 1; i >= 0; i--) {
            let temp = arr[0]
            arr[0] = arr[i]
            arr[i] = temp
            mutateHeap(arr, i, 0)
        }
    }

---
### 归并排序

---
[参考](https://www.cnblogs.com/yu-chao/p/4324485.html)
