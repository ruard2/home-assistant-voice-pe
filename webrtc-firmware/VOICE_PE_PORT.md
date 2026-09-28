# Home Assistant Voice PE WebRTC port

This directory is an isolated port of Espressif's `openai_demo`. It is kept
separate from the working ESPHome firmware so an unfinished WebRTC image can
never replace the recovery path by accident.

## Target architecture

```text
Voice PE -- Opus/SRTP WebRTC --> OpenAI Realtime
         <-- Opus/SRTP WebRTC --
    |
    +-- HTTPS session bootstrap --> Home Assistant add-on (API key stays here)
    +-- WebRTC data channel <----> session events and Home Assistant tools
```

The permanent OpenAI API key must not be compiled into the firmware. The
Espressif demo currently does that and is therefore only the upstream starting
point. The production port will obtain a short-lived client secret from the
Home Assistant add-on.

## Confirmed Voice PE hardware map

| Function | Voice PE connection |
|---|---|
| MCU | ESP32-S3, 16 MB flash, 8 MB octal PSRAM |
| I2C | SDA GPIO5, SCL GPIO6, 400 kHz |
| XMOS voice-kit reset | GPIO4 |
| Microphone I2S | LRCLK GPIO14, BCLK GPIO13, DIN GPIO15, stereo 32-bit, 16 kHz |
| Speaker I2S | LRCLK GPIO7, BCLK GPIO8, DOUT GPIO10, stereo 32-bit, 48 kHz |
| AIC3204 codec | I2C address 0x18 |
| Internal speaker amplifier | GPIO47 |
| LED ring | GPIO21, 12 GRB LEDs |
| Centre button | GPIO0, active low |
| Hardware mute | GPIO3 |
| Headphone detect | GPIO17 |
| Dial | GPIO16 / GPIO18 |

The XMOS firmware is already responsible for the Voice PE microphone frontend
and acoustic echo cancellation. The WebRTC capture provider must consume its
I2S output instead of instantiating the Korvo-2 ES7210 capture path used by the
unmodified demo.

## Port gates

1. Build the unmodified Espressif OpenAI demo reproducibly for ESP32-S3.
2. Replace Korvo-2 board initialization with Voice PE I2C/I2S/AIC3204/XMOS
   initialization and pass a local capture-to-playback loop test.
3. Add Home Assistant add-on bootstrap endpoint and remove the API key from the
   firmware image.
4. Establish direct OpenAI WebRTC with Opus and verify packet loss, jitter,
   capture level, playback level, and AEC before adding wake-word behavior.
5. Restore Okay Nabu, LED phases, mute/button/dial, Home Assistant tool routing,
   meeting recording, diarization, and safe OTA/recovery.

No image from this branch is safe to flash until gate 2 passes on the actual
Voice PE hardware and a recovery image is available over USB.
