---
name: commit-helper
description: 在提交前按当前仓库约定分析变更，优先按领域拆分为多个原子 commit，并生成可直接使用的 git commit message。
disable-model-invocation: false
argument-hint: [附加变更背景]
---

请基于当前仓库的变更，为即将执行的提交生成符合仓库约定的 commit message。

本仓库的提交辅助规则以 `RULES.md` 为唯一来源，请先读取并严格遵循 `RULES.md`。

用户补充背景：
`$ARGUMENTS`
