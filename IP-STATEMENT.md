# IP Statement — IntentGrant Repository

> This statement applies to the **entire IntentGrant repository** and everything published under it: the Core Specification, all Profiles (IG-Lite Consumer Profile, and future profiles including IG-Phy), all transport bindings, and all reference materials — regardless of directory.
>
> **Language precedence**（语言效力分流）: This statement is published in English (Part A) and Chinese (Part B). In case of conflict: provisions concerning **Chinese law anchors**（《电子签名法》《民法典》《个人信息保护法》etc.) — the Chinese version prevails; provisions concerning the **Apache-2.0 licensing mechanism** — the English version and the Apache License 2.0 English original prevail; general provisions — the English version prevails.

## 1. License Grant — What Is Covered

The following are licensed under the **Apache License 2.0**:

- All specification texts in `core/` and `profiles/` (the "Specifications");
- Reference pseudocode and any reference implementation published **in this repository**;
- Supporting materials authored for this repository (README, CONTRIBUTING, the Specifications' appendices).

Distributors of any of the above must carry the [NOTICE](NOTICE) file per Apache-2.0 §4(d).

## 2. Patent License — Restatement, Not Addition

Each Contributor, by submitting a Contribution, grants the patent license described in **Apache-2.0 §3** — a personal, royalty-free license to make, have made, use, offer to sell, sell, import, and otherwise transfer any compliant **Implementation** of the Specifications, to the extent of that Contributor's Contribution.

**This statement adds no additional patent promise and weakens none.** Beyond Apache-2.0 §3 and the license grant in §1, the protocol author makes no other patent-license declaration with respect to the Specifications. The security of a compliant implementer's patent position comes from Apache-2.0 §3 itself — restated here only so that implementers do not have to infer it.

## 3. Trademark Reservation

The following marks are **not** licensed under the Apache License (Apache-2.0 §6), whether or not registered:

- **Protocol marks**: "IntentGrant", "IG-Lite", "IG-Phy", and associated logos;
- **Product marks**: "RoleKey" and future hardware product names — RoleKey is a hardware product name, **not** a protocol component (its corresponding role within the protocol is the PAE, the Physical Authorization Endpoint).

- You may refer to any of the above descriptively (e.g., "implements the IntentGrant IG-Lite Consumer Profile v0.2") — accurate, fair reference is always permitted;
- You may **not** use them as your product's name, or in any way that suggests endorsement, certification, or affiliation, absent the (future) conformance and certification program described in §6.

## 4. Scope Exclusions — What This Repository Does NOT License

The Apache-2.0 grant above covers the Specifications and repository materials only. It **does not** license, and no right is granted or implied to:

a) **Hardware designs, firmware, and manufactured devices** — including RoleKey and any PAE hardware product, whether prototyped, mass-produced, or planned;
b) **Key custody / escrow services** and their operational infrastructure;
c) **Evidence-custody, audit, and timestamp operation services** (the custody operator role described in the Specifications);
d) **Enterprise connectors, integrations, and proprietary SDKs** beyond the reference pseudocode published here;
e) The **financial-grade (CA-based) legal track** implementation offered under the Core Specification's Enterprise path;
f) **Future conformance test suites and certification materials** — these, if published, will carry their own terms and are expected to sit on the certification track, not the open-repository track.

These remain the reserved commercial territory of the protocol author and its partners; cooperation models are to be agreed separately. **Implementing the Specifications compliantly requires no additional license; the exclusions target services, hardware, and operations — never the implementation of the protocol itself.**

**The exclusions above do not constitute an "access barrier."** Any vendor may freely implement the protocol and perform evidence hashing and verification locally — no license or authorization is required. The custodian role is reserved because it carries neutral third-party responsibility requiring qualifications and operational capability — a matter distinct from "whether one can implement the protocol." **The custodian role is not exclusive**: any qualified third party (notaries, judicial-appraisal bodies, cloud providers, etc.) may operate custody services per the Specifications; nothing in this statement grants or suggests exclusivity.

## 5. Implementation Tiers — Honesty Boundary

- What is open here is **specification + pseudocode-level reference material**, as-is, per the implementation-status statement in each Specification (e.g., IG-Lite §12).
- Production implementations, hardware integrations, and custody operations are **not part of this repository**; no license is granted or implied to them by this repository, and nothing here should be read as claiming their existence.

## 6. Conformance and Certification (Forward-looking)

A conformance program — self-declaration tiers at launch, third-party testing later — **does not exist today**; if and when launched, it will depend on community scale and the willingness of the first implementers, with no timetable. Nothing in this repository constitutes a conformance certificate, and no conformance claim may be made under the reserved trademarks until that program publishes its rules.

## 7. Contributions

Contributions are made under Apache-2.0, inbound = outbound, with Developer Certificate of Origin sign-off. See [CONTRIBUTING.md](CONTRIBUTING.md).

## 8. Amendments

