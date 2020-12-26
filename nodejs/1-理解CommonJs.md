# 理解 CommonJS

## 前言

最近在看 nodejs，这篇文章的内容，绝大部分都是参考 [node文档](https://nodejs.org/docs/latest/api/modules.html) ，主要是帮助自己理解记忆相关知识点，同时也便于今后查阅。如果能帮助到其他朋友，那更是荣幸至极了。

在 nodejs 模块系统中，一个文件就是一个独立的模块。在模块中定义的变量都是私有的，模块要对外暴露对象或者方法，就在模块内的 `module.exports` 或者 `exports`上添加属性，然后在需要使用该模块的地方用 `require` 引入。

## 为什么模块内定义的变量是私有的？

模块内的代码在执行之前，会被放到函数中，因此其中的定义的变量都变成了局部变量。

```
(function(exports, require, module, __filename, __dirname) {
// Module code actually lives in here
});
```

## exports 和 module.exports

### module.exports

`module.exports` 是模块导出的对象。在 module 内，在 `module.exports` 上添加 props 或者直接赋值，都能顺利导出。

添加 props：

```
module.exports.hi = 'Hi'
module.exports.sayHi = function() { console.log('Hi!') }
```

直接赋值：

```
module.exports = {
	hi: 'Hi!'
}
```

```
module.exports = function() { console.log('Hi!') }
```

再贴一个将实例赋值给 `module.exports` ，并且在模块内触发事件的例子：

module：

```
const EventEmitter = require('events');

module.exports = new EventEmitter();

// Do some work, and after some time emit
// the 'ready' event from the module itself.
setTimeout(() => {
  module.exports.emit('ready');
}, 1000);
```

require：

```
const a = require('./a');
a.on('ready', () => {
  console.log('module "a" is ready');
});
```

注意：

`module.exports` 的赋值不能放在回调中（`exports` 也一样），这样是错的：

```
setTimeout(() => {
  module.exports = { a: 'hello' };
}, 0);
```
### exports

`exports` 是 module 内的一个变量，就如同写了一句：

```
exports = module.exports
```

因此，如果直接在 `exports` 或者 `module.exports` 上添加 props，其效果是一样的；但是如果直接给 `exports` 赋值，它就不再指向 `module.exports`，所以也无法暴露相应的值了。

正如 node 文档示例那样：

```
module.exports.hello = true; // Exported from require of module
exports = { hello: false };  // Not exported, only available in the module
```

## require

> To illustrate the behavior, imagine this hypothetical implementation of require(), which is quite similar to what is actually done by require().

```
function require(/* ... */) {
  const module = { exports: {} };
  ((module, exports) => {
    // Module code here. In this example, define a function.
    function someFunc() {}
    exports = someFunc;
    // At this point, exports is no longer a shortcut to module.exports, and
    // this module will still export an empty default object.
    module.exports = someFunc;
    // At this point, the module will now export someFunc, instead of the
    // default object.
  })(module, module.exports);
  return module.exports;
}
```

require 的实现和上面这段代码类似，就不多赘述这一块了，理解代码就行。

### require 加载模块的逻辑

nodejs文档中给出了 require 加载的 [伪代码](https://nodejs.org/docs/latest/api/modules.html#modules_all_together)，为了便于查看，同时也理一下思路，我把伪代码转换成了脑图：

![image-20201226153328814](imgs/image-20201226153328814.png)

截图看不清楚，脑图的文件放在了[我的Github](https://github.com/bing321/blog/blob/main/nodejs/source/require_module.xmind)中（我画的可能有纰漏，欢迎指正），需要的朋友自取。

上面的脑图比较翔实的描述了`require`的加载逻辑，下面把常用的`require`方式总结一下。

#### case1：不带任何符号的模块

下面标的序号，都是上一步没有找到文件的情况下，继续寻找的步骤。

```
require("http")  // 不带任何符号的文件名
require("mycustom.js")  // 不带任何符号的文件名
```

1. 优先从 node 内置模块中查找
2. 内置模块没有，就从当前目录的 node_modules 中找
3. 从父目录中找，一直找到文件系统根目录（找到 `/node_modules/mycustom.js`）为止

#### case2：带`/`，`./`，`../`等符号

```
require("./custom")
```

1. 先找名为 `custom` 的文件

2. 再找 `custom.js`文件

3. 再找`custom.json`文件

4. 最后找`custom.node`文件

5. 找`custom/package.json`文件，再找其中 main 模块中指定的文件

   ```
   {
   	"name" : "some-library",
   	"main" : "./lib/some-library.js"
   }
   ```

6. 找`custom/index.js`文件

7. 找`custom/index.json`文件

8. 找`custom/index.node`文件

## 结语

文章写道这里，我长舒一口气，总算把主要部分理完了，我也对node的模块系统有了新的认识，算是小有收获。

其他的内容，如 require.cache，require 的循环引用，以及其他的属性及注意点，本文就不再赘述了，遇到问题再查[文档](https://nodejs.org/en/)吧！因为我实在想不出 require.cache 和 循环引用的使用场景是什么（也因为整理上面的内容花了太多精力），就不再花费精力去整理了（逃。

以上就是本文的全部内容，如果文章对你有所帮助，我感到非常荣幸！也欢迎指出本文中的纰漏！

写文不易，转载请注明出处。

