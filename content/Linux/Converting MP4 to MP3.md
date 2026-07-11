---
title: Converting MP4 to MP3
tags:
  - guide
  - topic-linux
date: 2026-08-22
---
Based off of [this article](https://www.fosslinux.com/44788/how-to-convert-mp4-to-mp3-in-linux.htm).

Assuming you have `ffmpeg` and `lame` installed, run the following command:

```sh
ffmpeg -i input.mp4 -vn -acodec libmp3lame -ac 2 -ab 160k -ar 48000 output.mp3
```

Here's a breakdown of each instruction from the article:
- `-i input.mp4` specifies the input file
- `-vn` disables the video stream (we only want audio)
- `-acodec libmp3lame` forces the LAME MP3 encoder
- `-ac 2` sets stereo output (2 channels)
- `-ab 160k` sets the bitrate to 160kbps (good balance of quality and size)
- `-ar 48000` sets the sample rate to 48kHz