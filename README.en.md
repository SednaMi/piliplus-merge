<div align="center">
    <img width="180" height="180" src="assets/images/logo/logo.png" alt="PiliPlus logo">
    <h1>PiliPlus Merge</h1>
</div>

<div align="center">

[中文](README.md) | English

![GitHub release (latest by date)](https://img.shields.io/github/v/release/SednaMi/piliplus-merge)
![GitHub all releases](https://img.shields.io/github/downloads/SednaMi/piliplus-merge/total)
![GitHub repo size](https://img.shields.io/github/repo-size/SednaMi/piliplus-merge)
![License](https://img.shields.io/github/license/SednaMi/piliplus-merge)

</div>

<div align="center">
    <p>A third-party Bilibili client built with Flutter · a three-repository merge</p>

<img src="assets/screenshots/main_screen.png" width="96%" alt="home" />
<br/>
</div>

<br/>

> ### ⚠️ This is a personal merge, not the official project
>
> This repository is a personal derivative of **[bggRGjQaUbCoE/PiliPlus](https://github.com/bggRGjQaUbCoE/PiliPlus)** that merges three repositories:
>
> | Source | What it provides | Commit |
> |---|---|---|
> | [bggRGjQaUbCoE/PiliPlus](https://github.com/bggRGjQaUbCoE/PiliPlus) | Main features (upstream) | `2.2.1` / `217fb54f5` |
> | [AritxOnly/Hyper-PiliPlus](https://github.com/AritxOnly/Hyper-PiliPlus) | Hyper UI (Miuix widgets, glass tab bar, shared transitions) | `0d08dd25b` |
> | [nishuodedui1145-del/PiliPlus](https://github.com/nishuodedui1145-del/PiliPlus) | BTR multi-range download acceleration | `e133cd90f` |
>
> For the official version, official documentation and official updates, go to the **[upstream repository](https://github.com/bggRGjQaUbCoE/PiliPlus)**.

<br/>

## Current version

| Item | Value |
|---|---|
| Version name | `2.2.1.0` (four parts: upstream three + this branch's revision) |
| versionCode | `20200100` |
| Package | `com.aritxonly.hyperpiliplus.btr` |
| App name | PiliPlus BTR |
| Based on upstream | PiliPlus `2.2.1` |
| Flutter | 3.47.4 (patched, not stock SDK) |

The four-part version lets this branch ship its own revisions on top of the same upstream release while still upgrading in place: the first three parts follow upstream and the revision resets to 0 on every upstream bump, then increments for our own fixes (`2.2.1.0 → 2.2.1.1 → …`).

<br/>

## Differences from upstream

### 1. Hyper UI

From [Hyper-PiliPlus](https://github.com/AritxOnly/Hyper-PiliPlus), built on `flutter_miuix`:

- **Miuix-style widgets** — switches, buttons, sliders and tab bars replaced with a HyperOS-like look
- **Glass tab bar** — sampled natively on Android and rendered with a dimmed Flutter layer; glass translucency is adjustable
- **Card-expand transitions** — video, dynamic and article cards expand into details, with fast-return and predictive-back gestures
- **Compact player controls** — common controls on one row, tuned for narrow screens, landscape and large text
- **Dynamic detail scroll restore** — restores both the body and the comment scroll positions on return
- **Native Android interaction & freeform window support** — native text selection toolbar, custom sampling notifications, HyperOS freeform window insets
- Desktop window sizing (minimum size, position correction, PiP size memory)

Each item can be toggled in `Settings → Feature settings`.

### 2. BTR multi-range acceleration

A port of the idea behind [Bilibili-thread-ripper](https://github.com/MrTangLuyao/Bilibili-thread-ripper), aimed at the **overseas** case where a cold Bilibili video or 4K stream stalls because a single CDN connection is throttled.

The player is untouched: a **local HTTP proxy (127.0.0.1)** sits in front of it, splits each request into byte ranges, **fetches them in parallel over several CDN connections**, then stitches them back in order.

```
mpv ──▶ 127.0.0.1:<random port>/media ──▶ BtrProxyServer ──▶ multi-range ──▶ upstream CDN
                     (local loopback, no crypto cost)       ├─ candidate pool: mainland / overseas groups
                                                            ├─ startup probe + per-chunk adaptive concurrency
                                                            ├─ sidx segment planning + cache reuse
                                                            ├─ slow-chunk rescue / dead-node strikes
                                                            └─ single↔multi connection switching + passthrough fallback
```

| Feature | Description |
|---|---|
| **Multi-range concurrency** | Concurrency is *derived* from measured per-connection speed (not "open them all"), with a hard in-flight cap |
| **CDN candidate pool** | Mainland / overseas groups, sticky preferred nodes, dead-node strikes, automatic removal of bad nodes |
| **Automatic CDN racing** | Background race over candidates on video entry (6 × 64 KB, 800 ms budget), results cached 5 minutes |
| **sidx segment planning** | Background prefetch of 256 KiB to parse init+sidx, then splits aligned to media segment boundaries |
| **Cross-node length consistency** | Aborts and switches route when nodes disagree on total length — never mixes bytes |
| **Fallback + auto re-takeover** | Falls back to direct passthrough after a 3.5 s grace period, then retries at 4 / 8 / 16 s |
| **No corruption on fallback** | Failed chunks are never skipped; it degrades to a sequential single-connection passthrough (skipping corrupts fMP4) |
| **Zero player changes** | All logic lives in the local proxy; turning the switch off restores native behaviour |

Code lives in `lib/services/btr_proxy/` (6 files, ~8.3k lines of Dart).

<br/>

## Download

Get an APK from **[Releases](../../releases)** and sideload it (no root needed).

Only one ABI of the same version can be installed at a time — pick by CPU:

- `app-arm64-v8a-release.apk` — **what nearly every phone wants**
- `app-armeabi-v7a-release.apk` — older 32-bit devices
- `app-x86_64-release.apk` — emulators

Because the package id carries a `.btr` suffix, it **coexists with an installed official PiliPlus**. Older builds of this same package id upgrade in place (same signing key).

> **About in-app "Check for updates"**: in this build it still points at the Hyper-PiliPlus upstream repo, not this one — so it will **not** notify you about releases here, and it may even say "already up to date". To update, watch the [Releases](../../releases) page directly, or use Watch → Custom → Releases on this repo. This pointer will be corrected in the next version.

<br/>

## Enabling BTR

1. `Mine → gear icon → Audio & video settings`
   - **BTR multi-thread acceleration** — master switch (off by default)
   - **BTR concurrency cap** — 4 / 8 / 16 / 32 / 64 (default 8)
   - **BTR node group** — auto / mainland / overseas
   - **BTR CDN auto-race** — pick the fastest node on entry (on by default)
2. Also raise **buffer size** from 4 to **32** and **buffer duration** to **60** — on jittery links buffering helps more than concurrency.
3. The `⋮ → BTR quick settings` panel in the player shows the current best node, race time and measured speed, with a "race again" action.

> Note: concurrency **cannot create international bandwidth that does not exist**. On a bad link even 4K will not be rescued — that is a physical limit.

<br/>

## Building

### Requirements

- **Flutter 3.47.4** (not stock — project patches required, see `tool/flutter-sdk.patch` and `lib/scripts/media_query.patch`)
- Android SDK (compileSdk 37 / build-tools 36.0.0), JDK 17
- `material_ui 1.11.0` (apply the 9 Android patches under `lib/scripts/material/`; the 10th, `bottom_sheet_ios_flutter_material.patch`, is iOS-only and **must not** be applied for Android)

See [`docs/flutter-environment.md`](docs/flutter-environment.md) for the patch restore flow.

### Commands

```bash
# verify SDK and patches
./scripts/flutterw --check-sdk

# dependencies
./scripts/flutterw pub get --enforce-lockfile

# build (split per ABI)
./scripts/flutterw build apk --release --split-per-abi --no-pub \
  --build-name=2.2.1.0 --build-number=20200100 \
  --android-project-arg btr
```

Version metadata is generated by `scripts/release_metadata.py` from `pubspec.yaml` + `tool/release.json`; never hand-write historical version numbers.

### Tests

```bash
python tool/make_btr_mirror.py                                     # mirror the proxy modules into a standalone harness (rerun after changing them)
./scripts/flutterw test --no-pub test/standalone/btr_bitrate_unit_test.dart   # bitrate units and threshold contracts
./scripts/flutterw test --no-pub test/standalone/btr_cdn_racer_test.dart      # CDN racing (local fake CDN, self-contained)
python3 -m unittest discover -s test/scripts -p '*_test.py'        # version metadata scripts
```

`test/standalone/btr_proxy_e2e_test.dart` needs `test/fixtures/media_url.txt` (a real, expiring CDN direct link) and is **deliberately not in the repository** — fill in a link to run it locally, but do not commit it.

<br/>

## Feature list

For the full upstream feature list (recommended videos, bangumi, live, dynamics, private messages, favourites, danmaku, subtitles, DLNA, WebDAV backup, …) see **[README-upstream.md](README-upstream.md)** — apart from the two areas above, this repository matches upstream `2.2.1` exactly.

<br/>

## License

- This repository is a derivative work of PiliPlus and **keeps upstream's GPL-3.0** (see [LICENSE](LICENSE), unchanged).
- The Hyper UI work comes from [AritxOnly/Hyper-PiliPlus](https://github.com/AritxOnly/Hyper-PiliPlus) (also GPL-3.0).
- The BTR concurrency design and default parameters reference **Bilibili-thread-ripper** ([web](https://github.com/MrTangLuyao/Bilibili-thread-ripper) / [desktop](https://github.com/MrTangLuyao/Bilibili-thread-ripper-desktop), **MIT**); this port is an independent Dart implementation and copies no JS code.

## Disclaimer (inherited from upstream)

PiliPlus is a personal project developed for educational purposes, intended only for learning and testing. Please delete it within 24 hours of downloading.
All APIs used were collected from the official website. This project does not provide any cracked content.

Credit to the original project: [guozhigq/pilipala](https://github.com/guozhigq/pilipala), [orz12/PiliPalaX](https://github.com/orz12/PiliPalaX), [bggRGjQaUbCoE/PiliPlus](https://github.com/bggRGjQaUbCoE/PiliPlus).

<br/>

## Acknowledgements

- [bilibili-API-collect](https://github.com/SocialSisterYi/bilibili-API-collect)
- [media-kit](https://github.com/media-kit/media-kit)
- [flutter_meedu_videoplayer](https://github.com/zezo357/flutter_meedu_videoplayer)
- [dio](https://pub.dev/packages/dio)
- [flutter_miuix](https://pub.dev/packages/flutter_miuix)
- [material_ui](https://pub.dev/packages/material_ui)
- And others
