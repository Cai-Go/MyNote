 # ES6 声明变量的六种方法
 ES5 只有两种声明变量的方法：var 命令和 function 命令。

 ES6 除了添加 let 和 const 命令，另外两种声明变量的方法：import 命令和 class 命令。所以，ES6 一共有 6 种声明变量的方法。

 # 顶层对象的属性

顶层对象，在浏览器环境指的是window对象，在 Node 指的是global对象。ES5 之中，顶层对象的属性与全局变量是等价的。

ES6 为了改变这一点，一方面规定，为了保持兼容性，var 命令和 function 命令声明的全局变量，依旧是顶层对象的属性；另一方面规定，let 命令、const 命令、class 命令声明的全局变量，不属于顶层对象的属性。也就是说，从 ES6 开始，全局变量将逐步与顶层对象的属性脱钩。

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