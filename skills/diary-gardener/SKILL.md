---
name: diary-gardener
description: 整理 Obsidian 日记的日常技能。读取日记 → 按白名单打标签 → 提炼知识卡片 → 挂 MOC → 建双链 → 规整待办 → 幂等标记。触发词：整理日记、整理昨天的日记、日记加工、处理日记、周回顾、weekly review、整理 Inbox。Use when the user asks to process/organize their Obsidian daily notes, do a weekly review of their vault, or tidy the 00 Inbox folder.
license: MIT
metadata:
  author: assddd81
  version: 1.1.0
  created: 2026-09-03
  last_reviewed: 2026-09-03
  review_interval_days: 90
  homepage: https://github.com/assddd81/diary-gardener
---

# Diary Gardener — Obsidian 日记整理技能

操作目标：用户当前的 Obsidian vault。本机路径用 `obsidian eval code="app.vault.adapter.basePath"` 动态获取（下文以 `<VAULT>` 代称）。所有操作通过 `obsidian` CLI 完成（要求 Obsidian 正在运行；不可用时提示用户打开 Obsidian）。

## 体系结构（必须遵守）

```
00 Inbox/      随手记中转站（必须定期清空）
10 Calendar/   日记 YYYY-MM-DD.md（type: daily, processed 属性）
20 Atlas/卡片/  知识卡片（本技能的主要产出）
20 Atlas/MOC/   主题地图（按用户实际主题组织）
30 Efforts/     项目、会议、事务（存量在这里，永远不要搬动）
40 Archive/     存档
90 Templates/   模板
98 附件/        随手附件
```

核心原则：**链接优先于搬移**——绝不移动 30 Efforts/40 Archive 里的存量文件，只用 `[[双链]]` 指向它们。

附件与产物放置规则：**随手粘贴的截图/图片放 `98 附件/`**（系统默认落点，不挪动）；**项目交付物（配图源文件、JSON 规范、HTML 产物、脚本代码）留在所属项目文件夹**，与内容同目录，保证项目自包含。往笔记里插图必须用真 embed：`![[98 附件/文件名.png]]` 或 `![[30 Efforts/项目/xxx/产物.png]]`（全路径最稳）。

## 标签白名单（先读 vault 里的「📖 使用说明（人与AI分工）」第六节，以那里为准）

`#会议` `#项目/<名>` `#人名（如 #张总）` `#想法` `#待办` `#资料` `#领域`

只从白名单选标签；需要新标签时先向用户提议，确认后同时更新使用说明第六节。

## 整理日记流程（触发：「整理日记」「整理昨天的日记」「整理 2026-09-01 的日记」）

### 1. 定位与幂等检查

```bash
obsidian daily:read                    # 今天；指定日期则 read path="10 Calendar/2026-09-01.md"
obsidian property:read name="processed" path="10 Calendar/2026-09-01.md"
```

`processed: true` → 报告"已整理过"，除非用户明确要求重做，否则停止。

### 2. 分析（不动手）

通读日记，识别：
- **会议/事件** → 应关联到哪个主题的 MOC（例会？某项目？）
- **人名** → 出现的关键人物（领导、同事、合作方…以 vault 实际人物为准）
- **项目线索** → 属于哪个项目（对照 MOC 与 30 Efforts 的项目文件夹名）
- **想法** → 值得沉淀成卡片的洞察（Statements 优先级最高）
- **待办** → `- [ ]` 项

### 3. 计划先行（默认模式）

输出整理计划清单，等用户确认：
- 打什么标签
- 建哪几张卡片（卡片名 + 一句话说清 says）
- 挂到哪个 MOC 的哪个章节
- 建哪些双链

用户明确开启全自动后，跳过确认直接执行。

### 4. 执行

```bash
# 建知识卡片（用自己的话说，不是复制原文；链接指回日记和相关存量文件）
# ⚠️ 必须用 path= 给出完整路径（含文件夹和 .md），create 没有 folder 参数，省略会建到 vault 根目录
obsidian create name="卡片名" path="20 Atlas/卡片/卡片名.md" silent content="---\ntype: card\nup:\n  - \"[[相关 MOC]]\"\ncreated: 2026-09-03\nsays: \"一句话核心观点\"\n---\n\n正文用自己的话重述，用 [[2026-09-01]] 指回日记，用 [[存量文件名]] 指向项目资料。\n"

# 挂 MOC（append 到对应章节；章节不合适时新建章节）
obsidian append file="某主题 MOC" content="- [[新卡片名]]"
```

