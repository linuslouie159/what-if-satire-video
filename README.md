# What-If Satire Video (a Claude skill)

Turn a narrated "what if" storyline into a vertical 1080×1920 satire video in a fixed house style: low-poly 3D characters with goofy body language, a Day / Week / Month counter, serif captions, and a generated soundtrack with sound effects. The scenes are built with three.js and rendered frame by frame in headless Chromium, then muxed with ffmpeg.

Example prompt:

> What if Yo-Chi disappeared? Day 1 the queue stays. Week 1 the xmms and xdds are estranged…

## Proof it works

Videos made with this skill, posted to TikTok. Seven-day analytics for Nov 5 – Nov 11, 2026, with the change against the previous seven days:

| Post views | Profile views | Likes | Shares | Comments |
| :---: | :---: | :---: | :---: | :---: |
| **1.3M** | **31.9K** | **117.1K** | **77.1K** | **646** |
| +4,359.4% | +5,079.3% | +7,281% | +14,474.5% | +1,166.7% |

<p align="center">
  <img src="assets/tiktok-analytics-nov-5-11.png" width="360" alt="TikTok Studio analytics for Nov 5 – Nov 11: 1.3M post views, 31.9K profile views, 117.1K likes, 646 comments, 77.1K shares">
</p>

## Install in Claude (web or desktop)

1. Download `what-if-satire-video.zip` from the [latest release](../../releases/latest).
2. In Claude, open **Customize > Skills** and upload the ZIP.
3. Make sure code execution is turned on, then ask Claude for a what-if video.

## Install in Claude Code

Copy the `what-if-satire-video` folder into `~/.claude/skills/`.

## What's inside

- `what-if-satire-video/SKILL.md` holds the workflow, the house style rules, and the templates for the renderer, the music generator and a complete reference episode.

## Before you install

Like any skill, read `SKILL.md` first. It tells Claude to install npm packages (three.js and two font packages) and to run Node, headless Chromium and ffmpeg in its sandbox.
