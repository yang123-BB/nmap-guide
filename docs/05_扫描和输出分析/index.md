# 第5章 Nmap 扫描和输出分析

> **难度**：⭐⭐⭐（初中级）
>
> **预计学习时间**：60 分钟
>
> **适用场景**：渗透测试报告编写、安全审计数据分析、自动化扫描结果处理

---

## 5.1 概述

### 场景引入

设想你是一名安全工程师，负责对公司内部网络进行定期的安全评估。你执行了一次 Nmap 扫描，扫描了 2000 台主机，发现了 15,000 个开放端口。但问题是——**你如何从这海量的原始输出中找到真正有价值的信息？**

仅仅运行 Nmap 扫描只是第一步。真正的挑战在于：

- **筛选关键信息**：从数千行输出中快速定位开放端口和运行的服务
- **识别安全风险**：发现非授权服务、高危端口和配置不当的服务
- **生成可读报告**：将扫描结果整理成管理层可理解的格式
- **自动化处理**：对大规模扫描结果进行程序化分析
- **长期跟踪**：对比不同时间点的扫描结果，发现变化

这正是本章要解决的问题。我们将深入探讨 Nmap 的多种输出格式，学习如何高效地解读和分析扫描结果，掌握将原始数据转化为 actionable intelligence（可操作情报）的技巧。

无论是渗透测试报告、合规审计还是日常安全运维，输出分析能力都是一项核心技能。**一次扫描的价值，不在于你发现了多少端口，而在于你如何理解和利用这些发现。**

---

## 5.2 学习目标

完成本章学习后，你将能够：

| # | 目标 | 对应技能 |
|---|------|----------|
| 1 | 理解 Nmap 的四种标准输出格式及其适用场景 | 文件格式识别与选择 |
| 2 | 熟练使用 `-oN` `-oX` `-oG` `-oS` 参数保存扫描结果 | 命令参数运用 |
| 3 | 能够解读 Nmap 扫描报告中的各类状态和标记 | 报告解读 |
| 4 | 掌握使用 `grep`、`awk`、`sed` 等工具过滤和提取 Nmap 输出中的关键信息 | 文本处理 |
| 5 | 学会将 XML 格式的扫描结果导入数据库进行查询分析 | 数据持久化 |
| 6 | 能够编写自动化脚本批量处理和分析 Nmap 扫描结果 | 脚本编程 |
| 7 | 掌握在实战中根据输出分析做出安全决策的方法 | 综合应用 |

---

## 5.3 背景知识：Nmap 输出格式详解

Nmap 提供多种输出格式，每种格式都有其独特的用途和优势。理解这些格式是进行有效输出分析的前提。

### 5.3.1 正常输出（`-oN`）

**格式说明**：`-oN` 参数将扫描结果以人类可读的格式保存到文件。这是最常用的输出格式，设计目标是让安全分析人员能直接阅读。

**命令示例**：

```bash
nmap -sS -sV -p 1-1000 scanme.nmap.org -oN scan_normal.txt
```

**输出示例**（`scan_normal.txt`）：

```
# Nmap 7.94 scan initiated 2026-06-07 10:30:00
Nmap scan report for scanme.nmap.org (45.33.32.156)
Host is up (0.15s latency).

PORT      STATE    SERVICE    VERSION
22/tcp    open     ssh        OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
25/tcp    filtered smtp
80/tcp    open     http       Apache httpd 2.4.7 ((Ubuntu))
135/tcp   filtered msrpc
139/tcp   filtered netbios-ssn
443/tcp   open     https      Apache httpd 2.4.7 ((Ubuntu))
445/tcp   filtered microsoft-ds
9929/tcp  open     nping-echo Nping echo
31337/tcp open     tcpwrapped

# Nmap done at 2026-06-07 10:30:45 -- 1 IP address (1 host up) scanned in 45.12 seconds
```

**特点**：

- ✅ **人类可读性最佳**：一眼就能看出主机状态、端口开放情况和服务版本
- ✅ **包含扫描元数据**：扫描时间、扫描方式、总耗时等信息
- ✅ **适合直接用于报告**：无需额外处理即可粘贴到报告中
- ❌ **解析困难**：不适合程序化提取数据，结构不够严格
- ❌ **占用空间较大**：夹杂了大量格式化和描述性文本

### 5.3.2 XML 输出（`-oX`）

**格式说明**：`-oX` 参数将扫描结果保存为结构化的 XML 格式。这是最强大、最灵活的格式，非常适合程序化处理。

**命令示例**：

```bash
nmap -sS -sV -p 1-1000 scanme.nmap.org -oX scan_result.xml
```

**输出示例**（`scan_result.xml` 核心片段）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE nmaprun PUBLIC "-//IDN nmap.org//DTD Nmap XML 1.04//EN"
 "https://svn.nmap.org/nmap/docs/nmaprun.dtd">
<?xml-stylesheet href="file:///usr/bin/../share/nmap/nmap.xsl" type="text/xsl"?>
<nmaprun scanner="nmap" args="nmap -sS -sV -p 1-1000 scanme.nmap.org" start="1717741800"
  startstr="Sun Jun  7 10:30:00 2026" version="7.94" xmloutputversion="1.05">
<verbose level="0"/>
<debugging level="0"/>
<scaninfo type="syn" protocol="tcp" numservices="1000" services="1-1000"/>
<host starttime="1717741800" endtime="1717741845">
  <status state="up" reason="localhost-response" reason_ttl="0"/>
  <address addr="45.33.32.156" addrtype="ipv4"/>
  <hostnames>
    <hostname name="scanme.nmap.org" type="user"/>
  </hostnames>
  <ports>
    <port protocol="tcp" portid="22">
      <state state="open" reason="syn-ack" reason_ttl="0"/>
      <service name="ssh" product="OpenSSH" version="6.6.1p1" extrainfo="Ubuntu Linux; protocol 2.0"
        ostype="Linux" method="probed" conf="10"/>
    </port>
    <port protocol="tcp" portid="80">
      <state state="open" reason="syn-ack" reason_ttl="0"/>
      <service name="http" product="Apache httpd" version="2.4.7"
        extrainfo="(Ubuntu)" method="probed" conf="10"/>
    </port>
    <port protocol="tcp" portid="25">
      <state state="filtered" reason="no-response"/>
    </port>
  </ports>
</host>
<runstats>
  <finished time="1717741845" timestr="Sun Jun  7 10:30:45 2025"
    summary="Nmap done at Sun Jun  7 10:30:45 2025; 1 IP address (1 host up) scanned in 45.12 seconds"
    elapsed="45.12" exit="success"/>
