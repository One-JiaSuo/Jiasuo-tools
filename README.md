# Jiasuo-tools
枷锁工具箱是一套面向 macOS 的安全测试工具集合，把常用的安全能力按场景整理到   一个统一入口里，覆盖信息收集、漏洞扫描、漏洞利用、中间件测试、数据库利用、   WebShell 管理和后渗透辅助。工具箱既支持图形界面，也支持通过 geshell 进行命令   行调用，并且为一部分工具提供了 AI 可调用接口，便于自动化执行、批量检查和标准   化操作。项目还针对 macOS 做了启动修复、JDK 探测、Python 依赖和 venv 管理，支   持主版本自用和分享版本分发，适合在明确授权的环境里做靶场验证、资产排查和漏洞复现。

# 枷锁工具箱

枷锁工具箱是一套面向 macOS 的安全测试工具集合，覆盖信息收集、漏洞扫描、漏洞利用、中间件测试、数据库测试、WebShell 管理和后渗透辅助。它支持图形界面操作，也提供 `geshell` 作为统一命令行入口。

> 仅限在明确授权的资产和测试范围内使用。未授权扫描、爆破、利用或访问他人系统可能违法。

## 目录

- [项目定位](#项目定位)
- [快速开始](#快速开始)
- [安装与修复](#安装与修复)
- [GUI 使用](#gui-使用)
- [CLI 使用](#cli-使用)
- [分发版本](#分发版本)
- [目录说明](#目录说明)
- [常见问题](#常见问题)

## 项目定位
 一个面向 AI 辅助渗透测试的统一工具调度平台。

  1. 统一入口，消除工具碎片化。 40+ 个安全工具（Python/Go/Java/二进制）被归一化为一个 geshell
  命令，屏蔽了底层调用方式的差异。不用记每个工具的路径、语言、启动方式。
  2. AI 原生设计。 AI可用 标志、tools.md 标注为「AI 参考手册」、风险分级（passive/active/exploit
  /brute/tunnel/post-exploit）——这些都不是给人类看的便利贴，而是给 AI agent
  做决策用的元数据。工具名忽略大小写、空格、横线的模糊匹配也是为 AI 调用设计的容错机制。
  3. 覆盖完整杀伤链。 从信息收集（nmap/httpx/urlfinder）→ 漏洞扫描（nuclei/sqlmap/xray）→
  漏洞利用（redis/ssti/spring）→
  后渗透（metasploit/冰蝎/哥斯拉/CobaltStrike），一条链路走到底，不需要在不同平台间切换。
  4. 人机协作分层。 CLI 工具交给 AI 自动调用，GUI 工具（Burp/Yakit/蚁剑/哥斯拉）标记为 ✗
  留给人工操作。不是要替代人，而是把机械化的扫描、枚举、爆破交给 AI，人做决策和复杂利用。

  简单说：这不是一个工具集，而是一个让 AI agent 能像渗透测试工程师一样调用工具的运行时。

## 页面
黑色主题
<img width="2690" height="1464" alt="image" src="https://github.com/user-attachments/assets/6b7ef394-b90e-476a-8f5c-e1fa09effd55" />
白色主题
<img width="2690" height="1464" alt="image" src="https://github.com/user-attachments/assets/d30c4b25-4347-4b47-9c33-1f9385f0a365" />

<img width="2690" height="1464" alt="image" src="https://github.com/user-attachments/assets/28d6a25c-eefa-4cfe-9d6d-6d26e883d056" />
支持ai的cli模式调用
例如：
<img width="1280" height="1730" alt="image" src="https://github.com/user-attachments/assets/e3f2b6f4-42e0-495f-b8c2-1b843d0bfee1" />
<img width="1340" height="1812" alt="image" src="https://github.com/user-attachments/assets/722559f0-8c50-427c-9d19-7b7cd1c76e27" />
<img width="1506" height="688" alt="image" src="https://github.com/user-attachments/assets/1173cba7-2f73-4d43-b912-a621b9024591" />




## 快速开始

主版本解压后进入目录：

```bash
cd GetShell
./setup.sh
```

默认动作是：

- 修复执行权限和 quarantine
- 修复 `geshell` 链接
- 检查 Python、JDK、系统命令和本地 venv
- 运行 `geshell doctor` 和 `geshell selftest`

如果你想做完整初始化，再运行：

```bash
./setup.sh --full
```

启动图形界面：

```bash
python3 main.py
```

查看命令行工具：

```bash
./geshell list
```

如果你已经把 `geshell` 加入 PATH，也可以直接执行：

```bash
geshell list
```

## 安装与修复

`setup.sh` 支持以下模式：

```bash
./setup.sh                 # 默认: 修复权限/路径 + 检查
./setup.sh --check         # 只检查环境
./setup.sh --repair        # 只修复权限、quarantine、JDK 路径、geshell 链接
./setup.sh --install-deps  # 安装缺失系统依赖
./setup.sh --rebuild-venv  # 重建 Python 虚拟环境
./setup.sh --full          # 完整初始化
./setup.sh --strict        # 检查失败时返回非零状态
./setup.sh --no-doctor     # 跳过 geshell doctor/selftest
./setup.sh --export-share  # 生成分享版本目录
./setup.sh -h              # 查看帮助
```

完整初始化会尝试做这些事：

- 安装或确认 Homebrew
- 确认 Python 3.10+
- 安装 PyYAML 和 customtkinter
- 探测并刷新 JDK 8 / JDK 17 路径
- 在 Apple Silicon 上按需提示 Rosetta 2
- 安装常用系统命令：`nmap`、`sqlmap`、`ffuf`、`nuclei`、`httpx`、`hydra`、`frp`、`interactsh`
- 重建主环境和各子工具的 Python venv

### 什么时候用 `--repair`

如果你只是从压缩包里解压出来，或者双击 `.command` 后提示权限、隔离属性、`geshell` 找不到，一般先跑：

```bash
./setup.sh --repair
```

### 什么时候用 `--full`

如果这台机器是第一次用，或者 GUI / CLI 一堆依赖不满足，直接跑：

```bash
./setup.sh --full
```


## GUI 使用

启动主界面：

```bash
python3 main.py
```

界面里有几个固定操作：

- 左侧分类栏，用来按用途筛工具
- 顶部搜索框，支持模糊搜索
- 工具卡片，点击直接启动
- 右上角有字体、主题和配置刷新入口

### App 类型工具

标记为 `app` 的工具不会帮你安装，只会在界面里提示你自行安装到本机。常见的有：

- Burp Suite
- Yakit
- TscanPlus
- 蚁剑

这类工具的行为是：

- 机器上存在对应 `.app` 时可直接打开
- 不存在时会显示“需要自行安装”的提示
- 分享版本里也保留这个提示，但不会强行把它们包装成安装步骤

### 分类说明

当前主版本按用途分组，主要分类包括：

- 常规工具
- 信息收集
- 漏洞扫描
- 重点系统
- 中间件&框架
- 数据库利用
- Webshell管理
- 后渗透


## CLI 使用

`geshell` 是统一命令行入口：

```bash
geshell list              # 列出工具
geshell info <工具名>      # 查看工具详情
geshell doctor            # 检查配置、依赖和入口
geshell selftest          # 运行最小回归测试
geshell gendocs           # 重新生成 ai/tools.md
geshell <工具名> [参数...]  # 调用工具
```

工具名匹配会忽略大小写、空格、横线和下划线，并支持 `aliases`。

示例：

```bash
geshell nmap -sV -p 22,80,443 192.168.1.1
geshell sqlmap -u "http://target.example/page?id=1" --batch --dbs
geshell dirsearch -u http://target.example -e php,html,bak
geshell ffuf -u http://target.example/FUZZ -w wordlist.txt -fc 404
geshell nuclei -u http://target.example -severity critical,high
geshell dalfox url http://target.example
```

每次通过 `geshell` 运行工具时，日志会写入：

```text
output/runs/<时间>_<工具名>/
```

里面通常包含：

- `command.txt`
- `stdout.txt`
- `stderr.txt`
- `meta.yaml`



### 接收方可能需要自行补齐

分享版已经尽量把本地路径和绑定依赖去掉了，但接收方机器仍可能需要这些基础条件：

- Python 3.10+
- Homebrew
- 常用系统命令，例如 `nmap`、`ffuf`、`nuclei`
- JDK 8 和 JDK 17
- Apple Silicon 上的 Rosetta 2

如果这些条件缺失，`./setup.sh --full` 会尽量补齐；其中商业软件和第三方 App 仍需要接收方自己安装。

## 目录说明

```text
app/                 图形界面代码
ai/                  geshell 后端、工具文档和自测
lib/                 公共辅助脚本，例如 JDK 启动器
config.yaml          工具配置、入口、依赖和 JDK 路径
state.yaml           本地状态
setup.sh             安装、修复、检查和导出脚本
geshell              命令行统一入口
启动.command         macOS 图形启动入口
output/              运行输出和日志
```

各工具按用途放在中文分类目录下，例如：

- `信息收集/`
- `漏洞扫描/`
- `中间件rce/`
- `数据库rce/`
- `后渗透/`

## 常见问题

### `geshell` 找不到

可以直接使用相对路径：

```bash
./geshell list
```

或者重新修复链接：

```bash
./setup.sh --repair
```

### `geshell doctor` 提示依赖缺失

先确认你是否真的需要这些工具。如果需要完整初始化：

```bash
./setup.sh --full
```

### Python 虚拟环境损坏

重建虚拟环境：

```bash
./setup.sh --rebuild-venv
```

### Java 工具启动失败

先修复，再检查：

```bash
./setup.sh --repair
./geshell doctor
```

如果仍失败，检查 `config.yaml` 中 `jdk8.home` 和 `jdk17.home` 是否指向真实 JDK 目录。

### App 工具打不开

如果工具类型是 `app`，先确认你本机是否安装了对应应用。工具箱不会代替你安装这些 App。
## 测试效果
<img width="1960" height="866" alt="image" src="https://github.com/user-attachments/assets/67de3cd2-acf0-4067-8d21-590e10bd27ca" />


