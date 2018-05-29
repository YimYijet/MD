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
> *   归并排序速度介于快排和堆排序之间, 对于中数量级数据有较优的表现( 非递归实现 )
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
     * 非递归实现归并排序, 将数组按照1, 2, 4, ..., 2^n个元素的顺序依次分割, 自底向上进行比较合并
     */        
    function mergeSort(arr) {
		// leftMin~leftMax 划分了左侧待比较区域, right同理, next标志temp数组的下标移动
        let i, leftMin, leftMax, rightMin, rightMax, next, temp = [], length = arr.length
        for (i = 1; i < length; i *= 2) {	// 以2^n划分比较区域
            for (leftMin = 0; leftMin < length - i; leftMin = rightMax) {	// 通过leftMin = rightMax, 将比较区域后移
                rightMin = leftMax = leftMin + i	// 初始化左右比较区域的标志Max, 和下标移动Min
                rightMax = leftMax + i
                if (rightMax > length) {		// 数组长度不足处理
                    rightMax = length
                }
                next = 0
                while (leftMin < leftMax && rightMin < rightMax) {
                    temp[next++] = arr[leftMin] > arr[rightMin] ? arr[rightMin++] : arr[leftMin++]
                }
                while (leftMin < leftMax) {		// 左侧比较区域只会比右侧大, 处理右侧数据不足时, 无法比较
                    arr[--rightMin] = arr[--leftMax]	// 将多出的部分存入arr  
                }
                while (next > 0) {
                    arr[--rightMin] = temp[--next]		// next记录了比较的部分, 将其从右(rightMax - (leftMax - leftMin))向左(leftMin)存入arr
                }
            }
        }
    }
	
---

### 冒泡排序
> *   时间复杂度O(n)~O(n^2), 平均O(n^2), 稳定
> *   只适合少量数据比较

	function bubbleSort(arr, left = 0, right = arr.length - 1) {
        let temp
        while (left < right) {
            for(let i = left; i < right; i++) {
                if (arr[i] > arr[i + 1]) {
                    temp = arr[i]
                    arr[i] = arr[i + 1]
                    arr[i + 1] = temp
                }
            }
            right --
            if (left >= right) {
                break
            }
            for (let i = right; i > left; i--) {
                if (arr[i] < arr[i - 1]) {
                    temp = arr[i]
                    arr[i] = arr[i - 1]
                    arr[i - 1] = temp
                }
            }
            left ++
        }
    }

---

### 希尔排序
> * 

---
### 二叉树的遍历
> * 数组模拟二叉树, 父节点 _index_, 左子节点 _2 * index + 1_, 右子节点 _2 * index + 2_

###### 前序遍历
> * 根节点 ---> 左子树 ---> 右子树
	
	function preOrderTraversal(arr, result = [], root = 0) {
        let left = 2 * root + 1, right = 2 * root + 2
        result.push(arr[root])
        if (left < arr.length) {
            preOrderTraversal(arr, result, left)
        }
        if (right < arr.length) {
            preOrderTraversal(arr, result, right)
        }
    }
	
	// 非递归
	function preOrderTraversal(arr) {
        let result = [], index = 0, stack = []
        while (stack.length != 0 || !!arr[index]) {
            if (index < arr.length) {
                result.push(arr[index])
                stack.push(index)
                index = 2 * index + 1 
            } else {
                index = 2 * stack.pop() + 2
            }
        } 
        return result
    }

###### 中序遍历
> * 左子树 ---> 根节点 ---> 右子树

	function inOrderTraversal(arr, result = [], root = 0) {
        let left = 2 * root + 1, right = 2 * root + 2
        if (left < arr.length) {
            inOrderTraversal(arr, result, left)
        }
        result.push(arr[root])
        if (right < arr.length) {
            inOrderTraversal(arr, result, right)
        } 
    }
	
	// 非递归
	function inOrderTraversal(arr) {
            let result = [], index = 0, stack = []
            while (stack.length != 0 || !!arr[index]) {
                if (index < arr.length) {
                    stack.push(index)
                    index = 2 * index + 1 
                } else {
                    index = stack.pop()
                    result.push(arr[index])
                    index = 2 * index + 2
                }
            } 
            return result
        }

###### 后序遍历
> * 左子树 ---> 右子树 ---> 根节点

	function postOrderTraversal(arr, result = [], root = 0) {
	    let left = 2 * root + 1, right = 2 * root + 2
	    if (left < arr.length) {
	        postOrderTraversal(arr, result, left)
	    }
	    if (right < arr.length) {
	        postOrderTraversal(arr, result, right)
	    } 
	    result.push(arr[root])
	}	
	
	// 非递归
	function preOrderTraversal(arr) {
        let result = [], index = 0, stack = [], flag = []
        while (stack.length != 0 || !!arr[index]) {
            if (index < arr.length && !flag[index]) {
                stack.push(index)
                index = 2 * index + 1 
            } else  {
                index = stack.pop()
                if (!flag[index]) {		// 当前节点第一次出栈, 需遍历右子树, 重新将父节点压栈
                    flag[index] = true	// 设置当前节点状态, 表示已遍历左子树
                    stack.push(index)
                    index = 2 * index + 2
                } else {		// 当前节点第二次出栈, 左右子树遍历完, 输出
                    result.push(arr[index])
                    index = stack.pop()
                    flag[index] = false		// 重置当前节点的父节点状态, 因为其出栈并非栈顶出栈
                }
            }
        } 
        return result
    }

###### 任意两种遍历还原二叉树
> * 

---
[参考](https://www.cnblogs.com/yu-chao/p/4324485.html)
[归并非递归](https://www.cnblogs.com/bluestorm/archive/2012/09/06/2673138.html)
