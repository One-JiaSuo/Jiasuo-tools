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

这套工具箱有两个使用场景：

1. 你自己的主版本，保留完整工具和本机配置。
2. 发给别人的分享版本，自动裁掉你不想分发的工具，并提示对方自行安装本机 App。

主版本以仓库根目录为准，分享版本由 `./setup.sh --export-share` 生成，默认输出到 `GetShell-分享版/`。

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

## 分发版本

如果你要把工具箱发给别人，先从主版本生成分享版：

```bash
./setup.sh --export-share
```

生成结果在：

```text
GetShell-分享版/
```

分享版的规则是：

- 去掉你不想分发的工具
- 去掉内部维护文件
- 保留 `app` 类型工具，但提示接收方自己安装
- 把 Python 工具的启动方式调整为更适合接收方环境的形式
- 自带可运行的 `config.yaml`、`ai/tools.md`、`state.yaml`

发出去之前，建议你在主版本先检查一次：

```bash
./setup.sh --check --strict
```

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

### 分享版和主版本的区别

- 主版本：保留你自己的完整工具和个人配置
- 分享版本：裁掉你不想发的工具，适合直接给别人解压使用

## 分发建议

建议你在主版本里先跑：

```bash
./setup.sh --check --strict
```

通过后再导出分享版：

```bash
./setup.sh --export-share
```

然后把 `GetShell-分享版/` 发给对方。对方通常只需要：

```bash
cd GetShell-分享版
./setup.sh
python3 main.py
```

如果对方机器缺少系统依赖，再补：

```bash
./setup.sh --full
```
