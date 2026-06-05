---
layout: post
title: "Text to Speech for Content Consumption"
date: 2024-03-04
---

# Text to Speech for Content Consumption

Over the years, I've built up quite a lot of content that I've meant to read "some day", and I've also been trying to cut down on my screen time. A discussion [on HackerNews](https://news.ycombinator.com/item?id=39442882) prompted me to try out solving these two problems using text-to-speech (TTS) to listen to my reading list.

I've had moderate success with this (at least for some types of content - diagram or code-heavy content is not enhanced by TTS).

# Technology

I generally do most of my scripting in Python or Deno, so I wanted to have something available using one of these.

Some exploration around the ecosystem makes [Piper](https://github.com/rhasspy/piper/tree/master) look like a good TTS package to use, as it's fairly straight-forwards to use, has multiple voices available, and actually sounds fairly good.

```python
from piper import PiperVoice
import wave

voice = PiperVoice.load(model_path="./models/en_GB-alan-medium.onnx", use_cuda=False)
with open("./output.wav", mode="wb") as f:
  with wave.open(f, "wb") as wav_file:
    voice.synthesize("Hello world", wav_file)
```

## Pre-processing

I've ended up having to have some pre-processing scripts for some of the content I've been consuming with this - mostly around removing things like footnote links, or making numbered lists read nicely.

At present, this is a cluster of very bespoke Python functions.

## Post-processing

The `.wav` files output by Piper are (as-expected), pretty huge, but a very simple [FFMpeg](https://ffmpeg.org/) script solves this by converting over to `mp3`s:

```bash
ffmpeg -i ./output.wav ./output.mp3
```

It's probably possible to do something a little more refined, but this got me far enough!

## Other Scaffolding

I've put together a small script locally that means I can keep a directory of text files, and only run TTS on new ones, as well as some scripting around getting the resulting files onto my phone, in a [XSPF](https://en.wikipedia.org/wiki/XML_Shareable_Playlist_Format) playlist for VLC.

# Follow-Up Work

Some things I intend to do here, but haven't yet:

1. Scripting fetching the text from websites, and running TTS on them (again, inspired by the HN post)
2. Some tooling around using TTS for proof-reading, as I find it much easier to spot typos and awkward phrasing when listening
3. Finding a slightly less bespoke way of cleaning up the text sources I'm feeding into TTS
4. Open sourcing the project if I can make a compelling end-to-end solution
