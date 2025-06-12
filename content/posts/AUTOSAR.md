+++
title = 'AUTOSAR'
date = 2024-09-10T20:10:19+08:00
categories = ['Project']
tags = ['Embedded']
draft = false
+++

> AUTOSAR学习指南

<!--more-->

> 文档阅读指南

| 缩写   | 详细含义                           | 解释                       |
| ------ | ---------------------------------- | -------------------------- |
| `EXP`  | Explation                          | 解释说明性的文档，优先阅读 |
| `RS`   | Requirement Specification          | 详细描述需求规范           |
| `SRS`  | Software Requirement Specification | 软件需求规范               |
| `SWS`  | Software   Specification           | 软件规范                   |
| `TPS`  | Template Specification             | 模板规范                   |
| `TR`   | Technical Report                   | 技术规范详细介绍           |
| `MOD`  | Model                              | 建模                       |
| `MMOD` | Meta Model                         | 元模型                     |

---

```mermaid
---
config:
  kanban:
    ticketBaseUrl: 'https://mermaidchart.atlassian.net/browse/#TICKET#'
---
kanban
  [Application Layer]
    [PowerTrain]
    [Body And Comfort]
  [Runtime Environment]
  [Services]
    [Memory Services]
    [Crypto Services]
    [Off Board Communication  Services]
    [Communication  Services]
  [Abstraction]
    [Onboard Device Abstraction]
    [Memory Hardware Abstraction]
    [Crypto Hardware Abstraction]
    [Wireless Communication Hardware Abstraction]
    [Communication Hardware Abstraction]
    [I/O Hardware Abstraction]
  [Drivers]
    [Microcontroller Drivers]
    [Memory Drivers]
    [Crypto Drivers]
    [Wireless Communication Drivers]
    [Communication Drivers]
    [I/O Drivers]
    [Complex Drivers]
  [Microcontroller]

```