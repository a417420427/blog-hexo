---
title: webpack搭建
subtitle: 从0开始搭建一个webpack+react+typescript环境
categories: webpack
tags: webpack
---

<!--
  name: webpack搭建
  description: 从0开始搭建一个webpack+react+typescript环境
-->

## 从 0 开始搭建一个 webpack+react+typescript 环境

1. 基础配置

```js
const config = {
  entry: {
    // 入口文件, 可以是相对路径或绝对路径
    main: "./src/index.js",
  },
  output: {
    // 输出的文件名
    filename: "[name].js",
    // 输入路径 必须是绝对路径
    path: path.resolve(__dirname, "../dist"),
  },
};
```

2. loaders 静态资源， css

```js
const generateLoaders = () => {
  return [
    // webpack 5 静态资源可以直接 使用type assets 设置
    {
      test: /\.(png|jpe?g|gif|ico|bmp)$/i,
      type: "asset",
      parser: {
        dataUrlCondition: {
          maxSize: 10 * 1024,
        },
      },
      generator: {
        filename: "images/[hash][ext][query]",
      },
    },
    {
      test: /\.s[ac]ss$/i,
      use: ["style-loader", "css-loader", "sass-loader"],
    },
  ];
};
```

3. ts 配置

- 增加一个 tsconfig.json 文件，它包含了输入文件列表以及编译选项

```json
{
  "exclude": ["dist", "node_modules"],
  "compilerOptions": {
    "jsx": "react",
    "module": "esnext",
    "target": "esnext",
    "sourceMap": true,
    "strict": true,
    "moduleResolution": "node",
    "forceConsistentCasingInFileNames": true,
    "noImplicitReturns": true,
    "noUnusedLocals": true,
    "noUncheckedIndexedAccess": false,
    "suppressImplicitAnyIndexErrors": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    // 没有export default的文件 可以使用 import * as  DefaultModule 的方式引入文件
    "allowSyntheticDefaultImports": true,
    // 允许引入js
    "allowJs": true,
    // 允许引入json文件
    "resolveJsonModule": true
  }
}
```

- loader 增加 ts 支持

```js
const generateLoaders = () => {
  return [
    // ...
    {
      test: /\.tsx?$/,
      use: [
        {
          loader: "babel-loader",
          options: {
            presets: ["@babel/preset-env"],
          },
        },
        "ts-loader",
      ],
    },
  ];
};
```

- 其他配置项 React ReactDOM 快捷引入(免去每个文件 import)

```js
// webpack.config.js
const config = {
  // ...
  plugins: [
    // 这里需要在ts文件内全局定义React和ReactDOM
    new webpack.ProvidePlugin({
      React: 'react',
      ReactDOM: 'react-dom',
    }),
  ],
}

// global.d.ts
declare const React: typeof import('react')
declare const ReactDOM: typeof import('react-dom')
```

- scss 配置 使用 astroturf

```js
// .babel.config.js
// https://4catalyzer.github.io/astroturf/setup
module.exports = {
  plugins: [
    [
      "astroturf/plugin",
      {
        tagName: "css",
        extension: ".scss",
        writeFiles: true,
        getFileName(hostFilePath, pluginsOptions) {
          console.log("hostFilePath", hostFilePath);
          const basepath = path.join(
            path.dirname(hostFilePath),
            path.basename(hostFilePath, path.extname(hostFilePath))
          );
          const relativePath = path.relative(__dirname, basepath);
          return `.astroturf/extracted_styles/${relativePath}.scss`;
        },
      },
    ],
  ],
};

// webpack.config.js
config.rules = [
  //...
  {
    test: /\.tsx?$/,
    use: [
      "babel-loader",
      {
        loader: "astroturf/loader",
        options: { extension: ".module.scss" },
      },
      "ts-loader",
    ],
  },
];
```

- 部分优化工作

1. splitChunks

```js
const config = {
  //...
  optimization: {
    minimize: true,
    splitChunks: {
      cacheGroups: {
        commons: {
          name: "vendors",
          // 匹配react相关的包 合并
          test: /[\\/]node_modules[\\/](react|react-.*)[\\/]/,
          chunks: "all",
        },
      },
    },
  },
};
```

2. 缩小路径，减少打包时索引时间

```js
const config = {
  // ...
  module: {
    // ...
    rules: [
      // ...
      {
        test: /\.s[ac]ss$/i,
        use: ["style-loader", "css-loader", "sass-loader"],
        // 确认是否只在src目录里面有引入scss文件
        // 如果在node_module内也引入了需要单独加上
        // 或者不设置
        include: path.join(__dirname, "../src"),
      },
    ],
  },
};
```

3. 其他

- 使用 webpack-bundle-analyzer 对具体情况进行分析，根据文件使用情况可以选择 cdn 或者合并等方式
