---
name: guitar-riff-composer
description: Use this skill when the user wants to compose, write, generate, or get ideas for a guitar riff, lick, progression, or melody. Triggers on keywords like "riff", "guitarra", "lick", "tab", "chord progression", "componer para guitarra", or any request to create a musical idea for guitar, even if phrased casually like "dame algo para tocar", "quiero un riff de rock", "escríbeme algo en blues", or "necesito una intro de guitarra". Also triggers when the user describes a mood or genre and wants a guitar part, even without saying "riff" explicitly.
---

# Guitar Riff Composer

Composes original guitar riffs with tablature, theory context, and tone suggestions.

## Step 1 — Gather context

Ask only what's missing. Prioritize these inputs:

| Input | Examples | Required? |
|---|---|---|
| Genre / mood | rock, blues, psych, jazz, metal, funk | Yes — guess if not given |
| Feel / tempo | "pesado y lento", "funky 120bpm", "dreamy" | Optional |
| Tuning | Standard, Drop D, Open G | Optional (assume standard) |
| Reference artist | Tame Impala, SRV, Black Sabbath | Optional but very useful |
| Player level | beginner / intermediate / advanced | Optional (assume intermediate) |

If genre is missing, make a reasonable assumption and state it.  
If a reference artist is given, read `references/artist-styles.md` to capture their fingerprint before composing.

---

## Step 2 — Compose the riff

Design the riff with intention:

- Choose a **scale** appropriate to the genre (see `references/scales.md`)
- Define **time signature** (default 4/4) and rough **tempo range**
- Make it **memorable** — a strong motif, not random notes
- Keep it **playable** at the stated level — no unnecessary shred
- For psych/ambient genres (e.g. Tame Impala), consider spacious phrasing, bends, and sustained notes
- For blues, include at least one characteristic bend or slide
- For funk/R&B, emphasize rhythmic displacement and muted strums

---

## Step 3 — Format the output

Always deliver all of these sections:

### 1. Riff context
- Genre, feel, scale used
- Why it works musically (1-2 sentences)
- Time signature + suggested BPM range

### 2. Tablature (ASCII, standard 6-string)

Use this exact format:
```
e |--------------------------|
B |--------------------------|
G |--------------------------|
D |--------------------------|
A |--------------------------|
E |--------------------------|
```

Notation conventions:
- `h` = hammer-on
- `p` = pull-off  
- `b` = bend (e.g. `7b9` = bend 7th fret up to sound of 9th)
- `/` = slide up, `\` = slide down
- `~` = vibrato
- `x` = muted/dead note
- `PM` below tab = palm mute

If the riff is more than one bar, show bar lines with `|`.

### 3. Chord progression (if applicable)
List chords with Roman numeral analysis (e.g. `I - bVII - IV` in A minor).

### 4. Tone / FX suggestion
One line: amp settings or pedal chain. Keep it practical.
Example: *"Overdrive ligero (gain ~40%), EQ mid-scooped, reverb room corto"*

### 5. Variation or next step (optional but encouraged)
Suggest one variation, extension, or improv direction over the riff.

---

## Step 4 — Offer iterations

After delivering the riff, offer:
- A different feel/tempo variation
- A more complex or simplified version
- A bridge or verse counterpart
- Full chord chart if it's a progression-based riff

---

## Reference files

- `references/scales.md` — Common scales by genre with fretboard positions
- `references/artist-styles.md` — Style fingerprints: scale choices, techniques, tone per artist
