# H3 视频提示词 · 写法规范（内化版）

方法论学自 MiniMax-H3 官方提示词指南（I2VA / 多图对齐模式），**内化成本 skill 自带文档——不依赖任何外部 skill**。写每段的 `h3Prompt` 照这份做，结构部分有质量门逐字对账。

**正文内容怎么写（一个运镜、动作要做得完、台词逐字、声音分层、不写画风）见 `shot-writing.md`，本文只讲 H3 特有的语法和字段。**

> **状态程序：** 先声明本段出现的对象，再写初始状态，之后每一步「当前状态 + 动作 = 新状态」；动作要在当前状态下可执行，缺的步骤补上；**行为与状态不得冲突**（初态已「躺着」就不能再写走路或「躺下」）。新一段开头先写上一段的末态。正文只用肯定句。细则见 `vibe-prompting`。

## 语言分工

- **默认整条英文**（`promptLang: 'en'`）——官方规范的口径：正文、对齐指令、字段名、镜头标记全英文，禁角色名（用 an old ferryman 这类通用身份）
- 三样东西保留原文语言（官方规定）：**台词**（`<d>[Chinese] …</d>` 逐字原文，一个标点都不许动，门盯着）、歌词、画面里可见的文字（英文双引号原样引用）
- `promptLang: 'zh'` 可切整条中文（对齐指令、字段名、镜头标记都有中文版）——偏离官方推荐的备选项，实测中文效果不稳就回英文。**禁角色名这一条不跟着语言变**：它来自官方规范，中文版同样用通用身份

## 结构（validate 逐字对账的部分）

```text
How the reference pictures align with the target video — Picture 1 (from Shot 1) aligns with the 0.00-second mark of the target video; Picture 2 (from Shot 2) aligns with the 3.00-second mark of the target video; ….
（单分镜的段改用官方 I2VA 固定句：For the target video, at 0.00 seconds into the target video, <Picture 1> (from [Shot 1]) is fully referenced.）

integrated_multimodal_description:
[Shot 1] 按 <Picture 1> 的构图锚定人物与状态，再写这几秒发生什么、镜头怎么动、谁说了什么（全英文，**不写画风句**——画风由调用方统一附加）。
[Shot 2] At 00:03.000, the camera cuts to <Picture 2>: ……（**每个镜头独立一行**，切点时刻开头，等于前面分镜秒数的累计）

overall_soundscape: 1–4 句英文：环境声、动作声、非语言人声。不复述台词。

non_diegetic_music: 1–3 句英文写配器与速度（角色听不见、只有观众听得见）。没有就写 N/A。
```

中文模式（promptLang=zh）的对应 token：`参考图与目标视频的对齐——` / `整体视听描述：` / `[镜头 k] 于 00:0X.XXX，`，配乐没有写「无」。

首行对齐指令和切点时刻**由分镜秒数推导**，改了秒数忘改提示词，validate 当场拦。

## 运镜词按语言

内容规则（一切一个运镜、写成动作句）见 `shot-writing.md`。H3 特有的是**词的语言**：英文模式用官方词（static shot / push in / tracking shot……），中文模式用词表的中文词（固定/推/拉/跟拍……）——门按 `promptLang` 检查，且**必须落在自己那一行里**。

## 说话人与台词

- 台词逐字、身份写在台词外面（见 `shot-writing.md`）。H3 特有的是**标记方式**：台词进 `<d>[Chinese] …</d>`，`<d>` 里只放语言标签和台词原文
- 说话人编号 `(S1)` `(S2)` 全段稳定，第一次出现给足辨识信息（身份、年龄段、音色、语速）；同说不同人用 `(S1,S2)`
- **画外音**：画面外的人（服务员、路人）先声明成 `<Subject N>`，写明不入画、由它承载对白，镜头句写「`<Subject N>`（S1）说：`<d>`…`</d>`」；画面里的听众只在 `retention_analysis` 写一次嘴唇闭着。只有**画内同一角色念自己的旁白**时，才照抄官方句式 `says in an off-screen voiceover … while their lips remain completely closed`
- 画面里真实可见的文字（招牌、字条）用英文双引号原样引用，不翻译

## 声音字段名

三层怎么分见 `shot-writing.md`。落到 H3 的字段：台词、歌声、剧内音乐 → `integrated_multimodal_description` 的镜头行；环境与动作声 → `overall_soundscape`；配乐 → `non_diegetic_music`（没有写 `N/A`，中文模式写「无」）。

## 关键帧在 H3 里怎么引用

主分镜图（f1）钉 0.00 秒，是这一段世界观的完全参照；每个 `[Shot k]` 先写 `<Picture k>` 锚定这一切的构图与人物状态，再写动作展开。动作能不能做完、人物此刻在哪要和图一致，见 `shot-writing.md`。

## Ref2VA / 多参考：声明先于入画

写六段式 Ref2VA 时遵循 `h3-prompt-writing`：**后面出现在镜头里的一切（含肩、手、背）本镜必须先声明**。**硬切**须完整「当前状态」；**引导上端**初态由上段末 22 帧绑定，勿写与引导冲突的强绑初态。`subject_definitions` 声明，有图则上传。`retention_analysis` 申明镜头开始状态：级别表示锁得多紧，`fully_preserved` 等于 I2V，`weak_reference` 等于自由发挥；破折号后写「当前状态：」，只写此刻看得见的姿势和物件，不用概括词。`[Shot 1]` 起句复述同一状态再写动作。硬切不继承上一镜声明；未声明就写 `<Subject N>` 易导致幻听或乱入。画外对白与画内肢体分开写。
