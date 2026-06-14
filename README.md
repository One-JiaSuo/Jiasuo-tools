# 枷锁工具箱 / Jiasuo Tools
枷锁工具箱是一套面向 macOS 的安全测试工具集合，把常用的安全能力按场景整理到一个统一入口里，覆盖信息收集、漏洞扫描、漏洞利用、中间件测试、数据库利用、WebShell 管理和后渗透辅助。工具箱既支持图形界面，也支持通过 geshell 进行命令   行调用，并且为一部分工具提供了 AI 可调用接口，便于自动化执行、批量检查和标准化操作。项目还针对 macOS 做了启动修复、JDK 探测、Python 依赖和 venv 管理，支   持主版本自用和分享版本分发，适合在明确授权的环境里做靶场验证、资产排查和漏洞复现。

> 仅限在明确授权的资产和测试范围内使用。未授权扫描、爆破、利用或访问他人系统违法。
<img width="692" height="644" alt="image" src="https://github.com/user-attachments/assets/d5409d11-4aaa-42b0-9a82-c2ba3b3b2bd7" />


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
## 给使用者的建议

这个工具箱目前主要是在我自己的 Mac 上开发和调试的，我这边能跑起来，不代表所有人的 Mac 下载后都能直接跑起来。因为每个人电脑里的 Python、JDK、Homebrew、系统权限、第三方工具路径都不一样，安全工具又比较依赖环境，所以换一台机器可能会遇到各种路径、权限或依赖问题。

所以我的建议是：下载项目后，先用自己的 AI 编程助手，比如 Claude Code、Cursor、Codex 等，把整个项目读一遍，让它先了解项目结构和启动方式，再根据你自己电脑上的报错来修。

你可以让 AI 重点看这些文件：

```text
README.md
setup.sh
config.yaml
geshell
main.py
ai/tools.md
```

然后先运行：

```bash
./setup.sh --check
```

有问题再根据情况运行：

```bash
./setup.sh --repair
```

或者：

```bash
./setup.sh --full
```

如果还是报错，就把终端报错复制给 AI，让它结合项目代码帮你改。这个项目不是那种完全商业化的一键安装软件，更偏向一个 AI 辅助开发出来的个人安全工具箱，适合有一定动手能力的人根据自己的 Mac 环境进行调整。


## 页面
黑色主题
<img width="2690" height="1464" alt="image" src="https://github.com/user-attachments/assets/6b7ef394-b90e-476a-8f5c-e1fa09effd55" />
白色主题
<img width="2690" height="1464" alt="image" src="https://github.com/user-attachments/assets/d30c4b25-4347-4b47-9c33-1f9385f0a365" />
其他页面
<img width="2690" height="1464" alt="image" src="https://github.com/user-attachments/assets/28d6a25c-eefa-4cfe-9d6d-6d26e883d056" />
支持ai的cli模式调用
例如：
<img width="800" height="800" alt="image" src="https://github.com/user-attachments/assets/e3f2b6f4-42e0-495f-b8c2-1b843d0bfee1" />
<img width="800" height="1000" alt="image" src="https://github.com/user-attachments/assets/722559f0-8c50-427c-9d19-7b7cd1c76e27" />

## 架构

<img width="1000" height="1000" alt="image" src="https://github.com/user-attachments/assets/9654700a-5576-4ea9-8992-57812e96ae52" />


### 核心设计原则

**1. 声明式配置驱动**

所有工具的定义集中在 `config.yaml` 中，不在代码中硬编码。添加新工具只需在 YAML 中新增一段配置，无需修改任何 Python 代码。工具定义与运行时状态分离——`config.yaml` 存放工具声明和 JDK 路径，`state.yaml` 存放用户偏好（主题、字号、窗口位置等），后者可被分享版覆盖而不丢失个性化设置。

**2. 统一的 Runner 抽象**

五种工具类型通过统一的 runner 层抽象：

| 类型      | 说明           | 启动方式                                            |
| --------- | -------------- | --------------------------------------------------- |
| `python`  | Python 脚本    | 独立 venv 中的 Python 解释器                        |
| `jar`     | Java 应用      | `lib/launch_jar.py` 解析 JDK 别名后调用 `java -jar` |
| `command` | Shell 脚本     | `bash start.command` 或直接执行二进制               |
| `cli`     | 系统命令       | 直接在终端中运行（如 `nmap`、`sqlmap`）             |
| `app`     | macOS 原生应用 | `open /Applications/XXX.app`                        |

每种类型都有一个对应的 runner 子类型（`python`/`jar`/`binary`/`system`/`command`/`app`），`launch.py` 中的 `_runner_command()` 负责将 runner 配置解析为可执行的命令列表。

