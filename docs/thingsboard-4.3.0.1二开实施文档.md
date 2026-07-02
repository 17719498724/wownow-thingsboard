# ThingsBoard 4.3.0.1 二开实施文档

> 项目：wownow-max ThingsBoard 二开  
> 版本基线：ThingsBoard CE `v4.3.0.1`  
> 创建日期：2026-07-02  
> 维护规则：后续每次修改前后端、移动端、部署脚本、设备字段或打包方式时，都必须同步更新本文档。

## 1. 当前状态

后端与 Web 前端源码已经拉取完成：

```bash
D:/program/thingsboard-custom/thingsboard
```

初次拉取后仓库处于 tag 检出后的 detached HEAD 状态：

```txt
HEAD (no branch)
```

已创建二开分支，避免后续修改无法稳定维护。

```bash
cd /d/program/thingsboard-custom/thingsboard
git switch -c custom/wownow-max-4.3.0.1
```

确认分支：

```bash
git status --short --branch
```

期望看到：

```txt
## custom/wownow-max-4.3.0.1
```

当前已确认分支：

```txt
custom/wownow-max-4.3.0.1
```

## 2. 总体二开目标

本项目需要同时支持：

1. ThingsBoard 后端处理 wownow-max 设备数据。
2. ThingsBoard Web 前端展示设备监控页面和自定义部件。
3. Android 移动端 App 可安装、可连接私有 ThingsBoard 服务、可查看设备状态。
4. Web 页面或服务器提供 Android APK 下载入口。

推荐目录规划：

```txt
D:/program/thingsboard-custom/
├─ thingsboard/                  # ThingsBoard CE 4.3.0.1 后端 + Web 前端
├─ flutter_thingsboard_app/       # ThingsBoard CE 移动端 Flutter App
├─ docs/                          # 项目文档，可后续复制本文档进去
└─ release/                       # APK、部署包、镜像说明等发布产物
```

## 3. 官方源码来源

ThingsBoard CE：

```txt
https://github.com/thingsboard/thingsboard
```

当前使用版本：

```txt
v4.3.0.1
commit: 94ed115bee363e1a9fa6998daa86e7ded3d10c6f
```

移动端 CE：

```txt
https://github.com/thingsboard/flutter_thingsboard_app
```

如果后续确认使用 ThingsBoard PE，则移动端应改用：

```txt
https://github.com/thingsboard/flutter_thingsboard_pe_app
```

## 4. 后端二开范围

ThingsBoard 后端主要负责：

1. 接收 MQTT / HTTP 上报的设备遥测数据。
2. 存储 wownow-max 各部件状态。
3. 通过 Rule Chain 计算运行状态和告警。
4. 提供 Web 前端和 Android App 使用的数据接口。
5. 必要时新增自定义 REST API。

优先使用 ThingsBoard 原生能力：

```txt
Telemetry
Attributes
Rule Chain
Alarm
Dashboard
Widget
REST API
```

只有当原生能力无法满足时，再修改 Java 后端源码。

## 5. Web 前端二开范围

ThingsBoard Web 前端目录：

```txt
thingsboard/ui-ngx
```

建议二开内容：

1. 登录页品牌化。
2. 菜单裁剪，只保留项目需要的功能。
3. 新增 wownow-max 设备监控首页。
4. 增加原料仓、废料仓、CNC、UV、NFC、机械手等专用 Dashboard Widget。
5. 增加 Android App 下载入口。
6. 替换 Logo、标题、主题色、中文文案。

前期建议优先使用 Dashboard 自定义部件，不急着大改前端源码。

## 6. 设备字段规范

所有设备字段必须增加部件前缀，避免不同部件字段冲突。

推荐前缀：

```txt
raw_         原料仓
waste_       废料仓
loading_     上料机械手
nfc_read_    NFC 读取
cnc_left_    左 CNC
cnc_right_   右 CNC
uv_          UV 打印
nfc_write_   NFC 写入
unloading_   下料机械手
product_     成品仓
demolding_   成品脱模
mold_        模具仓
```

废料仓字段：

```txt
waste_runtime_status_label
waste_current_capacity
waste_maximum_capacity
waste_full_inventory
waste_link_outage
waste_link_timeout
```

示例遥测：

```json
{
  "waste_runtime_status_label": "运行",
  "waste_current_capacity": 18,
  "waste_maximum_capacity": 24,
  "waste_full_inventory": false,
  "waste_link_outage": false,
  "waste_link_timeout": false
}
```

