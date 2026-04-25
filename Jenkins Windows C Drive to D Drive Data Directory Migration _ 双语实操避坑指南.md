# Jenkins Windows C Drive to D Drive Data Directory Migration \| 双语实操避坑指南
## 📌 Overview / 概述
**English**
By default, Jenkins stores all core data including plugins, task jobs and system configurations in the C drive directory: `C:\\ProgramData\\Jenkins\\\.jenkins`\. Long\-term use will continuously occupy C drive storage space and cause system disk shortage\. This document records all invalid migration attempts, real error causes, and the final stable and effective migration solution\. The whole process does not require reinstalling Jenkins and will not lose any configuration data\.
**中文**
Jenkins 默认会将插件、构建任务、系统所有配置等核心数据全部存放在C盘目录：`C:\\ProgramData\\Jenkins\\\.jenkins`。长期使用会持续占用C盘存储空间，导致系统盘空间不足。本文记录了迁移过程中所有无效尝试、真实报错原因，以及最终稳定可用的迁移方案，全程无需重装Jenkins，不丢失任何配置数据。

## 🎯 Migration Goal / 迁移目标
**English**
Migrate the entire Jenkins home data directory from C drive to D drive, retain all existing plugins, jobs and custom configurations, and ensure normal Jenkins startup and operation\.
**中文**
将Jenkins完整数据主目录从C盘迁移至D盘，保留原有所有插件、构建任务、自定义配置，保证Jenkins正常启动运行。

## ❌ Failed Attempts \&amp; Root Causes / 失败方案及根本原因
### 1\. Modify System Environment Variable JENKINS\_HOME / 修改系统环境变量
**English**
Set the system environment variable `JENKINS\_HOME=D:\\jenkins\_files\\\.jenkins` and restart the Jenkins service, but Jenkins still loads data from the original C drive path\.
**Cause:**The Jenkins Windows service runs under the LocalSystem system account\. Most Jenkins versions will ignore manually configured system environment variables and forcibly use the default built\-in C drive home directory path\.
**中文**
配置系统环境变量 `JENKINS\_HOME=D:\\jenkins\_files\\\.jenkins` 并重启Jenkins服务后，Jenkins依旧读取原C盘路径数据。
**原因：**Jenkins Windows服务以LocalSystem系统账户运行，大部分版本会忽略手动配置的系统环境变量，强制读取默认内置的C盘主目录路径。

### 2\. Modify Windows Registry Startup Parameters / 修改注册表添加启动参数
**English**
Edited the registry path `HKEY\_LOCAL\_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\Jenkins`, added the `\-\-workDir` parameter to the ImagePath entry, causing the Jenkins service to fail to start and report error 1053\.
**Cause:**The default `jenkins\.exe` on Windows is only a **Windows Service Wrapper \(WinSW\)**, not the Jenkins core program\. This service wrapper **does not support Jenkins runtime parameters such as \-\-workDir**\. Adding unsupported parameters will directly cause the service startup to fail\.
**中文**
编辑注册表路径 `HKEY\_LOCAL\_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\Jenkins`，在ImagePath中添加 `\-\-workDir` 启动参数，导致Jenkins服务无法启动，报1053错误。
**原因：**Windows安装后的默认 `jenkins\.exe` 只是**Windows服务包装器（WinSW）**，并非Jenkins核心程序。该服务包装器**不支持 \-\-workDir 这类Jenkins运行参数**，强行添加不兼容参数直接导致服务启动失败。

### 3\. Run jenkins\.exe with \-\-workDir / 直接用jenkins\.exe带参数启动
**English**
Executed command `jenkins\.exe \-\-workDir=\&\#34;D:\\jenkins\_files\\\.jenkins\&\#34;`, got error `Unknown command: \-\-workDir`\.
**Cause:**The jenkins\.exe service wrapper only supports basic service commands such as install, start and stop, and does not recognize any Jenkins working directory configuration parameters\.
**中文**
执行命令 `jenkins\.exe \-\-workDir=\&\#34;D:\\jenkins\_files\\\.jenkins\&\#34;`，报错 `Unknown command: \-\-workDir`。
**原因：**jenkins\.exe服务包装器仅支持install、start、stop等基础服务管理命令，不识别任何Jenkins工作目录配置参数。

