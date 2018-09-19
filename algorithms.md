> *   __*交换运算切勿使用*__ `[arr[i], arr[j]] = [arr[j], arr[i]]` __*会消耗大量时间去运算*__
> *   __*整数转换时，使用*__ `Math.floor` __*比之*__ `Number.parseInt` __*要快很多， 如果能使用位运算 `>>` 
、`<<` 更快*__
> *   __*数组替换，大量数据使用*__ `Array.prototype.splice` __*比*__ `for` __*循环快*__

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

算法函数
---

### 快速排序
> *   时间复杂度O(nlogn)~O(n^2), 平均为O(nlogn), 空间复杂度O(1), 不稳定
> *   双向进行比较, 运行时间更少

	function quickSort(arr, sub = 0, sup = arr.length - 1) {
		const pivot = arr[sub]
	    let left = sub, right = sup
	    if (sub < sup) {
	        while (left < right) {
				// 标志为低位时，需先从高位比较，反之亦然
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
	            return quickSort(arr, sub, left - 1)
	        }
	        if (left + 1 < sup) {
	            return quickSort(arr, left + 1 , sup)
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
        const originIndex = (arr.length - 1) >> 1
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
        let middle = (right + left) >> 1
        if (left < right) {
            if (left < middle) {
                return mergeSort(arr, left, middle, temp);
            }
            if (middle + 1 < right) {
                return mergeSort(arr, middle + 1, right, temp);
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
#### 插入排序
> *   时间复杂度O(n^2), 稳定

	function insertSort(arr) {
        for (let i = 1; i < arr.length; i ++) {
            let tmp = arr[i], j = i - 1
            while (j >= 0 && tmp < arr[j]) {
                arr[j + 1] = arr[j]
                j --
            }
            arr[j + 1] = tmp
        }
    }

#### 希尔排序
> *   时间复杂度O(n)~O(n^（3/2）), 改进型的插入排序
> *   希尔排序的效率与增量有关, 较优的增量为n/3+1, n/9+1, n/27+1 ... 1

	function shellSort(arr) {
        let incr = 1, gap
        do {
            incr *= 3
            gap = Math.floor(arr.length / incr) + 1
            for (let i = gap; i < arr.length; i ++) {
                let tmp = arr[i], j = i - gap
                while (j >= 0 && tmp < arr[j]) {
                    arr[j + gap] = arr[j]
                    j -= gap
                }
                arr[j + gap] = tmp
            }
        } while (gap > 1)
    }

---

### 线性查找算法(BFPRT)
> *   查找第k小(大)个元素, 时间复杂度O(n)

	function select(arr, k, left = 0, right = arr.length - 1) {
	    if (right - left < 5) {
	        insertSort(arr, left, right)
	        return arr[left + k - 1]
	    }
	    let median = left - 1, tmp
		// 每五个元素为一组，找到每组中位数并交换至数组前列
	    for (let i = left; i + 4 <= right; i += 5) {
	        insertSort(arr, i, i + 4)
	        tmp = arr[++median]
	        arr[median] = arr[i + 2]
	        arr[i + 2] = tmp
	    }
		// 找到所有中位数位于中间的下标
	    let pivotIndex = (left + median) >> 1
		// 中位数排序，调整位置
	    select(arr, pivotIndex - left + 1, left, median)
		// 根据中位数的中位数将数组分为小于这个数的部分和大于它的部分，并返回这个数
	    let midIndex = partition(arr, left, right, pivotIndex)
	    let index = midIndex - left + 1
	    if (k == index) {
	        return arr[midIndex]
	    } else if (k < index) {
	        return select(arr, k, left, midIndex - 1)
	    } else {
	        return select(arr, k - index, midIndex + 1, right)
	    }
	}

	function partition(arr, left, right, pivotIndex){
		let temp, index = left - 1
		temp = arr[left]
		arr[left] = arr[pivotIndex]
		arr[pivotIndex] = temp
	    for (let i = left; i < right; i++) {
	        if (arr[i] < arr[right]) {
	            temp = arr[i]
	            arr[i] = arr[++index]
	            arr[index] = temp
	        }
	    }
	    temp = arr[++index]
	    arr[index] = arr[right]
	    arr[right] = temp
	    return index
	}

---

### 二叉树的遍历
> *   数组模拟二叉树, 父节点 `index`, 左子节点 `2 * index + 1`, 右子节点 `2 * index + 2`  

#### 前序遍历
> *   根节点 ---> 左子树 ---> 右子树

	function preOrderTraversal(arr, result = [], root = 0) {
        let left = 2 * root + 1, right = 2 * root + 2
		if (arr[root]) {
            result.push(arr[root])
        }
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
                if (arr[index]) {
                    result.push(arr[index])
                }
                stack.push(index)
                index = 2 * index + 1
            } else {
                index = 2 * stack.pop() + 2
            }
        }
        return result
    }

#### 中序遍历
> *   左子树 ---> 根节点 ---> 右子树

	function inOrderTraversal(arr, result = [], root = 0) {
        let left = 2 * root + 1, right = 2 * root + 2
        if (left < arr.length) {
            inOrderTraversal(arr, result, left)
        }
        if (arr[root]) {
            result.push(arr[root])
        }
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
				if (arr[index]) {
                	result.push(arr[index])
				}
                index = 2 * index + 2
            }
        }
        return result
    }

#### 后序遍历
> *   左子树 ---> 右子树 ---> 根节点

	function postOrderTraversal(arr, result = [], root = 0) {
	    let left = 2 * root + 1, right = 2 * root + 2
	    if (left < arr.length) {
	        postOrderTraversal(arr, result, left)
	    }
	    if (right < arr.length) {
	        postOrderTraversal(arr, result, right)
	    }
	    if (arr[root]) {
            result.push(arr[root])
        }
	}

	// 非递归
	function postOrderTraversal(arr) {
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
					if (arr[index]) {
                    	result.push(arr[index])
					}
                    index = stack.pop()
                    flag[index] = false		// 重置当前节点的父节点状态, 因为其出栈并非栈顶出栈
                }
            }
        }
        return result
    }