## 7. 移动端二开方案

移动端使用官方 Flutter App 作为基线。

拉取源码：

```bash
cd /d/program/thingsboard-custom
git clone https://github.com/thingsboard/flutter_thingsboard_app.git
cd flutter_thingsboard_app
git switch -c custom/wownow-max-android
```

当前移动端源码已拉取完成：

```txt
D:/program/thingsboard-custom/flutter_thingsboard_app
```

当前移动端分支：

```txt
custom/wownow-max-android
```

项目 `.fvmrc` 指定 Flutter 版本：

```txt
3.29.0
```

如果 GitHub 网络不稳定，继续使用 Clash 代理：

```bash
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
```

移动端需要完成：

1. 修改 App 名称，例如 `Wownow Max`。
2. 修改 Android 包名，例如 `com.wownow.max`.
3. 修改 Logo、启动图、主题色。
4. 配置默认 ThingsBoard 服务地址。
5. 登录后进入自定义首页或设备列表。
6. 设备详情页展示 wownow-max 部件状态。
7. 支持告警列表和状态查看。
8. 打包 Android APK。
9. 将 APK 放到服务器供用户下载。

## 8. Flutter / Android 环境

移动端开发机需要安装：

```txt
Flutter SDK 3.29.0
Android Studio
Android SDK
JDK
Git
```

当前 Windows 环境执行 `flutter doctor` 提示：

```txt
'flutter' 不是内部或外部命令，也不是可运行的程序或批处理文件。
```

说明 Flutter 尚未安装，或 Flutter 的 `bin` 目录没有加入 Windows PATH。

推荐方式一：直接安装 Flutter 3.29.0。

1. 下载 Flutter Windows SDK 3.29.0。
2. 解压到固定目录，例如：

```txt
D:/dev/flutter
```

3. 将以下目录加入 Windows 用户环境变量 `Path`：

```txt
D:/dev/flutter/bin
```

4. 重新打开 CMD 或 PowerShell。
5. 检查：

```bash
flutter --version
flutter doctor
```

推荐方式二：使用 FVM 管理 Flutter 版本。

项目已经有 `.fvmrc`，因此可以使用 FVM 安装并锁定 Flutter 3.29.0。FVM 安装完成后，在移动端项目目录执行：

```bash
cd /d/program/thingsboard-custom/flutter_thingsboard_app
fvm install 3.29.0
fvm use 3.29.0
fvm flutter doctor
fvm flutter pub get
```

### Android Studio 代理设置

如果 Android Studio Setup Wizard 弹出：

```txt
Android Studio First Run
Unable to access Android SDK add-on list
```

说明 Android Studio 没有走本机代理。当前电脑使用 Clash Verge 时，Android Studio 需要单独设置 HTTP Proxy。

在 `HTTP Proxy` 窗口中选择：

```txt
Manual proxy configuration
HTTP
Host name: 127.0.0.1
Port number: 7890
No proxy for: localhost,127.0.0.1
```

如果 Clash Verge 的端口不是 `7890`，以 Clash Verge 设置里的 `Mixed Port` 或 `HTTP Port` 为准。

配置后点击 `Check Connection`，测试地址可以填写：

```txt
https://dl.google.com/android/repository/addons_list-5.xml
```

检查通过后点击 `OK`，回到 Setup Wizard 继续安装 Android SDK。

如果代理仍失败，可以先点击 `Cancel` 关闭弹窗，继续接受 License；安装完成后再进入：

```txt
File / Settings / Appearance & Behavior / System Settings / HTTP Proxy
```

重新配置同样的代理。

### Android SDK 下载损坏处理

如果安装 SDK 时出现：

```txt
java.io.EOFException: Unexpected end of ZLIB input stream
Warning: An error occurred while preparing SDK package Android SDK Platform 36.1
```

说明 SDK 压缩包没有下载完整，或本地半成品缓存已损坏。当前 SDK 目录：

```txt
C:/Users/4393/AppData/Local/Android/Sdk
```

已发现相关半成品目录：

```txt
C:/Users/4393/AppData/Local/Android/Sdk/.temp/PackageOperation01
C:/Users/4393/AppData/Local/Android/Sdk/platforms/android-36.1
```

处理步骤：

1. 关闭 Android Studio。
2. 删除损坏的临时下载目录和半成品平台目录。
3. 重新打开 Android Studio，确认 HTTP Proxy 配置正确。
4. 重新安装 Android SDK Platform。

PowerShell 删除命令：

