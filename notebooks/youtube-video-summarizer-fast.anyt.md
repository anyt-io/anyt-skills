---
schema: "2.0"
name: youtube-video-summarizer-fast
workdir: anyt_workspace
inputs:
  youtubeUrl: ""
  language: en
---

# youtube-video-summarizer-fast

<note id="overview">
## YouTube Video Summarizer (Fast)

This notebook creates a structured, section-by-section summary of a YouTube video with corresponding frame captures — **without downloading the full video**. Screenshots are captured directly from the stream URL using ffmpeg, making this pipeline significantly faster and lighter on disk.

### Pipeline
1. **Configure** — Provide YouTube URL and preferences
2. **Setup** — Install youtube-downloader skill via pspm
3. **Download Transcript & Cover** — Fetch transcript and cover image (no video download)
4. **Analyze** — Identify natural sections and timestamps
5. **Review** — Verify section breakdown before summarizing
6. **Capture** — Screenshot frames directly from stream (no video download)
7. **Summarize** — Generate per-section summaries
8. **Review** — Check summaries and images
9. **Compile** — Create final markdown document
</note>

<input id="video-config">
## Video Configuration

Provide the YouTube video URL and configure summary options.

<form type="json">
{
  "fields": [
    {
      "name": "youtubeUrl",
      "type": "text",
      "label": "YouTube URL",
      "required": true,
      "placeholder": "https://www.youtube.com/watch?v=..."
    },
    {
      "name": "language",
      "type": "text",
      "label": "Transcript Language",
      "default": "en",
      "description": "Language code for transcript (e.g., en, zh, ja, ko)"
    },
    {
      "name": "summaryStyle",
      "type": "select",
      "label": "Summary Style",
      "default": "detailed",
      "options": [
        { "value": "detailed", "label": "Detailed" },
        { "value": "concise", "label": "Concise" },
        { "value": "bullet-points", "label": "Bullet Points" }
      ]
    },
    {
      "name": "includeTimestamps",
      "type": "checkbox",
      "label": "Include Timestamps",
      "default": true,
      "description": "Include timestamp references in the summary"
    }
  ]
}
</form>
</input>

<shell id="install-skill">
#!/bin/bash
echo "=== Installing youtube-downloader skill ==="
npx @anytio/pspm@latest add @user/anyt/youtube-downloader -y

echo ""
echo "=== Verify Installed Skills ==="
ls -la .skills/youtube-downloader/ 2>/dev/null || true
</shell>

<task id="download-transcript-and-cover">
## Download Transcript and Cover Image

Download the transcript and cover image for the video — no video download needed.

### Requirements
1. Read `youtubeUrl` and `language` from the `video-config` input
2. Use the youtube-downloader skill to download the transcript in both JSON and text formats:
   ```bash
   uv run --project .skills/youtube-downloader/runtime .skills/youtube-downloader/runtime/download_transcript.py "<youtubeUrl>" -f json -l <language> -o transcripts/
   uv run --project .skills/youtube-downloader/runtime .skills/youtube-downloader/runtime/download_transcript.py "<youtubeUrl>" -f text -l <language> -o transcripts/
   ```
3. Download the cover/thumbnail image:
   ```bash
   uv run --project .skills/youtube-downloader/runtime .skills/youtube-downloader/runtime/download_cover.py "<youtubeUrl>" -o cover/
   ```
4. Verify the transcript files were created in `transcripts/` and cover image in `cover/`

**Output:** transcripts/<video-id>.json, transcripts/<video-id>.txt, cover/<video-id>.jpg
</task>

<task id="analyze-sections">
## Analyze Transcript and Identify Sections

Read the transcript JSON file from `transcripts/` and identify natural topic sections in the video.

### Requirements
1. Read the JSON transcript file from `transcripts/` (it contains an array of segments with `text`, `start` (seconds), and `duration` fields)
2. Analyze the content and identify 5-10 natural topic sections based on:
   - Topic changes and thematic shifts
   - Speaker transitions or pauses
   - Logical content boundaries
3. For each section, determine:
   - **Title**: A descriptive section title
   - **Start timestamp**: When the section begins (seconds)
   - **End timestamp**: When the section ends (seconds)
   - **Key topics**: Main topics covered
   - **Representative timestamp**: A good moment to capture a frame (middle of an important visual moment)
4. Save the section breakdown to `sections.json`

### Output Format (`sections.json`)
```json
{
  "videoTitle": "...",
  "totalDuration": 600,
  "sections": [
    {
      "id": 1,
      "title": "Introduction",
      "startTime": 0,
      "endTime": 45,
      "keyTopics": ["topic1", "topic2"],
      "frameTimestamp": 20,
      "transcriptExcerpt": "First 2-3 sentences of this section..."
    }
  ]
}
```

**Output:** sections.json
</task>

<break id="review-sections">
## Review Section Breakdown

Check `sections.json` to verify the identified sections make sense.

**Things to check:**
- Are the section titles descriptive and accurate?
- Are the timestamp boundaries reasonable?
- Are there too many or too few sections?
- Are the selected frame timestamps at meaningful moments?

Edit the previous task and re-run if adjustments are needed.
</break>

<task id="capture-screenshots">
## Capture Frame Screenshots (No Video Download)

Extract representative frame images directly from the video stream at timestamps identified in `sections.json` — without downloading the full video.

