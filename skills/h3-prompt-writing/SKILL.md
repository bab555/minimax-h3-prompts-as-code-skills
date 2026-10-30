---
name: h3-prompt-writing
description: Write MiniMax H3 video generation prompts for T2VA, I2VA, FL2VA, L2VA, and Ref2VA. Maps state-program prompting (declare objects, S1 = S0 + action, positive assignment) onto H3 fields. Use when rewriting multimodal requests into H3 structures, Ref2VA six-section blocks, soundscape, or reference labels.
compatibility: Portable to any agent that can read local files — no external API calls, MiniMax Hub tools, or proprietary runtime required. The agents/openai.yaml file only adds optional ChatGPT/Codex UI metadata; it does not restrict the skill to OpenAI agents.
---

# H3 Prompt Writing

提示词的逻辑按 `vibe-prompting` 写：先声明对象，再绑定开场状态，之后每一步「当前状态 + 动作 = 新状态」，正文只用肯定句。H3 的控制逻辑与顺序程序同构；排查问题时见 `vibe-prompting`「排查：当作调试程序」。本 skill 只规定这些逻辑落到 H3 的哪个字段、用什么句式。

本文所有示例均为通用示例，不对应任何具体项目。

**核心：本段没在 `subject_definitions` 声明的对象，正文和 `summary` 里都不能给它写动作、状态或台词。** 写了，模型找不到这句话的承载者，常见结果是整句或整段被当成语音念出来、台词落到画面里唯一已声明的人嘴上，或者临时造一个人乱入。画面外说话的服务员、「一只手」「其他客人」都算对象，都要先声明。

下表「角色」一列的编程说法只是类比，帮助理解字段的作用；提示词里不写 `import`、`S0`、「变量」「赋值」这类词，只写普通的画面描述。

## 字段语义（按键名理解）

| 字段 | 角色（类比） | 写什么 |
| --- | --- | --- |
| `subject_definitions` | 声明变量 | 每个 `<Subject N>` 是谁、来自哪个 `<Picture N>` / `<Video N>` / `<Audio N>`，或纯文字声明 |
| `retention_analysis` | 初始赋值（状态绑定） | 每个已声明标签一行。级别是这条开场状态锁得多紧；破折号后面写被绑定的开场状态 |
| `summary` | 注释 | 一句话概括这段画面 |
| `detailed_description`（Ref2VA）/ `integrated_multimodal_description`（基础模式） | 主程序 | 从已绑定的初值出发，按 S₁ = S₀ + a₁ 推进。基础模式没有 `retention_analysis`，初值写在 `[Shot 1]` 开头 |
| `overall_soundscape` | 环境音赋值 | 环境声、动作声、呼吸 |
| `non_diegetic_music` | 配乐赋值 | 配乐，或 `none` / `N/A` |

字段说明里，这一段常被写成「参考内容保留到什么程度，不描述镜头里发生的事」。通用方案不采用这层字面读法。这一段的实质是**申明镜头开始状态**：直接绑住 0 秒时看得见的状态，就是初始赋值。级别只表示这套状态锁得多死。

## 字段是整套必出，不是按需挑

选定一种输入关系后，对应的字段键名全部写出、按固定顺序，不能因为「这段没用到」就删掉某个键。空的音频层仍保留键名，值写 `N/A`（基础模式）或 `none`（Ref2VA 的 `non_diegetic_music`）。

| 模式 | 必出的键（按此顺序） |
| --- | --- |
| T2VA / I2VA / FL2VA / L2VA | `integrated_multimodal_description`、`overall_soundscape`、`non_diegetic_music`。I2VA / FL2VA / L2VA 另在三字段之前加一行首帧或尾帧对齐句 |
| Ref2VA | `subject_definitions`、`summary`、`retention_analysis`、`detailed_description`、`overall_soundscape`、`non_diegetic_music` |

按需的是**行**，不是**字段**：

- `subject_definitions` 只声明本段真正用到的标签。没上传的 `<Picture N>` / `<Video N>` / `<Audio N>` 不写。
- `retention_analysis` 给**每一个已声明的标签**各一行，级别用官方英文值。没声明的标签没有行；声明了就不能不写。破折号后面写这一标签被绑定的开场状态。
- Ref2VA 的初值写在 `retention_analysis`。`[Shot 1]` 起句复述同一初值，然后只写动作；两处必须是同一事实。基础模式没有这一段，初值写在 `[Shot 1]` 开头。I2VA 的首帧图本身就是最强初值。

### 硬切 vs 引导上端（初始状态谁绑）