</runstats>
</nmaprun>
```

**特点**：

- ✅ **结构严格**：XML Schema/DTD 定义了完整的结构规范
- ✅ **程序友好**：易于用各种编程语言解析（Python 的 `xml.etree`、Perl 的 `XML::Simple` 等）
- ✅ **信息完整**：包含所有扫描结果，没有任何信息丢失
- ✅ **支持 XSLT 转换**：可使用 XSL 样式表转换为 HTML 或其他格式
- ✅ **可导入数据库**：结构化数据可直接插入关系型数据库
- ❌ **可读性差**：直接阅读 XML 标签非常费眼

### 5.3.3 Grepable 输出（`-oG`）

**格式说明**：`-oG` 参数将扫描结果保存为一种"一行一个主机"的紧凑格式，专门为 `grep` 命令优化。每个主机的所有端口信息在一行内展示。

**命令示例**：

```bash
nmap -sS -sV -p 1-1000 scanme.nmap.org -oN -oG scan_grepable.gnmap
```

> **注意**：`-oN` 后面的 `-` 表示将正常输出同时发送到标准输出（终端），便于实时观察。

**输出示例**（`scan_grepable.gnmap`）：

```
# Nmap 7.94 scan initiated Sun Jun  7 10:30:00 2026 as: nmap -sS -sV -p 1-1000 -oG scan_grepable.gnmap scanme.nmap.org
# Ports scanned: TCP(1000;1-1000) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 45.33.32.156 (scanme.nmap.org)	Status: Up
Host: 45.33.32.156 (scanme.nmap.org)	Ports: 22/open/tcp//ssh//OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)///
80/open/tcp//http//Apache httpd 2.4.7 (Ubuntu)///
443/open/tcp//https//Apache httpd 2.4.7 (Ubuntu)///
9929/open/tcp//nping-echo//Nping echo///
31337/open/tcp//tcpwrapped///
25/filtered/tcp//smtp////135/filtered/tcp//msrpc////139/filtered/tcp//netbios-ssn////
445/filtered/tcp//microsoft-ds////
# Nmap done at Sun Jun  7 10:30:45 2026 -- 1 IP address (1 host up) scanned in 45.12 seconds
```

**格式解析**：

每个端口信息段的格式为：`端口号/状态/协议/服务名称/产品/版本/附加信息/`

字段之间以 `/` 分隔，端口之间以空格和制表符分隔。这种格式对 `grep` 极为友好。

**特点**：

- ✅ **极致简洁**：每个主机一行，grep 可以快速匹配
- ✅ **grep 友好**：设计初衷就是为了配合 `grep`、`cut` 等命令行工具使用
- ✅ **体积最小**：在所有格式中文件体积最小
- ❌ **信息丢失**：不包含操作系统检测的详细信息、Traceroute 等复杂数据
- ❌ **版本限制**：新版 Nmap 中该格式可能不完全支持所有新特性
- ❌ **可读性一般**：紧凑格式不太适合人类阅读

### 5.3.4 脚本输出（`-oS`）

**格式说明**：`-oS` 参数以一种"脚本小子"风格输出结果——这是一种调侃式的输出格式，混合了大小写和风格化文本，看起来像是电影中黑客屏幕的效果。

**命令示例**：

```bash
nmap -sS scanme.nmap.org -oS scan_script_kiddie.txt
```

**输出示例**（`scan_script_kiddie.txt`）：

```
# NmAp 7.94 sCaN iNiTiAtEd sUn JuN  7 10:30:00 2026
nMaP sCaN rEpOrT fOr sCaNmE.nMaP.oRg (45.33.32.156)
HoSt Is Up (0.15s LaTeNcY).

pOrT     sTaTe    sErViCe
22/tcp   oPeN     sSh
80/tcp   oPeN     hTtP
443/tcp  oPeN     hTtPs
9929/tcp oPeN     nPiNg-EcHo
```

**特点**：

- ✅ **趣味性**：展示给非技术人员看很有戏剧效果
- ❌ **实用价值低**：大小写混乱，解析困难
- ❌ **不推荐用于实际工作**：仅作为彩蛋存在
- ❌ **与 `-oN` 内容相同**：仅样式不同，信息量一致

### 5.3.5 输出格式对比表

| 特性 | `-oN` （正常） | `-oX` （XML） | `-oG` （Grepable） | `-oS` （脚本） |
|------|:---:|:---:|:---:|:---:|
| 人类可读性 | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| 程序解析性 | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐ |
| 信息完整度 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| 文件体积 | 中 | 大 | **小** | 中 |
| 适合 grep | ⭐⭐ | ⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| 可导入数据库 | ⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐ |
| 适合生成报告 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐ |
| 推荐使用场景 | 人工审查 | 程序处理/持久化 | 快速过滤搜索 | 娱乐演示 |

### 5.3.6 同时输出多种格式

在实际工作中，通常需要同时保存多种格式以满足不同需求。Nmap 允许在单次扫描中同时输出多个格式：

```bash
nmap -sS -sV -O -p- -A 192.168.1.0/24 \
     -oN report_normal.txt \
     -oX report_xml.xml \
     -oG report_grepable.gnmap
```

这条命令同时对 `192.168.1.0/24` 网段进行扫描，并生成三种不同格式的输出文件。建议养成**同时保存 `-oN` 和 `-oX`** 的习惯——前者用于人工查看，后者用于程序化分析。

> **最佳实践**：将多次不同时间的 `-oX` 输出导入数据库，即可实现网络安全的"基线对比"——这是长期安全监控的基础。

---

## 5.4 扫描输出分析技巧

### 5.4.1 如何解读扫描报告

一份完整的 Nmap 扫描报告包含以下几个核心部分，需要逐层解读：

#### 第一层：扫描元数据

```text
# Nmap 7.94 scan initiated Sun Jun  7 10:30:00 2026
```

- **版本号**：Nmap 7.94 — 更新的版本支持更多探测技术和 NSE 脚本
- **扫描时间**：记录何时开始扫描，用于审计和对比

#### 第二层：主机发现状态

```text
Host is up (0.15s latency).
```

- **Host is up**：主机在线，响应了探测包
- **Host seems down**：主机未响应，可能是防火墙过滤或主机离线
- **延迟**：RTT 延迟时间，帮助判断网络链路质量

#### 第三层：端口状态与版本信息

Nmap 将端口分为六种状态，理解每种状态的含义至关重要：

| 状态 | 含义 | 典型原因 |
|------|------|----------|
| **open** | 端口开放，有应用程序在监听 | 正常服务，如 Web、SSH |
| **closed** | 端口关闭，无应用程序监听 | 端口可达但无服务 |
| **filtered** | 防火墙/过滤规则阻止了探测 | 防火墙、ACL、iptables 规则 |
| **unfiltered** | 端口可达，但无法确定开/关 | 通常出现在 ACK 扫描中 |
| **open\|filtered** | 无法区分开放还是被过滤 | 常见于 UDP 扫描、FIN 扫描 |
| **closed\|filtered** | 无法区分关闭还是被过滤 | 较少见，特定扫描类型中出现 |

#### 第四层：服务版本与操作系统信息

```text
80/tcp    open     http       Apache httpd 2.4.7 ((Ubuntu))
```

- **服务名称**：Nmap 根据端口推断（如 80 → http）
- **产品/版本**：版本探测获取的实际服务版本
- **附加信息**：操作系统标识、编译选项等

**实战解读案例**：

```text
22/tcp    open     ssh        OpenSSH 7.4
53/tcp    open     domain     dnsmasq 2.76
443/tcp   open     ssl/http   nginx 1.14.0
3389/tcp  open     ms-wbt-server Microsoft Terminal Services
8080/tcp  open     http-proxy Squid http proxy 3.5.20
```

从上述结果可以判断：
- **22 → SSH**：远程管理服务，需关注暴力破解风险和密钥管理
- **53 → DNS**：DNS 服务，可能存在 DNS 放大攻击风险
- **443 → nginx**：Web 服务器，版本 1.14.0（检查已知漏洞）
- **3389 → RDP**：Windows 远程桌面，高危端口（蓝屏/勒索攻击常见入口）
- **8080 → Squid 代理**：开放代理可能导致 SSRF 攻击和带宽滥用

### 5.4.2 过滤和搜索关键信息的技巧

#### 场景1：快速找出所有开放端口

```bash
# 从默认输出中提取开放端口
grep "open" scan_normal.txt
```

#### 场景2：从 Grepable 输出中提取特定状态

```bash
# 找出所有 filtered 端口
grep "filtered" scan_grepable.gnmap
```

#### 场景3：搜索特定服务

```bash
# 查找 Web 服务器（80、443、8080 端口）
grep -E "(80|443|8080)/open" scan_normal.txt