```powershell
Remove-Item -LiteralPath "$env:LOCALAPPDATA\Android\Sdk\.temp\PackageOperation01" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item -LiteralPath "$env:LOCALAPPDATA\Android\Sdk\platforms\android-36.1" -Recurse -Force -ErrorAction SilentlyContinue
```

如果 `36.1` 继续失败，可以先安装稳定平台版本 `Android 35`，Flutter 构建通常不要求必须使用 Android 36.1。

### 当前环境巡检结果

巡检日期：2026-07-02

源码状态：

```txt
ThingsBoard CE:
路径: D:/program/thingsboard-custom/thingsboard
分支: custom/wownow-max-4.3.0.1
基线版本: v4.3.0.1
当前提交: 9978bff
远程仓库: https://github.com/17719498724/wownow-thingsboard.git
远程分支: custom/wownow-max-4.3.0.1

Flutter ThingsBoard App:
路径: D:/program/thingsboard-custom/flutter_thingsboard_app
分支: custom/wownow-max-android
提交: 63bc934
远程仓库: https://github.com/17719498724/wownow-thingsboard-app.git
远程分支: custom/wownow-max-android
```

软件状态：

```txt
Flutter: 已下载，路径 D:/dev/flutter，版本 3.29.0
Flutter PATH: 未配置，直接执行 flutter 会失败
Android Studio: 已安装，路径 D:/soft/AndroidStudio，版本 2026.1.1
Android SDK: 已安装，路径 C:/Users/4393/AppData/Local/Android/Sdk
Android SDK Platform: android-36.1 已存在
Android Build Tools: 36.1.0、37.0.0 已存在
Android cmdline-tools: 未安装
Android licenses: 未确认
JDK 17: 已安装，路径 C:/Program Files/Java/jdk-17.0.3.1
JAVA_HOME: 当前仍指向 C:/Program Files/Java/jdk1.8.0_221，需要调整到 JDK 17
Maven: 已安装，版本 3.9.16
Node.js: 已安装，版本 18.20.8
npm: 已安装，版本 10.8.2
```

`flutter doctor -v` 结果摘要：

```txt
Flutter 3.29.0 可用，但 D:/dev/flutter/bin 未加入 PATH。
Android toolchain 缺少 cmdline-tools。
Android license 状态未知，需要执行 flutter doctor --android-licenses。
网络检查 github.com 和 maven.google.com 超时，后续下载依赖时需要使用代理。
Visual Studio 未安装，仅影响 Windows 桌面应用开发，不影响 Android APK。
```

当前需要完成的环境动作：

1. 将 `D:/dev/flutter/bin` 加入 Windows `Path`。
2. 将 `JAVA_HOME` 改为 `C:/Program Files/Java/jdk-17.0.3.1`。
3. 设置 `ANDROID_HOME` 和 `ANDROID_SDK_ROOT` 为 `C:/Users/4393/AppData/Local/Android/Sdk`。
4. 在 Android Studio 的 SDK Manager 安装 `Android SDK Command-line Tools (latest)`。
5. 重新打开命令行，执行 `flutter doctor --android-licenses`。
6. 在移动端项目执行 `flutter pub get`。

### 代码仓库推送状态

PC / 后端 / Web 仓库：

```txt
本地路径: D:/program/thingsboard-custom/thingsboard
远程仓库: https://github.com/17719498724/wownow-thingsboard.git
推送分支: custom/wownow-max-4.3.0.1
远程提交: 9978bff3b50d9a5838e9c0fa31bea2f5f5972254
```

说明：由于本地 ThingsBoard 是从 tag `v4.3.0.1` 浅克隆得到，直接推送到空 GitHub 仓库时发生缺少历史对象错误。因此 PC 仓库采用“源码快照初始提交”的方式推送，完整包含当前源码内容和 `docs/thingsboard-4.3.0.1二开实施文档.md`，但不携带官方 ThingsBoard 历史提交。官方源仍保留为 `upstream`。

移动端 App 仓库：

```txt
本地路径: D:/program/thingsboard-custom/flutter_thingsboard_app
远程仓库: https://github.com/17719498724/wownow-thingsboard-app.git
推送分支: custom/wownow-max-android
远程提交: 63bc93420c87188352a27e5d87fce04a222e9e42
```

移动端 App 仓库保留官方 Flutter App 历史，官方源保留为 `upstream`。

检查环境：

```bash
flutter doctor
```

安装依赖：

