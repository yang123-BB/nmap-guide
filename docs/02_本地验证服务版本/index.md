# 在本地验证服务版本

> **Nmap 入门指南 · LabEx 技能树**
> 
> 标签：`cybersecurity` `nmap` `linux`

---

## 概述

本实验将指导你掌握 Nmap 的**服务版本检测**（Version Detection）功能，学会使用 `-sV` 参数识别目标主机上运行服务的具体版本信息。你将了解版本检测的工作原理，并通过多个实战练习逐步熟练运用这一关键扫描技术，从而在网络安全评估中获得更精确的目标系统情报。

---

## 学习目标

完成本实验后，你将能够：

1. 理解**服务版本检测**（Version Detection）的概念及其在网络安全评估中的重要性
2. 掌握 Nmap `-sV` 参数的基本用法，能够对目标主机执行版本扫描
3. 理解并运用 `--version-intensity` 参数调整检测深度与速度的平衡
4. 解读版本检测结果中各字段的含义，准确识别服务类型和版本号
5. 结合 `-p` 端口参数和 `-sV` 实现定向版本检测
6. 了解版本检测脚本（`--version-trace`、`--script`）的高级用法

---

## 背景知识

### 服务版本检测的重要性

在一次完整的网络安全评估中，**端口扫描**只是第一步。知道目标主机开放了哪些端口固然重要，但真正决定后续攻击路径的，是这些端口上运行的**服务类型及其精确版本**。

举例来说，端口扫描可能告诉你目标开放了 `80/tcp`，这意味着可能有一个 Web 服务器。但仅凭端口号，你无法判断它是 Apache 2.4.37、Nginx 1.14.0，还是某个存在严重漏洞的老旧版本 IIS。攻击者可以利用版本信息直接查找对应的 CVE 漏洞，从而发起精准攻击。

**版本检测的价值在于：**

- **精确识别服务**：区分运行在同一端口上的不同服务（如 MySQL vs PostgreSQL）
- **漏洞关联**：版本号是关联 CVE/CVEs（通用漏洞披露）的直接依据
- **风险评估**：帮助安全团队优先处理已知漏洞，避免资源浪费在无漏洞的系统上
- **合规报告**：在渗透测试报告中提供准确的服务版本信息，增强报告说服力

### 版本检测原理（Banner Grabbing / 指纹识别）

Nmap 的版本检测并非依赖"魔法"，而是基于两种核心技术的组合：

#### 1. Banner Grabbing（横幅抓取）

这是最简单直接的方法。Nmap 向目标端口发送标准协议请求，然后**读取服务返回的 banner 信息**（即服务在连接建立后主动发送的元数据文本）。例如，当 Nmap 连接 SSH 端口时，SSH 服务器通常会返回一个类似以下的 banner：

```
SSH-2.0-OpenSSH_7.4
```

这就是服务主动"自我介绍"的方式。Banner Grabbing 简单高效，但并非所有服务都会发送有意义的 banner。

#### 2. 指纹识别（Fingerprinting）

当服务不主动发送 banner，或者 banner 信息不足以判断版本时，Nmap 会采用**主动探测**的方式：

1. Nmap 向目标端口发送一系列精心设计的探测包（probe），针对不同协议有不同的试探策略
2. 根据服务返回的响应特征，与已知服务指纹数据库进行**模式匹配**
3. 通过响应中的特定字段、错误消息格式、握手行为等细节，综合判断服务类型和版本

指纹识别的优势在于，即使服务刻意隐藏了 banner，Nmap 仍能通过行为特征进行识别。

### 服务指纹数据库（nmap-service-probes）

Nmap 的版本检测能力核心依赖 `nmap-service-probes` 数据库文件。该文件定义了：

| 数据库内容 | 说明 |
|---|---|
| **探测规则**（probe） | 针对不同端口/协议发送的探测请求 |
| **匹配规则**（match） | 服务响应与已知服务指纹的对应关系 |
| **版本信息** | 匹配成功后输出的服务名称、版本号、操作系统兼容信息等 |

该文件通常位于 Nmap 安装目录下的 `nselib/data/` 文件夹中（Linux 发行版中路径为 `/usr/share/nmap/nselib/data/`）。

