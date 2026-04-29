# AnyWrite

这是整理后的 README 版本，命名统一为 **AnyWrite**，命令统一为 `anywrite`，环境变量统一为 `ANYWRITE_`。

------

# 核心特性

## 多维度审计与去 AI 味

AnyWrite 内置连续性审计员，会从 **33 个维度**检查每一章草稿，包括：

| 审计维度   | 检查内容                                         |
| ---------- | ------------------------------------------------ |
| 角色记忆   | 角色是否知道了自己不该知道的信息                 |
| 物资连续性 | 物品、金钱、武器、资源是否前后一致               |
| 伏笔回收   | 已埋伏笔是否推进，是否长期悬空                   |
| 大纲偏离   | 当前章节是否偏离故事主线和卷纲                   |
| 叙事节奏   | 是否拖沓、跳跃、重复或过度总结                   |
| 情感弧线   | 角色情绪变化是否自然、连贯                       |
| AI 痕迹    | 是否出现高频模板词、句式单调、总结腔、LLM 味表达 |

如果审计未通过，系统会自动进入：

```bash
修订 -> 再审计 -> 再修订
```

直到所有关键问题清零。

去 AI 味能力并不只发生在后处理阶段。AnyWrite 会在写手 Agent 的 prompt 层提前注入规则，包括：

- 词汇疲劳词表
- 禁用句式
- 文风指纹
- 对话比例约束
- 叙事节奏约束
- 反模板化表达规则

已有章节也可以使用专门的反检测改写模式：

```bash
anywrite revise --mode anti-detect
```

------

## 文风仿写

AnyWrite 支持从参考文本中提取文风指纹，并将其注入指定书籍。

```bash
anywrite style analyze reference.txt
anywrite style import 吞天魔帝 style.json
```

分析内容包括：

| 指纹类型     | 说明                                 |
| ------------ | ------------------------------------ |
| 句长分布     | 长短句比例、节奏密度                 |
| 词频特征     | 高频词、常用表达、语气倾向           |
| 节奏模式     | 描写、动作、对白、心理活动的比例     |
| LLM 风格指南 | 可供 Writer / Reviser 使用的风格约束 |

导入后，后续章节会自动采用该文风；修订者也会以该风格标准作为审计依据。

------

## 创作简报

可以通过创作简报把作者已有的脑洞、世界观、人设、设定文档传给 AnyWrite。

```bash
anywrite book create --brief my-ideas.md
```

建筑师 Agent 会基于简报生成：

| 文件                     | 作用                         |
| ------------------------ | ---------------------------- |
| `story_bible.md`         | 故事设定、世界观、基本规则   |
| `book_rules.md`          | 创作规则、角色限制、题材禁忌 |
| `story/author_intent.md` | 长期作者意图                 |

这意味着 AnyWrite 不是凭空创作，而是基于你的输入建立整本书的长期创作方向。

`author_intent.md` 会持续参与后续写作，不会只在建书时生效一次。

------

## 输入治理控制面

每本书都有两份长期可编辑的 Markdown 控制文档：

| 文件                     | 作用                             |
| ------------------------ | -------------------------------- |
| `story/author_intent.md` | 这本书长期想成为什么             |
| `story/current_focus.md` | 最近 1 到 3 章要把注意力拉回哪里 |

写作前可以先运行：

```bash
anywrite plan chapter 吞天魔帝 --context "本章先把注意力拉回师徒矛盾"
anywrite compose chapter 吞天魔帝
```

执行后会生成运行时产物：

| 文件                                         | 用途                                 |
| -------------------------------------------- | ------------------------------------ |
| `story/runtime/chapter-XXXX.intent.md`       | 本章目标、保留项、避免项，供人类审阅 |
| `story/runtime/chapter-XXXX.context.json`    | 本章实际选入的上下文                 |
| `story/runtime/chapter-XXXX.rule-stack.yaml` | 本章规则优先级与覆盖关系             |
| `story/runtime/chapter-XXXX.trace.json`      | 输入编译轨迹，便于调试               |

`plan` / `compose` 只编译本地文档和状态，不依赖在线 LLM。因此即使还没有配置 API Key，也可以先验证控制输入是否符合预期。

------

## 字数治理

`draft`、`write next`、`revise` 共享同一套保守型字数治理逻辑。

```bash
anywrite write next 吞天魔帝 --words 3000
```

治理规则：

- `--words` 表示目标字数，不承诺逐字精确命中。
- 系统会自动推导一个允许区间。
- 中文默认按 `zh_chars` 计数。
- 英文默认按 `en_words` 计数。
- 如果正文超出允许区间，AnyWrite 最多追加 1 次纠偏归一化。
- 纠偏方式包括压缩或补足，不会直接硬截断正文。
- 如果纠偏后仍超出 hard range，章节仍会保存，但会在结果和 chapter index 中留下 length warning / telemetry。

