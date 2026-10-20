# Jetson Orin Nano 交接手冊

本手冊記錄實驗室這台 Jetson Orin Nano 的環境設定、版本、參數與安裝指令。要把專案放到這台機器上時，可以在這裡查到需要配合的設定。

- 系統安裝日期：2026-10-06
- 最後更新：2026-10-20

---


## 0 實機部署操作（每次飛行照這裡做）

切換GUI、server指令

記憶體只有 8GB 且與 GPU 共用。開發時用桌面模式，上機實測時切成文字模式可多出約 1GB 記憶體。

```bash
sudo systemctl set-default multi-user.target   # 開機不進桌面
sudo systemctl set-default graphical.target    # 開機進桌面
sudo reboot                                    # 重新開機後生效
```

在文字模式下臨時啟動桌面：

```bash
sudo systemctl start gdm
```

檢查目前狀態：

```bash
systemctl get-default        # 下次開機的模式
systemctl is-active gdm      # 現在桌面有沒有在跑
free -h                      # 記憶體使用量
```

| 指令 | 輸出 | 意思 |
|---|---|---|
| `systemctl get-default` | `graphical.target` | 開機會進桌面 |
| | `multi-user.target` | 開機不進桌面（文字模式） |
| `systemctl is-active gdm` | `active` | 桌面正在執行，佔用記憶體 |
| | `inactive` | 沒有桌面，文字模式 |

## 0.1 SSH 連入 Jetson

確認筆電、Jetson兩者同一個網路
```bash
ssh ncrl@192.168.1.156       # 同一個網路，最方便
```

## 0.2 Docker 啟動與多個終端機

| 步驟 | 在哪裡 | 指令 |
|---|---|---|
| 1. 確認容器狀態 | 本機 | `docker ps -a`（`Up` 執行中，`Exited` 已停止） |
| 2. 容器已停止時先啟動 | 本機 | `docker start humble` |
| 3. 進入容器（每個終端機都做） | 本機 | `docker exec -it humble bash` |
| 離開容器（容器不會停） | 容器 | `exit` |
| 停止容器 | 本機 | `docker stop humble` |


### 0.3 各終端機的指令

重要!!!!!!!!!!!!!!!!!!!!!!!!!都在docker內運行

**終端機 1：Agent（DDS）**

```bash
MicroXRCEAgent serial --dev /dev/ttyUSB0 -b 921600
```

**終端機 2：mocap**

```bash
ros2 launch drone_mocap mocap.launch.py namespace:=MAV1 rigid_body:=MAV1 vrpn_server:=192.168.1.2
```

**終端機 3：開啟相機**

```bash
ros2 launch realsense2_camera rs_launch.py \
  rgb_camera.color_profile:=640x480x15 \
  enable_gyro:=true enable_accel:=true unite_imu_method:=2
```

**終端機 4：遠端看相機畫面（Foxglove)**

