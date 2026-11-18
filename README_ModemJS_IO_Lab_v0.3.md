# ModemJS I/O Lab v0.3 — JSON Reports & QRmulti transfer

An offline, single-file HTML/JavaScript diagnostic lab for investigating a future **4-FSK + chirp** acoustic modem. V0.3 adds **timestamped measurements, structured JSON reports, and QRF1 QR transfers compatible with the supplied `QRmulti.html`**.

**Downloads**: `ModemJS_IO_Lab_v0.3.html` is self-contained; no CDN or network calls are required. The original `QRmulti.html` is only needed on the receiving/reassembly side of an optical QR file transfer.

> **Scope:** ModemJS v0.3 is an audio diagnostic instrument. It does **not** yet implement chirp detection, decoded FSK data, packet framing, CRC on acoustic packets, ARQ, ACK/retransmission, BER, or measured throughput. QR transfer uses its **own** `QRF1` framing and CRC32 and is unrelated to the planned acoustic modem packets.

## Recommended setup (two laptops)

1. Open `ModemJS_IO_Lab_v0.3.html` on **both** computers. A recent Chromium-based browser on `https://` or `http://localhost` supports the widest range of microphone and output routing features. A local `file://` launch may work, depending on browser security/permissions.
2. On both sides enter **the same `Experiment ID`** in section **8. Link study and export**. Use distinct `Laptop label` values, such as `Laptop-A` and `Laptop-B`. Describe the direction and environment in `Test direction / condition` and `Notes`.
3. Laptop A: choose **TX only**, select output device, set modest speaker volume (start at 10%), and click **Start audio engine**.
4. Laptop B: choose **RX only**, select a microphone, leave `raw-ish input` checked (requests suppression/AGC/AEC off), click **Start audio engine**.
5. To characterize the room's noise floor, keep TX **silent**. On B choose `Baseline / TX silent`, choose 5–30 seconds, and click **Start RX capture**.
6. Start a continuous 18,500 Hz tone on A. On B choose `Signal / test transmission` and capture for 5–30 seconds. Stop A's tone afterwards.
7. Repeat with A's `Send chirp + 4-FSK probe` and/or individual carrier tones at **17,600 / 18,200 / 18,800 / 19,400 Hz**. For the short probe, begin the RX capture **before** pressing Send.
8. Reverse direction and repeat: B TX, A RX. Keep the experiment ID, change condition to `B-to-A`, and compare the two report files.

You can also open **two browser tabs on a single laptop**, with TX-only in one tab and RX-only in another. The inter-tab presence channel does *not* transmit the modem audio. Acoustic coupling must still occur between the speaker and microphone, and different browser tabs may share the same hardware audio device.

## Reports and captures

Section 8 provides two ways to record RX results:

- **Measure link for 5 s** (section 5): records one 5-second `signal` capture.
- **Start RX capture** / **Stop capture** (section 8): choose 5, 15, 30, 60 seconds or manual stopping (up to 180 seconds), and label as signal or baseline.

A capture stores approximately **five spectrum samples per second**, including:

| Field | Meaning |
|---|---|
| `relativeMs`, `at` | Relative time and ISO-8601 UTC timestamp |
| `rmsDbfs`, `peakDbfs` | Time-domain digital input levels after software RX gain |
| `dominantHz` | Frequency of the strongest FFT bin in the analysis band |
| `spectralPeakDbfs` | Strongest FFT-bin level in the selected band |
| `spectralFloorDbfs` | Median non-neighboring FFT-bin level |
| `spectralContrastDb` | Spectral peak minus spectral floor in dB (the old **SNR estimate** display) |
| `fskLevelsDbfs` | Four measured FSK-bin levels, ordered F0–F3 |
| `strongestSymbol` | Index (0–3) of the strongest FSK carrier |
| `clippingNearFullScale` | Whether the peak approached full-scale digital amplitude |

Each completed capture also includes a statistical summary (count, minimum, maximum, mean, median, p10, p90 and standard deviation) for applicable metrics; four per-FSK carrier statistics; an FSK winner histogram; and the capture-specific configuration.

A **TX-only** tab has no microphone capture, but still produces a valid report with TX tone/chirp/probe events, configuration, device and timing metadata, and impairment changes. You do not need a running audio engine just to generate a report.

### Downloading a full JSON report

1. Click **Freeze / refresh report snapshot** in section 8. This creates a consistent snapshot of everything measured *so far*.
2. Click **Download JSON report**. The saved filename includes laptop label, experiment ID, and UTC timestamp.
3. For further experiments, take a **new snapshot** to include later measurements. **Download JSON** and **Generate report QR frames** use the same frozen snapshot unless you refresh it.
4. Upload the two JSON files (A and B) for comparison. No server is contacted by ModemJS; downloads occur locally.

The JSON contains a `schema` identifier (`modemjs.io-lab-report.v1`), metadata, browser environment and audio device configuration, transmitter events, receiver captures, full event log, and interpretation caveats.

**Privacy:** A report includes browser/device labels and opaque device identifiers if exposed by the browser, as well as manual notes. Review it before sharing. It does not contain raw microphone recordings.

## QR transfer: exact `QRmulti.html` compatibility

The report's QR generator is self-contained and embeds **pako Deflate** and Kazuhiko Arase's **QRCode for JavaScript** from the uploaded `QRmulti.html` implementation. It uses the original **QRF1** protocol, with a compatible header in each QR:

