# 学习 Nmap 安装和基本用法

本实验将带你从零开始学习 Nmap——网络探测与安全审计领域最强大的开源工具。你将掌握 Nmap 在各平台的安装方法、基本扫描语法与常用参数，并通过三个实战练习巩固对主机发现和端口扫描的理解。

## 学习目标

- 理解 Nmap 的功能定位、发展历史及典型应用场景
- 在 Linux、Windows 和 macOS 上独立完成 Nmap 的安装与验证
- 熟练使用 Nmap 基本扫描语法，掌握至少 15 个常用参数
- 能够阅读和解读 Nmap 的扫描输出结果
- 完成本地主机扫描、常见端口扫描和指定端口范围扫描三个实战练习
- 了解 Nmap 使用的法律边界与合规要求

## 背景知识

### Nmap 是什么

Nmap（Network Mapper）是一个开源的网络探测与安全审计工具，由 Gordon "Fyodor" Lyon 开发并维护。它能够快速扫描大型网络，发现网络上的主机、主机提供的服务（端口号）、操作系统类型，以及大量其他特征。Nmap 是网络安全从业人员必备的基础工具之一，被广泛应用于**渗透测试**、**漏洞评估**和**资产管理**等场景。

Nmap 的核心功能包括：

- **主机发现（Host Discovery）**：识别网络中的活跃主机
- **端口扫描（Port Scanning）**：探测目标主机开放的端口及运行的服务
- **版本探测（Version Detection）**：识别端口上运行的具体服务及版本号
- **操作系统指纹识别（OS Fingerprinting）**：通过 TCP/IP 栈特征判断目标操作系统
- **Nmap Scripting Engine（NSE）**：通过 Lua 脚本扩展检测能力，如漏洞扫描、暴力破解等

### Nmap 的历史和发展

| 时间 | 里程碑 |
|------|--------|
| 1997 年 | Fyodor 在 Phrack 杂志第 51 期发表 Nmap 文章，项目正式公开 |
| 1998 年 | Nmap 1.0 发布，支持基本 TCP 扫描 |
| 2000 年 | 加入操作系统指纹识别功能，成为业界标杆 |
| 2003 年 | 引入版本探测功能（`-sV`） |
| 2006 年 | Nmap Scripting Engine（NSE）发布，极大扩展了可定制化能力 |
| 2010 年 | Nmap 5.30 发布，加入 Ncat 和 Nping 等配套工具 |
| 2018 年 | 持续更新，NSE 脚本库超过 600 个 |
| 2023 年 | Nmap 7.94 发布，持续维护与社区活跃 |

