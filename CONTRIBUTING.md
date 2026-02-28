# Contributing to the SP Study Guide

How to write, validate, and produce audio for sections in this guide.

## Repo Structure

```
sp-study-guide/
├── modules/
│   ├── 01-foundations/
│   │   └── 1.1-1.4-sp-foundations.md
│   ├── 02-igp/
│   │   ├── 2.1-isis-deep-dive.md
│   │   ├── 2.2-ospf-in-sp-networks.md
│   │   └── ...
│   └── 12-case-studies/
├── practice-exams/
│   ├── README.md
│   ├── mock-exam-1.md
│   └── module-XX-topic.md
├── tts/
│   ├── README.md
│   ├── pronunciation.txt
│   ├── scripts/
│   │   ├── apply-pronunciation.py
│   │   ├── chunk-text.py
│   │   └── generate-audio.sh
│   └── audio/          (gitignored — generated artifacts)
│       ├── 01-foundations/
│       ├── 02-igp/
│       └── ...
├── labs/               (hands-on lab exercises)
├── CONTRIBUTING.md     (this file)
├── README.md
├── PLAN.md
├── STATE.md
├── SOURCES.md
├── STUDY-PATH.md
├── TEMPLATE.md
└── LAB-ENVIRONMENT.md
```

## Writing a Section

### Use the Template

Every section follows `TEMPLATE.md`. The structure:

1. **Title + intro quote** — one-paragraph hook explaining why this matters
2. **Theory** — how it works, why it was designed this way
3. **Configuration** — IOS-XR and Junos side by side
4. **Verification** — show commands with annotated output
5. **Troubleshooting** — real failure scenarios and fixes
6. **War Stories** — at least one real-world incident
7. **Review Questions** — 5+ questions with answers
8. **Cert Relevance** — IE-SP objective mapping
9. **Sources** — RFCs, books, vendor docs

### File Naming

```
{module_number}.{section_number}-{kebab-case-title}.md

Examples:
  3.1-bgp-fundamentals-at-sp-scale.md
  6.4-srv6-fundamentals.md
  12.2-dci-with-evpn.md
```

### Style Rules

- **Operations-first.** Lead with "why" and "when it breaks," not RFC definitions
- **Dual-vendor.** IOS-XR and Junos configs in parallel — always
- **No vendor favoritism.** If Junos does something better, say so. Same for IOS-XR
- **War stories are mandatory.** Every section needs at least one real failure scenario
- **Progressive depth.** Build on prior modules. Reference them explicitly
- **No trademark cert names.** Use "IE-SP" or "expert-level" — never vendor cert brands

## Validating Content

Every section must pass this 6-point checklist before it ships:

### 1. RFC Accuracy
- All protocol behaviors validated against the governing RFC
- RFC number cited for every normative claim
- No "Cisco says X" without verifying the RFC agrees

### 2. Multi-Vendor Parity
- IOS-XR and Junos configurations are functionally equivalent
- Neither vendor presented as "the right way"
- Vendor-specific quirks called out explicitly (e.g., "Junos uses inet.3 for resolution")

### 3. Operational Realism
- Config examples use realistic values (real-world ASNs, plausible topologies)
- Troubleshooting scenarios based on actual failure modes, not textbook abstractions
- War stories are grounded in real operational experience

### 4. Technical Depth
- Covers the topic at expert-level depth — not just "what" but "why" and "what if"
- Edge cases and gotchas documented
- Scale implications discussed (what works at 10 routers vs 10,000)

### 5. Internal Consistency
- Cross-references to other sections are accurate (e.g., "See Section 4.2 for label operations")
- Terminology is consistent across the guide
- No contradictions with prior modules

### 6. No Vendor Trademark Issues
- Zero references to trademarked cert names (CCIE, JNCIE, CCNP, etc.)
- Use "IE-SP" for expert-level SP references
- External links to vendor sites are fine; branding in our content is not

## Generating Audio (TTS Pipeline)

### Prerequisites

- Kokoro TTS engine (via Open Speech, running locally)
- Python 3.10+
- ffmpeg (for MP3 concatenation)
- The `will` voice preset: `am_puck(1)+am_liam(1)+am_onyx(0.5)` at 1.1x speed

### Step 1: Write the Spoken Text

Convert the markdown section into natural spoken prose. This is a manual step — not automated.

**Rules for spoken text:**
- Strip all tables, code blocks, and bullet formatting
- Convert technical content to natural sentences
- Keep section headers as spoken transitions ("Next, we'll cover...")
- Expand abbreviations on first use ("BGP — Border Gateway Protocol")
- Remove anything that only makes sense visually (diagrams, ASCII art)
- Target 4,000-5,000 characters per section (~5-10 minutes of audio)

Save as: `{section-number}-{title}-spoken.txt` in the working directory.

### Step 2: Apply Pronunciation Dictionary

```bash
python3 tts/scripts/apply-pronunciation.py spoken.txt tts/pronunciation.txt > pronounced.txt
```

The dictionary (`tts/pronunciation.txt`) maps 80+ SP acronyms to phonetic spellings. Without it, the TTS engine mispronounces most networking terms.

**Adding new terms:** Edit `pronunciation.txt` using the format:
```
RAW_TERM|spoken replacement
```
Longer/compound terms must come first (e.g., `SR-MPLS` before `MPLS`). Case-sensitive with word boundary matching.

### Step 3: Chunk the Text

```bash
python3 tts/scripts/chunk-text.py pronounced.txt --max-chars 2000
```

Splits on paragraph boundaries. Each chunk must be under ~2000 characters for the TTS engine.

### Step 4: Generate Audio

```bash
for f in chunk_*.txt; do
  base="${f%.txt}"
  kokoro-tts "$(cat $f)" \
    --voice will --format mp3 --output "${base}.mp3"
done
```

### Step 5: Concatenate to Final MP3

```bash
ls chunk_*.mp3 | sort | sed "s|.*|file '&'|" > concat.txt
ffmpeg -y -f concat -safe 0 -i concat.txt -c:a libmp3lame -b:a 128k output.mp3
```

### Step 6: Verify and Store

```bash
ffprobe -v quiet -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 output.mp3
```

Move the final MP3 to `tts/audio/{module-dir}/{section}.mp3`.

Clean up working files:
```bash
rm -f chunk_*.txt chunk_*.mp3 concat.txt pronounced.txt
```

### Audio Settings Reference

| Setting | Value |
|---------|-------|
| Engine | Kokoro (Open Speech) |
| Voice | `will` = `am_puck(1)+am_liam(1)+am_onyx(0.5)` |
| Speed | 1.1x |
| Format | MP3, 128kbps |
| Max chunk size | ~2000 characters |
| Storage | `tts/audio/{module}/` (gitignored) |

## Practice Exams

### Format
- Scenario-based, config-analysis, or troubleshooting questions
- Multiple-choice with a single best answer unless noted
- Answers and explanations follow each section
- Calibrated to expert-level — expect to struggle

### Validation
- Every answer verified against the governing RFC
- Distractor options must be plausible (not obviously wrong)
- Explanations cite specific RFC sections or operational evidence
- Dual-reviewed by two independent reviewers before merge

## Commit Guidelines

- One logical change per commit
- Commit message format: `module X.Y: brief description`
- Audio files are gitignored — do not commit them
- `STATE.md` tracks progress — update it when completing sections
