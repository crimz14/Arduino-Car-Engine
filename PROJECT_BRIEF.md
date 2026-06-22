# Project Brief — Arduino Engine Sound Simulator

**Project Title:** Arduino Car Engine Sound Simulator  
**Platform:** Arduino Uno R3 (ATmega328P)  
**Version:** 1.0.0  
**Target Audience:** Beginner–Intermediate Arduino / A-Level CS students  

---

## 1. Problem Statement

Consumer-grade microcontrollers like the Arduino Uno lack the processing power and memory to play back real audio files. However, they can generate and modulate square waves in real time using the built-in `tone()` function. The challenge is to make that square wave feel like a car engine — using mathematics, randomness, and careful timing rather than recordings.

This project solves that problem by modelling RPM, gear physics, and engine character procedurally, then mapping those values to buzzer frequency in real time.

---

## 2. System Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                        Arduino Uno R3                            │
│                                                                  │
│  ┌─────────────┐    ADC (A0)     ┌──────────────────────────┐   │
│  │ Potentiometer│ ──────────────▶│     readThrottle()       │   │
│  └─────────────┘                 └───────────┬──────────────┘   │
│                                              │                   │
│  ┌─────────────┐    D2/D3/D4     ┌───────────▼──────────────┐   │
│  │   Buttons   │ ──────────────▶│     updateButtons()       │   │
│  └─────────────┘                 └───────────┬──────────────┘   │
│                                              │                   │
│                                  ┌───────────▼──────────────┐   │
│                                  │      updateRPM()          │   │
│                                  │  (gear ratio, easing,     │   │
│                                  │   rev limiter, shift dip) │   │
│                                  └───────────┬──────────────┘   │
│                                              │                   │
│                          ┌───────────────────┼──────────────┐   │
│                          │                   │              │   │
│               ┌──────────▼───────┐  ┌────────▼──────────┐  │   │
│               │ updateDisplay()  │  │updateEngineSound() │  │   │
│               │ (I2C LCD, 150ms) │  │ (tone(), 20ms)     │  │   │
│               └──────────────────┘  └───────────────────-┘  │   │
│                                                              │   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 3. Key Design Decisions

### 3.1 `millis()` Instead of `delay()`

Using `delay()` freezes the entire microcontroller. This project uses `millis()`-based interval checks throughout so that button reading, RPM updating, sound generation, and display refreshing all happen independently and concurrently — a fundamental embedded systems pattern.

### 3.2 Struct-Based Engine Profiles

Rather than writing separate logic for each engine, all engine-specific data is stored in an `EngineProfile` struct. Adding a new engine type is a single line in the `profiles[]` array — no code changes needed. This is an example of **data-driven design**.

### 3.3 Exponential RPM Easing

```
currentRPM += (targetRPM - currentRPM) × responseSpeed
```

This single line produces a smooth, non-linear approach to the target RPM. At high gaps it moves fast; as the gap narrows it slows down. It naturally models engine inertia, and `responseSpeed` gives each profile a different feel without any complex physics simulation.

### 3.4 Roughness as Perlin-like Noise

Real engines don't run at a perfectly constant pitch. The randomised `wobble` variable, which is larger at low RPM and smaller at high RPM, imitates the uneven combustion events at idle and the smoother power delivery at high revs.

---

## 4. Functional Requirements

| ID | Requirement |
|---|---|
| FR-01 | System shall read throttle position from a potentiometer on A0 |
| FR-02 | System shall output RPM-proportional sound via passive buzzer on D9 |
| FR-03 | System shall support 7 gear states (N, 1–6) via two buttons |
| FR-04 | System shall prevent gear shifting beyond limits (N or 6) |
| FR-05 | System shall apply a brief RPM dip and sound cut on gear change |
| FR-06 | System shall cycle through at least 4 engine profiles via a button |
| FR-07 | System shall display RPM, gear, throttle, and engine name on LCD |
| FR-08 | System shall debounce all buttons with at minimum 200ms lockout |
| FR-09 | System shall apply a rev limiter effect near each profile's maxRPM |
| FR-10 | All timing shall be non-blocking (no `delay()` in main loop) |

---

## 5. Non-Functional Requirements

| ID | Requirement |
|---|---|
| NFR-01 | Loop execution time shall not visibly lag button or throttle response |
| NFR-02 | Code shall be commented clearly for an A-Level CS student audience |
| NFR-03 | Total BOM cost shall remain under £15 / $20 using clone hardware |
| NFR-04 | Project shall be buildable on a standard 400-point breadboard |
| NFR-05 | No external libraries beyond LiquidCrystal_I2C shall be required |

---

## 6. Hardware Specification