数据库中包含了数千种已知服务的指纹定义，涵盖了常见的 HTTP、SMTP、FTP、SSH、MySQL、Redis 等协议。如果扫描结果中显示 `.Service Info:` 行，就说明数据库匹配到了服务信息。

### 为什么版本检测比端口扫描更重要

让我们用一个实际场景来理解这一点：

| 扫描类型 | 扫描结果 | 后续行动 |
|---|---|---|
| 端口扫描 | `22/tcp open ssh` | 只能尝试通用 SSH 攻击，效率低 |
| 版本检测 | `22/tcp open ssh OpenSSH 7.4 (protocol 2.0)` | 可直接查询 OpenSSH 7.4 的 CVE-2017-15906 |

仅知道端口号，安全评估人员不得不进行大量无意义的尝试。而有了版本信息，评估工作可以**直击要害**，大幅提升效率。

此外，在**授权渗透测试**场景中，精确的版本信息决定了测试报告的专业程度。一份列出"Apache httpd 2.4.38 (Unix)"比仅列出"80/tcp open http"的报告更有价值，也更能帮助客户理解实际风险。

---

## 版本检测技术详解

### -sV 参数详解

`-sV` 是 Nmap 版本检测的核心参数，也是你最常用的命令选项。

```bash
# 基本语法：扫描目标的所有端口并尝试检测服务版本
nmap -sV <目标>

# 常见组合：快速扫描常用端口并检测版本
nmap -sV -p 1-1000 <目标>

# 全端口扫描 + 版本检测（耗时较长）
nmap -sV -p- <目标>
```

**`-sV` 的行为特点：**

- 自动启用 `-sS`（TCP SYN 扫描）和 `-Pn`（跳过主机发现）的某些默认行为
- 会向已发现的开放端口发送探测包，**不会额外扫描未开放的端口**
- 版本检测会增加扫描时间，因为每个开放端口都需要进行额外的协议交互

#### 常见组合方式

```bash
# 示例 1：扫描本地主机所有端口的版本信息
nmap -sV 127.0.0.1

# 示例 2：扫描特定端口范围（-p 参数指定端口）
nmap -sV -p 22,80,443,3306 192.168.1.100

# 示例 3：与操作系统检测结合（-O 参数）
nmap -sV -O 192.168.1.100

# 示例 4：使用诱饵 IP 增加扫描隐蔽性
nmap -sV -p 80,443 -D 10.0.0.1,10.0.0.2 192.168.1.100
```

### --version-intensity 参数（0-9 级别）

`--version-intensity`（简写为 `--version-intensity`）用于控制版本检测的**深度**，取值范围为 **0 到 9**：

| 强度级别 | 说明 | 适用场景 |
|---|---|---|
| **0** | 最轻量级，仅发送最可靠的探测 | 快速扫描，节省时间 |
| **1-2** | 轻度检测，发送较少探测包 | 一般评估 |
| **3-4** | 中等强度（**默认值**） | 平衡速度与准确性 |
| **5-6** | 较深检测，发送更多探测包 | 需要更准确结果时 |
| **7-8** | 深度检测，覆盖更多边界情况 | 详细评估 |
| **9** | 最激进，发送全部可能探测 | 最全面检测，但耗时最长 |

```bash
# 示例：使用强度 9 进行最激进的版本检测
nmap -sV --version-intensity 9 -p 80,443 192.168.1.100

# 示例：使用强度 0 快速扫描（结果可能不完整）
nmap -sV --version-intensity 0 -p 1-1000 192.168.1.100
```

**实际效果对比：**

强度越高，Nmap 发送的探测包越多，耗时也越长。在网络带宽有限或目标系统负载敏感的情况下，适当降低强度可以减少对目标的影响。以下是实际扫描中不同强度的典型耗时对比（扫描 10 个端口）：

```
--version-intensity 0:  ~3-5 秒
--version-intensity 5:  ~8-15 秒
--version-intensity 9:  ~20-40 秒
```

### --version-all 强制检测

`--version-all` 等同于 `--version-intensity 9`，强制对**每个开放端口**使用最高强度检测。当你需要确保不遗漏任何版本信息时，使用此参数：

```bash
# 对所有端口使用最强检测
nmap -sV --version-all -p- 192.168.1.100
```

**注意**：`--version-all` 会显著增加扫描时间。在实际工作中，如果你已经知道某些端口不重要，可以先用 `-p` 参数限定范围，再用 `--version-all`：

