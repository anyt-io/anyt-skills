# YouTube Video Downloader Skill

Downloads YouTube videos with customizable quality and format options using `yt-dlp`.

The `runtime/` folder under this skill contains the uv project with pinned dependencies.

## Prerequisites

- `uv` ([install](https://docs.astral.sh/uv/getting-started/installation/))
- `ffmpeg` (`brew install ffmpeg`)

## Usage

Run from the skill folder (`skills/youtube-downloader/`):

```bash
# Download a video (best quality, mp4)
uv run --project runtime runtime/download_video.py "URL"

# Specific quality
uv run --project runtime runtime/download_video.py "URL" -q 720p

# Audio only (MP3)
uv run --project runtime runtime/download_video.py "URL" -a

# Custom output and format
uv run --project runtime runtime/download_video.py "URL" -o ~/Videos -f mkv -q 1080p
```

## Options

| Flag | Description | Default |
|------|-------------|---------|
| `-o, --output` | Output directory | `skills/youtube-downloader/output/` |
| `-q, --quality` | best, 1080p, 720p, 480p, 360p, worst | `best` |
| `-f, --format` | mp4, webm, mkv | `mp4` |
| `-a, --audio-only` | Extract audio as MP3 | `false` |

## Limitations

- Single video downloads only (playlists disabled by default)
- Higher quality requires more storage and download time
