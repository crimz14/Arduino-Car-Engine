# 🚗 Arduino Engine Sound Simulator

> Procedural car engine sound simulator for Arduino Uno R3. Potentiometer throttle, gear shift buttons, and 16x2 I2C LCD. Four engine profiles (Inline-4, V6, V8, Sports), 7-speed gearbox, RPM-based sound via passive piezo with idle wobble, rev limiter, and gear-change cuts.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Parts List & Cost](#parts-list--cost)
- [Pin Mapping](#pin-mapping)
- [Wiring Guide](#wiring-guide)
- [Dependencies](#dependencies)
- [Installation](#installation)
- [Usage](#usage)
- [Engine Profiles](#engine-profiles)
- [Gear System](#gear-system)
- [How the Sound Works](#how-the-sound-works)
- [Project Brief](#project-brief)
- [Testing Checklist](#testing-checklist)
- [Troubleshooting](#troubleshooting)
- [Possible Upgrades](#possible-upgrades)
- [License](#license)

---

## Overview

This project simulates the sound and feel of a car engine on an Arduino Uno R3 — no audio samples, no SD card. All sound is generated mathematically using `tone()`, shaped by RPM, throttle position, engine profile, and gear. Designed as a breadboard-first beginner/intermediate project with A-Level Computer Science–friendly code.

---

## Features

- 🎛️ **Potentiometer throttle** — analogue 0–100% input
- ⚙️ **7-speed gearbox** — Neutral + 6 drive gears with realistic RPM behaviour
- 🔊 **Procedural engine sound** — frequency scales with RPM, roughness at idle, smoothness at high revs
- 🏎️ **4 engine profiles** — Inline-4, V6, V8, High-revving Sports
- 📟 **16×2 I2C LCD** — live display of RPM, gear, throttle %, and engine name
- 🔴 **Rev limiter effect** — pitch flutter near redline
- 🔇 **Gear change sound cut** — brief RPM dip and silence on each shift
- ⏱️ **Non-blocking code** — all timing via `millis()`, no `delay()` in the main loop

---

## Parts List & Cost

| Part | Qty | Est. Unit Price (GBP) | Est. Unit Price (USD) | Notes |
|---|---|---|---|---|
| Arduino Uno R3 | 1 | £4 – £22 | $5 – $28 | Clone or official |
| 10kΩ Potentiometer | 1 | £0.50 | $0.60 | Any linear taper |
| Push buttons (momentary) | 3 | £0.10 each | $0.12 each | Standard 6mm tact |
| 16×2 I2C LCD (PCF8574) | 1 | £2 – £4 | $2.50 – $5 | Pre-soldered I2C backpack preferred |
| Passive piezo buzzer | 1 | £0.30 – £0.80 | $0.40 – $1 | Must be **passive** (not active) |
| Breadboard (400pt+) | 1 | £1.50 – £3 | $2 – $4 | |
| Jumper wires (M-M) | ~15 | £0.05 each | $0.06 each | Usually sold in packs |
| USB-A to USB-B cable | 1 | £1 – £3 | $1.50 – $4 | For programming |

### Estimated Total

| Scenario | GBP | USD |
|---|---|---|
| Buying individual parts (clone Uno) | ~£10 – £15 | ~$13 – $20 |
| Buying from a starter kit | ~£5 – £8 extra | ~$6 – $10 extra |
| Official Arduino Uno + new parts | ~£30 – £40 | ~$38 – $50 |

> 💡 Most of these parts are included in common Arduino starter kits (e.g. Elegoo or LAFVIN). If you already own a kit, your additional cost may be £0.

---

## Pin Mapping

| Function | Arduino Pin |
|---|---|
| Potentiometer wiper (signal) | A0 |
| Shift up button | D2 |
| Shift down button | D3 |
| Engine profile button | D4 |
| Passive piezo buzzer (+) | D9 |
| LCD SDA | A4 |
| LCD SCL | A5 |

---

## Wiring Guide

### Potentiometer
```
Pin 1 (left)   → 5V
Pin 2 (middle) → A0
Pin 3 (right)  → GND
```

### Buttons (all three)
```
One leg → Arduino pin (D2 / D3 / D4)
Other leg → GND
```
No resistors needed — `INPUT_PULLUP` is used in code.

### Passive Piezo Buzzer
```
+ (longer leg) → D9
- (shorter leg) → GND
```

### 16×2 I2C LCD
```
VCC → 5V
GND → GND
SDA → A4
SCL → A5
```

---

## Dependencies

Install via **Arduino IDE → Sketch → Include Library → Manage Libraries**:

| Library | Author | Version |
|---|---|---|
| `LiquidCrystal_I2C` | Frank de Brabander | 1.1.2+ |
| `Wire` | Arduino (built-in) | — |

---

## Installation

1. Clone or download this repository.
2. Open `engine_simulator.ino` in the Arduino IDE.
3. Install `LiquidCrystal_I2C` via the Library Manager.
4. Select **Tools → Board → Arduino Uno**.
5. Select the correct **COM port** under Tools → Port.
6. Click **Upload** (→).

---

## Usage

| Control | Action |
|---|---|
| Turn pot clockwise | Increase throttle / RPM |
| Turn pot anticlockwise | Decrease throttle / RPM |
| Press Shift Up (D2) | Increase gear (N → 1 → ... → 6) |
| Press Shift Down (D3) | Decrease gear (6 → ... → 1 → N) |
| Press Profile (D4) | Cycle engine type |

LCD shows:
```
RPM:3200    G:3
Thr:65%   V6
```

---

## Engine Profiles

| Profile | Idle RPM | Max RPM | Character |
|---|---|---|---|
| Inline-4 | 800 | 6,500 | Balanced, mid roughness |
| V6 | 750 | 6,800 | Slightly smoother, more wobble |
| V8 | 700 | 6,000 | Deep, slow response, heavy roughness |
| SportsHi | 1,200 | 9,000 | Fast response, high-pitched, smooth |

---

## Gear System

| Gear | Gear Ratio | Effect |
|---|---|---|
| N (Neutral) | 1.00 | Free-revving, throttle maps directly to full RPM range |
| 1 | 1.00 | Full RPM range available — aggressive acceleration |
| 2 | 0.85 | Slightly reduced RPM ceiling |
| 3 | 0.70 | Noticeably slower RPM rise |
| 4 | 0.58 | Cruising gear — RPM stays moderate |
| 5 | 0.48 | High-speed gear, low RPM climb |
| 6 | 0.40 | Highest gear, engine loafs at most throttle inputs |

A shift event briefly drops RPM to ~60% and cuts buzzer sound for ~180ms, imitating clutch engagement.

---

## How the Sound Works

The `tone()` function outputs a square wave on D9. The frequency is calculated like this:

```
baseFreq = profile.baseTone + (rpmFraction × profile.toneRange)
wobble   = random(±roughness) × (1 - rpmFraction × 0.8)
finalFreq = baseFreq + wobble
```

- **Low RPM** → low frequency + large wobble = rough, lumpy idle
- **High RPM** → higher frequency + small wobble = smooth, consistent rev
- **Near redline** → random ±100 Hz flutter = rev limiter bounce
- **Gear shift** → `noTone()` for 180ms = ignition cut / clutch sound

---

## Project Brief

### Background

This project was created as an embedded systems learning exercise, targeting beginner-to-intermediate Arduino users. It demonstrates key concepts from digital and analogue I/O, software debouncing, non-blocking timing patterns, struct-based data modelling, and PWM sound generation — all relevant to A-Level Computer Science coursework.

### Objectives

- Demonstrate analogue input processing (ADC reading, `map()`)
- Implement `millis()`-based non-blocking multitasking
- Model real-world entities (engines, gears) using C++ structs
- Drive a peripheral display via I2C
- Generate and modulate procedural sound without external audio hardware

### Scope

| In Scope | Out of Scope (v1) |
|---|---|
| Procedural sound via `tone()` | Real audio sample playback |
| 4 engine profiles | Engine damage / temperature simulation |
| 6 gears + neutral | Clutch physics |
| 16×2 LCD display | Graphical / animated display |
| Breadboard build | PCB / enclosure |

### Constraints

- **Platform**: Arduino Uno R3 (ATmega328P, 16 MHz, 2 KB SRAM)
- **Audio**: `tone()` only — single-channel square wave
- **Power**: USB or 9V barrel jack
- **Budget**: Under £15 / $20 using clone hardware

---

## Testing Checklist

- [ ] LCD shows splash screen on boot, then live data
- [ ] Pot at 0% → throttle shows 0, RPM stays at idle
- [ ] Pot at 100% → throttle shows 100, RPM rises toward max
- [ ] Sound pitch rises with RPM
- [ ] Shift Up increases gear, stops at 6
- [ ] Shift Down decreases gear, stops at N
- [ ] Brief sound cut + RPM dip on every gear change
- [ ] Profile button cycles all 4 engines, LCD updates name
- [ ] V8 feels slower to respond than SportsHi
- [ ] Near-max RPM produces audible flutter
- [ ] Gear 6 + full throttle has lower RPM than Gear 1 + full throttle

---

## Troubleshooting

| Problem | Likely Cause | Fix |
|---|---|---|
| LCD blank on boot | Wrong I2C address | Change `0x27` to `0x3F` in code |
| LCD garbled / random chars | Bad SDA/SCL connection | Check A4 (SDA) and A5 (SCL) wiring |
| No sound from buzzer | Active buzzer used, or wiring issue | Confirm **passive** piezo; check D9 and GND |
| Buttons don't respond | Wired to 5V instead of GND | Buttons must connect between pin and **GND** |
| Buttons double-trigger | Debounce too short | Increase `DEBOUNCE_DELAY` from 200 to 300ms |
| RPM doesn't reach max | Gear ratio limiting it | Test in Neutral (gear 0) for free-rev |
| RPM stuck at idle | Pot wired incorrectly | Check all 3 pot legs: 5V, GND, signal to A0 |
| Upload fails | Wrong board or port selected | Tools → Board: Uno; Tools → Port: correct COM |

---

## Possible Upgrades

### Tier 1 — Easy (drop-in additions)

| Upgrade | Benefit | Approx Cost |
|---|---|---|
| 10Ω resistor + small speaker (8Ω) | Louder, richer tone than piezo | £0.50 / $0.60 |
| Ignition / start button | Start sequence sound + RPM ramp from 0 | £0.10 / $0.12 |
| Clutch button | Disconnects gear load from RPM temporarily | £0.10 / $0.12 |
| LED rev counter (6 LEDs) | Visual tachometer strip | £0.50 / $0.60 |

### Tier 2 — Intermediate (new modules)

| Upgrade | Benefit | Approx Cost |
|---|---|---|
| DFPlayer Mini + SD card + speaker | Real engine audio samples — dramatic improvement | £3 – £6 / $4 – $8 |
| SSD1306 OLED 128×64 | Graphical tachometer needle, larger text | £2 – £4 / $2.50 – $5 |
| Potentiometer for rev limiter tuning | Adjustable redline per engine | £0.50 / $0.60 |
| Exhaust pop/crackle | Random short noise burst on throttle lift-off | Free (code only) |
| Turbo spool simulation | Rising whine tone above certain RPM | Free (code only) |

### Tier 3 — Advanced (platform change)

| Upgrade | Benefit | Approx Cost |
|---|---|---|
| ESP32 (replaces Uno) | DAC audio, Bluetooth, dual-core processing | £4 – £8 / $5 – $10 |
| PAM8403 amplifier module | Clean amplified speaker output | £1 / $1.20 |
| Custom PCB (EasyEDA / KiCad) | Permanent, compact build | £5 – £15 / $6 – $20 (PCB fab) |
| 3D printed enclosure | Dashboard / centre console aesthetic | Variable (filament cost) |
| Engine temperature simulation | Warm-up RPM drop, overheat cut-out | Free (code only) |

---

## File Structure

```
engine-simulator/
├── engine_simulator.ino     # Main Arduino sketch
├── README.md                # This file
└── PROJECT_BRIEF.md         # Extended project documentation
```

---

## License

MIT License — free to use, modify, and distribute. Attribution appreciated.
