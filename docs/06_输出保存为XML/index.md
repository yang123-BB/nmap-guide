# 第6章：输出保存为 XML

???+ info "章节信息"
    - **难度**：⭐⭐（初级）
    - **预计时间**：50 分钟
    - **适用对象**：网络安全初学者、系统管理员、渗透测试入门者

---

## 1. 概述

在网络扫描和安全评估工作中，Nmap（Network Mapper）是最常用的开源工具之一。当我们执行扫描任务时，生成的扫描结果包含了大量有价值的信息：开放端口、运行服务、操作系统类型、版本信息等。

### 为什么需要将 Nmap 扫描结果保存为 XML 格式？

想象以下场景：

**场景一：自动化扫描任务**
> 你负责对公司 500 台服务器进行定期安全扫描。每次扫描产生数万行输出，如果只保存在文本文件中，后续分析将非常困难。

**场景二：报告生成**
> 客户要求提供 PDF 格式的扫描报告，包含图表和统计分析。如果扫描结果是纯文本，你需要手动整理；如果是 XML 格式，可以用工具自动生成专业报告。

**场景三：与其他工具集成**
> 你使用的漏洞扫描器、SIEM 系统或资产管理系统需要读取 Nmap 结果。XML 是行业标准的数据交换格式，几乎所有工具都支持解析。

**场景四：数据追溯与审计**
> 三个月前的一次扫描发现了 80 端口开放，现在需要对比查看该主机的安全状态变化。结构化的 XML 数据可以轻松导入数据库进行历史对比。

**核心价值总结**：

| 需求 | XML 格式的优势 |
|------|---------------|
| 自动化处理 | 结构化数据，易于程序解析 |
| 数据交换 | 跨平台、跨语言兼容 |
| 长期存储 | 保留完整扫描元数据 |
| 报告生成 | 可转换为 HTML、PDF、Excel 等格式 |
| 工具集成 | 被 Nmap 生态工具广泛支持 |

---

## 2. 学习目标

完成本章学习后，你将能够：

1. **理解 Nmap 支持的多种输出格式**，并说明 XML 格式的独特优势
2. **熟练使用 Nmap 的 XML 输出参数**（`-oX`、`-oA`、`--no-stylesheet`）
3. **解读 Nmap XML 输出文件的结构**，识别关键标签的含义
4. **使用 xmllint 工具验证 XML 文件的格式正确性**
5. **使用 Python 的 xml.etree.ElementTree 模块解析 Nmap XML 文件**，提取开放端口和服务信息
6. **使用 xsltproc 将 Nmap XML 转换为 HTML 报告**
7. **编写简单的自动化脚本**，批量处理多个 Nmap XML 结果文件

---

## 3. 背景知识

### 3.1 Nmap 输出格式详解

Nmap 支持多种输出格式，可以通过不同的命令行参数指定：

| 参数 | 格式名称 | 说明 | 适用场景 |
|------|---------|------|---------|
| `-oN` | Normal（普通格式） | 与人类直接在终端看到的输出一致 | 人工查看、快速参考 |
| `-oX` | XML 格式 | 结构化数据，包含完整的扫描信息 | 自动化处理、工具集成 |
| `-oG` | Greppable（可 grep 格式） | 简洁的单行输出，便于命令行过滤 | 快速脚本处理（已弃用） |
| `-oA` | All（所有格式） | 同时输出上述三种格式 | 需要多种格式的场景 |
| `-oS` | Script Kiddie（脚本小子格式） | 娱乐性质，随机替换单词 | 不适用实际工作 |
| `-oA` + `-v` | 详细模式 | 包含所有格式 + 详细信息 | 完整记录扫描过程 |

#### 为什么选择 XML 格式？

**XML（eXtensible Markup Language）** 是一种标记语言，具有以下优势：

1. **结构化数据**：XML 使用标签（tag）来组织数据，层次清晰
2. **自描述性**：标签名称具有语义，人和程序都能理解
3. **跨平台兼容**：几乎所有编程语言都有 XML 解析库
4. **保留元数据**：XML 输出包含扫描时间、Nmap 版本、扫描参数等完整信息
5. **可扩展性**：支持自定义命名空间和扩展标签
6. **工具生态丰富**：Nmap 官方提供了 XSL 样式表，可将 XML 转换为 HTML

**对比示例**：

=== "普通格式（-oN）"
    ```txt
    Nmap scan report for 192.168.1.1
    Host is up (0.0023s latency).
    PORT     STATE SERVICE  VERSION
    22/tcp   open  ssh      OpenSSH 7.4
    80/tcp   open  http     Apache httpd 2.4.6
    443/tcp  closed https
    MAC Address: 00:11:22:33:44:55 (Unknown)
    ```

=== "XML 格式（-oX）"
    ```xml
    <host>
      <address addr="192.168.1.1" addrtype="ipv4"/>
      <hostnames></hostnames>
      <ports>
        <port protocol="tcp" portid="22">
          <state state="open" reason="syn-ack" reason_ttl="64"/>
          <service name="ssh" product="OpenSSH" version="7.4" method="probed"/>
        </port>
        <port protocol="tcp" portid="80">
          <state state="open" reason="syn-ack" reason_ttl="64"/>
          <service name="http" product="Apache httpd" version="2.4.6" method="probed"/>
        </port>
        <port protocol="tcp" portid="443">
          <state state="closed" reason="reset" reason_ttl="64"/>
        </port>
      </ports>
      <address addr="00:11:22:33:44:55" addrtype="mac"/>
    </host>
    ```

=== "Grepable 格式（-oG）**
    ```txt
    Host: 192.168.1.1 ()	Status: Up
    Host: 192.168.1.1 ()	Ports: 22/open/tcp//ssh//OpenSSH 7.4/, 80/open/tcp//http//Apache httpd 2.4.6/, 443/closed/tcp//https///	Ignored State: closed (1)
    ```

### 3.2 Nmap XML 输出的结构

