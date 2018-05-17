### _async/await_ 

###### 1. async返回值
>函数运行时, 首先运行macrotask, 并直接返回一个promise, return代表resolve, throw代表reject
>
>如果async函数内部存在promise, setTimeout, 再它们内部返回（return, throw）并不会执行, 执行函数内同步语句后, 若无返回语句, 直接返回resolver的promise, 后续then传入值为undefined

	function testPromise() {
        return new Promise((resolve, reject) => {
            console.log('this is promise')
            resolve()
        })
    }
    let testAsync = async () => {
        setTimeout(() => {
            console.log('hello, world!')
            return 'resolve'
        }, 1000)
        throw 'reject'
    }
    let promiseAsync = async () => {
        testPromise().then(() => {
            console.log('this is promiseAsync ')
            return 'promise resolve'
        })
    }
    promiseAsync().then((args) => {
        console.log(args)
    }).catch(err => {
        console.log(err)
    })
    testAsync().then((args) => {
        console.log('resolve!!!', args)
    }, (args) => {
        console.log(args)
    })
    /**
     * this is promise
     * this is promiseAsync
     * undefined
     * reject
     * hello, world!
     **/

###### 2. await执行顺序
>await后只能跟随promise/function, 执行到await时, await后跟随的语句会执行, 

	let testAwait = async () => {
        console.log('hello, world!')
        let o = await console.log('this is await') 
        console.log(o, 'await end')
    }
    testAwait()       
    console.log('wait testAwait')
	/**
     * hello, world!
     * this is await
     * wait testAwait
     * undefined "await end"
     **/

