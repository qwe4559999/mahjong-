# Mahjong AI Assistant 1.38 · 卡密系统攻防靶场

> **Authorized RE Range — License System Bypass (SUPHX-138)**
> 目标：`MahjongLittleSuphx-1.38.exe`（受商业壳保护、联网运营的第三方软件）

> 📅 本版要求依据**首轮侦察报告**修订。上一版将本题设计为「逆出算法写 Keygen」，与目标实际实现不符，已废弃。修订要点见下文「重要前提」。

一轮**授权授靶**的逆向工程挑战。目标程序受商业壳保护、授权判定在服务端，参赛者需完成**脱壳 → 协议还原 → 二进制补丁**链路，最终让程序在校验不可达的情况下进入主界面。

> 📌 **重要前提（据首轮侦察报告修订）**
> 本目标的卡密**不是客户端算法卡密**。客户端只负责把密钥 POST 给服务端、解析回包并写本地态，**没有任何本地生成或验签逻辑**。因此：
> - 「离线逆出算法自写 Keygen」与「提取内置主密钥」在本目标上**不存在对应物**，不作为本题要求；
> - 唯一的攻破路径是**脱壳后二进制补丁**（改校验分支）或**替换内置根证书集**（改 TLS 信任源以劫持校验响应）；
> - 本题的**真正门槛是脱壳与可重建镜像**，不是算法还原。

---

## ⚠️ 授权范围

- 本靶场面向**参加本轮授权测试的进攻方**开放，仅限在**隔离的本地虚拟机或专用测试机**内进行。
- **允许**：静态分析、动态调试、内存转储、**脱壳**、打补丁、运行时注入；抓包**观察**本机校验流量、本地代理拦截、在 **hosts 重定向到本地**的前提下伪造校验响应。**客户端侧的流量分析与本地伪造，属于本题的正常解法范围。**
- **禁止**：向任何线上服务器发送攻击流量 —— 包括但不限于扫描、探测、目录爆破、凭据爆破、参数 fuzzing、压测 / DoS、重放，以及对 Enigma Protector 厂商或程序作者的任何线上接口发起测试。**授权范围只覆盖本机运行的目标程序，不覆盖它连接的任何服务端。** 若校验逻辑需要服务端响应才能走通，请在本地自建应答，不要转向线上。
- **禁止**：传播破解补丁、注册机或免授权成品；用于商业用途或二次分发；在公开渠道发布完整解法（writeup 请提交至 `writeups/` 由维护者审核后放出）。
- 分析目标仅限本仓库提供的 `MahjongLittleSuphx-1.38.exe`，**不得**将技术手段迁移至未获授权的其他软件。
- 本项目 "AS IS" 提供，仅供安全研究与教学内容使用。

---

## 📦 靶场文件

```text
MahjongLittleSuphx-1.38/
├── MahjongLittleSuphx-1.38.exe      # 目标程序（21.5 MB，PE32+ x64 GUI）
└── suphx_data/
    └── settings.json                # 运行时配置，键名为序列化别名（授权态存于此）
```

**已知技术事实**（起步参考）

| 项目 | 值 |
| --- | --- |
| 文件格式 | PE32+ / x86-64 / Windows GUI 子系统，`ImageBase = 0x140000000` |
| 产品元数据 | `ProductName = Mahjong AI Assistant`、`CompanyName = fkc`、**PE 版本资源 `ProductVersion = 0.1.0`** |
| 发布版本 | 目录名 / 文件名中的 `1.38` 为发布号，与 PE 版本资源不一致（属正常现象，勿据此判错样本） |
| 保护壳 | **Enigma Protector 2.x**（9 节区名全为 `\x00`，节区熵 8.0000，导入表仅 28 dll × 1 函数；EP 落在虚拟节区尾部，非 OEP） |
| 技术栈 | 脱壳后可见 **Tauri 2.10.3**（tao / wry / hyper / **rustls 0.23.37** / aws-lc-rs）+ **WebView2 + React + TypeScript** 前端（内嵌，经 `http://tauri.localhost/` 提供）；另有独立 HUD 窗口 |
| 授权架构 | 全部校验在**腾讯云 CloudBase 云函数**侧，客户端仅做文案映射与本地态写入 |
| 配置依赖 | `suphx_data/settings.json`，键名为序列化别名（见下表） |
| 卡密格式 | 旧版为 **18 位字母数字**启用码；1.38 改为服务端签发的 `online_apikey`，前端已无格式校验 |
| 构建痕迹 | 编译机路径 `C:\Users\Fkc\.cargo\registry\src\...` |

**`settings.json` 关键键位（已交叉验证）**

| 键 | 含义 | 键 | 含义 |
| --- | --- | --- | --- |
| `x4` | `online_apikey`（卡密） | `n6` | `coach_boss_vip` |
| `i5` | `api_suphx_key` | `n7` | `coach_special_vip` |
| `k4` / `k5` | suphx / coach 到期展示 | `g8` | `client_path` |
| `w3` / `w5` | 模型线 / coach 密钥 | `j6` | `update_url` |

