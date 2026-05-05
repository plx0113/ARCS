# ARCS: The Anchor-Relative Coordinate System
### A Technical Grimoire for Guitar as a Solved Spatial Problem

**Version 0.2 — May 2026**
**Author: plx0**

---

## What This Is

ARCS is a formal coordinate system for guitar. It treats the instrument not as a musical object but as a spatial one — a 6×23 grid that a five-point kinematic system (your hand) navigates under biomechanical constraints.

If that sounds like robotics, that's because it is. ARCS was designed to describe everything a human hand can physically do on a guitar fretboard, and to make every one of those positions computable. The original goal: build a robot hand that plays guitar. The byproduct: a complete mathematical description of guitar dexterity that has never existed before.

This document explains the system from the ground up. No music theory required. If you understand coordinate pairs and basic logic, you're good.

---

## The Guitar Is a Grid

Forget music for a second. A guitar is a physical object with a measurable surface. The front of the neck has 6 horizontal strings crossing 23 vertical frets, creating a grid of 138 cells. Each cell is a unique location where a fingertip can press a string against a fret.

```
        Fret 1    Fret 2    Fret 3    ...    Fret 23
String 1 (E2)  [ (1,1) ]  [ (1,2) ]  [ (1,3) ]  ...  [ (1,23) ]
String 2 (A2)  [ (2,1) ]  [ (2,2) ]  [ (2,3) ]  ...  [ (2,23) ]
String 3 (D3)  [ (3,1) ]  [ (3,2) ]  [ (3,3) ]  ...  [ (3,23) ]
String 4 (G3)  [ (4,1) ]  [ (4,2) ]  [ (4,3) ]  ...  [ (4,23) ]
String 5 (B3)  [ (5,1) ]  [ (5,2) ]  [ (5,3) ]  ...  [ (5,23) ]
String 6 (e4)  [ (6,1) ]  [ (6,2) ]  [ (6,3) ]  ...  [ (6,23) ]
```

Every cell address is a tuple: `(string, fret)`. String 1 is the thickest string (low E). Fret 1 is closest to the headstock. This is the **front plane** — where your fingers go.

But there's a second grid most people forget about.

---

## The Back Plane: Where the Thumb Lives

Flip the guitar over. The back of the neck is a mirror image of the front — same 6 string positions running horizontally, same 23 fret positions running vertically. Your thumb sits on this back plane and acts as the **anchor** for your entire hand.

In ARCS, the thumb position is called **T** and it's defined the same way: `T(y, x)` where `y` is the string axis (1 = behind the low E string, 6 = behind the high e string) and `x` is the fret axis.

This is the key insight that makes ARCS different from every other guitar notation system: **the thumb is not a fixed reference point. It moves in two dimensions, and every time it moves, it changes what your fingers can reach.**

When your thumb is at `T(1, 5)` — behind the low E string at the 5th fret — your hand is in one configuration. Move it to `T(4, 5)` — behind the G string at the same fret — and your hand physically rotates around the neck. Your reach envelope changes shape. Notes that were impossible become possible. Notes that were easy become a stretch.

The thumb isn't just supporting the hand. It's the origin of a coordinate system, and everything else is measured relative to it.

---

## Fingers as Displacement Vectors

With T as the anchor, each finger is a **displacement vector** from that anchor point. Your index finger (F1), middle finger (F2), ring finger (F3), and pinky (F4) each land somewhere on the front plane — or they don't land at all (lifted).

An ARCS **expression** is a complete snapshot of the left hand at a moment in time:

```
(T_y, T_x, F1, F2, F3, F4)
```

Where each finger is either `None` (lifted) or a cell address `(string, fret)`.

Examples:

```
T(1,1)|F1(4,1)|F2(2,2)|F3(3,2)|F4(-)     ← Open E Major chord
T(1,5)|F1(1,5)|F2(-)|F3(2,7)|F4(3,7)     ← A5 Power Chord at fret 5
T(3,5)|F1(-)|F2(-)|F3(-)|F4(-)            ← All fingers lifted (open strings)
```

