### 快速排序
>*  时间复杂度O(nlogn)~O(n^2), 空间复杂度O(1)
>*  双向进行比较, 运行时间更少

	function quickSort(arr, sub = 0, sup = arr.length - 1) {
		const pivot = arr[Number.parseInt((sub + sup) / 2)] 
		let left = sub, right = sup
		if (sub < sup) {
			while (left <= right) {
				while (arr[left] < pivot) {
					left ++
				}
				while (arr[right] > pivot) {
					right --
				}
				if (left <= right) {
					[arr[left], arr[right]] = [arr[right], arr[left]]
					left ++
					right --
				}
			}
			if (sub < left - 1) {
				quickSort(arr, sub, left - 1)
			}
			if (left < sup) {
				quickSort(arr, left, sup)
			}
		}
	}

---
### 堆排序

	function heapSort() {
		 
	}

