# ESP32-Drum-Machine
Virtual/analog and FM modelling drum machine. Run on ESP32 S3 board.

ESP32-S3 Polyphonic Groovebox & Drum Machine
A high-performance, 8-track digital groovebox powered by the ESP32-S3 microcontroller. This project features a custom-built DSP engine capable of real-time FM synthesis, virtual analog subtractive synthesis, sample-accurate sequencing, and master bus effects, all running flawlessly on a microcontroller.

**Features**
8-Track Fixed DSP Allocation: 8 dedicated hardware tracks optimized for zero CPU overlap and monophonic retriggering (preventing phase cancellation and digital clicks).

Advanced Synthesizer Engines: * Analog-style Kicks, Snares, and Hats.

Deep FM Sub-basses.

Polyphonic-sounding Minor and 7th Chords using multi-oscillator clusters.

Master DJ Filter: State Variable Filter (SVF) at -12dB/oct with parameter smoothing (anti-zipper noise) for live performance transitions.

"Ghost Envelope" Sidechain: A mathematically optimized ducking compressor that uses theoretical envelopes (instead of audio signal detection) for zero-latency, CPU-light pumping effects.

Custom Choke Groups: User-assignable muting targets (e.g., Closed Hat perfectly cutting off the Open Hat).

Global Swing/Shuffle: Dynamically shifts odd 16th-note steps for an MPC-style groove.

Visual LED Envelopes: Real-time Trellis feedback using independent visual decay curves to represent audio output, even when tracks are muted.

**Hardware Requirements**
This project utilizes an I2C shared bus and an external ADC to maximize the available GPIO pins on the ESP32-S3.

Core Components:

ESP32-S3 Development Board (Required: 16MB Flash, 8MB PSRAM / Octal SPI).

I2S DAC Module (e.g., PCM5102A or MAX98357A) for high-fidelity audio output.

Adafruit NeoTrellis (4x4) for the RGB sequencer grid.

SSD1306 OLED Display (128x64) via I2C.

ADS1115 16-bit ADC via I2C (Used to read the push-button resistor ladder).

5x Rotary Encoders (Connected directly to ESP32-S3 GPIOs).

Resistor Ladder Network (For combining the 5 encoder push-buttons and external UI buttons into single analog signals).

**DSP & Software Architecture Highlights**
1. The "Ghost" Sidechain Compression
Standard sidechain compression requires a heavy envelope follower to detect the kick drum's audio peaks. To save CPU, this machine uses "Ghost Envelopes". When the Kick is triggered in the sequencer, it simultaneously fires a silent, mathematical envelope (ghost_sc_envs). The master mixer reads this variable and applies mathematical gain reduction to the bass tracks, resulting in a perfect, CPU-free ducking effect.

2. 808-Style Monophonic Retriggering
To prevent the catastrophic volume accumulation and clipping often found in DIY drum machines, this engine uses strict track allocation. If a Kick drum with a long decay is triggered while the previous Kick is still ringing, the DSP engine does not allocate a new voice. Instead, it resets the VCA envelope and filter state while leaving the oscillator phase untouched. This prevents "clicks" from phase discontinuities and keeps the low-end perfectly tight.

3. Parameter Smoothing (Anti-Zipper)
Connecting a digital rotary encoder directly to a DSP filter cutoff causes audio artifacts known as "zipper noise" due to instantaneous value jumping. The Master DJ Filter incorporates a one-pole lowpass filter directly on the control signal (smooth_dj += 0.005f * (dj_filter_val - smooth_dj);), resulting in an analog-feeling, buttery-smooth sweep.

4. Resistor Ladder UI Polling
To save precious GPIO pins for the I2S audio and Rotary Encoders, the physical button clicks (menu navigation, play/stop) are wired through a resistor ladder and read by the external ADS1115 ADC over the I2C bus. The code handles debouncing and long-press detection (e.g., 2-second hold for Swing/Filter menus) purely through voltage threshold logic.

 **Setup & Compilation**
Open the project in the Arduino IDE or PlatformIO.

Ensure you have the ESP32 Board Manager installed (v2.0.0 or higher).

Select ESP32S3 Dev Module.

CRITICAL: In the Tools menu, set PSRAM to OPI PSRAM (Without this, the 8MB RAM will not be utilized).

Set Flash Size to 16MB (128Mb).

Install the required libraries: Adafruit GFX, Adafruit SSD1306, Adafruit NeoTrellis, Adafruit ADS1X15, ESP32Encoder.

Compile and upload. Ensure you run the INIT PATCH command from the Setup Menu on the first boot to format the NVS flash memory.
