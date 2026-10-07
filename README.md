# Jetson Orin Nano 交接手冊
 
本手冊記錄實驗室這台 Jetson Orin Nano 的環境設定、版本、參數與安裝指令。要把專案放到這台機器上時，可以在這裡查到需要配合的設定。
 
- 系統安裝日期：2026-10-06
- 最後更新：2026-10-07
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
 
---
 
## 目錄
 
1. [目錄結構與環境設定](#1-目錄結構與環境設定)
2. [SSH](#2-ssh)
3. [切換GUI、server指令](#3-切換guiserver指令)
4. [硬體型號](#4-硬體型號)
5. [版本資訊](#5-版本資訊)
6. [安裝指令](#6-安裝指令)
7. [模擬啟動流程](#7-模擬啟動流程)
8. [ROS_DOMAIN_ID（重要）](#8-ros_domain_id重要)
9. [注意事項與踩過的坑](#9-注意事項與踩過的坑)
---
 
## 1. 目錄結構與環境設定
 
```
~/
├── PX4-Autopilot/           飛控韌體原始碼，用 make 編譯
├── Micro-XRCE-DDS-Agent/    Agent 原始碼（已安裝到系統，平常不用動）
├── QGroundControl-aarch64.AppImage
└── ros2_ws/                 ROS 2 workspace，專案都放這裡
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
 
（2026-10-07 更新）
 
| 軟體 | 版本 | 位置 |
|---|---|---|
| ROS 2 | Jazzy（desktop） | `/opt/ros/jazzy` |
| PX4-Autopilot 原始碼 | v1.17.0（`release/1.17` 分支） | `~/PX4-Autopilot` |
| px4_msgs | `release/1.17` | `~/ros2_ws/src/px4_msgs` |
| px4_ros_com | `release/1.16` | `~/ros2_ws/src/px4_ros_com` |
| Micro XRCE-DDS Agent | v2.4.3 | 原始碼 `~/Micro-XRCE-DDS-Agent`，執行檔在 `/usr/local` |
| QGroundControl | aarch64 AppImage（最新穩定版） | `~/QGroundControl-aarch64.AppImage` |
 
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
 
需要重灌或在另一台 Jetson 上重建環境時，依 6.1 到 6.4 的順序做。
 
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
 
**常用檢查指令**
 
```bash
df -h /              # 硬碟剩餘空間
free -h              # 記憶體
sudo tegrastats      # CPU、GPU、溫度、耗電（Ctrl+C 結束）
ros2 pkg list | grep px4
```
