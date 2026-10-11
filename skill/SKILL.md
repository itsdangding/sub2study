---
name: sub2study
description: >-
  Turn YouTube video subtitles (especially Russian ASR captions) into publication-grade bilingual study guides.
  Performs intelligent ASR preprocessing/proofreading, contextual translation, stress-marked vocabulary extraction,
  and exports Markdown (.md) and PDF study guides.
---

# sub2study: YouTube Subtitles to Bilingual Study Guide

`sub2study` automates the entire journey from raw video subtitles to polished language learning materials.
It fixes speech-to-text mistakes (proper nouns, missing commas, speech disfluencies), performs context-aware translation,
extracts high-value vocabulary with stress marks, and generates ready-to-print PDFs.

---

## Workflow Overview

```
YouTube Video URL
       │
       ▼  [yt-dlp + Smart Cookie Resolver]
Raw VTT / SRT Subtitles (e.g., .ru.vtt / .ru-orig.vtt)
       │
       ▼  [sub2study extract / scripts/extract_and_clean_subtitles.py]
Mechanical Cleaning & Paragraph Chunking (strip duplicates, group by timestamps)
       │
       ▼  [★ AI Preprocessing & Proofreading ★]
Grammar cleanup, proper noun correction, punctuation restoration, filler removal
       │
       ▼  [AI Translation & Vocabulary Extraction]
Bilingual Paragraphs + Stress-marked Vocabulary Table (lemma, accent, POS, collocations)
       │
       ▼  [sub2study render & sub2study interactive]
1. Study Markdown (.md)
2. Interactive Reader HTML (.html)
3. Print-Ready PDF (.pdf via Headless Chrome, whole words preserved)
4. Interactive Sentence Study Player (Karaoke Sync + Replay + Blind Test)
```

---

## Quick Reference / Commands

### Step 1: Download & Clean Subtitles into Paragraphs
```bash
sub2study extract "<YOUTUBE_URL>" -o "~/Desktop/Russian_Study" --lang auto
```
* **Output**: `cleaned_paragraphs.json`
* **Features**:
  * **Any Language Supported**: Automatically detects the video's original spoken/subtitle language (English, Russian, Japanese, German, French, etc.) via `--lang auto`, or accepts explicit language codes like `ru`, `en`, `de`, `ja`.
  * **Bulletproof Cookie Resolution**: Auto-locates primary browser profile cookies (Chrome/Edge/Brave) to bypass YouTube's "Sign in to confirm you're not a bot" challenge without extension interference.
  * Strips ASR rolling duplicates and inline `<c>` word tags.
  * Reconstructs full sentences ending in `.`, `!`, `?`.
  * Groups sentences into natural paragraphs (~45-75 words) with anchor timestamps `[MM:SS]`.

---

### Step 2: AI Preprocessing & Proofreading
Before translating, pass the cleaned paragraphs through an AI proofreading step:
- Correct misrecognized terms (e.g. `айл-экзамен` -> `IELTS-экзамен`, `веков X и двадцать первого` -> `веков XX и XXI`).
- Restore missing commas and conjunction punctuation for complex clauses.
- Remove stuttered duplicate words while preserving colloquial idioms.

### Step 3: High-Fidelity Translation & Vocabulary Extraction
For long transcripts (over 30-40 paragraphs), split the paragraphs into chunks and invoke concurrent `self` subagents or process sequentially:

#### 1. Zero-Omission Translation Rules (★ 严格禁止偷懒与漏译 ★)
- **100% 逐句忠实全译（Sentence-by-Sentence Integrity）**：
  必须逐句对照翻译全部内容，保证原意完整。**严禁漏译任何整句、从句、前置铺垫或细节**。绝不可因为段落较长或口语化而删减任何话语。
- **严禁提纲化或概括摘要（No Summarization or Forced Bulleting）**：
  原文是口语叙述流的，**严禁脑补“第一、第二、第三、第四”清单**将其压缩为提纲！必须保留演讲者原始的口语叙述流、语气、承上启下的过渡句、所有的生动举例、调侃、轶事以及具体人名/产品名/事件名（如 Sam Altman、OpenClaw、Jony Ive 等），不得将具体的生动案例概括成抽象结论。
- **高保真专业术语统一（Terminology Precision）**：
  精准还原专业术语（如 Harness、Durable Workflow、MicroVM、Prompt Caching、Cache Warming、Dynamic Context、Compaction、Accessibility Tree 等）。
- **字数与信息密度自检机制（Length & Density Sanity Check）**：
  翻译完成后，**必须进行字数比对自检**：
  - 英译中：正常中文字数约为英文词数的 1.6 ~ 2.2 倍（中文字数 / 英文词数 $\ge 1.5$）。
  - 俄译中：正常中文字数约为俄语词数的 1.1 ~ 1.6 倍。
  - **红线预警**：若中英比低于 1.3，必定存在严重漏译、跳译或概括偷懒，**必须立即打回重译，严禁直接渲染！**
- **时间戳与原文对应**：保持段落的 `timestamp` 与原文一一对应。

#### 2. Core Vocabulary Extraction (15-35 items):
- `word`: Lemma
- `accent`: Stress-marked form (e.g. `отма́зка`, `претендова́ть` / `proactive /prəʊˈæktɪv/`)
- `pos`: Part of speech and aspect/gender (e.g. `глагол несов.`, `сущ. ж.р.`, `adj.`, `n.`)
- `meaning`: Clear Chinese definition
- `collocation`: Real collocation/usage from the video with translation

Save combined results into a single `bilingual_result.json`:
```json
{
  "paragraphs": [
    { "timestamp": "00:00:00", "ru": "...", "zh": "..." }
  ],
  "vocabulary": [
    { "word": "отмазка", "accent": "отма́зка", "pos": "сущ. ж. р.", "meaning": "借口，托词", "collocation": "это отмазка (这只是托词)" }
  ]
}
```

---

### Step 4: Render Final Deliverables & Zero-Clutter Cleanup
```bash
sub2study render "<OUTPUT_DIR>/bilingual_result.json" \
  -o "<OUTPUT_DIR>" \
  --title "<VIDEO_TITLE>" \
  --video-url "<YOUTUBE_URL>" \
  --speaker "<SPEAKER_NAME>" \
  --summary "<SUMMARY>" \
  --source-lang "en"
```
* **Strict Output Cleanliness Guarantee**:
  `sub2study render` automatically cleans up temporary HTML files and safely archives intermediate working JSON files to `~/.sub2study/cache/`.
  Upon completion, the target directory will contain **ONLY TWO FINAL DELIVERABLES**:
  - `[Title].pdf`: Print-ready A4 PDF generated via Headless Google Chrome, featuring natural left-aligned typography (`text-align: left`) that strictly preserves whole words without hyphen cutting.
  - `[Title].md`: Standard Markdown with bilingual cards and vocabulary table for Obsidian/Notion.
