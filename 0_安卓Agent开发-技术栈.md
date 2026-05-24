# 安卓Agent开发-技术栈

本文总结自我问了AI半小时的产物，编写仅1h，可能写的比较混乱还请见谅。

所有人严肃学习**AutoGLM**（

可选项很多，还需要进一步讨论，确定下来具体用哪些。

## 权限、工具（Tool）实现方面

天气等联网服务太简单，不提。

### 无障碍服务

这是目前实现通用操控最核心的接口，也是开发 Agent 的首选方案。这是 Agent 的“眼睛”和“手指”。“眼睛”可以读取屏幕上的任何文字、识别控件甚至获取布局结构；“手指”可以模拟全局的点击、滑动、长按、返回等手势，且不受应用切换影响。

### Shizuku / ADB Shell

用于执行一些系统级指令、Shell命令。

执行 input tap 模拟点击、am force-stop 杀进程、wm 修改分辨率等。相比无障碍，它无法获取界面信息（Agent 看不到屏幕），通常是配合前者使用的“执行器”。

模拟输入 (核心操作)、应用与界面管理（包括截图）、系统设置与状态、文件操作。

一般会使用 无障碍服务负责“看懂”界面（识别控件位置），再用 ADB 负责“执行”操作（高精度点击/滑动），这样既稳定又高效。

ADB Shell 需要电脑辅助，而Shizuku可以仅用手机，这是我认为现阶段安卓应用开发的场景下较好的方式。

```
云端 LLM ←→ Agent 框架 ←→ ADB/Shizuku ←→ 安卓设备
                ↓
         截图 + 控件树 → 模型理解 → 执行操作
```

## 调用其他应用的接口（应用间通信）

### Intent 隐式意图

这是安卓系统最核心的跨应用通信机制，允许你的 Agent 在不关心目标应用具体是谁的情况下，执行标准动作。

原理：Agent 构造一个包含 action（动作）和 data（数据）的 Intent，系统会帮它找到能处理这个 Intent 的应用并打开。

示例：

```kotlin
// 工具：让LLM直接打开微信扫一扫
val intent = Intent("com.tencent.mm.action.BarcodeActivity") // 微信内部Action
// 或更通用的方式：
intent.action = Intent.ACTION_VIEW
intent.data = Uri.parse("weixin://scanqrcode")
context.startActivity(intent)
```

另外：`StartActivityForResult` 有返回值/链式操作

### AIDL & Messenger

直接调用另一个应用的内部方法（而不仅仅是打开界面），就需要 AIDL（Android 接口定义语言）。这是安卓进程间通信（IPC）的核心机制，基于 Binder 驱动实现。

原理：目标应用通过 AIDL 暴露一个接口，Agent 应用绑定该服务后，可以像调用本地方法一样调用远程应用的方法，获取返回值。

这需要目标应用主动暴露服务，普通应用无法随意调用任意应用的任意方法，但许多大型应用（如支付宝、微信支付 SDK）会提供 AIDL 接口供合作方使用。

### OpenAPI调用、接入\<xx\>CLI

**我认为值得考虑的方式**

例子：飞书开放平台提供了大量API，开发者可以通过RESTful API调用接口，实现联系人管理、云文档管理等大量功能。飞书CLI提供了OpenAPI的进一步封装，方便Agent调用和获取输出。

## Agent框架方面

关于云端LLM的密钥放在哪的问题，我的考虑是：我们Demo阶段可以简单做成在安卓应用侧让用户填API Key，这样可以实现完全不需要我们写一层没用的服务端。如果（我是说如果）Vivo欣赏我们的创意的话，他们会自己处理好模型密钥安全问题。

### Kotlin/Java

1. Koog（JetBrains 官方出品）（DeepSeek推荐这个）
2. Google ADK for Java
3. Koaks
4. LangChain4j

### Python

如果考虑Agent完全在云端，安卓应用仅提供Tools：云端可以直接上Python的langchain，或者更重量级的Dify来做Agent编排。

## 现成的应用框架方面

### Open-AutoGLM

~~点击输入文本~~

我认为目前最适合我们的，严肃阅读源码

但是它需要ADB，或许我们可以改成Shizuku，将APP做成模块，这样或许可以实现仅手机端。

```
用户自然语言 → 云端/本地模型(AutoGLM-Phone-9B) → ADB → 安卓设备
```

**技术栈**

| 组件 | 实现                             |
| ---- | -------------------------------- |
| 模型 | AutoGLM-Phone-9B（视觉语言模型） |
| 通信 | ADB（Android Debug Bridge）      |
| 感知 | OCR + UI树解析                   |
| 操作 | input tap、input swipe等ADB命令  |

### DroidRun - Python开发者首选、baremobile - 极简架构

这些可以考虑使用Termux在安卓上运行Python程序。

## 总结 - 需要学什么

1. 核心基础：Kotlin/Java 与 Android 四大组件
2. 核心能力：无障碍服务 (AccessibilityService)、系统与Shell：ADB/Shizuku
3. 跨应用交互：安卓原生Intent、Binder；HTTP API调用（更通用）、CLI接入（更适合语言模型体质）
4. 本地Agent部署：Kotlin/Java Agent框架，或云端Agent，Python开发更顺畅。