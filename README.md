# Daphne - Oramics Fluid Synth 🎨🎵

An interactive audio-visual synthesizer that combines WebGL fluid simulation with music theory-based sound generation, inspired by the pioneering electronic music work of **Daphne Oram** and her revolutionary Oramics technique.

## Overview

**Daphne** is a **real-time audio-visual instrument** where:
- **Visual**: WebGL-based fluid dynamics simulation responds to your gestures
- **Audio**: Web Audio API generates musical sounds quantized to proper scales
- **Interaction**: Paint with colors that each have unique sound and musical personality
- **Oramics**: Visual patterns directly control sound synthesis, inspired by Daphne Oram's technique of drawing sound onto film

## Who Was Daphne Oram?

Daphne Oram (1925-2003) was a British composer and electronic music pioneer who invented **Oramics** - a groundbreaking technique where visual patterns drawn on film strips would be read by photoelectric cells to control sound synthesis. She co-founded the BBC Radiophonic Workshop and created one of the first electronic music studios in the UK. This project honors her legacy by translating her vision of "drawn sound" into an interactive digital medium.

## Features

### 🎨 Color Voices (Musical Scales)

Each color uses a different musical scale, giving it a unique melodic character:

| Color | Scale | Character | Sound Texture |
|-------|-------|-----------|---------------|
| **Grey** | Pentatonic | Safe, balanced | Clean sine wave |
| **Brown** | Blues | Earthy, soulful | Crackles, distortion, sub-bass |
| **Blue** | Minor | Sad, emotional | Modulated sine waves |
| **Yellow** | Major | Bright, happy | Bright sawtooth, high frequencies |
| **Green** | Pentatonic | Natural, organic | Bamboo wind chimes |
| **Pink** | Harmonic Minor | Exotic, crystalline | Ice crystals, glass shatter |
| **Red** | Chromatic | Chaotic, industrial | Metal grinding, heavy distortion |

### 🎼 Musical Quantization System

All frequencies are **quantized to proper musical notes** using:
- **A440 Standard**: Universal tuning where A = 440 Hz
- **Equal Temperament**: 12-tone system, each semitone = 2^(1/12)
- **MIDI Note Mapping**: Cursor Y position maps to MIDI notes (48-84 range)
- **Scale Quantization**: Notes snap to the selected color's scale

**How it works:**
1. Cursor Y position → Raw frequency
2. Raw frequency → Nearest MIDI note
3. MIDI note → Quantized to scale
4. Result: Always musical, never out of tune!

### 🥁 Rhythmic Beat System

Four beat patterns with proper musical timing:

| Beat | Icon | Division | Description |
|------|------|----------|-------------|
| **Kick** | ● | Quarter notes | Every beat (1, 2, 3, 4) - Root note |
| **Hi-hat** | × | Eighth notes | Double time (1-and-2-and) - High frequency noise |
| **Snare** | ▬ | Half notes | Backbeat (beats 2 & 4) - Perfect fifth |
| **Pulse** | ◉ | Whole notes | Once per bar - Root note with reverb |

All beats sync to **120 BPM master tempo** and use **musical intervals** (root, fifth) that harmonize with the colors.

### ✨ Glitch Effects

Five visual and audio manipulation effects:

| Glitch | Icon | Effect |
|--------|------|--------|
| **Bit Crush** | ◼ | Extreme pixelation, bit-crushed audio, color quantization |
| **Corrupt** | ◆ | Color inversion, pitch jumps, velocity spikes |
| **Feedback** | ↻ | Visual trails, audio delay/feedback |
| **RGB Split** | ▣ | Prism effect - colors split into rainbow layers |
| **Freeze** | ❄ | Darkens fluid, freezes sound, icy blue tint |

All glitches are **cursor-responsive** and can be layered for complex effects.

### 🎵 Oramics Effects (Daphne Oram-Inspired)

Four special effects inspired by Daphne Oram's pioneering Oramics technique, where visual gestures directly control sound synthesis:

| Effect | Icon | Description |
|--------|------|-------------|
| **Painted Envelope** | ⟲ | Your gesture shape controls sound evolution - fast strokes create quick attacks, slow sustained gestures create long, evolving releases |
| **Drawn Waveforms** | ∿ | Visual patterns on the canvas become the sound wave itself - the brightness and complexity of your painting directly shapes the timbre using Fourier synthesis |
| **Pattern Sequencer** | ⊞ | Paint zones on the canvas that trigger notes when fluid flows through them - like a visual step sequencer where space becomes time |
| **Gesture Drones** | 〰 | Sustained painting (hold mouse ~1 second) creates evolving ambient drone layers that slowly drift in pitch and filter - layer up to 6 simultaneous drones |

**How Oramics Effects Work:**
- **Painted Envelope**: Tracks your gesture velocity, path length, and duration to create dynamic envelopes (attack/release times)
- **Drawn Waveforms**: Samples pixel brightness across the canvas and converts it into Fourier coefficients for PeriodicWave synthesis
- **Pattern Sequencer**: Stores painted zones and monitors fluid velocity at those positions, triggering notes when flow is detected
- **Gesture Drones**: Creates sustained oscillators with analog-style frequency drift and evolving lowpass filters (10-20 second lifetimes)

