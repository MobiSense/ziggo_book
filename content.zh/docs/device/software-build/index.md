---
title: 软件构建
summary: 启用 CaaS Device 的时间同步逻辑和设置数据包生成计划的文档
date: 2024-05-01
authors:
  - admin
tags:
  - ZIGGO-Device
  - ZIGGO
  - Markdown
image:
  
weight: 985
---
# 软件构建

此仓库包含启用 TSN/CaaS TSNPerf 时间同步逻辑和设置数据包生成计划的源代码。

## 目录

- [软件构建](#软件构建)
  - [目录](#目录)
  - [1. 构建](#1-构建)
  - [2. 配置](#2-配置)
  - [3. 运行](#3-运行)
  - [4. 分析](#4-分析)
    - [4.1 TSNPerf 直接分析延迟和抖动](#41-tsnperf-直接分析延迟和抖动)
    - [4.2 离线分析延迟和抖动](#42-离线分析延迟和抖动)

## 1. 构建

```bash
mkdir build
cd build
cmake ..
make
```

构建成功后，应生成两个可执行文件："time_sync" 和 "pkt_gen"

## 2. 配置

"build" 目录包含拓扑和 TSN/CaaS 调度结果。

* config.json: 拓扑文件，包含节点信息（类型、MAC、PTP 端口）、节点之间的链路（带端口）和转发表。
* schedule.json: 调度文件，包含每个链路的调度时间间隔和每个 CaaS 交换机的计算时间间隔。

（提示：这类似于一个 Ziggo 交换机）

## 3. 运行

* 启动时间同步

```bash
./time_sync
```

* 启动设备关键数据包生成

```bash
./pkt_gen
```

## 4. 分析

TSNPerf 有两种分析方法：

> 提示：如果您想使用离线分析，请切换到 'offline_analyze' 分支，并且需要通过网线将设备连接到另一台 PC（Linux）。

一种方法是在 TSNPerf 上直接分析延迟和抖动。

另一种方法是使用设备对接收的数据包进行时间戳标记，并通过另一个端口转发到功能强大的台式计算机进行数据包捕获分析。即使以千兆位速率进入设备，也不会丢包。缺点是需要离线编写单独的程序来分析数据包的延迟和抖动。

### 4.1 TSNPerf 直接分析延迟和抖动

在 `time sync` 程序运行期间，接收到测试数据帧后，程序会将数据帧的延迟信息保存在 `build/packet_log.csv` 文件下，同时将延迟和抖动的批量统计信息保存在 `build/critical_log.csv` 文件下。

您可以使用以下命令显示标题和最后 5 行的内容：

```bash
cat critical.log | head -n1
cat critical.log | tail -n5
```

### 4.2 离线分析延迟和抖动

如果进行离线分析，需要注意使用的硬件位于 `offline_analyze` 分支。此版本会将关键数据标记为接收时间戳，并从 ETH2 转发。我们需要使用一台装有 Linux 系统的 PC 来捕获数据包进行分析。

数据包捕获命令示例：

```bash
sudo tcpdump -i enx207bd272812b ether src 00:0a:35:00:00:14 -n -B 100000 -w packets.pcapng
```

`Enx207bd272812b` 是网卡名称，可以通过 ifconfig 获取。`Src` 用于指定捕获的数据包来自某个 MAC 地址。`-B` 用于指定缓冲区大小。如果需要设置更大的缓冲区大小以避免丢包，可以查看数据包捕获后的输出信息，查看 Dropped by kernel 是否为 0。如果不是 0，则表示内核中存在丢包。`-W` 指定文件的保存路径。

用于分析程序的代码是 `analyze_packet.py`

```bash
python ./analyze_packet.py [capture file path] --step [interval]
```

同时，将在当前目录生成一个数据包（`packet_log.csv` 和 `critical_log.csv`）。