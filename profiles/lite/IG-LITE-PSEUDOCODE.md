# IG-Lite v0.2 最小参考实现（伪代码）

> 性质：**伪代码级（pseudocode-level）**——非可执行实现。密码学原语（JCS 规范化、SHA-256、签名、CSPRNG）全部委托平台密码学库，本文件不内嵌任何算法细节；无 BLE 栈、无密钥管理、无真实持久化。
> 与规范的关系：**规范为准（normative），本文件为说明性（informative）**。字段名与流程步骤逐段引用《IG-Lite v0.2 规范》章节；所有能力声明以规范 §12 实现状态声明为准。
> 用途：向社区评议者展示披露 → 完备性校验 → 确认 → 凭证 → 回执 → 送达证明的最小闭环语义与字段流。
> 版本：基于《IG-Lite-v0.1-参考实现-伪代码》扩展（v0.1 文件保留不动，作为 v0.1.1 开源发布物档案）。**v0.2 新增三段：§1.5 required 完备性校验（含 fail 分支）、§8 Delivery Receipt（D1，含 nonce）、§9 Transit Fidelity 校验（D2）**；§1 增量修订（`schema_ref`/`delivery_tier` 字段）。**v0.2.1（2026-09-25）**：随规范勘误同步——审计层占位函数改名 `persist_audit`/`mark_audit`（原 `persist_L3`/`mark_L3`，L 简写收归保障等级），正文层名指称对齐 §2 编号消歧；无语义变更。

## 覆盖范围（附录 B 最小闭环 ＋ v0.2 扩展）

| 步骤 | 规范锚点 | 版本 |
| --- | --- | --- |
| 1. Disclosure Object 构造 | §3.1 | v0.1（v0.2 增量修订） |
| 1.5. required 完备性校验（含 fail 分支） | §6.3 / §4.2 / §4.4 | **v0.2 新增** |
| 2. 播报（呈现承诺，模式 b 结构化三元组） | §3.2 / §10.1.1 | v0.1 |
| 3. challenge 签发 | §3.3 | v0.1 |
| 4. 双击事件捕获 | §10.1.1 | v0.1 |
| 5. Confirm Grant 签发 | §4.2 | v0.1 |
| 6. Receipt 对齐 | §4.5 | v0.1 |
| 8. Delivery Receipt 生成（D1，含 nonce） | §4.7.1 / §5.6 | **v0.2 新增** |
| 9. Transit Fidelity 校验（D2） | §4.7.2 | **v0.2 新增** |

**不覆盖（诚实边界）**：激活层配对与绑定流程（Core v2.0 第 4 章 + §3.5 关联模型与 `association_model` 记录）、§6.2 Registry 治理与 schema 发布流程（本文件假定 schema 已经 Registry 获取并本地缓存）、§8 托管方提交协议全量、§10.2 频控状态机全量、跨设备路由（§5.4）、DCC-B 以上设备的 TEE 安全时钟实现（§4.7.1 安全时间源路径仅示范调用位置）——为后续参考实现范围。

## 0. 前置：常量与依赖

```
CONSTANTS:                                        # 默认值，行业 Profile 可调（evidence_rules_version 锁定）
  CHALLENGE_TTL_SECONDS = 60                      # §3.3 challenge 有效窗口
  DOUBLE_CLICK_INTERVAL_MS = 600                  # §10.1.1 双击判定间隔上限
  SILENT_PERIOD_MS      = 播报持续时长            # §10.1.1 播报静默期（播报中按键无效）
  LONG_PRESS_MS         = 800                     # §10.1.1 长按下限（对齐 Core v2.0 §5.5）
  GRANT_TTL_SECONDS     = 300                     # §4.2 Confirm Grant 过期（对齐 Core High 档）
  CONFIRM_PER_SESSION_MAX = 5                     # §10.1.2 频控阈值
  RECEIPT_NONCE_BITS    = 128                     # §4.7.1 per-receipt nonce 熵下限（隐私红线）
  RENDER_DURATION_DEFAULT_MS = 8000               # §4.7.1 claimed 路径默认值；安全时间源路径由双时戳实测

DEPENDS-ON（平台密码学库，本文件不实现）:
  JCS(obj) -> bytes                               # RFC 8785 JSON 规范化 [S15]
  SHA256(bytes) -> b64url                         # §3.1
  CSPRNG(n_bits) -> bytes                         # do_id ≥128bit / receipt nonce ≥128bit 熵源（§3.1/§4.7.1）
  sign(subject_key, bytes) -> sig                 # 签名主体由 attested_by 决定（§4.2）
  verify(pubkey_or_fingerprint, bytes, sig) -> bool
  secure_clock_now() -> timestamp | UNSUPPORTED   # §5.1 secure_clock 能力位；D1 双时戳来源（安全时间源路径）

SCHEMA-DEPS（来自 Registry，§6.2）:
  load_schema(schema_ref) -> schema               # {required: {field: spec}, na_rules, version}
  # 本地缓存 + 版本锁定；Registry 获取协议不在本文件范围
```

