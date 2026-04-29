# Playwright \+ Python Environment Installation \&amp; Dependency Issues Summary \(Dual\-Language 英中双语版\)

## Preface 前言

**English**: During my daily work and personal learning of automated testing with Playwright \+ Python, environment installation and dependency configuration issues were the very first problems I encountered repeatedly and spent a lot of time troubleshooting\. This document sorts out the common installation errors, actual performance phenomena, real root causes, and practical one\-click solutions based on my own practical operation and stepping on pits in projects\. It covers local Windows and Mac development environments as well as Linux server deployment and CI operation scenarios, recorded and sorted out for my own subsequent review and reference, and also shared for peers who are learning and using Playwright\.

**中文**: 我自己在日常工作实操和业余学习使用 Playwright \+ Python 做自动化测试的过程中，环境安装与依赖配置相关问题，是我最先遇到、也是反复踩坑、耗费较多时间排查解决的基础问题。本文结合我个人项目实操和实际踩坑经历，整理汇总工作学习中碰到的高频安装报错、真实故障现象、问题根本原因以及亲测有效的一键解决方案，覆盖Windows、Mac本地开发环境，以及Linux服务器部署、CI运行场景，纯属个人实操经验复盘总结，方便自己后续查阅复用，也分享给正在入门和使用Playwright的同行小伙伴参考避坑。

---

## 1\. Installation Error: Could not find a version / Download Timeout 安装时报错：版本找不到/下载超时

### Phenomenon 故障现象

**English**: The command `pip install playwright` fails to execute normally; the system prompts *Could not find a version that satisfies the requirement playwright*; the installation process gets stuck for a long time and finally reports network connection timeout or connection failure errors\.

**中文**: 执行 `pip install playwright` 安装命令运行失败；控制台提示找不到满足Playwright安装要求的对应版本；安装流程长时间卡住无进度，最终弹出网络连接超时、网络连接失败等报错信息。

### Root Cause 根本原因

**English**: Unsupported low Python version \(below 3\.8\); outdated pip tool version leading to incompatible package index protocol; poor domestic network environment and default official PyPI source slow access; damaged local pip cache data affecting normal installation parsing\.

**中文**: Python版本过低不兼容（版本低于3\.8）；pip工具版本老旧，不支持新版包索引协议；国内网络环境受限，默认官方PyPI源访问速度缓慢；本地pip缓存文件损坏异常，干扰安装包正常解析和下载流程。

### One\-Stop Solution 一站式解决方案

```bash
# Check Python version (must be ≥3.8) 检查Python版本（必须3.8及以上）
python --version

# Upgrade pip to the latest stable version 升级pip至最新稳定版本
python -m pip install --upgrade pip

# Install Playwright with domestic mirror source (Alibaba Cloud) 国内镜像源加速安装（阿里云推荐）
pip install playwright -i https://mirrors.aliyun.com/pypi/simple/

# Clean damaged cache and retry installation 清理损坏缓存后重试安装
pip cache purge
pip install playwright
```

---

## 2\. Post\-Installation Execution Error: Browser executable not found / Driver version mismatch 安装后执行报错：浏览器程序未找到/驱动版本不匹配

### Phenomenon 故障现象

**English**: Running Python test scripts prompts*Browser executable not found*; the system reports driver version mismatch and fails to launch Chromium, Firefox or WebKit browsers; Playwright Python package is installed successfully, but the browser crashes immediately after startup\.

**中文**: 运行Python自动化测试脚本时，提示找不到浏览器可执行程序；系统上报驱动版本不匹配，无法正常启动Chromium、Firefox、WebKit任意内核浏览器；Playwright Python包安装显示成功，但浏览器启动后直接闪退崩溃。

### Root Cause 根本原因

**English**: The core missing operation: only the Python dependency package was installed, without executing the dedicated browser installation command; manual manual installation of browser programs leads to version incompatibility with Playwright built\-in driver; the system environment variable does not configure the browser path, causing Playwright to fail to identify and call the browser normally\.

**中文**: 核心操作缺失：仅完成Python依赖包安装，未执行专属浏览器配套安装命令；手动自行下载安装浏览器程序，导致版本与Playwright内置驱动不兼容；系统环境变量未配置浏览器运行路径，Playwright无法正常识别调用浏览器程序。

### One\-Stop Solution 一站式解决方案

```bash
# Core mandatory command: Install official matched browsers 核心必执行命令：安装官方配套适配浏览器
playwright install

# Optional: Install only the specified browser to save disk space 可选：仅安装指定浏览器，节省磁盘空间
# playwright install chrome
# playwright install firefox
```

**English Note**: The `playwright install` command will automatically download and install the browser version fully matched with the current Playwright version\. There is no need to manually download and configure drivers such as ChromeDriver, which is the biggest difference between Playwright and Selenium\.

**中文备注**：`playwright install` 命令会自动下载安装与当前Playwright版本完全适配匹配的浏览器，无需像Selenium一样手动下载配置ChromeDriver等驱动，这是Playwright核心优势之一。

---

## 3\. Dependency Conflict: pyee / typing\-extensions Version Conflict 依赖冲突：pyee/typing\-extensions版本兼容报错

### Phenomenon 故障现象

**English**: Importing Playwright modules reports ImportError and fails to load normally; script runtime prompts event loop abnormality and non\-existent attribute calling; other project third\-party libraries conflict with Playwright basic dependencies\.

