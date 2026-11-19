<div align="center">

# 虎牙TV · 端上直连版

**一个可直接在 Android TV / 电视盒子上安装的虎牙直播客户端。所有解析在设备本地完成，视频流直连虎牙 CDN，不经过任何中转服务器。**

[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Platform](https://img.shields.io/badge/Platform-Android%20TV%205.0%2B-green.svg)](https://www.android.com/tv/)
[![Language](https://img.shields.io/badge/Language-Kotlin-orange.svg)](https://kotlinlang.org/)
[![Version](https://img.shields.io/badge/Version-1.8-brightgreen.svg)](https://github.com/<你的用户名>/huya-tv/releases)

</div>

---

## 为什么做这个项目

在电视/盒子上看虎牙直播，通常要么用手机投屏（依赖手机、延迟高），要么在 NAS 上跑代理（远程看会把视频流拉回家里，吃满家宽上行）。本项目走第三条路：

- **端上全解析**：antiCode 签名、匿名登录、线路解析全部在电视/盒子本地完成；
- **视频直连 CDN**：视频字节由设备直接从虎牙 CDN 拉取，不经过 NAS、VPS 等任何中转，远程观看没有上行瓶颈；
- **点开就播**：装好 APK，打开即是「一起看」分类的频道列表，遥控器选台即播。

## 功能特性

- 📺 **分类浏览**：默认「一起看」分类（GID 2135），可通过手机扫码设置页换成任意分类 GID
- 🎯 **房间直达**：在手机设置页输入房间号，一键让电视直接播放
- 📱 **局域网网页设置**：App 内嵌 HTTP 服务（端口 16628），首页按菜单键 / 长按弹出二维码，手机扫码后浏览器设置，保存即自动刷新频道列表
- 🖥️ **横屏适配**：启动即锁定横屏，沉浸式全屏播放，专为电视 / 盒子设计
- 🎮 **遥控器友好**：方向键导航 + OK 播放，播放中可暂停 / 唤出控制栏 / 调音量
- 🔐 **官方签名**：antiCode 签名 + 移动端 UA + Referer，多条 CDN 自动回退
- ⚡ **主动提前续接**：虎牙 FLV 约每 2 分钟强制断流，App 在断流前主动换新连接并冻结最后一帧过渡，画面短暂定格即续上，而非黑屏缓冲；周期自适应
- 🧩 **双协议**：FLV（默认，稳定）/ HLS 可选

> 本项目早期曾尝试 App 内嵌本地 FLV 代理（对标 Docker 版 `flvRelay` 的时间戳续接），但真机上代理长连接仍会被系统掐断、未达无缝，已放弃并改回直连 + 主动续接方案。

## 截图

<!-- 把电视界面截图放到 docs/ 目录后，取消下面注释即可展示（替换为你自己的文件名）：
| 频道列表 | 播放界面 | 扫码设置 |
|:---:|:---:|:---:|
| ![频道列表](docs/channel-list.jpg) | ![播放界面](docs/player.jpg) | ![扫码设置](docs/settings-qr.jpg) |
-->

## 快速开始

### 安装

项目根目录已附带编译好的 `app-debug.apk`，直接安装即可：

1. 把 `app-debug.apk` 拷到 U 盘，插到电视 / 盒子上，用文件管理器打开安装；
2. 或电脑连接设备后执行：`adb install app-debug.apk`；
3. 首次安装需在系统设置里允许「未知来源应用」（部分国产电视需在「应用管理」里手动授权）。

> 兼容 Android 5.0（API 21）及以上。APK 为 debug 签名，仅供个人使用。

### 使用

首页只显示频道列表，不显示任何设置输入框。需要改设置时：

1. 在**频道列表页**按遥控器的**菜单键（或信息键）**，触屏则**长按页面空白处**；
2. 屏幕弹出**二维码**，手机扫码（或手动输入二维码下方地址，如 `http://192.168.1.23:16628`）；
3. 手机与电视需在**同一局域网**。浏览器打开的是一个设置网页，可修改：
   - 分类 GID（一起看 = 2135，可换成任意分类）
   - 频道数量、协议（FLV / HLS）、线路 CDN
   - 房间号直达播放
4. 点「保存并刷新频道列表」后，电视端**自动重新加载频道列表**，无需操作遥控器；
5. 关闭二维码：按返回键或选「关闭」。

> 设置网页由电视上的 App 直接提供（内嵌 HTTP 服务，端口 16628），不经过任何外部服务器；App 关闭后网页即不可访问。

**播放操作**：遥控器上下左右选频道，OK 播放；进入播放器自动沉浸式全屏，不显示标题栏；播放中 OK / 播放暂停键 = 暂停或继续，方向键 = 唤出控制栏，菜单 / 信息键 = 显示控制栏，返回键 = 退出，音量键由电视系统调节。直播流不支持快进快退（会给出提示）。

### 怎么找分类 GID

打开虎牙网页版分类页，URL 里的数字就是 GID：

- 一起看：`https://www.huya.com/g/seeTogether` → `2135`
- 其他分类：`https://www.huya.com/g/<数字>` 中 `<数字>` 即 GID

也可在直播间页面源码里搜 `iGid` 或 `iGameId` 找到对应分类 ID。

## 工作原理

核心是把 Docker 版 `liveurls/huya.go` 的逻辑 1:1 移植到 Kotlin，并针对端上场景重做播放器续接策略：

| Go 版（Docker） | Kotlin 版（本项目） |
|---|---|
| `getLiveList?iGid=2135` | `HuyaApi.fetchChannels()` |
| `processAntiCode`（fm 模板 `$0~$3` 替换） | `HuyaApi.processAntiCode()` |
| `currentUid` 匿名登录 | `HuyaApi.currentUid()` |
| `parseGlobalInit` + `format` | `HuyaApi.lines()` |
| CDN 回退 | `cdnFallback` + 逐线重试 |

**续接策略**（这是直连方案能否流畅的关键）：

- **不能直接等断流**：虎牙 CDN 对单条 HTTP-FLV 连接约 2 分钟主动断开。若被动等 `STATE_ENDED` 才重连，此时流已 EOF、缓冲已空，必然黑屏几秒。
- **主动提前续接**：直播 FLV 是无「下一段」的字节流，等价做法是在被强制断流**之前**主动换新连接——提前 `force` 解析出新签名 URL（此时旧流仍在播），静默 `setMediaSource` 切换；切换前抓取最后一帧冻结覆盖，新流起播后淡出，遮住空窗。缓冲同时调大（起播前 5 秒、上限 25 秒）以加快起播。
- **周期自适应**：初始 75 秒；发生被动断流说明换晚了则缩短（下限 40 秒），连续主动成功则放宽（上限 105 秒）。
- **兜底**：主动续接失败、意外 EOF、缓冲假死仍会重新签名重连，同一线路连续失败 3 次换 CDN。
- **HLS**：分片加载，本身不受 2 分钟断流影响，仍走端上解析直连（不启用主动续接）。

## 项目结构

```
huya-tv/
├── app-debug.apk              # 编译好的安装包（随仓库提供）
├── app/
│   └── src/main/
│       ├── java/com/huya/tv/
│       │   ├── HuyaApi.kt         # 签名 / 线路解析 / 分类列表
│       │   ├── PlayerActivity.kt  # ExoPlayer 播放 + 主动续接 + 遥控
│       │   ├── MainActivity.kt    # 频道列表 + 扫码入口
│       │   ├── SettingsServer.kt  # 内嵌局域网 HTTP 设置服务
│       │   └── Prefs.kt           # 本地配置持久化
│       └── res/                   # 布局 / 样式 / 字符串
├── build.gradle.kts / settings.gradle.kts
└── gradlew                       # Gradle wrapper（无需本地装 Gradle）
```

## 从源码编译

需要 JDK 17+ 与 Android SDK（platform 34、build-tools 34.0.0）：

```bash
git clone https://github.com/<你的用户名>/huya-tv.git
cd huya-tv
./gradlew assembleDebug          # Windows 用 gradlew.bat assembleDebug
# 产物：app/build/outputs/apk/debug/app-debug.apk
```

用 Android Studio 打开工程目录直接 Run 也可以。首次编译会自动下载 Gradle 8.9 与依赖；若 Gradle 官方源较慢，可自行配置镜像。

## 常见问题（FAQ）

<details>
<summary>手机打不开扫码后的设置网页？</summary>

确认手机与电视在**同一局域网**（不能一个连访客 Wi-Fi 一个连主 Wi-Fi），且路由器未开启「AP 隔离」。也可在设置页直接手动输入二维码下方显示的 IP 地址。
</details>

<details>
<summary>播放几分钟后仍有短暂卡顿？</summary>

虎牙 FLV 的强制断流无法彻底消除，本项目已把空窗压到「画面短暂定格一下」。若明显卡顿，可在设置页切换 CDN 线路，或改用 HLS 协议（分片加载，不受断流影响）。
</details>

<details>
<summary>某些线路 403 或无法播放？</summary>

虎牙线路会变动，在设置页切换 CDN（alicdn / txcdn / hycdn 等），或改用 HLS。若全部失效，可能是虎牙接口变更，需更新 `HuyaApi.kt` 后重新编译。
</details>

<details>
<summary>为什么接口一变就要重新编译？</summary>

解析逻辑全部在端上，这是「不依赖中转服务器」的固有代价——好处是完全自主、无上行瓶颈，代价是接口变更需发新版。相比之下 Docker 版可以远端热修。
</details>

## 免责声明

- 本项目仅供**个人学习研究、家庭内部自用**，请勿公开分发、商用或用于任何盈利目的。
- 项目不托管、不存储任何直播内容，所有视频流均直连虎牙官方 CDN，版权归原作者及平台所有。
- 使用者应自行遵守所在地区的法律法规及虎牙平台的服务条款。作者不对使用本项目产生的任何后果负责。

## License

[Apache License 2.0](LICENSE) © 2026 huya-tv contributors

---

<div align="center">

如果这个项目对你有帮助，欢迎点一个 ⭐ Star。

</div>