```bash
ros2 launch foxglove_bridge foxglove_bridge_launch.xml
```
地面站到瀏覽器開 [app.foxglove.dev](https://app.foxglove.dev)，需註冊免費帳號
Foxglove → **Open connection** → **Foxglove WebSocket**，輸入 `ws://<Jetson IP>:8765`
新增 **Image** 面板，topic 選 `/camera/camera/color/image_raw/compressed`。(較不延遲)

### 0.4 hover_test：起飛測試程式

`hover_test` 是簡單的測試程式，用來確認整套系統（Agent、動捕、飛控）能否正常起飛、懸停、移動、降落。
此程式已經在jatson 裡面了(由郁翔在維護)


**終端機 ：飛控節點 `hover_control`**

```bash
ros2 run hover_test hover_control --ros-args -p namespace:=/MAV1
```

**終端機 ：指令輸入節點 `hover_commander`**

```bash
ros2 run hover_test hover_commander
```

在 `>` 提示字元後輸入指令：

| 輸入 | 作用 |
|---|---|
| `1` | 起飛到懸停高度（預設 0.5m） |
| `2` | 降落並上鎖 |
| `3` | X 方向前進一步（預設 0.5m） |
| `4` | Y 方向前進一步（預設 0.5m） |
| `5 <deg>` | 設定機頭朝向，以預設 30 度/秒漸變轉過去 |
| `6 <x> <y>` | 飛到絕對座標 (x, y)，單位公尺 |

---


## 快速查詢

放專案上來時最常需要的設定，細節見各節。

| 項目 | 值 | 詳見 |
|---|---|---|
| 帳號 / 主機名 | `ncrl` / `ncrl` | 第 4 節 |
| SSH | `ssh ncrl@ncrl.local`，USB-C 直連為 `ssh ncrl@192.168.55.1` | 第 2 節 |
| 架構 / 系統 | aarch64（ARM64），Ubuntu 24.04，Python 3.12.3 | 第 5 節 |
| ROS 2 版本 | Jazzy，`/opt/ros/jazzy` | 第 5 節 |
| ROS 2 workspace | `~/ros2_ws`，專案套件放 `~/ros2_ws/src` | 第 1 節 |
| `ROS_DOMAIN_ID` | 42 | 第 8 節 |
| PX4 韌體版本 | v1.17 | 第 5 節 |
| `px4_msgs` 分支 | `release/1.17` | 第 5 節 |
| Micro XRCE-DDS Agent | v2.4.3，模擬時 `MicroXRCEAgent udp4 -p 8888` | 第 7 節 |
| PX4 topic | `/fmu/in/...`（送給飛控）、`/fmu/out/...`（飛控送出） | 第 7 節 |
| 編譯 | `colcon build` 在 `~/ros2_ws` 執行 | 第 6 節 |
| 記憶體 | 8GB，與 GPU 共用 | 第 4 節 |
| RealSense | 本機 Jazzy 為 apt 版（6.5 節）；容器 Humble 為 RSUSB 原始碼編譯版，含 IMU（6.8 節） | 第 6 節 |
| 遠端看相機畫面 | 容器內 `foxglove_bridge`，筆電 Foxglove 連 `ws://<Jetson IP>:8765` | 最前面 |
| Docker 容器 | 名稱 `humble`（Ubuntu 22.04 + ROS 2 Humble），進入用 `docker start -ai humble` | 第 10 節 |
| Humble workspace | 容器內 `/root/shared/ros2_ws`（本機路徑 `~/docker_shared/ros2_ws`） | 第 10 節 |
| 電源輸入 | DC 孔 9 到 20V、最大 3.5A、中心正極；實機用 4S 電池經 VG60-T2412 降到 12V | 第 4、11 節 |
| 接飛控的 UART | 40 pin 第 8 腳 TX、第 10 腳 RX、第 6 腳 GND，3.3V；裝置名稱用 `ls /dev/ttyTHS*` 確認 | 第 4、12 節 |
| Agent（實機） | 在容器內 `MicroXRCEAgent serial --dev /dev/ttyUSB0 -b 921600` | 第 0、12 節 |
| 無人機 namespace | `MAV2`（topic 為 `/MAV2/fmu/...`） | 第 0、12 節 |
| Motive 電腦 | `192.168.1.2`，VRPN 埠 3883 | 第 0、13 節 |

---

## 目錄

0. [實機部署操作（每次飛行照這裡做）](#0-實機部署操作每次飛行照這裡做)
1. [目錄結構與環境設定](#1-目錄結構與環境設定)
2. [SSH](#2-ssh)
3. [切換GUI、server指令](#3-切換guiserver指令)
4. [硬體型號](#4-硬體型號)
5. [版本資訊](#5-版本資訊)
6. [安裝指令](#6-安裝指令)
7. [模擬啟動流程](#7-模擬啟動流程)
8. [ROS_DOMAIN_ID（重要）](#8-ros_domain_id重要)
9. [注意事項與踩過的坑](#9-注意事項與踩過的坑)
10. [Docker 容器（Ubuntu 22.04 + ROS 2 Humble）](#10-docker-容器ubuntu-2204--ros-2-humble)
11. [電池供電](#11-電池供電)
12. [實機部署（Docker + 飛控）](#12-實機部署docker--飛控)
13. [動捕（Mocap）驗證](#13-動捕mocap驗證)

---

## 1. 目錄結構與環境設定

```
~/
├── PX4-Autopilot/           飛控韌體原始碼，用 make 編譯
├── Micro-XRCE-DDS-Agent/    Agent 原始碼（已安裝到系統，平常不用動）
├── QGroundControl-aarch64.AppImage
├── docker_shared/           與 Docker 容器共用的資料夾（容器內是 /root/shared）
│   ├── librealsense/        RealSense SDK v2.58.4 原始碼（RSUSB 編譯），見 6.8 節
│   └── ros2_ws/             Humble 的 workspace，只能在容器裡編譯，見第 10 節
│       └── src/
│           ├── px4_msgs/
│           ├── drone_mocap/
│           └── realsense-ros/   RealSense ROS wrapper 4.58.4，見 6.8 節
└── ros2_ws/                 ROS 2 workspace（Jazzy），專案都放這裡
    ├── src/                 套件原始碼
    │   ├── px4_msgs/
    │   ├── px4_ros_com/
    │   └── （之後的專案套件）
    ├── build/               編譯暫存檔（自動產生）
    ├── install/             編譯成品（自動產生）
    └── log/                 紀錄檔（自動產生）
```

`PX4-Autopilot` 不可以放進 `ros2_ws/src`，否則 `colcon build` 會把它當成 ROS 套件去編譯。

`~/.bashrc` 最底下應有這幾行，讓每個新終端機自動載入環境：

```bash
source /opt/ros/jazzy/setup.bash
source ~/ros2_ws/install/setup.bash
export ROS_DOMAIN_ID=42
```

檢查方式：`grep -n "source\|ROS_DOMAIN_ID" ~/.bashrc`

---

## 2. SSH

**方式一：用 IP 連線**

筆電與 Jetson 需在同一個網路。在筆電的終端機或 PowerShell 輸入：
```bash
ssh ncrl@192.168.50.216
```
IP 是路由器動態分配的，重開機或換網路後可能改變。在 Jetson 上用 `hostname -I` 查詢目前的 IP。

**方式二：用主機名連線（最方便）**
```bash
ssh ncrl@ncrl.local
```
不管 IP 變成多少都能用，前提是筆電和 Jetson 在同一個網路。Windows 上連不到時改用方式三。

**方式三：USB-C 線直連（最可靠）**

用傳輸線把筆電接到 Jetson 的 USB-C 孔，位址永遠固定，不需要任何網路：
```bash
ssh ncrl@192.168.55.1
```
Jetson 連不上 Wi-Fi 時，也是靠這個方式進去設定。

**到新環境的連線流程**
1. 用 USB-C 線接上筆電，以 `ssh ncrl@192.168.55.1` 連入。
2. 讓 Jetson 連上當地的 Wi-Fi：
```bash
   nmcli device wifi list
   sudo nmcli device wifi connect "Wi-Fi名稱" password "密碼"
   hostname -I
```
3. 拿到新 IP 後即可拔線，改用 Wi-Fi 連線。

連過的 Wi-Fi 會被記住，下次到同一個地方會自動連上。

---

## 3. 切換GUI、server指令

記憶體只有 8GB 且與 GPU 共用。開發時用桌面模式，上機實測時切成文字模式可多出約 1GB 記憶體。

```bash
sudo systemctl set-default multi-user.target   # 開機不進桌面
sudo systemctl set-default graphical.target    # 開機進桌面
sudo reboot                                    # 重新開機後生效
```

在文字模式下臨時啟動桌面：

```bash
sudo systemctl start gdm
```

---

## 4. 硬體型號

| 項目 | 內容 |
|---|---|
| 產品 | NVIDIA Jetson Orin Nano Developer Kit Super |
| 記憶體 | 8GB（CPU 與 GPU 共用） |
| 模組 | P3767-0005（開發套件版，模組背面有 microSD 插槽） |
| 系統儲存 | microSD 64GB（U3 / V30 / A2） |
| 影像輸出 | DisplayPort（沒有 HDMI） |
| 電源 | 原廠 19V 變壓器，DC 圓孔 |
| 帳號 / 主機名 | `ncrl` / `ncrl` |
| 密碼 | 不寫在此，請向負責人取得 |

### 4.1 電源規格

依據 NVIDIA 官方載板規格書（Jetson Orin Nano DevKit Carrier Board Specification）與 NVIDIA 論壇官方回覆。

| 項目 | 規格 |
|---|---|
| DC 圓孔輸入電壓 | **9 到 20V**（5V 無法開機） |
| DC 圓孔最大電流 | **3.5A**（規格書另一表寫 4.2A，以 3.5A 為準） |
| 接頭 | 5.5 × 2.5mm，**中心為正極** |
| 原廠變壓器 | 19V |
| USB-C | 只傳資料，**不能供電** |
| 模組本身 | 吃 5V，由載板從 9 到 20V 轉換 |

電源模式（`sudo nvpmodel -q` 查詢、`sudo nvpmodel -m <ID>` 切換）：

| ID | 模式 | 功耗 |
|---|---|---|
| 0 | 15W | 15W |
| 1 | 25W | 25W（目前預設） |
| 2 | MAXN_SUPER | 不設上限（實驗性） |

實測閒置整板耗電約 3.6W（變壓器與電池供電相同）。

查詢這台支援的模式與 ID：`sudo nvpmodel -p --verbose`。看實際功耗：`sudo tegrastats`，`VDD_IN` 為整板輸入功率（mW）。

**各模式在 12V（VG60-T2412）供電時的電流（估算）**

輸入電壓不分模式，都是 DC 孔的 9 到 20V；模式改變的是功耗，所以只有電流改變。模式瓦數只算模組，載板、D435i（約 2 到 3.5W）、風扇、Wi-Fi 另計。以下為估算，實際以 `tegrastats` 為準。

| 模式 | 整板估計功耗（含 D435i） | 12V 時電流 | 與 DC 孔 3.5A 上限 |
|---|---|---|---|
| 15W（ID 0） | 約 18 到 22W | 約 1.5 到 1.8A | 餘裕大 |
| 25W（ID 1） | 約 35 到 40W | 約 3 到 3.3A | 接近上限 |
| MAXN_SUPER（ID 2） | 尖峰可能超過 40W | 可能超過 3.5A | 不建議電池供電使用 |

DC 孔可用最大功率 ＝ 電壓 × 3.5A：12V 約 42W，19V 約 66W。

**飛行時建議切到 15W**（`sudo nvpmodel -m 0`），電流餘裕較大、電池較耐用。

電流換算：電流 ＝ 功率 ÷ 電壓。25W 模式加相機等周邊約 35 到 40W，12V 時約 3.3A，接近 DC 孔 3.5A 上限；19V 時約 2.1A。

上電與斷電：
- 接上電源就會自動開機（按鍵排針 J14 第 5、6 腳短接可關閉自動開機）。
- 周邊要在上電前接好，不要通電時插拔。
- 關機先 `sudo shutdown now`，等電源燈熄滅再拔電源。

### 4.2 40 pin 排針（接飛控用）

所有訊號腳為 **3.3V 邏輯準位**，和 Pixhawk 序列埠相容。

| 腳位 | 訊號 | 用途 |
|---|---|---|
| 8 | UART1_TXD | 接飛控 TELEM2 的 RX（pin 3） |
| 10 | UART1_RXD | 接飛控 TELEM2 的 TX（pin 2） |
| 6 | GND | 接飛控 TELEM2 的 GND（pin 6） |
| 1、17 | 3.3V | 只能供 100mA，不可供電給其他設備 |
| 2、4 | 5V | 約 0.5A，不可供電給飛控或相機 |

- 飛控 TELEM2 的 pin 1（+5V）**不要接**。
- 第 1 腳的位置：板子擺成電源、USB、網路等接頭朝上時，在排針左上角。以板子上印的編號為準。
- 這組 UART 的裝置名稱會隨 JetPack 版本改變（JetPack 5 為 `/dev/ttyTHS0`，JetPack 6 為 `/dev/ttyTHS1`），用以下指令確認：
  ```bash
  ls -l /dev/ttyTHS*
  sudo dmesg | grep 3100000
  ```
- 自我測試：第 8、10 腳短接，執行
  ```bash
  stty -F /dev/ttyTHS1 921600 raw -echo
  cat /dev/ttyTHS1 &
  echo "hello" > /dev/ttyTHS1
  ```
  印出 `hello` 代表 UART 正常，測完 `kill %1`。
- 模組下方 12 pin 按鍵排針上的 UART 是**除錯主控台**，不要拿來接飛控。

---

## 5. 版本資訊

所有版本集中在這一節。各項的安裝指令見第 6 節。

### 5.1 系統

（2026-10-06 安裝完成後實測）

| 項目 | 版本 |
|---|---|
| JetPack | 7.2.1 |
| Jetson Linux（L4T） | R39.2.1 |
| Ubuntu | 24.04.4 LTS |
| Linux 核心 | 6.8.12-1021-tegra |
| 韌體 | 39.2.1（安裝前為 36.4.7），開機槽位 A |
| Python | 3.12.3 |

### 5.2 已安裝的軟體

（2026-10-12 更新）

**Jetson 本機**

| 軟體 | 版本 | 位置 |
|---|---|---|
| ROS 2 | Jazzy（desktop） | `/opt/ros/jazzy` |
| PX4-Autopilot 原始碼 | v1.17.0（`release/1.17` 分支） | `~/PX4-Autopilot` |
| px4_msgs | `release/1.17` | `~/ros2_ws/src/px4_msgs` |
| px4_ros_com | `release/1.16` | `~/ros2_ws/src/px4_ros_com` |
| Micro XRCE-DDS Agent | v2.4.3 | 原始碼 `~/Micro-XRCE-DDS-Agent`，執行檔在 `/usr/local` |
| QGroundControl | aarch64 AppImage（最新穩定版） | `~/QGroundControl-aarch64.AppImage` |
| RealSense SDK + ROS 套件 | ROS 伺服器的 Jazzy 版（`ros-jazzy-librealsense2`、`ros-jazzy-realsense2-*`） | `/opt/ros/jazzy` |
| Docker | 系統安裝時已內建 | 系統 |
| 中文輸入法 | 新酷音（`ibus-chewing`）、字型 `fonts-noto-cjk` | 系統 |

**Docker 容器 `humble`**

| 軟體 | 版本 | 位置 |
|---|---|---|
| 映像檔 | `ros:humble-perception`（Ubuntu 22.04 + ROS 2 Humble） | Docker |
| px4_msgs | `release/1.17` | `/root/shared/ros2_ws/src/px4_msgs` |
| drone_mocap | GitHub `yuhsiangsean/drone_mocap` | `/root/shared/ros2_ws/src/drone_mocap` |
| RealSense SDK（librealsense） | v2.58.4，RSUSB 後端，原始碼編譯 | 原始碼 `/root/shared/librealsense`，安裝在容器 `/usr/local` |
| RealSense ROS wrapper（realsense-ros） | 4.58.4，原始碼編譯 | `/root/shared/ros2_ws/src/realsense-ros` |
| foxglove_bridge | apt（`ros-humble-foxglove-bridge`） | `/opt/ros/humble` |

### 5.3 版本之間的對應關係

- **飛控韌體是 PX4 v1.17**，`PX4-Autopilot` 與 `px4_msgs` 都必須是同一個版本。
- Ubuntu 24.04 對應的 ROS 2 是 **Jazzy**，不能裝 Humble。
- ROS 2 Jazzy 對應的 Agent 版本是 **v2.4.3**（Humble 才是 v2.4.2）。
- `px4_ros_com` 官方沒有 1.17 的分支，使用最接近的 `release/1.16`。

### 5.4 查詢版本的指令

```bash
cat /proc/device-tree/model                 # 型號
cat /etc/nv_tegra_release                   # Jetson Linux
lsb_release -ds                             # Ubuntu
uname -r                                    # 核心
sudo nvbootctrl dump-slots-info | head -n 3 # 韌體
python3 --version                           # Python
echo $ROS_DISTRO                            # ROS 2
cd ~/PX4-Autopilot && git describe --tags   # PX4
```

---

## 6. 安裝指令

需要重灌或在另一台 Jetson 上重建環境時，依 6.1 到 6.7 的順序做。

### 6.1 Jetson 系統（JetPack 7.2.1）

根據以下連結一步步操作：
[NVIDIA 官方 Quick Start Guide](https://docs.nvidia.com/jetson/orin-nano-devkit/user-guide/latest/quick_start.html)

重點提醒：

- JetPack 7.2 之後改用 Jetson ISO 安裝。ISO 是用 Balena Etcher 寫到 **USB 隨身碟**（16GB 以上），不是寫到 microSD 卡。
- 寫入後 Windows 會跳出「需要格式化」的視窗，一律按取消。
- 用 USB 開機後，畫面詢問是否更新韌體時**要在 30 秒內按 `Y`**，漏按會導致後面安裝失敗。
- 首次開機建立帳號時，Username 設定後不能更改。

### 6.2 ROS 2 Jazzy

依據 [ROS 2 Jazzy 官方安裝文件](https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debs.html)。

```bash
# 1. 確認語系為 UTF-8（本機已是 en_US.UTF-8，不需額外設定）
locale

# 2. 啟用套件庫
sudo apt install software-properties-common
sudo add-apt-repository universe
sudo apt update && sudo apt install curl -y
export ROS_APT_SOURCE_VERSION=$(curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest | grep -F "tag_name" | awk -F'"' '{print $4}')
curl -L -o /tmp/ros2-apt-source.deb "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo ${UBUNTU_CODENAME:-${VERSION_CODENAME}})_all.deb"
sudo dpkg -i /tmp/ros2-apt-source.deb

# 3. 安裝開發工具與 ROS 2
sudo apt update && sudo apt install ros-dev-tools
sudo apt upgrade
sudo apt install ros-jazzy-desktop

# 4. 設定環境
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

測試（開兩個終端機）：

```bash
ros2 run demo_nodes_cpp talker
ros2 run demo_nodes_py listener
```

補充：Jetson 的 apt 來源寫在 `/etc/apt/sources.list`（舊格式），裡面已包含 `noble-updates` 與 `noble-backports`，符合官方對 Ubuntu 24.04 的要求，不需修改。

### 6.3 PX4 相關套件

實驗室既有的 PX4 手冊是為 **Ubuntu 22.04 + ROS 2 Humble** 寫的。在這台 Jetson（Ubuntu 24.04 + Jazzy + Python 3.12）上有下列差異，照抄會出錯：

| 既有手冊的內容 | 在 Jetson 上的做法 |
|---|---|
| `/opt/ros/humble` | 改成 `/opt/ros/jazzy` |
| `~/ws_sensor_combined` | 改用 `~/ros2_ws` |
| `git clone` 不指定分支 | 一律指定與飛控韌體相同的版本 |
| setuptools 必須是 59.6.0 | 不適用，維持系統 apt 的版本 |
| `rm -rf ~/.local/lib/python3.10/...` | 跳過，本機是 Python 3.12 |
| `pip3 install --user "empy==3.3.4" pyros-genmsg` | 會被系統擋下，改用 `sudo apt install python3-empy`（版本即 3.3.4） |
| `pip install --user "setuptools==65.5.1" --force-reinstall` | **不要執行** |
| Agent 用 `v2.4.2` | Jazzy 要用 `v2.4.3` |
| 修改 CMakeLists.txt 的 fastdds tag（`sed` 那行） | **不要執行**，v2.4.3 用的是 `2.14.x`，不需修改 |

**(1) PX4-Autopilot 原始碼**

```bash
cd ~
git clone -b release/1.17 https://github.com/PX4/PX4-Autopilot.git --recursive
cd PX4-Autopilot
git describe --tags        # 應顯示 v1.17 開頭
cd ~
bash ./PX4-Autopilot/Tools/setup/ubuntu.sh --no-sim-tools
sudo reboot
```

- 不指定分支會抓到 `main`（開發版，顯示為 `v1.18.0-beta…`），與飛控的 v1.17 不相容。已經抓錯的話用 `git checkout release/1.17` 和 `git submodule update --init --recursive` 切回來。
- `--no-sim-tools` 是不安裝 Gazebo。

**(2) px4_msgs 與 px4_ros_com**

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
git clone -b release/1.17 https://github.com/PX4/px4_msgs.git
git clone -b release/1.16 https://github.com/PX4/px4_ros_com.git
cd ~/ros2_ws
colcon build
source install/setup.bash
ros2 interface show px4_msgs/msg/SensorCombined     # 有印出欄位即成功
echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc
```

- `colcon build` 一定要在 `~/ros2_ws` 這層執行，不是在 `src` 裡。
- `px4_ros_com` 只是範例程式，不是必要元件。

**(3) Micro XRCE-DDS Agent**

獨立安裝到系統，不放進 workspace：

```bash
cd ~
git clone -b v2.4.3 https://github.com/eProsima/Micro-XRCE-DDS-Agent.git
cd Micro-XRCE-DDS-Agent
mkdir build && cd build
cmake ..
make -j4
sudo make install
sudo ldconfig /usr/local/lib/
MicroXRCEAgent --help      # 有印出使用說明即成功
```

### 6.4 QGroundControl

官方提供 ARM64 版本，支援 Ubuntu 24.04：

```bash
sudo usermod -aG dialout "$(id -un)"
sudo systemctl mask --now ModemManager.service
sudo apt install -y libfuse2 libxcb-xinerama0 libxkbcommon-x11-0 libxcb-cursor0
cd ~
wget https://d176tv9ibo4jno.cloudfront.net/latest/QGroundControl-aarch64.AppImage
chmod +x QGroundControl-aarch64.AppImage
```

`dialout` 群組要登出再登入才生效。QGC 是圖形程式，要在 Jetson 桌面上執行，不能從 SSH 啟動。

### 6.5 RealSense（本機 Jazzy）

依據 [realsense-ros 官方說明](https://github.com/realsenseai/realsense-ros)。SDK 選 Step 2 的 Option 2（ROS 伺服器），ROS 套件選 Step 3 的 Option 1（apt）：

```bash
sudo apt update
sudo apt install ros-jazzy-librealsense2*
sudo apt install ros-jazzy-realsense2-*
```

- 這個版本走 RSUSB 後端，不需要對 Linux 核心打補丁。代價是沒有 `realsense-viewer`，也不支援多台相機同時使用。
- SDK 只能用一種方式安裝，之後不要再從原始碼或 RealSense 伺服器另裝一份，以免版本衝突。
- 兩者都裝在 `/opt/ros/jazzy`，不在 `~/ros2_ws`，不需要 `colcon build`。查位置：`ros2 pkg prefix realsense2_camera`。
- 相機接藍色的 USB 3 孔，不要接 USB-C 孔。

啟動：

```bash
ros2 launch realsense2_camera rs_launch.py
```

### 6.6 中文輸入法

```bash
sudo apt update
sudo apt install fonts-noto-cjk ibus-chewing
```

登出再登入後，Settings → Keyboard → Input Sources → Add → Chinese (Taiwan) → Chinese (Chewing)。用 `Super + 空白鍵` 切換中英文。系統介面維持英文。

### 6.7 Docker 與 Ubuntu 22.04 + Humble 容器

Docker 在安裝系統時已經內建，不用另外安裝。

**(1) 讓帳號不用 sudo 就能使用 Docker**

```bash
sudo usermod -aG docker $USER
sudo reboot
groups       # 重開後輸出裡要有 docker
```

**(2) 下載映像檔**

```bash
docker pull ros:humble-perception
```

選這個映像檔的理由：它是 ROS 官方（OSRF）發布在 Docker Hub 的映像檔，有 ARM64 版本，內含 ROS 2 Humble、`colcon`、`rosdep`，以及 OpenCV、`cv_bridge`、PCL、`image_pipeline` 等影像與點雲套件。

**(3) 建立容器（只執行一次）**

```bash
mkdir -p ~/docker_shared
xhost +local:docker
docker run -it --name humble --network host --ipc host --privileged -v /dev:/dev -v ~/docker_shared:/root/shared -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -e ROS_DOMAIN_ID=42 ros:humble-perception bash
```

各參數的意思見第 10 節。

**(4) 在容器裡建立 workspace 並下載專案**

```bash
mkdir -p /root/shared/ros2_ws/src
cd /root/shared/ros2_ws/src
git clone -b release/1.17 https://github.com/PX4/px4_msgs.git
git clone https://github.com/yuhsiangsean/drone_mocap.git
```

### 6.8 RealSense（容器 Humble，RSUSB 原始碼編譯）

依據 [realsense-ros 官方說明](https://github.com/realsenseai/realsense-ros) 與 [librealsense 官方 repo](https://github.com/IntelRealSense/librealsense)。

**為什麼不用 apt 版**

- 容器內原本用 apt 安裝（`ros-humble-librealsense2*`、`ros-humble-realsense2-*`），影像正常，但開 IMU 時出現 `No HID info provided, IMU is disabled`。
- apt 版走 Linux 核心驅動（V4L 後端，`rs-enumerate-devices` 的 Physical Port 為 `.../video4linux/video0`）。D435i 的 IMU 透過核心的 HID 感測器模組傳資料。
- 這台的核心沒有該模組：本機 `sudo modprobe hid_sensor_accel_3d hid_sensor_gyro_3d` 回報 `Module hid_sensor_accel_3d not found in directory /lib/modules/6.8.12-1021-tegra`。
- 解法：SDK 改用 **RSUSB 後端**從原始碼編譯，直接透過 libusb 讀 IMU，不需要核心模組。apt 的 ROS wrapper 綁定 apt 的 SDK，所以 wrapper 也要從原始碼編譯。
- 版本對齊 apt 版原本的組合：SDK **v2.58.4** + ROS wrapper **4.58.4**。

**(1) 移除 apt 版**（容器內，先停掉相機節點）

```bash
apt remove -y 'ros-humble-librealsense2*' 'ros-humble-realsense2-*'
```

**(2) 編譯 SDK**（容器內，約 30 到 60 分鐘）

```bash
apt update && apt install -y git cmake build-essential pkg-config libssl-dev libusb-1.0-0-dev libudev-dev
cd /root/shared
git clone -b v2.58.4 --depth 1 https://github.com/IntelRealSense/librealsense.git
cd librealsense && mkdir build && cd build
cmake .. -DFORCE_RSUSB_BACKEND=ON -DBUILD_EXAMPLES=OFF -DBUILD_GRAPHICAL_EXAMPLES=OFF -DCMAKE_BUILD_TYPE=Release
make -j2
make install
ldconfig
```

- `-DFORCE_RSUSB_BACKEND=ON` 是讀得到 IMU 的關鍵。
- clone tag 時出現 `detached HEAD` 是正常的。
- `librealsense` 不是 ROS 套件，放在 `/root/shared`，不可放進 `ros2_ws/src`。
- 只用 `-j2`，Jetson 沒有 swap。

**(3) 編譯 ROS wrapper**（容器內）

```bash
cd /root/shared/ros2_ws/src
git clone -b 4.58.4 --depth 1 https://github.com/realsenseai/realsense-ros.git
cd /root/shared/ros2_ws
MAKEFLAGS=-j2 colcon build --executor sequential --packages-up-to realsense2_camera
source install/setup.bash
```

tag 名稱不對時，用 `git ls-remote --tags https://github.com/realsenseai/realsense-ros.git | grep 4.58` 查詢。

**(4) 確認相機**

相機接**藍色 USB 3 孔**。

```bash
rs-enumerate-devices | head -20
```

這台 D435i 的資訊：

| 項目 | 值 |
|---|---|
| 序號 | 261222078537 |
| 韌體 | 5.15.1.55 |
| USB | 3.2（顯示 2.x 代表接錯孔或線材不對） |
| IMU | BMI085 |

**(5) 啟動與 IMU 參數**

```bash
ros2 launch realsense2_camera rs_launch.py \
  rgb_camera.color_profile:=640x480x15 \
  enable_gyro:=true enable_accel:=true unite_imu_method:=2
```

| 參數 | 作用 |
|---|---|
| `rgb_camera.color_profile` | 彩色影像的 `寬x高x幀率`。飛行時調低，減少運算與頻寬 |
| `enable_gyro`、`enable_accel` | 打開陀螺儀、加速度計（預設關閉） |
| `unite_imu_method` | 0：不合併；1：copy；2：線性內插，合併成 `/camera/camera/imu` |

**(6) 主要 topic**

| topic | 內容 |
|---|---|
| `/camera/camera/color/image_raw` | 彩色影像 |
| `/camera/camera/color/image_raw/compressed` | 壓縮彩色影像，遠端看畫面用 |
| `/camera/camera/depth/image_rect_raw` | 深度影像（預設 848×480、30fps） |
| `/camera/camera/imu` | 合併後的 IMU（`sensor_msgs/Imu`） |
| `/camera/camera/accel/sample`、`/camera/camera/gyro/sample` | 原始加速度、角速度 |
| `/camera/camera/extrinsics/depth_to_*` | 各感測器之間的相對位置 |

檢查：`ros2 topic hz /camera/camera/imu`。靜止平放時，`linear_acceleration` 有一軸接近 ±9.8，`angular_velocity` 三軸接近 0。

**注意事項**

- IMU 訊息只有角速度與加速度，`orientation` 是空的，需要姿態要另接濾波器（例如 `imu_filter_madgwick`）。
- 相機 IMU 與飛控 IMU 無關，飛控不使用相機 IMU。
- 啟動時的 `No valid configuration file`、`re-enable the stream`、`HARDWARE_CLOCK` 警告可忽略。
- SDK 安裝在容器的 `/usr/local`，`docker rm humble` 後要在 `/root/shared/librealsense/build` 重新 `make install && ldconfig`。
- 本機 Jazzy 的 apt 版（6.5 節）與容器內這份各自獨立。同一台相機一次只能被一個程式開啟，本機與容器不可同時啟動相機。

---

## 7. 模擬啟動流程

Jetson 上使用 PX4 內建的輕量模擬 **SIH**（沒有 3D 畫面，不需要 Gazebo）。每一項各開一個終端機，依序啟動：

| 順序 | 項目 | 指令 |
|---|---|---|
| 1 | PX4 模擬 | `cd ~/PX4-Autopilot && make -j2 px4_sitl sihsim_quadx` |
| 2 | QGroundControl | `~/QGroundControl-aarch64.AppImage` |
| 3 | Agent | `MicroXRCEAgent udp4 -p 8888` |
| 4 | ROS 2 程式 | 例如 `ros2 topic list` |

- PX4 模擬啟動後會停在 `pxh>`，該終端機要保持開啟。
- **沒有開 QGC 無法解鎖起飛**，PX4 的飛行前檢查會要求地面站連線。只是看 topic 的話可以不開。
- `sihsim_quadx` 的意思：SIH 模擬器、X 型四旋翼。
- 正常接通時，`ros2 topic list` 會出現一整排 `/fmu/in/...`（可送給飛控的指令）與 `/fmu/out/...`（飛控送出的資料）。

在 `pxh>` 的常用指令：

```
commander takeoff     起飛
commander land        降落
commander arm         解鎖
commander disarm      上鎖
commander status      查看狀態
shutdown              結束模擬
```

結束時反向關閉：先停 ROS 2 程式，再關 Agent 與 QGC，最後結束 PX4 模擬。

**換成真實飛控時**：不需要啟動 PX4 模擬（飛控本身就是 PX4），Agent 改用序列埠連線，其餘相同。接線與飛控參數設定尚未完成。

---

## 8. ROS_DOMAIN_ID（重要）

實驗室網路上有其他人的 ROS 2 系統在執行。ROS 2 預設會自動探索同一個區域網路上、`ROS_DOMAIN_ID` 相同（預設為 0）的所有節點，因此在 Jetson 上執行 `ros2 topic list` 時，曾看到不屬於這台機器的 topic（`/MAV1/...` 到 `/MAV5/...`、`/swarm/...`）。

互通是雙向的：別人的程式可以對這台的 `/fmu/in/...` 發指令，這台的測試指令也可能被別人的系統收到。接上真實飛控後這是安全問題。

**做法：每台機器或每個專案使用不同的 `ROS_DOMAIN_ID`**

```bash
echo "export ROS_DOMAIN_ID=<編號>" >> ~/.bashrc
```

- 編號範圍 1 到 101，需與實驗室其他人協調，避免重複。
- 設定後要關掉所有終端機重開，PX4 模擬、Agent、ROS 2 程式都要重新啟動。
- PX4 模擬會讀取這個環境變數。**真實飛控**則要在 QGC 把參數 `UXRCE_DDS_DOM_ID` 設成相同的數字。
- 確認方式：`echo $ROS_DOMAIN_ID`，以及 `ros2 topic list` 不再出現別人的 topic。

實驗室 ROS_DOMAIN_ID 對照表（請補上）：

| 編號 | 使用者 / 機器 / 專案 |
|---|---|
| 0 | 預設值，請避免使用 |
| 42 | 這台 Jetson Orin Nano（`ncrl`） |
|   |   |

---

## 9. 注意事項與踩過的坑

| 狀況 | 原因 | 處理方式 |
|---|---|---|
| 編譯 PX4 的 Gazebo 模擬（`make px4_sitl gz_x500`）時整台凍住 | PX4 預設用全部 6 核心編譯，8GB 記憶體被用光，系統沒有 swap | 當時拔電重開機，之後放棄在這台上跑 Gazebo |
| `make px4_sitl gz_x500` 顯示 `Gazebo simulation dependencies not found` | PX4 先編譯過、之後才裝 Gazebo，編譯快取還記著找不到 | `rm -rf ~/PX4-Autopilot/build/px4_sitl_default` 後重新編譯 |
| 想在 Jetson 上跑 Gazebo | 桌面加 Gazebo 加 QGC 約需 6 到 7GB，超出這台的能力 | Jetson 上用 SIH 模擬；Gazebo 在筆電上跑 |
| `source install/local_setup.bash` 顯示找不到檔案 | 人在 `~/ros2_ws/src`，`install` 在上一層 | 先 `cd ~/ros2_ws` |
| `make px4_sitl` 顯示 `ninja: no work to do` | 只編譯不啟動，且已編譯完成 | 後面要加模擬目標，例如 `sihsim_quadx` |
| `ros2 topic list` 出現不認識的 topic | 與實驗室其他 ROS 2 系統使用相同的 `ROS_DOMAIN_ID` | 見第 8 節 |
| `git describe --tags` 顯示 `v1.18.0-beta…` | clone 時沒有指定分支 | `git checkout release/1.17` 並更新子模組 |
| `docker` 指令顯示 `permission denied while trying to connect to the docker API` | 帳號不在 `docker` 群組 | `sudo usermod -aG docker $USER` 後重開機，用 `groups` 確認有 `docker` |
| 執行 `usermod` 後 `groups` 還是沒有 `docker` | 群組設定要重新登入才生效，只開新終端機不夠 | 重開機 |
| 系統日期不對（曾顯示為 3 天前） | 開發套件斷電後不保存時間，用電池開機又沒連網路時無法校時 | 連網後 `timedatectl` 確認 `System clock synchronized: yes`，否則 `sudo timedatectl set-ntp true`；戶外無網路時要注意時間可能不準 |
| 飛控上下顛倒安裝 | 飛控會以為自己翻覆，無法解鎖或起飛即翻機 | QGC 設定 `SENS_BOARD_ROT`（倒裝且箭頭朝機頭為 `ROLL_180`），重開飛控後重新校正加速度計、陀螺儀、水平、羅盤，並用姿態儀確認方向 |

**常用檢查指令**

```bash
df -h /              # 硬碟剩餘空間
free -h              # 記憶體
sudo tegrastats      # CPU、GPU、溫度、耗電（Ctrl+C 結束）
ros2 pkg list | grep px4
```

---

## 10. Docker 容器（Ubuntu 22.04 + ROS 2 Humble）

Jetson 本機是 Ubuntu 24.04 + Jazzy。需要 Ubuntu 22.04 + Humble 的專案（例如 `drone_mocap`）放在 Docker 容器 `humble` 裡執行。建立方式見 6.7。

### 10.1 進出容器

| 要做的事 | 指令 |
|---|---|
| 進入容器（容器已停止時） | `docker start -ai humble` |
| 另開一個終端機進同一個容器 | `docker exec -it humble bash` |
| 離開容器 | 在容器裡輸入 `exit` |
| 查看容器狀態 | `docker ps -a`（`Up` 是執行中，`Exited` 是已停止） |
| 允許容器開圖形視窗 | 在本機執行 `xhost +local:docker`，每次開機後要重做 |

提示字元是 `root@ncrl` 代表在容器裡，`ncrl@ncrl` 代表在 Jetson 本機。

**之後進容器一律用 `docker start` 或 `docker exec`，不要再執行 `docker run`**。`docker run` 會建立一個全新的空容器；名稱 `humble` 已被使用時會直接報錯。

### 10.2 檔案會不會消失

| 動作 | 容器裡的檔案與 apt 裝的套件 | `/root/shared` 裡的檔案 |
|---|---|---|
| `exit` 或關掉終端機 | 保留 | 保留 |
| Jetson 重開機 | 保留 | 保留 |
| `docker rm humble`（刪除容器） | **全部消失** | 保留 |

程式碼和 workspace 一律放在 `/root/shared` 底下。這個資料夾就是本機的 `~/docker_shared`，可以直接用本機的編輯器修改。

### 10.3 兩個 ros2_ws

| | 本機 | 容器 |
|---|---|---|
| 路徑 | `~/ros2_ws` | 容器內 `/root/shared/ros2_ws`（本機 `~/docker_shared/ros2_ws`） |
| ROS 版本 | Jazzy | Humble |
| 內容 | `px4_msgs`、`px4_ros_com` | `px4_msgs`、`drone_mocap` |

- 容器的 workspace 只能在容器裡 `colcon build`，在本機編譯會用到 Jazzy，容器裡無法使用。
- 不要把本機的 `~/ros2_ws` 掛進容器，兩邊的編譯結果不能混用。

### 10.4 建立容器時的參數

| 參數 | 作用 |
|---|---|
| `--name humble` | 容器名稱 |
| `--network host` | 與 Jetson 共用網路，ROS 2 才能和外部通訊 |
| `--ipc host` | 共用記憶體通道，ROS 2 在同一台機器上傳資料時需要 |
| `--privileged` 與 `-v /dev:/dev` | 讓容器存取 USB、序列埠等硬體（RealSense、飛控） |
| `-v ~/docker_shared:/root/shared` | 本機與容器的共用資料夾 |
| `-e DISPLAY=$DISPLAY`、`-v /tmp/.X11-unix:/tmp/.X11-unix` | 讓容器能開圖形視窗 |
| `-e ROS_DOMAIN_ID=42` | 與本機相同的 ROS 2 通訊編號 |

### 10.5 注意事項

- 容器和本機共用同一個 Linux 核心，不是虛擬機，CPU 與記憶體效能和本機幾乎相同。
- `ros:humble-perception` 沒有 CUDA，容器裡不能用 GPU 加速。
- 容器內的 Humble 與本機的 Jazzy 屬於不同版本，ROS 2 不保證跨版本互通。需要互相傳資料的節點放在同一邊。Micro XRCE-DDS Agent 不分 ROS 版本；實機時 Agent 也裝在容器內執行（見 12.1），避免本機與容器之間收不到資料。
- `drone_mocap` 預設的 topic 帶 `/MAV1` namespace（這台用 `MAV2`，啟動時帶參數）。實驗室網路上曾出現其他電腦的 `/MAV1/...` 到 `/MAV5/...` topic，所以 `ROS_DOMAIN_ID` 一定要設定。

### 10.6 常用 Docker 指令

都在 Jetson 本機執行。

| 要做的事 | 指令 |
|---|---|
| 看容器狀態 | `docker ps -a` |
| 啟動並進入 | `docker start humble && docker exec -it humble bash` |
| 不進容器執行一行指令 | `docker exec humble bash -c "source /opt/ros/humble/setup.bash && ros2 topic list"` |
| 停止 / 重新啟動 | `docker stop humble` / `docker restart humble` |
| 看輸出紀錄 | `docker logs humble` |
| 開機自動啟動 | `docker update --restart unless-stopped humble` |
| 看環境變數 | `docker exec humble env \| grep ROS` |
| 複製檔案 | `docker cp ~/a.txt humble:/root/`、`docker cp humble:/root/a.txt ~/` |
| 看映像檔 | `docker images` |

會刪除東西的指令：`docker rm humble`（刪容器）、`docker rmi <映像檔>`（刪映像檔）。

### 10.7 Agent 與容器的 DDS 通訊

- Agent 把飛控資料轉成 DDS 後**透過網路**發送，ROS 2 底層就是 DDS。
- 容器建立時加了 `--network host` 與 `--ipc host`，和 Jetson 本機共用網路與共用記憶體。
- 實機時 Agent 改在容器內執行：Agent 在本機時，容器內雖然 `ros2 topic list` 看得到 `/MAV2/fmu/...`，`ros2 topic echo` 卻收不到資料。
- 飛控 `UXRCE_DDS_DOM_ID` 與容器 `ROS_DOMAIN_ID` 都是 42。
- Agent 不屬於任何 ROS 版本，容器裡 Humble 能否解讀資料取決於 `px4_msgs` 版本（容器內為 `release/1.17`，與飛控一致）。
- 同一時間只能開一個 Agent，兩個 Agent 搶同一個序列埠會出錯。

---

## 11. 電池供電

### 11.1 架構

馬達與 Jetson **使用兩顆獨立的 4S 電池**，避免馬達加油門時的電壓瞬降影響 Jetson。

```
4S 電池 A ──→ 電調、馬達、飛控（電源模組）

4S 電池 B ──→ XT60 ──→ VG60-T2412（降壓）──→ DC 線 ──→ Jetson DC 孔 ──USB 3──→ D435i
```

唯一連接兩套電源的是 Jetson 與飛控之間 UART 線上的 GND。兩顆電池的正負極不可互接。

### 11.2 使用的元件

| 元件 | 規格 |
|---|---|
| 電池 B | 4S 14.8V、4000mAh、60C，XT60 |
| 降壓模組 | VG60-T2412：輸入 **15 到 40V**，輸出 12V、5A（60W） |
| DC 線 | 5.5 × 2.5mm 公頭 |

接頭名稱：

| 接頭 | 位置 | 用途 |
|---|---|---|
| 平衡頭（JST-XH，4S 為 5 pin，白色小接頭） | 電池 | 充電平衡、插低壓警報器，不供電 |
| XT60 母頭（黃色） | 電池主電源 | 電池輸出 |
| XT60 公頭（黃色，帶紅黑線） | 接降壓模組輸入 | 插入電池 XT60 母頭 |
| DC 圓頭公頭（5.5 × 2.5mm） | 接降壓模組輸出 | 插入 Jetson DC 孔（中心正極） |

電池規格的意思（4000mAh、60C）：

| 項目 | 計算 | 意義 |
|---|---|---|
| 容量 4000mAh | 4A 放電約 1 小時 | VG60 輸入下限 15V，實際只用得到上半段 |
| 最大放電 60C | 4Ah × 60 ＝ 240A | 電池能瞬間輸出極大電流 |
| 實際 Jetson 這一路 | 整板 20W、電池 15V、轉換效率約 0.9：20 ÷ 15 ÷ 0.9 ≈ 1.5A | 負載決定電流，正常約 1 到 3A |

續航估算：可用電量約 25 到 30Wh，以 15W 模式整板約 20W 計，約 1 小時多。

VG60-T2412 接線：

| 線 | 接到 |
|---|---|
| 輸入紅（+） | XT60 紅 |
| 輸入黑（−） | XT60 黑 |
| **輸出黃（+12V）** | DC 線正極（接圓頭中心） |
| 輸出黑（−） | DC 線負極 |

DC 線哪條接中心要用三用電表通路檔確認，不要只看顏色。

### 11.3 限制

- VG60-T2412 是 24V 系統用的模組，輸入下限 15V。4S 電池在每顆約 3.75V（總電壓 15V，約剩一半電量）以下就可能停止輸出，所以**電池 B 大約只能用上半段容量**。
- 平衡頭插低壓警報器，**門檻設 3.8V**，在模組斷電前提醒。
- 電池 B 只供 Jetson 與 D435i（約 20 到 30W），電流小，幾乎不會有電壓瞬降。若改成和馬達共用電池，大油門時電壓瞬降可能低於 15V，造成 Jetson 突然斷電。
- 6S 電池（18 到 25.2V）完全符合這顆模組的規格；6S 不可直接接 Jetson（超過 20V）。
- 目前**沒有裝保險絲**。電池到模組輸入這一段沒有短路保護，建議補上 5A 串接式保險絲，裝在電池與模組之間、越靠近電池越好。電池 60C 短路時可放出上百安培，電線數秒內就會燒熔；正常電流只有 1 到 3A，5A 不會誤斷。

### 11.4 通電前檢查（用三用電表）

| 檢查 | 檔位 | 通過標準 |
|---|---|---|
| XT60 兩腳之間 | 通路檔（200 / •)))） | 不能持續嗶（不短路） |
| DC 頭中心與外殼之間 | 通路檔 | 不能持續嗶 |
| DC 頭電壓（接電池、不接 Jetson） | 直流電壓 20 | 紅筆插中心、黑筆碰外殼，約 **+12V** |

量電池時旋鈕一定要在「直流電壓」區，在電流或電阻檔碰電池會造成短路。

### 11.5 開關機順序

| 時機 | 順序 |
|---|---|
| 開機 | 先接電池 B（Jetson 開機），確認正常後再接電池 A |
| 關機 | 先拔電池 A → `sudo shutdown now` → 電源燈熄滅後拔電池 B → 拔掉低壓警報器 |

聞到焦味、冒煙或線材發燙：立刻從電池端拔掉 XT60。

---

## 12. 實機部署（Docker + 飛控）

架構：Agent 和 ROS 2 程式都跑在 Docker 容器 `humble` 裡（每次操作的步驟見第 0 節）。

```
飛控 ──USB 序列線（/dev/ttyUSB0）──→ Docker 容器 humble：MicroXRCEAgent
                                          ↕ DDS（ROS_DOMAIN_ID 一致）
                                     Docker 容器 humble：drone_mocap、RealSense、其他程式
```

### 12.1 一次性設定

**(1) 容器開機自動啟動**（本機）

```bash
docker update --restart unless-stopped humble
```

**(2) 容器內環境**

```bash
docker exec -it humble bash
apt update && apt install -y ros-humble-vrpn-mocap netbase
cd /root/shared/ros2_ws
MAKEFLAGS="-j2" colcon build --parallel-workers 1
grep -q "humble/setup.bash" ~/.bashrc || echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
grep -q "ros2_ws/install" ~/.bashrc || echo "source /root/shared/ros2_ws/install/setup.bash" >> ~/.bashrc
source ~/.bashrc
ros2 pkg list | grep -E "px4_msgs|drone_mocap|vrpn"
exit
```

`netbase` 提供 `/etc/protocols`。容器映像檔沒有這個檔案時，VRPN 會出現 `getprotobyname() failed` 與 `VRPN connection is bad`，連不上 Motive。

**(2-1) 容器內的 Micro XRCE-DDS Agent**

```bash
docker exec -it humble bash
apt update && apt install -y git cmake build-essential
cd /root
git clone -b v2.4.3 https://github.com/eProsima/Micro-XRCE-DDS-Agent.git
cd Micro-XRCE-DDS-Agent && mkdir build && cd build
cmake ..
make -j4          # Jetson 沒有 swap，卡住時改 -j2
make install
ldconfig /usr/local/lib/
which MicroXRCEAgent    # 應顯示 /usr/local/bin/MicroXRCEAgent
```

裝在容器的 `/root` 底下，`docker rm humble` 後要重裝。

**(3) 容器內的 RealSense**（容器裡的程式要用 D435i 時）

照 6.8 節以 RSUSB 從原始碼編譯。apt 版在這台讀不到 IMU。

相機節點要和使用它的程式放在同一邊（容器或本機擇一），同一台相機一次只能被一個程式使用。

**(4) 飛控參數**（QGC，改完重開飛控）

| 參數 | 值 |
|---|---|
| `MAV_1_CONFIG` | 0（關掉 TELEM2 上的 MAVLink） |
| `UXRCE_DDS_CFG` | TELEM2 |
| `SER_TEL2_BAUD` | 921600 |
| `UXRCE_DDS_DOM_ID` | 42（與容器 `ROS_DOMAIN_ID` 相同） |
| `SENS_BOARD_ROT` | 依飛控實際安裝方向 |

namespace：PX4 v1.17 的 `UXRCE_DDS_NS_IDX` 只能產生 `/uav_N` 格式。要用 `MAV2`，需在飛控 SD 卡 `etc/extras.txt` 以 `uxrce_dds_client start -n MAV2` 啟動，或沿用實驗室既有設定。

實驗室的群飛系統曾在網路上對 `/MAV3/fmu/in/vehicle_command` 發送指令。若這台的 namespace 與群飛系統用到的相同，且共用同一個 domain ID，測試前要先和負責群飛系統的人協調。

**(5)（建議）開機不進桌面**，省記憶體，全程用 SSH 操作：

```bash
sudo systemctl set-default multi-user.target
```

### 12.2 每次飛行

開機：接電池 B → 約 1 分鐘後用筆電 SSH 連入（`hostname -I` 查 IP，或 `ssh ncrl@ncrl.local`）。

| 終端機 | 位置 | 指令 |
|---|---|---|
| 1 | 容器 | `MicroXRCEAgent serial --dev /dev/ttyUSB0 -b 921600` |
| 2 | 容器 | `ros2 launch drone_mocap mocap.launch.py namespace:=MAV2 rigid_body:=MAV2 vrpn_server:=192.168.1.2` |
| 3 | 容器 | `ros2 topic list \| grep MAV2`、`ros2 topic echo /MAV2/fmu/out/estimator_status_flags --once \| grep cs_ev_` |
| 5（需要時） | 容器 | `ros2 launch realsense2_camera rs_launch.py` |

每個容器終端機都要先執行 `docker exec -it humble bash`。

結束：降落上鎖 → 拔電池 A → 各程式 `Ctrl + C` → `sudo shutdown now` → 電源燈熄滅後拔電池 B。

### 12.3 上機前檢查清單

| 項目 | 確認方式 |
|---|---|
| 容器狀態 | `docker ps` 顯示 `humble` 為 `Up` |
| Domain ID | 容器 `echo $ROS_DOMAIN_ID` 與飛控 `UXRCE_DDS_DOM_ID` 相同 |
| 飛控姿態 | QGC 姿態儀方向正確 |
| EKF | `cs_ev_pos`、`cs_ev_hgt`、`cs_ev_yaw` 全部 `true` |
| 電池 B | 低壓警報器顯示 15.2V 以上 |
| 系統時間 | `timedatectl` |

第一次照流程操作時不裝螺旋槳，完整跑一遍再裝槳試飛。

---

## 13. 動捕（Mocap）驗證

依據 [`drone_mocap`](https://github.com/yuhsiangsean/drone_mocap) 的 README。全程不裝螺旋槳，從資料源頭逐層確認。

```
Motive ──VRPN──→ vrpn_mocap ──→ drone_mocap ──→ Agent ──→ 飛控 EKF2
  ①                ②              ③                        ④ ⑤
```

### 13.1 飛控 EKF2 參數

| 參數 | 值 | 作用 |
|---|---|---|
| `EKF2_EV_CTRL` | 11 | 融合動捕的水平位置、高度、航向 |
| `EKF2_HGT_REF` | 3 | 高度以動捕為準 |
| `EKF2_BARO_CTRL` | 0 | 關閉氣壓計 |
| `EKF2_GPS_CTRL` | 0 | 室內無 GPS |
| `EKF2_MAG_TYPE` | 5 | 航向交給動捕 |
| `EKF2_EV_DELAY` | 40 | 動捕延遲補償（ms），改完要重開飛控 |

### 13.2 逐層驗證

**① Motive 與網路**
- Motive 建立剛體 `MAV2`，**建立時飛機水平放地上、機頭朝正前方**。
- 開啟 VRPN 串流（埠 3883）。
- Jetson 連上動捕場地網路，`ping <Motive 電腦 IP>`。

**② 動捕資料進入 ROS 2**（容器）

```bash
ros2 topic list | grep vrpn
ros2 topic hz /vrpn_mocap/MAV2/pose
ros2 topic echo /vrpn_mocap/MAV2/pose --once
```

- 頻率穩定，至少高於 5Hz。
- 飛機水平靜置時，四元數接近 `w≈1, x≈0, y≈0, z≈0`；否則回 Motive 重建剛體。
- 出現 `VRPN connection is bad`：檢查 IP 與網路。

**③ 橋接轉換**
- `drone_mocap` 每 5 秒回報收發數量，兩者都要持續增加。
- `ros2 topic hz /MAV2/fmu/in/vehicle_visual_odometry` 頻率與 ② 相近。

**④ 飛控融合**（Agent 要開著）

```bash
ros2 topic echo /MAV2/fmu/out/estimator_status_flags --once | grep cs_ev_
```

`cs_ev_pos`、`cs_ev_hgt`、`cs_ev_yaw` 全部 `true`。全部 `false` 時檢查 `EKF2_EV_CTRL` 與頻率；橋接有送但飛控沒反應時檢查 Agent、domain ID、namespace。

**⑤ 方向一致**

```bash
ros2 topic echo /MAV2/fmu/out/vehicle_local_position_v1 --once | grep -E "^(x|y|z):"
```

飛控為 NED，Motive（Z 朝上）為 ENU：

| 飛控 | 對應動捕 |
|---|---|
| x（北） | 動捕 y |
| y（東） | 動捕 x |
| z（下） | 動捕 z 的負值 |

手持飛機驗證：舉高 1 公尺 z 減少約 1；往動捕 +y 走 1 公尺 x 增加約 1；往動捕 +x 走 1 公尺 y 增加約 1；原地轉 90 度航向跟著轉；靜止時數值不飄移。

### 13.3 起飛條件

`cs_ev_pos` 未變成 `true` 之前不可起飛。①到⑤全部通過後才裝螺旋槳，第一次以定位模式（Position mode）手動低空懸停，確認穩定再進行自主飛行。