#### 层级遍历
> *   广度优先遍历

	function levelTraversal(arr) {
        let result = [], incer = 1, level = 0
        while (incer < arr.length) {
            for (let i = 0; i < incer; i++) {
                if (arr[incer - 1 + i]) {
                    result.push(arr[incer - 1 + i])
                }
            }
            level++
            incer *= 2
        }
        return result
    }

	// 减少遍历次数, 适合遍历非完全二叉树
	function levelTraversal(arr, callback) {
        if (![...arr].shift()) {
            return {}
        }
        const result = {}, queue = []
        let curIndex = 0, cur = arr[curIndex]

        queue.push(curIndex)
        result[cur] = {}
        result[cur].distance = 0
        result[cur].predecessor = null

        while (queue.length) {
            curIndex = queue.shift()
            cur = arr[curIndex]
            if (arr[2 * curIndex + 1]) {
                queue.push(2 * curIndex + 1)
                result[arr[2 * curIndex + 1]] = {}
                result[arr[2 * curIndex + 1]].distance = result[cur].distance + 1
                result[arr[2 * curIndex + 1]].predecessor = cur
            }
            if (arr[2 * curIndex + 2]) {
                queue.push(2 * curIndex + 2)
                result[arr[2 * curIndex + 2]] = {}
                result[arr[2 * curIndex + 2]].distance = result[cur].distance + 1
                result[arr[2 * curIndex + 2]].predecessor = cur
            }
            callback(cur)
        }
        return result
    }

#### 任意两种遍历还原二叉树
> *   二叉树还原必须知道中序遍历, 中序遍历用来控制左右子节点位置

