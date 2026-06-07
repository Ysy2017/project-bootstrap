# project-bootstrap

> 一个 [Claude Code](https://claude.com/claude-code) skill —— 用「**规划优先 + 分步实现**」把模糊 idea / 半成品 / 待重构,变成**可执行的规划文档 + 一条「一步一验证」的实现流水线**。

---

## 解决什么痛点

AI 编码爽,但放任不管会翻车。这个 skill 针对四类常见问题给了机制:

| 痛点 | 机制 |
|---|---|
| AI 自主乱规划、跑偏 | **规划优先**:先出 PRD / 计划,人过一遍才动手 |
| 长对话上下文爆、换会话丢状态 | **状态落盘 → 对话可弃**:真相在文件不在窗口,随时 `/clear` 零损失 |
| 越权改 / 假绿谎报 / 幻觉膨胀 / 回归 | **护栏 + 每步自审清单 + 人工确认门** |
| 多会话接力、封存恢复无歧义 | **封存 / 恢复双闸 + 三方对账** |

## 核心理念(3 条)

1. **规划优先** —— 不让 AI 自主乱规划。
2. **状态落盘、对话可弃** —— 真相在 `progress.md` / `architecture.md` 等文件里,不在上下文窗口里,所以对话随时能扔、能换。
3. **子代理隔离** —— 每步的重活在一次性子代理里干,只回窄摘要,主线不被污染。

## 何时用 / 何时别用

- ✅ **适合**:多步骤、跨文件、需要留档的工作 —— 新建项目、续做半成品、重构。
- ❌ **别用**:一行小改、临时脚本、单文件 bugfix —— 上整套(澄清 → 文档 → 子代理 → 门)是杀鸡用牛刀。

---

## 安装

把本仓库 clone 进 Claude Code 的 skills 目录即可(目录名必须是 `project-bootstrap`):

**macOS / Linux**
```bash
git clone git@github.com:Ysy2017/project-bootstrap.git ~/.claude/skills/project-bootstrap
```

**Windows (PowerShell)**
```powershell
git clone git@github.com:Ysy2017/project-bootstrap.git "$env:USERPROFILE\.claude\skills\project-bootstrap"
```

装好后重启 / 刷新 Claude Code,skill 即可被识别。

## 怎么用

Claude Code 会按 skill 描述**自动识别触发短语**,例如:

> 「开个新项目 / 新模块」「搭项目脚手架」「用 vibe coding 流程做 X」「把 idea 落成可实现的计划」「**继续做这个项目**」「封存 / 开新对话」

也可显式调用 `/project-bootstrap`。

触发后 skill 先**自动判断模式**(拿不准才问):

- **新建**(空目录 / 只有 idea)→ 走完整 Phase 0→4。
- **续做**(已有代码、做一半)→「续做对接」后进 Phase 3。
- **重构**(改现有代码、不能破坏现有行为)→「续做对接」+ **锁基线**(先补特征测试锁住当前行为)。

---

## 工作流概览

```
Phase 0  澄清        提 5~9 个关键问题,等回答再生成
Phase 1  规划文档    PRD / tech-stack / implementation-plan(只 MVP,每步带验证标准)
Phase 2  Always 规则 写进 CLAUDE.md(读架构再写码、模块化、改配置提示 .env…)
Phase 3  分步循环    子代理实现 → 人工确认门 → 跑验证 → commit → 下一步
Phase 4  增强打磨    每个增强单独 feature 文档,走 Phase 3 同款循环
```

围绕主流程还有几套横切机制:

- **分支生命周期**:分支 = 里程碑,让 **分支 / review / 封存三边界对齐** —— 里程碑开始从 master 开分支 → 每步 commit → 做完 review(`branch vs master`)→ 合 master → 提封存。命名缺则当场建一条落进 CLAUDE.md。
- **上下文管理**:`/context` 低于 ~35% 就在干净边界 checkpoint-and-clear;子代理只回结论不灌长日志。
- **当场落盘**:每做出 / 否决一个决定,顺手 append 一行到 `progress.md`「未整理决策」区,封存时再批量整理。
- **里程碑合并前独立 review**:合 master 前**新开窗口**跑 `/code-review`(干净视角),findings 拿回来分诊 → 改 → 重测 → 全过才合。
- **封存 / 恢复双闸**:判据是「只凭盘上文件 + git,能否让没参与的 agent 无歧义重建当前状态?」

## 产物:5 份职能文档

`memory_bank` 是**角色名**,不强制文件名。已有项目的等价文档(根目录的 PRD / CONTEXT / DECISIONS 等)**复用、不重造**;真没有才新建 `memory_bank/`。

```
PRD.md                  要做什么、给谁、成功标准。不写技术
tech-stack.md           技术选型 + 为什么(最简但稳健)
implementation-plan.md  分步计划:每步小而具体、带「验证标准」、不写代码
progress.md             滚动:每步做完写一条 + 末尾「未整理决策」草稿区
architecture.md         活文档:每个文件 / 模块干什么、数据结构 / schema
```

---
