# Transcript Pipeline Notebook

This repo is a small, public, reproducible reference implementation for how “paste a YouTube URL and get a transcript back” services typically work. The core artifact is a notebook that walks through URL parsing, caption retrieval, fallback speech-to-text, and basic transcript cleanup so you can learn the flow end to end and swap components as needed.

## What’s in here

- A notebook that demonstrates the pipeline step by step
- A minimal set of helpers for URL parsing and transcript normalization
- Optional fallbacks for when captions are not available

## Pipeline Overview

Most transcript services follow the same shape:

### 1. URL parsing and platform detection
Input is a video URL. The first job is to identify the platform (YouTube, Vimeo, etc) and extract the canonical video identifier.

For YouTube, that usually means:
- Parsing `v=` from `youtube.com/watch?v=...`
- Parsing the short ID from `youtu.be/<id>`

### 2. Transcript fetching
There are a few common strategies, usually attempted in this order:

#### Option A: Native captions endpoint (preferred)
If the video has captions, the fastest and cheapest path is to fetch caption tracks directly. For YouTube, this is typically done via a private (but widely used) captions endpoint that returns caption tracks in XML or VTT. Libraries like `youtube-transcript-api` wrap this approach.

#### Option B: Subtitle extraction via `yt-dlp`
If you need more control (or the direct caption path fails), `yt-dlp` can extract subtitle files and metadata. Many transcript APIs use `yt-dlp` under the hood because it handles a lot of edge cases around formats and availability.

#### Option C: Audio download + speech-to-text (fallback)
If captions do not exist, the pipeline downloads the audio stream (commonly via `yt-dlp`) and runs speech-to-text using a model or service such as Whisper, Google STT, or Deepgram. This is slower and more expensive than using captions, so it’s typically a fallback.

### 3. Post-processing
Raw transcripts tend to be messy:
- Filler words
- Missing punctuation
- Broken sentence boundaries
- Weird spacing and casing
