# Jetson Orin Nano 交接手冊

本手冊記錄實驗室這台 Jetson Orin Nano 的硬體型號、系統安裝流程，以及日常的連線與檔案傳輸操作，供接手的人直接照著使用。

- 安裝日期：2026-10-06
- 安裝版本：JetPack 7.2.1（Jetson Linux r39.2.1）
- 依據文件：[NVIDIA 官方 Quick Start Guide](https://docs.nvidia.com/jetson/orin-nano-devkit/user-guide/latest/quick_start.html)

---

## 目錄

1. [硬體型號](#1-硬體型號)
2. [切換GUI、server指令](#2-切換GUI、server指令)
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
| 密碼:ee405423

## 2. 切換GUI、server指令
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

