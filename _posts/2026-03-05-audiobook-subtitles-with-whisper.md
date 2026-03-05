---
layout: post
title: Adding Subtitles to Libro.fm Audiobooks with Whisper.cpp
lead: Transcribe DRM-free audiobooks locally and read along
---

**TLDR;** Convert the audiobook to WAV, run `whisper-cli` to generate an SRT, then mux it back into the m4b with `ffmpeg`.
{: .message }

[Libro.fm](https://libro.fm) is a great place to buy audiobooks — it's DRM-free, which means you get your audiobook as a standard [m4b file](https://en.wikipedia.org/wiki/M4B) that you actually own. I host mine on a home server and listen on any device I want.

The one thing Libro.fm audiobooks don't come with is subtitles. I find it much more immersive to read along with the narration, so here's how I add them using [whisper.cpp](https://github.com/ggerganov/whisper.cpp).

# Setup

Install whisper.cpp and a model. On Arch Linux via the AUR:

```bash
# With NVIDIA GPU (recommended)
paru -S whisper.cpp-cuda

# CPU-only fallback
paru -S whisper.cpp
```

For the model, `whisper.cpp-model-small` works well for most content. If you have more than 5.5 GB of VRAM, `whisper.cpp-model-large-v3-turbo` gives noticeably better accuracy.

# Procedure

**1. Convert the audiobook to a format whisper.cpp expects (16 kHz mono WAV):**

```bash
ffmpeg -i input.m4b -ar 16000 -ac 1 -c:a pcm_s16le 16khz_audio.wav
```

**2. Transcribe and generate an SRT subtitle file:**

```bash
whisper-cli 16khz_audio.wav -l en -pp -ps --output-srt \
  -m /usr/share/whisper.cpp-model-large-v3-turbo/ggml-large-v3-turbo.bin
```

This produces `16khz_audio.wav.srt`.

**3. Mux the subtitle back into the original audiobook, preserving chapters and metadata:**

```bash
ffmpeg -i input.m4b -i 16khz_audio.wav.srt \
  -map 0:a -map 0:v? -map 1:s \
  -map_metadata 0 -map_chapters 0 \
  -c copy -c:s mov_text \
  output.m4b
```

The resulting `output.m4b` has the embedded subtitle track and can be played back in any player that supports m4b subtitles, such as [Prologue](https://prologue.audio/) or VLC.