一个完整的 Nmap XML 文件遵循以下结构：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE nmaprun>
<?xml-stylesheet href="file:///usr/share/nmap/nmap.xsl" type="text/xsl"?>
<nmaprun>
  <scaninfo ... />
  <verbose level="0"/>
  <debugging level="0"/>
  <host>...</host>
  <host>...</host>
  <runstats>
    <finished time="..." timestr="..."/>
    <hosts up="..." down="..." total="..."/>
  </runstats>
</nmaprun>
```

#### 主要标签说明

**`<nmaprun>`** - 根元素
: 包含整个扫描结果，属性包括：
  - `scanner`：扫描器名称（"nmap"）
  - `args`：完整的命令行参数
  - `start`：扫描开始时间（Unix 时间戳）
  - `startstr`：扫描开始时间（人类可读格式）
  - `version`：Nmap 版本号
  - `xmloutputversion`：XML 输出格式版本

**`<scaninfo>`** - 扫描信息
: 描述扫描的类型和参数，属性包括：
  - `type`：扫描类型（"syn"、"connect"、"udp" 等）
  - `protocol`：协议（"tcp"、"udp"）
  - `numservices`：扫描的端口数量
  - `services`：端口范围

**`<host>`** - 主机信息
: 每个被扫描的主机对应一个 `<host>` 标签，包含：
  - `<status>`：主机状态（up/down）
  - `<address>`：IP 地址和 MAC 地址
  - `<hostnames>`：主机名列表
  - `<ports>`：端口列表
  - `<os>`：操作系统检测结果
  - `<trace>`：路由跟踪信息
  - `<times>`：时间统计

**`<ports>`** - 端口列表
: 包含多个 `<port>` 子标签

**`<port>`** - 单个端口信息
: 属性包括：
  - `protocol`：协议（"tcp"、"udp"）
  - `portid`：端口号
  
  子标签：
  - `<state>`：端口状态（open、closed、filtered 等）
  - `<service>`：服务信息（名称、版本、产品等）
  - `<script>`：NSE 脚本输出

**`<os>`** - 操作系统检测结果
: 包含 `<osclass>` 和 `<osmatch>` 标签，描述可能的操作系统类型

**`<runstats>`** - 扫描统计
: 包含扫描完成时间和主机统计信息

#### 完整的 XML 示例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE nmaprun>
<?xml-stylesheet href="file:///usr/share/nmap/nmap.xsl" type="text/xsl"?>
<nmaprun scanner="nmap" args="nmap -sT -oX scan.xml 192.168.1.0/24" 
         start="1717747200" startstr="Sat Jun  7 14:00:00 2026" 
         version="7.94" xmloutputversion="1.05">
  
  <scaninfo type="connect" protocol="tcp" numservices="1000" services="1-1000"/>
  <verbose level="0"/>
  <debugging level="0"/>
  
  <host>
    <status state="up" reason="arp-response" reason_ttl="0"/>
    <address addr="192.168.1.1" addrtype="ipv4"/>
    <hostnames>
      <hostname name="router.local" type="PTR"/>
    </hostnames>
    <ports>
      <port protocol="tcp" portid="22">
        <state state="open" reason="syn-ack" reason_ttl="64"/>
        <service name="ssh" product="OpenSSH" version="8.2p1" extrainfo="Ubuntu Linux" method="probed" conf="10"/>
      </port>
      <port protocol="tcp" portid="80">
        <state state="open" reason="syn-ack" reason_ttl="64"/>
        <service name="http" product="nginx" version="1.18.0" method="probed" conf="10"/>
      </port>
      <port protocol="tcp" portid="443">
        <state state="open" reason="syn-ack" reason_ttl="64"/>
        <service name="https" product="nginx" version="1.18.0" method="probed" conf="10"/>
      </port>
    </ports>
    <times srtt="1234" rttvar="50" to="100000"/>
  </host>
  
  <host>
    <status state="up" reason="arp-response" reason_ttl="0"/>
    <address addr="192.168.1.100" addrtype="ipv4"/>
    <hostnames>
      <hostname name="workstation.local" type="PTR"/>
    </hostnames>
    <ports>
      <port protocol="tcp" portid="445">
        <state state="open" reason="syn-ack" reason_ttl="128"/>
        <service name="microsoft-ds" method="table" conf="3"/>
      </port>
    </ports>
  </host>
  
  <runstats>
    <finished time="1717747260" timestr="Sat Jun  7 14:01:00 2026" elapsed="60.00" summary="Nmap done at Sat Jun  7 14:01:00 2026; 256 IP addresses (2 hosts up) scanned in 60.00 seconds"/>
    <hosts up="2" down="254" total="256"/>
  </runstats>
</nmaprun>
```

### 3.3 其他输出格式对比

#### 普通格式（-oN）

**优点**：
- 人类可读性强
- 与终端输出完全一致
- 适合快速查看

**缺点**：
- 难以被程序解析
- 格式可能随 Nmap 版本变化
- 不包含完整的元数据

#### Grepable 格式（-oG）

**优点**：
- 每行代表一个主机，便于 grep/awk 处理
- 格式紧凑

**缺点**：
- **已在新版本 Nmap 中弃用**（不推荐使用）
- 信息不完整（缺少某些元数据）
- 不支持某些高级功能（如 NSE 脚本输出）

#### 脚本小子格式（-oS）

娱乐性质，将输出中的单词随机替换（如 "open" 变成 "0p3n"），**不应在实际工作中使用**。

#### 推荐做法

**始终使用 `-oX` 或 `-oA`**，原因：
1. XML 格式是 Nmap 官方推荐的结构化输出格式
2. 保留最完整的信息
3. 向后兼容，旧版本 Nmap 生成的 XML 仍可被新工具解析
4. 支持所有 Nmap 功能（包括 NSE 脚本输出）

---

## 4. Nmap XML 输出参数

### 4.1 `-oX` - 保存为 XML 文件

**基本语法**：
```bash
nmap [扫描选项] -oX <文件名> <目标>
```

**示例 1：扫描单个主机，保存为 XML**
```bash
nmap -sT -p 22,80,443 192.168.1.1 -oX scan_result.xml
```

**示例 2：扫描网段，保存为 XML**
```bash
nmap -sS -p 1-1000 192.168.1.0/24 -oX network_scan.xml
```