**3. 环境隔离**

每个 Python 工具拥有独立的虚拟环境（venv），避免依赖冲突。Java 工具通过 JDK 别名系统支持同时使用 JDK 8 和 JDK 17，路径由 `config.yaml` 中的 `jdk.jdk8.home` 和 `jdk.jdk17.home` 指定，`setup.sh` 可自动探测和刷新这些路径。

**4. 双入口设计**

- **GUI** (`main.py`): CustomTkinter 实现的图形界面，左侧分类栏 + 右侧工具卡片，支持搜索、主题切换、拖拽排序、工具的增删改
- **CLI** (`geshell` → `ai/launch.py`): 为 AI agent 设计的命令行接口，支持模糊匹配工具名、运行日志记录、自检和文档生成

**5. AI 原生设计**

每个工具有 `ai_callable` 标志和 `risk` 风险等级（passive/active/exploit/brute/tunnel/post-exploit），供 AI agent 做调用决策。`geshell list` 的输出格式、`tools.md` 的文档结构、工具名的容错匹配（忽略大小写、空格、横线、下划线）都是为 AI 调用场景优化的。

---

## 目录结构

```
GetShell/
├── app/                      # GUI 前端
│   ├── gui.py                # 主界面（分类/搜索/启动/主题）
│   ├── config_loader.py      # config.yaml 解析与校验
│   ├── launcher.py           # 工具启动调度（终端/GUI/直接执行）
│   ├── models.py             # 数据模型（AppConfig/ToolConfig/Category/JdkConfig）
│   ├── cards.py              # 工具卡片渲染
│   ├── sidebar.py            # 侧边栏（分类列表/拖拽排序/右键菜单）
│   ├── dialogs.py            # 工具编辑对话框
│   ├── theme.py              # 主题系统（深色/浅色/配色方案）
│   ├── platform.py           # macOS 平台适配（标题栏/外观）
│   ├── scroll.py             # 滚动管理
│   └── constants.py          # 项目根路径常量
│
├── ai/                       # CLI 后端 & AI 接口
│   ├── launch.py             # geshell 后端（工具解析/调度/日志/doctor/selftest）
│   ├── selftest.py           # 最小回归自测
│   └── tools.md              # 自动生成的 AI 参考手册
│
├── lib/                      # 公共库
│   ├── launch_jar.py         # Java 工具启动器（解析 JDK 别名 → java -jar）
│   └── java_home.sh          # JDK 探测脚本
│
├── config.yaml               # 工具声明配置（分类/工具定义/JDK 路径）
├── state.yaml                # 用户运行时状态（主题/字号/窗口位置）
├── geshell                   # CLI 入口脚本（bash 包装 → ai/launch.py）
├── main.py                   # GUI 入口
├── setup.sh                  # 环境安装/修复/检查/分享版导出
├── requirements.txt          # 主环境 Python 依赖
├── 启动.command              # macOS 图形启动入口
│
├── 信息收集/                  # nmap, httpx, ffuf, fscan, hydra, frp...
├── 漏洞扫描/                  # nuclei, sqlmap, dalfox, Ingram...
├── 漏洞利用/                  # SSTImap, Fenjing, docem...
├── 中间件rce/                 # TomcatScanPro, WeblogicTool, PHP免杀马...
├── 组件框架/                  # SpringKit, Shiro, Struts2, ThinkPHP...
├── 数据库rce/                 # RedisKit, 数据库综合利用...
├── 重点系统/                  # 若依, Jenkins, Nacos, XXL-JOB...
├── OA梭哈/                    # IWannaGetAll, Hyacinth
├── CMS Getshell/             # DedeCMS
├── webshell/                 # 冰蝎, 哥斯拉
├── Xray/                     # 被动扫描器
├── CobaltStrike4.9.1/        # CS 团队服务器
└── 后渗透/                    # Metasploit, 反弹Shell
```

---

## 工具分类与调用方式

### 配置驱动下的工具注册机制

每个工具在 `config.yaml` 中的最小定义：

```yaml
- name: nmap
  description: 网络发现与安全审计工具
  type: command
  ai_callable: true
  runner:
    type: system
    command: nmap
  dependencies:
  - nmap
```

关键字段说明：

| 字段           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| `name`         | 工具显示名称，也是 `geshell` 的匹配目标                      |
| `type`         | 五大类型之一：`python` / `jar` / `command` / `cli` / `app`   |
| `ai_callable`  | 是否允许 AI 直接调用（GUI 工具设为 false）                   |
| `risk`         | 风险等级：`passive` / `active` / `exploit` / `brute` / `tunnel` / `post-exploit` |
| `runner`       | 启动器配置（类型 + 路径/入口/JDK 等）                        |
| `dependencies` | 系统依赖列表，doctor 检查时用                                |
| `aliases`      | 别名列表，用于模糊匹配                                       |
| `working_dir`  | 工作目录，相对于项目根目录                                   |
| `venv`         | 是否使用独立虚拟环境                                         |
| `interactive`  | 是否需要在终端中交互运行                                     |

