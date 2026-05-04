# YouTube Heatmap Clipper: Project Analysis

## Overview

**YouTube Heatmap Clipper** is a web-based and command-line application designed to automate the extraction of the most engaging moments from YouTube videos. It utilizes YouTube's "Most Replayed" heatmap data to identify these segments. It then automatically converts these highlights into vertical video formats (such as 9:16 for Shorts, Reels, and TikTok) and can optionally overlay AI-generated subtitles using Faster-Whisper. 

The project is a "human-friendly" GUI web version of a CLI-based tool, aiming to simplify the process of clipping content for modern social media platforms.

## Core Features

- **Heatmap Extraction**: Leverages YouTube's heatmap/most replayed data to find highly engaging segments without needing a YouTube API key.
- **Vertical Video Generation**: Automatically crops videos into a 9:16 vertical format.
- **AI Subtitles**: Incorporates `faster-whisper` for fast, AI-powered transcription and automatic subtitle burning in multiple languages.
- **Custom Crop Modes**: Offers default center crop, split-left (facecam bottom-left), and split-right (facecam bottom-right).
- **Web UI & CLI**: Offers both a clean Flask-based web interface and a flexible command-line interface.
- **Batch Processing**: Allows selecting multiple heatmap segments and creating clips concurrently or sequentially.

## Technical Stack

- **Backend / Core Logic**: Python (3.8+)
- **Web Framework**: Flask (`webapp.py`)
- **Video Processing**: FFmpeg (used heavily for cropping, encoding, and burning subtitles)
- **Downloading & Metadata**: `yt-dlp` (extracts video streams and heatmap data)
- **AI Transcription**: `faster-whisper` (Optimized implementation of OpenAI's Whisper model)
- **Frontend**: HTML/CSS/JS (served from `templates/` and `static/`)

## Architecture & Workflow

### 1. Web Application (`webapp.py`)
The web application is built with Flask and serves as the UI for the underlying CLI logic.
- **State Management**: Uses thread-safe dictionaries (`jobs_lock`, `jobs`) to manage asynchronous clipping jobs.
- **Endpoints**:
  - `/api/preview`: Fetches metadata (title, thumbnail, duration) using `yt-dlp`.
  - `/api/scan`: Extracts the heatmap "Most Replayed" segments.
  - `/api/clip`: Triggers the background video processing job (`run_job`).
  - `/api/job/<job_id>`: Polling endpoint for frontend clients to track clipping progress.

### 2. Core Processing Engine (`run.py` referenced as `core`)
The heavy lifting is delegated to `run.py`. When a job is initiated:
1. **Dependency Check**: Verifies the presence of FFmpeg and optionally installs Whisper dependencies.
2. **Video Download**: Uses `yt-dlp` to download the specific segment of the YouTube video.
3. **Audio Extraction & Transcription (Optional)**: If subtitles are enabled, audio is extracted and passed to `faster-whisper` to generate an SRT or VTT file.
4. **Video Processing (FFmpeg)**:
   - **Cropping**: Applies complex FFmpeg filter graphs depending on the crop mode (e.g., stacking a game area and a facecam area).
   - **Padding**: Adds pre- and post-padding to the clip duration to ensure context is not lost.
   - **Subtitle Burning**: Burns the generated subtitles directly into the video stream using the configured font and positioning.
   - **Encoding**: Outputs an MP4 file with H.264 codec (CRF 26, ultrafast preset) and AAC audio.

## File Structure

- `webapp.py`: Flask application server and REST API.
- `run.py`: The core script that orchestrates downloading, transcription, and FFmpeg processing. It can also be run directly as a CLI tool.
- `start.bat`: Windows batch script to automate environment setup (creating a `venv`, installing requirements, and running the app).
- `requirements.txt`: Python dependencies (`flask`, `yt-dlp`, etc.).
- `README.md` & `README_EN.md`: Documentation in Indonesian and English.
- `clips/`: Output directory where generated vertical videos are saved.
- `fonts/`: Directory for custom fonts used in subtitle rendering.
- `static/` & `templates/`: Assets and HTML templates for the web interface.

## Use Cases

1. **Gaming Content**: Streamers can easily convert their long VODs into Shorts by using the split-screen crop modes to capture both gameplay and facecam during peak moments.
2. **Podcasts & Interviews**: Automatically extracting the most replayed (and likely most controversial or interesting) parts of a podcast and framing them vertically.
3. **Tutorials & Vlogs**: Quickly summarizing long-form tutorials into bite-sized clips with burned-in subtitles for silent viewing on mobile.

## Potential Areas for Enhancement

- **Queue Management**: The current threading model for jobs is basic. Integrating a task queue like Celery or RQ could improve stability under heavy loads.
- **Custom Heatmap Alternative**: If a video lacks YouTube heatmap data, an alternative like audio volume spike detection could be used as a fallback mechanism for finding interesting clips.
- **Hardware Acceleration**: Explicitly supporting NVENC (Nvidia) or VideoToolbox (Mac) for FFmpeg encoding to drastically reduce processing times.
