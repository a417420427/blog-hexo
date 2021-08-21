---
title: 利用 github actions 进行自动部署
categories: git
tags: 自动部署 Actions
---

<!--
  name: 利用github actions进行自动部署
  description: GitHub Actions 帮助您自动完成软件开发周期内的任务。 GitHub Actions 是事件驱动的，意味着您可以在指定事件发生后运行一系列命令。 例如，每次有人为仓库创建拉取请求时，您都可以自动运行命令来执行软件测试脚本。
-->

概述： GitHub Actions 帮助您自动完成软件开发周期内的任务。 GitHub Actions 是事件驱动的，意味着您可以在指定事件发生后运行一系列命令。 例如，每次有人为仓库创建拉取请求时，您都可以自动运行命令来执行软件测试脚本。

## 使用步骤

1. 创建项目/使用已有的 github 项目
2. 找到 action tap， 选择 new workflow

3. 选择一个需要创建的 workflow 类型(这里我直接使用的 Deno， 如果只需要使用 webpack 也可以选择 webpack)

4. 然后他会给你一个模板

```yml
# This workflow uses actions that are not certified by GitHub.
# They are provided by a third-party and are governed by
# separate terms of service, privacy policy, and support
# documentation.

# This workflow will install Deno then run Deno lint and test.
# For more information see: https://github.com/denoland/setup-deno

name: Deno

on:
  #  action 执行时间
  # 推送后执行 [分支名]
  push:
    branches: [master]
  # 拉取后执行 [分支名]
  pull_request:
    branches: [master]

jobs:
  test:
    # 运行系统
    runs-on: ubuntu-latest
    # 执行步骤
    steps:
      # 步骤名称
      - name: Setup repo
        # 步骤内容
        uses: actions/checkout@v2

      - name: Setup Deno
        # uses: denoland/setup-deno@v1
        uses: denoland/setup-deno@004814556e37c54a2f6e31384c9e18e9833173669
        with:
          deno-version: v1.x

      # Uncomment this step to verify the use of 'deno fmt' on each commit.
      # - name: Verify formatting
      #   run: deno fmt --check

      - name: Run linter
        run: deno lint

      - name: Run tests
        run: deno test -A --unstable
```

5. 根据自己的情况,增加或修改 action 的步骤即可, 我这边增加了一个 node 版本的选择+安装依赖+打包 3 个步骤

```yaml
 - name: Use Node.js ${{ matrix.node-version }}
    uses: actions/setup-node@v1

  - name: Install dependencies
    run: npm install

  - name: build package
    run: make default
```

6. 提交 pr， 然后查看 action 是否生效