```bash
# 仅对关键端口使用最强检测
nmap -sV --version-all -p 22,80,443,3306,5432 192.168.1.100
```

### 版本检测结果字段解析

运行 `-sV` 扫描后，结果中会包含比普通端口扫描更丰富的信息。理解这些字段是解读扫描结果的关键：

```
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 7.4 (protocol 2.0)
80/tcp   open  http     Apache httpd 2.4.38
443/tcp  open  ssl      OpenSSL 1.1.1k  (FIPS)
3306/tcp open  mysql    MySQL 5.7.38
```

**各字段含义：**

| 字段 | 说明 |
|---|---|
| `PORT/tcp` | 端口号及协议类型 |
| `STATE` | 端口状态（open/closed/filtered） |
| `SERVICE` | 识别到的服务名称 |
| `VERSION` | 识别到的具体版本信息，包含软件名、版本号、可选附加信息 |

**进阶字段（使用 `-sVV` 双重冗余模式）：**

```bash
# 使用 -vv（very verbose）获取更详细的输出
nmap -sV -vv 127.0.0.1
```

双重冗余模式下，部分服务的检测结果会包含额外信息，例如脚本执行结果或更详细的版本字符串：

```
3306/tcp open  mysql     MySQL 5.7.38
|_mysql-databases: information_schema, mysql, performance_schema, sys
|mysql-brute:      'root' login has no password
```

**其他常见输出字段：**

| 字段 | 含义 |
|---|---|
| `product` | 识别到的产品名称 |
| `version` | 具体版本号 |
| `extrainfo` | 额外信息（如操作系统平台、编译选项等） |
| `ostype` | 操作系统类型（如果可识别） |
| `Service Info` | 综合服务信息（如 hostname、操作系统等） |

---

## 实战练习

> **实验环境说明**：以下练习建议在本地实验环境（如 LabEx 提供的靶机）中执行。扫描自己搭建的测试环境是学习版本检测的最佳方式。请务必在**授权环境**中进行扫描。

### 练习 1：基础版本检测

**目标**：对本地主机执行基本的版本检测扫描，了解 `-sV` 参数的标准输出格式。

**命令**：

```bash
# 扫描本地主机（127.0.0.1）的常用端口并检测版本
nmap -sV 127.0.0.1 -p 22,80,443,3306
```

**预期输出示例**：

```
Starting Nmap 7.94 ( https://nmap.org )
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00040s latency).

PORT     STATE  SERVICE  VERSION
22/tcp   open   ssh      OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp   open   http     Apache httpd 2.4.52
443/tcp  open   ssl      OpenSSL 1.1.1k  25 Mar 2021
3306/tcp open   mysql    MySQL 5.7.38

Service detection performed. Nmap done: 1 IP address (1 host up) scanned in 6.52 seconds.
```

**解读要点**：

- `PORT` 列显示了端口号和协议
- `STATE` 为 `open` 表示端口处于活跃状态
- `SERVICE` 是 Nmap 识别出的服务名称
- `VERSION` 包含了软件名称和精确版本号——**这正是版本检测的核心价值所在**

---

### 练习 2：调整检测强度

**目标**：对比不同 `--version-intensity` 设置下的扫描结果，理解强度参数对检测准确性的影响。

**命令**：

```bash
# 使用强度 2（轻量级）
nmap -sV --version-intensity 2 -p 80 127.0.0.1

# 使用强度 9（最强检测）
nmap -sV --version-intensity 9 -p 80 127.0.0.1
```

**预期输出对比**：

```
# --version-intensity 2 结果：
80/tcp open http Apache httpd (unknown version)

# --version-intensity 9 结果：
80/tcp open http Apache httpd 2.4.52 (Unix, OpenSSL 1.1.1k  25 Mar 2021)
```

**观察结论**：

- 强度较低时，Nmap 可能只能识别到**大类**（如 "Apache httpd"）而无法确定具体版本
- 强度最高时，版本号、操作系统平台、OpenSSL 版本等详细信息全部呈现
- 如果某个服务在强度 2 下无法识别版本，尝试提高强度是排查问题的第一步

**实用建议**：在实际渗透测试中，可以先用中等强度快速扫描，识别出大类型后，对重要端口再用高强度精确检测。

