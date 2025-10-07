## Project 509A- Python composer- current in wk1 module
### GOAL: Music Composer 
<p> driven by **Python**, **automatically** creating simple melodies </p>

### About Melodies
<p> A melody is the sequence of musical notes, or pitches, arranged in a specific rhythm.Music notes are symbols representing sounds of specific pitches and durations, using letters A through G in the Western system.</p>

### Thinking in modularized way
- Input
   - the program can stick to notes within a specific scale, like C Major or A Minor,
   - different notes shall be coded/transport in someway the program can pick and "write down" in sequence
   - the program may request some input from user as initial start, like a number or letter of note, bpm or scale.

- Output
   - Ideally, I think the program should produce a music score and can connect to some player for the notes. This requires them speak the same language about melody.
   - the music might look like: ~note/pause, note pitch, note duration~,  ~note/pause, note pitch, note duration~,etc.

- Representation
   - I prefer Strings the most, it contains the complete info for a note- duration and pitch.
   - maybe just use the beat and rhythm from music theory, it can be calculated using bpm

- Logic
   - random pick from musical scale?
   - maybe generate music for a fixed amount of time or fixed amount of note




## Assistant's draft design

Below is the assistant's draft design. It expands the idea into a modular plan you can implement in Python.

### Goal
Drive a small Python program that automatically composes simple melodies and exports them in human- or machine-readable formats (plain text, CSV, or MIDI).

## Design overview
This README documents a simple, modular design for the composer. It focuses on clear inputs, outputs, internal representations, and the generation logic. It also includes a short pseudocode roadmap and Git instructions to commit your work.

## Inputs
- Required program inputs
   - scale/key (e.g., C major, A minor) — determines allowed pitches.
   - tempo in BPM — used to convert beat lengths into real-time durations.
   - length: either number of measures, total duration in seconds, or number of notes.
- Optional inputs
   - seed note or motif (one or more notes to start from).
   - rhythmic pattern(s) or probability distributions for note durations.
   - musical style or constraints (e.g., mostly stepwise motion, prefer tonic/dominant).
   - allowed octaves and voice range (e.g., C4–C6).

Should the program request input from users?
- Case for interactive input: good for experiments and demos. It helps students explore how changing tempo, scale, or seed motifs affects the output. Simple CLI prompts or a short GUI are sufficient.
- Case for non-interactive/config-driven runs: better for reproducible experiments, batch generation, or automated tests. Use a configuration file (YAML/JSON) or command-line arguments to enable reproducible runs and CI-friendly behavior.

Recommendation: Support both. Provide sensible defaults so the program can run without user input, but allow interactive prompts or CLI flags for customization.

## Outputs
- Primary outputs
   - Plain-text melody representation (one note per line): e.g., "C4 quarter", "D4 eighth", "rest quarter".
   - CSV/TSV table with columns (time, pitch, duration, velocity, tie): easy to import into spreadsheets.
   - Standard MIDI file (.mid) for playback in DAWs or a media player.
- Secondary outputs
   - Simple score-like ASCII or MusicXML for notation programs (MusicXML is more work but more compatible).
   - Audio render (WAV/MP3) via a soundfont synth — useful but optional.

Example generated plain-text
```
# Melody (scale=C major, tempo=120)
0.00,C4,quarter
0.50,D4,eighth
0.75,E4,eighth
1.00,rest,eighth
1.25,G4,quarter
```

## Representation
- Note identity
   - Use a compact string format: <pitch><octave> (e.g., C4, D#5, A3). Use `rest` for silence.
   - Internally map pitches to MIDI note numbers (60 == C4) for MIDI export and easy transposition.
- Duration / time
   - Represent durations as fraction-of-beat units (e.g., 1.0 = quarter note at quarter-note beat) or absolute beats. Convert to seconds using tempo (BPM) when building audio/MIDI timestamps.
   - Example durations: whole=4.0, half=2.0, quarter=1.0, eighth=0.5, sixteenth=0.25.
- Data structure (per note)
   - A note object/dict: {start_beat: float, pitch: str|int, duration_beats: float, velocity: int, tie: bool}
   - A melody: ordered list of note objects.

## Logic (how to decide what comes next)
Several approaches can be mixed incrementally:

- Random-but-constrained
   - Uniformly pick notes from the allowed scale within a chosen octave range. Randomly choose durations from a rhythmic distribution.

- Markov chain (n-gram over pitches/durations)
   - Build a small n-gram model from example melodies (or handcrafted transition tables). Sample next note based on previous 1–2 notes for more musical continuity.

- Rule-based heuristics
   - Prefer stepwise motion (intervals of 1–2 scale degrees) most of the time; occasionally allow leaps.
   - Favor cadence tones (tonic/dominant) at phrase ends.
   - Enforce simple harmonic rhythm: change harmony every measure or every two beats.

- Probabilistic grammars / constraints
   - Use weighted probabilities for durations and intervals.
   - Ensure phrases have a beginning (unstable) and end (stable) by increasing tonic probability near phrase ends.

Termination conditions
- Fixed-note count: stop after N notes.
- Fixed-duration: stop after M beats or S seconds.
- Musical phrase completion: stop at a cadence or when phrase rules signal completion.

## Pseudocode / Roadmap
```
config = read_config_or_args()
scale = build_scale(config.key)
melody = []
state = initialize_state(seed=config.seed)
while not termination_condition(state, melody, config):
      choice = choose_next_note(state, scale, config)
      melody.append(choice)
      state = update_state(state, choice)
export_melody(melody, formats=["text","csv","midi"])
```

Key functions to implement in order
1. scale and pitch utilities (note <-> MIDI number, transpose)
2. durations and tempo conversions
3. simple random composer (baseline)
4. output exporters (text, csv, MIDI)
5. add Markov/rule-based generator and configuration options

## Extensions and blockers
- Blockers for longer/complex pieces
   - Musical structure: generating convincing long pieces requires hierarchical structure (motifs, development, harmony), which is significantly more complex than single-voice random notes.
   - Harmony and voice-leading: supporting multiple voices and chords needs additional modeling and increases combinatorial complexity.
   - Evaluation: measuring "musicality" is subjective; automatic fitness functions are hard to design.
   - Timbral/audio rendering: realistic audio output needs sampling, synthesis, or soundfont handling.

Possible extensions
- Multi-track support (bass, harmony, melody) with simple chord progressions.
- Export MusicXML for notation or integrate with MuseScore via MusicXML.
- Add a tiny web UI or Jupyter notebook for interactive parameter tweaks and playback.

## Git & GitHub: quick steps
1. Initialize local repo:
```
git init
git add README.md
git commit -m "Add design README for Python composer"
```
2. Create a GitHub repo and push:
```
git remote add origin <your-github-remote-url>
git branch -M main
git push -u origin main
```

## Next steps (implementation)
1. Create a minimal Python package with utilities for note conversion and a baseline random composer.
2. Add export to MIDI (use python-midi, mido, or pretty_midi) and simple playback instructions.
3. Add tests for scale generation, pitch conversion, and exports.

---