## 1. Disclosure Object 构造（§3.1，v0.2 增量修订）

```
function build_disclosure(action_intent, context_template, sensitive: bool):
    d = {
        "do_id":        "do_" + base32(CSPRNG(128)),      # ≥128bit CSPRNG，托管方禁录（§8）
        "schema_ref":   pick_schema(action_intent.type),  # v0.2 新增：ig:disclosure:<类型>@<版本>（§3.1/§6.4）
                                                          # 未登记类型回落 "ig:disclosure:T-0@…" 或置空（此时跳过 §1.5）
        "context_template": context_template.text,         # 语境模板（§3.2）
        "rendered":     fill(context_template, action_intent),
        "scope":        action_intent.scope,               # 如 "commerce.order.create"
        "counterparty": action_intent.counterparty,
        "amount":       action_intent.amount,              # {value, currency}
        "attrs":        action_intent.attrs,
        "risks":        context_template.risks,
        "sensitive":    sensitive,                         # 入 hash 计算对象（§3.1）
        "loa_target":   action_intent.loa_target,          # §5.3 静默降级禁令的锚点
        "delivery_tier": action_intent.delivery_tier,      # v0.2 新增：D0/D1/D2 声明，入 hash 防降级改写（§3.1/§5.6）
        "confirm_window": 60,
        "issued_at":    now_iso8601(),
        "evidence_rules_version": current_rules_version()
    }
    d["disclosure_hash"] = b64url(SHA256(JCS(d 减去 disclosure_hash 字段)))
    persist_audit(d)                                          # 判责链第 1 段：意图捕获留痕（§3.4）
    state = "drafted"                                      # §4.4 状态机
    return d
```

## 1.5 required 完备性校验（§6.3 强制力链第 1 环；v0.2 新增）

```
function validate_required(d):                             # 签发前置校验：§4.2 fail 分支 / §4.4 rejected_schema 态
    if d["schema_ref"] is None:
        return PASS                                        # 无 schema_ref → 跳过（回落 T-0 之外的未登记类型，§11.2 保留限制①）

    schema = load_schema(d["schema_ref"])                  # 本地缓存版本，交易全程锁定（§6.5 版本仲裁）
    for field, spec in schema.required:
        v = d.get(field)
        if v is None or v == "" or is_blank(v):
            # N/A 显式声明规则（§6.3）：缺失 ≠ 不适用；三态禁令——静默省略/空串/null 一律不合规
            na = d.get("applicability", {}).get(field)
            if na is None or na["applicability"] != "not_applicable" or is_blank(na["reason"]):
                return FAIL(reason="required_missing:" + field)

        if v is dict and v["applicability"] == "not_applicable":
            if is_blank(v["reason"]):
                return FAIL(reason="na_reason_blank:" + field)   # N/A 必须附真实法理理由（§6.3）
            # 滥用 N/A 的抽查由托管方准入审计承担（§6.3），签发端只做结构校验

    return PASS

# —— 签发链挂接点（§4.2 / §4.4）——
function on_issue_challenge(d):
    verdict = validate_required(d)
    if verdict == FAIL(reason):
        state = "rejected_schema"                          # §4.4 新增前置态
        persist_audit({"event": "rejected_schema",
                    "schema_ref": d["schema_ref"],
                    "reason": reason})                     # 拒签留痕入审计层（不含披露原文——§8 托管内容下限同构）
        return ABORT                                       # 拒绝生成 challenge：交易走不到确认环节（§6.3 第 1 环）
    return issue_challenge(d)
    # 注意分工（§6.3 第 2 环）：hash 保证完整性，本函数保证完备性——两条机制不得互相冒名
    # 生态定位（§6.3 诚实边界）：本校验防"无意漏字段"；有意绕过（不经 SDK）由验证端仲裁与法律端承担
```

