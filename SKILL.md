---
name: reel
description: Understand an Instagram reel, video post or photo post from its link. Use whenever the user sends an instagram.com/reel, /reels, /p or /tv link, or asks what a reel says or shows, even with no instruction beyond the link ("what's in this reel", "summarise this", "شنو بهذا الريل", "/reel <url>"). Downloads it, reads caption, metadata, audio and video with cheap local checks first, and spends Gemini or image tokens only where they are needed.
---

# /reel

## Run
```bash
python ~/.claude/skills/reel/reel.py "<url>"
```
Use the Python environment the requirements were installed into. The script prints the path of `bundle.md`.

Flags: `--force-transcribe` when the on-screen captions look incomplete, `--refresh` to download again, `--max-frames N` to change the frame cap (default 8), `--dense` when something shown within one camera shot matters (a product reveal, a prop, a gesture).

If it prints `REEL ERROR`, relay the message plainly. A cookies error means the user must export Instagram cookies (a browser extension such as "Get cookies.txt LOCALLY", while logged in) to `~/.config/reel/cookies.txt`. Never ask them to paste cookie contents into chat.

## Read
1. Read `bundle.md` in full. Text comes first: caption, Instagram metadata, song, speech, on-screen text.
2. For a video, open the overview sheet it names. Each cell is a detected moment with its time.
3. If something is still unclear, look further with the `look.py` commands printed at the end of the bundle: a sheet of any time range, or one full frame to read small text. Stop as soon as the post is understood.
4. For a photo post, read the frames listed under "Frames to read".
5. The Decisions section says what was skipped and why. If a skip looks wrong for this post (captions clearly do not match what is said), rerun with the relevant flag rather than guessing.

## Answer
Reply in the user's language. Say what the post is (format, who, topic), what is said, what is shown, the song if any, and whether the caption relates to the content or is just bait. Keep it proportional: a 10 second meme needs two lines.

Do not invent speech that is not in the bundle. If the speech source says "NOT transcribed", the post has speech nobody transcribed (no Gemini key, or the daily free quota is spent): say so, describe only what the frames and on-screen text show, and offer to rerun later. If the speech source is "none" and the audio kind is music, the post has no spoken content.

Never reproduce full song lyrics; name the song and describe it.

## Record
Write the file named on the `record file to write` line:

```markdown
---
url: <url>
account: <@account>
posted: <date>
song: <title by artist, or none>
---
# <one-line description>

<summary>

## Speech
<transcript or captions text, or none>

## On-screen text
<lines, or none>
```
