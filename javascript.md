# Javascript

## 闭包



### 词法作用域 () Lexical scoping)

函数作用域的嵌套关系由定义时决定而不是调用时决定.

1. 作用域指代码中定义变量的区域，规定了如何查找变量
2. 无块级作用域（let除外）
3. 函数作用域
4. 嵌套函数的作用域链
5. 执行前创建
6. 提前声明

当查找变量的时候，会先从当前上下文的变量对象中查找，如果没有找到，就会从父级(词法层面上的父级)执行上下文的变量对象中查找，一直找到全局上下文的变量对象，也就是全局对象。这样由多个执行上下文的变量对象构成的链表就叫做作用域链。

```
var global = 1
function localScope1 () {  
	//scope {global:1, local:2}
	var local = 2; 
	return function(num) {
		//scope{global:1, local:2, local1: 3}
		var local1 = 3
		return global + local + local1 + num;
	}
}

function localScope2() {
	var local = 3;
	var global = 10;
	return localScope1();
}

localScope2()(5) ; //1+2+3+5
```

### 作用域链

### 执行上下文（execution context）

> 执行时机：全局代码，函数代码，eval代码
> ECStack = [globalContext]
> // fun1()
> ECStack.push(<fun1> functionContext);
> // fun1中竟然调用了fun2，还要创建fun2的执行上下文
> ECStack.push(<fun2> functionContext);
> // fun2执行完毕
> ECStack.pop();
> // fun1执行完毕
> ECStack.pop();

每个执行上下文，包括三个重要属性

1. 变量对象（Variable object, VO）
2. 作用与链 (Scope chain)
3. this

executionContextObj = { 
  variableObject: { /* 函数中的arguments对象, 参数, 内部的变量以及函数声明 */ }, 
  scopeChain: { /* variableObject 以及所有父执行上下文中的variableObject */ },  //词法作用域，非运行时确定
  this: {} 
} 

变量对象会包括：

1. 函数的所有形参 (如果是函数上下文)
  * 由名称和对应值组成的一个变量对象的属性被创建
  * 没有实参，属性值设为 undefined

2. 函数声明
  * 由名称和对应值（函数对象(function-object)）组成一个变量对象的属性被创建
  * 如果变量对象已经存在相同名称的属性，则完全替换这个属性

3. 变量声明
  * 由名称和对应值（undefined）组成一个变量对象的属性被创建；
  * 如果变量名称跟已经声明的形式参数或函数相同，则变量声明不会干扰已经存在的这类属性

对象变量的创建过程

1. 全局上下文的变量对象初始化是全局对象
2. 函数上下文的变量对象初始化只包括 Arguments 对象
3. 在进入执行上下文时会给变量对象添加形参、函数声明、变量声明等初始的属性值
4. 在代码执行阶段，会再次修改变量对象的属性值

### this
this指向调用方法的对象,其值在函数被调用的时候才会指定
方法直接调用，则调用对象是window，不管方法嵌套的多少层。
对于this，我们可以理解为：特殊的Scope引用变量，其指向当前函数的执行环境Scope（并不是定义时的Scope）
箭头函数保存函数创建时的 `this`值，而不是调用时的值

```js
var x = 0;
var o = {}
function foo() {
  this. x = 1; 
  that = this;
  var inner = function() {
      console.log(that === window);console.log(that);
  }
  inner();
}
console.log(x); //0
foo();
console.log(x); //1
o.x = 2;
o.m = test;
console.log(o.x); //2
o.test();
console.log(o.x); //1
```



## ES6

### 变量

局部变量必须一个 var 开头，如果未使用var，则默认表示声明的是全局变量。

变量作用域：一对大括号`{}`内的代码都称为一个**作用域**

#### 全局作用域

1. 没有使用任何关键字声明的变量，都是全局变量
2. 不是在**函数**中使用 `var` 关键字声明的变量，也是全局变量

> 注意： 全局作域的 变量 可以被重复声明，并且可以被重新赋值。

```js
var n = 10;
var n = 100;
{
    var n = 10;
};

if (1 ===1){
    var n = '千锋云计算';
}

console.log(n) //千锋云计算
```

#### 函数作用域

1.  在函数中以关键字 `var` 开头声明的变量
2. var变量提升， var声明提升，但初始化不会， 所以var变量可以先使用，再声明，会引起混淆。

```js
var x = "global"
function test(inner) {
	if (inner) {
		var x = 'localInner'; // 作用于整个 function
        return x;
    }
    return x; // 因为第四行的声明提升，被重新定义
}

test(false); //undefined
test(true); //localInner
```

#### 块局部作用域（ES6）

`let` 和 `const` 声明的变量

1. `let`   声明的变量只在它所在的代码块有效，在同一个代码块(作用域)下，不可以重复声明，可以被重新赋值。
2. `const` 声明的常量只在它所在的代码块有效，在同一个代码块(作用域)下，不可以重复声明，并且不可以被重新赋值(基本类型的数据)。
3. let变量提升， let变量提升到代码块的顶部（非函数的顶部），而且是要先声明再使用。存在**暂存死区**（代码块开始直到变量被声明之间的区域）

```js
for (let i = 0; i < 10; i++) {
      console.log(i);
}

console.log("在代码块外面打印的变量 i 将会报错")
console.log(i);// ReferenceError: i is not defined
```

