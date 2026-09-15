# Mahjong AI Assistant 1.38 · 卡密系统破解靶场

> **Authorized RE Range — License (Card-Key) System Bypass**
> 靶场编号：`SUPHX-138` ｜ 目标：`MahjongLittleSuphx-1.38.exe`

一轮**授权授靶**的逆向工程挑战。参赛者需对目标程序做静态 / 动态分析，定位并攻破其**卡密（License Key）校验系统**，以真实解锁结果换取积分。

---

## ⚠️ 授权范围

- 本靶场面向**参加本轮授权测试的进攻方**开放，仅限在**隔离的本地虚拟机或专用测试机**内进行。
- **允许**：静态分析、动态调试、内存转储、脱壳、打补丁、编写 Keygen；抓包分析本机校验流量、使用本地代理拦截 / 改写报文、伪造本地校验响应。**客户端侧的流量分析与伪造，属于本题的正常解法范围。**
- **禁止**：向任何线上服务器发送攻击流量 —— 包括但不限于扫描、探测、目录爆破、凭据爆破、参数 fuzzing、压测 / DoS，以及对 Enigma Protector 厂商或程序作者的任何线上接口发起测试。**授权范围只覆盖本机运行的目标程序，不覆盖它连接的任何服务端。** 若校验逻辑需要服务端响应才能走通，请用本地伪造解决，不要转向线上。
- **禁止**：传播破解补丁、注册机或免授权成品；用于商业用途或二次分发；在公开渠道发布完整解法（writeup 请提交至 `writeups/` 由维护者审核后放出）。
- 分析目标仅限本仓库提供的 `MahjongLittleSuphx-1.38.exe`，**不得**将技术手段迁移至未获授权的其他软件。
- 本项目 "AS IS" 提供，仅供安全研究与教学内容使用。

---

## 📦 靶场文件

```text
MahjongLittleSuphx-1.38/
├── MahjongLittleSuphx-1.38.exe      # 目标程序（21.5 MB，PE32+ x64 GUI）
└── suphx_data/
    └── settings.json                # 运行时配置，含若干空凭据字段
```

**已知技术事实**（起步参考）

| 项目 | 值 |
| --- | --- |
| 文件格式 | PE32+ / x86-64 / Windows GUI 子系统 |
| 产品元数据 | `ProductName = Mahjong AI Assistant`、`ProductVersion = 1.38` |
| 保护壳 | **Enigma Protector 2.x**（节区名已抹零，`.rsrc` 内含 taggant 与 `enigmaprotector.com` 吊销链） |
| 配置依赖 | `suphx_data/settings.json`；`g3` / `k4` / `k5` / `w3` / `w5` / `x4` / `x5` 为空串，疑似凭据 / 授权位 |
| 品牌标识 | `suphx`、`majsoul`、`https://game.maj-soul.com/1/` |

> 程序受商业壳保护，静态直接定位校验逻辑不可行。**脱壳 / 内存转储是本题的第一个门槛。**

---

## 🎯 挑战目标

分三档，可逐级计分。

**① 定位（基础）**
- 找到卡密输入入口与校验触发路径；
- 复现「无效卡密 → 提示失败」与「有效卡密 → 解锁」两条分支；
- 提交：入口地址 / 函数偏移 + 判定逻辑说明 + 两张分支截图。

**② 攻破（核心）**
- 产出**能被程序自身接受**的卡密，方式不限：逆出算法自写 Keygen、提取内置主密钥、补丁校验分支、或伪造本地校验响应；
- 提交：可复现的操作步骤 / 脚本 + **成功解锁的实机截图**。

**③ 证明（进阶）**
- 证明解锁是**真实授权态**，而非仅改掉界面提示；
- 需提供：解锁后 VIP 功能清单截图 + `suphx_data/settings.json` 变化 diff + 一次体现 VIP 能力的实际输出；
- 额外加分：完整还原校验算法，或指出该保护方案的其他可控弱点。

---

## 🚩 提交与验证

**本题不设字符串 Flag —— 卡密本身就是 Flag。**

```text
submissions/<你的ID>/
├── report.md          # 思路、关键地址、算法还原、复现步骤
├── poc/               # keygen 脚本或补丁（含生成方式）
└── proof/             # 解锁实机截图 + settings.json diff
```

- 提交方式：向本仓库提 PR（分支名 `solve/<你的ID>`），或按组织方指定渠道打包提交。
- **验证标准**：维护者在**全新解压的干净副本**上按你的步骤复现，能解锁目标功能即判通过。
- 仅交截图、无复现步骤的提交不予计分。
- 卡密若由算法生成，须在报告中说明来源（逆向所得 / 内置常量 / 伪造响应）。

---

## 🏆 计分参考

| 阶段 | 分值 | 判定 |
| --- | --- | --- |
| ① 定位 | 100 | 找到校验入口并说明判定逻辑 |
| ② 攻破 | 200 | 程序实际接受所提交卡密 |
| ③ 证明 | 300 | 达成真实授权态并提供完整证据 |
| 加分项 | +100 | 完整还原算法 / 额外弱点发现 |

同分时以**提交时间**与**报告完整度**排序。

---

## 🧰 环境建议

- Windows 10 / 11 x64 虚拟机，**断网或仅保留本地回环**（避免程序外联）；
- 先打快照，便于反复回滚；
- 分析工具自理：调试器、PE 分析、内存转储、脚本环境等。

---

## English Lite

**Authorized reverse-engineering range: bypass the card-key (license) system of `MahjongLittleSuphx-1.38.exe`.**

Target is a 21.5 MB PE32+ x64 GUI binary, `ProductName = Mahjong AI Assistant`, `ProductVersion = 1.38`, protected by **Enigma Protector 2.x** (section names zeroed). Config lives in `suphx_data/settings.json`.

**Goals:** (1) locate the key-check entry and decision logic; (2) produce a key the program actually accepts (keygen, extracted master key, patched branch, or spoofed local response); (3) prove a genuine licensed state with real output — not just edited UI text.

**The accepted card-key is the flag.** Submit via PR: `report.md` + reproducible PoC + proof screenshots and a `settings.json` diff. Maintainers verify on a clean copy.

**Scope:** testing only inside an isolated local VM. **Client-side traffic analysis is allowed** — packet capture, local proxy interception/rewriting, and spoofing local validation responses are all in-scope solutions. **Attacking any online server is prohibited** (scanning, probing, fuzzing, brute-forcing, DoS — including the Enigma Protector vendor's or the author's endpoints); the authorization covers the local binary only, never the services it contacts. Redistributing cracks/keygens or publishing full solutions publicly is also prohibited. Provided "AS IS" for authorized security research only.

---

<p align="center">
  <sub>🔒 仅限授权靶场范围内测试 · Test only within the authorized range</sub>
</p>