Nmap 采用 **GPLv2** 许可证开源发布，代码托管在 GitHub 上（[https://github.com/nmap/nmap](https://github.com/nmap/nmap)），社区贡献活跃，至今仍是网络安全领域使用率最高的扫描工具。

### Nmap 的应用场景

Nmap 在以下场景中发挥着不可替代的作用：

**1. 安全审计（Security Auditing）**

安全团队使用 Nmap 定期扫描企业网络，发现未经授权开放的服务和端口，及时关闭安全风险点。这是 Nmap 最核心的应用场景。

```bash
# 对目标网络进行全面端口扫描
nmap -sS -sV -O 192.168.1.0/24
```

**2. 资产管理（Asset Management）**

IT 运维团队利用 Nmap 发现网络中的所有设备，建立和维护资产清单，确保没有"影子设备"脱离管理范围。

```bash
# 发现网络中的活跃主机
nmap -sn 192.168.1.0/24
```

**3. 漏洞扫描（Vulnerability Scanning）**

通过 NSE 脚本，Nmap 可以检测已知漏洞、弱口令和配置错误，辅助安全评估工作。

```bash
# 使用 NSE 脚本扫描常见漏洞
nmap --script vuln 192.168.1.100
```

**4. 网络拓扑发现（Network Topology Discovery）**

通过 traceroute 功能和主机发现，Nmap 可以帮助理解网络的拓扑结构，定位网络边界和关键节点。

```bash
# 扫描并执行 traceroute
nmap -sS --traceroute 192.168.1.0/24
```

**5. 防火墙规则验证（Firewall Rule Verification）**

管理员通过从外部和内部同时扫描，验证防火墙规则是否按预期生效，确保安全策略得到正确实施。

```bash
# 使用 ACK 扫描探测防火墙规则
nmap -sA 192.168.1.100
```

### 合法使用与法律边界

!!! warning "法律警告"
    未经授权扫描他人网络和系统在多数国家和地区属于**违法行为**。在中国，《网络安全法》和《刑法》第 285 条（非法侵入计算机信息系统罪）对未授权的网络扫描行为有明确的法律约束。

**合法使用原则：**

- **仅扫描自己拥有或获得明确授权的网络和系统**
- 在企业环境中，扫描前应获得**书面授权**和安全团队批准
- 渗透测试项目应签订正式的**授权协议（Rules of Engagement）**
- 教育和练习应使用**专用靶场环境**，如 LabEx 提供的实验环境
- 即使是善意扫描，也可能触发目标系统的**入侵检测系统（IDS）**告警

**安全练习建议：**

- 使用本地回环地址 `127.0.0.1` 或实验环境进行练习
- 不要对公共网站或他人 IP 地址进行扫描
- 在虚拟机中搭建靶场环境进行进阶练习

## 环境准备

### 系统要求

Nmap 具有优秀的跨平台支持，可以在以下操作系统上运行：

| 平台 | 最低要求 | 推荐配置 |
|------|----------|----------|
| Linux | 内核 2.4+，glibc 2.3+ | Ubuntu 20.04+ / Debian 11+ / CentOS 8+ |
| Windows | Windows 7 / Server 2008 R2+ | Windows 10/11，需 WinPcap 或 Npcap |
| macOS | macOS 10.12+ | macOS 12+（Apple Silicon 原生支持） |

### 安装方法

#### Linux（Debian/Ubuntu）

```bash
# 更新软件包索引
sudo apt update

# 安装 Nmap
sudo apt install nmap -y

# 验证安装
nmap --version
```

#### Linux（RHEL/CentOS/Fedora）

```bash
# CentOS/RHEL 使用 yum 或 dnf
sudo dnf install nmap -y

# Fedora
sudo dnf install nmap -y

# 验证安装
nmap --version
```

#### Linux（从源码编译）

如果需要最新版本或软件仓库中版本过旧，可以从源码编译：

```bash
# 安装编译依赖
sudo apt install build-essential libssl-dev -y

# 下载源码
wget https://nmap.org/dist/nmap-7.94.tar.gz
tar -xzf nmap-7.94.tar.gz
cd nmap-7.94

# 配置、编译和安装
./configure
make
sudo make install

# 验证安装
nmap --version
```

#### Windows

```bash
# 方法1：从官网下载安装包
# 访问 https://nmap.org/download.html
# 下载 Windows 版本安装程序（自带 Npcap）
# 运行安装程序，按向导完成安装

# 方法2：使用包管理器 Winget
winget install Insecure.Nmap

# 方法3：使用 Chocolatey
choco install nmap -y
```

!!! note "Windows 注意事项"
    Windows 版本需要安装 **Npcap**（数据包捕获驱动），Nmap 安装程序通常自带 Npcap 安装选项。如果未安装 Npcap，Nmap 将无法执行原始套接字扫描（如 SYN 扫描）。安装时请确保勾选"Install Npcap"选项。

#### macOS

```bash
# 方法1：使用 Homebrew（推荐）
brew install nmap

# 方法2：使用 MacPorts
sudo port install nmap

# 方法3：从官网下载 .dmg 安装包
# 访问 https://nmap.org/download.html 下载 macOS 版本

# 验证安装
nmap --version
```

### 验证安装成功

安装完成后，通过以下命令验证 Nmap 是否正确安装：

```bash
# 查看版本信息
nmap --version
```

预期输出示例：

```
Nmap version 7.94 ( https://nmap.org )
Platform: x86_64-unknown-linux-gnu
Compiled with: nmap-liblua-5.4.4 openssl-3.0.9 nmap-libpcre-7.8 nmap-libdnet-1.12 ipv6
Available nse scripts: 598
```

```bash
# 查看帮助信息（确认命令可用）
nmap -h
```

如果看到版本号和帮助信息输出，说明 Nmap 已经**安装成功**，可以开始使用了。

## 基本用法

### 基本扫描语法

Nmap 的基本命令格式如下：

```
nmap [扫描类型] [选项] [目标]
```

其中：

- **扫描类型**：决定 Nmap 使用何种方式探测目标，如 SYN 扫描、TCP 连接扫描等
- **选项**：控制扫描行为的各种参数，如端口范围、超时时间、输出格式等
- **目标**：可以是单个 IP、IP 范围、CIDR 网段或主机名

**目标指定方式：**

```bash
# 单个 IP 地址
nmap 192.168.1.1

# 多个 IP 地址
nmap 192.168.1.1 192.168.1.2 192.168.1.3

# IP 范围
nmap 192.168.1.1-10

# CIDR 网段
nmap 192.168.1.0/24

# 主机名
nmap scanme.nmap.org

# 排除特定主机
nmap 192.168.1.0/24 --exclude 192.168.1.1,192.168.1.2
```

### 常用参数表

| 参数 | 全称 | 说明 | 示例 |
|------|------|------|------|
| `-sS` | SYN Scan | **半开连接扫描**，默认扫描方式，速度快且隐蔽 | `nmap -sS 192.168.1.1` |
| `-sT` | TCP Connect Scan | **全连接扫描**，完成完整 TCP 三次握手，无需 root 权限 | `nmap -sT 192.168.1.1` |
| `-sU` | UDP Scan | **UDP 端口扫描**，速度较慢，用于发现 UDP 服务 | `nmap -sU 192.168.1.1` |
| `-sn` | Ping Scan | **主机发现**，仅探测主机是否在线，不扫描端口 | `nmap -sn 192.168.1.0/24` |
| `-sV` | Version Detection | **版本探测**，识别端口上运行的具体服务及版本 | `nmap -sV 192.168.1.1` |
| `-O` | OS Detection | **操作系统指纹识别**，通过 TCP/IP 栈特征判断目标 OS | `nmap -O 192.168.1.1` |
| `-A` | Aggressive Scan | **激进扫描**，同时启用 OS 检测、版本探测、脚本扫描和 traceroute | `nmap -A 192.168.1.1` |
| `-p` | Port Specification | **指定端口**，扫描特定端口或端口范围 | `nmap -p 80,443 192.168.1.1` |
| `-p-` | All Ports | **扫描全部 65535 个端口**，耗时较长但结果最完整 | `nmap -p- 192.168.1.1` |
| `-F` | Fast Mode | **快速模式**，仅扫描最常用的 100 个端口（默认 1000 个） | `nmap -F 192.168.1.1` |
| `-T4` | Timing Template | **时序模板**，T0（慢）到 T5（快），T4 为推荐值 | `nmap -T4 192.168.1.1` |
| `-v` | Verbose | **详细输出**，显示更多扫描过程信息 | `nmap -v 192.168.1.1` |
| `-oN` | Normal Output | **标准格式输出**到文件 | `nmap -oN result.txt 192.168.1.1` |
| `-oX` | XML Output | **XML 格式输出**到文件，便于程序解析 | `nmap -oX result.xml 192.168.1.1` |
| `-iL` | Input from List | **从文件读取目标列表**，批量扫描 | `nmap -iL targets.txt` |
| `--script` | NSE Script | **指定 NSE 脚本**执行高级检测 | `nmap --script=vuln 192.168.1.1` |
| `-Pn` | No Ping | **跳过主机发现**，直接对所有目标进行端口扫描 | `nmap -Pn 192.168.1.1` |
| `-n` | No DNS Resolution | **不做 DNS 反向解析**，加快扫描速度 | `nmap -n 192.168.1.1` |

### 扫描结果解读

让我们通过一个实际的扫描输出结果来理解 Nmap 报告的含义：

```bash
nmap -sS -sV 192.168.1.100
```

预期输出：

```
Starting Nmap 7.94 ( https://nmap.org ) at 2024-01-15 10:30 CST
Nmap scan report for 192.168.1.100
Host is up (0.0010s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 8.9p1 Ubuntu 3ubuntu0.1
80/tcp   open  http        Apache httpd 2.4.52
443/tcp  open  https       nginx 1.18.0
3306/tcp open  mysql       MySQL 8.0.32

Nmap done: 1 IP address (1 host up) scanned in 8.42 seconds
```

**逐行解读：**

| 字段 | 含义 |
|------|------|
| `Starting Nmap 7.94` | Nmap 版本号及扫描开始时间 |
| `Host is up` | 目标主机在线，延迟为 0.0010 秒 |
| `Not shown: 996 closed tcp ports` | 996 个端口已关闭，为简洁起见不逐一显示 |
| `PORT` | 端口号和协议（tcp/udp） |
| `STATE` | 端口状态：`open`（开放）、`closed`（关闭）、`filtered`（被防火墙过滤） |
| `SERVICE` | 该端口对应的服务名称 |
| `VERSION` | 服务的具体版本信息（需 `-sV` 参数） |
| `Nmap done` | 扫描统计：扫描 IP 数、在线主机数、耗时 |

**端口状态详解：**

| 状态 | 含义 | 可能原因 |
|------|------|----------|
| `open` | 端口开放，有应用程序在此端口监听连接 | 目标主机运行了对应服务 |
| `closed` | 端口关闭，目标主机可达但无应用监听 | 未运行对应服务，但 TCP RST 响应表明主机在线 |
| `filtered` | 端口被过滤，无法确定是否开放 | 防火墙规则阻止了探测数据包 |
| `open\|filtered` | 无法确定端口是开放还是被过滤 | UDP 扫描或某些特殊扫描类型中出现 |
| `closed\|filtered` | 无法确定端口是关闭还是被过滤 | 在 IDLE 扫描等特殊场景中出现 |

## 实战练习

以下练习请在**授权的实验环境**中完成，请勿对未经授权的目标进行扫描。

### 练习1：扫描本地主机

**目标**：使用 Nmap 扫描本地回环地址，发现本机开放的端口。

**完整命令**：

```bash
# 使用 TCP 连接扫描本地主机（无需 root 权限）
nmap -sT 127.0.0.1
```

**预期输出示例**：

```
Starting Nmap 7.94 ( https://nmap.org ) at 2024-01-15 10:35 CST
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00010s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
631/tcp  open  ipp
5432/tcp open  postgresql

Nmap done: 1 IP address (1 host up) scanned in 0.45 seconds
```

**结果分析**：

- 本机开放了 3 个端口：SSH（22）、IPP 打印服务（631）和 PostgreSQL 数据库（5432）
- `Not shown: 997 closed tcp ports` 表示其余 997 个默认扫描端口均处于关闭状态
- 扫描耗时仅 0.45 秒，因为目标是本地回环地址，网络延迟极低

!!! tip "提示"
    使用 `-sT`（TCP Connect Scan）扫描本地主机无需 root/sudo 权限。如果拥有 root 权限，可以使用 `-sS`（SYN Scan）获得更快的扫描速度。

### 练习2：扫描常见端口

**目标**：使用 Nmap 扫描实验环境目标主机的常见服务端口，并识别服务版本。

**完整命令**：

```bash
# 扫描常见端口并探测服务版本
nmap -sS -sV -F 192.168.1.100
```

**参数解析**：

- `-sS`：SYN 半开连接扫描，速度快且隐蔽
- `-sV`：版本探测，识别端口上运行的具体服务及版本号
- `-F`：快速模式，仅扫描最常用的 100 个端口

**预期输出示例**：

```
Starting Nmap 7.94 ( https://nmap.org ) at 2024-01-15 10:40 CST
Nmap scan report for 192.168.1.100
Host is up (0.0035s latency).
Not shown: 94 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.5
22/tcp   open  ssh         OpenSSH 8.9p1 Ubuntu 3ubuntu0.1
80/tcp   open  http        Apache httpd 2.4.52
110/tcp  open  pop3        Dovecot pop3d
143/tcp  open  imap        Dovecot imapd
443/tcp  open  https       nginx 1.18.0
3306/tcp open  mysql       MySQL 8.0.32

Service detection performed. Please report any incorrect results at https://nmap.org/
Nmap done: 1 IP address (1 host up) scanned in 12.87 seconds
```

**结果分析**：

- 目标主机开放了 7 个常见服务端口
- 版本探测成功识别了每个服务的具体版本号，这些信息对安全评估至关重要
- 例如，发现 `vsftpd 3.0.5`，可以进一步查询该版本是否存在已知漏洞

### 练习3：扫描指定端口范围

**目标**：扫描目标主机的特定端口范围，精确控制扫描范围以节省时间。

**完整命令**：

```bash
# 扫描 Web 服务相关端口范围（1-1024）
nmap -sS -p 1-1024 192.168.1.100
```

**其他端口指定方式**：

```bash
# 扫描特定端口
nmap -sS -p 22,80,443,3306,8080 192.168.1.100

# 扫描全部 65535 个端口
nmap -sS -p- 192.168.1.100

# 扫描常用 Web 端口范围
nmap -sS -p 80,443,8000-8100,8443 192.168.1.100

# 使用服务名称指定端口
nmap -sS -p http,https,ssh 192.168.1.100
```

**预期输出示例**：

```
Starting Nmap 7.94 ( https://nmap.org ) at 2024-01-15 10:50 CST
Nmap scan report for 192.168.1.100
Host is up (0.0035s latency).
Not shown: 1019 closed tcp ports (reset)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
110/tcp  open  pop3
143/tcp  open  imap
443/tcp  open  https
3306/tcp open  mysql
5900/tcp open  vnc

Nmap done: 1 IP address (1 host up) scanned in 2.15 seconds
```

**结果分析**：

- 指定端口范围 `1-1024` 后，扫描仅覆盖 1024 个端口，比默认的 1000 个端口略多
- 相比全端口扫描（`-p-`），指定范围大幅缩短了扫描时间
- 在 1-1024 范围内发现了 8 个开放端口，包括新发现的 VNC 服务（5900）

!!! tip "性能优化技巧"
    对于大型网络扫描，可以组合使用以下参数优化性能：

    ```bash
    # 快速扫描 + 时序优化 + 禁用 DNS 解析
    nmap -F -T4 -n 192.168.1.0/24
    ```

    - `-F`：减少扫描端口数
    - `-T4`：加快扫描速度（适用于稳定网络）
    - `-n`：跳过 DNS 反向解析

## 常见问题

### Q1：Nmap 扫描需要 root 权限吗？

**A**：取决于扫描类型。**SYN 扫描**（`-sS`）、**UDP 扫描**（`-sU`）和**操作系统检测**（`-O`）需要 root/sudo 权限，因为它们需要构造原始数据包（raw socket）。**TCP Connect 扫描**（`-sT`）和**版本探测**（`-sV`）不需要 root 权限。建议在 Linux 上始终使用 `sudo` 运行 Nmap 以获得完整的扫描能力。

```bash
# 需要 root 权限的扫描
sudo nmap -sS 192.168.1.1

# 不需要 root 权限的扫描
nmap -sT 192.168.1.1
```

### Q2：SYN 扫描和 TCP Connect 扫描有什么区别？

**A**：SYN 扫描（`-sS`）只发送 SYN 包，收到 SYN/ACK 后立即发送 RST 断开连接，不会建立完整的 TCP 连接，因此速度更快且日志记录更少。TCP Connect 扫描（`-sT`）完成完整的三次握手，速度较慢但不需要 root 权限。在渗透测试中，SYN 扫描更隐蔽；在日常安全检查中，TCP Connect 扫描更方便。

### Q3：扫描速度太慢怎么办？

**A**：可以通过以下方式优化扫描速度：

```bash
# 1. 使用时序模板加速（T0最慢 - T5最快，推荐T4）
nmap -T4 192.168.1.1

# 2. 仅扫描常用端口
nmap -F 192.168.1.1

# 3. 禁用 DNS 反向解析
nmap -n 192.168.1.1

# 4. 并行扫描多个主机
nmap --min-parallelism 100 192.168.1.0/24

# 5. 设置超时时间
nmap --host-timeout 30m 192.168.1.0/24
```

!!! warning "注意"
    时序模板 T5 速度最快，但可能导致扫描结果不准确，尤其在网络延迟较高时。**生产环境推荐使用 T4**。

### Q4：如何将扫描结果保存到文件？

**A**：Nmap 支持多种输出格式，可以同时指定多个输出选项：

```bash
# 标准文本格式
nmap -oN scan_result.txt 192.168.1.1

# XML 格式（便于程序解析和导入其他工具）
nmap -oX scan_result.xml 192.168.1.1

# Grep 可解析格式
nmap -oG scan_result.gnmap 192.168.1.1

# 同时输出所有格式
nmap -oA scan_result 192.168.1.1
```

`-oA` 参数会同时生成 `.nmap`、`.xml` 和 `.gnmap` 三个文件，是最常用的输出方式。

### Q5：Nmap 扫描会被防火墙拦截吗？

**A**：会。防火墙可能导致端口显示为 `filtered` 状态，表示无法确定端口是否开放。解决方法：

- 使用 `-Pn` 参数跳过主机发现阶段，直接进行端口扫描
- 使用 `-sA`（ACK 扫描）探测防火墙规则
- 尝试不同的扫描类型绕过特定防火墙规则
- 调整时序模板（降低速率可能避免触发 IDS/IPS）

```bash
# 跳过 Ping 直接扫描
nmap -Pn 192.168.1.1

# ACK 扫描探测防火墙
nmap -sA 192.168.1.1
```

### Q6：如何扫描整个子网？

**A**：使用 CIDR 表示法指定子网范围：

```bash
# 扫描 /24 子网（254 台主机）
nmap -sn 192.168.1.0/24

# 扫描 /16 子网（65534 台主机），建议使用快速模式
nmap -F -T4 10.0.0.0/16

# 排除特定主机
nmap 192.168.1.0/24 --exclude 192.168.1.1,192.168.1.250
```

### Q7：Nmap 和 Masscan 有什么区别？

**A**：两者定位不同。**Nmap** 功能全面，支持主机发现、端口扫描、版本探测、OS 检测和 NSE 脚本，适合精确扫描和深度检测。**Masscan** 专注于极速端口扫描，号称每秒可发送 1000 万个数据包，适合大规模网络的快速端口发现，但不支持版本探测和 OS 检测。在实际工作中，常用 Masscan 快速发现开放端口，再用 Nmap 对目标进行深度扫描。

### Q8：Nmap 可以扫描 UDP 端口吗？

**A**：可以，使用 `-sU` 参数。但 UDP 扫描比 TCP 扫描慢得多，因为 UDP 协议本身不提供可靠的响应机制。建议仅扫描已知的关键 UDP 端口：

```bash
# 扫描常见 UDP 端口
nmap -sU -p 53,67,68,123,161,500 192.168.1.1

# 同时进行 TCP 和 UDP 扫描
nmap -sS -sU -p T:1-1024,U:53,123,161 192.168.1.1
```

## 总结

本实验涵盖了 Nmap 从安装到实战使用的核心知识，以下是关键要点回顾：

1. **Nmap 是网络安全领域最重要的开源扫描工具**，支持主机发现、端口扫描、版本探测、OS 指纹识别和 NSE 脚本扩展

2. **安装简单**：Linux 使用包管理器（`apt`/`dnf`），macOS 使用 Homebrew，Windows 下载安装包或使用 Winget/Chocolatey

3. **基本语法**：`nmap [扫描类型] [选项] [目标]`，目标支持 IP、IP 范围、CIDR 网段和主机名

4. **常用扫描类型**：
   - `-sS`：SYN 扫描（默认，需 root）
   - `-sT`：TCP Connect 扫描（无需 root）
   - `-sn`：仅主机发现
   - `-sV`：版本探测
   - `-O`：OS 检测

5. **端口状态**：`open`（开放）、`closed`（关闭）、`filtered`（被过滤）

6. **性能优化**：`-T4`（推荐时序）、`-F`（快速模式）、`-n`（禁用 DNS 解析）

7. **输出格式**：`-oA` 同时输出所有格式，便于存档和后续分析

8. **法律合规**：始终确保获得明确授权后再扫描，仅对自有或授权的系统进行扫描

!!! success "恭喜"
    你已经完成了 Nmap 安装和基本用法的学习！掌握了这些基础知识后，你可以继续学习 Nmap 的高级功能，如 NSE 脚本编写、规避技术（evasion）和 Zenmap 图形界面的使用。

## 参考资料

### 官方资源

- **Nmap 官方网站**：[https://nmap.org](https://nmap.org)
- **Nmap 官方文档**：[https://nmap.org/book/man.html](https://nmap.org/book/man.html)
- **NSE 脚本文档**：[https://nmap.org/nsedoc/](https://nmap.org/nsedoc/)
- **Nmap GitHub 仓库**：[https://github.com/nmap/nmap](https://github.com/nmap/nmap)

### 推荐书籍

| 书名 | 作者 | 说明 |
|------|------|------|
| 《Nmap Network Scanning》 | Gordon "Fyodor" Lyon | Nmap 作者亲著，最权威的参考手册 |
| 《Nmap Cookbook》 | Paulino Calderon | 实战导向的 Nmap 使用指南 |
| 《Network Security Assessment》 | Chris McNab | 涵盖 Nmap 在安全评估中的实践应用 |

### 在线资源

- **Nmap 参考指南（中文翻译）**：[https://nmap.org/man/zh/](https://nmap.org/man/zh/)
- **LabEx Nmap 技能树**：[https://labex.io/skilltrees/nmap](https://labex.io/skilltrees/nmap)
- **Nmap SEC505 课程资料**：SANS Institute 提供的系统安全课程

### 实践环境

- **LabEx 实验平台**：提供预配置的 Nmap 实验环境
- **Metasploitable**：专为安全测试设计的漏洞靶机虚拟机
- **Vulnhub**：提供多种可下载的渗透测试靶机镜像