---

### 练习 3：特定端口的版本检测

**目标**：学会使用 `-p` 参数精准指定端口，避免扫描整个网络时的噪音和耗时。

**命令**：

```bash
# 仅检测 SSH 服务的版本
nmap -sV -p 22 192.168.1.100

# 检测多个非标准端口（常见数据库管理端口）
nmap -sV -p 3306,5432,27017,6379 192.168.1.100

# 扫描端口范围（1000-2000 范围内的版本检测）
nmap -sV -p 1000-2000 192.168.1.100
```

**预期输出示例**（多端口检测）：

```
PORT      STATE  SERVICE  VERSION
3306/tcp  open   mysql    MySQL 5.7.38
5432/tcp  open   postgresql PostgreSQL 13.4 on x86_64-pc-linux-gnu
27017/tcp open   mongodb  MongoDB 5.0.5
6379/tcp  open   redis    Redis 6.0.16
```

**场景应用**：在已知目标可能运行特定服务时，定向端口扫描是最高效的方式。例如，如果你在侦察阶段发现目标开放了非标准的高端口号，可以针对该端口进行版本检测以确认服务身份。

---

### 练习 4：版本检测与端口扫描组合

**目标**：将版本检测与其他扫描技术组合使用，构建完整的侦查流程。

**命令**：

```bash
# 组合 1：主机发现 + 端口扫描 + 版本检测（最完整的扫描链）
nmap -sV -Pn -p 1-1000 192.168.1.100

# 组合 2：SYN 扫描 + 版本检测（默认扫描行为）
nmap -sV -sS -p 22,80,443 192.168.1.100

# 组合 3：版本检测 + 操作系统检测（OS 指纹 + 服务版本双重识别）
nmap -sV -O -p 80,443 192.168.1.100
```

**预期输出示例**（组合 3）：

```
PORT    STATE  SERVICE  VERSION
80/tcp  open   http     Apache httpd 2.4.52
443/tcp open   ssl      OpenSSL 1.1.1k  25 Mar 2021

Device type: general purpose
Running: Linux 4.X
OS details: Linux 4.15-5.0 (Ubuntu)
```

**组合策略建议**：

| 场景 | 推荐组合 | 原因 |
|---|---|---|
| **快速侦察** | `-sV -p 1-1000` | 平衡速度与覆盖面 |
| **详细评估** | `-sV -O -p-` | 最全面的信息收集 |
| **定向攻击准备** | `-sV -p <已知端口>` | 针对已知目标精确打击 |
| **规避检测** | `-sV -p -T2 -f` | 降低扫描速率并分片 |

> **提示**：`--top-ports <N>` 参数可以自动扫描最常见的 N 个端口，与 `-sV` 组合使用非常方便：
> ```bash
> nmap -sV --top-ports 20 192.168.1.100
> ```

---

## 高级技巧

### 版本检测脚本

Nmap 内置了丰富的 **NSE 脚本**（Nmap Scripting Engine），其中部分脚本专门服务于版本检测的进一步深入分析：

```bash
# 使用版本相关脚本进行深度检测
nmap -sV --script "version-*" 127.0.0.1

# 使用所有相关脚本并显示详细输出
nmap -sV -sVV --script "version,default" -p 80,443 127.0.0.1
```

**常用的版本相关 NSE 脚本：**

| 脚本名称 | 功能 |
|---|---|
| `version` | 获取所有检测到的服务的详细版本信息 |
| `banner` | 简单抓取服务 banner 横幅 |
| `ssl-cert` | 提取 SSL 证书详细信息（包括有效期、颁发者、域名） |
| `http-title` | 获取 HTTP 服务的页面标题 |
| `mysql-info` | 获取 MySQL 连接信息和版本字符串 |
| `ssh-hostkey` | 获取 SSH 服务的主机密钥指纹 |

```bash
# 示例：获取 SSL 证书详细信息（常用于发现过期证书）
nmap -sV -p 443 --script ssl-cert 127.0.0.1

# 示例：获取 Web 服务标题
nmap -sV -p 80 --script http-title 127.0.0.1

# 示例：一次性获取多个服务的关键信息
nmap -sV -sVV --script banner,mysql-info,ssh-hostkey 127.0.0.1
```