> 静态字符串全表仅 4,784 条且无任何业务串 → **静态直读不可行，脱壳 / 内存转储是硬门槛**（首轮已实测确认）。
> ⚠️ 但注意：**② 关的次级攻击面不依赖脱壳**，是本轮性价比最高的方向。
>
> 上述事实为首轮侦察结论，**允许并鼓励参赛者独立复核**。发现与本表不符之处，请连同证据一并提交（计入 ④ 加分项）。

---

## 🚫 产物边界（新增，务必遵守）

本目标是**联网运营的商业第三方软件**，与纯本地靶机不同，以下红线为硬性要求：

1. **不得对真实服务端发起任何请求性测试。** 已知域名 `env-00jxgjneh2o7.dev-hz.cloudbasefunction.cn`、`env-00jxgx7altlg.dev-hz.cloudbasefunction.cn` 属第三方线上资产。禁止对其扫描、探测、fuzzing、爆破、重放、压测。**抓包观察**可以，**发请求**不可以。
2. **劫持校验流量必须落在本地。** 正确做法是把目标域名通过 hosts / 本地 DNS 指向 `127.0.0.1`，由自建服务应答；**不要**用透明代理把真实请求转发到上游再改写响应，那会让你的实验流量真实触达第三方服务。
3. **不得产出可分发的免授权成品。** 补丁、脱壳镜像、伪造响应工具仅限作为本题提交物与内部复现用途。**禁止**公开发布、上传网盘、二次分发或用于任何商业场景。
4. **不得改装后用于线上对局。** 该程序内置自建 MITM 代理（读 `%USERPROFILE%\mitmproxy-ca.pem`）用于劫持雀魂客户端流量。**严禁**把完成补丁后的程序接入真实线上游戏环境 —— 这属于破坏他人游戏服务与公平性，与本靶场无关，且后果自负。
5. **提交物中不得包含真实卡密。** 若通过任何途径获得了有效卡密，**不要**写入报告、补丁或截图，请立即上报组织方。
6. **环境隔离。** 全程在断网虚拟机内进行；需要联网装工具时，与运行目标分时进行。

---

## 🎯 挑战目标

按可达成性分层，逐级计分。**① 为基础门槛，② 为本轮唯一确定可达的攻破路径，③ 为核心难点。**

**① 定位与协议还原（基础）**
- 找到卡密输入入口与校验触发路径（登录页 → `invoke` 命令链）；
- 还原客户端命令表与**服务端返回码 → 客户端文案**的完整判定映射；
- 还原校验协议：端点、请求参数、响应字段；
- 提交：入口位置 + 命令调用链 + 判定分支表 + 协议契约 + 实机截图。
- ✅ 首轮已完成，见 `reports/`。**复核 / 勘误同样计分**（发现首轮结论有误并给出证据者，按加分项计）。

**② 次级攻击面（本轮确定可达，重点）**
- 利用**不依赖脱壳**的攻击面达成实际效果。已识别候选：更新链路走 `curl` 子进程 + `MahjongLittleSuphx_apply_*.bat`（`copy /Y` + 重启），若 `updateUrl`（配置位 `j6`）可控则存在**本地任意文件覆盖 / 代码执行**面；
- 也接受其他等效发现：Tauri IPC 越权、`http://ipc.localhost/` 接口滥用、HUD 窗口注入等；
- 提交：**可复现的 PoC** + 影响说明 + 实际执行证据（不是理论推测）。
- ⚠️ 要求给出真实可利用性判定。若经实测证明不可利用，**同样计分**（负结果 + 严谨论证按半分计）。

**③ 脱壳与二进制补丁（核心难点）**
- 完成 Enigma Protector 2.x 脱壳，得到**可重建、可运行的镜像**；
- 定位 `online.rs` 校验成功分支，补丁使其在**校验不可达**的情况下进入主界面；
- 提交：脱壳方法 / 工具 + 重建步骤 + 补丁点与原理 + 补丁后实机运行截图。
- 判定：维护者在干净副本上按步骤复现，程序**进入主界面且不依赖任何服务端响应**即通过。
- 允许替代路径：不重建镜像，改为**运行时内存补丁 / 注入**（须说明持久化方式与复现步骤）。

**④ 加分项（任选，可叠加）**
- **替换内置根证书集**：绕过 rustls 0.23 + WebPKI 内置 Mozilla 根证书（系统级 CA 注入无效），使本地伪造响应被接受；
- **完整配置键反混淆表**并交叉验证（首轮已还原部分，补充 + 勘误计分）；
- 指出该保护方案的其他可控弱点，并给出**经实测**的结论；
- 发现并证明首轮报告中的任何错误结论。

---

## 🚩 提交与验证

**本题不设字符串 Flag。判定标准是「程序在无有效授权的情况下进入主界面」。**