------

## 续写已有作品

AnyWrite 可以从已有小说文本导入章节，并自动逆向工程长期记忆。

```bash
anywrite import chapters source.txt
anywrite write next 吞天魔帝
```

导入时会自动生成 7 个真相文件：

| 文件     | 说明                             |
| -------- | -------------------------------- |
| 世界状态 | 当前世界局势、角色位置、已知信息 |
| 角色矩阵 | 角色关系、相遇记录、信息边界     |
| 资源账本 | 物品、金钱、资源、武器状态       |
| 伏笔钩子 | 未闭合伏笔、承诺、冲突           |
| 章节摘要 | 每章关键事件和状态变化           |
| 支线进度 | A/B/C 线进展、停滞检测           |
| 情感弧线 | 角色情绪变化和成长轨迹           |

支持：

- `第X章` 自动分割
- 自定义分割模式
- 断点续导
- 导入后无缝续写

------

## 同人创作

AnyWrite 支持从原作素材创建同人书。

```bash
anywrite fanfic init --from source.txt --mode canon
```

支持四种模式：

| 模式    | 说明     |
| ------- | -------- |
| `canon` | 正典延续 |
| `au`    | 架空世界 |
| `ooc`   | 性格重塑 |
| `cp`    | CP 向    |

同人模式内置：

- 正典导入器
- 同人专属审计维度
- 信息边界管控
- 设定冲突检查
- 原作事实一致性约束

用于确保同人创作不与既有设定冲突。

------

## 多模型路由

不同 Agent 可以使用不同模型和 Provider。

例如：

| Agent   | 推荐模型             |
| ------- | -------------------- |
| Writer  | Claude，适合创意生成 |
| Auditor | GPT-4o，适合快速审计 |
| Radar   | 本地模型，降低成本   |

可以按 Agent 粒度配置模型：

```bash
anywrite config set-model writer claude-3-5-sonnet
anywrite config set-model auditor gpt-4o
```

未单独配置的 Agent 会自动回退到全局模型。

------

## 守护进程与通知推送

可以启动后台循环自动写章：

```bash
anywrite up
```

管线行为：

- 非关键问题自动处理。
- 关键问题暂停，等待人工审核。
- 支持静默模式：

```bash
anywrite up -q
```

日志写入：

```bash
anywrite.log
```

日志格式为 JSON Lines，便于后续分析和接入外部系统。

通知推送支持：

| 渠道             | 支持情况 |
| ---------------- | -------- |
| Telegram         | 支持     |
| 飞书             | 支持     |
| 企业微信         | 支持     |
| Webhook          | 支持     |
| HMAC-SHA256 签名 | 支持     |
| 事件过滤         | 支持     |

------

## 本地模型兼容

AnyWrite 支持任何 OpenAI 兼容接口。

CLI 方式：

```bash
anywrite write next 吞天魔帝 --provider custom
```

环境变量方式：

```bash
ANYWRITE_LLM_PROVIDER=custom
```

Studio 中也可以新增自定义服务。

服务测试会自动尝试：

- 不同协议
- 流式 / 非流式开关
- 可用 transport
- 自定义 endpoint
- OpenAI-compatible API 变体

针对小模型不规范输出，AnyWrite 内置 fallback 解析器。流式输出中断时，也会尽量恢复已生成的部分内容。

------

## 可靠性保障

AnyWrite 在每章写作过程中提供多层可靠性保障。

| 机制          | 说明                                            |
| ------------- | ----------------------------------------------- |
| 状态快照      | 每章自动创建状态快照                            |
| 回滚重写      | `anywrite write rewrite` 可回滚任意章节         |
| 写前自检      | Writer 动笔前输出上下文、资源、伏笔、风险检查表 |
| 写后结算      | Writer 完成后输出章节结算表                     |
| 交叉验证      | Auditor 对 Writer 输出进行连续性审计            |
| 文件锁        | 防止并发写入导致状态损坏                        |
| 写后验证器    | 检查跨章重复和硬规则问题                        |
| 自动 spot-fix | 对 11 条硬规则进行自动修复                      |

伏笔系统使用 Zod schema 校验：

- `lastAdvancedChapter` 必须是整数。
- `status` 只能是 `open`、`progressing`、`deferred`、`resolved`。
- LLM 输出的 JSON delta 会先经过 `applyRuntimeStateDelta` 做 immutable 更新。
- 写入前必须通过 `validateRuntimeState` 结构校验。
- 坏数据会被直接拒绝，不会污染长期状态。

