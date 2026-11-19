# GitHub 仓库信息（建仓库时复制以下内容）

## Repository name

```
huya-tv
```

## Description（仓库 About 一句话描述）

```
虎牙直播 Android TV 客户端：端上签名解析、视频直连虎牙 CDN、无中转，支持一起看分类与自定义房间，遥控器友好。
```

## Topics（搜索标签，多个用逗号分隔）

```
android-tv, kotlin, exoplayer, huya, iptv, live-stream, android-tv-box, media3, flv, hls
```

## 建议的仓库设置

- **Visibility**: Public（若要公开）或 Private
- **Include**: 不要勾选 "Add a README file"（本地已有 README.md，避免冲突）；License 选 **Apache License 2.0**（本地已有 LICENSE，也可建仓库时选，GitHub 会自动生成）
- **Default branch**: `main`（推送时若本地是 `master`，可用 `git branch -M main` 重命名）

## Release 说明文案（上传 APK 时填写）

标题：
```
v1.8 · 直连 + 主动提前续接
```

正文：
```
### 本次更新
- 放弃 App 内嵌本地 FLV 代理，改回纯直连虎牙 CDN（避免代理长连接被系统掐断）
- 新增「主动提前续接」：在虎牙约 2 分钟强制断流之前换新连接，切换瞬间冻结最后一帧过渡，画面短暂定格即续上，不再黑屏缓冲
- 续接周期自适应（40~105 秒），失败时自动换 CDN 兜底
- 调大播放缓冲（起播前 5 秒 / 上限 25 秒），加快重连后起播

### 安装
下载下方 `app-debug.apk` 安装到电视 / 盒子（需允许「未知来源应用」）。兼容 Android 5.0+。

### 已知说明
- 仅供个人学习、家庭自用，请勿公开分发或商用
- 虎牙接口变更后需重新编译新版
- 若某线路不可用，在设置页（首页按菜单键扫码）切换 CDN 或改用 HLS
```