| 接法 | 初始状态从哪来 | `retention_analysis` / `[Shot 1]` 初态 |
| --- | --- | --- |
| **硬切** | 本段文字申明 | 必须写全「当前状态：」并在主程序复述；与上一段无像素连续 |
| **引导上端**（上一段成片最后 22 帧） | **上端画面已绑定** | 实质等于 I2V：0 秒状态由引导帧提供。可**不写或极简**初态，只写本段**新动作**；若写初态，必须与上段末帧**同一事实**，否则与引导打架 |

引导段若**强绑**一套与引导帧不符的「当前状态」（例如上端她仍端着杯，本段却写「她空手站在柜前」），模型会在接点**切镜、跳变**，引导失效。硬切段则必须完整申明，不能假设上段末态。

细则与何时用引导见 `h3-measured-units`「引导」、`h3-cinema-prompt` 长镜拆段。

没有参考素材要跨段追踪时，用基础模式，不进 Ref2VA，也就没有 `retention_analysis`。

## H3 固定规则

- 严禁在 reference generation 中加上「几秒」这种时间描述词，否则整段会被识别为音频。时间只出现在切镜点：`[Shot 1]` 不加时间戳，后续镜头写成 `[Shot N] At MM:SS.mmm`。单镜头内的动作按先后写成连续叙述。
- 严禁在 `summary` 中写「说」以及「说」的同义词，否则生成时会把后面的文字变成语言。
- H3 没有负面提示词字段。**只有三处允许出现否定**：`<d>…</d>` 台词内、画面内可见文字、`none` / `N/A`。其余正文一律用正面赋值。

## 逻辑落到哪个字段

| 逻辑步骤 | Ref2VA 字段 | T2VA / I2VA / FL2VA / L2VA 字段 |
| --- | --- | --- |
| 声明对象 | 上传 `<Picture N>` 等 + `subject_definitions` | `integrated_multimodal_description` 第一句锚定主体与构图 |
| 开场状态 | `retention_analysis`：级别 + 被绑定的开场状态 | `[Shot 1]` 开头。I2VA 的首帧图即最强初值 |
| 注释 | `summary`，不写说话动词 | 不写 |
| 动作与新状态 | `detailed_description` 的 `[Shot 1]`，起句复述同一初值 | `integrated_multimodal_description` 的 `[Shot 1]` |
| 台词 | 镜头句里的 `<d>[Language] …</d>` | 同左 |
| 环境音 | `overall_soundscape` | `overall_soundscape` |
| 配乐 | `non_diegetic_music: none` | `non_diegetic_music: N/A` |

## 声明（subject_definitions）

| 情况 | 做法 |
| --- | --- |
| 有参考图 | 上传 `<Picture N>`，写「`<Subject N>` 是来自 `<Picture N>` 的××」 |
| 无参考图但画面需要此人/此物 | 只用文字写「`<Subject N>` 是××，本段××入画」；模型按文字补外观 |
| 正文或 summary 里点名却本段未定义 | **未声明** → 成段念成语音、乱入、换脸的高频原因 |

- 本段 `[Shot 1]` 里出现的每个 `<Subject N>`，都必须在本段 `subject_definitions` 定义。有图则上传；无图不强行上传，但声明不能省。
- 声明写具体身份，不写「配角」「路人」这类笼统标签；笼统标签会让模型把不同角色混成一个。
- 每次生成是独立的一段。上一次生成里定义过的对象，本段要出现就重新定义。
- 本段不出现的对象，不写名字，不上传图。
- 画面外说话的人（服务员、路人、旁白者）同样在 `subject_definitions` 声明成 `<Subject N>`，通常不上传图，写明本镜不入画、由它承载本镜对白；镜头句写 `<Subject N>（S1）说：<d>…</d>`。听众怎么写见下文「简洁」。

**声明越笼统，模型补得越多：** 声明越笼统（「一个服务员」），模型拉进来的默认属性越多（整张脸、整个人）。只露出局部的对象，在 `retention_analysis` 里把可见范围绑成开场状态：「`<Subject 2>` 的双手与小臂在画面右侧。」不要写「只看见手」「脸不在画内」。

## 初始赋值（retention_analysis）

**`retention_analysis` 仅用于 Ref2VA。** 基础模式 T2VA / I2VA / FL2VA / L2VA 只用 `integrated_multimodal_description`，不写 `retention_analysis`。基础模式的初值在 `[Shot 1]` 开头；I2VA 的 `<Picture 1>` 就是 0.00 秒的整帧，锁死程度等于最强的状态绑定。