模型输出上限由 provider bank 的模型卡管理。

`llm.extra` / `ANYWRITE_LLM_EXTRA_*` 中的保留键会被自动过滤，包括：

```text
max_tokens
temperature
model
messages
stream
```

用于防止外部配置意外覆盖核心请求参数。

------

# 工作原理

AnyWrite 的每一章由多个 Agent 接力完成，全程可以零人工干预。

## 管线流程

| Agent                | 职责                                                         |
| -------------------- | ------------------------------------------------------------ |
| Radar 雷达           | 扫描平台趋势和读者偏好，指导故事方向。可插拔，可跳过。       |
| Planner 规划师       | 读取作者意图、当前焦点、记忆检索结果，产出本章意图，包括 must-keep / must-avoid。 |
| Composer 编排师      | 从全量真相文件中按相关性选择上下文，编译规则栈和运行时产物。 |
| Architect 建筑师     | 规划章节结构、大纲、场景节拍和节奏控制。                     |
| Writer 写手          | 基于编排后的精简上下文生成正文，执行字数治理和对话引导。     |
| Observer 观察者      | 从正文中结构化提取 9 类事实：角色、位置、资源、关系、情感、信息、伏笔、时间、物理状态。 |
| Reflector 反射器     | 输出 JSON delta，而不是完整 Markdown；由代码层完成 Zod schema 校验和 immutable 写入。 |
| Normalizer 归一化器  | 单 pass 压缩或扩展，将章节字数拉入允许区间。                 |
| Auditor 连续性审计员 | 对照 7 个真相文件验证草稿，执行 33 维度检查。                |
| Reviser 修订者       | 修复审计发现的问题；关键问题自动修复，其他问题标记给人工审核。 |

如果审计不通过，管线会自动进入：

```text
修订 -> 再审计 -> 再修订
```

直到所有关键问题清零。

------

# 长期记忆

每本书维护 7 个真相文件，作为唯一事实来源。

| 文件                   | 用途                                             |
| ---------------------- | ------------------------------------------------ |
| `current_state.md`     | 世界状态：角色位置、关系网络、已知信息、情感弧线 |
| `particle_ledger.md`   | 资源账本：物品、金钱、物资数量及衰减追踪         |
| `pending_hooks.md`     | 未闭合伏笔：铺垫、对读者的承诺、未解决冲突       |
| `chapter_summaries.md` | 各章摘要：出场人物、关键事件、状态变化、伏笔动态 |
| `subplot_board.md`     | 支线进度板：A/B/C 线状态、停滞检测               |
| `emotional_arcs.md`    | 情感弧线：按角色追踪情绪变化和成长               |
| `character_matrix.md`  | 角色交互矩阵：相遇记录、信息边界                 |

连续性审计员会对照这些文件检查每一章草稿。

例如：

- 角色是否“记起”了自己从未亲眼见过的事。
- 主角是否拿出了两章前已经丢失的武器。
- 伏笔是否被遗忘。
- 支线是否长期停滞。
- 情感变化是否突兀。
- 资源消耗是否前后不一致。

从 `0.6.0` 起，真相文件的权威来源从 Markdown 迁移到：

```text
story/state/*.json
```

这些结构化状态文件使用 Zod schema 校验。

Markdown 文件仍然保留，但主要作为人类可读的投影。旧书首次运行时会自动从 Markdown 迁移到结构化 JSON，无需人工操作。

在 Node 22+ 环境下，AnyWrite 会自动启用 SQLite 时序记忆数据库：

```text
story/memory.db
```

用于按相关性检索历史事实、伏笔和章节摘要，避免把所有历史内容一次性塞进上下文，降低上下文膨胀问题。

------

# 控制面与运行时产物

除了 7 个真相文件，AnyWrite 还把“护栏”和“自定义输入”拆成可审阅的控制层。

| 文件                                         | 作用                               |
| -------------------------------------------- | ---------------------------------- |
| `story/author_intent.md`                     | 长期作者意图                       |
| `story/current_focus.md`                     | 当前阶段关注点                     |
| `story/runtime/chapter-XXXX.intent.md`       | 本章目标、保留项、避免项、冲突处理 |
| `story/runtime/chapter-XXXX.context.json`    | 本章实际选入的上下文               |
| `story/runtime/chapter-XXXX.rule-stack.yaml` | 本章规则优先级和覆盖关系           |
| `story/runtime/chapter-XXXX.trace.json`      | 本章输入编译轨迹                   |

这样 brief、卷纲、书级规则、当前任务不会混成一坨 prompt，而是先编译，再写作。

------

# 创作规则体系

Writer Agent 内置约 25 条通用创作规则，覆盖：