```bash
cd /d/program/thingsboard-custom/flutter_thingsboard_app
flutter pub get
```

调试运行：

```bash
flutter run
```

打包 APK：

```bash
flutter build apk --release
```

打包产物通常位于：

```txt
build/app/outputs/flutter-apk/app-release.apk
```

如果要上传应用市场，通常需要打包 AAB：

```bash
flutter build appbundle --release
```

## 9. Android 下载方案

推荐提供两种下载方式。

方式一：Nginx 静态下载

```txt
https://your-domain.com/download/wownow-max.apk
```

服务器目录示例：

```txt
/var/www/download/wownow-max.apk
```

方式二：ThingsBoard Web 前端增加下载入口

在 Web 前端增加“移动端下载”菜单或按钮，点击后跳转：

```txt
/download/wownow-max.apk
```

也可以展示二维码，扫码下载 APK。

## 10. 后端与移动端数据对接方式

移动端不要直接连接设备 MQTT。推荐方式：

```txt
设备
  ↓ MQTT / HTTP
ThingsBoard 后端
  ↓ REST API / Dashboard API / Telemetry API
Android App
```

移动端读取内容：

1. 设备列表。
2. 设备最新遥测。
3. 设备告警。
4. Dashboard 或自定义页面数据。

如果 App 只做监控，可以优先复用 ThingsBoard 官方 App 的 Dashboard 展示能力。

如果 App 要做 wownow-max 专用页面，则新增 Flutter 页面，从 ThingsBoard API 拉取对应字段。

## 11. 构建后端与 Web

进入 ThingsBoard 源码目录：

```bash
cd /d/program/thingsboard-custom/thingsboard
```

完整构建通常使用 Maven：

```bash
mvn clean install -DskipTests
```

如果只开发前端，需要进入：

```bash
cd ui-ngx
```

具体前端命令以后根据 `package.json` 和项目实际依赖确认后补充。

## 12. 建议实施顺序

第一阶段：源码与环境

1. ThingsBoard 创建二开分支。
2. 拉取 Flutter 移动端源码。
3. 安装 JDK、Maven、Node、Flutter、Android Studio。
4. 后端和 Web 能本地启动。
5. Android App 能本地运行。

第二阶段：设备字段与 Dashboard

1. 整理所有部件字段。
2. 配置 ThingsBoard 设备和遥测。
3. 制作原料仓、废料仓等自定义 Widget。
4. 配置 Dashboard 总览页面。

第三阶段：移动端

1. App 品牌化。
2. 配置默认服务器地址。
3. 调整首页和设备详情页。
4. 打包测试 APK。
5. Web 增加 APK 下载入口。

第四阶段：发布

1. 构建后端包或 Docker 镜像。
2. 构建 Web 前端。
3. 构建 Android APK。
4. 部署服务器。
5. 形成版本号和发布记录。

## 13. 变更记录

每次二开修改都必须在这里追加记录。

格式：

```txt
日期：
修改人：
模块：
改动内容：
影响范围：
验证方式：
相关文件：
```

### 2026-07-02

修改人：Codex / 4393  
模块：项目初始化  
改动内容：

1. 确认 ThingsBoard CE `v4.3.0.1` 已拉取到本地。
2. 确认当前仓库初始处于 detached HEAD。
3. 已创建 `custom/wownow-max-4.3.0.1` 二开分支。
4. 制定后端、Web 前端、Android 移动端二开路线。
5. 明确移动端基于 `thingsboard/flutter_thingsboard_app` 进行二开。
6. 明确后续每次修改必须同步更新本文档。

影响范围：项目二开流程、移动端开发流程、发布流程。  
验证方式：已执行 `git status --short --branch`，确认当前为 `custom/wownow-max-4.3.0.1`。  
相关文件：`outputs/thingsboard-4.3.0.1二开实施文档.md`

### 2026-07-02

修改人：Codex / 4393  
模块：Git 分支  
改动内容：

1. 在 `D:/program/thingsboard-custom/thingsboard` 创建二开分支。
2. 当前分支为 `custom/wownow-max-4.3.0.1`。

影响范围：ThingsBoard 后端与 Web 前端源码维护。  
验证方式：

```bash
git -C 'D:\program\thingsboard-custom\thingsboard' status --short --branch
```

结果：

```txt
## custom/wownow-max-4.3.0.1
```

相关文件：无源码文件变更。

### 2026-07-02

修改人：Codex / 4393  
模块：Android 移动端  
改动内容：

