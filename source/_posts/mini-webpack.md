---
title: 实现一个不到 100 行的 mini-webpack
date: 2021-04-10 21:10:23
categories:
  - Webpack
---

我们知道 webpack 是一个 javascript 的模块打包器，他可以将很多具有依赖关系的文件，打包成 js 文件，使其能在浏览器中运行

### Webpack 打包结果分析

我们先来看个例子：

```javascript
// world.js
export const world = "world";
```

```javascript
// message.js
import { world } from "./world.js";
export const message = `hello ${world}`;
```

```javascript
// index.js
import { message } from "./message.js";
console.log(message);
```

有三个 js 文件，我们使用 webpack 先来打包一下，得到的结果大致是这样的一个立即执行函数，里面自定义一个`__webpack_require__`方法，使得我们的模块化在浏览器中能够正常运行，除此之外还有`__webpack_modules__` 里面存储了包括入口文件之外依赖的两个文件`world.js`和`message.js`的内容，那么它是怎么做到的呢，这就需要用到 babel 来实现了， 接下来我们一步步来实现

```javascript
(() => {
  // webpackBootstrap
  "use strict";
  var __webpack_modules__ = {
    "./index.js": (
      __unused_webpack_module,
      __webpack_exports__,
      __webpack_require__
    ) => {
      eval(/*....*/);
    },

    "./message.js": (
      __unused_webpack_module,
      __webpack_exports__,
      __webpack_require__
    ) => {
      eval(/*....*/);
    },

    "./world.js": (
      __unused_webpack_module,
      __webpack_exports__,
      __webpack_require__
    ) => {
      eval(/*....*/);
    },
  };

  // The module cache
  var __webpack_module_cache__ = {};

  // The require function
  function __webpack_require__(moduleId) {
    // Check if module is in cache
    if (__webpack_module_cache__[moduleId]) {
      return __webpack_module_cache__[moduleId].exports;
    }
    // Create a new module (and put it into the cache)
    var module = (__webpack_module_cache__[moduleId] = {
      // no module.id needed
      // no module.loaded needed
      exports: {},
    });

    // Execute the module function
    __webpack_modules__[moduleId](module, module.exports, __webpack_require__);

    // Return the exports of the module
    return module.exports;
  }

  /* webpack/runtime/define property getters */

  /* webpack/runtime/hasOwnProperty shorthand */

  /* webpack/runtime/make namespace object */

  /************************************************************************/
  // startup
  // Load entry module
  __webpack_require__("./index.js");
  // This entry module used 'exports' so it can't be inlined
})();
```

### 获取文件中的依赖模块和代码转换

我们需要用到 babel 中的四的包

`@babel/core`: 将代码转成指定 ES 版本的代码

`@babel/parser`: 将代码转成 AST 抽象语法输

`@babel/traverse`: 是一个 AST 遍历工具，这里我们用他来遍历使用 import 的模块

`@babel/preset-env` : 这个是 babel 的默认的一些配置项

```
// 生成依赖和转义代码
function generateDepsAndCode(filename){
    const content =  fs.readFileSync(filename, 'utf-8')
    const ast = parser.parse(content, {
        sourceType: 'module'//babel官方规定必须加这个参数，不然无法识别ES Module
    })
    const dependencies = {}
    //遍历AST抽象语法树
    traverse(ast, {
        //获取通过import引入的模块
        ImportDeclaration({node}){
            const dirname = path.dirname(filename)
            const newFile = './' + path.join(dirname, node.source.value)
            //保存所依赖的模块
            dependencies[node.source.value] = newFile
        }
    })

    //通过@babel/core和@babel/preset-env进行代码的转换
    const {code} = babel.transform(content, {
        presets: ["@babel/preset-env"]
    })
    return {
        filename,//该文件名
        dependencies,//该文件所依赖的模块集合(键值对存储)
        code //转换后的代码
    }
}
```

