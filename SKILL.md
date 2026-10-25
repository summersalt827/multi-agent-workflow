---
name: multi-agent-workflow
description: WorkBuddy（设计/探索/沙箱跑通）+ Claude Code（合并/测试 gate/UI/CI/文档）+ 人（决策门）的多 Agent 协作流水线。腾讯云黑客松 Meridian Pay 项目验证过：1+5 会话、9→337 断言、单人队 2 周交付。触发词：「多agent协作」「WorkBuddy+Claude Code 流水线」「agent 协作开发」。
---

# 多 Agent 协作流水线（WorkBuddy × Claude Code）

## 何时用
- 新项目想用「一个 agent 写、另一个 agent 验、人拍板」的协作模式
- 单人/小团队要在短窗口（1-3 周）交付有测试背书的完整项目
- 需要把 AI 工具使用证据做成可展示的开发流程

## 三角色分工（固定）

| 角色 | 负责 | 红线 |
|---|---|---|
| 👤 人（决策门） | 定位叙事、语料/事实、每个取舍拍板 | Agent 不替你决定「做什么」，只决定「怎么做」 |
| 🟣 WorkBuddy（主力开发） | 一次一整段 prompt 建一个子系统，沙箱跑通全绿才导出 | 不直接改主仓库 |
| 🟠 Claude Code（合并+验收+打磨） | 取回补丁、跑全量测试 gate、UI/文档/CI 收口 | 不绿不进 main |

## 标准流程（一轮迭代）

1. **人** 定义本轮子系统边界 + 验收标准，写成一段 session prompt（存档到 `docs/build-sessions/`——prompt 即设计文档）
2. **WorkBuddy** 在沙箱建完，跑通相关测试
3. **导出**：沙箱内 `git add -N . && git diff --binary > xxx.patch`，或打 bundle
4. **取回**：`git fetch <bundle> HEAD`——⚠️ bundle ref 名是 HEAD 不是 main
5. **Claude Code** 合并 → 跑全量测试 gate → 修 UI/文档 → commit（合并 commit 带上 Claude co-author 行，诚实叙事）
6. **推 CI** 全量重跑 → 部署验证 → 红队/评审反馈写成下一段 session prompt，回到第 1 步

## 铁律（Meridian Pay 踩坑实录）

1. **不变量进 AGENTS.md 硬约束**——换 agent 干活前先写清「哪些是刻意设计不许改」。真实事故：self_ceiling 的 OOV 计入上界是压制无关查询的刻意设计，被当成 bug 改掉后 21 条标定全崩（无关分 0.15→0.73）
2. **数字必须三处一致**：README = 代码 = 测试输出，每次改动后跑 verify.py 级别的自验证脚本，评委/读者一 grep 就穿
3. **短 chunk 的 BM25 长度归一化红利**：小节标题撞上问题词会压过真答案——标题词与查询词刻意错开
4. **红队套件当 gate 的 gate**：注入/角色伪造/多跳拼图这类对抗测试单独成套，CI 每次 push 重跑
5. **每轮收尾产出下一轮 prompt**——反馈循环是流程的一部分，不是事后补

## 产出物清单（可展示的开发流程证据）
- session prompt 存档 + 截图（≥3 张，官方口径）
- 诚实分工文档（AI-COLLABORATION.md 模式：谁建了什么、工具痕迹为什么不一致）
- 测试增长曲线（9 → 337 这类数字很有叙事力）
- 流程图物料（本机模板：`~/Desktop/腾讯云黑客松/发布物料/` 的 HTML+shoot.py 可复用，改文字即可）
