# react 项目中使用 web-worker

web-worker 的 api 很简单，`new Worker('file.js')` 但是在 react 项目中如果这样实例化一个 worker 的话，会出现错误：

```
Uncaught SyntaxError: Unexpected token <
```

错误原因很简单，因为经过 webpack 等打包工具打包后，不存在这个文件。所以由此可见，我们有两种解决办法：

1. 将 worker 文件单独打包，然后实例化 worker 时填打包后文件的相对地址
2. 动态创建一个 script 文件，然后引入

方法一需要打包时需要文件名不能带内容 hash ，这不利于浏览器缓存，所以本文只介绍第二种方法。

## 动态创建 script 文件

将 js 代码转换为字符串，然后实例化一个 MIME 类型为 `application/javascript`的 blob 对象，最终通过 createObjectURL 动态生成 url。从而可以使实例化 worker 时可以正常工作。

```js
// worker.js
const workerCode = () => {
  onmessage = ({ data }) => {
		// your code
    postMessage(rst);
  };
};
// 把脚本代码转为string
let code = workerCode.toString();
code = code.substring(code.indexOf('{') + 1, code.lastIndexOf('}'));

const blob = new Blob([code], { type: 'application/javascript' });

const workerScript = URL.createObjectURL(blob);
export default workerScript;
```

```js
// component.jsx
import workerScript from './worker.js'
const worker = new Worker(workerScript);
// ...
```

看到 webpack 的社区中有一个 `worker-loader` 也可以使用，但是原理应该是上述提到的两种方法，使用 worker-loader 可能更契合 webpack 和模块化开发的理念。

## 出现的问题

使用上述方法在本地开发时没有遇到问题，但是打包部署到线上时，worker 执行时报错:

```
n is not defined
```

检查打包后的代码：

```js
onmessage = function(e) {
		// ...
    var s = Object(n.a)(i, ["equipmentId", "uid"]);
  	// ...
    postMessage();
}
```

第一反应是作用域污染问题，所以通过 webpack.splitChunks 将 worker 文件和其它文件分隔开。

```js
(this["webpackJsonpgraduation-design"] = this["webpackJsonpgraduation-design"] || []).push([[1], {
    449: function(e, a, t) {
        "use strict";
        var n = t(89)
          , i = function() {
            onmessage = function(e) {
               var s = Object(n.a)(i, ["equipmentId", "uid"]);
                postMessage(a)
            }
        }
        .toString();
        i = i.substring(i.indexOf("{") + 1, i.lastIndexOf("}"));
        var s = new Blob([i],{
            type: "application/javascript"
        })
          , u = URL.createObjectURL(s);
        a.a = u
    }
}]);
//# sourceMappingURL=workerGroup~admin~user.025d2179.chunk.js.map
```

发现没有解决问题，通过动态创建 script 文件后，作用域问题还是存在。所以去检查了一下罪魁祸首，是源代码中的一个对象解构打包后被 babel 编译成了一个依赖外部 module 的代码。

```js
// 打包前
const {equipmentId, uid, ...rest} = obj;

// 打包后
var s = Object(n.a)(i, ["equipmentId", "uid"]);
// n 是上层作用域传递过来的变量
```

然后改写了一下对象解构，通过其它的形式实现，最终解决了该问题。