# 查找 SSH 服务
grep "ssh" scan_normal.txt
```

#### 场景4：查找特定 IP 范围的扫描结果

```bash
grep "192.168.1." scan_grepable.gnmap
```

#### 场景5：排除干扰项

```bash
# 排除 filtered 端口，只关注 open 端口
grep "open" scan_normal.txt | grep -v "filtered"
```

### 5.4.3 使用 grep/awk/sed 处理 Nmap 输出

#### grep 进阶用法

```bash
# 统计各种端口状态的数量
grep -c "/open/" scan_grepable.gnmap
grep -c "/filtered/" scan_grepable.gnmap
grep -c "/closed/" scan_grepable.gnmap
```

```bash
# 多模式匹配：找出开放的高危端口
grep -E "(21|23|25|3389|3306|5432|6379|27017)/open" scan_grepable.gnmap
```

```bash
# 使用 -v 反向匹配：排除特定服务
grep "open" scan_normal.txt | grep -v "http" | grep -v "ssh"
```

#### awk 进阶用法

`awk` 非常适合处理 Nmap 的正常输出和 Grepable 输出。

**提取端口列表（从正常输出）**：

```bash
awk '/^[0-9]/ && /open/ {print $1, $3, $4}' scan_normal.txt
```

预期输出：
```
22/tcp ssh OpenSSH
80/tcp http Apache httpd
443/tcp https Apache httpd
```

**提取 IP 地址和开放端口（从 Grepable 输出）**：

```bash
awk '/Status: Up/ {ip=$2; gsub(/[()]/, "", $0); for(i=4;i<=NF;i++) if($i ~ /\/open\//) print ip, $i}' scan_grepable.gnmap
```

**格式化为 CSV**：

```bash
awk '/^[0-9]/ && /open/ {
  split($1, port, "/");
  split($0, arr, "  ");
  gsub(/^ +/, "", $0);
  print port[1] "," port[2] "," $3 "," $4 "," $5
}' scan_normal.txt
```

#### sed 进阶用法

**清理输出中的额外空格**：

```bash
sed 's/  */ /g' scan_normal.txt
```

**提取指定主机的信息**：

```bash
sed -n '/45.33.32.156/,/^$/p' scan_normal.txt
```

**在 Grepable 格式中提取并标准化端口信息**：

```bash
# 将 /open/tcp//ssh// 这样的格式简化为: port/open/service
sed 's|\([0-9]*\)/open/tcp//\([^/]*\)/.*|\1/open/\2|' scan_grepable.gnmap | grep -oE '[0-9]+/open/[a-ZA-Z-]+'
```

### 5.4.4 将 XML 输出导入数据库

当你需要长期跟踪数百台主机的安全状态变化时，将 Nmap 的 XML 结果导入数据库是最佳实践。

#### 步骤1：创建数据库表结构

首先设计一个简单的 SQLite 数据库来存储扫描结果：

```sql
-- nmap_scans.db
CREATE TABLE scans (
    scan_id INTEGER PRIMARY KEY AUTOINCREMENT,
    scan_timestamp DATETIME,
    target TEXT,
    nmap_version TEXT,
    scan_type TEXT,
    duration_seconds REAL
);

CREATE TABLE hosts (
    host_id INTEGER PRIMARY KEY AUTOINCREMENT,
    scan_id INTEGER,
    ip_address TEXT,
    hostname TEXT,
    host_status TEXT,
    os_name TEXT,
    os_accuracy INTEGER,
    FOREIGN KEY (scan_id) REFERENCES scans(scan_id)
);

CREATE TABLE ports (
    port_id INTEGER PRIMARY KEY AUTOINCREMENT,
    host_id INTEGER,
    protocol TEXT,
    port_number INTEGER,
    port_state TEXT,
    service_name TEXT,
    service_product TEXT,
    service_version TEXT,
    FOREIGN KEY (host_id) REFERENCES hosts(host_id)
);
```

#### 步骤2：使用 Python 解析 XML 并导入数据库

```python
#!/usr/bin/env python3
"""
nmap_xml_import.py - 将 Nmap XML 输出导入 SQLite 数据库
"""

import sqlite3
import xml.etree.ElementTree as ET
import sys
from datetime import datetime


def parse_nmap_xml(xml_file):
    """解析 Nmap XML 文件，返回结构化数据"""
    tree = ET.parse(xml_file)
    root = tree.getroot()

    # 扫描元数据
    scan_info = {
        'nmap_version': root.get('version', 'unknown'),
        'scan_type': root.get('args', ''),
        'start_time': root.get('startstr', ''),
    }

    hosts_data = []

    for host in root.findall('host'):
        # 主机状态
        status_elem = host.find('status')
        if status_elem is None:
            continue

        host_status = status_elem.get('state', 'unknown')

        # IP 地址
        address_elem = host.find('address')
        if address_elem is None:
            continue
        ip_address = address_elem.get('addr', '')
        addr_type = address_elem.get('addrtype', '')

        # 主机名
        hostname = ''
        hostnames = host.find('hostnames')
        if hostnames is not None:
            hname = hostnames.find('hostname')
            if hname is not None:
                hostname = hname.get('name', '')

        # OS 信息
        os_name = ''
        os_accuracy = None
        os_elem = host.find('os')
        if os_elem is not None:
            osmatch = os_elem.find('osmatch')
            if osmatch is not None:
                os_name = osmatch.get('name', '')
                os_accuracy = osmatch.get('accuracy', None)

        host_data = {
            'ip_address': ip_address,
            'addr_type': addr_type,
            'hostname': hostname,
            'host_status': host_status,
            'os_name': os_name,
            'os_accuracy': os_accuracy,
            'ports': []
        }

        # 端口信息
        ports_elem = host.find('ports')
        if ports_elem is not None:
            for port in ports_elem.findall('port'):
                protocol = port.get('protocol', '')
                port_number = int(port.get('portid', 0))

                state_elem = port.find('state')
                port_state = state_elem.get('state', '') if state_elem is not None else ''

                service_elem = port.find('service')
                if service_elem is not None:
                    service_name = service_elem.get('name', '')
                    service_product = service_elem.get('product', '')
                    service_version = service_elem.get('version', '')
                else:
                    service_name = service_product = service_version = ''

                host_data['ports'].append({
                    'protocol': protocol,
                    'port_number': port_number,
                    'port_state': port_state,
                    'service_name': service_name,
                    'service_product': service_product,
                    'service_version': service_version,
                })

        hosts_data.append(host_data)

    return scan_info, hosts_data


