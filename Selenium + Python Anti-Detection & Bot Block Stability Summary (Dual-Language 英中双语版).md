# Selenium \+ Python Anti\-Detection \&amp; Bot Block Stability Summary \(Dual\-Language 英中双语版\)

## Preface 前言

**English**: In actual Selenium automation practice, **website anti\-bot detection and human\-machine verification interception are core causes of unstable script execution**\. The code positioning and interaction logic are correct, but the website forcibly pops up sliders, captchas or access restriction prompts\. These problems are not script logic errors, but Selenium default browser feature exposure and obvious automated operation characteristics\. This document independently summarizes the most common anti\-detection interception problems in actual combat, including real phenomena, cause analysis and practical stable solutions, for personal long\-term maintenance and peer automation reference\.

**中文**：日常实操Selenium自动化过程中，**网站反爬检测与人机验证拦截，是脚本频繁翻车、运行不稳定的核心关键**。代码定位和交互逻辑都没问题，但网站会强制弹出滑块验证、图形验证码或访问受限提示。这类问题不是代码写错，而是Selenium默认浏览器特征暴露、自动化操作痕迹太明显导致。本文独立整理实操中最高频的反爬检测拦截踩坑问题，记录真实现象、分析核心原因和稳定可用的解决方案，自用复盘备查，也供同行做自动化避坑参考。

---

## 1\. Automated Bot Detected: Captcha / Slider / Risk Interception 被网站检测为自动化：触发人机验证、滑块风控拦截

### Phenomenon 故障现象

**English**: The script can start the browser and load the page normally, but after entering core business operations, the website pops up human\-machine verification, slider drag verification or picture captcha forcibly\. Normal business operations are blocked, and use cases fail continuously\.

**中文**：脚本可以正常启动浏览器、加载页面，一旦进入核心业务操作，网站就强制弹出人机校验、滑块拖拽验证或图形验证码。正常业务流程被拦截阻断，自动化用例持续失败无法推进。

### Cause Analysis 分析原因

**English**: Selenium carries obvious automated browser default marks after startup; the `navigator\.webdriver` attribute is exposed as true, which is easily identified by website risk detection rules; browser automation switch and experimental parameters are not hidden, resulting in bot identity exposure and forced verification interception\.

**中文**：Selenium启动后自带明显的自动化浏览器标识；`navigator\.webdriver`属性暴露为true，极易被网站风控规则识别；浏览器自动化开关和实验参数未做隐藏处理，直接导致机器人身份暴露，触发强制验证拦截。

### Practical Solution 实操解决方案

**English**: Disable browser automation default detection parameters before starting the driver; hide the webdriver native attribute through JS script injection; turn off automatic extension and automation prompts to simulate real human browsing behavior and avoid risk rule identification\.

**中文**：启动驱动前关闭浏览器自动化检测默认参数；通过JS注入脚本隐藏webdriver原生特征；关闭自动化扩展与提示弹窗，模拟真人正常浏览行为，规避网站风控规则识别。

---

## Summary 总结

**English**: Most anti\-detection and risk interception problems in Selenium automation are not operation logic issues, but unhidden browser automated feature exposure\. Standardizing browser startup parameters and hiding webdriver attributes can effectively avoid most bot detection and human\-machine verification interception\.

**中文**：实操总结下来，Selenium自动化反爬风控拦截问题，都不是交互定位逻辑问题，而是浏览器自动化特征未隐藏导致。规范浏览器启动参数、屏蔽webdriver标识，就能有效规避大部分机器人检测和人机验证拦截。
