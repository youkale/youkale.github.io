---
title: linux监控相关
date: 2019-02-13 15:15:50
tag:
---

## 端口连接情况

    lsof -i :6379

## 网卡速率

    sar -n DEV 1

## IO监控

    iostat -x -t 1

## 综合监控

    dstat -cdlmnpsy