### 自定义探测字符串

在某些特殊场景下，目标服务可能使用了非标准配置，导致默认探测失败。此时可以通过 `--script-args` 传递自定义参数：

```bash
# 自定义 HTTP User-Agent（绕过某些 WAF 的检测）
nmap -sV -p 80 --script http-headers,http-title \
  --script-args http.useragent="Mozilla/5.0 (Windows NT 10.0; Win64; x64)" \
  127.0.0.1

# 自定义 SSH 握手参数
nmap -sV -p 22 --script ssh2-algos \
  --script-args ssh2-algos="ecdh-sha2-nistp256" \
  127.0.0.1
```

对于更高级的需求，可以直接编辑 `nmap-service-probes` 文件添加自定义匹配规则，但这需要深入理解 Nmap 的探测协议格式，通常仅在企业环境中针对特定内部服务时才需要。

### 排除某些服务的版本检测

如果某些端口的版本检测导致连接超时、连接被拒绝或触发防护机制，可以使用 `--exclude-ports` 排除这些端口：

```bash
# 排除某些端口不做版本检测（但仍然扫描端口状态）
nmap -sV -p 1-1000 --exclude-ports 8080,8443 192.168.1.100
```

另一个有用的技巧是 `--version-trace`，它可以输出版本检测的完整调试日志，帮助你分析为什么某些服务无法被正确识别：

```bash
# 开启追踪日志，查看版本检测的详细过程
nmap -sV -p 80 --version-trace /tmp/version-debug.log 127.0.0.1
cat /tmp/version-debug.log
```

通过追踪日志，你可以看到 Nmap 发送了哪些探测包、收到了哪些响应，以及为什么某个服务没有被匹配到。

---

## 常见问题

### Q1：为什么 `-sV` 扫描比普通端口扫描慢很多？

**A**：`-sV` 需要与每个开放端口进行**协议级别的交互**。它不是简单地发送 TCP SYN 包判断端口是否开放，而是发送特定的探测包并等待服务响应。这个过程涉及网络往返延迟（Round Trip Time）和可能的超时等待，因此耗时显著增加。你可以通过降低 `--version-intensity` 减少探测数量，或使用 `-p` 参数限定端口范围来控制扫描时间。

---

### Q2：版本检测无法识别某些服务怎么办？

**A**：按以下步骤排查：

1. **提高检测强度**：尝试 `--version-intensity 9`
2. **使用 NSE 脚本**：如 `banner` 脚本可能抓取到默认 `-sV` 遗漏的 banner
3. **开启追踪日志**：`--version-trace` 查看探测过程
4. **手动 Banner Grabbing**：使用 `nc` 或 `telnet` 直接连接端口查看服务 banner
5. **更新 Nmap**：新版本包含更新的 `nmap-service-probes` 数据库

```bash
# 手动抓取 banner（适用于 Web 服务）
echo "GET / HTTP/1.0\r\n\r\n" | nc 127.0.0.1 80

# 使用 netcat 获取任意服务的 banner
nc -nv 127.0.0.1 3306
```

---

### Q3：`-sV` 会触发入侵检测系统（IDS/IPS）吗？

**A**：**是的**。`-sV` 的行为模式与普通端口扫描有显著不同——它会发送非标准的协议请求，与服务进行实际的数据交互，这比简单的端口探测更容易被 IDS/IPS 检测到。规避建议包括：

- 降低扫描速率（`-T2` 或更低）
- 使用 `--version-intensity 0` 或 `1` 减少探测数量
- 在时间窗口允许的情况下分批次扫描
- 优先对目标进行侦察后再精准定向扫描

---

### Q4：版本检测结果中显示 `unknown` 或 `tcpwrapped` 是什么意思？

**A**：

- **`unknown`**：Nmap 发送了探测但没有找到匹配的指纹，服务无法被识别。可以尝试更高强度或手动排查。
- **`tcpwrapped`**：表示端口虽然开放，但连接被 TCP wrapper（`hosts.allow`/`hosts.deny`）或防火墙限制，Nmap 无法与服务完成完整的协议交互，因此无法进行版本检测。

---

### Q5：如何在自动化脚本中使用版本检测？

**A**：可以将 Nmap 的 XML 输出格式与脚本结合使用：

