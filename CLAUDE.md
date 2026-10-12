# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

This is an Arduino library. Use Arduino CLI for compilation:

```bash
# Install ESP32 core
arduino-cli core update-index
arduino-cli core install esp32:esp32

# Install dependencies
arduino-cli lib install "Adafruit TinyUSB Library"
arduino-cli lib install "Adafruit MPU6050"
arduino-cli lib install "Adafruit_VL53L0X"
arduino-cli lib install "Adafruit SSD1306"
arduino-cli lib install "Adafruit GFX Library"
arduino-cli lib install "Mozzi"

# Compile a specific example
arduino-cli compile --fqbn esp32:esp32:esp32 examples/dongle

# Compile all examples
for d in examples/*/; do arduino-cli compile --fqbn esp32:esp32:esp32 "$d"; done
```

## Architecture

**Core Library Files:**
- `esp_now_midi.h` - Main library class implementing MIDI over ESP-NOW protocol
- `midiHelpers.h` - MIDI protocol constants and the `midi_message` struct (4 bytes: channel, status, firstByte, secondByte)
- `esp_now_midi.py` - CircuitPython version for MicroPython boards

**Message Protocol:**
All MIDI messages use a fixed 4-byte structure transmitted via ESP-NOW. 14-bit values (pitch bend, song position) are split into 7-bit LSB/MSB pairs.

**Callback Architecture:**
The library uses function pointers for event handling. Users register handlers via `setHandleNoteOn()`, `setHandleControlChange()`, etc. A static instance pointer enables ESP-NOW callbacks to invoke member methods.

**Typical Setup:**
```
Client Device (sensors) ──ESP-NOW──► Dongle (ESP32-S2) ──USB-MIDI──► Computer
                        ◄──ESP-NOW──                   ◄──USB-MIDI──
```

## Key Examples

- **dongle/** - USB-MIDI bridge with OLED display, supports up to 20 peers. Set `HAS_DISPLAY` in config.h.
- **client/** - Sensor integration (MPU6050 accelerometer → MIDI CC)
- **sender/** - Simple periodic MIDI message demo
- **print_mac/** - Utility to discover device MAC addresses

## Important Notes

- Dongle requires a USB-MIDI capable board (ESP32-S2 Mini recommended)
- Any ESP32 with WiFi works as a sender/client
- Peer MAC addresses must be hardcoded in client firmware (no runtime discovery)
- All ESP-NOW communication is unencrypted