**示例 3：输出到标准输出（stdout）**
```bash
nmap -sT localhost -oX - | tee scan.xml
```
注意：`-oX -` 表示输出到标准输出，可以配合 `tee` 命令同时查看和保存。

### 4.2 `-oA` - 保存所有格式

**基本语法**：
```bash
nmap [扫描选项] -oA <文件前缀> <目标>
```

此参数会同时生成三个文件：
- `<前缀>.nmap` - 普通格式
- `<前缀>.xml` - XML 格式
- `<前缀>.gnmap` - Grepable 格式

**示例**：
```bash
nmap -sT 192.168.1.1 -oA scan_result
```

执行后生成：
```
scan_result.nmap   # 普通格式
scan_result.xml    # XML 格式
scan_result.gnmap  # Grepable 格式（已弃用）
```

**使用建议**：
- 初次学习时可以使用 `-oA`，方便对比不同格式
- 生产环境中建议只使用 `-oX`，避免生成不必要的文件

### 4.3 `--no-stylesheet` - 不包含 XSL 样式表

默认情况下，Nmap 会在 XML 文件开头添加以下处理指令：
```xml
<?xml-stylesheet href="file:///usr/share/nmap/nmap.xsl" type="text/xsl"?>
```

这行指令告诉 XML 浏览器使用 XSL 样式表来渲染 XML 文件（在浏览器中打开时会显示美观的 HTML 页面）。

**使用 `--no-stylesheet` 的效果**：
- 移除 XML 中的样式表引用
- 生成的 XML 文件更适合程序解析
- 文件体积略微减小

**示例**：
```bash
nmap -sT 192.168.1.1 -oX scan.xml --no-stylesheet
```

**对比**：

=== "默认（带样式表）"
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE nmaprun>
    <?xml-stylesheet href="file:///usr/share/nmap/nmap.xsl" type="text/xsl"?>
    <nmaprun ...>
    ```

=== "使用 --no-stylesheet"
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <nmaprun ...>
    ```

**使用场景**：
- 在无法访问本地文件系统样式的环境中处理 XML（如容器、远程服务器）
- 减小文件体积（样式表引用路径可能在不同系统上无效）
- 避免某些 XML 解析器对处理指令的警告

### 4.4 结合 `-v` 增加详细信息

`-v`（verbose）参数可以增加输出的详细程度，对 XML 输出同样有效。

**详细级别**：
- `-v` 或 `-v1`：显示详细信息
- `-v2`：显示更详细的信息
- `-d`：启用调试模式（非常详细）

**示例**：
```bash
# 基本详细输出
nmap -sT -v -oX scan_verbose.xml 192.168.1.1

# 更详细的输出
nmap -sT -v2 -oX scan_very_verbose.xml 192.168.1.1

# 调试模式（不推荐常规使用）
nmap -sT -d -oX scan_debug.xml 192.168.1.1
```

**详细模式对 XML 的影响**：
- 在 `<verbose>` 和 `<debugging>` 标签中记录详细级别
- 某些情况下会包含额外的元数据
- 对 XML 结构本身影响较小，主要影响扫描过程的输出

### 4.5 实用组合示例

**场景 1：快速扫描并保存 XML**
```bash
nmap -T4 -F -oX quick_scan.xml 192.168.1.1
```
- `-T4`：快速扫描模板
- `-F`：快速模式（扫描最常见的 100 个端口）

**场景 2：全面扫描（带服务和版本检测）**
```bash
nmap -sS -sV -O -p- -oX full_scan.xml 192.168.1.1
```
- `-sS`：SYN 半开扫描
- `-sV`：版本检测
- `-O`：操作系统检测
- `-p-`：扫描所有 65535 个端口

**场景 3：扫描多个目标并保存**
```bash
nmap -iL target_list.txt -oX multi_scan.xml --no-stylesheet
```
- `-iL`：从文件读取目标列表

**场景 4：将 XML 输出到标准输出（用于管道）**
```bash
nmap -sT localhost -oX - | python3 parse_nmap.py
```

---

## 5. XML 输出解析技巧

### 5.1 使用 xmllint 验证 XML 格式

`xmllint` 是 `libxml2` 工具集的一部分，用于验证和格式化 XML 文件。

#### 安装 xmllint

=== "Ubuntu/Debian"
    ```bash
    sudo apt-get install libxml2-utils
    ```

=== "CentOS/RHEL"
    ```bash
    sudo yum install libxml2
    ```

=== "macOS"
    ```bash
    brew install libxml2
    ```