### ⚠️ 实测已知坑（Obsidian CLI 1.13.7 / Win11 中文环境）

1. **`obsidian property:set` 稳定失败**（exit 127、无输出，无论 file= 还是 path=，无论文件是否在编辑器中打开）→ **改 frontmatter 一律用磁盘文件编辑**（Edit 工具直接改 `<VAULT>/10 Calendar/xxx.md` 的 YAML），Obsidian 会自动热加载外部修改，实测可靠。
2. **`obsidian create` 的 folder= 参数不存在**，用 `path="文件夹/文件名.md"` 全路径。
3. 中文路径偶发在 Bash 里被编码损坏 → 验证文件存在与否用 Glob/Read 工具，别依赖 bash head/cat 中文路径。
4. 日记的 tags 用 YAML 列表格式写：`tags:\n  - 想法\n  - 项目/项目A`。
5. **附件链接语法（实测踩坑：AI 插的图片在 Obsidian 里不显示）**：指向非 .md 文件的双链/embed **必须带扩展名**（`[[xxx.png]]`，省略扩展名会解析失败显示为断链）；图片要内联显示必须用 embed 感叹号语法 `![[...]]`，普通 `[[...]]` 只是文字链。最稳写法是 vault 全路径：`![[98 附件/截图.png]]`。`.html` 等不受支持的文件**永远不要用双链**（默认设置下解析不了），用纯文本提及文件名。
6. **`obsidian backlinks` 结果不可靠**（对日记文件常返回空，file= 和 path= 表现不一致）→ 验证链接关系用 eval 直查活索引：`obsidian eval code="JSON.stringify(Object.entries(app.metadataCache.resolvedLinks).filter(function(e){return e[1]['10 Calendar/2026-09-03.md']}).map(function(e){return e[0]}))"`，磁盘上有没有链接文本用 grep/Grep 工具复核。
7. **`obsidian search` 的 query 里不能含 `[[`**（报 "Property cannot be nested within a property"）→ 搜链接文本时去掉方括号只搜里面的词。

### 双链怎么落地（重要：写显式动作，不要只靠卡片模板隐式完成）

Obsidian 的双链 = **一处出链 + 自动反链**：卡片正文写了 `[[某笔记]]`，某笔记的反向链接面板就会自动显示这张卡片，双向关系即建立，不需要两边都写。但要保证三个**显式动作**：

1. **卡片出链**：建卡时正文必须含 `[[本篇日记日期]]` 指回日记 + `[[相关存量文件]]` 指向资料（若有）
2. **MOC 挂载**：append 让地图能走到卡片（上面命令）
3. **日记加指针**：在日记对应条目（那个待办、那句想法）后面追加 `[[卡片名]]`，让读日记的人正向可见、可跳转

整理完成的四件套：① 打标签（frontmatter tags 列表）② 建卡 + 挂 MOC ③ 日记对应条目追加 `[[卡片名]]` 指针 ④ processed: true。

## 建卡标准（什么值得变成卡片）

- ✅ 自己的想法/判断/决策依据（Statements）——最高优先级
- ✅ 会议里敲定的事项、领导要求（将来写材料要引用）
- ✅ 反复出现的话题（第 2 次出现就该有卡片了）
- ❌ 纯流水账、待办清单、情绪碎片——留在日记里就好

卡片正文要**用自己的话重写**，禁止整段复制日记原文；每张卡片至少 1 个指回日记的链接 + 1 个指向存量资料的链接（若有）。

## 周回顾流程（触发：「周回顾」「weekly review」）

1. 找出本周 `10 Calendar/` 下的日记（周一至今天）
2. 汇总：本周会议、项目进展、产生的卡片、未闭环的待办
3. 生成 `10 Calendar/YYYY-[W]ww 周记.md`（type: weekly），链接到各天日记
4. 汇报时给出"下周建议关注"2~3 条

## Inbox 清理流程（触发：「整理 Inbox」）

逐个读取 `00 Inbox/` 文件 → 判断归属（项目/会议/资料/卡片）→ `obsidian move` 到对应分区 → 挂 MOC → 汇报。Inbox 必须清空。

## 边界

- 永远不删除用户笔记；不确定价值的一律保留并询问
- 永远不搬动 30 Efforts 和 40 Archive 里的文件
- **隐私纪律：写进本文件与任何对外可见产物的示例，只用通用占位（张总、项目A、某主题 MOC），严禁出现真实人名、单位内部项目名与内部系统名**
- 修改前如果 vault 已接入 git，可先跑 `git -C <VAULT> status` 查看干净状态
