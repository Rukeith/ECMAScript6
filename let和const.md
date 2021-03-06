ECMAScript 6  
環境的設置

```
$ npm install -g babel-cli
$ npm install --save babel-preset-es2015
```
在當前的目錄下，新建一個設定文件`.babelrc`。  
Babel 自帶一個`babel-node`命令，提供 ES6 的環境

	$ babel-node
	>
	> console.log([1,2,3].map(x => x * x))
	    [ 1, 4, 9 ]
	>

使用`babel`可以將ES6轉換為ES5。

	$ babel es6.js
	"use strict";
	
	console.log([1, 2, 3].map(function (x) {
	  return x * x;
	}));

`-o`參數將轉換後的代碼，從標準輸出導入文件

	$ babel es6.js -o es5.js
	# 或者
	$ babel es6.js --out-file es5.js

`-d`參數用於轉換整個目錄

	$ babel -d build-dir source-dir

`-s`參數可以生成 source map 文件

	$ babel -d build-dir source-dir -s

### 瀏覽器環境
Babel 也可以用於瀏覽器。但是 Babel 6 之後，不再直接提供瀏覽器版本，而是要用建構工具建構出來。如果不想使用的話，只能安裝 5.x 版本的`babel-core`模組。

執行`npm install babel-core@5`  
可以在當前目錄的`node_modules/babel-core/`子目錄裡面，找到`babel`的瀏覽器版本`browser.js`和`browser.min.js`。然後將下面的程式碼插入網頁。需要註明`type="text/babel"`。

	<script src="node_modules/babel-core/browser.js"></script>
	<script type="text/babel">
	// Your ES6 code
	</script>

下面是`Babel`配合`Browserify`一起使用，生成瀏覽器能夠直接載入的腳本

	$ npm install --save-dev babelify babel-preset-es2015
	$ browserify script.js -o bundle.js -t [ babelify --presets [ es2015 react ] ]

上面程式碼會將 ES6 腳本`script.js`轉為`bundle.js`，瀏覽器直接加載後者就可以了。在`package.json`設置下面的程式碼，就不用每次終端機都輸入參數了。

	{
	  "browserify": {
	    "transform": [["babelify", { "presets": ["es2015"] }]]
	  }
	}
---
##`let`和`const`
ES6 新增了`let`命令，用來宣告變數。用法類似於`var`，但是所宣告的變數，只在`let`命令所在的程式碼內才有效。

	{ 
	  let a =  10 ; 
	  var b =  1 ; 
	}
	
	a // ReferenceError: a is not defined.
	b // 1

`let`很適合使用於`for`迴圈的計數器
`for (let i = 0; i < arr.length; i++) {}`

###不存在變數提升
`let`不像`var`會有“變數提升”。所以變數的使用一定要在宣告之後。

	console.log(foo); // ReferenceError
	let foo = 2;

###temporal dead zone，簡稱TDZ
只要塊級作用域內存在`let`命令，它所宣告的變數就"綁定"這個區域，不再受外部的影響。

	var tmp = 123;
	
	if (true) {
		tmp = "abc"; // ReferenceError
		let tmp;
	}

ES6 規定，如果區域中存在`let`和`const`命令

	if  ( true )  { 
	 // TDZ開始
	   tmp =  'abc' ; // ReferenceError
	   console . log ( tmp ) ; // ReferenceError
	 
	  let tmp ; // TDZ結束
	   console . log ( tmp ) ; // undefined
	 
	  tmp =  123 ; 
	  console . log ( tmp ) ; // 123
	 }

###不允許重複宣告
`let`不允許在相同作用域內，重複宣告同一個變數。

	// 報錯
	function () {
	  let a = 10;
	  var a = 1;
	}
	
	// 報錯
	function () {
	  let a = 10;
	  let a = 1;
	}

##塊級作用域
`let`實際上為 JavaScript 新增了塊級作用域。

	function f1() {
		let n = 5;
		if (true) {
			let n = 10;
		}
		console.log(n); // 5
	}

內層作用域可以定義外層作用域的同名變數。

	{ { { { 
	  let insane =  'Hello World' ; 
	  { let insane =  'Hello World' ; } 
	} } } } ;

##`const`命令
`const`也用來宣告變數，但是是宣告常量。一旦宣告，值就不能改變。

	const PI =  3.1415 ; 
	PI // 3.1415
	 
	PI =  3 ;
	// TypeError: "PI" is read-only

`const`的作用域與`let`命令相同：只在宣告所在的塊級作用域內有效。

	if (true) { 
	  const MAX =  5; 
	}
	
	MAX // Uncaught ReferenceError: MAX is not defined

對於複合類型的變數，變數不指向資料，而是指向資料的地址。`const`只是保證變數指向的地址不便，並不保證該地址的資料不變。

	const foo = {}; 
	foo.prop = 123;
	
	foo.prop
	// 123
	
	foo = {} // TypeError: "foo" is read-only不起作用

下面是另一個例子。

	const a = [];
	a.push("Hello"); // 可執行
	a.length = 0; // 可執行
	a = ["Dave"]; // 報錯

如果真的想將物件凍結，應該使用Object.freeze方法。

	const foo = Object.freeze({}); 
	foo.prop = 123; //不起作用

##跨模組常量
`const`宣告的常量只在當前程式碼有效。如果想設置跨模組的常量，可以採用下面的寫法：

	// constants.js 模块
	export const A = 1;
	export const B = 3;
	export const C = 4;
	
	// test1.js 模块
	import * as constants from './constants';
	console.log(constants.A); // 1
	console.log(constants.B); // 3
	
	// test2.js 模块
	import {A, B} from './constants';
	console.log(A); // 1
	console.log(B); // 3

##全域物件的屬性
全域物件是最頂層的物件，在瀏覽器是指`window`，在 Node.js 是指`global`。ES5 中，全域物件的屬性與全域變數是相等的。

	window.a = 1;
	a // 1
	
	a = 2
	window.a // 2

這規範是為 JavaScript 的一個大問題，因為容易在不知情之下就建立了全域變數。ES6 改變了這一點，`var`和`function`宣告的全域變數，依舊是全域物件的屬性；但`let`、`const`和`class`宣告的全域變數，不屬於全域物件的屬性。

	var a = 1;
	// 如果在Node的REPL环境，可以写成global.a
	// 或者采用通用方法，写成this.a
	window.a // 1
	
	let b = 1;
	window.b // undefined






