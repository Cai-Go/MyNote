 # ES6 声明变量的六种方法
 ES5 只有两种声明变量的方法：var 命令和 function 命令。

 ES6 除了添加 let 和 const 命令，另外两种声明变量的方法：import 命令和 class 命令。所以，ES6 一共有 6 种声明变量的方法。

 # 顶层对象的属性

顶层对象，在浏览器环境指的是window对象，在 Node 指的是global对象。ES5 之中，顶层对象的属性与全局变量是等价的。

ES6 为了改变这一点，一方面规定，为了保持兼容性，var 命令和 function 命令声明的全局变量，依旧是顶层对象的属性；另一方面规定，let 命令、const 命令、class 命令声明的全局变量，不属于顶层对象的属性。也就是说，从 ES6 开始，全局变量将逐步与顶层对象的属性脱钩。

# global 对象

ES5 的顶层对象，本身也是一个问题，因为它在各种实现里面是不统一的。

- 浏览器里面，顶层对象是window，但 Node 和 Web Worker 没有window。
- 浏览器和 Web Worker 里面，self也指向顶层对象，但是 Node 没有self。
- Node 里面，顶层对象是global，但其他环境都不支持。

同一段代码为了能够在各种环境，都能取到顶层对象，现在一般是使用this变量，但是有局限性。

- 全局环境中，this会返回顶层对象。但是，Node 模块和 ES6 模块中，this返回的是当前模块。
- 函数里面的this，如果函数不是作为对象的方法运行，而是单纯作为函数运行，this会指向顶层对象。但是，严格模式下，这时this会返回undefined。
- 不管是严格模式，还是普通模式，new Function('return this')()，总是会返回全局对象。但是，如果浏览器用了 CSP（Content Security Policy，内容安全政策），那么eval、new Function这些方法都可能无法使用。

两种勉强取到顶层对象的方法:
````js
	//方法一
	（typeof window !== 'undefined'
		? window
		:(typeof process === 'object' &&
		  typeof require === 'function' &&
		  typeof global === 'object')
		? global
		: this）;

	// 方法二
	var getGlobal = function(){
		if(typeof self !== 'undefined'){return self;}
		if(typeof window !== 'undefined'){return window;}
		if(typeof global !== 'undefined'){return global;}
		throw new Error('unable to locate global object');
	};
````
