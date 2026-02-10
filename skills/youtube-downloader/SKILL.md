---
name: youtube-downloader
description: Download YouTube videos and transcripts. Use when the user wants to download a YouTube video, extract audio from a YouTube video, or get the transcript/subtitles of a YouTube video in text, JSON, or SRT format.
---

# YouTube Downloader

## Prerequisites

Ensure `uv` and `ffmpeg` are installed before running any scripts. If missing, install with:
- `uv`: https://docs.astral.sh/uv/getting-started/installation/
- `ffmpeg`: `brew install ffmpeg`

## Download Video

Run from the skill folder (`skills/youtube-downloader/`):

```bash
uv run --project runtime runtime/download_video.py "URL" [options]
```

| Flag | Description | Default |
|------|-------------|---------|
| `-o, --output` | Output directory | `output/` |
| `-q, --quality` | best, 1080p, 720p, 480p, 360p, worst | `best` |
| `-f, --format` | mp4, webm, mkv | `mp4` |
| `-a, --audio-only` | Extract audio as MP3 | `false` |

## Download Cover Image

```bash
uv run --project runtime runtime/download_cover.py "URL" [options]
```

| Flag | Description | Default |
|------|-------------|---------|
| `-o, --output` | Output directory | `output/` |
| `-n, --name` | Custom filename (without extension) | video ID |

Downloads the highest-resolution thumbnail/cover image available for the video. The file extension (`.jpg`, `.png`, `.webp`) is detected automatically from the thumbnail URL.

## Download Transcript

```bash
uv run --project runtime runtime/download_transcript.py "URL" [options]
```

| Flag | Description | Default |
|------|-------------|---------|
| `-o, --output` | Output directory | `output/` |
| `-f, --format` | text, json, srt | `text` |
| `-l, --lang` | Language code | `en` |

If the requested language is unavailable, available languages are listed in the error output.

## Limitations

- Single video only (playlists disabled)
- Transcripts require captions to be available on the video
- Video download requires `ffmpeg` for format merging; transcript download does not