The notation reads left to right: thumb position, then each finger's placement. A dash means that finger isn't touching a string.

---

## The Reach Envelope: Why Not Everything Is Possible

Here's where it gets interesting — and where the human hand imposes its rules on the math.

If fingers could go anywhere, each finger would have 138 possible positions plus one lifted state. With four fingers, that's roughly 139⁴ ≈ 374 million combinations per thumb position, times 138 thumb positions. Trillions of configurations. Most of them would require fingers that bend backward, stretch across twelve frets, or pass through each other.

The **reach envelope** is the set of cells each finger can actually reach from a given T position. It's shaped by anatomy:

**Your hand is not symmetrical around the thumb.** Fingers reach much farther toward the bridge (up) than toward the headstock (down). Your index finger is the only one that reaches significantly behind the thumb.

**The thumb's vertical position changes everything.** When T is low on the neck (y=1, behind the low E), the hand sits more parallel to the strings and the reach is compact. When T is high (y=4 or above), the hand wraps around the neck, the wrist opens up, and fingers can extend dramatically farther — sometimes doubling their reach.

**The ring finger is the weakest link — but not always.** F3 has the least independent reach at low T positions. But at high T positions where the wrist opens, it catches up or even exceeds the other fingers. This isn't a linear relationship. It's a biological quirk of tendon coupling.

**Strings far from the thumb get bonus reach.** When T is on the low E side, fingers reaching toward the high e side get roughly one extra fret of backward reach. This is consistent regardless of T position — it's a geometric consequence of the hand curving around the neck.

### Measured Reach Data

ARCS uses empirically measured reach limits. The reference calibration was performed on a Schecter C-6 Deluxe at four thumb positions, measuring each finger's maximum comfortable reach in both directions across both string groups.

The raw numbers show the reach is not uniform, not linear, and not what you'd guess:

**At T(1, 5) — thumb low, fret 5:**
- F1 reaches 2 frets behind T (3 on high strings), 3 ahead
- F4 reaches 0 frets behind, 4-5 ahead
- Total envelope: modest, compact

**At T(4, 5) — thumb mid-high, same fret:**
- F1 reaches 3-4 frets behind, 5 ahead
- F4 reaches 0-1 behind, 9 ahead
- Total envelope: massive, almost three times the area

The reach engine interpolates between measured positions to estimate the envelope at any T location on the neck.

---

## BSA: The Right Hand

ARCS describes the left hand (fretting hand). The right hand gets its own model: the **Binary Strike Actuator**, or BSA.

The BSA is simpler because the right hand makes a binary decision per string: strike it or don't. With 6 strings, that's a 6-bit vector. Add a direction bit (downstroke D or upstroke U), and you get:

```
2 directions × (2⁶ - 1) non-null patterns = 126 BSA states
```

We subtract 1 because striking zero strings produces no sound and isn't a meaningful event.

BSA notation looks like this:

```
D[EADGBe]    ← Downstroke, all 6 strings
D[EAD---]    ← Downstroke, strings 1-2-3 only
U[---GBe]    ← Upstroke, strings 4-5-6 only
D[-A----]    ← Downstroke, string 2 only
```

Each letter appears if the string is struck, replaced by a dash if it's not. The letters correspond to the open note names of each string in standard tuning.

---

## The Bang Point: Where Sound Happens

A **bang point** is the moment where the left hand configuration meets the right hand action and produces sound. It's the atomic unit of guitar output:

```
ARCS Expression × BSA Pattern → MIDI Notes
```

The calculation is straightforward. For each string that the BSA strikes:

1. Check if any finger is fretting that string
2. If yes, the sounding note is the open string pitch plus the fret number (in semitones)
3. If no finger is on that string, the open string sounds

In MIDI terms:

```
String 1 (E2) open = MIDI 40
String 2 (A2) open = MIDI 45
String 3 (D3) open = MIDI 50
String 4 (G3) open = MIDI 55
String 5 (B3) open = MIDI 59
String 6 (e4) open = MIDI 64

Sounding note = open_midi[string] + fret_number
```

