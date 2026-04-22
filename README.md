# commit-helper-skill

这是一个面向提交信息整理场景的提示资产仓库，可同时适配：

- `Claude Code`：通过 `SKILL.md` 作为自定义 skill 使用
- `Codex CLI`：通过 `AGENTS.md` 作为仓库级工作指令使用
- `Cursor`：通过 `.cursor/rules/*.mdc` 作为项目规则使用

它的核心目标是：基于当前仓库变更，优先判断是否应拆分为多个原子提交，并生成可直接使用的约定式 `git commit message`。

## 项目目录

```txt
commit-helper-skill/
├── .cursor/
│   └── rules/
│       └── commit-helper.mdc
├── AGENTS.md
├── README.md
├── RULES.md
└── SKILL.md
```

各文件用途：

- `RULES.md`：三端共享的唯一规则源
- `SKILL.md`：给 `Claude Code` 使用的轻量入口，负责指向 `RULES.md`
- `AGENTS.md`：给 `Codex CLI` 使用的轻量入口，负责指向 `RULES.md`
- `.cursor/rules/commit-helper.mdc`：给 `Cursor` 使用的轻量入口，负责指向 `RULES.md`
- `README.md`：安装、接入和维护说明

## 能力说明

无论接入哪个工具，这套提示词都遵循同一目标：

- 优先分析 `staged diff`，没有已暂存变更时再分析工作区变更
- 先判断是否应拆分提交，再生成 commit message
- 支持 `feat`、`fix`、`refactor`、`docs`、`test`、`chore`
- 提交说明以中文描述变更目的，技术术语保留英文

## 安装方法

### Claude Code

适用场景：你希望在 `Claude Code` 里把它作为一个可调用的 skill。

接入方式：

1. 将当前项目放入你的本地 skills 目录，或拷贝其中的 `SKILL.md` 与 `RULES.md` 到某个 skill 目录下
2. 确保 skill 目录根部存在 `SKILL.md`
3. 确保 `SKILL.md` 能读取同目录下的 `RULES.md`
4. 重新打开 `Claude Code`，或在新会话中使用该 skill

推荐目录示例：

```txt
<your-skills-dir>/commit-helper/
├── RULES.md
└── SKILL.md
```

### Codex CLI

适用场景：你希望在 `Codex CLI` 中进入该仓库后，自动继承这套 commit message 工作规则。

接入方式：

1. 将 `AGENTS.md` 与 `RULES.md` 放在目标仓库根目录，或放在你希望生效的子目录根部
2. 在该目录下启动 `Codex CLI`
3. 当你提出“生成 commit message”“判断是否拆分提交”等请求时，Codex 会先读取 `AGENTS.md`，再按其中指示遵循 `RULES.md`

推荐目录示例：

```txt
<your-repo>/
├── AGENTS.md
├── RULES.md
└── ...
```

### Cursor

适用场景：你希望在 `Cursor` 中让 AI 助手遵循同样的 commit message 规则。

接入方式：

1. 将 `.cursor/rules/commit-helper.mdc` 放入目标项目的 `.cursor/rules/` 目录
2. 将 `RULES.md` 放在项目根目录
3. 打开该项目的 `Cursor`
4. 在相关请求中触发这条规则，例如让它分析变更并生成 commit message

推荐目录示例：

```txt
<your-repo>/
├── .cursor/
│   └── rules/
│       └── commit-helper.mdc
├── RULES.md
└── ...
```

## 推荐维护方式

如果你希望这个仓库长期同时服务于三种工具，建议这样维护：

- 只修改 `RULES.md`
- `SKILL.md`、`AGENTS.md`、`.cursor/rules/commit-helper.mdc` 只保留最小入口职责
- `README.md` 只说明接入方式和目录结构

这样后续改规则时，只需要维护一份核心文案。

## 使用示例

常见请求包括：

- “根据当前变更帮我生成 commit message”
- “看看这些改动要不要拆成两个 commit”
- “把这个 commit message 改成更符合 conventional commits”
