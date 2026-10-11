---
name: audio-pipeline
contains: narration-tts-timing-and-mux-pipeline-for-code-generated-video
description: Use when a code-generated video needs narration, voice, or music muxed in. Covers choosing a voice, writing a timing-locked narration script, generating TTS that matches the video length, and the final mux command. Triggers on "add narration / voiceover", "voice for the video", "align audio to scenes", "burn in subtitles", "mux audio".
---

# Audio Pipeline — narration, timing, mux

Code-generated video renders **silent**. Audio is a separate, muxed step. This skill keeps
narration locked to the shot list so the two never drift.

## 1. Write the script against the shot list, not the clock

One line per scene. Put the **start second** next to each line, taken directly from the
shot list in `references/06-storyboard-prompting.md`. This is the source of truth:

| # | Start (s) | Line |
|---|---|---|
| 1 | 0.0 | ... |
| 2 | 4.5 | ... |

Rule: a line must fit inside its scene's duration at normal delivery speed (~4–5 CJK
characters/second). If it doesn't, shorten the line — never stretch the video silently.

## 2. Choose a voice (offer a choice, don't guess)

If the user hasn't named a voice, offer a small set (documentary, energetic, warm) rather
than picking silently. For attribute-described voices use `search_media`; for browsing use
`show_media_selector`. See the `asset-selection` skill for the voice pre-flight rule.

## 3. Generate TTS that matches the video length

- Target exactly the video duration (e.g. 48.0s). Note the TTS pipeline can add a
  little head/foot silence — trim to the video length before muxing.
- Prefer lossless (WAV) for the master; convert to MP3/AAC only at mux time.
- Verify duration: `ffprobe -v error -show_entries format=duration -of csv=p=0 narration.mp3`

## 4. Mux (the universal final step)

Keep video stream untouched, add/replace the audio stream:

```bash
# add audio (video has none)
ffmpeg -y -i out/video.mp4 -i audio/narration.mp3 \
  -c:v copy -c:a aac -b:a 192k -shortest out/final.mp4

# replace an existing track (explicit stream maps)
ffmpeg -y -i out/video.mp4 -i audio/narration.mp3 -map 0:v -map 1:a \
  -c:v copy -c:a aac -shortest out/final.mp4
```

`-shortest` prevents audio trailing past video. If narration is longer than the video,
re-cut the narration, don't just crop it mid-word.

## 5. Music bed (optional)

Mix a quiet bed under the voice with `amix`:

```bash
ffmpeg -y -i narration.mp3 -i music.mp3 -filter_complex \
  "[1:a]volume=0.18[m];[0:a][m]amix=inputs=2:duration=first:dropout_transition=2[a]" \
  -map "[a]" -c:a aac -b:a 192k out/mix.m4a
```
Then mux `out/mix.m4a` as in step 4. Fade the music out over the last ~2s:
`afade=t=out:st=<len-2>:d=2`.

## 6. Burned subtitles (optional)

Only if the video must carry text subtitles. Generate an SRT aligned to the script, then:

```bash
ffmpeg -i in.mp4 -vf "subtitles=subs.srt:force_style='FontSize=24,FontName=NotoSansSC'" \
  -c:a copy out.mp4
```

For styled/animated captions in a code-generated video, prefer an in-engine caption
component over burned SRT — see the `video-captions` skill.

## Pitfalls
| Symptom | Cause | Fix |
|---|---|---|
| Voice drifts from visuals | script not timed to shots | lock each line to its scene start |
| Audio cut mid-word | narration longer than video | shorten script or extend video |
| Silent output | forgot the mux step | run step 4 |
| Music drowns voice | wrong level | bed volume 0.15–0.2, sidechain if available |
| Audio ends abruptly | no fade | `afade` out last 2s |