##### 前序遍历, 中序遍历
> *   可还原任意二叉树, 空节点数组中亦为空

	function recoverBinaryTree(preOrder, inOrder) {
        let tmp = [], index = 0, node = null, float = 0
        do {
            node = preOrder.shift()
            tmp[index] = node
            float = inOrder.indexOf(node)
            if (float == 0) {
                while (tmp.indexOf(inOrder[0]) != -1) {
                    index = tmp.indexOf(inOrder.shift())
                }
            }
            if (inOrder.indexOf(preOrder[0]) < float) {
                index = 2 * index + 1
            } else {
                index = 2 * index + 2
            }
        } while (inOrder.length != 0 || preOrder.length != 0)
        return tmp
    }

##### 前序遍历, 后序遍历
> *   只能还原完全二叉树, 前序 + 后序只能确定父子层级关系无法确定位置

	function recoverBinaryTree(preOrder, postOrder) {
        let tmp = [], index = 0, node = postNode = null
        while (preOrder.length != 0 || postOrder.length != 0) {
            postNode = postOrder[0]
            if (tmp.indexOf(postNode) == -1) {
                node = preOrder.shift()
                if (!tmp[index]) {
                    tmp[index] = node
                }
                if (!tmp[2 * index + 1]) {
                    index = 2 * index + 1
                } else {
                    index = 2 * index + 2
                }
            } else {
                postOrder.shift()
                index = tmp.indexOf(postNode) + 1
            }
        }
        return tmp
    }

##### 中序遍历, 后序遍历
> *   可还原任意二叉树, 空节点数组中亦为空

	function recoverBinaryTree(inOrder, postOrder) {
        let tmp = [], index = 0, node = null, float = 0
        do {
            node = postOrder.pop()
            tmp[index] = node
            float = inOrder.indexOf(node)
            if (float == inOrder.length - 1) {
                while (tmp.indexOf(inOrder[inOrder.length - 1]) != -1) {
                    index = tmp.indexOf(inOrder.pop())
                }
            }
            if (inOrder.indexOf(postOrder[postOrder.length - 1]) < float) {
                index = 2 * index + 1
            } else {
                index = 2 * index + 2
            }
        } while (inOrder.length != 0 || postOrder.length != 0)
        return tmp
    }

### 图
> * 无权重的图

    class Graph {
        /** 
         * 图构造函数
         * @params vertices 图顶点
         * @params edges    如果参数为Map类型为 图的边，链接表保存，为boolean时判断有向图
         * @params directed 判断有向图 
         */        
        constructor(vertices = [], edges = new Map(), directed = false) {
            if (edges instanceof Map) {
                this.edges = edges
            }
            if (typeof edges == 'boolean') {
                directed = edges
            }
            this.vertices = vertices
            this.directed = directed
            this.vertices.forEach((item) => {
                this.edges.set(item, [])
            })
        }

        /** 
         * 添加顶点
         * @params vertex   如果参数类型为Array，替换原有全部顶点，否则添加顶点
         */        
        addVertex(vertex) {
            if (vertex instanceof Array) {
                this.edges.clear()
                this.vertices = vertex
                this.vertices.forEach((item) => {
                    this.edges.set(item, [])
                })
            } else {
                this.vertices.push(vertex)
                this.edges.set(vertex, [])
            }
        }

        addEdge(m, n) {
            if (this.vertices.includes(m) && this.vertices.includes(n)) {
                if (this.directed) {
                    this.edges.get(m).push(n)
                } else {
                    this.edges.get(n).push(m)
                    this.edges.get(m).push(n)
                }
            } else {
                throw new Error('unknown vertex')
            }
        }
    }