每个已定义标签一行。级别是官方英文值，表示**这条开始状态锁得多紧**。破折号后先保留绑定说明，再写「当前状态：」，申明这一镜 0 秒时看得见的位置、姿势、朝向、手持物、可见范围。

当前状态只写此刻看得见的事实，拆开写。不用一个概括词把姿势和物件压在一起。概括词会被读成另一套默认画面。

| 概括说法 | 当前状态 |
| --- | --- |
| 端坐 | 她坐在椅上，双脚平放在地面，双手放在膝上 |
| 捧杯 | 右手托着杯底，杯口朝上 |
| 背手 | 双手叠在身后，掌心朝外 |

| 素材 | 级别 | 锁的松紧 |
| --- | --- | --- |
| 视觉（Subject / Picture / Video） | `fully_preserved` | 最强，等于这一主体的 I2V：开场按这一行写定的状态执行，参考图锁住外观 |
| | `partially_preserved` | 仍锁着，只锁可见的一部分，或相对参考改掉一部分特征 |
| | `attribute_transfer` | 不把参考当成开场画面，只把其中一项属性绑到另一个主体的开场状态上 |
| | `weak_reference` | 最弱，等于自由发挥：只给类别，外观由模型补。开场状态仍由这一行点名 |
| 音频（Audio） | `fully_copy` | 整条音频原样作为成片音轨 |
| | `partially_copy` | 只复制一段或部分音层，或复制后增删替换 |
| | `reference` | 不复制信号，只绑音色、节奏、风格或台词内容 |
| | `weak_reference` | 只留大致类别或氛围，其余自由发挥 |

通用示例：

```text
<Subject 1> (appears in [Shot 1]): fully_preserved - 外观按 <Picture 1>。当前状态：她坐在靠窗的桌边，双脚平放在地面，双手放在桌面。
<Subject 2> (appears in [Shot 1]): weak_reference - 文字声明。当前状态：双手与小臂在她身侧，悬在杯子上方。
<Video 1> (motion reference): attribute_transfer - 只把走路的步伐绑到 <Subject 1> 的开始动作。
<Audio 1>: reference - 只绑音色与语速。
```

只做文字声明的 Subject 用 `weak_reference`，破折号后仍要写开场状态，不能只写「文字声明」。

## 主程序（从已绑定的初值做状态转移）

`[Shot 1]` 是一段顺序执行的状态程序。Ref2VA 的初值已经在 `retention_analysis` 里赋过：

```text
retention 绑定的初值 S0 → [Shot 1] 起句复述 S0 → 动作，新状态 → … → 末态
```

- 起句复述同一初值，然后只写动作。retention 写「坐在桌边」，主程序不能改成已经站在门口。
- **行为与状态不得冲突**（与 `vibe-prompting` 规则 2.1 相同）：当前状态里已有的姿势或物件，后面不能写与之矛盾的动词；也不能写「进入」已在的状态（例如已躺在地上，不能再写「躺下了」；桌上已有空杯，不能再写「放下一杯」）。动作只能改变当前状态，不能重复绑定同一状态。
- 每个动作只从当前状态出发。前置条件不满足时补一步（坐着要走，先站起来）。
- 改变状态的动作后面写出新状态。
- 一段只放几步动作，做不完就在稳定状态处拆段。
- **硬切**后，下一段的 `retention_analysis` 把上一段末态绑成新的初值，`[Shot 1]` 起句再复述它。
- **引导上端**段：初态由上端帧承担；主程序从停住处只写新动作，勿重复上段已完成的动作（见 `h3-measured-units`）。

通用示例（Ref2VA 的 `detailed_description`）：

```text
[Shot 1] 中景。平视，50mm。镜头始终锁定<Subject 1>，她在画面中心。她坐在靠窗的桌边，双手放在桌面。她双手撑桌站起来，站在桌边。她转身面向画面右侧的门，走到门前停住。画面右缘是门框。Static, slow, small amplitude。
```

## 输出语言

- 标签、字段名、`<Subject N>`、`<Picture N>`、`<Video N>`、`<Audio N>`、`[Shot N]`、保留级别必须保持英文。
- 正文语言按项目统一（全中文或全英文），同一项目内不要混用。
- 台词、歌词、画面内文字保留原语言。

## 简洁（写进 H3 的正文，不是写给人看的教程）

Agent 产出提示词时 **宁短勿啰嗦**。把 skills 里的排查说明、括号注释、重复强调 **留在大纲/对话里，不要抄进六段字段**。

**谁说话、谁在听，只在声明层写一次；`[Shot 1]` 正文只写动作和 `<Subject N>（Sx）说：<d>…</d>`。**

