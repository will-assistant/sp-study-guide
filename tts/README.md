# SP Study Guide — TTS Audio Library

Generate spoken-word audio from study guide modules using Kokoro TTS.

## Voice Settings

| Setting | Value |
|---------|-------|
| Engine | Kokoro (via Open Speech) |
| Voice | `will` — `am_puck(1)+am_liam(1)+am_onyx(0.5)` |
| Speed | 1.1x |
| Format | MP3 (`--format mp3`) |
| Bitrate | 128kbps MP3 |
| Max chunk | ~2000 chars per TTS call |

## Pronunciation Dictionary

See [`pronunciation.txt`](pronunciation.txt) — 80+ SP protocol acronyms mapped to phonetic spellings.

Applied as a preprocessing step before TTS. Without this, Kokoro mispronounces most networking acronyms (IS-IS → "isis", OSPF → sneeze, etc.).

### Format
```
# Comments start with #
RAW_TERM|spoken replacement
```

### Rules
- Longer/compound terms come first (e.g., `SR-MPLS` before `MPLS`)
- Case-sensitive matching with word boundaries
- Periods between letters force letter-by-letter pronunciation: `I.S.I.S.`

## Pipeline

### 1. Convert markdown → spoken text
Strip tables, code blocks, bullet points. Convert to natural prose paragraphs.
Keep section headers as spoken transitions ("Next, we'll cover...").

### 2. Apply pronunciation dictionary
```bash
python3 scripts/apply-pronunciation.py <input.txt> pronunciation.txt > pronounced.txt
```

### 3. Chunk text (~2000 chars per chunk)
```bash
python3 scripts/chunk-text.py <pronounced.txt> --max-chars 2000
```

### 4. Generate audio per chunk
```bash
for chunk in chunk_*.txt; do
  kokoro-tts "$(cat $chunk)" \
    --voice will --format opus --output "${chunk%.txt}.mp3"
done
```

### 5. Concatenate chunks
```bash
# Build file list
ls chunk_*.mp3
*.ogg | sed 's/.*/file '\''&'\''/' > concat.txt
# Merge
ffmpeg -y -f concat -safe 0 -i concat.txt -c:a libmp3lame -b:a 128k output.mp3
```

### 6. Clean up
```bash
rm chunk_*.txt chunk_*.mp3
*.ogg concat.txt
```

## Module Index

| Module | Section | Duration | Status |
|--------|---------|----------|--------|
| 02 | 2.1 IS-IS Deep Dive | ~9 min | ✅ Generated (v2 w/ pronunciation) |
| 02 | 2.2 OSPF in SP Networks | | 🔲 |
| 03 | 3.x BGP Deep Dive | | 🔲 |
| 04 | 4.x MPLS Fundamentals | | 🔲 |
| 05 | 5.x Traffic Engineering | | 🔲 |
| 06 | 6.1-6.7 Segment Routing | | 🔲 |
| 07 | 7.1-7.5 L3VPN | | 🔲 |
| 08 | 8.1-8.5 L2VPN & EVPN | | 🔲 |
| 09 | 9.1-9.6 Transport & Optical | | 🔲 |
| 10 | 10.1-10.3 Network Slicing & 5G | | 🔲 |
| 11 | 11.1-11.5 Automation & Operations | | 🔲 |
| 12 | 12.1-12.5 Design Case Studies | | 🔲 |
| 01 | 1.x Foundations | | 🔲 |

## Storage

Audio files are NOT stored in git (too large — ~25 hours estimated).

Options:
- Local NAS or shared storage
- Generate on demand using this pipeline
- Future: host on a simple web server for mobile playback

## Estimated Total

- 12 modules, ~60 sections
- ~223K words source material
- ~25 hours audio at 1.1x speed
- ~300MB total at 128kbps MP3
