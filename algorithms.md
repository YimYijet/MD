> *   __*交换运算切勿使用*__ `[arr[i], arr[j]] = [arr[j], arr[i]]` __*会消耗大量时间去运算*__
> *   __*整数转换时，使用*__ `Math.floor` __*比之*__ `Number.parseInt` __*要快很多*__
> *   __*数组替换，使用*__ `Array.prototype.splice` __*比*__ `for` __*循环快*__
	
    function random(min, max) {
        return Math.floor(Math.random() * (max - min + 1)) + min
    }
    // 生成len长度的随机数组
    function generateArr(len) {
		let arr = []
        for (var i = 0; i < len; i++) {
            arr.push(random(1, len))
        }
		return arr
    }

---
### 快速排序
> *   时间复杂度O(nlogn)~O(n^2), 平均为O(nlogn), 空间复杂度O(1), 不稳定
> *   双向进行比较, 运行时间更少

	function quickSort(arr, sub = 0, sup = arr.length - 1) {
		const pivot = arr[sub]
	    let left = sub, right = sup
	    if (sub < sup) {
	        while (left < right) {
	            while (arr[right] >= pivot && left < right) {
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
	        if (left + 1 < sup) {
	            quickSort(arr, left + 1 , sup)
	        }
	    }
	}

---
### 堆排序
> *   时间复杂度O(nlogn), 不稳定
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
> *   时间复杂度O(nlogn), 稳定
> *   归并分为两种实现, 递归和非递归, 递归实现在数据非常大的时候, 容易栈溢出, 无法递归
	 
	/** 
     * 递归实现归并排序
     */        
    function mergeSort(arr, left = 0, right = arr.length - 1, temp = []) {
        let middle = Math.floor((right + left) / 2 )
        if (left < right) {
            if (left < middle) {
                mergeSort(arr, left, middle, temp);
            }
            if (middle + 1 < right) {
                mergeSort(arr, middle + 1, right, temp);
            }
            merge(arr, left, right, middle, temp);
        }
    }

    function merge(arr, left, right, middle, temp) {
        let i = left, j = middle + 1, k = 0; 
        // 左右两个数组比较存入temp
        while (i <= middle && j <= right) {
            if (arr[i] < arr[j]) {
                temp[k++] = arr[i++]
            } else {
                temp[k++] = arr[j++]
            }
        }
        // 存放由于两个数组长度不一没能比较的值
        while (i <= middle) {
            temp[k++] = arr[i++]
        }
        while (j <= right) {
            temp[k++] = arr[j++]
        }
        temp.length = k
        arr.splice(left, k, ...temp)
    }

	/** 
     * 非递归实现归并排序, 先将
     */        
    function mergeSort(arr) {
        let i, leftMin, leftMax, rightMin, rightMax, next, temp = [], length = arr.length
        for (i = 1; i < length; i *= 2) {
            for (leftMin = 0; leftMin < length - i; leftMin = rightMax) {
                rightMin = leftMax = leftMin + i
                rightMax = leftMax + i
                if (rightMax > length) {
                    rightMax = length
                }
                next = 0
                while (leftMin < leftMax && rightMin < rightMax) {
                    temp[next++] = arr[leftMin] > arr[rightMin] ? arr[rightMin++] : arr[leftMin++]
                }
                while (leftMin < leftMax) {
                    arr[--rightMin] = arr[--leftMax]
                }
                while (next > 0) {
                    arr[--rightMin] = temp[--next]
                }
            }
        }
    }
	
---
[参考](https://www.cnblogs.com/yu-chao/p/4324485.html)
[归并非递归](https://www.cnblogs.com/bluestorm/archive/2012/09/06/2673138.html)
