# HyperCeiler Fork（v5 定制版）

基于 [ReChronoRain/HyperCeiler](https://github.com/ReChronoRain/HyperCeiler) 的定制编译版本，
针对 **HyperOS 3.0.5.0 / Android 15** 进行了若干本地化修改。

> ⚠️ 本仓库遵循上游 **AGPL-3.0** 协议开源，完整协议见 [`LICENSE`](./LICENSE)。

---

## 📱 版本信息

| 项目 | 值 |
|---|---|
| 包名 | `com.sevtinge.hyperceiler` |
| 版本名 | `2.10.166-5e46860-r6` |
| 版本码 | `6` |
| 上游基线提交 | `5e4686069dd7ab1f3697e256d5fc7d68fb73e317` |
| 最小 / 目标 SDK | `35` / `37` |
| ABI | `arm64-v8a` |
| 构建环境 | JDK 25 / Gradle 9.7.1 / Android SDK 37 |
| 签名 | Android Debug 签名（**与官方不同，无法直接覆盖安装**） |
| APK SHA256 | `fc4072eb99609d3bbb00c35325423463818dec35f877ad2f01f5d4bf0730249c` |

---

## 🆚 相比原版（上游 `5e4686069`）的修改

共修改 **7 个文件**（+78 / −20 行），另有 3 个新增文件。

### 1️⃣ 禁用「小米互传」自动关闭 ⭐ 核心功能

**文件**：`library/libhook/.../rules/mishare/NoAutoTurnOff.kt`

小米互传在开启约 10 分钟后会被系统自动关闭。本修改通过多层拦截保持其常开：

| 拦截点 | 作用 |
|---|---|
| `stopAdvertAllDelay()` | 拦截自动关闭的广播调用 |
| `t0.x(Context, boolean)` | 拦截写入 `mishare_enabled = false`，保留开启写入 |
| `MiShareSwitchHelper.changeSwitchStatus(int)` | 将 `status < 2` 强制改写为 `2`（全部设备可发现） |
| 倒计时文案资源 | 将 `toast_auto_close_in_minutes` 等文案替换为空白 |
| `Toast.show()`（**v5 新增**） | 屏蔽上述文案被置空后产生的空白 Toast |

> 💡 **v5 关键改进**：原实现用「0 字段 + 2 方法」的结构计数去匹配互传内部的 `showToast` 类，
> 小米一旦改动该类结构即失效（实测 5.1.12 已失配）。
> v5 改为 hook **系统 `Toast.show()`** —— 系统 API 永不变化，**永久免疫互传版本更新**，
> 且仅拦截「空白文本」的 Toast，互传其它提示（如传输状态）不受影响。

**开关位置**：HyperCeiler → 小米互传 → 禁用自动关闭（`prefs_key_disable_mishare_auto_off`）

### 2️⃣ 隐藏通知省略点

**文件**：
- 新增 `library/libhook/.../rules/systemui/statusbar/icon/all/HideNotificationDots.java`
- 新增 `library/core/src/main/res/values/strings_rmdots.xml`（英文）
- 新增 `library/core/src/main/res/values-zh-rCN/strings_rmdots.xml`（中文）
- 注册：`SystemUIB.java` / `SystemUIV.java`
- 开关：`library/core/src/main/res/xml/system_ui_status_bar_icon_manage_new.xml`

状态栏通知图标溢出时，原版会显示省略点（`...`）。本修改在
`NotificationIconContainer$IconState.applyToView()` 前将 `visibleState` 由 `1`（显示点）
改为 `2`（隐藏），从而隐藏省略点，**不影响图标布局与动画**。

默认关闭。开关位置：系统界面 → 状态栏图标管理 → 通知 → 隐藏通知省略点
（`prefs_key_system_ui_status_bar_hide_notification_dots`）

### 3️⃣ 移除签名校验相关提示与采集

**文件**：
- `app/src/main/java/.../home/banner/HomePageBannerManager.java`
- `app/src/main/java/.../utils/DeviceInfoBuilder.java`
- `app/src/main/java/.../utils/NoticeProcessor.java`

| 原行为 | 修改后 |
|---|---|
| 签名校验失败时首页显示警告横幅 | 不再显示（条件短路为 `false`） |
| 设备信息页采集并显示 `Signature` / `SignCheckPass` | 不再采集 |
| 通知/公告需通过签名校验才展示 | 一律展示 |

---

## 📦 仓库内容

| 文件 | 说明 |
|---|---|
| `HyperCeiler-2.10.166-5e46860-r6-toastfix-v5.apk` | v5 编译产物（可安装） |
| `full-modifications.patch` | **v5 相对上游基线的完整改动**（7 个文件） |
| `toastfix-v5.patch` | v5 的 Toast 修复专项补丁 |
| `history-v3-rmdots.patch` | v3 阶段的 RmDots 补丁（历史追溯） |
| `LICENSE` | AGPL-3.0 协议原文（与上游一致） |

---

## 🔧 自行编译

```bash
git clone <本仓库地址>
# 1. 拉取上游源码
git clone https://github.com/ReChronoRain/HyperCeiler.git
cd HyperCeiler
git checkout 5e4686069dd7ab1f3697e256d5fc7d68fb73e317

# 2. 应用补丁
git apply /path/to/full-modifications.patch

# 3. 编译（需要 GitHub Packages 凭据以拉取 Miuix 依赖）
export GIT_ACTOR=<你的GitHub用户名>
export GIT_TOKEN=<你的GitHub Token>
./gradlew :app:assembleCanary
```

产物位于 `app/build/outputs/apk/canary/`。

---

## ⚠️ 重要提示

1. **签名不同**：本版本使用 Android Debug 签名，与官方 HyperCeiler 签名不一致。
   **无法直接覆盖安装官方版本**，需先卸载。
2. **仅限个人使用**：定制版本未经过官方完整测试，请在理解风险的前提下使用。
3. **依赖 LSPosed**：需要设备已安装 LSPosed / KernelSU + Zygisk 环境。
4. **适用范围**：修改针对 **HyperOS 3.0.5.0（Android 15）** 验证，
   其它系统版本可能需要调整。

---

## 📄 许可

本仓库衍生的代码部分遵循 **GNU Affero General Public License v3.0 (AGPL-3.0)**，
与上游 [ReChronoRain/HyperCeiler](https://github.com/ReChronoRain/HyperCeiler) 保持一致。
完整协议见 [`LICENSE`](./LICENSE)。

上游项目版权归 HyperCeiler Contributions (2023-2026) 所有。
