# What-If Satire Video (a Claude skill)

Turn a narrated "what if" storyline into a vertical 1080×1920 satire video in a fixed house style: low-poly 3D crowds with goofy body language, detailed hero characters, cinematic lighting with bloom, colour grading and depth of field, a Day / Week / Month counter, serif captions, and a generated soundtrack with sound effects. The scenes are built with three.js and rendered frame by frame in headless Chromium, then muxed with ffmpeg.

Example prompt:

> What if Yo-Chi disappeared? Day 1 the queue stays. Week 1 the xmms and xdds are estranged…

## Proof it works

Videos made with this skill, posted to TikTok. Seven-day analytics for Nov 7 – Nov 13, 2026, with the change against the previous seven days:

| Post views | Profile views | Likes | Shares | Comments |
| :---: | :---: | :---: | :---: | :---: |
| **1.5M** | **37.5K** | **139.7K** | **80.2K** | **724** |
| +2,843.1% | +3,220.6% | +4,079.1% | +4,334.8% | +878.4% |

<p align="center">
  <img src="assets/tiktok-analytics-nov-7-13.png" width="360" alt="TikTok Studio analytics for Nov 7 – Nov 13: 1.5M post views, 37.5K profile views, 139.7K likes, 724 comments, 80.2K shares">
</p>

## Install in Claude (web or desktop)

1. Download `what-if-satire-video.zip` from the [latest release](../../releases/latest).
2. In Claude, open **Customize > Skills** and upload the ZIP.
3. Make sure code execution is turned on, then ask Claude for a what-if video.

## Install in Claude Code

Copy the `what-if-satire-video` folder into `~/.claude/skills/`.

## What's inside

- `what-if-satire-video/SKILL.md` holds the workflow, the house style and quality rules, recipes for detailed hero characters, and the templates for the renderer, post-processing, a helper library, the music generator and a complete reference episode.

## Before you install

Like any skill, read `SKILL.md` first. It tells Claude to install npm packages (three.js and two font packages), download free CC0 assets (the `@pmndrs/assets` npm package and a water texture from the three.js repo on GitHub), and run Node, headless Chromium and ffmpeg in its sandbox. Full-quality renders take a while: about 70–80 minutes for a 43-second video.