#### 广度优先搜索(BFS)
> *   初始化每个点的标记为未检查，选一个起始点入队列，设为已检查
> *   依次出队列，查询当前顶点的相邻点，遍历相邻点，检查相邻点是否已检查，未检查设为检查，并推入队列
> *   终止条件：队列为空

	function bfs(graph, callback, start = [...graph.vertices].shift()) {
        if (!start) {
            return {}
        }
        // vertices: 便利的顶点，有路径距离，前溯点属性  checked: 表示当前节点是否被检查过
        const vertices = {}, queue = [], checked = []
        graph.vertices.forEach((item) => {
            vertices[item] = {
                distance: 0,
                predecessor: null,
            },
            checked[item] = 0
        })
        queue.push(start)
        checked[start] = 1
        while(queue.length) {
            let cur = queue.shift()
            graph.edges.get(cur).forEach((item) => {
                if (!checked[item]) {
                    queue.push(item)
                    vertices[item].distance = vertices[cur].distance + 1
                    vertices[item].predecessor = cur
                    checked[item] = 1
                }
            })
            callback(cur)
        }
        return vertices
    }

#### 深度优先搜索(DFS)
> *   采用迭代的方式，类似于树的遍历，但是要在遍历过程中检查当前点是否已经并遍历过

	function dfs(graph, callback, start = [...graph.vertices].shift()) {
        if (!start) {
            return {}
        }
        const vertices = {}, checked = []
        graph.vertices.forEach((item) => {
            vertices[item] = {
                distance: 0,
                predecessor: null,
            },
                checked[item] = 0
        })
        dfsRecur(graph, callback, start, checked, vertices)
        return vertices
    }
    
    function dfsRecur(graph, callback, start, checked, vertices) {
        callback(start)
        checked[start] = 1
        graph.edges.get(start).forEach((item) => {
            if (!checked[item]) {
                vertices[item].distance = vertices[start].distance + 1
                vertices[item].predecessor = start
                dfsRecur(graph, callback, item, checked, vertices)
            }
        })
    }

---
算法思想
---

> *   __*无后效性：*__ 某阶段状态一旦确定，就不受这个状态以后决策的影响

### 贪心算法
> *   每个阶段的最优状态由上一个阶段最优状态决定
> *   前提是选择的贪心策略具备无后效性
> *   自顶向下的处理问题
> *   贪心算法根据贪心策略的选择并不一定可以得到问题的最优解，通常是近似解

#### 思路
> 1.  建立数学模型描述问题
> 2.  求解问题划分多个子问题
> 3.  求解每个子问题，得到子问题最优解
> 4.  子问题最优解合成原问题的解

### 分治
> *   将复杂问题分解成若干个独立的相似的子问题，通过子问题解决合并到原问题

#### 特征
> 1.  问题可以通过缩小规模解决
> 2.  分解出的问题具有相似性
> 3.  子问题可以合并为原问题的解
> 4.  子问题应相互独立


### 动态规划
> *   阶段的当前状态依赖于之前的某个或某些状态
> *   每个阶段的最优状态可以从之前阶段的某个或某些状态直接获得而不管之前这个状态如何得到的

#### 特征
> 1.  问题具有最优子结构，即问题的最优解其包含的子问题的解也最优
> 2.  每个阶段都具有无后效性
> 3.  子问题不独立

#### 思路
> 1.  根据问题的时间或空间特征，将问题划分为若干阶段，划分后的阶段应该是有序的或可排序的，否则无法求解
> 2.  将问题各个阶段的不同情况下的不同状态存储起来
> 3.  因为决策和状态转移有着天然的联系，状态转移就是根据上一阶段的状态和决策来导出本阶段的状态。所以如果确定了决策，状态转移方程也就可写出。但事实上常常是反过来做，根据相邻两个阶段的状态之间的关系来确定决策方法和状态转移方程
> 4.  给出的状态转移方程是一个递推式，需要一个递推的终止条件或边界条件

---
[参考](https://www.cnblogs.com/yu-chao/p/4324485.html)
[归并非递归](https://www.cnblogs.com/bluestorm/archive/2012/09/06/2673138.html)
[五大算法思想](https://blog.csdn.net/KingCat666/article/details/73611009)