### Requirements
1. Read `sections.json` for the `frameTimestamp` of each section
2. Read `youtubeUrl` from the `video-config` input
3. Build a comma-separated timestamp list from all section `frameTimestamp` values
4. Use the youtube-downloader skill's screenshot tool to capture all frames in one call:
   ```bash
   uv run --project .skills/youtube-downloader/runtime .skills/youtube-downloader/runtime/screenshot_video.py "<youtubeUrl>" -t "<comma-separated-timestamps>" -o frames/ -p section
   ```
   For example, if sections.json has frameTimestamps [20, 90, 180], run:
   ```bash
   uv run --project .skills/youtube-downloader/runtime .skills/youtube-downloader/runtime/screenshot_video.py "<youtubeUrl>" -t "20,90,180" -o frames/ -p section
   ```
5. Verify the `frames/manifest.json` was created and check that screenshots were captured successfully
6. Update `frames/manifest.json` to include section metadata by merging with `sections.json`:
   - Add `sectionId` and `sectionTitle` to each frame entry

### Output Format (`frames/manifest.json`)
```json
{
  "video_url": "...",
  "video_title": "...",
  "frames": [
    {
      "timestamp": 20,
      "formatted_time": "0:20",
      "file": "section_001.jpg",
      "path": "frames/section_001.jpg",
      "success": true,
      "sectionId": 1,
      "sectionTitle": "Introduction"
    }
  ]
}
```

**Output:** frames/manifest.json, frames/section_001.jpg
</task>

<task id="summarize-sections">
## Generate Section Summaries

Read `sections.json` and the transcript to create detailed summaries for each section.

### Requirements
1. Read `sections.json` for section boundaries and the transcript JSON from `transcripts/`
2. Read the `video-config` input to determine the user's preferred summary style:
   - **Detailed**: 3-5 paragraph summary with key quotes from the transcript
   - **Concise**: 1-2 paragraph summary
   - **Bullet Points**: 5-10 key bullet points
3. If `includeTimestamps` is enabled, include `[MM:SS]` timestamp references inline
4. For each section, also extract 2-3 key takeaways
5. Save individual section summaries to `summaries/section_XX.md` (numbered 01, 02, etc.)
6. Save a combined summaries file to `summaries/all_summaries.json`

### Output Format (`summaries/all_summaries.json`)
```json
{
  "style": "detailed",
  "sections": [
    {
      "id": 1,
      "title": "Section Title",
      "startTime": 0,
      "endTime": 45,
      "summary": "The summary text...",
      "keyTakeaways": ["takeaway 1", "takeaway 2"]
    }
  ]
}
```

**Output:** summaries/all_summaries.json, summaries/section_01.md
</task>

<break id="review-content">
## Review Summaries and Frame Captures

Review the generated content before compiling the final markdown document.

**Files to check:**
- `summaries/` — Section summaries (individual `.md` files and combined JSON)
- `frames/` — Captured video frames (one per section)
- `frames/manifest.json` — Frame-to-section mapping

**Things to verify:**
- Are the summaries accurate and well-written?
- Do the frame captures show relevant content for each section?
- Are any frames blank, blurry, or showing transitions?

Re-run `capture-screenshots` with adjusted timestamps if frames need improvement.
</break>

<task id="compile-markdown">
## Compile Final Markdown Document

Create a polished markdown document combining section summaries and frame images.

### Requirements
1. Read `sections.json`, `summaries/all_summaries.json`, and `frames/manifest.json`
2. Read the `video-config` input for the video URL and preferences
3. Create a comprehensive markdown document `summary.md` with:

   **Header:**
   - Cover image: `![Video Cover](cover/<video-id>.jpg)` (from `cover/` folder)
   - Video title (from `sections.json`)
   - Original video URL (linked)
   - Total duration formatted as `HH:MM:SS`
   - Date generated

   **Table of Contents:**
   - Linked list of sections with timestamps (e.g., `[1. Introduction (0:00)](#1-introduction)`)

   **Overview:**
   - A brief 2-3 sentence overview of the entire video content

   **Section-by-Section Content:** For each section:
   - Section heading with timestamp range (e.g., `## 1. Introduction [0:00 - 0:45]`)
   - Embedded frame image: `![Section Title](frames/section_XXX.jpg)`
   - Summary text (from `summaries/all_summaries.json`)
   - Key takeaways as a bullet list

   **Key Takeaways:**
   - Combined top highlights from all sections

4. Ensure all image paths are relative to the workdir
5. Format timestamps as `M:SS` or `H:MM:SS` depending on video length

**Output:** summary.md
</task>

<note id="complete">
## Summary Complete

### Generated Files
```
anyt_workspace/
├── transcripts/
│   ├── <video-id>.json         # Structured transcript
│   └── <video-id>.txt          # Plain text transcript
├── cover/
│   └── <video-id>.jpg          # Video cover/thumbnail image
├── sections.json               # Section breakdown with timestamps
├── summaries/
│   ├── all_summaries.json      # Combined section summaries
│   └── section_XX.md           # Individual section summaries
├── frames/
│   ├── manifest.json           # Frame-to-section mapping
│   └── section_XXX.jpg         # Screenshot per section (no video download)
└── summary.md                  # Final markdown document
```

### Compared to youtube-video-summarizer
- **No video download** — frames are captured directly from the stream URL
- **Faster** — skips the largest download step
- **Less disk space** — only small JPEG screenshots, no video file

### Next Steps
- Open `summary.md` to view the complete video summary
- Share the markdown file or convert to PDF/HTML
- Re-run individual cells to adjust summaries or re-capture frames
</note>