## Technical Architecture

### WebGL Fluid Simulation

**Shaders:**
- `advectionProgram`: Moves fluid based on velocity field
- `diffusionProgram`: Spreads colors through the fluid
- `divergenceProgram`: Calculates velocity field divergence
- `pressureProgram`: Solves pressure to maintain incompressibility
- `gradientSubtractProgram`: Makes fluid incompressible
- `splatProgram`: Injects color and velocity at cursor position
- `displayProgram`: Renders the final fluid to screen

**Framebuffers:**
- `velocity`: Stores fluid velocity vectors (double-buffered)
- `colorBuffer`: Stores color information (double-buffered)
- `pressure`: Pressure field for incompressibility
- `divergence`: Velocity divergence field
- Uses floating-point textures for smooth simulation

**Parameters:**
- `simWidth/simHeight`: 128×128 simulation resolution
- `persistence`: How long colors stay visible (per-color)
- `diffusion`: How much colors spread (per-color, modified by glitches)

### Web Audio API Architecture

**Signal Flow:**
```
Oscillator → Envelope → Filter → [Glitches] → Panner → Dry/Wet Mix → Master → Output
                                                            ↓
                                                         Reverb
                                                            ↓
                                                    [Oramics Processing]
```

**Audio Nodes:**
- **Oscillators**: Generate base frequencies (sine, triangle, sawtooth, custom PeriodicWave for Drawn Waveforms)
- **Envelopes**: Control volume over time (attack/release, dynamically modified by Painted Envelope)
- **Filters**: Shape frequency content (lowpass, highpass, bandpass, evolving for Gesture Drones)
- **Effects**: Reverb, delay, distortion, bit crushing
- **Panner**: Stereo positioning based on cursor X position

**Special Sound Generators:**
- Bamboo chimes (Green): Multiple harmonically-tuned oscillators
- Ice crystals (Pink): Rapid high-frequency crackles with frozen resonance
- Metal grinding (Red): Noise + modulated oscillators with heavy distortion
- Crackles (Brown): Randomized short bursts for organic texture
- Gesture Drones: Sub-octave oscillators with analog drift and evolving filters

### Oramics Implementation Details

**Painted Envelope:**
```javascript
// Gesture metrics tracked per frame:
- gestureStartTime: When gesture began
- gesturePathLength: Total distance traveled
- gestureAvgVelocity: Average speed of movement
- isGestureActive: Whether painting is happening

// Envelope parameters dynamically adjusted:
attackTime = 0.001 + (1.0 - velocityNorm) * 1.0  // 1ms to 1s
soundRelease = 0.1 + (1.0 - velocityNorm) * 4.9  // 0.1s to 5s
soundRelease *= pathLengthBoost  // Up to 3x
soundRelease *= durationBoost    // Up to 2.5x
```

**Drawn Waveforms:**
```javascript
// Visual sampling and synthesis:
1. Sample pixel brightness across horizontal line (256 samples)
2. Convert brightness (0-1) to amplitude (-1 to 1)
3. Use as Fourier coefficients for PeriodicWave
4. Apply to oscillator for custom timbre

// Creates real and imaginary Fourier components
real[i] = visualValue * 0.5
imag[i] = visualValue * 0.5
periodicWave = audioContext.createPeriodicWave(real, imag)
```

**Pattern Sequencer:**
```javascript
// Spatial-temporal triggering:
1. Mousedown stores zone: {x, y, color, lastTriggerTime}
2. Every frame, read velocity texture from GPU
3. Sample velocity magnitude at each zone position
4. If velocity > 0.05, trigger note
5. Cooldown of 500ms prevents note spam
```

**Gesture Drones:**
```javascript
// Ambient layer generation:
1. Mousedown starts timer
2. After 800ms sustained press, create drone
3. Drone properties:
   - Frequency: baseFreq / (2 + random)  // Sub-octave
   - Type: 'triangle' for smooth timbre
   - Filter: lowpass 400-1600 Hz
   - Gain: 0 → 0.08 over 3 seconds (fade in)
4. Evolution every frame:
   - Frequency drift: ±10 Hz sine wave
   - Filter sweep: 200-1400 Hz
5. Auto-fade after 10-20 seconds
```

### Music Theory Implementation

**Core Functions:**

```javascript
midiToFreq(midiNote)
// Converts MIDI note number to Hz: 440 * 2^((note - 69) / 12)

quantizeToScale(rawFreq, scaleName, rootNote)
// Snaps any frequency to nearest note in the specified scale

quantizePitch(mouseYPos)
// Maps cursor Y → MIDI note → quantized frequency using color's scale
```