### Microcontroller
- **Board:** Arduino Uno R3
- **MCU:** ATmega328P @ 16 MHz
- **Flash:** 32 KB (sketch uses approx. 3–5 KB)
- **SRAM:** 2 KB (runtime state uses approx. 300–400 bytes)
- **ADC Resolution:** 10-bit (0–1023)
- **PWM Pins:** D3, D5, D6, D9, D10, D11 (D9 used for tone)

### Power Budget (approx.)
| Component | Current Draw |
|---|---|
| Arduino Uno R3 | ~45 mA |
| 16×2 LCD with backlight | ~20–30 mA |
| Passive piezo buzzer | ~5–30 mA (peak) |
| 3× buttons (pull-up) | negligible |
| Potentiometer | ~0.5 mA |
| **Total** | **~75–110 mA** |

Safe for USB power (max 500 mA). No external power supply needed.

---

## 7. Software Module Reference

| Function | Purpose | Called From | Interval |
|---|---|---|---|
| `readThrottle()` | ADC read → throttlePercent (0–100) | loop() | Every loop |
| `updateButtons()` | Poll 3 buttons, debounce, call handlers | loop() | Every loop |
| `shiftUp()` | Increment gear, trigger shift effect | updateButtons() | On press |
| `shiftDown()` | Decrement gear, trigger shift effect | updateButtons() | On press |
| `triggerShiftEffect()` | Set flag + timestamp for shift dip | shiftUp/Down | On shift |
| `cycleEngineProfile()` | Advance currentProfile, reset RPM | updateButtons() | On press |
| `updateRPM()` | Calculate target + smooth currentRPM | loop() | Every loop |
| `updateEngineSound()` | Map RPM → freq, apply wobble, call tone() | loop() | Every 20ms |
| `updateDisplay()` | Write RPM/gear/throttle/profile to LCD | loop() | Every 150ms |

---

## 8. Variable Reference

| Variable | Type | Description |
|---|---|---|
| `throttlePercent` | `int` | Throttle position (0–100%) |
| `currentRPM` | `float` | Smoothed real-time RPM |
| `targetRPM` | `float` | RPM the engine is moving toward |
| `currentGear` | `int` | Active gear (0 = Neutral) |
| `currentProfile` | `int` | Index into `profiles[]` array |
| `shiftingEffect` | `bool` | True during gear-change dip window |
| `shiftEffectStart` | `unsigned long` | `millis()` timestamp of last shift |
| `lastDebounce*` | `unsigned long` | Per-button debounce timestamps |
| `lastSoundUpdate` | `unsigned long` | Last `tone()` update timestamp |
| `lastDisplayUpdate` | `unsigned long` | Last LCD update timestamp |

---

## 9. Known Limitations (v1.0)

- `tone()` produces a square wave only — no waveform shaping. The buzzer will never sound exactly like a real engine; it is an approximation.
- The Arduino Uno has one `tone()` channel. Multi-cylinder firing order timing (which would require separate frequency channels) is not possible without additional hardware.
- LCD refresh at 150ms means rapidly-changing RPM values may briefly show intermediate numbers.
- No EEPROM persistence — settings reset to defaults on power-off.

---

## 10. Curriculum Links (OCR A-Level Computer Science)

| Topic | Demonstrated By |
|---|---|
| Data types and structures | `struct EngineProfile`, arrays of structs |
| Subroutines / modularity | Separate named functions per concern |
| Boolean logic | `INPUT_PULLUP` polarity, `shiftingEffect` flag |
| Iteration | `loop()` main cycle, interval comparisons |
| Hardware / physical computing | ADC, PWM, I2C peripherals |
| Systems architecture | MCU, memory constraints, I/O buses |
| Non-blocking concurrency | `millis()` pattern as scheduling |
| Abstraction | Profiles abstracting engine differences |

---

## 11. Version History

| Version | Date | Notes |
|---|---|---|
| 1.0.0 | 2026 | Initial release — all core features complete |

---

## 12. Possible Future Versions

### v1.1 — Code enhancements (no new hardware)
- Exhaust pop/crackle on sudden throttle lift
- Turbo whine tone layered above main engine tone
- Ignition sequence (startup RPM ramp from 0)
- EEPROM save of last selected profile

### v1.2 — Minor hardware additions
- 6-LED bar graph rev counter
- Ignition/start button
- Clutch button (decouples RPM from gear load temporarily)
- Gear indicator LED (shift-up prompt at high RPM)

### v2.0 — DFPlayer Mini + real samples
- Replace piezo with 8Ω speaker + PAM8403 amp
- DFPlayer Mini module reads WAV files from SD card
- Separate samples per engine profile
- Estimated additional cost: £6 – £10 / $8 – $13

### v3.0 — ESP32 port
- DAC audio output for proper waveform generation
- Bluetooth for mobile phone throttle/display app
- Dual-core: one core for audio DSP, one for I/O
- OLED graphical tachometer with animated needle
- Estimated platform cost: £4 – £8 / $5 – $10 (replaces Uno)