- 人物塑造
- 叙事技法
- 逻辑自洽
- 对话节奏
- 情绪推进
- 信息披露
- 语言约束
- 去 AI 味
- 节奏控制
- 章节钩子

在通用规则基础上，每个题材还可以有专属规则，包括：

- 题材禁忌
- 语言约束
- 爽点节奏
- 审计维度
- 结构偏好
- 读者期待

每本书还拥有独立配置：

| 文件                | 说明                           |
| ------------------- | ------------------------------ |
| `book_rules.md`     | 主角人设、数值上限、自定义禁令 |
| `story_bible.md`    | 世界观设定                     |
| `author_intent.md`  | 长期方向                       |
| `current_focus.md`  | 近期关注点                     |
| `volume_outline.md` | 默认卷纲规划                   |

在 v2 输入治理模式下，`volume_outline.md` 不再天然压过当前任务意图。当前任务、作者意图和书级规则会通过 rule-stack 显式编译和排序。

------

# 使用模式

AnyWrite 提供三种交互方式，底层共享同一组原子操作。

------

## 1. 完整管线

适合一键写作。

```bash
anywrite write next 吞天魔帝
```

执行流程：

```text
写草稿 -> 审计 -> 自动修订 -> 保存章节 -> 更新长期记忆
```

连续写 5 章：

```bash
anywrite write next 吞天魔帝 --count 5
```

`write next` 默认走：

```text
plan -> compose -> write
```

也就是 v2 输入治理链路。

如果需要回退到旧的 prompt 拼装路径，可以在 `anywrite.json` 中显式设置：

```json
{
  "inputGovernanceMode": "legacy"
}
```

默认值为：

```json
{
  "inputGovernanceMode": "v2"
}
```

`legacy` 仅作为显式 fallback 保留。

------

## 2. 原子命令

适合外部 Agent、脚本系统或高级用户组合调用。

```bash
anywrite plan chapter 吞天魔帝 --context "本章重点写师徒矛盾" --json
anywrite compose chapter 吞天魔帝 --json
anywrite draft 吞天魔帝 --context "本章重点写师徒矛盾" --json
anywrite audit 吞天魔帝 31 --json
anywrite revise 吞天魔帝 31 --json
```

每个命令只执行单一职责：

| 命令      | 职责                 |
| --------- | -------------------- |
| `plan`    | 规划本章意图         |
| `compose` | 编排上下文和规则栈   |
| `draft`   | 生成正文草稿         |
| `audit`   | 审计章节质量和连续性 |
| `revise`  | 修订章节问题         |

`--json` 会输出结构化结果，便于外部 AI Agent 通过 `exec` 调用，也适合脚本编排。

------

## 3. 自然语言 Agent 模式

适合直接用自然语言驱动 AnyWrite。

```bash
anywrite agent "帮我写一本都市修仙，主角是个程序员"
anywrite agent "写下一章，重点写师徒矛盾"
anywrite agent "先扫描市场趋势，然后根据结果创建一本新书"
```

内置工具包括：

| 工具                   | 作用             |
| ---------------------- | ---------------- |
| `write_draft`          | 写章节草稿       |
| `plan_chapter`         | 规划章节         |
| `compose_chapter`      | 编排章节输入     |
| `audit_chapter`        | 审计章节         |
| `revise_chapter`       | 修订章节         |
| `scan_market`          | 扫描市场趋势     |
| `create_book`          | 创建新书         |
| `update_author_intent` | 更新长期作者意图 |
| `update_current_focus` | 更新近期关注点   |
| `get_book_status`      | 获取书籍状态     |
| `read_truth_files`     | 读取真相文件     |
| `list_books`           | 列出书籍         |
| `write_full_pipeline`  | 执行完整写作管线 |
| `web_fetch`            | 获取网页内容     |
| `import_style`         | 导入文风         |
| `import_canon`         | 导入正典素材     |
| `import_chapters`      | 导入已有章节     |
| `write_truth_file`     | 写入真相文件     |

推荐 Agent 工作流：

```text
先调整控制面
-> plan / compose
-> 决定写草稿或运行完整管线
-> 审计
-> 修订
-> 更新长期记忆
```

------

# 一句话总结

**AnyWrite 是一个面向长篇小说创作的多 Agent 写作系统。**

它不是简单地“让模型写下一章”，而是把小说创作拆成：

```text
作者意图
-> 输入治理
-> 上下文编排
-> 写作
-> 审计
-> 修订
-> 长期记忆更新
```

通过长期记忆、连续性审计、文风指纹、去 AI 味规则、字数治理和多模型路由，让长篇连载具备更强的一致性、可控性和可维护性。