画外有人说话（服务员、路人等），画面里的人在听：

- `subject_definitions`：说话人声明成 `<Subject N>`，写明本镜不入画、由它承载本镜对白。
- `retention_analysis`：说话人那行写明人不在画面内、声源在画外；听众那行在当前状态末尾写一次嘴唇闭着、是该画外声的听众。
- `[Shot 1]`：听众只写姿势和动作，不再写嘴唇；台词只挂在说话人的 `<d>` 上。说话人这句前用句号断开，不接在听众的句子后面。

只有一个人、也没有画外声的镜头，不写嘴唇。

| 避免 | 原因 |
| --- | --- |
| 正文里写「嘴唇闭着」，或每句后面叠一遍 | 重复的唇部描述会变成多余的口型指令 |
| 正文里写「该句为画外声」「勿对口型」等解释句 | 解释句是写给人看的，不是画面状态 |
| 写「服务员说」但服务员没在 `subject_definitions` 声明 | 主语未声明，声音容易落到画面里唯一的人身上 |
| retention 与 `[Shot 1]` 整段复制 | 初值在 `retention_analysis` 写全；Shot 起句一句复述位置姿势，后面只写动作 |

另一条官方规则：**画内看得见的同一个角色**用 `says in an off-screen voiceover` 念旁白时，官方要求在该 `<d>` 后写 `while his lips remain completely closed`（见 `references/base-en.txt`）。只在这种「本人在画里、声音是他的旁白」的情况照抄官方句，别处不用。

## 自检

- `[Shot 1]` 里每个 `<Subject N>`，本段都在 `subject_definitions` 定义了吗？
- Ref2VA 每个已定义标签都有一行 `retention_analysis`，级别是官方英文值，破折号后是开场状态吗？基础模式里没有写 `retention_analysis` 吧？
- `[Shot 1]` 起句和 `retention_analysis` 的当前状态是同一事实吗？当前状态是看得见的姿势和物件，没有用一个概括词吗？每个动作的前置条件都成立吗？初态与后续动作是否冲突（躺着却走、已有物件却又重复「放置」、已处于末态却又写「完成该末态」）？
- 否定词只出现在 `<d>…</d>`、画面内文字、`none` / `N/A` 里吗？
- `summary` 里有「说」或同义词吗？
- 有画外声时：说话人声明了吗？听众的嘴唇和听众身份只在 `retention_analysis` 写了一次吗？`[Shot 1]` 里没有嘴唇和解释句吗？

## 发布前检查项（平台 UI 约束，不是 H3 模型规则）

- 发到短视频频道的视频，画面必须铺满到右边缘。右侧约 1/20 的那一条不能长时间留白，更不能是白底；否则点赞、评论、转发的白色图标落在空白上看不见。

## Workflow

1. Identify the input mode: T2VA, I2VA, FL2VA, L2VA, or full-reference Ref2VA.
2. For base text/keyframe modes, read `references/base-en.txt` and follow its final prompt structure.
3. For full-reference mode, read `references/ref-en.txt` and follow its six-section rewrite format.
4. Preserve the exact field names, section order, labels, and timing notation from the selected guide.

## Base Modes

- T2VA: build the full audiovisual timeline from text.
- I2VA: start from the first frame and develop forward from it.
- FL2VA: describe the continuous path between the first and last frames.
- L2VA: infer a plausible opening and converge to the supplied last frame.

Use `integrated_multimodal_description`, `overall_soundscape`, and `non_diegetic_music` in the order shown in `references/base-en.txt`. No `retention_analysis` in base modes.

## Full-Reference Mode

Ref2VA rewrites use `subject_definitions`, `summary`, `retention_analysis`, `detailed_description`, `overall_soundscape`, and `non_diegetic_music` in that order. Reference labels stay consistent across all sections.

Read `references/ref-en.txt` for label rules, retention levels, and complete examples.

## Output Rules

- Labels, field names, reference tags, and retention levels stay in English; body text follows the project's single language (see 输出语言).
- Describe each shot by composition, subjects, environment, actions, camera, sound, and the exact point where referenced content appears.
- Avoid plot summaries, unresolved reference labels, and timing that does not match the requested duration.

## Tips for Better Results

- Always match the total duration of the description to the requested video length (4–15 seconds).
- Keep reference labels consistent (e.g. `<Picture 1>`, `<Video 1>`, `<Audio 1>`) across every section.
- Prefer concrete visual and audio details over abstract words like "cinematic" or "beautiful".
- When using keyframes (I2VA / FL2VA / L2VA), clearly state how the first and/or last frame connects to the timeline.