```
QRF1|transfer-id|part|total|original-bytes|compressed-bytes|crc32|filename-base64url|data-base64url
```

CRC32 applies to the **compressed** bytes. Part numbering begins at **1**. Duplicate scan parts are ignored during QRmulti reassembly; all original parts must be present to reconstruct the file correctly.

### Generate QR frames on a laptop

1. First **Freeze / refresh report snapshot**.
2. Select **QR content**:
   - **Full JSON** transfers the exact same full report as the `.json` download, including samples and logs. It can require **many** QR frames.
   - **Compact summary JSON** keeps captured summaries, device/settings metadata, and non-log events, omits time-series sample arrays and verbose console text. This reconstructs a **separate `.summary.json`** file and does **not** replace a full report.
3. Select QR version and error correction level. Default **Version 12 / M** is a compromise between per-frame density and frame count. Try version **15** or **20** to reduce the number of frames at the cost of denser QR images. Higher error correction makes individual frames more resilient but reduces capacity.
4. Click **Generate report QR frames**. Use **Previous / Next** to show every frame for an optical scan. Alternatively download **current QR PNG** or **QRF1 payload text** (the latter stores all frames as newline-separated strings).
5. If additional captures are taken after freezing the report, **refresh the snapshot and regenerate the frames**. Do not mix frames from different transfer IDs.

### Reconstruct the JSON using the unmodified QRmulti.html

1. Open the supplied `QRmulti.html` on the receiving device.
2. Scan each QR frame with a QR scanner that can copy the **complete decoded QRF1 text**. Paste all captured frames, one per line, into **3. Reassemble scanned payloads → Scanned payloads**. Frames can be pasted in any order; duplicates are ignored.
3. Alternatively, if you transferred `*.qrf1.txt` by another means, paste its `QRF1|...` lines directly into the same field. Comment lines beginning with `#` can also be pasted.
4. Wait for **all N/N unique parts** to be present, then click **Validate, decompress, and download**. QRmulti checks part ordering, compressed size, CRC32 and decompressed size, then downloads the reconstructed `.json` or `.summary.json` file.

**Important:** QRmulti does *not* scan a laptop camera directly in this uploaded version; its reconstruction input accepts pasted decoded QR payload text. A separate QR camera scanner application may be used to collect these strings. A PNG image of a QR code is *not* the restored report file.

## Distortion emulator

Section 6 applies disturbances **at the transmitter**: attenuation, fading, two echo paths, sub-cycle timing/phase delay, frequency offset, broadband noise, band limiting, and nonlinearity. Presets include Clean, Mild room, Strong echo, Low SNR, Phase/delay stress, Fading, Clipping and Band-limited. Click **Apply impairments** after manually editing values. Presets apply automatically.

The report records the *current* impairment configuration and a timestamped history of applied changes. To compare impairments, run a new labeled RX capture after changing the transmitter, keeping location, gain and orientation stable. Because captures are stored in a single report, you can record multiple test conditions in one session, though unique `Test direction / condition` labels are recommended.

## Interpreting measurements

The legacy SNR label in the live receiver panel is the difference between the **largest FFT bin** and a **median spectral floor estimate**. It can report values above 80–90 dB even when true demodulation-channel SNR is much poorer. This is why the JSON field is named **`spectralContrastDb`**. Do not use these numbers as calibrated RF-style SNR, BER, packet success, or audio SINAD.

Dominant frequency uses FFT-bin resolution; reporting an average of 18,501 Hz does *not* imply that independent audio clocks were measured to 1-Hz accuracy. Software microphone sensitivity is **post-capture gain** and is not guaranteed to control the hardware mic preamplifier.

Changes incorporated in v0.3 also ensure that the **Clean** preset's waveshaper has an identity response and that the channel attenuation also affects the echo taps, reducing two unintended asymmetries in the earlier diagnostic emulator.

## Common troubleshooting

- **No devices visible:** grant microphone access; refresh device enumeration; try Chrome/Edge via localhost/HTTPS.
- **Speaker selection does not work:** `setSinkId()` and `selectAudioOutput()` are browser dependent. Try the system default output or use a browser supporting explicit output routing.
- **Receiver capture disabled:** set role to **RX only** or **TX + RX**, then start the audio engine.
- **No 18.5 kHz received:** verify physical speaker/mic routing, speaker volume, and actual high-frequency response. Try 15–17 kHz for a preliminary hardware check.
- **QR has many frames:** select **Compact summary** (if raw samples are not required), or use a larger QR version; full JSON still downloads separately.
- **Cannot reconstruct in QRmulti:** collect *every* QRF1 frame of one transfer ID; check scanner truncation; do not mix old and freshly regenerated sequences.
- **QR reading difficult:** display at 100% or higher zoom with good focus and contrast, lower QR version / lower error correction to reduce density if possible, and verify that the decoded payload begins `QRF1|`.

## Verification

The v0.3 application was browser-tested for JSON export and QR generation. Its exported QRF1 sequence was passed into the **original uploaded `QRmulti.html`**, which restored a **byte-for-byte identical** JSON file. A synthetic 18.5-kHz microphone stream was also used to test a timed 5-second capture and verify time-series and summary structure. Actual two-laptop acoustic measurements and individual laptop browser/device compatibility must still be verified on the user's hardware.