这一步我们就获得了`./index.js`文件的依赖文件是 `./message.js`,而且我们 import 的代码被转成了 require 了

```
{
  filename: './index.js',
  dependencies: { './message.js': './message.js' },
  code: '"use strict";\n' +
    '\n' +
    'var _message = require("./message.js");\n' +
    '\n' +
    'console.log(_message.message);'
}
```

### 递归遍历生成依赖图

那么 message 中依赖的文件怎么读取呢，这就需要我们进行递归遍历了

```javascript
function generateGraphsByDeps(deps) {
  const modules = [deps];

  for (let i = 0; i < modules.length; i++) {
    const module = modules[i].dependencies;
    for (let m in module) {
      const exits = modules.find((m) => m.filename === module[m]);
      if (!exits) {
        modules.push(generateDepsAndCode(module[m])); // 递归
      }
    }
  }

  const graphs = {};

  modules.forEach((module) => {
    const key = module.filename;
    graphs[key] = {
      dependencies: module.dependencies,
      code: module.code,
    };
  });

  return graphs;
}
```

生成的结果：

```json
{
  "./index.js": {
    "dependencies": { "./message.js": "./message.js" },
    "code":
      "\"use strict\";\n" +
      "\n" +
      "var _message = require(\"./message.js\");\n" +
      "\n" +
      "console.log(_message.message);"
  },
  "./message.js": {
    "dependencies": { "./world.js": "./world.js" },
    "code":
      "\"use strict\";\n" +
      "\n" +
      "Object.defineProperty(exports, \"__esModule\", {\n" +
      "  value: true\n" +
      "});\n" +
      "exports.message = void 0;\n" +
      "\n" +
      "var _world = require(\"./world.js\");\n" +
      "\n" +
      "var message = \"hello \".concat(_world.world);\n" +
      "exports.message = message;"
  },
  "./world.js": {
    "dependencies": {},
    "code":
      "\"use strict\";\n" +
      "\n" +
      "Object.defineProperty(exports, \"__esModule\", {\n" +
      "  value: true\n" +
      "});\n" +
      "exports.world = void 0;\n" +
      "var world = 'world';\n" +
      "exports.world = world;"
  }
}
```

这下我们就拥有了所有文件的依赖图了，这而且你会发现，我们 exports 出来的模块被存在了一个 exports 对象想。这个依赖图对应的就是`__webpack_modules__`

### require 函数

接下来我们只要写个 require，然后去执行 code 中的代码，那么上面的代码就可以跑通了。这里其实 require 函数的本质是执行一个模块的代码，然后将相应变量挂载到 exports 对象上

```javascript
function require(module, code) {
  var exports = {};
  (function (exports, code) {
    eval(code);
  })(exports, code);
  return exports;
}
```

我们来像 webpack 一样写一个立即函数，然后 require 一个入口文件

```javascript
function generateDistByGraphs(entry, graphs) {
  return `
        (function(graph) {
            //require函数的本质是执行一个模块的代码，然后将相应变量挂载到exports对象上
            function require(module) {
                var exports = {};
                (function(exports, code) {
                    eval(code);
                })(exports, graph[module].code);
                return exports;
            }
            require('${entry}')
        })(${JSON.stringify(graphs)})`;
}

function build(entry, dist) {
  const deps = generateDepsAndCode(entry);
  const graphs = generateGraphsByDeps(deps);
  console.log(graphs);

  const code = generateDistByGraphs(entry, graphs);
  fs.writeFileSync(dist, code); // 将生成的代码写入dist文件
}

build("./index.js", "./dist.js");
```

你可以试试将打包出来的 dist 放到浏览器中执行以下看是否能正确的输出`hello world`

至此我们就完成了一个简单的 mini-webpack，可以打包多个 js 文件了，虽然离真正的 webpack 还差很远，loader、plugins，这些我们都没有，但是通过这个 mini-webpack，相信你至少能清晰的知道 webpack 内部是怎么构建依赖图，怎么将多个模块文件打包再一起了。
