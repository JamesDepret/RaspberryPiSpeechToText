# Local Dutch Speech-to-Text on Raspberry Pi

## Goal

Build a fully local Dutch speech-to-text system on a Raspberry Pi.

Requirements:

* Record audio from microphone
* Transcribe Dutch speech
* No cloud APIs
* Free and open-source
* Run entirely on the Raspberry Pi

---

## Initial Implementation

### Recording

Used:

* `sounddevice` for microphone capture
* `soundfile` for WAV export

Recording settings:

```python
SAMPLERATE = 48000
CHANNELS = 1
DURATION = 5
dtype = "int16"
```

Output:

```text
test_python.wav
```

### Transcription

Used:

* `whisper.cpp`
* Multilingual Whisper models
* Forced Dutch language

```bash
--language nl
```

Initial model:

```text
ggml-base.bin
```

---

## Problem 1: Poor Accuracy

### Test 1

Input:

```text
test 1 2 3
```

Output:

```text
Teste in 2-3
```

### Test 2

Input:

```text
Hallo, dit is een testzin in het Nederlands.
```

Output:

```text
Ik hoop dat het een test is in het Nederlandse.
```

### Conclusion

The `base` model was too small to provide reliable Dutch transcription.

---

## Improvement 1: Upgrade the Whisper Model

### Small Model

Changed:

```text
base → small
```

Reason:

* Better language understanding
* Faster than larger models

Results:

* Noticeably faster
* Dutch transcription still unreliable

---

### Medium Model

Changed:

```text
small → medium
```

Reason:

* Accuracy became more important than speed
* Needed reliable Dutch transcription

Results:

* Significant improvement in transcription quality
* Much slower inference

Performance:

```text
Audio Duration: 5 seconds
Transcription Time: 28.02 seconds
RTF: 5.60x
```

Conclusion:

```text
Quality good
Speed poor
```

---

## Improvement 2: Add Voice Activity Detection (VAD)

Added:

```bash
--vad
--vad-model ggml-silero-v6.2.0.bin
```

Reason:

* Ignore silence
* Focus transcription on actual speech
* Reduce unnecessary processing

---

## Improvement 3: Measure Performance

Added timing around the transcription step.

Metrics:

* Transcription Time
* Audio Duration
* Real-Time Factor (RTF)

Formula:

```text
RTF = transcription_time / audio_duration
```

Reason:

Without measurements, optimization becomes guesswork.

---

## Benchmark Results

### Small Model

```text
Quality: Poor
Transcription Time: 7.89 seconds
Audio Duration: 5.00 seconds
RTF: 1.58x
```

### Medium Model

```text
Quality: Good
Transcription Time: 28.02 seconds
Audio Duration: 5.00 seconds
RTF: 5.60x
```

### Conclusion

```text
Small  = Fast but inaccurate
Medium = Accurate but slow
```

---

## Improvement 4: Audio Resampling

Original recording format:

```text
48 kHz
Mono
16-bit PCM
```

Whisper models are optimized around:

```text
16 kHz
Mono
16-bit PCM
```

Added FFmpeg conversion:

```bash
48 kHz → 16 kHz
Mono → Mono
PCM 16-bit
```

Reason:

Provide Whisper with audio in its preferred format.

---

## Results After Resampling

### Accuracy Improved

Dutch transcription quality became noticeably better.

### Speed Improved

The medium model completed transcription faster than before.

### Conclusion

Audio preprocessing has a significant impact on both quality and performance.

---

## Final Architecture

```text
Microphone
    ↓
48 kHz Recording
    ↓
Save WAV
    ↓
FFmpeg Resample
(16 kHz Mono PCM)
    ↓
Whisper.cpp
    ↓
Medium Model
    ↓
Dutch Transcript
```

Configuration:

```text
Language: Dutch (nl)
VAD: Enabled
Performance Monitoring: Enabled
```

---

## Key Lessons Learned

### 1. Model Size Matters

For Dutch transcription:

```text
base   → unusable
small  → mediocre
medium → good
```

### 2. Audio Format Matters

Providing Whisper with properly formatted audio improved:

* Accuracy
* Inference speed

### 3. Measure Everything

Adding performance metrics made it possible to compare:

* Models
* Audio preprocessing changes
* Future optimizations

### 4. Keep VAD Enabled

VAD reduces unnecessary processing of silence and is especially useful for fixed-duration recordings.

---

## Current State

Successfully achieved:

* Fully local Dutch speech-to-text
* Raspberry Pi compatible
* No internet connection required
* Good transcription quality using `medium` (BUT it's very slow, 30 seconds to transcribe 5 seconds)
* Audio preprocessing pipeline (48 kHz → 16 kHz)
* VAD-enabled transcription
* Performance monitoring (RTF)

---

## Future Optimizations

Potential next steps:

* Quantized medium models
* BLAS acceleration
* Thread tuning
* Streaming / live transcription
* Automatic stop-on-silence recording

The core local Dutch transcription pipeline is now working successfully and provides acceptable accuracy using the medium Whisper model.


1) Install and build whisper.cpp on the Pi
sudo apt update
sudo apt install -y git cmake build-essential ffmpeg

git clone https://github.com/ggml-org/whisper.cpp.git
cd whisper.cpp

cmake -B build
cmake --build build -j --config Release

2) download models (https://github.com/openai/whisper -> ggml folder):
cd whisper.cpp
./models/download-ggml-model.sh base

other version that can be used
./models/download-ggml-model.sh tiny
./models/download-ggml-model.sh small

3) make sure models are in the right folder on the raspberry pi, for me it was: 'home/jamesberry/speech_test/'

