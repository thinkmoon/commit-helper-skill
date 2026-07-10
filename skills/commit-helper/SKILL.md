---
name: commit-helper
description: 使用最低成本的可用 subagent 分析当前仓库变更，按领域拆分原子 commit，生成 conventional commit message，并在用户明确要求提交时完成 git commit。当用户要提交代码、检查提交拆分、生成或修正 commit message 时使用。
---

将提交工作委托给一个 subagent，主 agent 只负责调度、传递用户背景和汇报结果。

## Subagent 调度

1. 只启动一个 subagent，选择当前运行时可用的最低成本、最轻量模型。
2. 如果 subagent 接口支持指定模型，显式选择最廉价的可用档位；如果不支持，使用默认 subagent，不在主 agent 中重复完成提交分析。
3. 给 subagent 传递完整任务：用户补充背景、当前仓库路径、是否授权实际提交，以及下方全部提交规则。
4. 要求 subagent 从仓库中自行读取 status、diff、历史提交风格和项目约定；不要由主 agent 预先总结 diff，避免遗漏。

## 提交规范
- 优先使用约定式提交：
```txt
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```
- `type` 使用：`feat`、`fix`、`refactor`、`docs`、`test`、`chore`
- 提交说明使用中文描述**变更目的**，专业术语保留英文
- 保持原子化提交：一次 commit 只包含一个逻辑变更

## Subagent 任务

1. 先查看当前变更，优先读取 staged diff；如果没有 staged 变更，再看工作区变更
2. 先做“是否需要拆分提交”的判断，而不是默认把所有变更合并成 1 次提交
3. 按业务领域、子系统边界、变更目的评估是否属于不同原子变更；只要跨领域、跨模块且可以独立提交，就应拆分
4. 常见应拆分场景包括但不限于：服务端与客户端、算法与接口层、测试与生产代码、重构与功能修复、文档与代码、不同目录下互不依赖的业务变更
5. 如果变更其实服务于同一个目的，即使涉及多个文件，也应保持为 1 次提交，不要机械按文件数拆分
6. 对每个建议提交分别判断最合适的 `type`，必要时补充简短 `scope`
7. 直接产出可用于提交的 commit message；如果需要拆分，按提交顺序给出多条 message，并明确每条对应的文件或变更范围
8. 只有在用户已经提供 commit message 草稿时，才做规范性修正
9. 用户明确要求“提交”时，按原子边界使用 `git add -- <paths>` 精确暂存，然后逐个执行 `git commit`
10. 提交前运行与变更相关且成本合理的验证；如果验证失败，停止提交并报告原因
11. 提交后使用 `git status --short` 和 `git log -n <commit-count> --oneline` 核对结果

## 安全边界

- 只有用户明确要求执行提交时才运行 `git commit`；如果只要 commit message 或拆分建议，不修改仓库状态
- 不要暂存、修改、覆盖或删除与本次目标无关的现有变更
- 不使用 `git add .` 或 `git add -A`；必须按每个原子提交精确指定路径
- 不使用破坏性 Git 命令，不跳过 hooks，不修改历史
- 不执行 `pull`、`push` 或其他远程操作，除非用户明确要求

## 输出约束
- 以可直接提交的 commit message 为核心输出
- 不要输出与提交规范无关的长篇说明
- `subject` 保持简洁，聚焦“为什么改”而不是流水账式“改了什么”
- 如果无法判断 `scope`，可省略
- 如果建议拆分提交，输出格式优先为：`Commit 1` / `Commit 2` ...，每项包含“建议纳入的文件或领域”与对应 commit message
- 如果判断不需要拆分，也要明确说明“建议单次提交”的原因
- 已执行提交时，返回 commit hash、message、包含的范围、验证结果和剩余未提交变更

用户补充背景：
`$ARGUMENTS`
