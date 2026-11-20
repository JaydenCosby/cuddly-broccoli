# OpenWebRX for Android (native client)

A native Kotlin / Jetpack Compose client for OpenWebRX and OpenWebRX+ receivers. It speaks the
receiver's websocket protocol directly (no WebView): it decodes the FFT and audio streams itself,
draws its own spectrum + waterfall, and has its own tuner controls.

## Features
- Receiver browser fed from ReceiverBook (receiverbook.de), with search, favourites, offline cache,
  and "add your own address" for any other receiver.
- Waterfall + spectrum on a Canvas: tap/drag to tune, pinch to zoom, two-finger pan, zoom buttons.
- Tuner: tappable-digit frequency dial (top half of a digit = up, bottom half = down), step buttons,
  step presets, direct frequency entry, AM / NFM / WFM / LSB / USB / CW (taken from the server's mode list),
  bandwidth, squelch, volume/mute, relative signal meter.
- Band shortcut chips (only bands inside the receiver's coverage), server bookmarks, SDR profile switch.
- Audio keeps playing with the screen off (foreground service).

Not included: digital-mode decoders (FT8, DMR, ...), which OpenWebRX does server side and sends as text/data.

## Build
Open the folder in Android Studio (Koala or newer) and run the `app` configuration, or:

    gradle assembleDebug        # needs the Android SDK (platform 34) and network access

minSdk 26. The first launch downloads the receiver list from ReceiverBook.

## Protocol notes (derived from the OpenWebRX web client)
- Endpoint: `ws(s)://host[:port]/path/ws/`
- Client sends `SERVER DE CLIENT client=... type=receiver`, then `connectionproperties` (audio rates).
- Server sends JSON (`config`, `smeter`, `profiles`, `modes`, `bookmarks`, `clients`, ...) and binary frames
  whose first byte is the type: 1 = FFT, 2 = audio, 3 = secondary FFT, 4 = HD audio.
- FFT/audio are float32 / int16 or IMA-ADPCM depending on `fft_compression` / `audio_compression`.
- Tuning = `dspcontrol` with `offset_freq` (relative to `center_freq`), `mod`, `low_cut`, `high_cut`,
  `squelch_level`, then `{"type":"dspcontrol","action":"start"}` once.

## Tests
`tools/run-core-tests.sh` compiles the Android-independent code (JSON, ADPCM, protocol, URL handling,
modes, ReceiverBook parser) and runs 26 checks. It needs only `kotlinc`.

## Build with GitHub Actions
Push this folder to a GitHub repo; `.github/workflows/build.yml` builds a debug APK on every push and
exposes it as the `openwebrx-debug-apk` artifact on the run page. It can also be started by hand from the Actions tab.
