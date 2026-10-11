# video-codegen-skills

Three add-on skills for the `video-codegen` kit. Together they close the quality gaps
that show up when a code-generated video has to look *premium*, render *offline-safe*,
and carry *time-locked narration*.

| Skill | Fixes |
|---|---|
| `motion-design-recipe` | Flat, functional output -> depth / rhythm / grain+glow / threaded continuity / fast-slow pacing. |
| `font-strategy` | Blank-at-frame-0 text and CJK tofu boxes; makes text render on a machine with no network. |
| `audio-pipeline` | Hand-rolled narration; standardizes script timing, voice choice, TTS length lock, mux/music/subtitle commands. |

Core principle (unchanged): **every frame is a pure function of the frame index**.
No wall-clock, no unseeded randomness, pinned dependencies, `yuv420p`, even dimensions.
Output is code + a render tutorial only — never rendered locally.

## Install as skills

Drop each folder into your skills directory, or paste the `SKILL.md` frontmatter + body
into your agent's skill index. Frontmatter `name` / `description` drive routing.

## Layout

```
video-codegen-skills/
  README.md
  skills/
    motion-design-recipe/SKILL.md
    font-strategy/SKILL.md
    audio-pipeline/SKILL.md
```
