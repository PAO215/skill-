# Skill 文档撰写规范

本仓库定义团队内 Agent Skill 的统一撰写标准，确保每个 Skill 可被触发、可被验证、可被维护。

## 1. 适用范围

本规范适用于所有以 `SKILL.md` 为入口的 Agent Skill。不适用于普通提示词、一次性脚本或纯知识文档。

## 2. 目录结构

每个 Skill 必须是一个独立文件夹，`SKILL.md` 为必需入口：

```
skill-name/
├── SKILL.md          # 必需：YAML frontmatter + Markdown 指令正文
├── references/       # 可选：按需加载的参考文档
├── scripts/          # 可选：确定性逻辑（解析、计算、格式转换）
└── assets/           # 可选：模板、静态资源
```

`SKILL.md` 正文不超过 **500 行**。超出的内容拆入 `references/`，由正文按需引用。

## 3. Frontmatter 规范

```yaml
---
name: skill-name
description: >
  <做什么>。Use when <触发场景1>、<触发场景2>、<用户会说的短语>。
license: MIT
metadata:
  version: "1.0.0"
  category: <分类>
allowed-tools: Read Write
---
```

| 字段 | 要求 |
| :--- | :--- |
| `name` | 小写字母/数字/连字符，1–64 字符，不能以连字符开头结尾或连续连字符，**必须与父目录名一致** |
| `description` | 1–1024 字符，**必须同时包含“做什么”和“什么时候用”**，至少 2 个触发短语；建议 150–300 字符 |
| `license` | 建议填写，默认 `MIT` |
| `metadata.version` | 语义化版本，Skill 行为变化时递增 |
| `allowed-tools` | 遵循最小权限，禁止 `Bash(*)`、`Read(*)`、`Write(*)` 等全量授权 |
| `disable-model-invocation` | 涉及部署、发消息、删除、改生产数据等**有副作用**的操作时必须设为 `true` |

`description` 写法对比：

```yaml
# 合格
description: >
  Extracts text and tables from PDF files and fills PDF forms.
  Use when working with PDF documents, or when the user mentions
  PDFs, forms, or document extraction.

# 不合格
description: Helps with PDFs.
```

## 4. 正文结构

正文是**可执行的操作手册**，不是知识描述。按复杂度选择骨架：

### 轻量（单一步骤、无外部依赖、少于 3 个判断分支）

```markdown
## 任务
## 步骤
## 输出格式
## 异常处理
```

### 中等（多步骤、有工具调用、有异常处理）

```markdown
## 前置条件
## 执行步骤
## 输出契约
## 异常处理
```

### 复杂（多阶段、大量参考知识、需要脚本）

`SKILL.md` 只做协调器，用阶段路由表引用 `references/`，确定性逻辑放 `scripts/`。

## 5. 输出契约（必填）

每个 Skill 必须定义结构化输出，至少包含三部分：

1. **状态**：`success` / `partial` / `failure`
2. **数据**：结构化结果字段
3. **可观测信息**：耗时、关键中间状态、失败原因

缺少“部分成功”状态视为不完整，因为它没有表达多子任务部分失败时的行为。

## 6. 边界与反模式（必填）

正文末尾必须包含：

```markdown
## 不适用场景
- 当 <条件> 时，不应调用本 Skill，因为 <原因>。
- 当 <条件> 时，应改用 <替代方案>。

## 反模式
- **<错误做法>**：<为什么错> → <应该怎么做>
```

只写“能做什么”的 Skill 是说明书；写清“不该做什么”的才是工程资产。

## 7. 提交前自检清单

- [ ] `name` 与目录名一致，符合小写 + 连字符规则。
- [ ] `description` 同时包含“做什么”和“什么时候用”，含至少 2 个触发短语，不超过 1024 字符。
- [ ] `SKILL.md` 正文不超过 500 行。
- [ ] 输出契约包含状态、数据、可观测信息三部分。
- [ ] 有“不适用场景”和“反模式”章节。
- [ ] 有副作用的操作已设 `disable-model-invocation: true`。
- [ ] `allowed-tools` 遵循最小权限，无 `Bash(*)` 等全量授权。
- [ ] 确定性逻辑已考虑放入 `scripts/`，而非让模型用自然语言硬算。
- [ ] 至少记录了一轮改动前后的最小回归用例结果（正向 2、边界 2、应拒绝 2）。
- [ ] `metadata.version` 已填写，且与本次改动匹配。

## 8. 模板与示例

- 空白模板：[`templates/SKILL.md`](templates/SKILL.md)
- 完整示例：[`examples/pdf-processing/SKILL.md`](examples/pdf-processing/SKILL.md)