**Scales Implemented:**
- Pentatonic: [0, 2, 4, 7, 9] - 5 notes
- Major: [0, 2, 4, 5, 7, 9, 11] - 7 notes (happy)
- Minor: [0, 2, 3, 5, 7, 8, 10] - 7 notes (sad)
- Blues: [0, 3, 5, 6, 7, 10] - 6 notes (soulful)
- Harmonic Minor: [0, 2, 3, 5, 7, 8, 11] - 7 notes (exotic)
- Chromatic: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11] - All 12 notes

## File Structure

```
Daphne/
├── index.html          # Single-file application (all code ~4100 lines)
├── README.md           # This file
└── .git/               # Git repository
```

The entire application is contained in a single `index.html` file with:
- HTML structure
- CSS styling (modern UI with animations)
- JavaScript:
  - WebGL setup and shaders
  - Color voice definitions with unique scales
  - Musical quantization system
  - Beat system with musical timing
  - Glitch effects system
  - Oramics effects system (Painted Envelope, Drawn Waveforms, Pattern Sequencer, Gesture Drones)
  - Audio generation and synthesis
  - Fluid simulation rendering loop

## How to Use

### Getting Started

1. Open `index.html` in a modern web browser (Chrome, Firefox, Safari)
2. Click **"Sound On"** to enable audio
3. Select a color from the palette
4. Move your cursor around the canvas to paint with sound

### Controls

**Color Selection:**
- Click any color swatch to select it
- Selected color has a white border
- Each color has a unique sound and scale

**Beat System:**
- Click beat buttons to toggle rhythms
- Active beats pulse and have white borders
- Multiple beats can be layered

**Glitch Effects:**
- Click glitch buttons to toggle effects
- Active glitches have white borders
- Effects respond to cursor movement
- Can be combined for complex results

**Oramics Effects:**
- **Painted Envelope (⟲)**: Enable, then paint - gesture speed/length controls sound evolution
- **Drawn Waveforms (∿)**: Enable, then paint patterns - visual complexity shapes timbre
- **Pattern Sequencer (⊞)**: Enable, click to place zones, then paint to trigger them with fluid flow
- **Gesture Drones (〰)**: Enable, hold mouse down ~1 second while painting to create ambient layers

### Performance Tips

- **Cursor Speed**: Faster movement = more energetic sounds and visuals
- **Cursor Position**: 
  - Y-axis controls pitch (top = high, bottom = low)
  - X-axis controls stereo panning (left/right)
- **Layering**: Switch colors while painting to layer different scales
- **Beats**: Start with kick, add hi-hat for groove
- **Glitches**: Try one at a time first, then combine
- **Oramics Workflow**:
  - Start with Pattern Sequencer to create rhythmic triggers
  - Add Gesture Drones for ambient bed
  - Use Painted Envelope for expressive control
  - Enable Drawn Waveforms for timbral variety

## Development

### Branch Structure

- `main`: Stable releases with all features
- `Harmony`: Music theory and quantization system (merged)
- `glitch-and-beat-system`: Development of glitch/beat features (merged)
- `ORAM1`: Oramics effects development (merged)

### Key Technologies

- **WebGL**: Hardware-accelerated fluid simulation
- **Web Audio API**: Real-time audio synthesis with PeriodicWave for custom waveforms
- **Vanilla JavaScript**: No frameworks, pure web standards
- **CSS3**: Modern styling with animations

### Musical Settings

Current settings (configurable in code):

```javascript
musicalSettings = {
    rootNote: 60,        // Middle C (MIDI 60 = 261.63Hz)
    scale: 'pentatonic', // Default scale (overridden by colors)
    octaveRange: 3       // 3 octaves range (MIDI 48-84)
}

musicalClock = {
    bpm: 120,           // Master tempo
    beatsPerBar: 4      // 4/4 time signature
}

oramicsEffects = {
    paintedEnvelope: { active: false },
    drawnWaveforms: { active: false },
    patternSequencer: { active: false },
    gestureDrones: { active: false }
}
```

## Browser Compatibility

**Recommended:**
- Chrome 90+
- Firefox 88+
- Safari 14+
- Edge 90+

**Requirements:**
- WebGL support
- Web Audio API support
- Floating-point texture support (for fluid simulation)
- PeriodicWave support (for Drawn Waveforms)

## Inspiration & Credits

**Daphne** is inspired by:
- **Daphne Oram** (1925-2003): Pioneer of electronic music and inventor of Oramics
- **Oramics Technique**: Drawing sound onto 35mm film strips, read by photoelectric cells
- **BBC Radiophonic Workshop**: Where early electronic music experimentation flourished
- **Graphic Score Tradition**: Visual notation systems in experimental music

Created as an exploration of interactive audio-visual synthesis combining:
- Fluid dynamics simulation
- Music theory and quantization
- Real-time audio synthesis
- Oramics-inspired gestural sound control
- Glitch aesthetics

## License

This project is provided as-is for educational and creative purposes.

---

**Version**: 1.0 - Oramics Complete  
**Last Updated**: November 13, 2025  
**In Honor Of**: Daphne Oram, electronic music pioneer