1. 已拉取 `thingsboard/flutter_thingsboard_app` 移动端源码。
2. 已创建移动端二开分支 `custom/wownow-max-android`。
3. 确认项目 `.fvmrc` 指定 Flutter `3.29.0`。
4. 确认当前 Windows 环境尚不能执行 `flutter doctor`，需要安装 Flutter 或配置 PATH。

影响范围：Android 移动端二开、Flutter 环境搭建、后续 APK 打包。  
验证方式：

```bash
git -C 'D:\program\thingsboard-custom\flutter_thingsboard_app' status --short --branch
```

结果：

```txt
## custom/wownow-max-android
```

相关文件：

```txt
D:/program/thingsboard-custom/flutter_thingsboard_app/.fvmrc
```

### 2026-07-02

修改人：Codex / 4393  
模块：Android Studio 环境  
改动内容：

1. 记录 Android Studio Setup Wizard 首次启动时无法访问 Android SDK add-on list 的处理方式。
2. 明确 Android Studio 需要单独配置 HTTP Proxy。
3. 推荐 Clash Verge 代理配置为 `127.0.0.1:7890`，实际端口以 Clash Verge 设置为准。

影响范围：Android SDK 安装、Flutter Android 构建环境。  
验证方式：待在 Android Studio 中执行 `Check Connection` 并完成 SDK 安装。  
相关文件：无源码文件变更。

### 2026-07-02

修改人：Codex / 4393  
模块：Android SDK 安装  
改动内容：

1. 记录 `Unexpected end of ZLIB input stream` 错误处理方式。
2. 确认 SDK 路径为 `C:/Users/4393/AppData/Local/Android/Sdk`。
3. 确认损坏下载相关目录包括 `.temp/PackageOperation01` 和 `platforms/android-36.1`。
4. 建议清理损坏目录后重新下载，必要时先安装 Android 35。

影响范围：Android SDK 安装、Flutter Android 构建环境。  
验证方式：待清理后重新安装 SDK，并执行 `flutter doctor`。  
相关文件：无源码文件变更。

### 2026-07-02

修改人：Codex / 4393  
模块：环境巡检  
改动内容：

1. 检查 ThingsBoard CE 源码路径、分支和版本。
2. 检查 Flutter 移动端源码路径和分支。
3. 确认 Flutter 3.29.0 已下载到 `D:/dev/flutter`，但未加入 PATH。
4. 确认 Android SDK 已安装，但缺少 `cmdline-tools`。
5. 确认当前 `JAVA_HOME` 指向 JDK 8，需要改到 JDK 17。
6. 确认 Node.js、npm、Maven 已安装。

影响范围：本地开发环境、移动端运行和打包、后端构建。  
验证方式：执行 Git、Flutter Doctor、Java、Maven、Node、Android SDK 目录检查。  
相关文件：无源码文件变更。

### 2026-07-02

修改人：Codex / 4393  
模块：代码仓库推送  
改动内容：

1. 将移动端 App 仓库 `origin` 设置为 `https://github.com/17719498724/wownow-thingsboard-app.git`。
2. 将移动端分支 `custom/wownow-max-android` 推送到远程仓库。
3. 将 PC / 后端 / Web 仓库 `origin` 设置为 `https://github.com/17719498724/wownow-thingsboard.git`。
4. PC 仓库因浅克隆历史对象缺失，改用源码快照初始提交方式推送。
5. 将 PC 分支 `custom/wownow-max-4.3.0.1` 推送到远程仓库。
6. PC 仓库保留官方 ThingsBoard 源为 `upstream`，移动端仓库保留官方 Flutter App 源为 `upstream`。

影响范围：远程代码托管、后续协作开发、二开分支管理。  
验证方式：

```bash
git ls-remote https://github.com/17719498724/wownow-thingsboard.git refs/heads/custom/wownow-max-4.3.0.1
git ls-remote https://github.com/17719498724/wownow-thingsboard-app.git refs/heads/custom/wownow-max-android
```

验证结果：

```txt
9978bff3b50d9a5838e9c0fa31bea2f5f5972254 refs/heads/custom/wownow-max-4.3.0.1
63bc93420c87188352a27e5d87fce04a222e9e42 refs/heads/custom/wownow-max-android
```

相关文件：

```txt
D:/program/thingsboard-custom/thingsboard/docs/thingsboard-4.3.0.1二开实施文档.md
C:/Users/4393/Documents/Codex/2026-06-30/can/outputs/thingsboard-4.3.0.1二开实施文档.md
```
