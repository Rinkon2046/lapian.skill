---
name: short-video-lapian
description: 用于拆解视频内容并生成复刻报告。重点覆盖短视频、自媒体、评测、教程、口播、Vlog 和影视片段；当用户说“拉片”“拆这个视频”“复刻这个博主”“分析这个视频为什么有效”“给我 SOP”时使用。
---

# 短视频拉片与复刻

## 目标

把视频拆成“看得懂的结构”和“做得出来的步骤”。最终交付一份自包含 HTML 报告；如果是短视频或自媒体内容，同时给出复刻 SOP。

优先回答四个问题：

1. 这条视频靠什么吸引人？
2. 信息是怎样一段一段推进的？
3. 画面、字幕、图形、剪辑和声音各自承担什么作用？
4. 如果用户要做一条类似视频，下一步该怎么执行？

## 工作边界

- 不只写观感，要写因果：某个镜头、台词或图形为什么有效。
- 能引用时间码就引用时间码，能配真实截帧就配真实截帧。
- 工具链、字体、软件等无法确认时要标注置信度，不要装作知道。
- 报告正文不要写工具环境缺失、生成限制或排版迭代说明；这类信息只放工作日志。
- 默认输出 HTML，不以纯文本作为最终产物。

## 脚本定位

脚本位于本 skill 的 `scripts/` 目录。执行前先找到 `SKILL.md` 所在目录：

```bash
SKILL_DIR="$(dirname "$(find -L . ~/.codex/skills ~/.claude .claude -path '*/lapian*' -name 'SKILL.md' -maxdepth 6 2>/dev/null | head -1)")"
```

生成报告时使用：

```bash
node "$SKILL_DIR"/scripts/generate_report.mjs \
  --analysis /path/to/analysis.md \
  --keyframes /path/to/keyframes \
  --output /path/to/report.html
```

## 输入分流

### 1. 只有作品名或描述

用于讨论型分析。根据用户提供的信息、共同认知和题材框架，输出结构化判断，再生成 HTML。

### 2. 有截图、关键帧、字幕

先看图和字幕，再补充分析。适合快速判断版式、字幕风格、信息顺序和画面证据。

### 3. 有本地视频或 URL

走完整流程：下载或读取视频，切场景，抽关键帧，解析字幕，必要时转写音频，最后生成配帧报告。

URL 下载示例：

```bash
yt-dlp -f 'bestvideo[height<=720]+bestaudio/best[height<=720]' \
  --merge-output-format mp4 -o '/tmp/lapian/<name>/video.%(ext)s' '<URL>'
```

## 完整视频流程

### Step 1：定目标

先判断内容类型，再决定表格列。不要所有视频都套同一个模板。

常见类型：

- 口播教程：台词结构、观点推进、字幕强调、图形动效、露脸/录屏比例
- 纯口播：开头痛点、语速、停顿、表情手势、字幕节奏、金句
- 科技爆料/评测：参数顺序、证据画面、信源处理、购买建议、价格/风险收口
- Vlog/叙事：事件顺序、情绪曲线、生活细节、转场、音乐
- 影视片段：镜头语言、剪辑、声音、表演、场面调度、主题表达

### Step 2：拆素材

```bash
node "$SKILL_DIR"/scripts/extract_scenes.mjs \
  --input /path/to/video.mp4 \
  --output /tmp/lapian/scenes/ \
  --threshold 0.3

node "$SKILL_DIR"/scripts/extract_subtitles.mjs \
  --input /path/to/video.mp4 \
  --srt /path/to/subtitle.srt
```

没有字幕时再转写：

```bash
node "$SKILL_DIR"/scripts/transcribe.mjs \
  --input /path/to/video.mp4 \
  --output /tmp/lapian/transcript.json \
  --model base
```

### Step 3：建时间轴

```bash
node "$SKILL_DIR"/scripts/build_timeline.mjs \
  --scenes /tmp/lapian/scenes/scenes.json \
  --subtitles /tmp/lapian/scenes/subtitles.json \
  --output /tmp/lapian/timeline.json
```

### Step 4：逐段判断

逐段看关键帧、字幕和时间轴。若某段转场、字幕跳变、动效或手势看不清，只对该段追加密集抽帧，不要整条视频盲目加密。

### Step 5：写分析稿

短视频最小字段：

| 字段 | 写什么 |
|---|---|
| 时间码 | 该段起止时间 |
| 代表帧 | 最能说明问题的一张截图 |
| 台词/字幕 | 可见字幕或转写摘要 |
| 记忆点 | 本段最值得复刻的表达 |
| 画面层 | 字幕、贴纸、标注、画中画、转场、动效 |
| 作用 | 钩子、立信、解释、证明、反转、催促、收口 |

### Step 6：生成 HTML

```bash
node "$SKILL_DIR"/scripts/generate_report.mjs \
  --analysis /tmp/lapian/analysis.md \
  --keyframes /tmp/lapian/scenes/keyframes \
  --output /tmp/lapian/report.html
```

## 报告模板

### 短视频 / 自媒体

1. 概况：标题、来源、时长、切点数、内容类型
2. 一句话结论：这条视频为什么能成立
3. 拉片维度：内容层和制作层分别看什么
4. 可复刻度：内容结构、素材获取、拍摄、图形/动效、节奏、工具链
5. 关键发现：3-5 条真正可迁移的规律
6. 逐段配帧表：每段至少一张真实截帧
7. 视觉系统：字体、颜色、字幕、图形元素、转场、素材类型
8. 工具链判断：事实 / 高置信推断 / 低置信推断 / 未知
9. 复刻 SOP：按用户目标写成行动步骤

### 影视 / 长视频

1. 作品概况：题材、时长、结构
2. 整体判断：先给结论
3. 分段分析：场景、段落或幕结构
4. 专项分析：镜头、剪辑、声音、视觉、叙事
5. 可学习手法：哪些技法值得迁移

## 题材参考

可以按内容加载 `references/frameworks/` 下的框架：科幻、悬疑、犯罪、动作、纪录片、教程、产品评测、Vlog、Video Essay、新闻评论等。多题材内容可组合使用。

## 粒度

| 粒度 | 适合场景 | 输出密度 |
|---|---|---|
| 速览 | 快速判断视频是否值得学 | 3-5 张关键帧 |
| 标准 | 默认拉片报告 | 每个主要段落一张代表帧 |
| 精读 | 要复刻或训练团队 | 重点段落追加密集帧 |

## 依赖

- `ffmpeg`：视频切分和抽帧
- `yt-dlp`：URL 下载，可选
- `whisper`：音频转写，可选

依赖缺失时说明缺什么，并退回到可执行的输入模式。
