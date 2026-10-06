# Jetson Orin Nano 交接手冊

本手冊記錄實驗室這台 Jetson Orin Nano 的硬體型號、系統安裝流程，以及日常的連線與檔案傳輸操作，供接手的人直接照著使用。

- 安裝日期：2026-10-06
- 安裝版本：JetPack 7.2.1（Jetson Linux r39.2.1）

---

## 目錄

1. [硬體型號](#1-硬體型號)
2. [系統版本](#3-系統版本)
3. [切換GUI、server指令](#3-切換GUI、server指令)
4. [安裝方式](#4-安裝方式)
---

## 1. 硬體型號
| 項目 | 內容 |
|---|---|
| 產品 | NVIDIA Jetson Orin Nano Developer Kit Super |
| 記憶體 | 8GB（CPU 與 GPU 共用） |
| 模組 | P3767-0005（開發套件版，模組背面有 microSD 插槽） |
| 系統儲存 | microSD 64GB（U3 / V30 / A2） |
| 影像輸出 | DisplayPort（沒有 HDMI） |
| 電源 | 原廠 19V 變壓器，DC 圓孔 |
| 帳號 / 主機名 | `ncrl` / `ncrl` |
| 密碼 | ee405423

## 2. 系統版本
（2026-10-06 安裝完成後實測）
| 項目 | 版本 |
|---|---|
| JetPack | 7.2.1 |
| Jetson Linux（L4T） | R39.2.1 |
| Ubuntu | 24.04.4 LTS |
| Linux 核心 | 6.8.12-1021-tegra |
| 韌體 | 39.2.1（安裝前為 36.4.7），開機槽位 A |
| Python | 3.12.3 |
| CUDA 等 JetPack 元件 | **尚未安裝**，需執行 `sudo apt install nvidia-jetpack` |
| 對應的 ROS 2 版本 | Jazzy（Ubuntu 24.04），尚未安裝 |

## 3. 切換GUI、server指令
記憶體只有 8GB 且與 GPU 共用。開發時用桌面模式，上機實測時切成文字模式可多出約 1GB 記憶體。

```bash
sudo systemctl set-default multi-user.target   # 開機不進桌面
sudo systemctl set-default graphical.target    # 開機進桌面
sudo reboot（重新開機即可以啟動）
```

在文字模式下臨時啟動桌面：

```bash
sudo systemctl start gdm
```

## 4. 安裝方式
根據以下連結一步步操作
[NVIDIA 官方 Quick Start Guide](https://docs.nvidia.com/jetson/orin-nano-devkit/user-guide/latest/quick_start.html)


