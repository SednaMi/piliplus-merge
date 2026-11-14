<div align="center">
    <img width="180" height="180" src="assets/images/logo/logo.png" alt="PiliPlus logo">
    <h1>PiliPlus Merge</h1>
</div>

<div align="center">

中文 | [English](README.en.md)

![GitHub release (latest by date)](https://img.shields.io/github/v/release/SednaMi/piliplus-merge)
![GitHub all releases](https://img.shields.io/github/downloads/SednaMi/piliplus-merge/total)
![GitHub repo size](https://img.shields.io/github/repo-size/SednaMi/piliplus-merge)
![License](https://img.shields.io/github/license/SednaMi/piliplus-merge)

</div>

<div align="center">
    <p>使用 Flutter 开发的 B 站第三方客户端 · 三仓合并版</p>

<img src="assets/screenshots/main_screen.png" width="96%" alt="home" />
<br/>
</div>

<br/>

> ### ⚠️ 这是个人合并仓库，不是官方版本
>
> 本仓库是 **[bggRGjQaUbCoE/PiliPlus](https://github.com/bggRGjQaUbCoE/PiliPlus)** 的个人衍生版，把三个仓库合到一起：
>
> | 来源 | 提供了什么 | 对应提交 |
> |---|---|---|
> | [bggRGjQaUbCoE/PiliPlus](https://github.com/bggRGjQaUbCoE/PiliPlus) | 主体功能（上游官方） | `2.2.1` / `217fb54f5` |
> | [AritxOnly/Hyper-PiliPlus](https://github.com/AritxOnly/Hyper-PiliPlus) | Hyper UI 定制（Miuix 组件、柔光玻璃底栏、共享转场） | `0d08dd25b` |
> | [nishuodedui1145-del/PiliPlus](https://github.com/nishuodedui1145-del/PiliPlus) | BTR 多 Range 并发加速 | `e133cd90f` |
>
> 想要官方版本、官方功能说明与官方更新，请前往 **[上游仓库](https://github.com/bggRGjQaUbCoE/PiliPlus)**。

<br/>

## 当前版本

| 项目 | 值 |
|---|---|
| 版本名 | `2.2.1.0`（四段式：上游三段 + 本分支修订号） |
| versionCode | `20200100` |
| 包名 | `com.aritxonly.hyperpiliplus.btr` |
| 应用名 | PiliPlus BTR |
| 基于上游 | PiliPlus `2.2.1` |
| Flutter | 3.47.4（带项目补丁，非原版 SDK） |

四段式版本号是为了让「同一上游版本下的本分支多次修订」能覆盖安装又互不混淆：上游升级时前三位跟上游、第四位重置为 0；本分支自己的修复递增第四位（`2.2.1.0 → 2.2.1.1 → …`）。

<br/>

## 和上游的差别

### 1. Hyper UI 定制

来自 [Hyper-PiliPlus](https://github.com/AritxOnly/Hyper-PiliPlus)，基于 `flutter_miuix`：

- **Miuix 风格组件** —— 开关、按钮、滑块、标签栏等一整套替换，风格更接近 HyperOS
- **柔光玻璃底栏** —— Android 原生侧采样 + Flutter 侧压暗渲染，可调「柔光玻璃通透程度」
- **卡片展开转场** —— 视频 / 动态 / 专栏卡片展开进入详情，带快速返回与预测返回手势
- **紧凑播放器控件** —— 单行排列常用控件，适配窄屏、横屏和大字体
- **动态详情滚动恢复** —— 返回时恢复正文和评论两层滚动位置
- **Android 原生交互与自由窗口适配** —— 原生文本选择操作栏、自定义采样通知、HyperOS 自由窗口边距
- 桌面端窗口尺寸计算（最小尺寸 / 位置修正 / PiP 尺寸记忆）

以上都能在 `设置 → 功能设置` 里逐项开关。

### 2. BTR 多 Range 并发加速

来自 [Bilibili-thread-ripper](https://github.com/MrTangLuyao/Bilibili-thread-ripper) 的原理移植，解决**海外**看 B 站冷门视频 / 4K 时「一直转圈、加载卡死」的问题。

不碰播放器：在 Flutter 层起一个**本地 HTTP 代理（127.0.0.1）**，播放器照常请求，代理把一条请求按字节区间切成多块，**并行从多条 CDN 连接拉取**，按序拼好再写回播放器。

```
mpv ──▶ 127.0.0.1:<随机端口>/media ──▶ BtrProxyServer ──▶ 多 Range 并发 ──▶ 上游 CDN
                     （本地回环，无加密开销）              ├─ 候选池：大陆 / 海外分组
                                                          ├─ 启动探测 + 逐块自适应并发
                                                          ├─ sidx 分段索引规划 + 缓存复用
                                                          ├─ 慢块补救 / 死节点拉黑
                                                          └─ 单连接↔多连接自动切换 + 降级透传
```

| 特性 | 说明 |
|---|---|
| **多 Range 并发** | 按实测每连接速度**反推**并发数（不是"先开满"），带在途连接硬上限 |
| **CDN 候选池** | 大陆 / 海外分组，粘性优选节点，死节点拉黑，坏节点自动剔除 |
| **CDN 自动竞速** | 进视频后后台竞速候选节点（6 × 64KB，预算 800ms），结果缓存 5 分钟 |
| **sidx 分段索引规划** | 起播后台预取 256 KiB 解析 init+sidx，按媒体分段边界对齐切分 |
| **跨节点长度一致性校验** | 不同节点返回的文件总长度不一致即终止换路线，绝不混拼 |
| **降级直连 + 自动重接管** | 加速不可行时 3.5 s 宽限后降级直连，再按 4 / 8 / 16 s 自动重接管 |
| **降级不花屏** | 分块失败绝不跳块，降级为单连接顺序透传（fMP4 跳块会花屏） |
| **播放器零改动** | 逻辑全在本地代理里；关掉开关即回到原生行为 |

代码在 `lib/services/btr_proxy/`（6 个文件，约 8.3k 行 Dart）。

<br/>

## 下载

从 **[Releases](../../releases)** 下载 APK，直接侧载安装（无需 root）。

同一版本下不同 ABI 的包只能装一个，按手机 CPU 选：

- `app-arm64-v8a-release.apk` —— **绝大多数手机选这个**
- `app-armeabi-v7a-release.apk` —— 老旧 32 位设备
- `app-x86_64-release.apk` —— 模拟器

因为包名带 `.btr` 后缀，**可以和你已安装的官方 PiliPlus 共存**。同包名历史版本可直接覆盖升级（签名一致）。

<br/>

## BTR 怎么开

1. `我的 → 右上齿轮 → 音视频设置`
   - **BTR 多线程加速** —— 总开关（默认关，需要手动打开）
   - **BTR 并发上限** —— 4 / 8 / 16 / 32 / 64（默认 8）
   - **BTR 节点分组** —— 自动 / 大陆 / 海外
   - **BTR CDN 自动竞速** —— 进视频后自动选最快节点（默认开）
2. 建议把 **缓冲大小** 从 4 调到 **32**、**缓冲时长** 调到 **60** —— 线路抖动时缓冲比并发更能救体感。
3. 播放页右上 `⋮ → BTR 快设` 可查看当前最优节点、竞速时间与实测速度，可「立即重新竞速」。

> 注意：并发**不能创造不存在的国际带宽**。差线路上 4K 也救不回来，这是物理限制。

<br/>

## 自己编译

### 环境

- **Flutter 3.47.4**（不是原版，需要打项目补丁，见 `tool/flutter-sdk.patch` 与 `lib/scripts/media_query.patch`）
- Android SDK（compileSdk 37 / build-tools 36.0.0）、JDK 17
- `material_ui 1.11.0`（需要打 `lib/scripts/material/` 下的 9 个 Android 补丁；第 10 个 `bottom_sheet_ios_flutter_material.patch` 只用于 iOS，**不要**在 Android 构建里打）

补丁恢复流程见 [`docs/flutter-environment.md`](docs/flutter-environment.md)。

### 命令

```bash
# 校验 SDK 与补丁
./scripts/flutterw --check-sdk

# 依赖
./scripts/flutterw pub get --enforce-lockfile

# 出包（分 ABI）
./scripts/flutterw build apk --release --split-per-abi --no-pub \
  --build-name=2.2.1.0 --build-number=20200100 \
  --android-project-arg btr
```

版本元数据由 `scripts/release_metadata.py` 从 `pubspec.yaml` + `tool/release.json` 生成，不要手填历史版本号。

### 测试

```bash
python tool/make_btr_mirror.py                                     # 把代理模块镜像成独立测试台（改了模块必须重跑）
./scripts/flutterw test --no-pub test/standalone/btr_bitrate_unit_test.dart   # 码率单位与阈值契约
./scripts/flutterw test --no-pub test/standalone/btr_cdn_racer_test.dart      # CDN 竞速（本地假 CDN，自包含）
python3 -m unittest discover -s test/scripts -p '*_test.py'        # 版本元数据脚本
```

`test/standalone/btr_proxy_e2e_test.dart` 需要 `test/fixtures/media_url.txt`（真实 CDN 时效直链），**故意不在仓库里**——把链接填进去后可在本地跑，但别提交。

<br/>

## 功能清单

上游的完整功能清单（推荐视频、番剧、直播、动态、私信、收藏、弹幕、字幕、DLNA、WebDAV 备份……）请直接看 **[README-upstream.md](README-upstream.md)** —— 本仓库除上面两块之外与上游 `2.2.1` 完全一致。

<br/>

## 许可

- 本仓库是 PiliPlus 的衍生作品，**整体沿用上游的 GPL-3.0**（见 [LICENSE](LICENSE)，未改动）。
- Hyper UI 定制来自 [AritxOnly/Hyper-PiliPlus](https://github.com/AritxOnly/Hyper-PiliPlus)（同为 GPL-3.0）。
- BTR 并发下载的原理与默认参数参考 **Bilibili-thread-ripper**（[网页版](https://github.com/MrTangLuyao/Bilibili-thread-ripper) / [桌面版](https://github.com/MrTangLuyao/Bilibili-thread-ripper-desktop)，**MIT**）；本移植为 Dart 独立实现，未复制其 JS 代码。

## 声明（沿用上游）

此项目是个人为了兴趣而开发，仅用于学习和测试，请于下载后 24 小时内删除。
所用 API 皆从官方网站收集，不提供任何破解内容。

致敬原作者：[guozhigq/pilipala](https://github.com/guozhigq/pilipala)、[orz12/PiliPalaX](https://github.com/orz12/PiliPalaX)、[bggRGjQaUbCoE/PiliPlus](https://github.com/bggRGjQaUbCoE/PiliPlus)。

<br/>

## 致谢

- [bilibili-API-collect](https://github.com/SocialSisterYi/bilibili-API-collect)
- [media-kit](https://github.com/media-kit/media-kit)
- [flutter_meedu_videoplayer](https://github.com/zezo357/flutter_meedu_videoplayer)
- [dio](https://pub.dev/packages/dio)
- [flutter_miuix](https://pub.dev/packages/flutter_miuix)
- [material_ui](https://pub.dev/packages/material_ui)
- 等等
