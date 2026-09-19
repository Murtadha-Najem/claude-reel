# claude-reel

A Claude Code skill that lets Claude understand an Instagram reel, video post or photo post from its link.

Send Claude a link and it reads the post the way a person would: the caption, what is said, what is written on screen, the song, and what is shown. Every step runs a cheap local check first, so a paid API or an image is only spent where the post actually needs it.

Built for Arabic and English content, and tested mostly on Iraqi and Gulf Arabic reels.

## What it does

For every link, the pipeline:

1. **Downloads the post** with yt-dlp and keeps Instagram's full media response, not just the handful of fields yt-dlp exposes. That gives the posting time, account type, collaborators, tagged users, location, repost notes and Instagram's own music tag.
2. **Reads the video locally.** One low resolution decode finds cuts, still stretches and text changes in the top and bottom strips. Moments are grouped into camera shots (ORB feature matching), so a two minute talking head costs one frame, not forty.
3. **Reads on-screen text locally** with RapidOCR (Arabic and Latin models), with a second pass on clipped words. A dropped negation such as "مو" changes the meaning of a sentence, so this matters.
4. **Classifies the audio locally** with PANNs CNN14 in 5 second windows: speech, singing, music, silence.
5. **Names the song**: from Instagram's own music tag when there is one, otherwise Shazam on up to three clips, accepted only when two agree.
6. **Transcribes speech only when needed.** If the burned-in captions already carry the speech (checked by words per speech second and coverage), Gemini is skipped. Otherwise one Gemini request transcribes it verbatim, keeping the dialect as spoken. Background singing is not transcribed; a song whose lyrics are written on screen is.
7. **Writes one compact `bundle.md`**: text first, then an overview sheet of the detected moments, a timeline, and a Decisions section saying what each gate chose and why.

Claude then reads the bundle, opens the overview sheet, and can look further on its own with `look.py`: a contact sheet of any time range, or a single full frame to read small text.

Photo posts and carousels are handled too: their images come from the same media response.

## How it differs from other video skills

Several good skills already give Claude a video, most of them built on [bradautomates/claude-video](https://github.com/bradautomates/claude-video). This one is narrower and goes deeper on Instagram.

| | claude-video (/watch) | instagram-reel-extractor | claude-reel |
|---|---|---|---|
| Any yt-dlp site | yes | Instagram only | Instagram only |
| Instagram metadata beyond yt-dlp | no | likes, comments | full media response |
| Song identification | no | Shazam | Instagram tag, then Shazam with agreement check |
| On-screen text (OCR) | no | no | yes, Arabic and English |
| Speech vs music detection | no | no | yes, local classifier |
| Skips transcription when captions carry it | no | no | yes |
| Photo posts and carousels | no | no | yes |
| Claude can browse the video itself | no | no | yes, `look.py` |
| Transcription | captions or Whisper | local Whisper | Gemini, dialect preserved |

If you want any site, local Whisper, or no API key at all, use claude-video. If your feed is Instagram, especially in Arabic, this is the one built for it.

## Install

Requirements: Python 3.10 or later, and `ffmpeg` and `ffprobe` on PATH.

```bash
git clone https://github.com/murtadha203/claude-reel ~/.claude/skills/reel
```

```bash
cd ~/.claude/skills/reel
```

```bash
pip install -r requirements.txt
```

```bash
python setup_models.py
```

`setup_models.py` downloads the PANNs audio model (about 330 MB) to `~/panns_data`. Without it everything still runs, but speech cannot be told from music, so Gemini is called more often.

### Instagram cookies

Instagram often refuses anonymous downloads. Export your cookies once, while logged in, with a browser extension such as "Get cookies.txt LOCALLY", and save the file as:

```
~/.config/reel/cookies.txt
```

Keep this file private. It is your Instagram session. Use a secondary account if you can.

### Gemini key

Speech transcription uses Gemini. The free tier is enough for daily use.

- set `GEMINI_API_KEY`, or
- set `GEMINI_API_KEYS` to several keys separated by commas, or
- put one key per line in `~/.config/reel/gemini_keys.txt`.

With several keys from separate Google accounts, a key whose daily quota is spent hands over to the next. With no key the pipeline still runs and marks the speech as not transcribed.

## Use

In Claude Code, just send the link:

```
/reel https://www.instagram.com/reel/XXXXXXXXXXX/
```

or paste the link with a question. From a terminal:

```bash
python reel.py "https://www.instagram.com/reel/XXXXXXXXXXX/"
```

| Flag | Use |
|---|---|
| `--force-transcribe` | transcribe even when on-screen captions seem to carry the speech |
| `--refresh` | download again instead of using the stored copy |
| `--max-frames N` | frame cap (default 8) |
| `--dense` | one frame per distinct look instead of one per camera shot |

```bash
python look.py <shortcode> sheet --start 10 --end 30 --n 12
```

```bash
python look.py <shortcode> frame 14.5
```

## Where things are kept

| Path | Contents |
|---|---|
| `~/reel-data/cache/<shortcode>/` | video, frames, metadata, transcript, `bundle.md` |
| `~/reel-data/records/` | one Markdown summary per post, written by Claude |
| `~/.config/reel/` | cookies and Gemini keys |

Nothing is deleted automatically, so a second question about the same post costs nothing. Set `REEL_HOME`, `REEL_CONFIG` or `REEL_COOKIES` to move them. `REEL_GEMINI_MODEL` changes the model.

## Layout

```
SKILL.md           instructions Claude follows
reel.py            entry point: link in, bundle path out
look.py            contact sheets and full frames on demand
setup_models.py    one-time download of the audio model
reel/
  download.py      yt-dlp, Instagram media response, photo posts
  video.py         cuts, shots, text changes, OCR, overview
  frames.py        frame store and contact sheets
  audio.py         loudness, PANNs classes, Shazam
  speech.py        caption coverage check, Gemini transcription
  bundle.py        bundle.md and bundle.json
  pipeline.py      the gates in order
```

## Notes

- Use it on public posts, or posts you have the right to view. Respect the people in them.
- Shazam access goes through the unofficial `shazamio` library and may stop working without notice. The pipeline reports that and carries on.
- Instagram changes often. When a download fails, the script upgrades yt-dlp once and retries.

## Credits

The scene-change expression and the thumbnail dedup are adapted from [bradautomates/claude-video](https://github.com/bradautomates/claude-video) (MIT). See [THIRD_PARTY.md](THIRD_PARTY.md).

## License

MIT, see [LICENSE](LICENSE).
