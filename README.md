# ARCS — Anchor-Relative Coordinate System for Guitar

**A formal coordinate framework that quantifies every physically possible hand position on a guitar fretboard.**

**The number: 95,463,125,226 unique bang points.** That's every legal combination of left-hand finger placement and right-hand string strike on a standard 6-string, 23-fret guitar. From one pair of hands.

Guitar has a surprisingly large local state space: millions of physically legal hand configurations from a single thumb position.

---

## What Is This?

ARCS treats the guitar not as a musical instrument but as a spatial problem. The fretboard is a 6×23 lattice. Your thumb is an anchor point on the back plane of the neck — a 2D coordinate `T(y, x)` that determines what your fingers can reach. Each finger is a displacement vector from that anchor, constrained by biomechanics.

The system:
- Models the thumb as a **2D origin** on the back of the neck (not a fixed support — a moving variable)
- Defines each finger's **reach envelope** as a function of thumb position, empirically measured
- Generates every **legal expression** (hand configuration) by filtering for biomechanical constraints
- Pairs left-hand expressions with a **Binary Strike Actuator (BSA)** model of the right hand (126 patterns)
- Maps each combination to a **bang point** — a MIDI note or chord output

The result is a complete, enumerable library of everything the human hand can physically do on a guitar.

## Key Findings

- **757 million** legal left-hand expressions across all thumb positions
- **15.4 million** expressions at the most flexible thumb position (T(4,5) — behind the G string)
- **548,608** at the most constrained (T(6,20) — high e string, 20th fret)
- The ring finger (F3) is **conditionally weak** — worst finger at low thumb positions, best at high positions
- Backward reach follows a **U-curve**: minimum at center neck, maximum at the edges
- Thumb rotation increases with T_y position, approaching **180°** at extreme positions

I have not found another published framework that models guitar this way: as a thumb-anchored, enumerable hand-state space.

## Quick Start

```bash
# Clone
git clone https://github.com/plx0113/ARCS.git
cd ARCS

# Install dependency
pip install midiutil

# See the numbers
python arcs_generator.py stats

# Verify known chords through the engine
python arcs_generator.py demo

# Generate 24 random legal bang points → MIDI file
python arcs_generator.py random 24

# Inspect all expressions at a specific thumb position
python arcs_generator.py lookup 4 5

# Export a sequence to MIDI (drag into your DAW)
python arcs_generator.py midi my_sequence.mid 64

# Analyze unique chord outputs
python arcs_generator.py unique
```

## ARCS Notation

```
T(1,1)|F1(4,1)|F2(2,2)|F3(3,2)|F4(-) × D[EADGBe] → E2 B2 E3 G#3 B3 E4
```

| Symbol | Meaning |
|--------|---------|
| `T(y,x)` | Thumb position — string axis y (1-6), fret axis x (1-23) |
| `F1(s,f)` | Index finger on string s, fret f |
| `F2(s,f)` | Middle finger |
| `F3(s,f)` | Ring finger |
| `F4(s,f)` | Pinky |
| `F#(-)` | Finger lifted |
| `D[EADGBe]` | Downstroke, all strings struck |
| `U[--D---]` | Upstroke, string 3 only |
| `→` | Produces (bang point output) |

## Calibration

The reach data was measured empirically — a real hand on a Schecter C-6 Deluxe, testing each finger's maximum comfortable reach at four thumb positions. The system interpolates between measured points to estimate reach at all positions.

**You can plug in your own hand.** Measure your reach at T(1,5), T(3,5), T(4,5), and T(6,5), update the `CALIBRATION` dict in the code, and generate your personal expression library. Different hands → different libraries → different numbers.

## Roadmap

- [x] Core ARCS formalism and notation
- [x] Empirical reach calibration (reference hand)
- [x] Expression generator with collision filtering
- [x] BSA (right hand) model — 126 patterns
- [x] Bang point calculator → MIDI mapping
- [x] MIDI file export
- [ ] Barre chord extension (F1 as a line, not a point)
- [ ] Thumb rotation modeling (T_rotation as a continuous variable)
- [ ] T transition model (slide vs jump, legal movements between T positions)
- [ ] Fret-width scaling (narrower frets = tighter envelopes at high positions)
- [ ] Hand profile system (parameterized reach scaling for different players)
- [ ] Continuous sub-fret T positioning
- [ ] Force/pressure modeling for string depression
- [ ] MuJoCo/PyBullet robot hand simulation
- [ ] Musical intelligence layer (harmonic filtering of the expression library)

## Documentation

See [ARCS_Grimoire_v02.md](ARCS_Grimoire_v02.md) for the full technical explanation of the framework, the measurement methodology, and the math.

## Applications

**Robotics** — ARCS provides a concrete, finite benchmark for robotic hand dexterity. A robot hand's capability can be measured as the percentage of the human expression library it can execute.

**Music education** — The reach envelope data reveals that the ring finger's weakness is a function of thumb position, not inherent anatomy. This has direct implications for how guitar technique is taught.

**Generative music** — The expression library is a complete source of physically playable note combinations, filterable by harmonic rules, voice leading, and genre constraints.

**Instrument design** — Comparing expression libraries across different scale lengths, string counts, and tunings quantifies how design changes affect playability.

## Origin

ARCS was conceived ~2015 while walking on train tracks near UMass, when the importance of thumb position in guitar performance became suddenly, obviously clear. The formalization and computation happened in 2026.

## Author

**plx0** — Jazz guitar (UMass Amherst), music technology (UMass Dartmouth), decade in QA at inMusic Brands, independent AV/software developer. Cape Cod, MA.

## License

MIT