## 2. 播报——呈现承诺模式 b（§3.2 / §10.1.1）

```
function present(d):
    triple = render_structured_triple(                   # 动作 × 对方 × 数额（弱表面播报形态）
        d["rendered"], d["counterparty"], d["amount"])
    t_broadcast = now()
    speak_or_display(triple)                              # TTS 或屏显；渲染留痕入审计层（§3.2）——v0.2 起即 D0（§3.3/§5.6）
    hold_at_least(PRESENTATION_MIN_MS)                    # 最短呈现时长（§3.2）
    state = "presented"
    return t_broadcast                                    # 静默期与 T 窗口起点（§10.1.3）
```

## 3. challenge 签发（§3.3）

```
function issue_challenge(d):
    c = {
        "nonce":           hex(CSPRNG(128)),
        "disclosure_hash": d["disclosure_hash"],          # 按键与披露绑定（防解绑攻击）
        "issued_at":       now_iso8601(),
        "expires_at":      now_iso8601() + CHALLENGE_TTL_SECONDS
    }
    persist_audit(c)                                         # 禁录 do_id（§8 托管内容下限）
    state = "challenge_issued"
    return c
```

## 4. 双击事件捕获（§10.1.1）

```
on PAE_button_event(ev):
    if state != "challenge_issued":
        discard(ev); return                               # 窗口外按键一律无效（防误触）

    if now() < t_broadcast + SILENT_PERIOD_MS:
        discard(ev); return                               # 静默期：播报中按键无效

    if ev == SINGLE_CLICK:
        discard(ev); await_second_click(DOUBLE_CLICK_INTERVAL_MS)
        return                                            # 单击不构成同意信号（§10.1.1）

    if ev == DOUBLE_CLICK and interval(ev) <= DOUBLE_CLICK_INTERVAL_MS:
        if now() > challenge["expires_at"]:
            state = "expired"; restart_from(step 2)       # 过期：整窗作废，重新披露
            return
        route_challenge_to_PAE(challenge)                 # DCC-B 以上：设备端对 challenge 签名（§3.5）
        state = "confirmed"（待 Grant 签发）

    if ev == LONG_PRESS(duration >= LONG_PRESS_MS):
        route(high_sensitivity_flow)                      # 高敏感动作通道（§10.1.1）
```

## 5. Confirm Grant 签发（§4.2）

```
function issue_grant(d, c, dcc_class):
    g = {
        "grant_id":   "urn:uuid:" + uuid4(),
        "do_id":      d["do_id"],
        "dv":         "1.0",
        "scope":      d["scope"],                         # 跟随 Disclosure 的动作单元，禁止扩大
        "schema_ref": d["schema_ref"],                    # v0.2 新增：版本随披露锁定，验证端按其复算完备性（§6.5）
        "delivery_receipt_ref": null,                     # v0.2 新增：D1/D2 回执关联（§4.7）；签发时点可空
        "dcc_class":  dcc_class,
        "challenge":  c,                                  # 全对象嵌入（含 disclosure_hash）
        "evidence_rules_version": d["evidence_rules_version"],
        "session_ref": current_session_id(),              # 频控计数用（§10.1.2）
        "issue_time": now_iso8601(),
        "expire_time": now_iso8601() + GRANT_TTL_SECONDS
    }
    # attested_by 判定（§3.5 / 附录 B 升级阶梯）：
    #   DCC-C → "app"（应用自证，LoA 封顶 L1.5）
    #   DCC-B → "device"（设备端密钥签名，L1.5 可标设备自证；本文件仅示范调用位置，
    #           不声明硬件已具备——DCC-B 路径为 §12「设计意图」）
    g["attested_by"] = (dcc_class == "B" ? "device" : "app")

    g["signature"] = sign(key_of(g["attested_by"]),
                          JCS(g 减去 signature 字段))      # 签名覆盖全部字段除自身（§4.2）
    g["verify_key_fingerprint"] = fingerprint_of(key_of(g["attested_by"]))
    # 验证公钥随凭证分发：争议方无需回连注册表即可验签（§4.2）

    persist_audit(g)                                          # 判责链第 3 段：确认留痕
    return g
```

