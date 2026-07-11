---
sidebar_position: 25

doc_kind: page
source_of_truth: local
title: "Kiln：主线内核上的 NPU（LLM + 视觉）"
description: "Kiln 让 ROCK 4D 在主线 Linux 内核上运行厂商 RKLLM/RKNN NPU 栈：一条命令在 Armbian 上装好本地大模型对话、图像识别，以及 OpenAI 兼容 API。"
locale: zh
board: rock4d
task_type: install
last_verified: 2026-07-11
---

# Kiln：主线内核上的 NPU（LLM + 视觉）

[Kiln](https://github.com/gahingwoo/kiln) 是一个开源社区项目，让 ROCK 4D 在**主线 Linux 内核**
（而非厂商 6.1 BSP）上运行厂商的 RKLLM / RKNN NPU 软件栈。它把厂商 GPL `rknpu` 驱动以
out-of-tree 方式编译，配上闭源的 `librkllmrt`（大模型）和 `librknnrt`（视觉）运行时，再加一小组
针对 RK3576 NPU 的内核补丁（时钟 / 电源域 / 双 IOMMU），让 NPU 在 `linux-7.1.3` 上正常工作。

装好后你会得到：

- **`kiln-chat`** —— 在 NPU 上跟本地大模型对话；
- **`kiln-vision`** —— 图像分类 / YOLO 目标检测；
- **`kiln-serve`** —— OpenAI 兼容的 HTTP API，可直接接 Open WebUI、LangChain 或任意 OpenAI 客户端；
- `kiln`（菜单）、`kiln-config`（设置）、`kiln-convert`（板上转模型）、`kiln-doctor`（健康检查）。

:::note
Kiln 是独立的第三方开源项目，不属于 Radxa 官方支持范围。问题请到其
[仓库](https://github.com/gahingwoo/kiln) 反馈。
:::

## Prerequisites

- 一块 **ROCK 4D**（RK3576），运行 **Armbian**。
- 可用网络（首次安装需要联网下载内核与运行时；以太网最稳）。
- 模型自备（Kiln 不附带）：大模型放一个 `*-rk3576-w4a16.rkllm` 到 `/opt/models`；视觉可用
  `kiln-convert` 在板上现转。

## Steps

在 Armbian 上一条命令安装：

```bash
curl -fsSL https://raw.githubusercontent.com/gahingwoo/kiln/main/scripts/kiln-install.sh | bash
```

它是**免手动**的：预下载所需内容、装上 Kiln 主线内核，然后**自己重启两次（共约 10–15 分钟）**
完成安装并进入就绪系统。这是正常的，**别断电**。两次重启之间板载 Wi-Fi 会短暂断开（安装的第二
阶段离线完成、不需要网）。装完登录即可看到欢迎信息。

想自己掌控重启，用手动模式：

```bash
KILN_MANUAL=1 bash kiln-install.sh
```

## Expected Output

装完登录后，健康检查应基本正常：

```bash
kiln-doctor
```

关键项应为 `[ OK ]`：`rknpu` 已加载、存在 `/dev/dri/renderD128`、四个 MMU bank
`st=0x19/0x19/0x19/0x19`、运行时 `librkllmrt` 1.2.0 与 `librknnrt` 2.3.0。

## Validation

视觉（用 `kiln-convert` 在板上现转一个分类器，再跑）：

```bash
kiln-convert mobilenet --set-active
kiln-vision /opt/models/test.jpg
```

大模型（把一个 `.rkllm` 放到 `/opt/models` 后）：

```bash
kiln-chat
```

OpenAI 兼容 API + Open WebUI：

```bash
kiln-serve
# 另一台机器上验证：
curl http://<board-ip>:8080/v1/models
```

`kiln-serve` 启动时会打印填好板子 IP 的连接串，直接抄给 Open WebUI 即可。

## Troubleshooting

- 先跑 `kiln-doctor`，把完整输出贴到 issue。
- 常见报错（版本锁、YOLO 导出、Wi-Fi 等）对照 Kiln 的
  [故障排查文档](https://github.com/gahingwoo/kiln/blob/main/docs/zh/TROUBLESHOOTING.md)。
- 更多说明见 Kiln 的 [中文文档](https://github.com/gahingwoo/kiln/blob/main/README.zh-CN.md)。
