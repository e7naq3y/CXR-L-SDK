# CXR-L SDK 开发文档

> 版本：v1.0.1  
> 来源：https://custom.rokid.com/prod/rokid_web/84feb39f8ef141b0ad0326f902ab881f/pc/cn/index.html  
> 整理时间：2026-04-26

CXR-L SDK 是面向手机与 Rokid 眼镜协同的开发工具包。本文件汇总了官方在线开发文档中的所有章节，便于离线阅读与本地存档。
这一版除了功能形式发生转变外（请求手机端rokid hi app）授权方式也发生了改变，原来需要开发者在后台手动添加sn码授权文件请求sdk的crx-m方式变成了用户直接在rokid hi app中点击同意授权就可以了，不需要任何其他的操作

## 目录

1. [简介](#简介)
2. [SDK 集成](#sdk-集成)
3. [快速开始](#快速开始)
4. [授权与Token获取](#授权与token获取)
5. [连接与会话](#连接与会话)
6. [自定义View](#自定义view)
7. [自定义应用控制](#自定义应用控制)
8. [音频](#音频)
9. [拍照](#拍照)
10. [自定义指令](#自定义指令)
11. [历史版本](#历史版本)


---

# CXR-L SDK 简介


CXR-L SDK 是面向手机与 Rokid 眼镜协同的开发工具包，帮助开发者在手机侧快速通过与Rokid AI APP 建立与眼镜侧应用的联动能力。


其核心目标是：


- 建立手机与眼镜的稳定连接通道；
- 在不同业务会话中复用连接能力；
- 支持自定义视图、音频、拍照、自定义指令等典型场景；
- 降低双端交互功能的落地成本。



## 架构定位


CXR-L SDK 主要运行在手机端，常见工作流为：


1. 通过 Rokid AI App 完成授权并拿到通信 token；
2. 使用 `CXRLink` 建立会话连接；
3. 按场景类型配置 session（如 `CUSTOMAPP`、`CUSTOMVIEW`）；
4. 调用对应能力接口并通过回调接收状态和结果。



## 典型能力


- **连接与会话管理**：连接状态回调、蓝牙状态回调、会话配置。
- **自定义 View**：打开/更新/关闭眼镜侧自定义界面，支持图标资源下发。
- **音频能力**：开启/停止音频流，接收 PCM 数据并本地保存。
- **拍照能力**：触发眼镜拍照并接收图片字节流。
- **自定义指令**：手机与眼镜应用间传输自定义 `Caps` 消息。



## 示例工程参考


[手机端SDK 使用Sample](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.1/image/samples/cxrlsample101.zip) 已提供 CXR-L SDK 的完整调用示例，推荐优先参考以下模块：


- `activities/main`：安装检查、授权、入口跳转；
- `activities/customAppType`：`CUSTOMAPP` 会话与应用控制（[眼镜端Sample](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.1/image/samples/sSDKSampleforCXR.zip)）；
- `activities/customViewType`：`CUSTOMVIEW` 会话与自定义视图；
- `activities/audio`：音频流采集与文件处理；
- `activities/photo`：拍照调用与结果展示；
- `activities/customCMD`：自定义指令收发。[眼镜端Sample](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.1/image/samples/sSDKSampleforCXR.zip)



## 文档阅读地图


建议按以下顺序阅读：


1. `SDK集成`：完成仓库、依赖与权限配置；
2. `授权与Token获取`：完成授权并拿到 `token`；
3. `功能开发/连接与会话`：建立 `connect(token)` 与会话；
4. `功能开发/自定义View` ~ `功能开发/自定义指令`：按能力模块接入。



## 功能使用前置条件


在示例工程中，各功能模块存在明确前置关系：


- **音频流（Audio）**：必须先启动 `CustomView` 或 `CustomAPP` 任一会话；
- **拍照（Photo）**：必须先启动 `CustomView` 或 `CustomAPP` 任一会话；
- **自定义指令（CustomCMD）**：必须先启动 `CustomAPP` 会话，`CustomView` 下不可用。



## 示例工程主流程


![sdk-01-01](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.1/image/cxrL-doc-01-01.jpg)



## 状态机与能力可用性


![sdk-01-02](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.1/image/cxrL-doc-01-02.jpg)



## 能力矩阵



| 会话状态 | Audio | Photo | CustomCMD |
| --- | --- | --- | --- |
| Idle（未启动会话） | 否 | 否 | 否 |
| CustomViewReady | 是 | 是 | 否 |
| CustomAppReady | 是 | 是 | 是 |


---

# SDK 集成



## 1. 前置条件


- Android Studio + Android项目；
- 设备已安装 Rokid AI App；
- 授权获取流程见 `功能开发/授权与Token获取`。



## 2. 配置 Maven 仓库


在 `settings.gradle.kts` 的 `dependencyResolutionManagement.repositories` 中增加 Rokid Maven 仓库：



```kotlin
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        maven { url = uri("https://maven.aliyun.com/repository/google") }
        maven { url = uri("https://maven.rokid.com/repository/maven-public/") }
        google()
        mavenCentral()
    }
}

```



## 3. 引入依赖


在应用模块 `build.gradle.kts` 中添加 CXR-L SDK 依赖（版本号按实际发布版本替换）：



```kotlin
dependencies {
    implementation("com.rokid.cxr:client-l:1.0.1")

```



> 建议：`minSdk` 不低于 28，示例工程使用更高版本配置。



## 4. AndroidManifest 基础配置


根据业务能力声明网络、文件访问等权限。以下为示例项目中已使用的最小集合之一：



```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.MANAGE_MEDIA" />
<uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE" />
<uses-permission
    android:name="android.permission.READ_EXTERNAL_STORAGE"
    android:maxSdkVersion="32" />

```


如果涉及蓝牙、Wi-Fi P2P 等能力，请按目标能力补充对应权限。



## 5. 章节分工说明


- 授权与 token 获取详见 `功能开发/授权与Token获取`；
- 会话初始化已合并到 `功能开发/连接与会话`。



## 6. 下一步


- 若尚未完成授权，先阅读 `功能开发/授权与Token获取`；
- 若已拿到 `token`，继续阅读 `功能开发/连接与会话`。


---

# 快速开始


本章节用于提供 Sample 下载入口、Sample 使用引导和最小跑通流程。



## 1. Sample 下载


- **手机端 Sample（源码）**：[CXRLSample](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.1/image/samples/cxrlsample101.zip)；
- **眼镜端 Sample（源码）**：[sSDKSampleforCXR](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.1/image/samples/sSDKSampleforCXR.zip)。



## 2. 如何使用 Sample


1. 按 `SDK集成` 完成仓库、依赖与权限配置；
2. 按 `功能开发/授权与Token获取` 完成授权并获取 `token`；
3. 按 `功能开发/连接与会话` 建立 `CUSTOMVIEW` 或 `CUSTOMAPP` 会话；
4. 进入对应入口页面验证能力：
  - `CustomViewTypeActivity`：`customViewOpen / customViewUpdate / customViewClose`；
  - `CustomAppTypeActivity`：应用安装/启动/停止/卸载；
  - `AudioUsageActivity`：`startAudioStream / stopAudioStream`；
  - `PhotoUsageActivity`：`takePhoto(width, height, quality)`；
  - `CustomCmdActivity`：`sendCustomCmd` 与 `setCXRCustomCmdCbk`（仅 `CUSTOMAPP`）。



## 3. 最小流程


1. 打开 Sample 并安装到测试手机；
2. 启动应用，确认已安装 Rokid AI App；
3. 在首页完成授权，拿到 `token`；
4. 选择 `CustomView` 或 `CustomAPP` 建立连接；
5. 进入一个能力页面触发接口并观察回调结果。


---

# 授权与 Token 获取



## 1. 场景说明


在使用 CXR-L 任意能力前，需要先完成 Rokid AI App 授权并获取通信 `token`。该 `token` 是后续 `connect(token)` 建链的必要输入。



## 2. 使用前提


- 设备已安装 Rokid AI App；
- 手机端已集成授权依赖（`AuthorizationHelper`）；
- 已在页面中预留授权结果回传处理（`onActivityResult`）；
- 建议先完成应用安装检查，再触发授权流程。



## 3. 核心接口说明



### 3.1 AuthorizationHelper.requestAuthorization(activity, requestCode)


- **功能**：发起授权请求并跳转授权页；
- **参数**：
  - `activity`：当前页面实例；
  - `requestCode`：请求码，建议使用常量统一管理；
- **返回值**：
  - `Unit`。



### 3.2 AuthorizationHelper.parseAuthorizationResult(resultCode, data)


- **功能**：解析授权返回结果并提取 token；
- **参数**：
  - `resultCode`：`onActivityResult` 返回值；
  - `data`：`onActivityResult` 返回的 `Intent`；
- **返回值**：
  - 授权结果对象（包含 `success`、`token`、`msg` 等字段，具体以 SDK 实际定义为准）。



## 4. 回调与结果处理


建议在 `onActivityResult` 中统一处理授权结果：


1. 校验 `requestCode` 是否匹配；
2. 调用 `parseAuthorizationResult(...)`；
3. 成功时缓存 `token`；
4. 失败时提示原因并保留重试入口。



## 5. 调用示例



```kotlin
// 发起授权
AuthorizationHelper.INSTANCE.requestAuthorization(activity, AUTH_REQUEST_CODE)

// 结果处理
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)
    if (requestCode != AUTH_REQUEST_CODE) return

    val result = AuthorizationHelper.INSTANCE.parseAuthorizationResult(resultCode, data)
    if (result.success) {
        token = result.token.orEmpty()
    } else {
        errorMsg = result.msg ?: "授权失败"
    }
}

```



## 6. 功能前置与关系约束


- `token` 是 `connect(token)` 的硬前置；
- 未获取有效 `token` 时，不应开放会话建立和能力入口；
- 建议将 `token` 作为页面间透传参数，或统一保存在状态容器中；
- `Audio`、`Photo`、`CustomView`、`CustomAPP`、`CustomCMD` 均依赖该流程的成功结果。



## 7. 模块流程图


![sdk-4bf0090a-01](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.1/image/cxrL-doc-4bf0090a-01.png)



## 8. 授权状态机


![sdk-4bf0090a-02](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.1/image/cxrL-doc-4bf0090a-02.png)



## 9. 相关文档


- `SDK集成`：授权前的依赖与权限配置；
- `功能开发/连接与会话`：拿到 `token` 后的会话初始化与连接建立。


---

# 连接与会话管理



## 1. 目标


建立手机与眼镜的可用链路，并在业务页面复用同一 `CXRLink` 实例。



## 2. 推荐流程


1. 获取授权 token；
2. 创建 `CXRLink(context)`；
3. 调用 `configCXRSession(...)` 配置会话；
4. 注册 `setCXRLinkCbk(...)`；
5. 调用 `connect(token)` 建立连接；
6. 将 `CXRLink` 缓存在 `Application` 供子页面复用。



## 3. 会话类型


- **CUSTOMAPP**：用于眼镜端应用安装/启动/停止等协同场景；
- **CUSTOMVIEW**：用于自定义视图渲染与更新场景。



## 4. 核心接口说明



### 4.1 CXRLink(context: Context)


- **功能**：创建 SDK 连接实例。
- **参数**：
  - `context: Context`：建议使用页面或应用上下文。
- **返回值**：
  - `CXRLink` 实例。



### 4.2 configCXRSession(session: CxrDefs.CXRSession)


- **功能**：配置会话类型和会话目标。
- **参数**：
  - `session: CxrDefs.CXRSession`
    - `type`：`CUSTOMAPP` 或 `CUSTOMVIEW`；
    - `packageName`：仅 `CUSTOMAPP` 需要，表示眼镜端目标应用包名。
- **返回值**：
  - `Unit`。



### 4.3 setCXRLinkCbk(cbk: ICXRLinkCbk)


- **功能**：注册链路状态回调。
- **参数**：
  - `cbk: ICXRLinkCbk`：连接状态监听器。
- **返回值**：
  - `Unit`。



### 4.4 connect(token: String)


- **功能**：使用授权 token 发起连接。
- **参数**：
  - `token: String`：授权成功后拿到的通信令牌。
- **返回值**：
  - `Boolean`（是否成功发起连接请求；最终连通状态以回调为准）。



## 5. 关键回调


- `onCXRLConnected(Boolean)`：CXR 服务连接状态；
- `onGlassBtConnected(Boolean)`：蓝牙连接状态；
- 可按业务选择 `onGlassAiAssistStart/Stop`。


建议仅在上述两个状态均满足时进入“可操作”状态。



## 6. 调用示例



```kotlin
val cxrLink = CXRLink(context).apply {
    configCXRSession(
        CxrDefs.CXRSession(
            CxrDefs.CXRSessionType.CUSTOMAPP,
            "com.rokid.cxrswithcxrl"
        )
    )
    setCXRLinkCbk(object : ICXRLinkCbk {
        override fun onCXRLConnected(connected: Boolean) {
            // CXR 服务连接结果
        }

        override fun onGlassBtConnected(connected: Boolean) {
            // 蓝牙连接结果
        }

        override fun onGlassAiAssistStart() {}
        override fun onGlassAiAssistStop() {}
    })
}

val startResult: Boolean = cxrLink.connect(token)

```



### 6.1 CUSTOMVIEW 初始化模板



```kotlin
val cxrLink = CXRLink(context).apply {
    configCXRSession(CxrDefs.CXRSession(CxrDefs.CXRSessionType.CUSTOMVIEW))
    setCXRLinkCbk(linkCallback)
}
cxrLink.connect(token)

```



### 6.2 CUSTOMAPP 初始化模板



```kotlin
val cxrLink = CXRLink(context).apply {
    configCXRSession(
        CxrDefs.CXRSession(
            CxrDefs.CXRSessionType.CUSTOMAPP,
            "com.xxx.your.glass.app"
        )
    )
    setCXRLinkCbk(linkCallback)
}
cxrLink.connect(token)

```



## 7. 能力可用前置规则


- `Audio` 与 `Photo`：前置为 `CUSTOMVIEW` 或 `CUSTOMAPP` 任一会话连接成功；
- `CustomCMD`：前置为 `CUSTOMAPP` 会话连接成功；
- 当处于未连接（Idle）或连接失败状态时，不应开放上述能力入口；
- 推荐统一以会话状态管理入口显隐与可点击状态。



## 8. 会话建立主流程


![sdk-9292ab69-01](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.1/image/cxrL-doc-9292ab69-01.png)



## 9. 会话状态机


![sdk-9292ab69-02](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.1/image/cxrL-doc-9292ab69-02.png)



## 10. 相关文档


- `功能开发/授权与Token获取`：会话连接前的授权与 `token` 获取；
- `自定义View`、`音频`、`拍照`、`自定义指令`、`自定义应用控制`：具体能力调用。


---

# 自定义 View



## 1. 场景说明


通过 `CUSTOMVIEW` 会话在眼镜端展示自定义界面，并支持动态更新。



## 2. 初始化步骤


1. 配置 `CxrDefs.CXRSessionType.CUSTOMVIEW`；
2. 注册连接回调 `setCXRLinkCbk`；
3. 注册自定义 View 回调 `setCXRCustomViewCbk`；
4. 连接成功后下发 icon 资源（可选但推荐）。



## 3. 核心接口说明



### 3.1 setCXRCustomViewCbk(cbk: ICustomViewCbk)


- **功能**：注册自定义视图生命周期回调。
- **参数**：
  - `cbk: ICustomViewCbk`：视图回调监听器。
- **返回值**：
  - `Unit`。



### 3.2 customViewSetIcons(iconsJson: String)


- **功能**：下发图标资源，供 `ImageView.name` 引用。
- **调用时机**：仅在存在 icon 资源需要上传时调用；若当前页面不依赖 icon，可跳过该接口。
- **参数**：
  - `iconsJson: String`：icon 列表 JSON 字符串，包含 `name` 和 base64 `data`。
- **返回值**：
  - `Unit`（结果通过 `onCustomViewIconsSent` / `onCustomViewError` 反馈）。



### 3.3 customViewOpen(viewJson: String)


- **功能**：首次打开完整视图树。
- **参数**：
  - `viewJson: String`：完整 UI 结构 JSON（`type/props/children`）。
- **返回值**：
  - `Unit`（结果通过 `onCustomViewOpened` / `onCustomViewError` 反馈）。



### 3.4 customViewUpdate(updateJson: String)


- **功能**：增量更新视图节点属性。
- **参数**：
  - `updateJson: String`：更新协议 JSON（通常为 `id + props`）。
- **返回值**：
  - `Unit`（结果通过 `onCustomViewUpdated` / `onCustomViewError` 反馈）。



### 3.5 customViewClose()


- **功能**：关闭当前自定义视图。
- **参数**：无。
- **返回值**：
  - `Unit`（结果通过 `onCustomViewClosed` 回调反馈）。



## 4. 回调说明


- `onCustomViewOpened`：标记已打开，可开始交互；
- `onCustomViewUpdated`：记录更新成功；
- `onCustomViewClosed`：回收页面状态；
- `onCustomViewError(code, msg)`：打印错误码并回退 UI 状态。
- `onCustomViewIconsSent`：图标资源发送成功。



## 5. 调用示例



```kotlin
cxrLink.setCXRCustomViewCbk(object : ICustomViewCbk {
    override fun onCustomViewOpened() {}
    override fun onCustomViewUpdated() {}
    override fun onCustomViewClosed() {}
    override fun onCustomViewIconsSent() {}
    override fun onCustomViewError(code: Int, msg: String?) {}
})

val iconsJson = """
[
  {"name":"icon1","data":"<base64-data>"},
  {"name":"icon2","data":"<base64-data>"}
]
""".trimIndent()
cxrLink.customViewSetIcons(iconsJson)

val openJson = """
{"type":"LinearLayout","props":{"id":"root"},"children":[]}
""".trimIndent()
cxrLink.customViewOpen(openJson)

val updateJson = """
{"updateList":[{"id":"textView","props":{"text":"Hello Rokid"}}]}
""".trimIndent()
cxrLink.customViewUpdate(updateJson)

cxrLink.customViewClose()

```



## 6. 数据协议建议


- `open` 阶段发送完整结构（`type + props + children`）；
- `update` 阶段仅发送变化项（`id + props`）降低带宽占用；
- 图片建议使用 base64 + 资源名映射方式统一管理。



## 7. 功能前置与关系约束


- 使用本章节能力前，必须先完成授权并建立 `CUSTOMVIEW` 会话连接；
- `CustomView` 会话成功后，可进入 `Audio` 和 `Photo` 模块；
- `CustomCMD` 仅在 `CUSTOMAPP` 会话下可用，`CUSTOMVIEW` 下不可用。



## 8. 模块流程图


![sdk-5f1e4cde-01](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.1/image/cxrL-doc-5f1e4cde-01.png)



## 9. 状态机（CustomView）


![sdk-5f1e4cde-02](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.1/image/cxrL-doc-5f1e4cde-02.png)



## 10. 相关文档


- `连接与会话`：`CUSTOMVIEW` 会话初始化与连接前置；
- `音频`、`拍照`：在 `CustomView` 会话下复用连接调用能力；
- `自定义指令`：`CustomCMD` 的会话限制说明（仅 `CUSTOMAPP`）。


---

# 自定义应用控制（CUSTOMAPP）



## 1. 场景说明


在 `CUSTOMAPP` 会话下，手机侧可控制眼镜端应用的安装、启动、停止、卸载与安装状态查询。



## 2. 回调注册



### 2.1 appIsInstalled(cbk: IGlassAppCbk)


- **功能**：查询目标应用是否已安装。
- **参数**：
  - `cbk: IGlassAppCbk`：应用控制回调。
- **返回值**：
  - `Unit`（结果通过 `onQueryAppResult(installed)` 反馈）。



## 3. 核心接口说明



### 3.1 appUploadAndInstall(apkPath: String, cbk: IGlassAppCbk)


- **功能**：上传 APK 并安装到眼镜端。
- **参数**：
  - `apkPath: String`：本地 APK 文件绝对路径；
  - `cbk: IGlassAppCbk`：安装结果回调。
- **返回值**：
  - `Unit`（结果通过 `onInstallAppResult(success)` 反馈）。



### 3.2 appStart(target: String, cbk: IGlassAppCbk)


- **功能**：启动眼镜端目标页面。
- **参数**：
  - `target: String`：完整入口字符串，通常为 `包名 + Activity 路径`；
  - `cbk: IGlassAppCbk`：启动结果回调。
- **返回值**：
  - `Unit`（结果通过 `onOpenAppResult(success)` 反馈）。



### 3.3 appStop(cbk: IGlassAppCbk)


- **功能**：停止当前眼镜端应用。
- **参数**：
  - `cbk: IGlassAppCbk`：停止结果回调。
- **返回值**：
  - `Unit`（结果通过 `onStopAppResult(success)` 反馈）。



### 3.4 appUninstall(cbk: IGlassAppCbk)


- **功能**：卸载眼镜端应用。
- **参数**：
  - `cbk: IGlassAppCbk`：卸载结果回调。
- **返回值**：
  - `Unit`（结果通过 `onUnInstallAppResult(success)` 反馈）。



## 4. 回调说明（IGlassAppCbk）


- `onInstallAppResult(success)`：安装完成结果；
- `onUnInstallAppResult(success)`：卸载完成结果；
- `onOpenAppResult(success)`：打开应用结果；
- `onStopAppResult(success)`：停止应用结果；
- `onGlassAppResume(resumed)`：应用前台状态变化；
- `onQueryAppResult(installed)`：安装状态查询结果。



## 5. 调用示例



```kotlin
val appCallback = object : IGlassAppCbk {
    override fun onInstallAppResult(success: Boolean) {}
    override fun onUnInstallAppResult(success: Boolean) {}
    override fun onOpenAppResult(success: Boolean) {}
    override fun onStopAppResult(success: Boolean) {}
    override fun onGlassAppResume(resumed: Boolean) {}
    override fun onQueryAppResult(installed: Boolean) {}
}

cxrLink.appIsInstalled(appCallback)
cxrLink.appUploadAndInstall("D:/DCIM/Rokid/cxrL.apk", appCallback)
cxrLink.appStart("com.rokid.cxrswithcxrl.activities.main.MainActivity", appCallback)
cxrLink.appStop(appCallback)
cxrLink.appUninstall(appCallback)

```



## 6. 功能前置与关系约束


- 自定义应用控制能力必须在 `CUSTOMAPP` 会话下使用；
- `CUSTOMVIEW` 会话下不支持安装/启动/停止/卸载应用；
- 建议流程为：先查询安装状态，再执行安装或启动；
- `CustomCMD` 依赖 `CUSTOMAPP`，通常与本模块配合使用；
- `Audio` 与 `Photo` 可在 `CUSTOMAPP` 下使用，但不依赖应用控制动作本身。



## 7. 模块流程图


![sdk-b5f3f609-01](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.1/image/cxrL-doc-b5f3f609-01.png)



## 9. 相关文档


- `连接与会话`：`CUSTOMAPP` 会话初始化与连接前置；
- `自定义指令`：依赖 `CUSTOMAPP` 会话的指令收发能力；
- `音频`、`拍照`：在同一会话下可复用的媒体能力。



## 8. 应用控制状态机


![sdk-b5f3f609-02](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.1/image/cxrL-doc-b5f3f609-02.png)


---

# 音频能力



## 1. 场景说明


手机侧开启眼镜音频流监听，接收回调字节并落盘，可用于识别、存档或调试。



## 2. 初始化


1. 复用已建立的 `CXRLink`；
2. 注册 `setCXRAudioCbk(...)`；
3. 在可连接状态下调用 `startAudioStream(...)`。



## 3. 核心接口说明



### 3.1 setCXRAudioCbk(cbk: IAudioStreamCbk)


- **功能**：注册音频流回调。
- **参数**：
  - `cbk: IAudioStreamCbk`：用于接收音频分片和状态变化。
- **返回值**：
  - `Unit`。



### 3.2 startAudioStream(codeType: Int): Boolean


- **功能**：开启音频流。
- **参数**：
  - `codeType: Int`：音频编码类型，示例使用 `1`（实际以 SDK 定义为准）。
- **返回值**：
  - `Boolean`：是否成功发起开启请求；真实状态以回调 `onAudioStreamStateChanged` 为准。



### 3.3 stopAudioStream()


- **功能**：关闭音频流。
- **参数**：无。
- **返回值**：
  - `Unit`（状态通过 `onAudioStreamStateChanged(false)` 反馈）。



## 4. 回调说明


- `onAudioReceived(data, offset, length)`：接收音频分片；
- `onAudioError(errorCode, errorInfo)`：音频异常；
- `onAudioStreamStateChanged(started)`：状态变化。



## 5. 调用示例



```kotlin
cxrLink.setCXRAudioCbk(object : IAudioStreamCbk {
    override fun onAudioReceived(data: ByteArray?, offset: Int, length: Int) {
        // data 可能为 null，offset/length 需要做安全校验
    }

    override fun onAudioError(errorCode: Int, errorInfo: String?) {
        // 记录错误并更新 UI 状态
    }

    override fun onAudioStreamStateChanged(started: Boolean) {
        // started=true 代表音频流已开启
    }
})

val startOk: Boolean = cxrLink.startAudioStream(1)
// ... 业务处理
cxrLink.stopAudioStream()

```



## 6. 实践建议


- 对 `offset/length` 做边界保护，避免越界写入；
- 录制中实时写 PCM，结束时统一转 WAV；
- 在页面退出时主动执行 `stopAudioStream()`；



## 7. 功能前置与关系约束


- 音频能力不能独立启动，必须先建立会话连接；
- 可用前置为：`CUSTOMVIEW` 或 `CUSTOMAPP` 任一会话已成功；
- 若当前为未连接状态（Idle），应禁止调用 `startAudioStream(...)`；
- `CustomCMD` 仅在 `CUSTOMAPP` 下可用，与音频能力互不依赖。



## 8. 模块流程图


![sdk-c3f4fa0f-01](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.1/image/cxrL-doc-c3f4fa0f-01.png)



## 9. 音频状态机


![sdk-c3f4fa0f-02](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.1/image/cxrL-doc-c3f4fa0f-02.png)



## 10. 相关文档


- `连接与会话`：音频能力的会话前置条件；
- `自定义View`、`自定义应用控制`：分别对应 `CUSTOMVIEW` 与 `CUSTOMAPP` 入口场景；
- `拍照`：同类媒体能力调用模式对照。


---

# 拍照能力



## 1. 场景说明


手机侧触发眼镜拍照并接收图片字节流，适用于记录、识别、上传等业务。



## 2. 初始化


1. 复用全局 `CXRLink`；
2. 注册 `setCXRImageCbk(IImageStreamCbk)`；
3. 在连接就绪后调用拍照接口。



## 3. 核心接口说明



### 3.1 setCXRImageCbk(cbk: IImageStreamCbk)


- **功能**：注册拍照结果回调。
- **参数**：
  - `cbk: IImageStreamCbk`：接收拍照成功/失败通知。
- **返回值**：
  - `Unit`。



### 3.2 takePhoto(width: Int, height: Int, quality: Int)


- **功能**：触发眼镜拍照。
- **参数**：
  - `width: Int`：目标图片宽度，如 `1024`；
  - `height: Int`：目标图片高度，如 `768`；
  - `quality: Int`：压缩质量，通常 `0~100`。
- **返回值**：
  - `Unit`（结果通过 `onImageReceived` / `onImageError` 反馈）。



## 4. 调用示例



```kotlin
cxrLink.setCXRImageCbk(object : IImageStreamCbk {
    override fun onImageReceived(data: ByteArray?) {
        // data 为图片字节流，可直接 decode 成 Bitmap
    }

    override fun onImageError(code: Int, msg: String?) {
        // 拍照失败处理
    }
})

cxrLink.takePhoto(1024, 768, 80)

```



## 5. 回调说明


- `onImageReceived(data)`：返回图片字节；
- `onImageError(code, msg)`：拍照失败。



## 6. 实践建议


- 在 `onImageReceived` 中立即解码并更新 UI；
- 失败时清空上一次结果，避免误导用户；
- 拍照前确保当前连接与蓝牙状态正常。



## 7. 功能前置与关系约束


- 拍照能力不能独立启动，必须先建立会话连接；
- 可用前置为：`CUSTOMVIEW` 或 `CUSTOMAPP` 任一会话已成功；
- 若当前为未连接状态（Idle），应禁止调用 `takePhoto(...)`；
- `CustomCMD` 仅在 `CUSTOMAPP` 下可用，与拍照能力互不依赖。



## 8. 模块流程图


![sdk-cf4b960f-01](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.1/image/cxrL-doc-cf4b960f-01.png)



## 9. 拍照状态机


![sdk-cf4b960f-02](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.1/image/cxrL-doc-cf4b960f-02.png)



## 10. 相关文档


- `连接与会话`：拍照能力的会话前置条件；
- `自定义View`、`自定义应用控制`：分别对应 `CUSTOMVIEW` 与 `CUSTOMAPP` 入口场景；
- `音频`：同类媒体能力调用模式对照。


---

# 自定义指令



## 1. 场景说明


通过 `Caps` 协议在手机与眼镜应用之间传递业务消息，构建双端自定义通信。



## 2. 使用前提


- 必须在 `CUSTOMAPP` 会话中使用；
- 眼镜端应用需要实现对应 key 的接收与回传逻辑；
- 手机端需先注册 custom command 回调。



## 3. 核心接口说明



### 3.1 setCXRCustomCmdCbk(cbk: ICustomCmdCbk)


- **功能**：注册自定义指令接收回调。
- **参数**：
  - `cbk: ICustomCmdCbk`：接收眼镜端返回的消息。
- **返回值**：
  - `Unit`。



### 3.2 sendCustomCmd(channel: String, payload: ByteArray)


- **功能**：向眼镜端应用发送自定义指令。
- **参数**：
  - `channel: String`：业务通道名，例如 `"rk_custom_client"`；
  - `payload: ByteArray`：协议数据，示例中由 `Caps().serialize()` 生成。
- **返回值**：
  - `Unit`（发送结果与响应内容通过 `onCustomCmdResult` 处理）。



## 4. 调用示例



```kotlin
cxrLink.setCXRCustomCmdCbk(object : ICustomCmdCbk {
    override fun onCustomCmdResult(key: String?, payload: ByteArray?) {
        if (key != "rk_custom_key") return
        val caps = payload?.let { Caps.fromBytes(it) } ?: return
        // TODO 解析 caps 内容
    }
})

cxrLink.sendCustomCmd("rk_custom_client", Caps().apply {
    write("rk_custom_key")
    write("from client click times = ${count++}")
}.serialize())

```



## 5. 回调处理


在 `onCustomCmdResult(key, payload)` 中：


1. 按 key 过滤目标消息；
2. 解析 `Caps.fromBytes(payload)`；
3. 将结构化字段转换为可读文本用于展示或业务处理。



## 6. 实践建议


- 约定稳定的 key 与字段顺序，避免协议漂移；
- 对二进制字段使用 base64 做日志输出；
- 非目标 key 直接忽略，降低误触发风险。



## 7. 功能前置与关系约束


- 自定义指令不能独立启动，必须先建立会话连接；
- 严格前置为：`CUSTOMAPP` 会话已成功；
- 在 `CUSTOMVIEW` 会话或 Idle 状态下，均应禁止调用 `sendCustomCmd(...)`；
- `Audio` 与 `Photo` 可在 `CUSTOMVIEW` 或 `CUSTOMAPP` 下使用，但不构成 `CustomCMD` 的前置。



## 8. 模块流程图


![sdk-9c8fa43c-01](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.1/image/cxrL-doc-9c8fa43c-01.png)



## 9. 自定义指令状态机


![sdk-9c8fa43c-02](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.1/image/cxrL-doc-9c8fa43c-02.png)



## 10. 相关文档


- [眼镜端APP Sample](https://rokid-ota.oss-cn-hangzhou.aliyuncs.com/toB/Document/CXR-L/v1.0.1/image/samples/sSDKSampleforCXR.zip)
- `眼镜端开发指南`:如何构建与`CUSTOMAPP`会话进行自定义指令的交互；
- `连接与会话`：`CUSTOMAPP` 会话建立与可用性前置；
- `自定义应用控制`：眼镜端应用安装/启动与指令链路联动；
- `自定义View`：`CUSTOMVIEW` 场景限制说明（不支持 `CustomCMD`）。


---

# 版本历史



## v1.0.1


- 初版SDK
- 支持获取Rokid AI APP 的授权
- 支持选择创建自定义View 场景或自定义眼镜端应用场景
- 支持获取眼镜端音频
- 支持通过眼镜进行拍照
- 支持在自定义眼镜端应用场景下与眼镜端应用进行自定义指令交互
