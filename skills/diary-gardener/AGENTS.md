# diary-gardener

## 这是什么

让 AI 当 Obsidian 笔记管家：一句话「整理日记」，自动打标签、提炼知识卡片、挂 MOC 主题地图、建立双链，并用 `processed` 标记防止重复整理。支持周回顾和 Inbox 清理。

## 何时激活

- 中文触发：整理日记 / 整理昨天的日记 / 日记加工 / 周回顾 / 整理 Inbox
- 英文触发：process my daily notes / weekly review

## 前置要求

- Obsidian 1.13+（含官方 CLI），且处于运行状态
- vault 采用四区结构：`00 Inbox` / `10 Calendar`（日记）/ `20 Atlas`（卡片+MOC）/ `30 Efforts`（项目）/ `40 Archive`
- Claude Code（或任何读取本文件的 agent 工具）

## 怎么用

完整流程、目录约定、标签白名单和 CLI 踩坑清单见同目录 **SKILL.md**（本文件是跨工具入口，SKILL.md 是完整规范）。

主页：https://github.com/assddd81/diary-gardener（MIT）