#### Immediately Invoked Function Expression(IIFE)

- 实现单例

  ```javascript
  var module= (function() {
  	var aObj = {
          'type': 1,
          'price': 1
      };
      function addPrice(addition) {
          aObj.price = aObj.price + addition;
      };
      function getType() {
          return aObj.type;
      };
      function getPrice() {
          return aObj.price;
      };
      return {
          addPrice,
          getType,
          getPrice
      }
  
  })();
  ```

  



### 函数

#### 箭头函数

```
let user = {
    name,    // 相当于'name': name
    'age': 18,
    'show': () => { // 相对于'show': function(){...}
    	console.log(`user info: name(${name}) age(${this.age})`)
    }
}
```

相对于传统的匿名函数，箭头函数的this是可预测的。其`this` 是基于词法绑定，这仅仅是意味着它的值被绑定到父级作用域的一种奇特的方式，并且永远不会改变。

#### 生成器

生成器是可以暂停和恢复的函数。

声明生成器只是在关键字function的后面增加一个星号，生成器返回的是一个迭代器。迭代器的next函数，用于执行生成器的代码，直到遇到一个yield。 返回结果对象包括两个键-value和done。





### 内置数据类型

#### 字符串

- 字符串模板

  ```javascript
  var name = "shark"
  var age = 18
  var tag = `<tr>
  <td>${name}</td>
  <td>${age}</td>
  </tr>`
  ```

#### 数组

- 扩展运算符（spread）是三个点（`...`）。将一个数组转为用逗号分隔的参数序列。

  ```javascript
  Math.max(...[1,2,3])  
  Math.max(1,2,3)
  ```

- 解构赋值

  变量从数组中抽取出来

  ```js
  let [name, age] = ['emma', 18]
  ```

- 遍历

  ```jsx
  for (let [index, elem] of arr.entries()) {
    console.log(index, elem);
  }
  ```

### 对象

JS的对象相当于其他语言的字典或Map对象。

#### 对象拷贝

1. `Object.assign`方法的第一个参数是目标对象，后面的参数都是源对象。
2. 深拷贝与浅拷贝

#### 对象访问

- 定义

  ```js
  let a = 1
  var obj = {
      a,    // 相当于'a': a
      'b': 30,
      show(){ // 相对于'show': function(){...}
      console.log(this.a)
      }
  }
  ```

- 解构赋值

  变量从对象中抽取出来

  ```javascript
  let name = "emma"
  let user = {
      name,    // 相当于'name': name
      'age': 18,
      show(){ // 相对于'show': function(){...}
      	console.log(`user info: name(${name}) age(${this.age})`)
      }
  }
  const {userName:name, userAge:age, show} = user
  ```

  

- 动态属性

  ```
  let attName = 'name'
  let user = {
  	[attName]: name, //  引用了变量attName
      'age': 18,
      show(){ // 相对于'show': function(){...}
      	console.log(`user info: name(${this[attName]}) age(${this.age})`)
      }
  }
  ```

  

- 遍历

```js
// 变量 key 和 value
for (let [k,v] of Object.entries(o)){
    console.log(k,v)
}
```

## Webpack

webpack是 JavaScript 应用程序的模块打包器，强调的是一个前端模块化方案，更侧重模块打包，我们可以把开发中的所有资源（图片、js文件、css文件等）都看成模块，通过loader（加载器）和plugins（插件）对资源进行处理，打包成符合生产环境部署的前端资源.

![webpack](img\webpack.jpg)

```mermaid
graph LR
	初始化参数 -->确定入口 
	确定入口 --> 编译模块
	编译模块 --> 输出资源
```

### 概念

- 入口 entry

  > 告诉 webpack 从哪个文件开始构建，这个文件将作为 webpack 依赖关系图的起点

  ```javascript
  module.exports = {
    entry: {
      app: './src/app.js',
      vendors: './src/vendors.js'
   }
  };
  ```

- 出口 Output

  > 告诉 webpack 在哪里输出 构建后的包、包的名称 等

  ```javascript
  const path = require('path');
  
  module.exports = {
    entry: {
      app: './src/app.js',
      vendors: './src/vendors.js'
    },
    output: {
      filename: '[name].js',
      path: path.resolve(__dirname, 'dist')
    }
  }
  ```

- 加载器 Loader

  > loader 让 webpack 能够去处理那些非 JavaScript 文件（webpack 自身只理解 JavaScript）
  >
  > loader 可以将所有类型的文件转换为 webpack 能够处理的有效模块

  ```javascript
  module.exports = {
    module: {
      rules: [{ 
          test: /\.css$/, 
          use: ['style-loader', {
              loader: 'css-loader',
              options: {
                  modules: true
              }
          }]
      }]
    }
  };
  ```

- 插件 Plugin

  > 可以处理各种任务，从打包优化和压缩，一直到重新定义环境中的变量

  ```javascript
   plugins: [
          new HtmlWebpackPlugin({
              title: '首页', // 用于生成的HTML文档的标题
              filename: 'index.html', //写入HTML的文件。默认为index.html。也可以指定一个子目录（例如：）assets/admin.html
              template: 'index.html' // Webpack需要模板的路径
          }),
          new webpack.HotModuleReplacementPlugin() // 需要结合 启用热替换模块(Hot Module Replacement)，也被称为 HMR
      ]
  ```