def import_to_sqlite(xml_file, db_file):
    """将解析后的数据导入 SQLite 数据库"""
    scan_info, hosts_data = parse_nmap_xml(xml_file)

    conn = sqlite3.connect(db_file)
    cursor = conn.cursor()

    # 创建表
    cursor.executescript('''
        CREATE TABLE IF NOT EXISTS scans (
            scan_id INTEGER PRIMARY KEY AUTOINCREMENT,
            scan_timestamp DATETIME,
            target TEXT,
            nmap_version TEXT,
            scan_type TEXT
        );

        CREATE TABLE IF NOT EXISTS hosts (
            host_id INTEGER PRIMARY KEY AUTOINCREMENT,
            scan_id INTEGER,
            ip_address TEXT,
            hostname TEXT,
            host_status TEXT,
            os_name TEXT,
            os_accuracy INTEGER,
            FOREIGN KEY (scan_id) REFERENCES scans(scan_id)
        );

        CREATE TABLE IF NOT EXISTS ports (
            port_id INTEGER PRIMARY KEY AUTOINCREMENT,
            host_id INTEGER,
            protocol TEXT,
            port_number INTEGER,
            port_state TEXT,
            service_name TEXT,
            service_product TEXT,
            service_version TEXT,
            FOREIGN KEY (host_id) REFERENCES hosts(host_id)
        );
    ''')

    # 插入扫描记录
    cursor.execute('''
        INSERT INTO scans (scan_timestamp, target, nmap_version, scan_type)
        VALUES (?, ?, ?, ?)
    ''', (
        datetime.now().isoformat(),
        ' | '.join(h['ip_address'] for h in hosts_data),
        scan_info['nmap_version'],
        scan_info['scan_type'][:255]
    ))
    scan_id = cursor.lastrowid

    # 插入主机和端口信息
    for host in hosts_data:
        cursor.execute('''
            INSERT INTO hosts (scan_id, ip_address, hostname, host_status, os_name, os_accuracy)
            VALUES (?, ?, ?, ?, ?, ?)
        ''', (
            scan_id,
            host['ip_address'],
            host['hostname'],
            host['host_status'],
            host['os_name'],
            host['os_accuracy']
        ))
        host_id = cursor.lastrowid

        for port in host['ports']:
            cursor.execute('''
                INSERT INTO ports (host_id, protocol, port_number, port_state,
                                   service_name, service_product, service_version)
                VALUES (?, ?, ?, ?, ?, ?, ?)
            ''', (
                host_id,
                port['protocol'],
                port['port_number'],
                port['port_state'],
                port['service_name'],
                port['service_product'],
                port['service_version']
            ))

    conn.commit()
    cursor.execute(
        "SELECT COUNT(*) FROM hosts WHERE scan_id = ?", (scan_id,)
    )
    host_count = cursor.fetchone()[0]
    cursor.execute(
        "SELECT COUNT(*) FROM ports WHERE host_id IN "
        "(SELECT host_id FROM hosts WHERE scan_id = ?)", (scan_id,)
    )
    port_count = cursor.fetchone()[0]

    conn.close()
    print(f"[+] 导入完成：{host_count} 台主机，{port_count} 个端口记录")


def query_open_ports(db_file, state='open'):
    """查询指定状态的端口"""
    conn = sqlite3.connect(db_file)
    cursor = conn.cursor()
    cursor.execute('''
        SELECT h.ip_address, h.hostname, p.port_number, p.protocol,
               p.service_name, p.service_version
        FROM ports p
        JOIN hosts h ON p.host_id = h.host_id
        WHERE p.port_state = ?
        ORDER BY h.ip_address, p.port_number
    ''', (state,))
    results = cursor.fetchall()
    conn.close()
    return results


if __name__ == '__main__':
    if len(sys.argv) < 2:
        print("用法: python nmap_xml_import.py <nmap_xml_file> [database.db]")
        sys.exit(1)

    xml_file = sys.argv[1]
    db_file = sys.argv[2] if len(sys.argv) > 2 else 'nmap_scans.db'

    print(f"[*] 解析 XML: {xml_file}")
    scan_info, hosts_data = parse_nmap_xml(xml_file)

    print(f"[*] Nmap 版本: {scan_info['nmap_version']}")
    print(f"[*] 发现主机数: {len(hosts_data)}")

    for host in hosts_data:
        open_ports = [p for p in host['ports'] if p['port_state'] == 'open']
        print(f"    ├─ {host['ip_address']} ({host['hostname']}) "
              f"— {len(open_ports)} 个开放端口")

    print(f"\n[*] 导入数据库: {db_file}")
    import_to_sqlite(xml_file, db_file)

    # 查询所有开放端口
    print(f"\n[*] 查询所有开放端口:")
    for row in query_open_ports(db_file):
        print(f"    {row[0]:15s} :{row[2]:5d}/{row[3]:3s}  "
              f"{row[4]:10s} {row[5] or ''}")
```

**使用方法**：

```bash
# 将 XML 导入 SQLite 数据库
python3 nmap_xml_import.py scan_result.xml nmap_scans.db

# 预期的输出：
# [*] 解析 XML: scan_result.xml
# [*] Nmap 版本: 7.94
# [*] 发现主机数: 1
#     ├─ 45.33.32.156 (scanme.nmap.org) — 5 个开放端口
#
# [*] 导入数据库: nmap_scans.db
# [+] 导入完成：1 台主机，8 个端口记录
#
# [*] 查询所有开放端口:
#     45.33.32.156   :   22/tcp  ssh        OpenSSH 6.6.1p1 Ubuntu
#     45.33.32.156   :   80/tcp  http       Apache httpd 2.4.7
#     45.33.32.156   :  443/tcp  https      Apache httpd 2.4.7
#     45.33.32.156   : 9929/tcp  nping-echo Nping echo
#     45.33.32.156   :31337/tcp tcpwrapped
```

#### 进阶：使用 SQL 进行分析

将数据导入数据库后，你可以使用 SQL 进行强大的数据分析：

```sql
-- 1. 统计各种端口状态的数量
SELECT port_state, COUNT(*) as count
FROM ports
GROUP BY port_state
ORDER BY count DESC;

-- 预期输出：
-- open       | 5
-- filtered   | 3

-- 2. 查找所有 HTTP 服务
SELECT DISTINCT h.ip_address, p.port_number, p.service_name, p.service_version
FROM ports p
JOIN hosts h ON p.host_id = h.host_id
WHERE p.service_name IN ('http', 'https', 'http-proxy')
  AND p.port_state = 'open';

-- 3. 对比两次扫描的差异（基线对比）
SELECT COALESCE(n.ip_address, o.ip_address) as ip,
       CASE
           WHEN n.ip_address IS NULL THEN '消失'
           WHEN o.ip_address IS NULL THEN '新增'
           ELSE '存在'
       END as host_status
FROM scan_new n
FULL OUTER JOIN scan_old o ON n.ip_address = o.ip_address;
```

---

## 5.5 实战演练

### 练习1：生成多种格式的输出文件

**目标**：掌握 `-oN`、`-oX`、`-oG`、`-oS` 四种输出格式的生成方法，并理解它们的区别。

**场景**：你被要求对 `scanme.nmap.org` 进行快速扫描，生成三种格式的报告供团队不同角色使用。

**命令**：

```bash
# 创建输出目录
mkdir -p ~/nmap_practice/ch05

# 执行扫描并同时生成4种格式的输出
nmap -sS -sV -p 22,80,443,8080,3306,3389 scanme.nmap.org \
     -oN ~/nmap_practice/ch05/normal.txt \
     -oX ~/nmap_practice/ch05/xml_output.xml \
     -oG ~/nmap_practice/ch05/grepable.gnmap \
     -oS ~/nmap_practice/ch05/script_kiddie.txt
