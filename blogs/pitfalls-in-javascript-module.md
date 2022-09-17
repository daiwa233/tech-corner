# 模块化开发中的陷阱

总结 JavaScript 模块化开发过程中遇到的 “陷阱”

## 循环依赖

模块之间循环依赖，在 ESmodule 的环境下，可能会导致 Reference Error。在诸如 webpack 这种打包工具打包后，虽然不会抛出异常，但是获得不到应有的值。例如：

```js
// test/b.js
import { testVar } from "../";
console.log("testVar", testVar);

export const b = testVar;

// test/a.js
export * from "./b.js";

export const a = 123;

// index.js 
export * from "./test/a";

export const testVar = 123;
```

[demo](https://codesandbox.io/s/pedantic-sea-g8ulk?file=/src/test/index.ts)

上述例子中，index.js 作为程序的入口，import 了 a.js， a.js 又 import 了 b.js， b.js 中引入 index.js 中的变量来使用，最终导致 b.js 中的代码出现 bug。这种循环依赖究其本质其实是互相依赖，总有一个引用的先后顺序，一旦顺序错乱，则会导致程序出现问题。

在程序中应该杜绝这种循环依赖的代码，保留程序的单一入口，同时入口内部的模块应该禁止从入口处引用模块

在写代码时应该特别注意 export all 这种写法，这种写法有一个隐式的 import 语句，可能会让我们一时想不到哪里出现了循环依赖。

```js
export * from './xxx.js'
```