```text
submissions/<你的ID>/
├── report.md          # 思路、关键地址、协议契约、复现步骤
├── poc/               # 补丁 / 脚本 / PoC（含生成方式）
└── proof/             # 实机截图 + settings.json diff + 运行日志
```

- 提交方式：向本仓库提 PR（分支名 `solve/<你的ID>`），或按组织方指定渠道打包提交。
- **验证标准**：维护者在**全新解压的干净副本**上按你的步骤复现，结果与报告一致即判通过。
- 仅交截图、无复现步骤的提交不予计分。
- **提交物必须标注所处关卡**（① / ② / ③ / ④），并明确区分「已实测」与「推测」。
- 提交物中**不得包含对真实服务端的请求记录**；若发现提交中存在此类流量，该提交作废。

---

## 🏆 计分参考

| 关卡 | 分值 | 判定 |
| --- | --- | --- |
| ① 定位与协议还原 | 100 | 提交完整判定分支表 + 协议契约 |
| ① 复核 / 勘误 | +50 | 有证据地修正首轮结论 |
| ② 次级攻击面 | 200 | 可复现 PoC + 实际影响（负结果按半分计） |
| ③ 脱壳 + 补丁进入主界面 | 300 | 干净副本复现成功，不依赖服务端响应 |
| ④ 替换根证书集 / 其他弱点 | +100 | 经实测的有效结论 |
| ④ 配置键反混淆（完整 + 验证） | +50 | 提供机读版映射表 |

**不设「卡密伪造」独立关卡** —— 该路径依赖服务端私钥，本轮不作为要求。

同分时以**提交时间**与**报告完整度**排序；报告须能被他人在无沟通的情况下独立复现。

---

## 🧰 环境建议

- Windows 10 / 11 x64 虚拟机，**全程断网**（目标校验会外联，断网可避免误触第三方服务）；
- 先打快照，便于反复回滚；脱壳 / 补丁实验务必在快照内进行；
- 需要程序走到联网校验环节时，用 hosts 把目标域名指向 `127.0.0.1` 并自建应答服务，**不要**放行真实出站；
- 已知可复现的取证手段（首轮验证有效，供参考）：
  - 内存转储：`rundll32 comsvcs.dll, MiniDump` 在部分环境被 DCOM 拦截，改用 `dbghelp!MiniDumpWriteDump`（ctypes）；
  - 前端 JS 实际位于 **WebView2 renderer 进程**，主进程内没有，转储时勿取错进程；
  - 程序遵循 `HTTPS_PROXY` / `ALL_PROXY`，可据此确认外联目标（但见「产物边界」第 2 条）。
- 分析工具自理：调试器、PE 分析、内存转储、反混淆、脚本环境等。

---

## English Lite

**Authorized reverse-engineering range: defeat the license system of `MahjongLittleSuphx-1.38.exe` well enough to reach the main UI without a valid key.**

Target is a 21.5 MB PE32+ x64 GUI binary (`ProductName = Mahjong AI Assistant`, PE `ProductVersion = 0.1.0`; `1.38` is only the release name), protected by **Enigma Protector 2.x** (all 9 section names zeroed, entropy 8.0). Unpacked, it is a **Tauri 2.10.3** app (tao/wry/hyper/**rustls 0.23.37**) with a WebView2 + React frontend. Config lives in `suphx_data/settings.json`.

**Crucial premise:** this is **not** a client-side algorithmic key. All validation happens on a Tencent CloudBase function; the client only POSTs the key and parses the reply — there is **no local keygen or signature check**. Offline keygen and "extract the embedded master key" have **no counterpart** here and are not required.

**Levels:** ① locate the key entry, command chain, server-response→message mapping and protocol contract (100, +50 for evidenced corrections); ② exploit a **no-unpacking-needed** attack surface — the `curl`-child-process updater + `MahjongLittleSuphx_apply_*.bat` overwrite/restart path and similar (200, half credit for a rigorous negative result); ③ **unpack Enigma 2.x, rebuild a runnable image, patch the success branch** so the app reaches the main UI with the server unreachable (300); ④ bonus — replace the built-in rustls root store to accept spoofed responses, complete the config-key de-obfuscation, or disprove a first-round finding (+100/+50).

**Verification:** maintainers reproduce from a clean copy; the app must enter the main UI **without any server response**. The real barrier is unpacking + rebuildable image, not algorithm recovery.

**Scope:** testing only inside an isolated, **offline** local VM. Client-side traffic *observation* is allowed; **sending requests to the third-party servers is not** — redirect the domains to `127.0.0.1` via hosts and answer locally instead. No scanning, fuzzing, brute-forcing or DoS against any online server, including the Enigma vendor's endpoints. Do not redistribute patched/unpacked binaries, and **never** run a patched build against the live game service. Provided "AS IS" for authorized security research only.

Full Chinese sections above are authoritative.

---

<p align="center">
  <sub>🔒 仅限授权靶场范围内测试 · Test only within the authorized range</sub>
</p>