## 6. Receipt 对齐（§4.5）

```
function on_action_executed(execution_record):
    r = {
        "grant_id":           g["grant_id"],
        "executed_action_hash": b64url(SHA256(JCS(execution_record.action_core))),
        "disclosure_hash":    d["disclosure_hash"],
        "executed_at":        now_iso8601(),
        "result":             execution_record.result
    }
    # 对齐判定：动作、主体、数额/限额在 scope 对齐规则容差内一致（§4.5，非裸 hash 相等）
    if aligns(r["executed_action_hash"], d, rules):
        r["alignment"] = "ALIGNED"; state = "receipted"    # TOCTOU 闭合（判责链第 4 段）
    else:
        r["alignment"] = "VIOLATED"; mark_audit("越权执行")   # 不对齐 = 凭证链不为其提供授权证明（§4.5/§11.1）
    persist_audit(r)
    submit_to_custodian([ d["disclosure_hash"], g["grant_id"], r["executed_action_hash"] ])
    # §8 托管提交：仅 hash 与时间戳，禁止 do_id 与任何原文要素
    return r
```

## 7. 频控与状态机骨架（§4.4 / §10.1.2，示意）

```
on session_start():
    confirm_count = 0

function confirm_frequency_guard(action):
    if action.is_meta_confirmation:         # 元确认（如 Access Grant 签发确认）
        return ALLOW                        # 不占会话配额（§10.1.2）
    confirm_count += 1
    if confirm_count > CONFIRM_PER_SESSION_MAX:
        force_full_disclosure_replay()      # 超阈值：整段披露重放，短播报通道熔断
        return THROTTLED
    return ALLOW

on same_disclosure_re-request(hash, dt):
    if dt < 30s:   mark_audit("Agent 违规：30 秒内重复请求确认")   # §10.1.2
    if dt < T:     skip_rebroadcast(); require_physical_confirm()  # §10.1.3 豁免语义：免播报不免按键
    else:          restart_from(step 2)                      # 超出 T 窗口：完整重放
```

## 8. Delivery Receipt 生成（D1；§4.7.1；v0.2 新增）

```
function generate_delivery_receipt(d, render_session):
    # 调用时机：弱表面渲染完成时（DCC-B 以上端点；DCC-C 无设备签名通道，回落 D0 渲染留痕——§4.7.3）

    nonce = CSPRNG(RECEIPT_NONCE_BITS)                     # per-receipt nonce ≥128bit（隐私红线，§4.7.1）
                                                           # 低熵披露裸 hash 可字典重建交易内容（§11.1）——
                                                           # nonce 入凭证、托管方禁录 nonce↔内容对应关系（§8）

    content_hash = b64url(SHA256(JCS(render_session.rendered_disclosure) || nonce))
                                                           # hash 计算混入 nonce（§4.7.1 结构）

    # —— 时长证明力分层（§4.7.1，red-team E-2/P-2 修订）——
    t_start = secure_clock_now()                           # 安全时间源路径（§5.1 secure_clock 能力位）
    ... 渲染 ...
    t_end   = secure_clock_now()
    if t_start != UNSUPPORTED and t_end != UNSUPPORTED:
        duration_proof = {"render_start": t_start, "render_end": t_end}   # 双时戳入证明对象
    else:
        duration_proof = {"render_duration_ms": render_session.duration}  # claimed 字段：参与 hash、不参与证明
                                                           # "渲染 1 秒签成 8 秒"——自报时长不可作证明对象

    r1 = {
        "receipt_type":  "delivery",
        "receipt_id":    "urn:uuid:" + uuid4(),            # 高熵 receipt_id：托管 hash 的熵前提之一（§8/C.8）
        "device_id_cert": device_cert_chain(),             # DCC-B 以上证书链
        "grant_ref":     (g exists ? g["grant_id"] : null),# 可空（§4.7.1）：先播报后确认/浏览未确认 → null
                                                           # 空值语义＝"披露送达但确认未发生"（不承载同意语义）
        "content_hash":  content_hash,
        "schema_ref":    d["schema_ref"],
        "nonce":         b64url(nonce),                    # 随凭证存放（§4.7.1 隐私红线）
        "render_start":  duration_proof.render_start,      # claimed 路径下仅 start + duration_ms
        "render_duration_ms": duration_proof["render_duration_ms"] 或 duration(t_start, t_end),
        "signature":     sign(device_key, JCS(r1 减去 signature 字段))
    }
    persist_audit(r1)
    if g exists: g["delivery_receipt_ref"] = r1["receipt_id"]   # 回填关联（重签场景见 §9 串链）
    submit_to_custodian([ r1["content_hash"], r1["receipt_id"] ])   # 回执哈希入托管（§8/C.8 采纳）
    return r1
```

