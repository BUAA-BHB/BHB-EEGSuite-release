# BHB EEGSuite

北航 BHB 团队脑电（EEG）采集软件发行仓库。同一版本号下提供两种发行形态，按需下载其一即可，无需同时安装。

| 发行形态 | 界面 | 适用场景 |
| --- | --- | --- |
| Windows Web 上位机 | 浏览器图形界面 | 实验室常规采集：实时波形、阻抗、tDCS 控制 |
| Linux 终端轻量版 | 命令行交互 | Linux 工作站/服务器采集、脚本批处理、LSL 对接 |

## 选择与下载

1. 打开 [Releases 页面](https://github.com/BUAA-BHB/BHB-EEGSuite-release/releases) 选择需要的版本
2. Windows 版下载 `BHB-EEGSuite-base.zip`
3. Linux 版下载 `BHB-EEGSuite-Linux-v<版本号>.tar.gz`
4. 两个发行包功能侧重不同，版本号保持一致

## Windows Web 上位机

### 系统要求

- Windows 11（64 位）
- 解压目录必须可写（用于保存配置、模块和实验数据），建议放在 D 盘或「文档」目录
- 请完整解压后再运行，不要在 ZIP 压缩包内直接双击 exe

### 三步开始

1. **下载**：`BHB-EEGSuite-base.zip`
2. **解压**：右键 ZIP → 全部解压，选一个可写目录（如 `D:\BHB-EEGSuite`）
3. **运行**：进入解压目录，双击 `BHB-EEGSuite.exe`

首次启动约 10–30 秒用于初始化运行环境，随后自动打开浏览器进入主页；若未弹出，手动访问 <http://127.0.0.1:8001/>。

### 工作模式

启动后先进入设备页：扫描并连接蓝牙采集设备，选择通道组合后点击「应用到系统」，进入模式页。

| 模式 | 用途 |
| --- | --- |
| **EEG** | 实时脑电波形 + 离线录制，停止后可导出 CSV / EDF（支持带通滤波） |
| **阻抗** | 周期性刷新各通道阻抗，用颜色反映电极接触质量 |
| **tDCS** | 经颅电刺激控制：开/停刺激、下发指令、监测电流/电压/故障/电量 |

可选「音乐范式模块」：上位机「模块管理」中在线安装，无需单独下载。

### 常见问题（Windows）

- **端口 8001 被占用**：多为上次未正常退出，重启电脑后重试
- **双击 exe 无反应**：确认已解压后再运行；右键以管理员身份运行；换到可写目录重试
- **浏览器打不开页面**：手动访问 <http://127.0.0.1:8001/>；任务管理器确认进程存在；首次启动弹防火墙时选择「允许访问」
- **扫描不到设备 / 无波形**：确认设备开机且电量充足、电脑蓝牙已开启、设备在 1 米内；模式页点击「开始」后才开始采集
- **导出失败**：确认磁盘剩余空间充足且解压目录可写；可先试 CSV 再试 EDF
- **提示缺少 dll**：安装 [Microsoft Visual C++ Redistributable x64](https://aka.ms/vs/17/release/vc_redist.x64.exe) 后重试

## Linux 终端轻量版

### 环境要求

- Linux x86_64
- **Python 3.11**：发行包由 Cython 编译为原生扩展，与构建时的 Python 版本绑定，其他版本无法加载
- 可用的蓝牙适配器，当前用户需具备访问权限（通常加入 `bluetooth` 用户组）
- 可选依赖：`pylsl`（LSL 推流）、`pyedflib`（EDF 导出），已包含在 `requirements.txt` 中

### 安装与运行

```bash
tar -xzf BHB-EEGSuite-Linux-v3.6.0.tar.gz
cd BHB-EEGSuite-Linux
python3 -m venv .venv
source .venv/bin/activate
python3 -m pip install -r requirements.txt
./bhb-eegsuite
```

若系统限制全局 pip 安装（PEP 668），使用上面的虚拟环境方式即可。

### 主菜单

| 选项 | 操作 |
| --- | --- |
| `1` | 扫描蓝牙设备（约 5 秒，按 RSSI 排序） |
| `2` | 连接设备（按扫描结果编号选择） |
| `3` | 断开设备 |
| `4` | 开始 EEG 采集（回车停止，数据自动保存） |
| `5` | 开始阻抗检测（回车停止，每秒刷新） |
| `6` | 查看设备状态（连接、模式、通道、电量、数据目录） |
| `7` | 导出数据（CSV / EDF / CSV+EDF） |
| `0` | 退出并清理资源 |

典型流程：`1` 扫描 → `2` 连接 → `5` 检查阻抗 → `4` 采集 → `7` 导出。

### 数据与配置

- `configs/config.yaml`：设备名前缀、采集与导出参数等可编辑；帧协议与指令参数已编译进二进制，发行版配置中不含这些段
- `configs/config.local.yaml`：运行时自动生成，保存本机通道模式偏好
- `offlinedata/`：会话原始数据与导出文件

### 常见问题（Linux）

- **扫描不到设备**：确认蓝牙服务运行（`systemctl status bluetooth`）；当前用户加入 `bluetooth` 组后需重新登录
- **`ModuleNotFoundError`**：确认使用 Python 3.11 并已执行 `pip install -r requirements.txt`
- **`./bhb-eegsuite` 无法执行**：需在解压目录内运行；必要时 `chmod +x bhb-eegsuite`
- **EDF 导出失败**：安装 `pyedflib`
- **LSL 无数据**：安装 `pylsl`，并确认接收端与发送端网络可达