If multiple fingers are on the same string (which the constraint engine prevents in most cases, but could occur with advanced techniques), the highest fret wins — it's the one closest to the bridge and therefore the one that determines the vibrating string length.

Full bang point notation:

```
T(1,1)|F1(4,1)|F2(2,2)|F3(3,2)|F4(-) × D[EADGBe] → E2 B2 E3 G#3 B3 E4
```

Read it as: "With the thumb behind the low E at fret 1, index on G string fret 1, middle on A string fret 2, ring on D string fret 2, pinky lifted — downstroke all six strings — produces E major."

---

## The Expression Library: How Big Is Guitar?

This is where it gets fun.

The expression library is the complete set of all legal ARCS expressions — every physically possible left-hand configuration on the guitar. Not every configuration that sounds good. Not every chord in a theory textbook. Every single position a human hand can achieve on the fretboard, filtered only by biomechanical reality.

The numbers, computed from the reference hand calibration:

```
Total T positions (back plane):                 138
Average legal expressions per T position:   5,490,173
Estimated total left-hand expressions:    757,643,851
× 126 BSA right-hand patterns:
Total bang points:                     95,463,125,226
```

**95.4 billion bang points.** From one pair of hands on one guitar.

For context: chess has roughly 10⁴⁴ possible games, but only about 20-30 legal moves from any given position. Guitar has 15.4 million legal left-hand positions from a single thumb placement (the maximum, at T(4,5)), each of which pairs with 126 right-hand actions. The branching factor at any given moment is in the millions.

And these numbers are conservative — they don't include barre chords (one finger pressing multiple strings), harmonics, bends, slides, hammer-ons, pull-offs, or any other technique that modifies the simple "finger presses string at fret" model.

The smallest envelope is T(6, 20) — thumb behind the high e string at the 20th fret, pinned into the far corner of the neck — with 548,608 expressions. Even the worst-case position has over half a million legal configurations.

Nobody has ever quantified this before. The full dexterity of the human hand on a guitar, expressed as a finite, enumerable set. Every chord, every voicing, every weird angular stretch your jazz teacher showed you — it's in there. Every position you've never tried is also in there. The library doesn't judge. It just lists what's possible.

---

## The Code: How It Works

The generator is a Python script with five layers stacked on each other.

### Layer 1 — The Reach Table

The calibration data (four measured T positions) is stored as a nested dictionary. For unmeasured T_y values, the engine uses linear interpolation between the nearest measured points. This produces a complete reach table for all six T_y positions.

Each entry specifies four numbers per finger: how far it reaches behind T for low strings (1-3), behind T for high strings (4-6), ahead of T for low strings, and ahead of T for high strings. These four numbers define an asymmetric, non-uniform reach envelope that changes shape with the thumb's vertical position.

### Layer 2 — The Expression Generator

For a given T position, the generator:

1. Computes each finger's reachable cells from the reach table
2. Adds `None` (lifted) as an option for each finger
3. Iterates through all combinations of four fingers
4. Filters out any combination where two fingers occupy the same cell

This is a four-deep nested loop with collision checks. It's brute force, but it's fast — the worst case (T(4,5) with 15.4 million expressions) runs in about two seconds.

### Layer 3 — The BSA Engine

The 126 BSA patterns are pre-computed: two directions times 63 non-null 6-bit masks. Each pattern is a tuple of a direction character and a boolean tuple.

### Layer 4 — The Bang Point Calculator

Takes one expression and one BSA pattern. Converts the expression to a fretboard state (which frets are pressed on which strings), then iterates through the struck strings to compute MIDI note numbers. Pure arithmetic, no lookup tables.

### Layer 5 — The MIDI Exporter

Uses the `midiutil` library to write standard MIDI files. Each bang point becomes a set of simultaneous note-on events at a given time position. The output can be dragged directly into any DAW.

---

## What's Not Here Yet

ARCS v0.2 models the guitar hand as a set of independent single-cell fingertips. Several real-world techniques are not yet captured:

