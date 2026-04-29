# Playwright \+ Python Network, Popup \&amp; Authentication Troubleshooting Summary \(Dual\-Language 英中双语版\)

## Preface 前言

**English**: In my daily Playwright automation work, **network fluctuation, unexpected popups and repeated login authentication are common causes of random use case failures**\. The script code and element positioning are correct, but the page loads abnormally, the script gets stuck unexpectedly, or the whole test suite runs extremely slowly\. These problems are not related to element operation logic, but unreasonable network processing, unhandled dialog alerts and unreasonable login state management\. This document independently summarizes three typical problems I often encountered in actual projects, with practical phenomena, root causes and ready\-to\-use code examples for daily maintenance and quick troubleshooting\.

**中文**：我实操Playwright自动化过程中发现，**网络请求波动、页面弹窗未处理、登录态未复用，是用例随机失败、执行效率低下的常见核心原因**。代码定位都没问题，但页面偶尔空白加载异常、脚本莫名卡住挂起、整套用例执行巨慢无比。这类问题和元素操作无关，核心是网络没管控、弹窗没监听、登录状态没做持久化。本文独立整理工作中高频碰到的三类网络、弹窗与认证踩坑问题，搭配现象原因\+可直接复用的实操代码，自用排查省心，也供同行参考。

---

## 1\. Unstable Network Requests: API 500 / Timeout Causing Page Rendering Abnormal 网络请求不稳定：接口500/超时导致页面渲染异常

### Phenomenon 故障现象

**English**: The page appears blank occasionally with data loading failure; test cases fail randomly and cannot be stably reproduced\. The script logic is correct, but unstable backend interfaces and network jitter lead to abnormal page rendering\.

**中文**：自动化运行时页面偶发空白、数据加载不出来；用例随机失败、复现无规律。脚本代码逻辑没问题，只因后端接口不稳定、网络波动，导致页面渲染异常，用例直接翻车。

### Root Cause 根本原因

**English**: The business page relies heavily on third\-party or unstable backend interfaces; no network interception, waiting or Mock strategy is configured; script runs directly without waiting for core network requests to complete\.

**中文**：页面业务强依赖不稳定后端或第三方接口；脚本没做网络拦截、请求等待和接口Mock兜底策略；不等核心关键接口请求完成就直接执行页面操作。

### Practical Solution \&amp; Code Example 实操解决方案\+代码示例

**English**: Add waiting for core network requests before business operations, intercept and block unstable useless interfaces, and ensure the page core data is fully loaded before executing follow\-up logic\.

**中文**：业务操作前等待核心关键接口请求完成，拦截屏蔽无用且不稳定的异常接口，确保页面核心数据渲染完毕，再执行后续自动化操作。

```python
# Wait for core interface request stabilize 等待核心接口就绪 & 拦截异常无效接口
# Wait key business API response 等待核心业务接口返回
page.wait_for_response(lambda response: "/api/business/data" in response.url and response.status == 200)

# Block unstable third-party useless requests 拦截不稳定第三方垃圾接口，避免干扰页面
page.route("**/third-party/ad/**", lambda route: route.abort())
```

---

## 2\. Unhandled Alert / Confirm Popup Causes Script Hang 弹窗/alert/confirm未处理导致脚本卡死超时

### Phenomenon 故障现象

**English**: After the page pops up alert or confirm dialog, the script is stuck and cannot continue running, finally triggering timeout failure\. The script has no response and cannot automatically close the popup\.

**中文**：页面弹出alert、confirm原生弹窗后，脚本直接卡住不动、无法继续往下执行，最终超时报错失败。脚本不会自动处理弹窗，一直阻塞等待。

### Root Cause 根本原因

**English**: Playwright does not automatically handle native dialog events by default; the script does not listen to the `dialog` event in advance, resulting in the popup always occupying the page and blocking subsequent operations\.

**中文**：Playwright默认不会自动处理原生弹窗事件；脚本没有提前监听`dialog`弹窗回调，弹窗弹出后一直阻塞页面，导致后续所有操作无法执行。

### Practical Solution \&amp; Code Example 实操解决方案\+代码示例

**English**: Predefine a global dialog monitoring function to automatically accept or cancel all popups, prevent script blocking, and ensure the process continues normally\.

**中文**：提前全局监听弹窗事件，自动确认或取消弹窗，不让脚本阻塞卡死，保证自动化流程持续正常运行。

```python
# Global dialog popup auto-handle 全局自动处理所有alert/confirm弹窗
def handle_dialog(dialog):
    # Auto confirm all popups 自动确认弹窗，如需取消改为 dialog.dismiss()
    dialog.accept()

# Register dialog event listener 注册弹窗监听，全局生效
page.on("dialog", handle_dialog)
```

---

## 3\. Repeated Login For Every Case: Slow Execution \&amp; Fragile Stability 登录态复用问题：重复登录，运行慢且脆弱易翻车

### Phenomenon 故障现象

**English**: Every test case repeats the login process, resulting in extremely slow test suite execution; login steps are prone to random failures due to verification codes and risk control interception\.

**中文**：每条测试用例都重复走一遍登录流程，整套用例执行速度极慢；频繁登录容易触发网站验证码、风控拦截，导致登录步骤随机失败。

### Root Cause 根本原因

**English**: Not using Playwright `storageState` to save and restore login session; test cases do not isolate running states, resulting in repeated login operations every time\.

**中文**：没有利用Playwright的`storageState`缓存保存和恢复登录会话；用例之间没有做好状态隔离，每次都要重新走登录流程。

### Practical Solution \&amp; Code Example 实操解决方案\+代码示例

**English**: Save the login state to a JSON file after the first successful login; directly load the storage state for subsequent use cases to skip repeated login, speed up execution and avoid risk control verification\.

**中文**：首次登录成功后把登录态保存为JSON缓存文件；后续所有用例直接加载登录状态免登跳过登录步骤，提速又避风控。

```python
# Step1: Save login state after first login 首次登录后保存登录态
context.storage_state(path="login_auth_state.json")

# Step2: Load login state and skip login in new cases 新用例直接加载登录态，免登录
context = browser.new_context(storage_state="login_auth_state.json")
```

---

## Summary 总结

**English**: Most network, popup and authentication failures I encountered are not code bugs, but lack of network control, missing dialog monitoring and no login state reuse\. With network waiting \&amp; interception, auto dialog handling and storageState login caching, these random unstable failures can be completely solved\.

**中文**：实操下来，网络、弹窗和认证相关问题都不是代码bug，全是没管控网络请求、没监听弹窗、没复用登录态导致。做好网络等待拦截、弹窗自动处理、storageState登录缓存，就能彻底解决这类随机不稳定报错。
