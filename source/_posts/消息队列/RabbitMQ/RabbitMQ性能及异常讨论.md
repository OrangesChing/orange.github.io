---
title: RabbitMQ性能及异常讨论
date: 2020-09-15 17:20:26
tags:
  - RabbitMQ
categories:
  - 消息队列
  - RabbitMQ
---

# 性能讨论

## 如何创建队列

- 队列影响QPS，交换器不影响
- 预先分配创建资源的静态方式
- 动态创建方式