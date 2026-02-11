# Android-RTL-SDR-DRM-Receiver
Open-source Android app scaffold for Digital Radio Mondiale (DRM) reception with RTL-SDR.

## Status

This repository currently implements a Phase-1-ready scaffold:

- Android app module in Kotlin + Jetpack Compose
- Native NDK module (CMake, C++20, JNI bridge)
- Foreground `DrmDecoderService`
- `NativeBridge` with `StateFlow` streams for spectrum, signal, station, Journaline, slideshow
- UI with 5 screens: Tuning, Player, Journaline, Slideshow, Settings
- C++ module layout for RTL-SDR, DSP, OFDM, DRM, Audio, Data layers
- Lock-free SPSC ring buffer template (`util/ring_buffer.h`)
- Native pipeline split into reader + processing threads with SPSC queue
- FFT-based spectrum generation (`FftEngine`) and FAC lock/status in UI
- Early OFDM helpers wired in (`TimeSync`, `FreqSync`) for sync/frequency-offset indicators
- Multi-mode (A-D) sync scoring and FAC lock hysteresis via `FacProcessor`
- OFDM -> FAC bitstream path wired (`OfdmDemod` -> `DrmDecoder` -> `StationInfo` updates)
- Frame assembly path active: IQ -> FAC/SDC/MSC partitioning with early SDC metadata decode
- Station metadata now resolved from FAC + SDC in `DrmDecoder` (`ServiceInfo`)
- SDC includes TLV + CRC8 parser path (`SdcProcessor`) with fallback decode path
- FAC/SDC/MSC bits are now extracted via symbol-level differential carrier demodulation
- FAC payload uses CRC8 validation and MSC parser feeds audio profile/rate/channel hints
- Journaline text and MOT slideshow items are emitted from native decoder outputs to UI
- EWF alert payload is parsed from MSC, exposed via JNI, shown in Player screen, and forwarded as Android notifications
- Dedicated EWF screen with severity/event-code filter and clear-history action
- EWF event history is persisted via SharedPreferences (restored on app restart, capped to 30 entries)
- Native sanity tests cover FAC raw decode and FAC-FEC path (convolutional + interleaving + Viterbi)

Current JNI worker behavior:

- Tries real `rtl_tcp` connection (`host:port`) and reads live uint8 IQ stream.
- Applies `SET_SAMPLERATE`, `SET_FREQUENCY`, `SET_FREQ_CORRECTION` (PPM), and gain mode/gain commands.
- Generates spectrum and simple SNR estimate from incoming IQ bytes.
- Reader and processing are separated into two native threads with lock-free buffering.
- Falls back automatically to simulation when `rtl_tcp` is unreachable or stream is lost.

## Planned integration

- FFTW3 float plans for DRM OFDM modes A-D
- Replace current built-in FFT implementation with FFTW3 backend
- libsamplerate for 240 kHz -> 12 kHz resampling
- FDK-AAC for DRM transport decoding (`TT_DRM`)
- Oboe-based low-latency audio output

## Build prerequisites

- Android Studio Ladybug+ (or Gradle/AGP with NDK + CMake installed)
- Android SDK Platform 35
- NDK (r26+ recommended)
- CMake 3.22.1

## Native sanity check

Run host-side decoder smoke tests (no Android build required):

```bash
./scripts/run_native_tests.sh
```

## Package

- Kotlin package: `org.drmreceiver`
- Native library: `drm_native`
