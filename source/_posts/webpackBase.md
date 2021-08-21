---
title: webpack 基础
subtitle: Webpack 核心概念，Webpack 调优，Webpack 运行原理
categories: webpack
tags: webpack
---

<!--
  name: webpack 基础
  description: Webpack 核心概念，Webpack 调优，Webpack 运行原理
-->

# Webpack 基础

## 配置项

1. 入口(entry)

- 入口是 webpack 构建开始的地方，通过入口文件，webpack 可以找到入口文件所依赖的文件，并逐步递归，找出所有依赖的文件。
- 可指定一个入口起点（或多个入口起点

```js
module.exports = {
  /**
   * entry: {
   *  file1: './path/to/file1.js',
   *  file2: './path/to/file2.js',
   * }
   * */
  entry: "./path/to/file.js",
};
```

2. 出口(output)

- output 属性告诉 webpack 在哪里输出它所创建的 bundles，以及如何命名这些文件，默认值为 ./dist
- output.path 必须是绝对路径

```js
module.exports = {
  entry: './path/to/file.js',
  output: {
    // webpack默认输出路径为dist
    path: path.resolve(__dirname, 'dist')
    filename: '[name].js'
  }
}
```

3. loader

- 本质上，webpack loader 将所有类型的文件，转换为应用程序的依赖图（和最终的 bundle）可以直接引用的模块。
- loader 其实就是一个 function，接收一个参数 source，就是当前的文件内容，然后稍加处理，就可以 return 出一个新的文件内容

```js
// sampleLoader.js
module.exports = function () {
  this.callback(null, "console.log('sampleLoader worked')" + source);

  // 或者 return  "console.log('sampleLoader worked')" + source
};
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\*.js$/,
        loader: "path/to/sampleLoader",
      },
    ],
  },
};
```

4. 插件(plugins)

- 通过监听 webpack 执行流程上的钩子函数，可以更精密地控制 webpack 的输出，包括：打包优化、资源管理和注入环境变量等

```js
class SamplePlugin {
  apply(compiler) {
    compiler.hooks.compilation.tap("SamplePlugin", (compilation) => {
      // compilation.hooks列表
      // https://webpack.js.org/api/compilation-hooks/#root
      compilation.hooks.afterOptimizeChunkAssets.tap(
        "SamplePlugin",
        (chunks) => {
          //  这边拿到chunk实例，进行更多操作
          console.log("SamplePlugin worked", chunks);
        }
      );
    });
  }
}
// webpack.config.js
module.exports = {
  plugins: [new SamplePlugin()],
};
```

## 优化

- 常规优化

1. 在处理 loader 时，配置 include，缩小 loader 检查范围。

2. 使用 alias 可以更快地找到对应文件。

3. 如果在 require 模块时不写后缀名，默认 webpack 会尝试.js,.json 等后缀名匹配，配置 extensions，可以让 webpack 少做一点后缀匹配。

4. thread-loader 可以将非常消耗资源的 loaders 转存到 worker pool 中。

5. 使用 cache-loader 启用持久化缓存。使用 package.json 中的 postinstall 清除缓存目录。

6. 使用 mode 中的 noParse 属性，可以设置不必要的依赖解析，例如：我们知道 lodash 是无任何依赖包的，就可以设置此选项，缩小文件解析范围。

- 开发阶段

1. 选择合理 devtool，在大多数情况下，cheap-module-eval-source-map 是最好的选择。
2. 可以直接引用 cdn 上的库文件，使用 externals 配置全局对象，避免打包。

- 生成环境

1. cdn 静态资源
2. [tree shaking](https://webpack.docschina.org/guides/tree-shaking/) + sideEffects

- 通常用于描述移除 JavaScript 上下文中的未引用代码(dead-code)
- 在一个纯粹的 ESM 模块世界中，很容易识别出哪些文件有副作用。然而，我们的项目无法达到这种纯度，所以，此时有必要提示 webpack compiler 哪些代码是“纯粹部分”。
- 通过 package.json 的 "sideEffects" 属性作为标记，向 compiler 提供提示，表明项目中的哪些文件是 "pure(纯正 ES2015 模块)"，由此可以安全地删除文件中未使用的部分。
- 如果所有代码都不包含副作用，我们就可以简单地将该属性标记为 false，来告知 webpack 它可以安全地删除未用到的 export。
- 源码必须采用 ES6 模块化语句，不然它将无法生效

```js
// sample.js
export function methodA(){
  console.log('methodA run')
}
export function methodB(){
  console.log('methodB run')
}

// webpack.config.js
module.exports = {
  ...
  mode: 'development',
  optimization: {
   usedExports: true,
  },
  module: {
    rules: [
      {
        test: ...
        // 在规则中添加 sideEffects
        sideEffects: true
      }
    ]
  }
}

// package.json
{
  name: 'my-project',
  //
  sideEffects: boolean | 'path/to/target/file'
}

// src/index.js 入口文件
import {methodA} from 'sample'
methodA()
```

3. 配置 (scope hoisting)[https://webpack.docschina.org/plugins/module-concatenation-plugin/] 作用域提升，将多个 IIFE 放在一个 IIFE 中。

- 原理： 分析出模块之间的依赖关系，尽可能的把打散的模块合并到一个函数中去，但前提是不能造成代码冗余。 因此只有那些被引用了一次的模块才能被合并。
- 源码必须采用 ES6 模块化语句，不然它将无法生效
