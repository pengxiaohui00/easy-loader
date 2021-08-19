
1、背景
首先我们来看一下为什么需要loader，以及他能干什么?
webpack 只能理解 JavaScript 和 JSON 文件。loader 让 webpack 能够去处理其他类型的文件，并将它们转换为有效模块，以供应用程序使用，以及被添加到依赖图中。

本质上来说，loader 就是一个 node 模块，这很符合 webpack 中「万物皆模块」的思路。既然是 node 模块，那就一定会导出点什么。在 webpack 的定义中，loader 导出一个函数，loader 会在转换源模块resource的时候调用该函数。在这个函数内部，我们可以通过传入 this 上下文给 Loader API 来使用它们。最终装换成可以直接引用的模块。

2、xml-Loader 实现
前面我们已经知道，由于 Webpack 是运行在 Node.js 之上的，一个 Loader 其实就是一个 Node.js 模块，这个模块需要导出一个函数。 这个导出的函数的工作就是获得处理前的原内容，对原内容执行处理后，返回处理后的内容。
一个简单的loader源码如下

module.exports = function(source) {
  // source 为 compiler 传递给 Loader 的一个文件的原内容
  // 该函数需要返回处理后的内容，这里简单起见，直接把原内容返回了，相当于该 Loader 没有做任何转换
  return source;
};
由于 Loader 运行在 Node.js 中，你可以调用任何 Node.js 自带的 API，或者安装第三方模块进行调用：

const xml2js = require('xml2js');
const parser = new xml2js.Parser();

module.exports =  function(source) {
  this.cacheable && this.cacheable();
  const self = this;
  parser.parseString(source, function (err, result) {
    self.callback(err, !err && "module.exports = " + JSON.stringify(result));
  });
};
这里我们事简单实现一个xml-loader;

注意：如果是处理顺序排在最后一个的 loader，那么它的返回值将最终交给 webpack 的 require，换句话说，它一定是一段可执行的 JS 脚本 （用字符串来存储），更准确来说，是一个 node 模块的 JS 脚本，所以我们需要用module.exports =导出。

整个过程相当于这个 loader 把源文件

// 这里是 source 模块
转化为

// example.js
module.exports = '这里是 source 模块';
然后交给 require 调用方：

// applySomeModule.js
var source = require('example.js'); 
console.log(source); // 这里是 source 模块
写完后我们要怎么在本地验证呢？下面我们来写个简单的demo进行验证。

d