### Runner 类型路由

工具名称匹配后，`ai/launch.py` 的 `build_command()` 函数按 runner 类型路由到不同的命令构建逻辑：

```
runner.type = "system"   → 直接调用系统 PATH 中的命令
runner.type = "binary"   → 调用项目内预编译的二进制文件
runner.type = "python"   → 调用 Python 解释器 + 入口脚本（可选择 venv）
runner.type = "jar"      → 调用 java -jar（通过 lib/launch_jar.py 解析 JDK 别名）
runner.type = "command"  → 调用 .command shell 脚本
runner.type = "app"      → 调用 macOS open 命令
```
---

## 快速开始

```bash
cd GetShell
./setup.sh                 # 修复权限 + 环境检查
```

启动 GUI：

```bash
python3 main.py
```

CLI 工具：

```bash
./geshell list             # 列出所有工具
./geshell info nmap        # 查看工具详情
./geshell nmap -sV -p 22,80,443 192.168.1.1
./geshell nuclei -u http://target.com -severity critical,high
./geshell sqlmap -u "http://target/page?id=1" --batch --dbs
```

如果环境缺失依赖：

```bash
./setup.sh --full          # 完整初始化（Homebrew + Python + JDK + 系统命令 + venv）
```

---

## setup.sh 模式说明

```
./setup.sh                 默认: 修复权限/路径 + 环境检查
./setup.sh --check         只检查环境
./setup.sh --repair        修复 quarantine、权限、geshell 链接、JDK 路径
./setup.sh --install-deps  安装缺失的系统依赖（Homebrew/Python/JDK/nmap 等）
./setup.sh --rebuild-venv  重建所有 Python 虚拟环境
./setup.sh --full          完整初始化（上述所有步骤）
./setup.sh --strict        检查失败时以非零状态退出
./setup.sh --no-doctor     跳过 geshell doctor/selftest
./setup.sh --export-share  生成分享版本
```


---

## 技术栈

| 层级          | 技术                                           |
| ------------- | ---------------------------------------------- |
| GUI           | Python 3.10+, CustomTkinter, tkinter           |
| 配置          | YAML (PyYAML)                                  |
| Java 工具调度 | JDK 8 / JDK 17 双版本支持, `lib/launch_jar.py` |
| 环境管理      | Bash (setup.sh), Python venv, Homebrew         |
| 平台          | macOS (Darwin), Apple Silicon + Rosetta 2      |
| AI 接口       | `ai/launch.py` (纯 Python, 无额外依赖)         |

---

## 常见问题

### geshell 找不到

```bash
./geshell list              # 使用相对路径
./setup.sh --repair         # 修复链接
```

### Python 虚拟环境损坏

```bash
./setup.sh --rebuild-venv
```

### Java 工具启动失败

```bash
./setup.sh --repair         # 自动探测并刷新 JDK 路径到 config.yaml
./geshell doctor            # 检查 JDK 配置
```

### 工具提示依赖缺失

```bash
./setup.sh --full           # 完整安装系统依赖
```

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

已经尽量把本地路径和绑定依赖去掉了，但接收方机器仍可能需要这些基础条件：

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
## 开源与共建

枷锁工具箱是一个完全开源的项目，项目代码、配置和工具调度逻辑都会尽量保持透明。这个项目目前主要是在 AI 辅助下完成开发和整理的，很多功能也是在我自己的使用场景中一点点打磨出来的，所以它不一定完美，也不一定适配所有人的 macOS 环境。

我非常欢迎大家基于这个项目进行二次开发、功能扩展、环境适配和体验优化。无论是新增工具、修复兼容性问题、改进安装流程、完善文档说明，还是优化 GUI、CLI 和 AI 调用方式，都可以在这个项目基础上继续改进。

如果你在使用过程中遇到问题，或者有更好的建议、功能想法、工具推荐、兼容性修复方案，也欢迎通过 Issue、Pull Request 或其他方式反馈。希望这个项目不只是我个人的安全工具箱，也可以慢慢变成一个大家一起维护、一起完善的 macOS 安全测试工具集合。

本项目仅用于合法授权的安全研究、教学、靶场验证和企业内部安全测试。希望大家在合规、授权的前提下使用和改进它，一起把这个工具箱做得更稳定、更好用。