**中文**: 导入Playwright相关模块时报导入异常错误，模块无法正常加载；脚本运行过程中提示事件循环运行异常、调用不存在的属性；项目中其他第三方库与Playwright基础依赖库出现版本互相排斥冲突。

### Root Cause 根本原因

**English**: Playwright relies on fixed compatible versions of basic libraries such as pyee and typing\-extensions\. If the project environment already has old versions of these dependencies, version conflicts will be triggered, resulting in abnormal module import and script runtime failures\.

**中文**：Playwright运行依赖pyee、typing\-extensions等基础库的固定兼容版本，若项目运行环境中已存在这些依赖的老旧版本，就会触发版本冲突，进而导致模块导入异常、脚本运行报错失效。

### One\-Stop Solution 一站式解决方案

```bash
# Solution 1: Force upgrade dependencies to Playwright compatible versions 方案1：强制升级依赖至Playwright兼容版本
pip install pyee typing-extensions --upgrade

# Solution 2 (Recommended): Use virtual environment to isolate dependencies 方案2（推荐）：虚拟环境隔离依赖，彻底规避冲突
# Create virtual environment 创建虚拟环境
python -m venv venv

# Activate virtual environment 激活虚拟环境
# Windows:
venv\Scripts\activate
# Mac/Linux:
source venv/bin/activate

# Reinstall Playwright in isolated environment 隔离环境内重新安装
pip install playwright
playwright install
```

---

## 4\. Linux Server Installation: Missing System Libraries, Browser Startup Failure Linux服务器安装：缺失系统依赖库，浏览器启动失败

### Phenomenon 故障现象

**English**: Installation completed successfully on Ubuntu/Debian/CentOS servers, but the browser exits immediately after startup; headless mode operation reports missing system library errors such as libnss3, libatk, libgbm; browsers cannot render pages normally in non\-graphical interface server environments\.

**中文**：Ubuntu/Debian/CentOS服务器环境安装流程显示成功，但浏览器启动后直接闪退；无头模式运行脚本时报错，提示缺失libnss3、libatk、libgbm等系统核心库；无图形界面的服务器环境下，浏览器无法正常渲染加载页面。

### Root Cause 根本原因

**English**: The Linux server default minimal installation environment does not pre\-install graphical rendering, audio and video, network communication and other basic system dependent libraries required for normal browser operation\.

**中文**：Linux服务器默认极简安装环境，未预装浏览器正常运行所必需的图形渲染、音视频解码、网络通信等基础系统依赖组件库。

### One\-Stop Solution 一站式解决方案

#### Ubuntu / Debian System 适配Ubuntu/Debian系统

```bash
sudo apt-get update
sudo apt-get install -y libnss3 libatk1.0-0 libatk-bridge2.0-0 libcups2 \
libdrm2 libxkbcommon0 libxcomposite1 libxdamage1 libxfixes3 libxrandr2 \
libgbm1 libasound2 libpangocairo-1.0-0
```

#### CentOS / RHEL System 适配CentOS/RHEL系统

```bash
sudo yum install -y nss atk cups libdrm libxkbcommon libXcomposite \
libXdamage libXfixes libXrandr mesa-libgbm alsa-lib pango
```

---

## Standardized Installation Process 个人实测标准化安装流程

**English**: During my personal work and study practice, I encountered many repeated environment and dependency problems when configuring Playwright automation projects\. After continuous troubleshooting and practice, I have summarized a set of standardized and stable installation processes\. Following this process can effectively avoid common installation errors and environment compatibility problems\.

**中文**: 我在个人工作实操与技术学习过程中，配置Playwright自动化项目时反复遇到各类环境及依赖配置问题。经过多次问题排查与实操验证，我总结整理了这套稳定规范的标准化安装流程，按照该流程部署环境，能够有效规避常见安装报错与版本兼容类问题。

1. **Environment Check 环境检查**: Ensure Python version ≥3\.8 and network connection is normal\.

2. **Upgrade Tool 工具升级**: Update pip to the latest version\.

3. **Install Package 包体安装**: Install Playwright with domestic mirror source\.

4. **Match Browser 浏览器适配**: Must execute `playwright install` to install supporting browsers\.

5. **Environment Verification 环境验证**: Run a simple test script to confirm normal startup and access\.

1. **环境检查**：确保Python版本≥3\.8，网络连接正常稳定。

2. **工具升级**：将pip工具升级至最新稳定版本。

3. **包体安装**：使用国内镜像源安装Playwright核心包。

4. **浏览器适配**：必须执行`playwright install`安装配套浏览器。

5. **环境验证**：运行简易测试脚本，确认环境启动、访问无异常。

---

## Summary 总结

**English**: Most Playwright \+ Python environment setup issues I faced are actually easy to fix, as long as you follow a regular installation routine\. Just make sure your Python version meets the requirements, update pip on time, install the official matching browsers, use virtual environments to keep dependencies clean, and prepare necessary system libraries for Linux servers\. I wrote this dual\-language note based on my own real setup experience, hoping it can help anyone who’s just getting started avoid unnecessary troubles\.

**中文**：我实操下来发现，Playwright \+ Python 大部分环境安装问题都很好解决，只要安装时按规范一步步来就行。核心做好这几点：Python版本达标、及时升级pip、装好官方配套浏览器、用虚拟环境隔离依赖，Linux服务器提前装好系统库就够了。这份双语文档是我自己实操踩坑后的随手记录总结，希望能帮刚上手的小伙伴少走点弯路，不用反复折腾环境问题。



> （注：文档部分内容可能由 AI 生成）
