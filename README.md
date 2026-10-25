<div align="center">

# 改造老 Unity 安卓游戏 · AI Agent 技能包

### Retro Unity Android Game Modding · AI Agent Skills

**dex 等长改写 · 渠道 SDK 剥离 · DLL 汉化 · 资源文本 · 中文字体 · 自动验证**

六个在真实汉化项目里逐个踩出来的技能，合起来就是「改造一棵老 Unity 安卓游戏」的完整流程。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](./LICENSE)
![Skills](https://img.shields.io/badge/skills-6-8957e5?style=flat-square)
![Platform](https://img.shields.io/badge/platform-Android%20%7C%20Unity%204.x--5.x-3ddc84?style=flat-square&logo=android&logoColor=white)
![Lang](https://img.shields.io/badge/lang-%E4%B8%AD%E6%96%87%20%2F%20English-1f6feb?style=flat-square)
[![Last commit](https://img.shields.io/github/last-commit/lueuru/game-modding-skills?style=flat-square)](https://github.com/lueuru/game-modding-skills/commits)
[![Stars](https://img.shields.io/github/stars/lueuru/game-modding-skills?style=flat-square)](https://github.com/lueuru/game-modding-skills/stargazers)

**[中文](#) · [English](./README.en.md)**

</div>

---

## 这是什么

这是一组 **AI Agent 技能（Skill）**，面向一个具体而顽固的场景：

> 手里有一个几年前的老 Unity 安卓游戏（Mono / .NET 3.5），想汉化它、去掉渠道 SDK 的弹窗、
> 或者改掉里面的一句英文——但反编译重打包的老路每一步都在闪退边缘。

六个技能覆盖完整流程的每一段，共同的设计取向只有一条：

**能原地等长替换的，绝不重打包；必须先判定的，绝不找到字符串就替换。**

老游戏改错一个字节就可能启动即崩，所以这些技能全是「零长度变化」流派：
文件长度不变 → 偏移不变 → 签名校验和的重算范围被压到最小，出错面呈指数级缩小。

## 技能总览

| 分组 | 技能 | 解决什么 | 代表场景 |
|---|---|---|---|
| **改造底座** | `android-dex-string-patch` | dex 字符串原地等长替换 | 启动 Toast 换自己的署名 |
| **改造底座** | `apk-channel-sdk-strip` | 渠道 SDK 入口剥离 | 进游戏弹实名认证 / 广告 |
| **改造底座** | `android-emulator-autotest-setup` | 无人值守验证 APK 能否跑 | 改完的包没设备怎么测 |
| **汉化三件套** | `unity-dll-chinese-patch` | DLL 层文本汉化（#US 堆） | 翻了就闪退 / 有东西翻不到 |
| **汉化三件套** | `unity-resource-text-localization` | 资源层文本汉化（level0 等） | 按钮文字变英文 / 功能失效 |
| **汉化三件套** | `unity-bmfont-chinese-font` | 位图字体中文化 | 汉化后全是方块字 |

## 安装

技能目录需要放在 Agent 的技能根目录下。以 WorkBuddy 为例（路径按你的环境调整）：

**整仓库克隆（推荐，六个技能一起用）**

```bash
git clone https://github.com/lueuru/game-modding-skills.git
cp -r game-modding-skills/skills/* ~/.workbuddy/skills/
```

**只取一个（每个技能都独立可用）**

```bash
cp -r game-modding-skills/skills/unity-dll-chinese-patch ~/.workbuddy/skills/
```

装好后 Agent 会在匹配到触发场景时自动加载；也可以直接点名，例如「用 `unity-dll-chinese-patch` 处理这个 DLL」。

## 技能详解

### android-dex-string-patch

在已打包的 APK 里改写 `classes.dex` 的字符串（典型场景：把启动 Toast 换成署名 / 中文文案），
**不重新编译、不动任何字节码指令、不改变文件长度**：挑一条可牺牲的等长槽位原地重写，
只改 `const-string` 指令的 2 字节操作数，再重算签名与校验和。

两条会直接导致「点击图标即闪退」的硬约束（技能内写清了推导，这里给结论）：

1. `utf16_size` 是 **UTF-16 字符数，不是 UTF-8 字节数** —— 纯 ASCII 串上二者恰好相等，
   写成字节数在英文串上完全看不出问题，**一换中文就启动即崩**；
2. 字符串索引表必须保持**字典序升序**，而中文码位大于全部 ASCII —— 中文串不能随便挑槽位。

### apk-channel-sdk-strip

渠道包（百分网 / 应用宝 / 九游等）会在原包上二次打包，注入实名认证、广告、统计模块，启动就弹窗。
本技能通过**原地等长改写 `AndroidManifest.xml` 的 AXML 字符串池**，切断渠道 SDK 的
`Application` / `Provider` 激活入口 —— 不重打包、不改字节码。

删代码 ≠ 去掉功能：渠道 SDK 的激活是「入口驱动」的，切断入口远比删代码安全。

### android-emulator-autotest-setup

在 Windows 上搭 Android 模拟器 + 全自动「安装 / 启动 / 崩溃检测」流水线，无人值守地验证一个
APK 能否正常运行。包含**资源可行性预检**（内存 / 磁盘 / 虚拟化 / 硬件加速）——
很多环境根本跑不起模拟器，先花两分钟预检，比折腾一小时才发现跑不起来划算。
并给出**模拟器不可用时的静态验证替代方案**（解析 APK 结构、校验偏移、检查字符串）。

### unity-dll-chinese-patch

解析 `Assembly-CSharp.dll` / `*-firstpass.dll` 的 `#US` 字符串堆，用 **IL 上下文**判定每条英文是
「界面显示文本」（可翻）还是「代码引用名 / 比较值」（禁译），再做**块级原地替换**（长度与偏移零变化）。

- 偏移必须解析 PE 元数据得到，**不能启发式扫描猜** —— 猜会偏 30+ 字节，读出 `敩r杲b` 这类错位乱码，而且不报错；
- 同一个字符串可能有**双重身份**（既是显示文本、又被 `==` 当查找键）—— 一刀切两头都错，必须逐条判；
- 附带两个脚本：`scripts/il_logic.py`（IL 上下文判定）、`scripts/fix_dll_logic.py`（写回）。

### unity-resource-text-localization

扫描 `level0` / `sharedassets*.assets` 里的字符串字段，建立**禁译键**，把译文写回。
解决「汉化后按钮文字变英文」「某些文本仍是英文」「翻了就崩 / 功能失效」。

核心是禁译键：老游戏里有大量靠字符串比较驱动逻辑的写法（如 `if (shipData.shipclass == "Type XXI")`），
这类字符串翻了等于把功能删掉。

### unity-bmfont-chinese-font

原版 BMFont 图集只含 ASCII，汉化后全是方块。本技能自制 **2048² 图集 + `.fnt` 描述文本**，
写回 `sharedassets*.assets` 里的 `Texture2D` / `TextAsset`。

- 字符集取「汉化后**实际用到**的字」—— 零缺字，且比塞入全字库字号更大更清晰；
- **必须最后做**：顺序反了只能用全集，字号被迫变小。

## 共同纪律

这些不是从文档抄的，是在一个真实的 Unity 老游戏汉化项目上反复踩出来的：

- **等长 / 原地替换 = 零风险**：长度不变 → 偏移不变 → 校验和重算范围就那两处；
- **先判定再动手**：可译性、禁译键、槽位合法性，全部先查后写；
- **偏移必须解析元数据**：启发式扫描会静默给出错误答案（乱码且不报错）；
- **判据要防假通过**：验证脚本自己坏掉时报的「通过」一律作数前，先故意弄坏一次确认它会红。

## 适用与不适用

**适用**：Unity 4.x ~ 5.x（Mono / .NET 3.5）安卓游戏的**文本汉化、字体替换、署名注入、渠道 SDK 剥离、无人值守验证**。

**不适用**：

- 加壳 / 加密资源的游戏（需要先脱壳）
- IL2CPP 打包的游戏（没有 `Assembly-CSharp.dll`，走 `global-metadata.dat`，路径完全不同）
- 需要**改动逻辑**的改造（改代码逻辑请走重编译路线，原地改写做不到）
- iOS（完全不同的一套）

## 合规提醒

这些技能用于**你自己拥有或已获授权修改**的应用。去掉渠道 SDK 弹窗、替换署名，
都应在你有权修改该应用的前提下进行；**不要用它去篡改他人应用后分发**。

## 相关仓库

- **[agent-engineering-skills](https://github.com/lueuru/agent-engineering-skills)** —— AI Agent 工程实践 —— 验收判据防假通过、仓库主页美化、Windows 清理审计、DSH 排障与会话复盘

## 支持一下

如果这套技能帮你省下了时间，欢迎点个 ⭐ Star——它能让更多改造老游戏的人看到这套方法。

## 许可证

[MIT](./LICENSE) —— 自由使用、修改、再分发，保留版权声明即可。

</div>
