---
title: 快速入门
summary: ZIGGO Device 快速入门文档
date: 2024-05-01
authors:
  - admin
tags:
  - ZIGGO-Device
  - Basic
image:
  
weight: 982
---
# 快速入门
## 目录

- [快速入门](#快速入门)
  - [目录](#目录)
  - [文件结构](#文件结构)
  - [Git 分支介绍](#git-分支介绍)
  - [构建和启动 TSNPerf](#构建和启动-tsnperf)


## 文件结构

```bash
.
├── docs                     // ZIGGO 的文档
├── figs                     // 文档的图表
├── hardware                 // 硬件代码，PL 部分（pkt_gen，FPGA 中的时间同步）
├── readme.md                // 整个项目的自述文件
├── software                 // 软件代码，PS 部分（pkt_gen，时间同步）
└── testbed-build            // 构建测试平台的代码
```

## Git 分支介绍

| 分支名称        | 基本功能  | 包大小调整 | 在线分析 | 离线分析  |
|:---------------:|:---------:|:----------:|:--------:|:---------:|
| main            | √         |            | √        |           |
| packet_resize   | √         | √          | √        |           |
| offline_analyze | √         |            |          | √         |

此次提交的 TSNPerf 分支可分为三类。但 main 分支在两个方面不便使用。为了解决上述两种限制，我们还提交了两个补丁分支。

* `main` 分支包含基本的时间同步和 pkt_gen 功能，以及在线分析功能。然而，其限制是只能发送 MTU（1500B）大小的以太网，且由于 zynq 开发板性能有限，在线分析可能会导致丢包。
* `packet_resize` 分支可以发送任意大小的以太网帧（64B-1500B）。
* `offline_analyze` 分支可以将数据包转发到 PC 进行离线分析，以避免丢包。

> 注意1：切换到 'packet_resize' 分支时，还需要修改 `software/config/flow.json` 
> 
> ```
> {
>         "job_id": 0, 
>         "flow_id": 0,
>         "src": 1,
>         "dst": 2,
>         "period": 2048,
>         "MD": 1024,
>         "packet_size": 750 // 添加此属性 
>  },
> ```

> 注意2：切换到 'offline_analyze' 分支时，需要通过网线将设备连接到另一台 PC（Linux），使用 ETH1 接口。

## 构建和启动 TSNPerf

[设置 FPGA 板并初始化 PS 系统](/ziggo_book/zh/docs/device/hardware-build/)

[编译软件代码，运行时间同步和 pkt_gen_app](/ziggo_book/zh/docs/device/software-build/)