This statement changes only via commits to this repository; the version on the default branch is authoritative. The commit history is the change log.

---

# Part B：中文对照版

> 本声明适用于**整个 IntentGrant 仓库**及其发布的全部内容：Core 规范、全部 Profile（IG-Lite 消费者 Profile，及未来的 IG-Phy 等）、全部传输绑定与参考材料——不区分目录。
>
> **语言效力分流**：本声明以英文（Part A）与中文（Part B）发布。如有冲突：涉及**中国法锚点**的条款（《电子签名法》《民法典》《个人信息保护法》等），以中文版为准；涉及 **Apache-2.0 授权机制**的条款，以英文版及 Apache License 2.0 英文原文为准；一般性表述，以英文版为准。

## 1. 授权范围——覆盖什么

以下内容按 **Apache License 2.0** 授权：

- `core/` 与 `profiles/` 中的全部规范文本（"本规范"）；
- 本仓库内发布的参考伪代码及任何参考实现；
- 为本仓库撰写的支持材料（README、CONTRIBUTING、规范附录）。

分发上述内容者须按 Apache-2.0 §4(d) 随附 [NOTICE](NOTICE) 文件。

## 2. 专利授权——重申而非新增

各贡献者提交贡献时，即按 **Apache-2.0 §3** 授予专利许可——就其贡献部分，向任意合规**实现者**授予免费的制造、使用、销售、进口等权利。

**本声明不新增、也不减弱任何专利承诺。** 除 Apache-2.0 §3 及第 1 节授权外，协议作者未就本规范内容作出任何其他专利许可声明。合规实现者的专利安全感来自 Apache-2.0 §3 本身——此处重申，只是为了让实现者不必自行推断。

## 3. 商标保留

以下标识**不在** Apache 授权范围内（Apache-2.0 §6），无论是否注册：

- **协议标识**："IntentGrant"、"IG-Lite"、"IG-Phy" 及关联标识；
- **产品标识**："RoleKey" 及未来硬件产品名——RoleKey 是硬件产品名而非协议组件（其在协议内对应角色为 PAE，物理授权端点）。

- 允许描述性引用（如"实现了 IntentGrant IG-Lite Consumer Profile v0.2"）——准确、公允的引用永远被允许；
- 未经 §6 所述（未来的）一致性认证计划授权，不得用作产品名，或以任何暗示背书、认证、隶属关系的方式使用。

## 4. 范围排除——本仓库不授权什么

上述授权仅覆盖规范与仓库材料。以下内容**不因本仓库开源而被授权**，不存在任何明示或默示许可：

a) **硬件设计、固件与量产设备**——含 RoleKey 及任何 PAE 硬件产品（原型、量产或规划中）；
b) **密钥托管/代管服务**及其运营基础设施；
c) **证据托管、审计与时间戳运营服务**（规范中的托管方角色）；
d) **企业连接器、集成与专有 SDK**（本仓库发布的参考伪代码除外）；
e) Core 规范企业路径下的**金融级（CA 认证式）轨道**实现；
f) **未来的一致性测试套件与认证材料**——如发布，将自带条款，预期归认证轨道而非开源轨道。

上述保留为协议作者及其合作方的商业领域，合作模式另行约定。**合规实现协议不需要任何额外许可；排除项针对服务、硬件与运营，不是协议实现本身。**

**本声明的排除项不构成"接入门槛"**：任何厂商均可自由实现协议、并在本地完成存证计算与验证——无需任何许可或授权。托管方角色之所以保留，是因为其承担中立第三方的责任，需具备资质与运营能力——这与"能不能实现协议"是两个问题。**托管方不设独家**：任何具备资质的第三方（公证处、司法鉴定机构、云厂商等）均可按规范运营托管服务，本声明不为其设置或暗示排他授权。

## 5. 实现分层——诚实边界

- 本仓库开放的是**规范＋伪代码级参考材料**，按各规范"实现状态声明"（如 IG-Lite §12）如实分级；
- 生产实现、硬件集成、托管运营**不在本仓库**；本仓库不向其授权、不默示许可，任何内容不得被解读为声称其存在。

## 6. 一致性与认证（前瞻声明）

一致性认证计划——启动期为自我声明分级、后续引入第三方测试——**今日尚不存在**；如启动，将取决于社区规模与首批实现者意愿，无时间表。本仓库任何内容均不构成一致性认证；在该计划发布规则前，不得以保留商标作出一致性宣称。

## 7. 贡献

贡献按 Apache-2.0 授权（inbound = outbound），需 DCO 签署。见 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 8. 修订

本声明仅通过仓库 commit 修改，默认分支上的版本为权威版本；commit 历史即变更记录。

---

*IP Statement v1.1 · 2026-09-20 · IntentGrant repository root · v1.0→v1.1: absorbed external review (language-precedence split, trademark grouping, no-access-barrier clause, patent declaration neutrality, conformance timetable honesty)*
