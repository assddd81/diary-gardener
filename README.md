# diary-gardener 🌱

**让 AI 当你的 Obsidian 笔记管家** —— 一个为 [Claude Code](https://claude.com/claude-code) 打造的技能（Skill）：你说一句「整理日记」，它负责打标签、提炼知识卡片、维护主题地图（MOC）、建立双链，并自动防止重复整理。

## 它解决什么问题

大多数人的 Obsidian 是一座"笔记坟场"：记了 700 篇笔记，86% 从未被链接、从未被再次找到。传统方法（Zettelkasten、PARA）要求你手工加工每一条笔记——一张卡片 20 分钟，没人坚持得下来。

**diary-gardener 把"加工"这个环节自动化了**：你只管用最随意的方式记日记（关键词、半句话都行），AI 负责后续一切整理工作。

## 工作流程

```
你写日记（每天 2 分钟，关键词即可）
   └─> 你对 Claude Code 说「整理日记」
         └─> AI 读取日记，先给出整理计划，你确认后执行：
               ├─ 打标签（按白名单，可自定义）
               ├─ 有价值的内容提炼成知识卡片（用你的话重写，不是复制）
               ├─ 挂到对应的 MOC 主题地图
               ├─ 建立 [[双链]] 指向相关笔记
               └─ 标记 processed: true（幂等，绝不重复整理）
```

默认**计划先行**：AI 先列出"打算打什么标签、建什么卡片、挂哪个 MOC"，你确认才动手。跑顺之后一句话切全自动。

还支持：「周回顾」（汇总本周日记生成周报）、「整理 Inbox」（清空中转站）。

## 安装

### 方式一：手动安装（最简单）

把 `skills/diary-gardener/` 整个文件夹复制到：

- **Windows**: `C:\Users\你的用户名\.claude\skills\`
- **macOS / Linux**: `~/.claude/skills/`

### 方式二：git clone

```bash
git clone https://github.com/assddd81/diary-gardener.git
cp -r diary-gardener/skills/diary-gardener ~/.claude/skills/
```

### 方式三：skills CLI

```bash
npx skills add assddd81/diary-gardener
```

## 前置要求

1. [Obsidian](https://obsidian.md) **1.13+**（内置官方 CLI）
2. [Claude Code](https://claude.com/claude-code)
3. 推荐搭配 [kepano/obsidian-skills](https://github.com/kepano/obsidian-skills)（Obsidian CEO 维护的官方技能包）一起安装，让 AI 更懂 Obsidian 方言
4. 使用时 **Obsidian 需处于运行状态**（CLI 操作的是运行中的实例）

## 推荐目录结构

技能默认假设以下结构（可在 SKILL.md 里改成你自己的）：

```
你的笔记库/
├── 00 Inbox/       # 随手记中转站
├── 10 Calendar/    # 日记（唯一每天要写的入口）
├── 20 Atlas/       # 知识卡片 + MOC 主题地图
├── 30 Efforts/     # 进行中的项目、会议材料
├── 40 Archive/     # 存档
├── 90 Templates/   # 模板
└── 98 附件/        # 附件
```

结构脱胎于 Nick Milo 的 [LYT 框架](https://www.linkingyourthinking.com/)，核心是"链接优先于搬移"——技能永不移动你的存量文件，只用 `[[双链]]` 建立秩序。

## 自定义

- **标签白名单**：技能从库里《使用说明》文档读取标签体系；新标签会先向你提议
- **建卡标准**：自己的想法/判断 > 会议敲定事项 > 反复出现的话题；纯流水账不建卡
- **整理模式**：默认计划先行；说一句"以后全自动"即切换

## 已知问题

- Windows 中文环境下 `obsidian property:set` 偶发静默失败（CLI bug）——技能已内置回退方案（直接编辑 frontmatter，Obsidian 会热加载）
- 整理某篇日记时，请避免同时在 Obsidian 里编辑同一篇（避免写入竞争）

## 致谢

- 方法论：[LYT / Nick Milo](https://www.linkingyourthinking.com/)、Zettelkasten（Luhmann）
- 官方技能包：[kepano/obsidian-skills](https://github.com/kepano/obsidian-skills)

## License

MIT