### 4\. Start Jenkins\.war with \-\-workDir via Java / Java直接启动war带工作目录参数
**English**
Tried to start Jenkins\.war directly with the \-\-workDir parameter, the command could not be parsed and failed to run\.
**Cause:**Windows CMD is very sensitive to double quotes and spaces in file paths, which easily leads to parameter parsing exceptions and startup failures\.
**中文**
尝试直接用java启动Jenkins\.war并携带\-\-workDir参数，命令无法正常解析，运行失败。
**原因：**Windows CMD对路径中的引号、空格格式要求严格，极易出现参数解析异常，导致启动失败。

### 5\. Default Java Version Mismatch / 默认Java版本不匹配
**English**
Java 21 was installed manually, but CMD defaults to Java 17 when starting Jenkins, triggering a version incompatibility startup error\.
**Cause:**Java 17 has a higher priority in the system PATH environment variable, while the new Jenkins version requires Java 21 or above to run normally\.
**中文**
系统已手动安装Java21，但CMD启动Jenkins时默认调用Java17，触发版本不兼容启动报错。
**原因：**系统PATH环境变量中Java17优先级更高，而新版Jenkins强制要求Java21及以上版本才能正常运行。

## ✅ Final Stable Working Solution / 最终稳定实操方案
**English**
Use a batch script to start Jenkins directly, specify the correct Java 21 version and temporary environment variable to set the working directory\. No registry modification, no Windows service dependency, zero risk and stable operation\.
**中文**
采用批处理脚本直接启动Jenkins，指定Java21运行版本\+临时环境变量设置工作目录。无需修改注册表、不依赖Windows服务，零风险且运行稳定。
### Step 1: Copy Jenkins Data to D Drive / 迁移数据目录至D盘
**English**
1. Stop Jenkins service via `services\.msc`\.
2. Copy the entire folder `C:\\ProgramData\\Jenkins\\\.jenkins` to `D:\\jenkins\_files\\\.jenkins` \(copy only, do not cut to prevent data loss\)\.
**中文**
1. 打开`services\.msc`，停止Jenkins服务。
2. 将`C:\\ProgramData\\Jenkins\\\.jenkins`完整文件夹复制到`D:\\jenkins\_files\\\.jenkins`（仅复制不剪切，防止数据丢失）。

### Step 2: Create Jenkins Startup Batch Script / 编写一键启动脚本
**English**
Create `start\_jenkins\.bat` with the following code:
**中文**
新建`start\_jenkins\.bat`，写入以下脚本：
```Plain Text
@echo off
cd /d D:\installpath\jenkins2.555.1
set JENKINS_HOME=D:\jenkins_files\.jenkins
"D:\installpath\JDK21\bin\java.exe" -jar Jenkins.war --httpPort=8085
pause
```

### Step 3: Set Windows Boot Auto\-Start / 设置开机自启
**English**
1. Press `Win \+ R`
2. Input `shell:startup`
3. Put `start\_jenkins\.bat` into the startup folder\.
Jenkins will start automatically after Windows boots up\.
**中文**
1. 按下`Win \+ R`
2. 输入`shell:startup`
3. 将`start\_jenkins\.bat`放入开机启动文件夹。
后续开机自动运行Jenkins，无需手动启动。

### Step 4: Verify Migration Success / 验证迁移生效
**English**
1. Run the batch script until `Jenkins is fully up and running` appears\.
2. Access Jenkins page \&gt; Manage Jenkins \&gt; System Information\.
3. Confirm JENKINS\_HOME points to D drive path\.
4. Check plugins and jobs to ensure all data is complete\.
**中文**
1. 运行脚本，看到`Jenkins is fully up and running`即为启动成功。
2. 访问Jenkins页面 \&gt; 管理Jenkins \&gt; 系统信息。
3. 确认JENKINS\_HOME已指向D盘路径。
4. 检查插件和构建任务，确认数据完整无丢失。

## 📝 Solution Summary / 方案总结
**English**
This solution does not rely on Windows services, registry configuration, or system environment variables\. It uses a temporary environment variable in a batch script to specify the Jenkins working directory and strictly uses Java 21 to start the war package\. It avoids all compatibility and startup failure problems, realizing stable and permanent migration of Jenkins from C drive to D drive on Windows\.
**中文**
该方案不依赖Windows服务、注册表配置及系统环境变量，通过批处理临时环境变量指定Jenkins工作目录，严格使用Java21启动war包，规避所有兼容问题和启动报错，实现Windows环境下Jenkins从C盘到D盘稳定永久迁移。