**Barre chords.** In reality, the index finger can press across multiple strings simultaneously. This effectively turns F1 from a point into a line — a horizontal bar across the front plane. The barre extension will let F1 occupy a range of cells on the same fret, dramatically expanding the expression library.

**Thumb rotation.** The calibration data revealed that the thumb physically rotates as T_y increases — nearly 180 degrees from T(1) to T(6). This rotation is coupled to the reach envelope but isn't modeled as a separate variable yet. Capturing it would allow the system to predict when a hand position requires uncomfortable wrist angles.

**T transitions.** The current model treats each T position independently, like snapshots. In reality, moving from T(1,3) to T(4,8) involves a physical traversal — a slide, a jump, or a pivot. The transition model will define legal movements between T positions, including diagonal slides (the thumb traces a path along the neck) and jumps (the hand lifts off and repositions).

**Fret width scaling.** Frets get narrower as you go up the neck. At fret 1, the distance between frets is roughly twice what it is at fret 12. The reach data was measured at fret 5 and applied uniformly. A fret-width correction factor would tighten the envelope at low frets and loosen it at high frets.

**The hand profile system.** The current implementation hardcodes one person's reach data. The planned system defines a small set of hand parameters — span, thumb flexibility, finger lengths, experience modifier — that scale the universal reach topology to any individual. A beginner with small hands gets a smaller library. A seasoned player with long fingers gets a larger one. A robot hand with uniform fingers and full rotation gets the largest of all.

---

## Why This Matters

ARCS isn't a music theory system. It doesn't know what sounds good. It knows what's physically possible, and it can tell you — with a number — how much of that possibility space you've explored.

If you've been playing guitar for twenty years and you've learned 500 chords, you've covered approximately 0.00000066% of the legal expression space. Even accounting for the fact that most of those 757 million positions sound terrible, the unexplored territory is staggering.

For robotics, ARCS provides something that doesn't exist in the literature: a concrete, finite, measurable benchmark for hand dexterity on a standardized task. If a robot hand can execute every legal ARCS expression, it has matched human capability on that instrument. If it can execute expressions outside the human reach envelope, it has exceeded it. The comparison is quantitative, not subjective.

For music, the expression library is a raw material source. Layer it with harmonic analysis, voice-leading rules, genre constraints, and rhythmic patterns, and you get a generative system that produces music from first principles — not from learned patterns, but from the physics of what the instrument allows.

The robot hand doesn't need to exist yet. The library already does.

---

## Quick Reference

### Notation Cheat Sheet

| Symbol | Meaning |
|--------|---------|
| `T(y,x)` | Thumb position: string axis y (1-6), fret axis x (1-23) |
| `F1(s,f)` | Index finger on string s, fret f |
| `F2(s,f)` | Middle finger |
| `F3(s,f)` | Ring finger |
| `F4(s,f)` | Pinky |
| `F#(-)` | Finger lifted (not touching any string) |
| `D[...]` | Downstroke BSA pattern |
| `U[...]` | Upstroke BSA pattern |
| `→` | "produces" (bang point result) |

### CLI Commands

```
python arcs_generator.py stats          # See the numbers
python arcs_generator.py demo           # Verify known chords
python arcs_generator.py random 24      # 24 random bang points → MIDI
python arcs_generator.py lookup 4 5     # All expressions at T(4,5)
python arcs_generator.py midi out.mid 64 # Export 64 events to MIDI
python arcs_generator.py unique         # Unique chord analysis
```

### Key Numbers (Reference Hand)

| Metric | Value |
|--------|-------|
| Fretboard cells | 138 |
| T positions (back plane) | 138 |
| BSA patterns | 126 |
| Max expressions at one T | 15,398,203 |
| Min expressions at one T | 548,608 |
| Est. total expressions | 757,643,851 |
| Est. total bang points | 95,463,125,226 |

---

*ARCS is an open framework. The expression library, reach model, and generation code are tools for anyone who wants to think about guitar — or robotic manipulation — as a computable spatial problem.*

*Built on Cape Cod, 2026.*
