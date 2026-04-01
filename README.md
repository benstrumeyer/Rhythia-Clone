# Rhythia Clone — Plan

> Reference: [Rhythia](https://www.rhythia.com/) | [Steam](https://store.steampowered.com/app/2250500/Rhythia/) | [Original source (archived)](https://github.com/kermeow/RhythiaRewrite)
> Original engine: Godot 4.x (GDScript) by Capo Games

---

## What Is Rhythia

An aim-based rhythm game. Blocks fly toward a 3x3 grid in time with music. You aim your cursor at the correct cell and hit it on beat. Precision + rhythm + spatial awareness.

- **Core loop:** blocks approach → aim at correct cell → hit on beat → build combo
- **Scoring:** perfect timing = max score, good timing = combo alive but lower accuracy, miss = combo break
- **Difficulty scales** via speed, pattern complexity, diagonal movement, rapid direction changes

---

## What We're Cloning

### MVP
- 3x3 grid with blocks flying in from a distance
- Hit detection: cursor position + timing window (perfect / good / miss)
- Combo counter + score
- Single song playback synced to a beatmap
- Basic visual feedback (hit flash, miss shake, combo counter)
- Beatmap format: timestamped notes with grid position

### Post-MVP
- Multiple songs with difficulty ratings
- Beatmap editor / creator
- Custom map import (community maps)
- Leaderboards (local first, then online)
- Ranked multiplayer (PvP)
- Co-op mode (shared score)
- Replay system with seeking, speed adjustment, hitbox visualization
- Full customization: game elements, UI, shapes, models, presets
- Anti-cheat for ranked play

---

## Core Mechanics

### The Grid
- 3x3 cell layout, centered on screen
- Blocks spawn in the distance and fly toward the grid
- Each block targets a specific cell
- Blocks arrive at a time synced to the beat

### Timing Windows
| Rating | Window | Score | Combo |
|--------|--------|-------|-------|
| Perfect | +/- ~30ms | 100% | +1 |
| Good | +/- ~80ms | 50% | +1 |
| Miss | > 80ms | 0 | Reset |

(Tune these values during playtesting)

### Scoring
- Base points per note * timing multiplier * combo multiplier
- Accuracy % = weighted average of all hit ratings
- Final grade: S / A / B / C / D based on accuracy thresholds

### Input
- Mouse aim (cursor position determines target cell)
- Click or key to hit
- Potentially support keyboard-only mode (numpad = grid positions)

---

## Beatmap Format

```json
{
  "song": "song_name",
  "artist": "artist_name",
  "bpm": 140,
  "difficulty": 5,
  "notes": [
    { "time": 0.500, "cell": [1, 1] },
    { "time": 0.750, "cell": [0, 2] },
    { "time": 1.000, "cell": [2, 0] },
    { "time": 1.125, "cell": [1, 0] }
  ]
}
```

- `time`: seconds from song start
- `cell`: [row, col] on the 3x3 grid (0-indexed)
- Could add `type` field later for hold notes, slides, etc.

---

## Tech Stack Options

| Option | Engine | Language | Pros | Cons |
|--------|--------|----------|------|------|
| **Godot 4** | Godot | GDScript/C# | Same as original, great for 2D/3D, free, open source | Smaller ecosystem |
| **Unity** | Unity | C# | Huge ecosystem, good audio sync, WebGL export | Heavier, licensing |
| **Web (Three.js)** | Browser | TypeScript | No install, instant sharing, easy multiplayer via WebSocket | Audio sync harder, performance ceiling |
| **Bevy** | Bevy | Rust | Performance, modern ECS, fun to learn | Steep learning curve, ecosystem still maturing |

**Recommendation:** Godot 4 — matches the original, lightweight, perfect for this scope, C# support if preferred over GDScript.

---

## Architecture (Godot)

```
Main
├── AudioManager         — song playback, beat sync, timing
├── BeatmapLoader        — parse JSON beatmap, queue notes
├── Grid (3x3)
│   └── Cell[row][col]   — visual state (idle, targeted, hit, miss)
├── NoteSpawner          — instantiate blocks, fly toward grid
│   └── Note (scene)     — position, target cell, arrival time, visual
├── InputHandler         — cursor position → active cell, hit detection
├── ScoreManager         — combo, accuracy, grade calculation
├── HUD
│   ├── ComboCounter
│   ├── ScoreDisplay
│   ├── AccuracyDisplay
│   └── TimingFeedback   — "Perfect!" / "Good" / "Miss" popups
└── SongSelect (post-MVP)
    ├── SongList
    ├── DifficultySelector
    └── Leaderboard
```

### Key Timing Challenge
Audio sync is the hardest part of any rhythm game. Approach:
- Use `AudioStreamPlayer` with `get_playback_position()` for song time
- Compensate for audio output latency (let player calibrate offset in settings)
- Note arrival time = note.time - travel_duration
- Hit window checks: `abs(current_song_time - note.time) < window`

---

## Visual Style

- Dark background, neon/glowing blocks
- Blocks fly in from depth (3D perspective or faked 2D parallax)
- Grid cells glow on hover
- Hit: satisfying flash + particle burst
- Miss: cell shakes, subtle red flash
- Combo milestones: screen effects (glow intensifies, colors shift)

---

## Development Phases

### Phase 1 — Core Loop
- [ ] 3x3 grid rendering
- [ ] Single block spawning and flying toward grid
- [ ] Cursor → cell detection
- [ ] Hit timing detection (perfect/good/miss)
- [ ] Score + combo counter
- [ ] One hardcoded test beatmap

### Phase 2 — Playable Game
- [ ] Beatmap JSON loader
- [ ] Song audio sync
- [ ] Multiple test songs
- [ ] Visual feedback (particles, flashes, combo text)
- [ ] Results screen (accuracy, grade, max combo)
- [ ] Latency calibration setting

### Phase 3 — Content & Polish
- [ ] Song select screen
- [ ] Difficulty system
- [ ] Beatmap editor
- [ ] Custom map import
- [ ] Settings menu (keybinds, visual customization)
- [ ] Local leaderboards

### Phase 4 — Multiplayer
- [ ] Online PvP (real-time scoring)
- [ ] Co-op mode
- [ ] Global leaderboards
- [ ] Replay system
- [ ] Anti-cheat

---

## Open Questions

1. Godot GDScript or C#?
2. 2D with faked depth or actual 3D scene?
3. Beatmap creation — manual JSON, visual editor, or auto-generate from audio analysis?
4. How to handle audio latency calibration across different hardware?
5. Osu! beatmap import compatibility? (huge existing library)

---

## References

- [Rhythia](https://www.rhythia.com/)
- [Rhythia on Steam](https://store.steampowered.com/app/2250500/Rhythia/)
- [RhythiaRewrite (archived Godot source)](https://github.com/kermeow/RhythiaRewrite)
