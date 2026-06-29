# commit-helper-skill

`commit-helper` 是一个面向 AI 编程 agent 的提交辅助 skill。它会在提交前先分析当前仓库变更，判断是否应拆分为多个原子 commit，再生成可直接使用的 conventional commit message。

## 推荐安装入口

请把下面这个 GitHub 目录链接提供给支持从 GitHub 安装 skill 的 agent：

```txt
https://github.com/thinkmoon/commit-helper-skill/tree/main/skills/commit-helper
```

标准 skill 本体位于：

```txt
skills/commit-helper/
├── SKILL.md
└── agents/
    └── openai.yaml
```

仓库根目录不再保留兼容副本；新安装请统一使用 `skills/commit-helper/`。

## Codex 安装

在 Codex 中使用内置 `$skill-installer`，传入推荐安装入口：

```txt
$skill-installer https://github.com/thinkmoon/commit-helper-skill/tree/main/skills/commit-helper
```

安装后重启 Codex，或开启新会话让技能列表刷新。

Codex 的 GitHub 安装器也可以等价写成：

```bash
install-skill-from-github.py --repo thinkmoon/commit-helper-skill --path skills/commit-helper
```

## Claude Code / Cursor 安装

在 Claude Code 或 Cursor 的官方 skill 安装入口中，使用同一个 GitHub 目录链接：

```txt
https://github.com/thinkmoon/commit-helper-skill/tree/main/skills/commit-helper
```

如果工具要求选择仓库内路径，请选择：

```txt
skills/commit-helper
```

## 使用方式

安装后可以直接提问：

```txt
使用 commit-helper，根据当前变更判断是否需要拆分 commit，并生成 commit message
```

也可以补充背景：

```txt
使用 commit-helper，这次主要是修复登录超时，README 只是配套更新
```

常见场景：

- 提交前不确定当前改动是否应该拆成多个 commit
- 想快速生成符合 conventional commits 风格的 commit message
- 已经写了 commit message 草稿，但想让 AI 帮你规范化
- 希望 Claude Code、Codex、Cursor 复用同一套提交规则

## 核心行为

`commit-helper` 默认会：

1. 优先读取 staged diff；如果没有 staged 变更，再分析工作区变更
2. 先判断是否应拆分提交，再生成 commit message
3. 按业务领域、模块边界和变更目的判断原子提交边界
4. 对每个建议提交分别选择合适的 `type` 和可选 `scope`
5. 在用户明确要求提交代码时，仍然先做拆分判断

## 提交规范

优先使用 conventional commits：

```txt
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

`type` 使用：

- `feat`
- `fix`
- `refactor`
- `docs`
- `test`
- `chore`

提交说明使用中文描述变更目的，专业术语保留英文。`subject` 聚焦“为什么改”，不是流水账式描述“改了什么”。

## 典型输出

不需要拆分时：

```txt
建议单次提交：这些改动都服务于同一个登录超时修复。

fix(auth): 修复登录态超时后无法续期
```

需要拆分时：

```txt
Commit 1
建议纳入的文件或领域：src/auth/**

fix(auth): 修复登录态刷新异常

Commit 2
建议纳入的文件或领域：README.md

docs: 补充登录流程说明
```

## 目录结构

这个仓库只保留一个标准安装入口：

- `skills/commit-helper/SKILL.md`：标准 skill 入口，用于 GitHub skill 安装
- `skills/commit-helper/agents/openai.yaml`：Codex UI 元数据
