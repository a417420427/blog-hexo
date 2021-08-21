---
title: 利用 github issues 实现评论系统
categories: git
tags: 评论系统
---

<!--
  name: 利用github issues实现评论系统
  description: 利用github的issues作为数据库，构建一个评论系统
-->

## github issues

- github 官网关于 issue 功能的介绍
- Issues 是一种非常好的跟踪你项目中任务、改善和 bug 的方式。它们某种程度上类似于邮件，但是它们可以与团队中其他人分享和讨论。
- 功能

1. title 和 description：标题和描述
2. labels：标签，用于分类
3. milestone：时间点
4. assignee：指定
5. Comments：评论
6. Notifications, @mentions, and References
7. Notifications：消息提醒
8. @mentions： 提到他人。github 建议使用/cc，即 carbon copy，概念抄送。
9. References： 依赖的其他 issue、pr 等

- 基于每个 issues 制作一条评论

## 实现

1. 创建 OAuth applications， 评论需要涉及 GitHub 授权登录，所以在这里你先要有一个 GitHub application
