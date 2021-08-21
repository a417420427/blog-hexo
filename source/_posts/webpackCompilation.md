---
title: webpack构建流程
subtitle: webpack构建流程解析
categories: webpack
tags: webpack
---

## 整体流程

1. 开始运行 Webpack。

- 读取与合并参数，加载 plugin。
- 实例化 Compiler(编译器)。

```js
// webpack 5 部分源码解读
const webpack = (options, callback) => {
  const create = () => {
    // 检查，读取与合并参数
    if (!webpackOptionsSchemaCheck(options)) {
      getValidateSchema()(webpackOptionsSchema, options);
    }
    let compiler;
    let watch = false;
    let watchOptions;
    if (Array.isArray(options)) {
      compiler = createMultiCompiler(options, options);
      watch = options.some((options) => options.watch);
      watchOptions = options.map((options) => options.watchOptions || {});
    } else {
      const webpackOptions = options;
      // 创建 编译器 实例
      compiler = createCompiler(webpackOptions);
      watch = webpackOptions.watch;
      watchOptions = webpackOptions.watchOptions || {};
    }
    return { compiler, watch, watchOptions };
  };
  if (callback) {
    try {
      const { compiler, watch, watchOptions } = create();
      if (watch) {
        // 开始编译 并监听
        compiler.watch(watchOptions, callback);
      } else {
        // 开始编译
        compiler.run((err, stats) => {
          compiler.close((err2) => {
            callback(err || err2, stats);
          });
        });
      }
      return compiler;
    } catch (err) {
      process.nextTick(() => callback(err));
      return null;
    }
  } else {
    const { compiler, watch } = create();
    if (watch) {
      util.deprecate(
        () => {},
        "A 'callback' argument needs to be provided to the 'webpack(options, callback)' function when the 'watch' option is set. There is no way to handle the 'watch' option without a callback.",
        "DEP_WEBPACK_WATCH_WITHOUT_CALLBACK"
      )();
    }
    return compiler;
  }
};

// 创建编译器
const createCompiler = (rawOptions) => {
  const options = getNormalizedWebpackOptions(rawOptions);
  applyWebpackOptionsBaseDefaults(options);
  const compiler = new Compiler(options.context);
  compiler.options = options;
  // 对文件做了处理，重新封装node.js 对fs模块做了以一些处理，文件的输入，输出，缓存，监听等
  new NodeEnvironmentPlugin({
    infrastructureLogging: options.infrastructureLogging,
  }).apply(compiler);
  // 加载plugin, 将 compiler 实例 传给每个plugin
  if (Array.isArray(options.plugins)) {
    for (const plugin of options.plugins) {
      if (typeof plugin === "function") {
        plugin.call(compiler, compiler);
      } else {
        plugin.apply(compiler);
      }
    }
  }
  // 应用参数 context devtools target mode ...
  applyWebpackOptionsDefaults(options);
  // 触发hook 在准备环境之前运行插件。
  compiler.hooks.environment.call();
  // 触发hook 执行插件环境设置完成。
  compiler.hooks.afterEnvironment.call();
  // 应用各类中间件
  new WebpackOptionsApply().process(options, compiler);
  // 触发hook 编译器对象被初始化
  compiler.hooks.initialize.call();
  return compiler;
};
```

- compiler.run 开始编译(这里先分享单入口的情况)

```javascript
// 1. 编译路径 起始 (webpack.js)
compiler.run();
// 2. 触发 compiler的entryOption hook
// WebpackOptionsApply 中调用 EntryOptionPlugin().apply()之后设置的hook EntryOptionPlugin.js
compiler.hooks.entryOption.tap("EntryOptionPlugin", (context, entry) => {
  EntryOptionPlugin.applyEntryOption(compiler, context, entry);
  return true;
});
// 3. 触发 compiler的 make hook EntryPlugin.js
// compiler.compile 内设置的hook
compiler.hooks.make.tapAsync("EntryPlugin", (compilation, callback) => {
  compilation.addEntry(context, dep, options, (err) => {
    callback(err);
  });
});
// 4. 通过(Compilation.js)addEntry => _addEntryItem => addModuleTree=> handleModuleCreation => addModule => (NormalModule.js)build
// NormalModule.build 中调用 doBuild方法 然后将代码经过 runLoaders方法 进行转译
NormalModule.build = function (args, callback) {
  // ...
  return this.doBuild(args, () => {
    // ...
    try {
      result = this.parser.parse();
      handleBuildDone();
      return callback();
    } catch (e) {
      handleParseError(e);
    }
  });
};

NormalModule.doBuild = function (args, callback) {
  runLoaders(args, () => {
    return callback();
  });
};

// 5. 然后通过parser.parse方法将代码转成ast语法树， 然后接下来对所依赖的对象进行收集

//6. 做Compilation 通过回调中调用 Compilation.seal方法根据依赖关系生成结果代码
```

-

2. 使用 Parser 分析项目依赖。
3. 使用 Template 生成结果代码。

## 通过构建 plugin 监听上述各个步骤

```js
class WatchModulePlugin {
  constructor(props) {}
  /**
   * @param {Webpack.Compiler} compiler
   */
  apply(compiler) {
    const pluginName = "WatchModulePlugin";
    const hooks = [
      "environment",
      "afterEnvironment",
      "initialize",
      "compilation",
      "make",
      "run",
      "entryOption",
      "normalModuleFactory",
    ];
    const compilationHooks = [
      "addEntry",
      "seal",
      "buildModule",
      "finishModules",
      "beforeModuleIds",
    ];
    const _this = this;

    function logHook(hook) {
      console.log("hooks-", hook);
    }

    function logCompilationHook(compilation) {
      compilationHooks.forEach((hook) => {
        compilation.hooks[hook].tap(pluginName, function () {
          console.log("hooks-compilation-" + hook);
        });
      });
    }

    hooks.forEach(function (hook) {
      compiler.hooks[hook].tap("WatchModulePlugin", function (compilation) {
        if (hook === "compilation") {
          logCompilationHook(compilation);
        }
        logHook(hook);
      });
    });
  }
}

/**
 * 输出
hooks- environment
hooks- afterEnvironment
hooks- entryOption
hooks- initialize
hooks- run
hooks- normalModuleFactory
hooks- compilation
hooks- make
hooks-compilation-addEntry
hooks-compilation-buildModule
hooks-compilation-buildModule
hooks-compilation-finishModules
hooks-compilation-seal
 * 
*/
```
