# AGENTS.md - Instructions for Autonomous Coding Agents (Codex, Cursor, Windsurf)

## Purpose
`sub2study` provides automated processing of raw YouTube subtitles into publication-grade bilingual study guides (PDF + Markdown).

## Agent Workflow
When requested by the user to process a video URL:

1. **Extract & Clean**:
   Execute CLI command:
   ```bash
   sub2study extract "<URL>" -o "./output" --lang auto
   ```
   Reads `./output/cleaned_paragraphs.json`.

2. **Proofread & Translate**:
   Process `./output/cleaned_paragraphs.json` through the Agent's LLM reasoning:
   - Fix ASR speech-to-text anomalies, clause punctuation, and proper nouns.
   - **Zero-Omission Full Translation (Strict)**: 100% faithful sentence-by-sentence translation into natural Chinese. **Never summarize or compress narrative lectures into bullet points**. Keep all anecdotes, speaker transitions, and concrete examples.
   - **Density Sanity Check**: Chinese character count should be $\ge 1.5\times$ English word count (red flag if $< 1.3\times$).
   - Extract 15-25 key vocabulary terms (word, IPA/stress, POS, meaning, video collocation).
   - Write to `./output/bilingual_result.json`.

3. **Render PDF & Deliver (Zero Clutter)**:
   Execute CLI command:
   ```bash
   sub2study render "./output/bilingual_result.json" -o "./output" --title "<TITLE>" --video-url "<URL>" --source-lang "en"
   ```
   `sub2study render` automatically removes intermediate HTML and archives working JSON files to `~/.sub2study/cache/`.
   Verify that `./output` contains **strictly only** `<TITLE>.pdf` and `<TITLE>.md`.