```

**预期输出**：

```text
Starting Nmap 7.94 ( https://nmap.org ) at 2026-06-07 10:30 CST
Nmap scan report for scanme.nmap.org (45.33.32.156)
Host is up (0.15s latency).

PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http       Apache httpd 2.4.7 ((Ubuntu))
443/tcp  open  https      Apache httpd 2.4.7 ((Ubuntu))
8080/tcp open  http-proxy

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.34 seconds
```

**验证**：

```bash
# 检查文件是否生成
ls -la ~/nmap_practice/ch05/

# 查看文件的字节数
wc -l ~/nmap_practice/ch05/*
```

**预期结果**：

```text
-rw-r--r-- 1 user user  497 Jun  7 10:30 grepable.gnmap
-rw-r--r-- 1 user user  519 Jun  7 10:30 normal.txt
-rw-r--r-- 1 user user 2000 Jun  7 10:30 script_kiddie.txt  (注：此行为风格化输出)
-rw-r--r-- 1 user user 2800 Jun  7 10:30 xml_output.xml
```

> **观察**：XML 文件最大（约 2.8KB），Grepable 文件最小（约 500 字节）。这正是不同格式在信息密度上的体现。

### 练习2：使用 grep 过滤开放端口

**目标**：学会从扫描结果中快速提取关键信息。

**场景**：你刚刚完成了对一个 `/24` 网段的扫描，需要快速找出所有开放了 SSH 服务的主机。

**准备**：如果你没有大规模扫描数据，可以用一个包含多台主机的模拟文件来练习：

```bash
cat > ~/nmap_practice/ch05/mock_scan.txt << 'EOF'
# Nmap 7.94 scan initiated Mon Jun  7 10:30:00 2026
Nmap scan report for 192.168.1.1
Host is up (0.0010s latency).
PORT     STATE  SERVICE    VERSION
22/tcp   open   ssh        OpenSSH 7.4
80/tcp   open   http       nginx 1.14.0
443/tcp  open   https      nginx 1.14.0
3306/tcp open   mysql      MySQL 5.7.28
8080/tcp closed http-proxy

Nmap scan report for 192.168.1.10
Host is up (0.0020s latency).
PORT     STATE    SERVICE     VERSION
22/tcp   open     ssh         OpenSSH 7.9
80/tcp   filtered http
443/tcp  open     https       Apache httpd 2.4.41
3389/tcp open     ms-wbt-server

Nmap scan report for 192.168.1.20
Host is up (0.0015s latency).
PORT     STATE  SERVICE    VERSION
22/tcp   open   ssh        OpenSSH 6.7
23/tcp   open   telnet     Linux telnetd
80/tcp   open   http       Apache httpd 2.4.7
6379/tcp open   redis      Redis key-value store
27017/tcp open  mongod     MongoDB 3.6.8
EOF
```

**任务 1：找出所有开放 SSH（22）的主机**：

```bash
grep -B 3 "22/tcp.*open" ~/nmap_practice/ch05/mock_scan.txt | grep "report for"
```

预期输出：
```
Nmap scan report for 192.168.1.1
Nmap scan report for 192.168.1.10
Nmap scan report for 192.168.1.20
```

**任务 2：提取所有开放端口及其服务**：

```bash
grep "open" ~/nmap_practice/ch05/mock_scan.txt | awk '{print $1, $3, $4}'
```

预期输出：
```
22/tcp ssh OpenSSH
80/tcp http nginx
443/tcp https nginx
3306/tcp mysql MySQL
22/tcp ssh OpenSSH
443/tcp https Apache
3389/tcp ms-wbt-server
22/tcp ssh OpenSSH
23/tcp telnet Linux
80/tcp http Apache
6379/tcp redis Redis
27017/tcp mongod MongoDB
```

**任务 3：找出所有非标准服务（非 SSH/HTTP/HTTPS 的开放端口）**：

```bash
grep "open" ~/nmap_practice/ch05/mock_scan.txt | grep -v -E "(\bssh\b|\bhttp\b|\bhttps\b)"
```

预期输出：
```
3306/tcp open   mysql      MySQL 5.7.28
3389/tcp open   ms-wbt-server
23/tcp   open   telnet     Linux telnetd
6379/tcp open   redis      Redis key-value store
27017/tcp open  mongod     MongoDB 3.6.8
```

> **安全分析**：上述结果中，telnet（23）、Redis（6379）、MongoDB（27017）都是安全威胁的常见入口点，需要优先处理。

**任务 4（进阶）：使用 Grepable 格式快速统计**：

```bash
# 模拟 Grepable 输出
cat > ~/nmap_practice/ch05/mock_grepable.gnmap << 'EOF'
# Nmap 7.94 scan initiated Mon Jun  7 10:30:00 2026
Host: 192.168.1.1 ()	Status: Up
Host: 192.168.1.1 ()	Ports: 22/open/tcp//ssh//OpenSSH 7.4///80/open/tcp//http//nginx 1.14.0///443/open/tcp//https//nginx 1.14.0///3306/open/tcp//mysql//MySQL 5.7.28///
Host: 192.168.1.10 ()	Status: Up
Host: 192.168.1.10 ()	Ports: 22/open/tcp//ssh//OpenSSH 7.9///443/open/tcp//https//Apache httpd 2.4.41///3389/open/tcp//ms-wbt-server////
Host: 192.168.1.20 ()	Status: Up
Host: 192.168.1.20 ()	Ports: 22/open/tcp//ssh//OpenSSH 6.7///23/open/tcp//telnet//Linux telnetd///80/open/tcp//http//Apache httpd 2.4.7///6379/open/tcp//redis//Redis key-value store///27017/open/tcp//mongod//MongoDB 3.6.8///
# Nmap done at Mon Jun  7 10:30:45 2026 -- 3 IP addresses (3 hosts up) scanned in 45.12 seconds
EOF

# 统计每台主机的开放端口数
awk '/Ports: / {
  ip = $2;
  count = 0;
  for(i=4; i<=NF; i++) {
    split($i, parts, "/");
    if(length(parts[1]) > 0 && parts[2] == "open") count++;
  }
  print ip, "->", count, "open ports"
}' ~/nmap_practice/ch05/mock_grepable.gnmap
```

预期输出：
```
192.168.1.20  -> 5 open ports
192.168.1.10  -> 3 open ports
192.168.1.1   -> 4 open ports
```

### 练习3：解析 XML 输出提取服务信息

**目标**：学会从 XML 输出中结构化地提取服务信息，验证服务版本是否正确。

**场景**：在进行渗透测试时，你需要从扫描结果中提取所有 Web 服务的信息（包括版本号），用于后续的漏洞匹配。

**准备工作**：

```bash
# 使用之前的 XML 输出练习
cd ~/nmap_practice/ch05
```

**方法 A：使用 xmllint（命令行）**

```bash
# 安装 xmllint（如未安装）
sudo apt-get install -y libxml2-utils  # Debian/Ubuntu
# sudo yum install libxml2             # RHEL/CentOS

# 提取所有开放端口和服务信息
xmllint --xpath "//port[state/@state='open']/service/@*[local-name()='name' or local-name()='product' or local-name()='version']" xml_output.xml
```

预期输出：
```
name="ssh" product="OpenSSH" version="6.6.1p1 Ubuntu 2ubuntu2.13"
name="http" product="Apache httpd" version="2.4.7"
name="https" product="Apache httpd" version="2.4.7"
name="http-proxy"
```

**方法 B：使用 Python 脚本（更灵活）**

```python
#!/usr/bin/env python3
"""
extract_services.py - 从 Nmap XML 提取服务信息
"""

import xml.etree.ElementTree as ET
import sys
import json


def extract_services(xml_file, output_format='text'):
    """从 Nmap XML 文件中提取服务信息"""
    tree = ET.parse(xml_file)
    root = tree.getroot()

    results = []

    for host in root.findall('host'):
        # 获取主机 IP
        addr = host.find('address')
        ip = addr.get('addr', 'unknown') if addr is not None else 'unknown'

        # 获取主机名
        hostname_elem = host.find('.//hostname')
        hostname = hostname_elem.get('name', '') if hostname_elem is not None else ''

        # 获取端口信息
        for port in host.findall('.//port'):
            state_elem = port.find('state')
            if state_elem is None or state_elem.get('state') != 'open':
                continue

            port_id = port.get('portid')
            protocol = port.get('protocol')
            service_elem = port.find('service')

            service_info = {
                'ip': ip,
                'hostname': hostname,
                'port': int(port_id),
                'protocol': protocol,
                'state': state_elem.get('state'),
            }

            if service_elem is not None:
                service_info['service'] = service_elem.get('name', '')
                service_info['product'] = service_elem.get('product', '')
                service_info['version'] = service_elem.get('version', '')
                service_info['extrainfo'] = service_elem.get('extrainfo', '')

            results.append(service_info)

    if output_format == 'json':
        print(json.dumps(results, indent=2, ensure_ascii=False))
    else:
        # 文本格式输出
        print(f"{'IP':18s} {'Port':8s} {'Service':12s} {'Version':30s}")
        print("-" * 70)
        for svc in results:
            version = f"{svc['product']} {svc['version']}".strip()
            print(f"{svc['ip']:18s} {svc['port']:5d}/{svc['protocol']:3s}"
                  f" {svc['service']:12s} {version:30s}")


if __name__ == '__main__':
    if len(sys.argv) < 2:
        print("用法: python extract_services.py <nmap_xml_file> [--json]")
        sys.exit(1)

    fmt = 'json' if '--json' in sys.argv else 'text'
    extract_services(sys.argv[1], fmt)
```

**运行脚本**：

```bash
cd ~/nmap_practice/ch05
python3 extract_services.py xml_output.xml
```

预期输出：
```
IP                          Port       Service      Version
45.33.32.156                 22/tcp   ssh          OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13
45.33.32.156                 80/tcp   http         Apache httpd 2.4.7
45.33.32.156                443/tcp   https        Apache httpd 2.4.7
45.33.32.156               8080/tcp   http-proxy
```

> **实战价值**：这个脚本可以直接集成到渗透测试工作流程中——将输出的 JSON 传递给漏洞扫描器或资产管理工具，实现自动化安全的"服务发现→漏洞匹配"链路。

### 练习4：编写脚本自动化分析扫描结果

**目标**：综合运用 Python 和命令行工具，编写一个自动化分析脚本。

**场景**：你的团队管理着 500 台服务器。你需要一个工具，能自动分析 Nmap 扫描结果并生成一个安全风险报告。

**任务**：编写 `nmap_analyzer.py` 脚本，实现以下功能：

1. 读取 `-oX` 或 `-oN` 格式的扫描结果
2. 识别高风险服务（telnet、mysql、redis 等）
3. 统计端口状态分布
4. 生成简洁的 Markdown 报告

```python
#!/usr/bin/env python3
"""
nmap_analyzer.py - Nmap 扫描结果自动化分析工具

用法: python nmap_analyzer.py <nmap_output_file> [--format xml|txt]
"""

import sys
import os
import re
from collections import Counter, defaultdict

# 高风险服务的定义
HIGH_RISK_SERVICES = {
    21: 'FTP (明文传输，匿名访问风险)',
    23: 'Telnet (明文传输，高危)',
    25: 'SMTP (开放中继风险)',
    53: 'DNS (放大攻击风险)',
    135: 'MSRPC (远程过程调用漏洞)',
    139: 'NetBIOS (信息泄露风险)',
    445: 'SMB (永恒之蓝等漏洞)',
    1433: 'MSSQL (数据库暴破风险)',
    1521: 'Oracle (数据库暴破风险)',
    3306: 'MySQL (数据库暴破风险)',
    3389: 'RDP (暴破/蓝屏风险)',
    5432: 'PostgreSQL (数据库暴破风险)',
    5900: 'VNC (未授权访问风险)',
    6379: 'Redis (未授权访问风险)',
    8080: 'HTTP Proxy (SSRF/代理滥用风险)',
    8443: 'HTTPS Alt (证书管理风险)',
    9200: 'Elasticsearch (未授权访问风险)',
    27017: 'MongoDB (未授权访问风险)',
    11211: 'Memcached (DDoS放大攻击风险)',
}


def parse_xml_content(xml_file):
    """解析 XML 格式"""
    try:
        import xml.etree.ElementTree as ET
        tree = ET.parse(xml_file)
        root = tree.getroot()
        hosts_data = []

        for host in root.findall('host'):
            status_elem = host.find('status')
            if status_elem is None:
                continue

            addr = host.find('address')
            ip = addr.get('addr', 'unknown') if addr is not None else 'unknown'

            hostname = ''
            hn_elem = host.find('.//hostname')
            if hn_elem is not None:
                hostname = hn_elem.get('name', '')

            ports = []
            for port in host.findall('.//port'):
                pid = port.get('portid')
                proto = port.get('protocol')
                state_elem = port.find('state')
                state = state_elem.get('state') if state_elem is not None else ''

                svc_elem = port.find('service')
                svc = svc_elem.get('name', '') if svc_elem is not None else ''

                ports.append({
                    'port': int(pid) if pid else 0,
                    'protocol': proto or '',
                    'state': state,
                    'service': svc,
                })

            hosts_data.append({
                'ip': ip,
                'hostname': hostname,
                'status': status_elem.get('state', ''),
                'ports': ports,
            })

        return hosts_data
    except Exception as e:
        print(f"[!] XML 解析失败: {e}", file=sys.stderr)
        return None


def parse_txt_lines(txt_file):
    """解析纯文本格式（-oN）"""
    hosts_data = []
    current_host = None

    with open(txt_file, 'r', encoding='utf-8') as f:
        lines = f.readlines()

    port_pattern = re.compile(
        r'^(\d+)/tcp\s+(open|filtered|closed)\s+(\S+)(?:\s+(.*))?$'
    )

    for line in lines:
        # 检测主机开始
        m = re.search(r'Nmap scan report for (.+)', line)
        if m:
            if current_host:
                hosts_data.append(current_host)
            target = m.group(1)
            ip_match = re.search(r'\(([\d.]+)\)', target)
            hostname = target.split(' (')[0]
            ip = ip_match.group(1) if ip_match else target
            current_host = {
                'ip': ip,
                'hostname': hostname,
                'status': '',
                'ports': [],
            }

        # 检测主机状态
        if current_host and 'Host is up' in line:
            current_host['status'] = 'up'
        elif current_host and 'Host seems down' in line:
            current_host['status'] = 'down'

        # 提取端口信息
        m = port_pattern.match(line.strip())
        if m and current_host:
            port_num = int(m.group(1))
            state = m.group(2)
            service = m.group(3)
            current_host['ports'].append({
                'port': port_num,
                'protocol': 'tcp',
                'state': state,
                'service': service,
            })

    if current_host:
        hosts_data.append(current_host)

    return hosts_data


def analyze(hosts_data):
    """分析扫描结果"""
    if not hosts_data:
        return None

    # 总体统计
    total_hosts = len(hosts_data)
    up_hosts = sum(1 for h in hosts_data if h['status'] == 'up')
    down_hosts = sum(1 for h in hosts_data if h['status'] == 'down')

    # 端口统计
    all_ports = []
    for host in hosts_data:
        for port in host['ports']:
            all_ports.append(port)

    state_counts = Counter(p['state'] for p in all_ports)
    service_counts = Counter(p['service'] for p in all_ports)

    # 高风险服务检测
    open_ports = [p for p in all_ports if p['state'] == 'open']
    high_risk_findings = []

    for port in open_ports:
        if port['port'] in HIGH_RISK_SERVICES:
            risk_desc = HIGH_RISK_SERVICES[port['port']]
            # 找到对应的主机
            for host in hosts_data:
                if any(p['port'] == port['port'] and p['protocol'] == port['protocol']
                       for p in host['ports']):
                    high_risk_findings.append({
                        'ip': host['ip'],
                        'hostname': host['hostname'],
                        'port': port['port'],
                        'protocol': port['protocol'],
                        'service': port['service'],
                        'risk': risk_desc,
                    })
                    break

    return {
        'total_hosts': total_hosts,
        'up_hosts': up_hosts,
        'down_hosts': down_hosts,
        'state_counts': dict(state_counts),
        'service_counts': dict(service_counts.most_common(10)),
        'total_open_ports': len(open_ports),
        'high_risk_findings': high_risk_findings,
    }


def generate_markdown_report(analysis, filename='scan_report.md'):
    """生成 Markdown 格式报告"""
    if not analysis:
        print("[!] 无数据可生成报告")
        return

    lines = [
        "# Nmap 扫描分析报告\n",
        f"> 生成时间：自动分析\n",
        f"> 工具：Nmap 扫描结果分析器\n\n",
        "---\n",
        "## 1. 扫描概览\n",
        f"- **扫描主机总数**：{analysis['total_hosts']}",
        f"- **在线主机**：{analysis['up_hosts']}",
        f"- **离线主机**：{analysis['down_hosts']}",
        f"- **开放端口总数**：{analysis['total_open_ports']}\n",
    ]

    # 端口状态分布
    lines.append("## 2. 端口状态分布\n")
    lines.append("| 状态 | 数量 |")
    lines.append("|------|------|")
    for state, count in sorted(analysis['state_counts'].items()):
        lines.append(f"| {state} | {count} |")
    lines.append("")

    # 前10个最常用的服务
    lines.append("## 3. 最常用服务（Top 10）\n")
    lines.append("| 服务 | 出现次数 |")
    lines.append("|------|----------|")
    for svc, count in list(analysis['service_counts'].items())[:10]:
        lines.append(f"| {svc or '(未识别)'} | {count} |")
    lines.append("")

    # 高风险服务报告
    lines.append("## 4. 高风险服务检测\n")
    if analysis['high_risk_findings']:
        lines.append("> ⚠️ 以下服务存在已知安全风险，建议立即审查：\n")
        lines.append("| 主机 | 端口 | 服务 | 风险描述 |")
        lines.append("|------|------|------|----------|")
        for finding in analysis['high_risk_findings']:
            lines.append(
                f"| {finding['ip']} ({finding['hostname']}) "
                f"| {finding['port']}/{finding['protocol']} "
                f"| {finding['service']} "
                f"| {finding['risk']} |"
            )
    else:
        lines.append("✅ 未检测到高风险服务。\n")

    lines.append("\n---\n")
    lines.append("> *报告由 nmap_analyzer.py 自动生成*\n")

    report = '\n'.join(lines)

    with open(filename, 'w', encoding='utf-8') as f:
        f.write(report)

    print(f"[+] 报告已生成：{filename}")
    return report


def main():
    if len(sys.argv) < 2:
        print("用法: python nmap_analyzer.py <nmap_output_file> [--format xml|txt]")
        print("  --format: 指定输入格式 (默认自动检测)")
        sys.exit(1)

    input_file = sys.argv[1]
    fmt = 'auto'
    if '--format' in sys.argv:
        idx = sys.argv.index('--format')
        if idx + 1 < len(sys.argv):
            fmt = sys.argv[idx + 1]

    if not os.path.exists(input_file):
        print(f"[!] 文件不存在: {input_file}")
        sys.exit(1)

    # 自动检测格式
    if fmt == 'auto':
        if input_file.endswith('.xml'):
            fmt = 'xml'
        else:
            fmt = 'txt'

    print(f"[*] 正在解析: {input_file} ({fmt} 格式)")

    if fmt == 'xml':
        hosts_data = parse_xml_content(input_file)
    else:
        hosts_data = parse_txt_lines(input_file)

    if not hosts_data:
        print("[!] 解析失败或无数据")
        sys.exit(1)

    print(f"[+] 成功解析 {len(hosts_data)} 台主机")

    # 执行分析
    analysis = analyze(hosts_data)

    # 生成报告
    report_file = 'scan_report.md'
    generate_markdown_report(analysis, report_file)

    # 屏幕输出摘要
    print(f"\n{'='*50}")
    print("扫描摘要")
    print(f"{'='*50}")
    print(f"在线主机: {analysis['up_hosts']}/{analysis['total_hosts']}")
    print(f"开放端口总数: {analysis['total_open_ports']}")
    print(f"高风险服务: {len(analysis['high_risk_findings'])} 个")
    if analysis['high_risk_findings']:
        print(f"\n⚠️  高风险服务列表:")
        for f in analysis['high_risk_findings']:
            print(f"   {f['ip']}:{f['port']} → {f['service']} ({f['risk']})")


if __name__ == '__main__':
    main()
```

**使用示例**：

```bash
# 使用之前生成的 mock 数据进行分析
cd ~/nmap_practice/ch05

# 单机扫描
python3 nmap_analyzer.py xml_output.xml

# 扩展扫描（用 mock 数据）
python3 nmap_analyzer.py mock_scan.txt
```

**预期输出**（mock_scan.txt 分析）：

```text
[*] 正在解析: mock_scan.txt (txt 格式)
[+] 成功解析 3 台主机

==================================================
扫描摘要
==================================================
在线主机: 3/3
开放端口总数: 12
高风险服务: 4 个

⚠️  高风险服务列表:
   192.168.1.20:23 → telnet (Telnet (明文传输，高危))
   192.168.1.1:3306 → mysql (MySQL (数据库暴破风险))
   192.168.1.10:3389 → (RDP (暴破/蓝屏风险))    <-- 注意：RDP 服务显示 ms-wbt-server
   192.168.1.20:6379 → redis (Redis (未授权访问风险))
   192.168.1.20:27017 → mongod (MongoDB (未授权访问风险))
```

**生成的报告文件**（`scan_report.md` 预览）：

```markdown
# Nmap 扫描分析报告

> 生成时间：自动分析
> 工具：Nmap 扫描结果分析器

---

## 1. 扫描概览

- **扫描主机总数**：3
- **在线主机**：3
- **离线主机**：0
- **开放端口总数**：12

## 2. 端口状态分布

| 状态 | 数量 |
|------|------|
| open | 12 |
| filtered | 1 |

## 3. 最常用服务（Top 10）

| 服务 | 出现次数 |
|------|----------|
| ssh | 3 |
| http | 2 |
| https | 2 |
| mysql | 1 |
| ms-wbt-server | 1 |
| telnet | 1 |
| redis | 1 |
| mongod | 1 |

## 4. 高风险服务检测

> ⚠️ 以下服务存在已知安全风险，建议立即审查：

| 主机 | 端口 | 服务 | 风险描述 |
|------|------|------|----------|
| 192.168.1.20 () | 23/tcp | telnet | Telnet (明文传输，高危) |
| 192.168.1.1 () | 3306/tcp | mysql | MySQL (数据库暴破风险) |
| 192.168.1.10 () | 3389/tcp | ms-wbt-server | RDP (暴破/蓝屏风险) |
| 192.168.1.20 () | 6379/tcp | redis | Redis (未授权访问风险) |
| 192.168.1.20 () | 27017/tcp | mongod | MongoDB (未授权访问风险) |
```

> 💡 **实战提示**：你可以将这个脚本配置为定期任务（cron），对目标网络进行每日扫描，自动生成安全报告。当高风险服务出现变化时，自动触发告警通知。

---

## 5.6 常见问题（FAQ）

### Q1: 我应该使用哪种输出格式？

**A**: 取决于你的实际需求：

- **日常人工审查** → `-oN`（正常格式）
- **导入数据库/程序处理** → `-oX`（XML 格式）
- **快速 grep 过滤** → `-oG`（Grepable 格式）
- **最佳实践**：同时保存 `-oN` 和 `-oX`，两者互补

```bash
# 推荐的工作流
nmap -sS -sV -oN scan.txt -oX scan.xml target
# 人工查看 scan.txt，程序处理 scan.xml
```

### Q2: Grepable 格式（-oG）是否已经被弃用？

**A**: 不完全。Nmap 官方文档确实提到 `-oG` 格式在未来版本中可能不会获得完整的新特性支持（例如某些 NSE 脚本输出），但它仍然是一个非常高效的工具。对于快速的命令行过滤和 grep 操作，`-oG` 仍然是首选。如果你的工作流高度依赖 `-oG`，建议同时保留 `-oX` 作为备份。

### Q3: 如何使用 xsltproc 将 XML 转换为 HTML 报告？

**A**: Nmap 自带了一个 XSL 样式表，可以一键将 XML 转换为格式美观的 HTML 报告：

```bash
# 安装 xsltproc（如未安装）
sudo apt-get install xsltproc

# 生成 HTML 报告
xsltproc -o nmap_report.html /usr/share/nmap/nmap.xsl scan_result.xml

# 或者使用更现代的样式表
wget https://raw.githubusercontent.com/honze-net/nmap-bootstrap-xsl/stable/nmap-bootstrap.xsl
xsltproc -o nmap_report.html nmap-bootstrap.xsl scan_result.xml
```

生成的 HTML 报告包含交互式表格、搜索过滤功能，适合向管理层展示。

### Q4: 如何处理扫描结果中的误报（false positive）？

**A**: 误报常见于某些防火墙和 IDS 场景。以下策略可以减少误报：

1. **使用多种扫描类型验证**：SYN 扫描 + TCP Connect 扫描对比
2. **增加扫描间隔**：使用 `--scan-delay 1s` 或 `--max-rate 50` 降低扫描速度
3. **版本探测加深**：使用 `-sV --version-intensity 9` 获取更精确的服务识别
4. **端口状态交叉验证**：open 和 open|filtered 的区别

```bash
# 验证可疑端口
nmap -sT -p 80,443 --reason target  # TCP Connect + 显示原因
nmap -sV --version-intensity 9 -p 80 target  # 深度版本探测
```

### Q5: 如何对比两次扫描的结果？

**A**: 推荐使用 `-oX` 格式并导入数据库进行比较。Ndiff 工具（Nmap 套件的一部分）也可以直接对比：

```bash
# 使用 ndiff 对比两次扫描
ndiff scan_before.xml scan_after.xml

# 输出示例
# -192.168.1.10: adding port tcp/8080/open (新增开放端口)
# -192.168.1.20: removing port tcp/23/open (端口关闭)
```

也推荐使用 `nmap-analyzer.py` 脚本配合 SQLite 数据库，通过 SQL 查询做更精细的差异分析：

```sql
-- 两次扫描之间的新开放端口
SELECT p.port_number, h.ip_address
FROM scan_2_ports p
JOIN scan_2_hosts h ON p.host_id = h.host_id
WHERE p.port_state = 'open'
  AND NOT EXISTS (
    SELECT 1 FROM scan_1_ports p2
    JOIN scan_1_hosts h2 ON p2.host_id = h2.host_id
    WHERE p2.port_number = p.port_number
      AND h2.ip_address = h.ip_address
      AND p2.port_state = 'open'
  );
```

### Q6: 大规模扫描（上万台主机）的结果如何处理？

**A**: 处理大规模扫描的核心方法是**结构化存储 + 自动化分析**：

1. **分段扫描**：将大网段拆分成 /24 子网分别扫描
2. **统一输出 XML**：所有子网扫描结果使用 `-oX` 保存
3. **批量导入数据库**：写一个循环脚本将所有 XML 文件导入 SQLite

```bash
#!/bin/bash
# batch_import.sh - 批量导入 Nmap XML 结果
for xml in /var/nmap/scans/*.xml; do
    echo "导入: $xml"
    python3 nmap_xml_import.py "$xml" /var/nmap/nmap_scans.db
done
```

4. **按条件查询**：数据库查询支持复杂过滤（按 IP 段、端口、服务、时间等）
5. **可视化**：使用 Grafana 等工具连接 SQLite 数据库生成图表

### Q7: 输出分析中常见的误解有哪些？

**A**: 以下是一些常见的误解和纠正：

| 误解 | 真相 |
|------|------|
| "filtered = 端口不存在" | filtered 只是表示探测包被过滤了，服务可能仍然存在 |
| "closed = 安全" | 端口关闭是安全的，但主机本身可能仍有其他漏洞 |
| "版本号完全可靠" | 版本检测可能被修改过的服务误导，或者被蜜罐欺骗 |
| "扫描报告 = 攻击面完整" | 只在特定时间点对特定类型的探测有效，动态防火墙和时效性可能导致遗漏 |
| "全部 open 才是好结果" | 合理最小化暴露面才是安全的目标，不必要的开放端口都是攻击面 |

---

## 5.7 总结

### 核心要点回顾

在本章中，我们深入探讨了 Nmap 扫描结果的输出与分析，涵盖了以下关键知识：

| 知识点 | 关键内容 |
|--------|----------|
| **四种输出格式** | `-oN`（人类可读）、`-oX`（XML/程序友好）、`-oG`（Grepable/快速过滤）、`-oS`（趣味演示） |
| **格式选择策略** | 同时保存 `-oN` + `-oX` 是最佳实践 |
| **输出分析技巧** | 从元数据到端口状态到服务版本的逐层解读方法 |
| **命令行过滤** | `grep` 搜索、`awk` 提取、`sed` 清理的强大组合 |
| **XML 数据库导入** | SQLite 持久化存储，SQL 查询分析，实现基线对比 |
| **自动化分析** | Python 脚本从解析到分析到报告生成的完整流水线 |

### 实战价值总结

**输出分析是连接"扫描"和"行动"的关键桥梁。** 一次精心的扫描可能产生数千行输出，但只有通过有效的分析，才能将这些原始数据转化为以下 actionable 成果：

1. **安全风险评估** → 识别高危端口和非授权服务
2. **合规审计报告** → 证明安全控制措施的有效性
3. **攻击面管理** → 发现不必要的暴露面
4. **变更监控** → 追踪配置和服务的异常变化
5. **自动化运维** → 将扫描分析集成到 CI/CD 安全流水线

### 下一步

掌握了 Nmap 的输出分析后，下一步可以探索：

- **第6章：NSE 脚本引擎** — 使用 Nmap 脚本扩展扫描和分析能力
- **Ndiff** — 学习更专业的扫描结果对比工具
- **Grafana + SQLite** — 构建可视化安全仪表板
- **Nmap API** — 将 Nmap 扫描和输出分析嵌入自己的安全工具

> 🔑 **记住**：真正的安全专家不仅知道如何扫描，更知道如何**理解**扫描结果，并从中提取有意义的洞察。输出分析是一项需要不断实践和积累的技能——你现在学到的，正是从新手通往专家之路上的关键一步。