## 9. Transit Fidelity 校验（D2；§4.7.2；v0.2 新增）

```
function render_with_fidelity(source_payload, source_anchor):
    # 链路前提：渲染控制方 ≠ 披露生成方（B 端中转/手机主 Agent 场景，§4.7.2）
    # source_anchor：源平台的托管留痕引用（完整托管路径→采信级；仅时间戳路径→限审计参考，§4.7.2 仲裁基准）

    rendered = render(source_payload.disclosure)           # 中转方渲染
    nonce = CSPRNG(RECEIPT_NONCE_BITS)                     # D2 同守 nonce 隐私红线（§4.7.2）

    r2 = {
        "receipt_type":         "transit_fidelity",
        "receipt_id":           "urn:uuid:" + uuid4(),
        "device_id_cert":       device_cert_chain(),
        "source_disclosure_hash": source_payload.source_hash,     # 源平台生成
        "source_anchor_ref":    source_anchor.ref,
        "rendered_content_hash": b64url(SHA256(JCS(rendered) || nonce)),
        "render_agent_id":      current_agent_id(),           # KYA 对接点
        "render_timestamp":     secure_clock_now() 或 now(),
        "fidelity_verdict":     null,                          # 待比对
        "prior_receipt_id":     current_chain.head,            # 重渲染串链（§4.7.2）——链首为 null
        "signature":            sign(device_key, JCS(r2 减去 signature 字段))
    }

    # —— 比对基准（§4.7.2 仲裁基准，red-team L-5 修订）——
    baseline = custodian_lookup_latest_anchor_before(
                   source_payload.source_hash, r2["render_timestamp"])
                   # 一律以"渲染时点前最新的托管锚"为基准——
                   # 防源平台事后重铸新版本哈希、把偷换伪装成正常 match

    r2["fidelity_verdict"] = (r2["rendered_content_hash"] == baseline ? "match" : "mismatch")
    persist_audit(r2)

    if r2["fidelity_verdict"] == "mismatch":
        mark_chain(r2["prior_receipt_id"], "VIOLATED")     # 该链状态 VIOLATED（§4.7.2）
        mark_audit("中转保真 mismatch：中转方或源平台改动待判")   # 判责归属：改的一方持对方证据反证（§4.7.2）
        require_new_disclosure_chain()                     # 后续披露须新开链；部分字段更新走增量重渲染＋全量重签
        return r2                                          # VIOLATED 链不进入确认环节（与 rejected_schema 同构的硬闸）

    if source_anchor.mode == "timestamp_only":
        mark_audit("回执限审计参考（仅时间戳路径，不得作采信级中转保真主张）")   # §4.7.2 分级
    return r2
```

## 诚实边界（与 §12 一致）

- 本文件为伪代码：`persist_audit` / `broadcast` / `route_*` / `custodian_lookup_*` 均为占位调用，无真实实现。
- `attested_by: "device"`（DCC-B 路径）与 `secure_clock_now()`（安全时间源）仅示范调用位置——设备端签名与 TEE 保护区时钟为 §12「设计意图」，当前无硬件具备。
- §8/§9 两个 v0.2 新增段落为**语义设计**级（规范 §12 实现状态声明：D1/D2 凭证结构＝规范定义，工程实现未开始）——本文件展示字段流与判定逻辑，不构成"已实现"声明。
- 托管方提交在试点期为降级单日志模式，降级状态须公开声明（§8 独立性条款）；回执哈希入托管（C.8）的熵前提：计算输入含高熵 `receipt_id` 与 per-receipt nonce（§8）。
- N/A 滥用的**语义**抽查（显易不适用却声称豁免）由托管方准入审计承担——签发端只做结构校验（缺失/空串/空理由），两者分工不得混淆（§6.3）。
- 任何实现者动手写代码前应先读规范 §9 证据规则与 §11.1 残余风险表——本文件不替代规范。
