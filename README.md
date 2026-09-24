# make-knowledge-cards

把粘贴的文章或本地 Markdown/TXT 提炼成便于阅读复习的知识卡片。这是由 AI 助手读取并执行的指令型 Skill。

## 项目解决什么问题

阅读文章后，手工整理笔记容易照搬原文、重复记录，或把多个知识点混在一起。本项目提供统一的提炼规则，帮助助手选出重要知识，整理成可独立阅读和自测的卡片，并保留原文的适用条件与限制。

## 主要功能

- 接收粘贴的正文，以及助手有权限读取的本地 `.md`、`.markdown`、`.txt` 文件。
- 通常生成 5–8 张卡片；信息不足时减少数量，没有可提炼知识时说明原因。
- 每张卡只讲一个知识点，包含标题、核心知识、简明解释，以及原文例子或附答案的自测题。
- 合并重复内容，要求事实、数字、例子和答案有原文依据，不补入外部知识。
- 默认在对话中输出 Markdown；用户指定保存时，由助手写入目标文件。

暂不支持网页抓取、PDF、Anki 导出或图形界面。生成结果仍需结合原文核对。

## 安装方法

前提：已可使用 Codex 或能够读取 Skill 指令的 AI 助手；处理本地文件时，助手需要相应的文件读取权限。日常使用无需安装 Python、Node.js 或项目依赖。

本项目的 Skill 源文件保存在 `skills/make-knowledge-cards/`。可以直接按下一节指定路径使用；若希望 Codex 自动发现该 Skill，可将整个文件夹复制到项目的 `.agents/skills/` 下。

在项目根目录执行以下 PowerShell 命令（目标已存在时停止，避免覆盖）：

```powershell
$skillTarget = Join-Path (Get-Location) '.agents/skills/make-knowledge-cards'
if (Test-Path -LiteralPath $skillTarget) {
    throw '目标 Skill 已存在，请先检查现有内容。'
}
New-Item -ItemType Directory -Path '.agents/skills' -Force | Out-Null
Copy-Item -LiteralPath './skills/make-knowledge-cards' -Destination $skillTarget -Recurse
```

复制后的目录应包含 `SKILL.md` 和 `agents/openai.yaml`。Codex 会自动检测 Skill 变更；如果未出现，可重启 Codex。安装目录及发现机制依据 [OpenAI 官方 Skill 文档](https://learn.chatgpt.com/docs/build-skills)。源文件更新后，安装副本也需同步更新。

## 使用方法

### 直接读取项目内的 Skill

让支持 Skill 的助手读取 `skills/make-knowledge-cards/SKILL.md`，再提供正文或本地文件路径。无需运行转换脚本。

```text
请按 skills/make-knowledge-cards/SKILL.md 的规则，
将本地文件 tests/technical.md 整理成知识卡片。
```

`tests/technical.md` 是项目自带的测试文章，可替换为自己的文件路径；路径含空格时用引号括起。

### 安装后调用

在已加载该 Skill 的 Codex 环境中输入：

```text
请使用 $make-knowledge-cards 将下面的文章整理为知识卡片：
……粘贴正文……
```

需要保存结果时，明确指定输入和输出路径，例如：

```text
请使用 $make-knowledge-cards 处理 tests/opinion.txt，
将知识卡片保存到 opinion-cards.md。
```

## 输入输出示例

输入：

> 主动回忆是先不看资料，尝试从记忆中提取知识。自测后应对照原文检查答案，找出遗漏或错误。

输出：

原文包含 2 个独立知识点，因此生成 2 张卡片。

### 1. 主动回忆

- **核心知识**：主动回忆是在不看资料时从记忆中提取知识。
- **简明解释**：先尝试回想，而不是先查看资料。
- **自测问题**：主动回忆时，应该先看资料还是先回想？
  **答案**：先回想。

### 2. 自测后核对

- **核心知识**：自测后应对照原文，找出答案中的遗漏或错误。
- **简明解释**：用原文检查刚才回忆的内容。
- **自测问题**：自测后需要对照什么检查答案？
  **答案**：原文。

## 开发与验证

Skill 由官方 `skill-creator/scripts/init_skill.py` 初始化，交付文件为 `SKILL.md` 和 `agents/openai.yaml`，不包含转换脚本，也未配置额外的连接器或服务。生成卡片依赖所使用的 AI 助手，不承诺离线运行。

使用带 PyYAML 的 Python 运行官方验证器（将路径替换为本机 skill-creator 所在位置）：

```text
python -X utf8 <skill-creator目录>/scripts/quick_validate.py skills/make-knowledge-cards
```

多类型材料的人工试运行、输出和迭代记录见 [tests/manual-tests.md](tests/manual-tests.md)。结构验证不等于生成质量保证。

本项目已完成本地开发与 Git 仓库初始化，尚未上传 GitHub。
