# minimax-h3-prompts-as-code-skills

Cursor Agent Skills：把 [MiniMax H3](https://www.minimaxi.com/) / Ref2VA 提示词按**可执行的程序**来写——不是润色一段散文，而是声明、赋初值、再逐步改状态。

```
git clone https://github.com/bab555/minimax-h3-prompts-as-code-skills.git
```

本仓库只有 `skills/` 与格式参考，不含剧本、分镜或项目生成器；示例均为通用场景。

---

## 为什么叫 prompts as code

MiniMax H3 的提示词在官方产品里已经是一套**固定字段、固定顺序、带引用标签**的结构（Ref2VA 六段、基础模式三字段等）。模型读它的方式更接近**按顺序执行一段程序**，而不是「感受整体氛围」：

| 编程概念 | H3 / Ref2VA 里对应什么 |
| --- | --- |
| 变量声明 | `subject_definitions`：每个 `<Subject N>` / 参考图 `<Picture N>` 是谁、从哪 import |
| 初始赋值 | `retention_analysis` 破折号后的「当前状态：…」（或基础模式 `[Shot 1]` 开头、I2VA 首帧） |
| 注释（不参与执行） | `summary`：概括画面；**不能**写「说」类词，否则后面文字会被当台词执行 |
| 主程序 | `detailed_description` / `integrated_multimodal_description` 的 `[Shot 1]`：从初值出发，一步一改 |
| 副作用 I/O | `overall_soundscape`（环境声）、`non_diegetic_music`（配乐或 `none`） |
| 未声明就使用 | 正文或 summary 里出现未在 `subject_definitions` 定义的标签 → 乱入、换脸、**来源不明的说话声** |
| 新作用域 | 每次生成是独立一段；硬切不继承上一段的声明，须在本段重新 declare |
| 调试 | 幻听、跳镜、重复动作 → 查声明是否齐全、初值与动作是否矛盾、summary 是否误触发语言 |

状态转移可以写成：**S₁ = S₀ + a₁**（当前状态 + 一个动作 = 下一状态）。`h3-prompt-writing` 里的 `references/*.txt` 与 MiniMax 官方字段对齐；产品更新时请对照官方文档自行 diff。

---

## 安装

将 `skills/` 下**每个子文件夹**复制到 Agent Skills 目录：

| 位置 | 路径 |
| --- | --- |
| 项目级 | `<项目根>/.agents/skills/` |
| 用户级（Windows） | `%USERPROFILE%\.cursor\skills\` |
| 用户级（macOS / Linux） | `~/.cursor/skills/` |

复制后重启 Cursor 或重新打开项目，Agent 即可加载这些 skill。

---

## 包内技能与阅读顺序

| 目录 | 作用 |
| --- | --- |
| `vibe-prompting` | 核心：状态程序、肯定赋值、硬切 vs 引导、排查当作调试 |
| `h3-prompt-writing` | H3 字段语义、Ref2VA 六段与基础模式、官方句式（`references/`） |
| `h3-measured-units` | 4–15 秒拆段、引导上端 22 帧、段内动作上限 |
| `h3-cinema-prompt` | 景别、机位、焦段、一种主运镜 |
| `storyboard-h3-bridge` | 从分镜表落到 H3 时的衔接（可选） |

建议顺序：**vibe-prompting → h3-prompt-writing → h3-measured-units → h3-cinema-prompt**；从分镜出稿时再加 **storyboard-h3-bridge**。

---

## 许可与说明

Skills 文本可自由使用与修改；MiniMax、Hailuo、H3 等为相应商标。本仓库与 MiniMax 官方无隶属关系。

---

**天枢** · 抖音 `@x98888666888` · 欢迎交流