```bash
# 输出为 XML 格式，便于程序解析
nmap -sV -oX output.xml 192.168.1.100

# 输出为 grepable 格式，便于 grep/awk 处理
nmap -sV -oG output.grep 192.168.1.100
```

示例：用 grep 快速提取所有 MySQL 版本信息：

```bash
grep "mysql" output.grep
```

---

### Q6：版本检测能穿透 NAT 吗？

**A**：**可以**。版本检测工作在 TCP/UDP 层面，只要 Nmap 能与目标端口建立连接（通过 NAT 映射），就可以进行版本检测。但需要注意：NAT 环境中的端口映射通常是动态的，如果目标在内网中只有特定端口被映射出来，你需要先知道这些端口才能进行扫描。

---

### Q7：`-sV` 和 `-A` 参数有什么区别？

**A**：

| 参数 | 包含功能 | 说明 |
|---|---|---|
| `-sV` | 版本检测 | 仅检测服务版本 |
| `-A` | 版本检测 + 操作系统检测 + 脚本扫描 + traceroute | 综合检测，最全面但耗时最长 |

`-A` 等价于 `-sV -sS -sC -O --traceroute`。如果你只需要版本信息，使用 `-sV` 更加高效且不易触发告警。

---

### Q8：扫描结果显示 `.Service Info:` 行是什么意思？

**A**：`Service Info` 行来自 Nmap 的**服务指纹数据库**（`nmap-service-probes`）的综合匹配。当数据库中的多个匹配规则都对同一个端口生效时，Nmap 会汇总这些信息。例如：

```
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 7.4
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

`Service Info` 提供了超越单一端口的全局服务信息，包括**操作系统**（通过 CPE 格式标识）、服务的主机名、证书信息等。`Service Info` 来自 `nmap-service-probes` 数据库的综合分析，而不是单个端口的探测结果。

---

## 总结

本实验涵盖了 Nmap 版本检测的核心知识点。以下是你需要记住的关键要点：

| 核心概念 | 关键命令 | 注意事项 |
|---|---|---|
| 基本版本检测 | `nmap -sV <目标>` | 扫描时间显著长于普通端口扫描 |
| 调整检测强度 | `--version-intensity 0-9` | 强度越高越准确，但越耗时 |
| 强制最强检测 | `--version-all` | 等同于 `--version-intensity 9` |
| 定向端口检测 | `-sV -p <端口>` | 节省时间，减少噪音 |
| 版本检测脚本 | `--script "version-*"` | 深入检测特定服务 |
| 追踪调试 | `--version-trace` | 排查无法识别的服务 |
| 综合扫描 | `-A` | 最全面，但开销最大 |

**三条最重要的经验**：

1. **始终从 `-sV` 开始**：在端口扫描完成后，对重要端口进行版本检测是标准操作流程
2. **强度不是越高越好**：根据任务目标在准确性和速度之间做出合理取舍
3. **版本信息是漏洞评估的基础**：没有准确的版本号，就无法关联可靠的 CVE 漏洞数据

掌握版本检测，意味着你从"知道有什么服务在运行"升级到了"知道那些服务具体是什么、哪个版本、面临什么风险"。这是成为专业安全评估人员的关键一步。

---

## 参考资料

- **Nmap 官方文档 - Version Detection**：[https://nmap.org/book/man-version-detection.html](https://nmap.org/book/man-version-detection.html)
- **Nmap Reference Guide**：[https://nmap.org/book/man.html](https://nmap.org/book/man.html)
- **Nmap Scripting Engine (NSE)**：[https://nmap.org/book/nse.html](https://nmap.org/book/nse.html)
- **nmap-service-probes 数据库**：[https://nmap.org/book/data-files.html](https://nmap.org/book/data-files.html)
- **CVE 漏洞数据库**：[https://cve.mitre.org/](https://cve.mitre.org/)
- **LabEx Nmap 技能树**：[https://labex.io/skills/nmap](https://labex.io/skills/nmap)
- **Nmap Cheat Sheet（快速参考）**：[https://nmap.org/book/man-briefoptions.html](https://nmap.org/book/man-briefoptions.html)

---

> **下一步建议**：完成本实验后，继续学习 Nmap 的 **NSE 脚本使用**和**操作系统检测（-O）**相关实验，构建更完整的网络侦查技能体系。