=== "Windows"
    下载并安装 [libxml2 Windows 版本](http://xmlsoft.org/sources/win32/)

#### 基本用法

**1. 验证 XML 格式是否正确**
```bash
xmllint --noout scan.xml
```
- 如果 XML 格式正确，无输出
- 如果格式错误，显示错误信息（如：行号、列号、错误描述）

**示例**：
```bash
$ xmllint --noout scan.xml
scan.xml:15: parser error : Opening and ending tag mismatch: port line 10 and ports
  </host>
         ^
```

**2. 格式化输出（美化 XML）**
```bash
xmllint --format scan.xml > formatted_scan.xml
```

**3. 验证 XML 结构（使用 DTD）**
```bash
xmllint --dtdvalid nmap.dtd scan.xml
```
注意：Nmap 安装包中通常包含 `nmap.dtd` 文件。

**4. 提取特定元素（使用 XPath）**
```bash
xmllint --xpath "//host/ports/port[@portid='80']" scan.xml
```

#### 实战示例

**验证 Nmap XML 文件**
```bash
# 生成扫描结果
nmap -sT localhost -oX localhost_scan.xml

# 验证格式
xmllint --noout localhost_scan.xml

# 如果无输出，说明格式正确
echo $?  # 输出 0 表示成功
```

**使用场景**：
- 在自动化脚本中验证 Nmap 输出是否有效
- 在解析 XML 之前确保文件格式正确
- 调试 XML 生成脚本

### 5.2 使用 xsltproc 转换为 HTML

`xsltproc` 是一个 XSLT 处理器，可以将 XML 转换为 HTML、PDF 或其他格式。

Nmap 自带了 XSL 样式表（`nmap.xsl`），可以将 XML 扫描结果转换为美观的 HTML 报告。

#### 安装 xsltproc

=== "Ubuntu/Debian"
    ```bash
    sudo apt-get install xsltproc
    ```

=== "CentOS/RHEL"
    ```bash
    sudo yum install libxslt
    ```

=== "macOS"
    ```bash
    brew install libxslt
    ```

#### 基本用法

**1. 使用默认的 nmap.xsl 样式表**
```bash
xsltproc scan.xml -o scan_report.html
```

**2. 指定自定义样式表**
```bash
xsltproc custom_style.xsl scan.xml -o custom_report.html
```

**3. 传递参数给 XSLT**
```bash
xsltproc --param showSummary 1 nmap.xsl scan.xml -o report.html
```

#### Nmap 自带的 XSL 样式表

Nmap 安装的 `nmap.xsl` 通常位于：
- Linux：`/usr/share/nmap/nmap.xsl`
- macOS：`/usr/local/share/nmap/nmap.xsl`

**在浏览器中查看 XML**
如果 XML 文件包含样式表引用（默认情况），直接在浏览器中打开即可看到格式化的 HTML 页面：
```bash
# 生成带样式表的 XML
nmap -sT localhost -oX scan.xml

# 在浏览器中打开（会自动应用 nmap.xsl）
firefox scan.xml
```

**转换为独立的 HTML**
如果要在没有 nmap.xsl 的环境中查看报告，需要转换：
```bash
xsltproc scan.xml -o scan_report.html
```

#### 自定义 XSL 样式表

你可以编写自定义 XSL 来生成符合需求的报告。

**示例 XSL（简化版）**：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
  <xsl:output method="html" encoding="UTF-8"/>
  
  <xsl:template match="/">
    <html>
      <head>
        <title>Nmap 扫描报告</title>
      </head>
      <body>
        <h1>Nmap 扫描报告</h1>
        <xsl:apply-templates select="//host"/>
      </body>
    </html>
  </xsl:template>
  
  <xsl:template match="host">
    <h2>主机: <xsl:value-of select="address[@addrtype='ipv4']/@addr"/></h2>
    <ul>
      <xsl:for-each select="ports/port[state/@state='open']">
        <li>端口 <xsl:value-of select="@portid"/>: <xsl:value-of select="service/@name"/></li>
      </xsl:for-each>
    </ul>
  </xsl:template>
</xsl:stylesheet>
```

使用自定义样式表转换：
```bash
xsltproc custom.xsl scan.xml -o custom_report.html
```

### 5.3 使用 Python 解析 Nmap XML

Python 内置了 `xml.etree.ElementTree` 模块，可以方便地解析 XML 文件。

#### 基本解析流程

```python
import xml.etree.ElementTree as ET

# 1. 解析 XML 文件
tree = ET.parse('scan.xml')
root = tree.getroot()

# 2. 遍历所有主机
for host in root.findall('host'):
    # 获取 IP 地址
    ip = host.find('address[@addrtype="ipv4"]').get('addr')
    print(f"主机: {ip}")
    
    # 获取主机状态
    status = host.find('status').get('state')
    print(f"状态: {status}")
    
    # 遍历端口
    ports = host.find('ports')
    if ports is not None:
        for port in ports.findall('port'):
            portid = port.get('portid')
            protocol = port.get('protocol')
            state = port.find('state').get('state')
            
            # 获取服务信息
            service = port.find('service')
            if service is not None:
                service_name = service.get('name')
                print(f"  端口 {portid}/{protocol}: {state} ({service_name})")
```

#### 完整示例：提取开放端口和服务

以下脚本读取 Nmap XML 文件，输出每个主机的开放端口和服务信息：

```python
#!/usr/bin/env python3
"""
Nmap XML 解析脚本
功能：从 Nmap XML 输出中提取开放端口和服务信息
"""

import xml.etree.ElementTree as ET
import sys

def parse_nmap_xml(xml_file):
    """
    解析 Nmap XML 文件
    
    Args:
        xml_file: XML 文件路径
    """
    try:
        tree = ET.parse(xml_file)
        root = tree.getroot()
        
        # 打印扫描信息
        print("=" * 60)
        print(f"扫描参数: {root.get('args')}")
        print(f"Nmap 版本: {root.get('version')}")
        print(f"扫描开始时间: {root.get('startstr')}")
        print("=" * 60)
        
        # 遍历所有主机
        for host in root.findall('host'):
            # 获取主机状态
            status = host.find('status').get('state')
            
            if status != 'up':
                continue
            
            # 获取 IP 地址
            addr_elem = host.find('address[@addrtype="ipv4"]')
            if addr_elem is not None:
                ip = addr_elem.get('addr')
            else:
                ip = "未知"
            
            # 获取主机名
            hostname_elem = host.find('hostnames/hostname')
            if hostname_elem is not None:
                hostname = hostname_elem.get('name')
            else:
                hostname = "无"
            
            print(f"\n[主机] {ip} ({hostname})")
            print("-" * 60)
            
            # 获取端口信息
            ports_elem = host.find('ports')
            if ports_elem is None:
                print("  未扫描端口")
                continue
            
            # 统计端口状态
            open_ports = []
            closed_ports = []
            filtered_ports = []
            
            for port in ports_elem.findall('port'):
                portid = port.get('portid')
                protocol = port.get('protocol')
                state = port.find('state').get('state')
                
                # 获取服务信息
                service = port.find('service')
                if service is not None:
                    service_name = service.get('name', '未知')
                    product = service.get('product', '')
                    version = service.get('version', '')
                    service_info = f"{service_name} {product} {version}".strip()
                else:
                    service_info = "未知服务"
                
                port_info = {
                    'portid': portid,
                    'protocol': protocol,
                    'state': state,
                    'service': service_info
                }
                
                if state == 'open':
                    open_ports.append(port_info)
                elif state == 'closed':
                    closed_ports.append(port_info)
                elif state == 'filtered':
                    filtered_ports.append(port_info)
            
            # 输出开放端口
            if open_ports:
                print(f"  开放端口 ({len(open_ports)} 个):")
                for p in open_ports:
                    print(f"    {p['portid']}/{p['protocol']}: {p['service']}")
            
            # 输出关闭端口统计
            if closed_ports:
                print(f"  关闭端口: {len(closed_ports)} 个")
            
            # 输出过滤端口统计
            if filtered_ports:
                print(f"  被过滤端口: {len(filtered_ports)} 个")
            
            # 获取操作系统检测结果
            os_elem = host.find('os')
            if os_elem is not None:
                osmatch = os_elem.find('osmatch')
                if osmatch is not None:
                    print(f"  操作系统: {osmatch.get('name')} (准确度: {osmatch.get('accuracy')}%)")
        
        # 打印扫描统计
        runstats = root.find('runstats')
        if runstats is not None:
            finished = runstats.find('finished')
            hosts = runstats.find('hosts')
            
            print("\n" + "=" * 60)
            print("扫描统计:")
            if finished is not None:
                print(f"  扫描耗时: {finished.get('elapsed')} 秒")
                print(f"  完成时间: {finished.get('timestr')}")
            if hosts is not None:
                print(f"  主机总数: {hosts.get('total')}")
                print(f"  在线主机: {hosts.get('up')}")
                print(f"  离线主机: {hosts.get('down')}")
            print("=" * 60)
    
    except ET.ParseError as e:
        print(f"XML 解析错误: {e}", file=sys.stderr)
        sys.exit(1)
    except FileNotFoundError:
        print(f"文件未找到: {xml_file}", file=sys.stderr)
        sys.exit(1)

if __name__ == '__main__':
    if len(sys.argv) != 2:
        print("用法: python3 parse_nmap.py <xml_file>")
        print("示例: python3 parse_nmap.py scan.xml")
        sys.exit(1)
    
    xml_file = sys.argv[1]
    parse_nmap_xml(xml_file)
```

**使用方法**：
```bash
# 1. 先生成 Nmap XML 文件
nmap -sT -sV localhost -oX localhost_scan.xml

# 2. 运行解析脚本
python3 parse_nmap.py localhost_scan.xml
```

**预期输出**：
```
============================================================
扫描参数: nmap -sT -sV -oX localhost_scan.xml localhost
Nmap 版本: 7.94
扫描开始时间: Sat Jun  7 14:00:00 2026
============================================================

[主机] 127.0.0.1 (localhost)
------------------------------------------------------------
  开放端口 (3 个):
    22/tcp: ssh OpenSSH 8.2p1 Ubuntu Linux
    80/tcp: http Apache httpd 2.4.6
    443/tcp: https Apache httpd 2.4.6
  关闭端口: 997 个

============================================================
扫描统计:
  扫描耗时: 12.34 秒
  完成时间: Sat Jun  7 14:00:12 2026
  主机总数: 1
  在线主机: 1
  离线主机: 0
============================================================
```

#### 高级技巧

**1. 提取特定端口的信息**
```python
# 查找所有开放了 80 端口的主机
for host in root.findall('host'):
    ip = host.find('address[@addrtype="ipv4"]').get('addr')
    port_80 = host.find(".//port[@portid='80']/state[@state='open']")
    if port_80 is not None:
        print(f"{ip}:80 端口开放")
```

**2. 导出为 CSV 格式**
```python
import csv

with open('scan_results.csv', 'w', newline='', encoding='utf-8') as f:
    writer = csv.writer(f)
    writer.writerow(['IP', '端口', '协议', '状态', '服务'])
    
    for host in root.findall('host'):
        ip = host.find('address[@addrtype="ipv4"]').get('addr')
        for port in host.findall('.//port'):
            portid = port.get('portid')
            protocol = port.get('protocol')
            state = port.find('state').get('state')
            service = port.find('service').get('name') if port.find('service') is not None else ''
            writer.writerow([ip, portid, protocol, state, service])
```

**3. 使用 XPath 表达式**
```python
# 需要安装 lxml 库: pip install lxml
from lxml import etree

tree = etree.parse('scan.xml')

# 查找所有开放端口
open_ports = tree.xpath("//port[state/@state='open']")
for port in open_ports:
    print(port.get('portid'))
```

### 5.4 使用 Perl 解析 Nmap XML

Perl 社区提供了 `Nmap::Parser` 模块，专门用于解析 Nmap XML 文件。

#### 安装 Nmap::Parser

```bash
# 使用 CPAN 安装
cpan Nmap::Parser

# 或使用 cpanminus
cpanm Nmap::Parser
```

#### 基本用法

```perl
#!/usr/bin/perl
use strict;
use warnings;
use Nmap::Parser;

my $parser = new Nmap::Parser;
$parser->parsefile('scan.xml');

# 遍历所有主机
for my $host ($parser->all_hosts()) {
    my $ip = $host->addr();
    my $hostname = $host->hostname() || '无';
    my $status = $host->status();
    
    print "[主机] $ip ($hostname)\n";
    print "  状态: $status\n";
    
    # 遍历开放端口
    for my $port ($host->tcp_ports()) {
        my $portid = $port->portid();
        my $state = $port->state();
        my $service = $port->service() || '未知';
        
        if ($state eq 'open') {
            print "  端口 $portid/tcp: $service\n";
        }
    }
}
```

#### 完整示例

```perl
#!/usr/bin/perl
use strict;
use warnings;
use Nmap::Parser;

# 检查命令行参数
if (@ARGV != 1) {
    die "用法: perl parse_nmap.pl <xml_file>\n";
}

my $xml_file = $ARGV[0];
my $parser = new Nmap::Parser;

eval {
    $parser->parsefile($xml_file);
};

if ($@) {
    die "解析 XML 文件失败: $@\n";
}

# 打印扫描信息
print "=" x 60 . "\n";
print "扫描参数: " . $parser->nmap_run_args() . "\n";
print "Nmap 版本: " . $parser->nmap_version() . "\n";
print "扫描开始时间: " . $parser->nmap_start_time() . "\n";
print "=" x 60 . "\n\n";

# 遍历主机
for my $host ($parser->all_hosts()) {
    next unless $host->status() eq 'up';
    
    my $ip = $host->addr();
    my $hostname = $host->hostname() || '无';
    
    print "[主机] $ip ($hostname)\n";
    print "-" x 60 . "\n";
    
    # TCP 端口
    my @tcp_open = grep { $_->state() eq 'open' } $host->tcp_ports();
    if (@tcp_open) {
        print "  开放 TCP 端口 (" . scalar(@tcp_open) . " 个):\n";
        for my $port (@tcp_open) {
            my $service = $port->service() || '未知';
            my $product = $port->service_product() || '';
            my $version = $port->service_version() || '';
            print "    " . $port->portid() . "/tcp: $service $product $version\n";
        }
    }
    
    # UDP 端口
    my @udp_open = grep { $_->state() eq 'open' } $host->udp_ports();
    if (@udp_open) {
        print "  开放 UDP 端口 (" . scalar(@udp_open) . " 个):\n";
        for my $port (@udp_open) {
            my $service = $port->service() || '未知';
            print "    " . $port->portid() . "/udp: $service\n";
        }
    }
    
    # 操作系统检测
    if ($host->os_sig()) {
        print "  操作系统: " . $host->os_sig()->name() . "\n";
    }
    
    print "\n";
}

# 打印统计信息
print "=" x 60 . "\n";
print "扫描统计:\n";
print "  扫描耗时: " . $parser->nmap_finish_time() - $parser->nmap_start_time() . " 秒\n";
print "  主机总数: " . scalar($parser->all_hosts()) . "\n";
print "=" x 60 . "\n";
```

**使用方法**：
```bash
perl parse_nmap.pl scan.xml
```

---

## 6. 实战演练

### 练习 1：生成 XML 输出文件

**目标**：使用 Nmap 扫描本地主机，并将结果保存为 XML 格式。

**步骤**：

1. 打开终端，执行以下命令：
   ```bash
   nmap -sT -p 22,80,443,3306,6379 localhost -oX exercise1.xml
   ```

2. 查看生成的 XML 文件：
   ```bash
   cat exercise1.xml
   ```

3. 在浏览器中打开 XML 文件（如果系统安装了 Nmap）：
   ```bash
   firefox exercise1.xml
   ```

**预期输出**：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE nmaprun>
<?xml-stylesheet href="file:///usr/share/nmap/nmap.xsl" type="text/xsl"?>
<nmaprun scanner="nmap" args="nmap -sT -p 22,80,443,3306,6379 localhost -oX exercise1.xml" start="1717747200" startstr="Sat Jun  7 14:00:00 2026" version="7.94" xmloutputversion="1.05">
  <!-- ... 扫描结果 ... -->
</nmaprun>
```

**思考题**：
- 尝试不使用 `-oX` 参数，直接将输出保存到文件（`nmap ... > output.txt`），对比两种方式的区别。
- 使用 `-oA` 参数，观察生成了哪些文件。

---

### 练习 2：使用 xmllint 验证 XML 格式

**目标**：学会使用 xmllint 工具验证 Nmap XML 文件的正确性。

**步骤**：

1. 生成一个有效的 XML 文件：
   ```bash
   nmap -sT localhost -oX valid.xml
   ```

2. 验证 XML 格式：
   ```bash
   xmllint --noout valid.xml
   echo $?  # 应该输出 0（表示成功）
   ```

3. 创建一个故意错误的 XML 文件（用于测试）：
   ```bash
   echo '<nmaprun><host></nmaprun>' > invalid.xml
   ```

4. 验证错误文件：
   ```bash
   xmllint --noout invalid.xml
   echo $?  # 应该输出非 0（表示失败）
   ```

5. 使用 xmllint 格式化 XML：
   ```bash
   xmllint --format valid.xml > formatted.xml
   cat formatted.xml
   ```

**预期输出（验证错误文件）**：
```
invalid.xml:1: parser error : Opening and ending tag mismatch: host line 1 and nmaprun
<nmaprun><host></nmaprun>
                        ^
invalid.xml:1: parser error : Premature end of data in tag nmaprun line 1
<nmaprun><host></nmaprun>
                        ^
```

**思考题**：
- 如果 XML 文件很大（几 MB），如何快速验证格式而不打印内容？
- 如何使用 xmllint 只提取特定主机的信息？

---

### 练习 3：使用 Python 解析 XML 提取开放端口

**目标**：编写 Python 脚本，从 Nmap XML 文件中提取所有开放端口的信息。

**步骤**：

1. 生成包含多个主机的扫描结果：
   ```bash
   nmap -sT scanme.nmap.org -oX scanme.xml
   ```

2. 创建 Python 脚本 `extract_ports.py`：
   ```python
   #!/usr/bin/env python3
   import xml.etree.ElementTree as ET
   import sys
   
   def extract_open_ports(xml_file):
       tree = ET.parse(xml_file)
       root = tree.getroot()
       
       print("开放端口列表:")
       print("=" * 60)
       
       for host in root.findall('host'):
           # 获取 IP 地址
           addr = host.find('address[@addrtype="ipv4"]')
           if addr is None:
               continue
           ip = addr.get('addr')
           
           # 获取开放端口
           open_ports = []
           ports = host.find('ports')
           if ports is not None:
               for port in ports.findall('port'):
                   state = port.find('state').get('state')
                   if state == 'open':
                       portid = port.get('portid')
                       protocol = port.get('protocol')
                       
                       service = port.find('service')
                       if service is not None:
                           service_name = service.get('name', '')
                       else:
                           service_name = '未知'
                       
                       open_ports.append(f"{portid}/{protocol} ({service_name})")
           
           if open_ports:
               print(f"\n主机: {ip}")
               for port_info in open_ports:
                   print(f"  - {port_info}")
       
       print("\n" + "=" * 60)
   
   if __name__ == '__main__':
       if len(sys.argv) != 2:
           print("用法: python3 extract_ports.py <xml_file>")
           sys.exit(1)
       
       extract_open_ports(sys.argv[1])
   ```

3. 运行脚本：
   ```bash
   python3 extract_ports.py scanme.xml
   ```

**预期输出**：
```
开放端口列表:
============================================================

主机: 45.33.32.156
  - 22/tcp (ssh)
  - 80/tcp (http)
  - 9929/tcp (nmap)
  - 31337/tcp (Elite)

============================================================
```

**思考题**：
- 如何修改脚本，使其同时支持 IPv6 地址？
- 如何将结果保存为 JSON 格式？

---

### 练习 4：转换 XML 为 HTML 报告

**目标**：使用 xsltproc 将 Nmap XML 文件转换为 HTML 报告。

**步骤**：

1. 生成扫描结果：
   ```bash
   nmap -sT -sV localhost -oX localhost_scan.xml
   ```

2. 使用 xsltproc 转换：
   ```bash
   xsltproc localhost_scan.xml -o localhost_report.html
   ```

3. 在浏览器中查看 HTML 报告：
   ```bash
   firefox localhost_report.html
   ```

4. （可选）自定义样式表：
   - 创建 `custom.xsl` 文件（参考 5.2 节的示例）
   - 使用自定义样式表转换：
     ```bash
     xsltproc custom.xsl localhost_scan.xml -o custom_report.html
     ```

**预期输出**：
- 生成一个 `localhost_report.html` 文件
- 在浏览器中打开后，显示格式化的扫描报告，包括：
  - 扫描参数和版本信息
  - 每个主机的 IP 地址和主机名
  - 开放端口列表（带服务名称和版本）
  - 扫描统计信息

**思考题**：
- 如何修改 XSL 样式表，使报告包含图表（如端口状态饼图）？
- 如果要在团队中共享报告，如何确保接收方也能正确查看（考虑 nmap.xsl 的路径问题）？

---

## 7. 常见问题（FAQ）

### Q1: 为什么我的 XML 文件在浏览器中打开时显示乱码？

**A**: 这通常是因为 XML 文件的编码声明与实际编码不匹配。Nmap 默认使用 UTF-8 编码生成 XML 文件。

**解决方法**：
1. 确保 XML 文件开头包含正确的编码声明：
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   ```

2. 如果手动编辑 XML 文件，保存时选择 UTF-8 编码（不带 BOM）。

3. 在浏览器中手动选择编码：
   - Firefox：`查看` → `文字编码` → `Unicode (UTF-8)`
   - Chrome：`更多工具` → `编码` → `UTF-8`

---

### Q2: 使用 `-oX` 参数时，如何将 XML 输出到标准输出（stdout）？

**A**: 使用 `-` 作为文件名：

```bash
nmap -sT localhost -oX -
```

这在以下场景中很有用：
- 将 XML 通过管道传递给其他命令
- 在脚本中捕获 XML 输出进行处理
- 同时查看输出和保存文件（`nmap ... -oX - | tee output.xml`）

---

### Q3: 为什么建议使用 `--no-stylesheet` 参数？

**A**: 有以下几个原因：

1. **跨平台兼容性**：XSL 样式表的路径（`file:///usr/share/nmap/nmap.xsl`）在不同系统上可能不同，导致在某些环境中无法正确渲染。

2. **程序解析**：某些 XML 解析器会对处理指令（PI）发出警告，移除样式表引用可以避免这些警告。

3. **文件大小**：虽然影响很小，但移除样式表引用可以略微减小文件体积。

4. **自动化处理**：在自动化脚本中，通常不需要在浏览器中查看 XML，因此不需要样式表。

**建议**：
- 如果要在浏览器中查看报告，不要使用 `--no-stylesheet`
- 如果用于程序解析或自动化处理，使用 `--no-stylesheet`

---

### Q4: 如何解析包含多个扫描任务的 XML 文件？

**A**: Nmap 每次扫描只能生成一个 XML 文件。如果你需要合并多个扫描结果，有以下几种方法：

**方法 1：使用 Nmap 的 `-oX` 和追加模式**
```bash
# 第一次扫描
nmap -sT 192.168.1.1 -oX - > combined.xml

# 后续扫描（手动合并，需要编辑 XML）
```

**方法 2：使用 Python 脚本合并**
```python
import xml.etree.ElementTree as ET

# 解析多个 XML 文件
trees = [ET.parse(f'scan{i}.xml') for i in range(1, 4)]

# 创建新的根元素
combined_root = ET.Element('nmaprun')

# 合并所有主机
for tree in trees:
    root = tree.getroot()
    for host in root.findall('host'):
        combined_root.append(host)

# 保存合并后的文件
combined_tree = ET.ElementTree(combined_root)
combined_tree.write('combined.xml', encoding='UTF-8', xml_declaration=True)
```

**方法 3：使用专用工具**
- Nmap 官方提供了 `nmap-parse-output` 等工具
- 某些 Nmap 前端工具（如 Zenmap）支持合并扫描结果

---

### Q5: XML 输出中的 `reason` 属性是什么意思？

**A**: `reason` 属性解释了为什么 Nmap 认为端口处于特定状态。

**常见 reason 值**：

| reason | 说明 |
|--------|------|
| `syn-ack` | 收到 SYN-ACK 响应，端口开放（SYN 扫描） |
| `ack-rst` | 收到 RST 响应，端口未过滤（ACK 扫描） |
| `reset` | 收到 RST 响应，端口关闭 |
| `port-unreach` | 收到 ICMP 端口不可达错误，端口关闭（UDP 扫描） |
| `conn-refused` | 连接被拒绝，端口关闭（TCP Connect 扫描） |
| `timedout` | 超时，端口可能被过滤 |
| `admin-prohibited` | 被管理员禁止（防火墙规则） |

**示例**：
```xml
<port protocol="tcp" portid="22">
  <state state="open" reason="syn-ack" reason_ttl="64"/>
</port>
```
这表示端口 22 开放，因为收到了 SYN-ACK 响应。

---

### Q6: 如何将 Nmap XML 导入数据库？

**A**: 可以使用 Python 脚本解析 XML 并插入数据库。

**示例（使用 SQLite）**：
```python
import xml.etree.ElementTree as ET
import sqlite3

# 连接到数据库
conn = sqlite3.connect('nmap_scans.db')
c = conn.cursor()

# 创建表
c.execute('''
CREATE TABLE IF NOT EXISTS scans (
    id INTEGER PRIMARY KEY,
    scan_time TIMESTAMP,
    target TEXT
)
''')

c.execute('''
CREATE TABLE IF NOT EXISTS hosts (
    id INTEGER PRIMARY KEY,
    scan_id INTEGER,
    ip TEXT,
    status TEXT,
    FOREIGN KEY (scan_id) REFERENCES scans(id)
)
''')

c.execute('''
CREATE TABLE IF NOT EXISTS ports (
    id INTEGER PRIMARY KEY,
    host_id INTEGER,
    portid INTEGER,
    protocol TEXT,
    state TEXT,
    service TEXT,
    FOREIGN KEY (host_id) REFERENCES hosts(id)
)
''')

# 解析 XML 并插入数据
tree = ET.parse('scan.xml')
root = tree.getroot()

# 插入扫描记录
c.execute("INSERT INTO scans (scan_time, target) VALUES (?, ?)",
          (root.get('startstr'), root.get('args')))

scan_id = c.lastrowid

# 插入主机和端口
for host in root.findall('host'):
    ip = host.find('address[@addrtype="ipv4"]').get('addr')
    status = host.find('status').get('state')
    
    c.execute("INSERT INTO hosts (scan_id, ip, status) VALUES (?, ?, ?)",
             (scan_id, ip, status))
    
    host_id = c.lastrowid
    
    for port in host.findall('.//port'):
        portid = port.get('portid')
        protocol = port.get('protocol')
        state = port.find('state').get('state')
        service = port.find('service').get('name') if port.find('service') is not None else ''
        
        c.execute("INSERT INTO ports (host_id, portid, protocol, state, service) VALUES (?, ?, ?, ?, ?)",
                 (host_id, portid, protocol, state, service))

# 提交更改
conn.commit()
conn.close()
```

---

### Q7: Nmap XML 输出中的时间戳是什么格式？

**A**: Nmap 使用两种格式表示时间：

1. **Unix 时间戳**（秒级）
   - 属性名：`time`（如 `<finished time="1717747200" .../>`）
   - 表示从 1970-01-01 00:00:00 UTC 到现在的秒数

2. **人类可读格式**
   - 属性名：`timestr`（如 `timestr="Sat Jun  7 14:00:00 2026"`）
   - 格式：`%a %b %d %H:%M:%S %Y`

**在 Python 中转换**：
```python
import datetime

# Unix 时间戳转 datetime
timestamp = 1717747200
dt = datetime.datetime.fromtimestamp(timestamp)
print(dt)  # 2026-06-07 14:00:00

# datetime 转 Unix 时间戳
dt = datetime.datetime(2026, 6, 7, 14, 0, 0)
timestamp = int(dt.timestamp())
print(timestamp)  # 1717747200
```

---

## 8. 总结

### 本章要点回顾

在本章中，我们学习了：

1. **为什么需要 XML 格式**
   - 结构化数据，便于程序解析
   - 跨平台、跨语言兼容
   - 保留完整的扫描元数据
   - 支持自动化处理和报告生成

2. **Nmap 输出格式对比**
   - `-oN`：普通格式，适合人工查看
   - `-oX`：XML 格式，推荐用于自动化
   - `-oG`：Grepable 格式，已弃用
   - `-oA`：同时输出所有格式

3. **Nmap XML 结构**
   - 根元素：`<nmaprun>`
   - 主机信息：`<host>`
   - 端口信息：`<ports>` 和 `<port>`
   - 扫描统计：`<runstats>`

4. **XML 输出参数**
   - `-oX <file>`：保存为 XML 文件
   - `-oA <prefix>`：保存所有格式
   - `--no-stylesheet`：不包含 XSL 样式表
   - `-v`：增加详细信息

5. **XML 解析技巧**
   - 使用 `xmllint` 验证格式
   - 使用 `xsltproc` 转换为 HTML
   - 使用 Python `xml.etree.ElementTree` 解析
   - 使用 Perl `Nmap::Parser` 模块解析

6. **实战演练**
   - 生成 XML 输出文件
   - 验证 XML 格式正确性
   - 使用 Python 提取开放端口
   - 转换 XML 为 HTML 报告

### 最佳实践建议

1. **始终使用 `-oX` 或 `-oA`**
   - 保留完整扫描数据，便于后续分析

2. **在自动化脚本中验证 XML**
   - 使用 `xmllint --noout` 检查格式

3. **选择合适的工具解析 XML**
   - Python：适合快速脚本和数据处理
   - Perl：适合与现有 Perl 工具集成
   - XSLT：适合生成报告

4. **定期备份扫描结果**
   - XML 文件体积小，可长期保存
   - 可用于历史对比和趋势分析

5. **考虑安全性**
   - 扫描结果包含敏感信息（网络拓扑、开放服务）
   - 妥善保存 XML 文件，避免泄露

### 下一步学习

- **第7章：Nmap 脚本引擎（NSE）** - 学习如何使用和编写 Nmap 脚本
- **第8章：高级扫描技术** - 学习绕过防火墙和 IDS 的技巧
- **第9章：Nmap 自动化与集成** - 学习将 Nmap 集成到安全工具链中

### 参考资源

- [Nmap 官方文档 - 输出格式](https://nmap.org/book/output.html)
- [Nmap XML 输出格式说明](https://nmap.org/book/output-formats-xml-output.html)
- [Python xml.etree.ElementTree 文档](https://docs.python.org/3/library/xml.etree.elementtree.html)
- [XSLT 教程](https://www.w3schools.com/xml/xsl_intro.asp)

---

**章节完成！** 🎉

恭喜你完成了第6章的学习。现在你应该能够：
- 熟练使用 Nmap 的 XML 输出功能
- 解读 Nmap XML 文件的结构
- 使用工具解析和处理 XML 扫描结果
- 生成专业的扫描报告

在下一章中，我们将学习 Nmap 脚本引擎（NSE），它可以帮助你自动化更多安全检测任务。
