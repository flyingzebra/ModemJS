# ModemJS I/O Lab v0.2 — User Manual

**Companion application:** `ModemJS_IO_Lab_v0.2.html`  
**Version:** 0.2 — audio I/O diagnostics, near-ultrasonic carrier testing and simulated channel impairments  
**Status:** Experimental diagnostic tool; **not yet a packet modem**

## 1. Purpose

ModemJS I/O Lab helps you determine whether a browser, audio output, loudspeaker and microphone can carry near-ultrasonic signals between two laptops (or between two browser instances on one laptop). It can generate test tones, chirps and a short four-frequency FSK probe; the receiving instance shows a spectrum, tone levels and approximate signal-to-noise indicators. The transmitting instance can deliberately introduce echoes, noise, fading and other distortions.

This version **does not** yet implement bit decoding, chirp correlation/acquisition, packet framing, CRC validation, TDD scheduling, acknowledgements, retransmissions, BER or packet-error-rate measurements. A high SNR display or visible FSK tone alone does not prove reliable data transfer.

## 2. Requirements and safety

- A current desktop browser with **Web Audio API** support; Chrome/Edge are recommended for the widest range of speaker-selection features.
- One speaker/output for **TX**, one microphone/input for **RX**, or both for **TX + RX**.
- Microphone permission for RX. Some device names are hidden until permission is granted.
- Prefer **HTTPS or `http://localhost`**. Opening the HTML as `file://` may work in some browsers, but microphone access, permissions and speaker routing vary.
- When performing over-the-air tests, place the receiving microphone within a reasonable distance of the transmitting speaker. Start close (for example 20–50 cm) and increase distance only after a baseline test.
- **Begin with low output volume.** Frequencies around 17–20 kHz may be inaudible to some people but audible and unpleasant to others, including children and animals. Do not assume that an inaudible signal is harmless; avoid high-level or prolonged tests. The TX gain slider does not replace the system's hardware volume control.
- Headphones, laptop speakers and microphones may sharply attenuate these frequencies. Detection of devices does **not** guarantee a usable near-ultrasonic channel.

### Optional local server

In the folder containing the HTML file, run:

```bash
python3 -m http.server 8000
```

Then open `http://localhost:8000/ModemJS_IO_Lab_v0.2.html` in each local browser instance. For two **different laptops**, each may run its own localhost server. A server on laptop A accessed from laptop B over a plain `http://192.168...` address does **not** automatically count as a secure context; use HTTPS for that arrangement.

No internet connection, account, web service, or external JavaScript library is required by the application itself.

## 3. Quick-start: two laptops (recommended)

1. Open `ModemJS_IO_Lab_v0.2.html` on **Laptop A** and **Laptop B**.
2. On A, select **TX only**. On B, select **RX only**.
3. On B, click **Request microphone access**, approve permission, then **Refresh devices**. Choose the correct **Microphone / input**.
4. On A, choose **Speaker / output**, or leave **System default**. Use **Choose/authorize speaker** if offered by the browser. Keep the laptop/system volume low.
5. Click **Start audio engine** on both instances.
6. On A, choose **Clean / bypass** and click **Reset to clean**. Set **Speaker software volume** to a low value, initially **10%**.
7. On B, set **Microphone software sensitivity** to **0 dB**. Leave the analysis band at **16,000–21,000 Hz** and FFT size at **16,384**.
8. On A, set the tone to **18,500 Hz** and click **Start continuous tone**.
9. On B, look for a narrow spectral peak near 18.5 kHz and check **Dominant frequency**, **Tone level**, **Noise floor** and **SNR estimate**. Press **Stop tone** on A when finished.
10. On A, click **Send chirp + 4-FSK probe**. On B, observe the spectrum/strongest-FSK indicator while the short test runs. Repeat as required.
11. Run **Measure link for 5 s** on B while A transmits a **continuous tone**: the one-off chirp/probe is too short for a meaningful five-second average.
12. Reverse A/B roles and repeat to test the other acoustic direction.

**Expected result:** The RX spectrum should show a peak near the TX frequency. Its amplitude should rise above the background noise and change when TX level, frequency or device positioning is changed. The probe does **not** produce a decoded payload or success/failure verdict in v0.2.

## 4. Quick-start: one laptop, two instances

1. Open the same HTML in two tabs or windows of the same browser/profile and origin.
2. Set **Tab 1: TX only** and **Tab 2: RX only**.
3. Select the laptop speaker in TX and built-in microphone in RX. Grant microphone permission.
4. Start both audio engines and follow the tone/probe test above.
5. Check **Local ModemJS instances**: the tabs may show each other's instance IDs and roles through `BroadcastChannel`.

**Important:** The cross-tab feature exchanges **presence information only**. It does not route sound, modem data or measurements from TX to RX. The acoustic route remains **speaker → air → microphone**; it is subject to proximity, audio routing, feedback and OS/browser processing. Cross-tab presence may not work if origins or browser profiles differ. It is not a link-quality measurement.

If the physical speaker–microphone loop is unstable on the same laptop, use two laptops or an appropriately level-matched audio loopback. Never connect a headphone output directly to an unprotected microphone input.

## 5. Screen-by-screen control reference

### 5.1 Browser and device assertion

| Control/status | Meaning |
|---|---|
| **Context** | Whether the page is in a secure context. |
| **Audio APIs** | Availability of microphone/device-enumeration and optional speaker-selection APIs. |
| **Minimum link capability** | Whether input and/or output devices are visible to the browser. Visibility is not proof of functional audio. |
| **Refresh devices** | Re-enumerates audio inputs/outputs. |
| **Request microphone access** | Requests microphone permission, briefly opens the input, then stops it; helps reveal device labels. |
| **Choose/authorize speaker** | Opens the system/browser output selector if `selectAudioOutput()` is available. |

**Troubleshooting:** When only the default speaker is shown, try granting device permissions or using the system default. Output selection relies on the browser's `setSinkId()` support; in other browsers the system output may remain in use.

### 5.2 Instance / role

| Role | Needs | What it enables |
|---|---|---|
| **TX only** | Speaker/output | Continuous tone, chirp and FSK probe generation. |
| **RX only** | Microphone/input | FFT, levels, FSK-bin readings and five-second measurement. |
| **TX + RX** | Both | Combined lab testing; use cautiously to avoid acoustic feedback. |

**Start audio engine** initializes the Web Audio graph. **Stop** closes the context and microphone stream. If you change role while running, **stop and restart** to ensure the required capture path is opened. Each instance receives a short ID.

### 5.3 Audio devices

- **Microphone / input:** Select a browser-visible audio input. Changing the selection during active RX attempts to reopen the microphone.
- **Speaker / output:** Select a browser-visible output. Routing is attempted with `HTMLMediaElement.setSinkId()` where available.
- **Request raw-ish microphone input:** Requests `echoCancellation: false`, `noiseSuppression: false`, `autoGainControl: false`, and mono capture. These are **requests**, not guarantees; check **Actual input** track settings displayed below.

For acoustic modems, avoid speech enhancement/AGC wherever possible, since such processing may suppress or distort high-frequency tones.

### 5.4 Transmitter controls

| Control | Default | Use |
|---|---:|---|
| **Speaker software volume** | 10% | Scales generated audio; OS output volume is separate. |
| **Single tone frequency** | 18,500 Hz | Frequency used by **Start continuous tone**. |
| **Start / Stop continuous tone** | — | Sustained carrier for tuning the RX and collecting stable measurements. |
| **Chirp start / end** | 17,000 / 20,000 Hz | Linear-frequency sweep bounds. |
| **Chirp duration** | 40 ms | Duration of the sweep. |
| **Send chirp** | — | One short swept signal. |
| **F0 / F1 / F2 / F3** | 17,600 / 18,200 / 18,800 / 19,400 Hz | Test tones associated with `00`, `01`, `10`, `11`. |
| **Send chirp + 4-FSK probe** | — | Sends chirp, 50 ms guard, then two passes of F0–F3. Each FSK tone lasts 100 ms followed by a 15 ms spacing. |

At **48 kHz sampling**, the Nyquist limit is 24 kHz; at **44.1 kHz** it is 22.05 kHz. Real speakers/microphones often stop being useful much earlier. Keep probe frequencies comfortably below the displayed Nyquist and within the demonstrated device response. The numerical fields are not all automatically validated against Nyquist.

**Meaning of “4-FSK” here:** v0.2 generates a known sequence of four distinct test frequencies. It does not yet turn arbitrary bytes into 4-FSK symbols or decode them.

### 5.5 Receiver controls and metrics

- **Microphone software sensitivity** (`-30` to `+30 dB`): Applies gain **after capture**. It cannot recover information already lost to analog noise or clipping. A higher number is not necessarily better.
- **Analysis band start/end:** Frequency region used when finding the dominant spectral bin and approximate background floor. Default **16–21 kHz**.
- **FFT size:** 4,096 / 8,192 / 16,384 / 32,768. Larger FFTs give finer frequency resolution but generally slower response and longer observation windows.

| Metric | Interpretation | Limitation |
|---|---|---|
| **RMS (dBFS)** | Overall digital input root-mean-square level. | Includes noise and signals outside analysis band. |
| **Peak (dBFS)** | Maximum absolute sampled amplitude in current buffer. | Near 0 dBFS suggests possible clipping; clipping may already have happened in hardware. |
| **Dominant (Hz)** | Frequency of strongest FFT bin within analysis band. | May identify an interference line instead of the TX signal. |
| **Tone level (dBFS)** | Strongest spectral-bin level in analysis band. | FFT/analyser windowing and smoothing affect the displayed level. |
| **Noise floor (dBFS)** | Median of other FFT bins, excluding a small guard around strongest bin. | Heuristic *spectral floor*, not integrated noise power. |
| **SNR estimate (dB)** | Dominant-bin level minus median spectral-bin floor. | **Not a calibrated communication-channel SNR**. May be misleading for chirps, broadband noise, multiple tones and echoes. |
| **FSK detector (F0–F3)** | Spectral level near each configured FSK frequency (±1 FFT bin). | No symbol timing, bit decisions or packet decoding. |

**Strongest FSK symbol** is an instantaneous strongest-bin indication. It can change between frames and should not be interpreted as a decoded symbol stream.

**Measure link for 5 s** averages periodic RX display readings and reports average RMS, average SNR estimate, highest sampled SNR estimate and average dominant frequency. For meaningful comparisons, hold the transmitter on a single **continuous tone** for the complete measurement interval. The routine samples on animation frames, not with a calibrated measurement instrument.

### 5.6 Receiver spectrum

The live FFT plot displays spectral magnitude over approximately **0 Hz to the Nyquist limit**. The selected analysis band is shaded; green markers identify F0–F3. Also shown are:

- **Sample rate:** AudioContext's actual sampling rate.
- **Nyquist:** Half the sample rate.
- **FFT resolution:** `sampleRate / fftSize` Hz per bin.

Use the graph to inspect frequency roll-off, unexpected peaks, spectral spreading, noise and clipping products. If there is no microphone stream, the graph will not update.

### 5.7 Event log

Records startup, device enumeration, permissions, output routing, TX actions, impairment changes and timed measurements. **Clear log** empties the display. There is no built-in log export or data persistence in this version; copy text manually if you need a record.

## 6. Channel impairment emulator

Impairments are applied **on the transmitter**, before the audio stream reaches the selected speaker. They add to—not replace—the room, speaker, microphone and OS distortions. To test an impairment:

1. Establish a **clean** baseline with a sustained tone.
2. In the TX instance, select a preset, or change individual settings.
3. Click **Apply impairments** after editing a custom setting. Selecting a named preset applies it immediately if TX is running.
4. Observe RX tone level, spectral floor, dominant frequency, FSK-bin levels and five-second measurements.
5. Reset to **Clean / bypass**, repeat the baseline and check for drift or physical changes.

**Important implementation detail:** Many impairment controls are applied only after pressing **Apply impairments**. Changing the **frequency offset** also affects *newly generated* oscillators; restart a continuous tone after changing this control. A currently playing tone does not automatically retune.

### 6.1 Presets

| Preset | Illustrative programmed disturbances | Purpose |
|---|---|---|
| **Clean / bypass** | No configured attenuation, echoes, added noise, phase delay or fading | Baseline; still passes through the implementation's filters/waveshaper and real hardware. |
| **Mild room** | −4 dB, 2 dB fading at 0.4 Hz; echoes 3.5 ms/−18 dB and 8 ms/−28 dB; small phase error, noise and bandwidth limits | Mild channel impairment. |
| **Strong echo / multipath** | −3 dB; echoes 5 ms/−6 dB and 13 ms/−10 dB | Delay-spread/ISI stress. |
| **Low SNR** | −10 dB signal attenuation and increased white-noise source gain | Lower received tone-to-background contrast. |
| **Phase / delay stress** | 90° equivalent phase at 18.5 kHz plus +25 Hz TX oscillator offset | Timing and frequency sensitivity. |
| **Amplitude fading** | −3 dB with 15 dB fading at 1.2 Hz | Time-varying received signal strength. |
| **Clipping / overload** | +4 dB gain; soft-clip drive 8× | Nonlinear spectral products. |
| **Band-limited channel** | −3 dB; high-pass 16.5 kHz and low-pass 19.5 kHz | Test tones near cutoff frequencies. |
| **Custom** | User-defined controls | Controlled parameter sweeps. |

Preset names describe **intended stress scenarios**, not calibrated emulations of a measured physical room or microphone.

### 6.2 Individual impairment controls

| Control | Range | Effect / interpretation |
|---|---|---|
| **Channel attenuation** | −60 to +12 dB | Scales transmitted signal before filtering; positive values may cause clipping. |
| **Slow fading depth** | 0 to 30 dB | Periodic gain modulation, approximating a swing from nominal level toward the chosen attenuation depth. |
| **Fading rate** | 0.01–20 Hz | Modulation frequency of the fading envelope. |
| **Echo 1/2 delay** | 0–500 ms | Delayed signal replicas mixed with the direct path. |
| **Echo 1/2 level** | −60 to 0 dB | Relative echo gain; −60 dB is treated as **off** in this implementation. |
| **Phase error** | −180° to +180° | Converted to a short non-negative delay at the selected reference frequency. |
| **Reference frequency** | 100–23,000 Hz | Frequency at which requested phase shift is specified. |
| **Frequency offset** | −1,000 to +1,000 Hz | Added to newly generated single tones, chirps and 4-FSK probe oscillator frequencies. |
| **Noise level** | −90 to −10 dB control | Scales an internally generated noise source; **not a calibrated receiver dBFS or SNR setting**. |
| **High-pass / low-pass** | 20–22,000 / 100–23,000 Hz | Biquad high-/low-pass filters; cutoffs are clamped by the AudioContext's available bandwidth. |
| **Soft clipping drive** | 1–20× | A nonlinear `tanh()` waveshaper; increasing drive compresses and distorts amplitudes. |

**Phase caution:** This is not an independent, constant-angle phase rotation of each FSK carrier. The implementation uses `DelayNode`; the phase shift depends on frequency, and a global pure delay is generally not a significant impairment for **noncoherent FSK energy detection**. It will matter more when the future receiver implements chirp synchronization and packet timing.

**Echo caution:** The UI sets two delays and gains, not true room impulse-response measurements; for short 100 ms symbols a large delayed tap can overlap adjacent symbols. A receiver may show altered frequency-bin power even without a data decoder.

**Noise caution:** Added broadband noise is generated as a looping pseudo-random buffer before the speaker. Acoustic hardware and processing determine how much reaches the microphone. The label is a **control level**, not a guaranteed SNR.

**Clipping caution:** The “clean” preset sets waveshaper drive to **1**, which still uses a normalized `tanh()` response, rather than electrically bypassing the waveshaper. The bypass label means *minimum configured impairments*, not mathematically bit-transparent output.

## 7. Suggested test procedures

### Test A — Device and carrier sanity check

- Set TX to **Clean / bypass**, **10%** software volume, **18.5 kHz** continuous tone.
- Set RX to default analysis band, FFT 16,384 and **0 dB** sensitivity.
- Confirm a spectral peak close to 18.5 kHz.
- Increase/decrease system speaker volume slightly and verify that RX tone level follows.
- Stop TX; RX dominant frequency should no longer consistently track the original tone (unless a matching interferer exists).

### Test B — Four-frequency passband assessment

- Keep distance, orientation, RX gain and system volume constant.
- In turn, transmit continuous tones at **17,600**, **18,200**, **18,800**, and **19,400 Hz**.
- For each, measure for five seconds on RX and record dominant frequency, tone level, floor estimate and peak level.
- Favor frequencies with consistently strong received energy and sufficient separation from interfering peaks. The weakest carrier often limits the practical 4-FSK design.
- You may repeat with other frequencies such as 15–20 kHz to map the laptop's response.

### Test C — Attenuation threshold

- With one continuous tone running, set channel attenuation successively to **0, −6, −12, −18, −24, −30 dB** on TX.
- Press **Apply impairments** each time, collect a stable RX measurement and record the readings.
- The threshold where the RX peak becomes difficult to distinguish from background gives a *rough carrier-detection limit*. It is **not yet a BER threshold**.

### Test D — Noise robustness

- Begin at clean baseline.
- Keep the continuous test tone stable and change the added noise level progressively (e.g., off, −65, −55, −45, −35 dB control values).
- Compare the RX spectrum and SNR estimate.
- Avoid interpreting the noise control itself as “SNR = 35 dB”; the actual spectrum depends on the hardware path and analysis method.

### Test E — Echo/multipath stress

- Select **Strong echo / multipath**, then compare against clean.
- For more controlled work, use **Custom** and change one echo at a time (e.g., 1, 3, 5, 10, 20 ms at −12 dB).
- Inspect frequency-dependent peaks/nulls as you sweep carrier frequency.
- The full effect on symbol recovery must wait for a timed 4-FSK decoder.

### Test F — Frequency and phase stress

- Set a clean baseline at 18.5 kHz.
- Try frequency offsets of **+10, +25, +50, +100 Hz**; **restart the tone** after each change.
- Compare reported dominant frequency with the intended offset and watch how a fixed FSK-bin reading falls as the tone moves outside its ±1-bin window.
- Independently change phase-equivalent delay and note that it may produce little change in a steady, noncoherently measured single tone.

### Test G — Fading, band limits and clipping

- **Fading:** select **Amplitude fading** and watch RX tone level change over several seconds; a five-second average may hide the deepest fades.
- **Band limiting:** select **Band-limited channel** and compare tones near 17, 18.5 and 20 kHz.
- **Clipping:** use low initial system volume, select **Clipping / overload**, and compare new harmonics or spectral broadening. Reduce level if RX approaches 0 dBFS or if the speaker audibly distorts.

### Test H — 4-FSK probe visibility

- Select clean TX and RX defaults.
- Click **Send chirp + 4-FSK probe** several times.
- Observe whether the strongest FSK indicator visits all four frequencies while the probe runs.
- This is **visual inspection**, not verified symbol detection or a successful packet transmission. A proper time-aligned demodulator is planned for a subsequent version.

## 8. Recording results

Record the TX and RX settings for each run. One suggested Markdown table:

| Run | Laptop/speaker → laptop/mic | Distance | TX test | TX gain | Impairment | RX RMS | RX tone level | RX floor | RX SNR estimate | Observation |
|---|---|---|---|---|---|---|---|---|---|---|
| 001 | A → B | 0.3 m | 18,500 Hz tone | 10% | Clean | — | — | — | — | Baseline |
| 002 | A → B | 0.3 m | 18,500 Hz tone | 10% | −12 dB | — | — | — | — | Attenuation test |

Also note the **browser/version, OS, actual audio sample rate, devices, physical geometry and microphone processing settings**. For repeatable comparisons, change only one variable at a time.

## 9. Troubleshooting

| Symptom | Checks / fixes |
|---|---|
| No microphone permission | Open through HTTPS/localhost; inspect browser permission and OS microphone privacy settings; click **Request microphone access**. |
| Audio I/O device missing | Click **Refresh devices**; connect it before restarting the engine; check OS sound settings. |
| Speaker selector ineffective | `setSinkId()` may be unavailable or blocked; use the OS default device and confirm in the event log. |
| No RX spectrum | Confirm RX-only/both role, **Start audio engine**, correct microphone selected, live permissions and nonmuted hardware input. |
| TX tone not visible at RX | Check output routing, system volume, gain, distance, sample rate, and speaker/microphone high-frequency response. Try a clearly audible test frequency at low level to prove basic connectivity first. |
| Input clipping | Reduce TX system/software volume, move microphones farther apart, or reduce OS microphone input level; post-capture RX sensitivity does not undo clipping. |
| Low or unstable SNR estimate | Background ultrasound/interference, room echoes, AGC/DSP or insufficient tone level; compare with TX off and with several test frequencies. |
| FSK winner stuck on one tone | Make sure RX and TX F0–F3 settings agree; check FFT resolution and whether that tone is dominant even with TX off. |
| Impairment slider seems ineffective | Press **Apply impairments**. For **frequency offset**, restart the continuous tone. |
| Two tabs do not show each other | Use the same browser profile and origin; presence only works through `BroadcastChannel` and is not essential to audio testing. |
| RX and TX on same laptop feed back | Use separate TX-only and RX-only tabs, lower system volume and increase mic–speaker separation, or test with two laptops. |

## 10. Known limitations and implementation notes

1. **No digital modem yet:** The application does not transmit user files, encode bytes, detect packet preambles, validate CRCs or retry packets.
2. **Receiver readings are approximate:** The FFT-based “SNR estimate” is a peak-to-median-bin contrast, not a laboratory SNR or demodulator performance metric.
3. **No automatic channel sweep/equalization:** You can manually test frequencies; v0.2 does not automatically identify the best four carriers.
4. **No impairment calibration:** UI attenuation/echo/noise values define internal graph settings, not guaranteed values at the remote microphone.
5. **Only two echo taps:** A simplified multipath model, not a measured room response.
6. **Phase simulation uses delay:** It does not create an independent phase-only distortion or oscillator phase-noise process.
7. **Frequency offset applies to newly created oscillators**, not as a general sample-rate mismatch or post-mix frequency shifter.
8. **TX and RX have different gain controls:** TX volume changes actual generated output level; RX sensitivity is digital post-capture gain.
9. **Clean is not a strict DSP bypass:** Even the clean setting uses filters and the low-drive soft waveshaper.
10. **No persistent settings, automatic reports or CSV export.** Configuration and event log are not saved across reloads.
11. **Near-ultrasonic performance is device-dependent:** 48 kHz browser audio does not imply usable response up to 24 kHz.
12. **Self-test is acoustic, not internal digital loopback:** Two tabs do not bypass the actual hardware channel.

## 11. Planned ModemJS evolution

The next layers can reuse this diagnostic harness:

1. **Channel survey:** receiver-side timed sweeps, automatic carrier recommendations and repeatable measurement export.
2. **Physical-layer receiver:** chirp matched-filter/correlation, packet acquisition, symbol timing and 4-FSK noncoherent energy detection.
3. **Packet layer:** payload framing, sequence numbers and CRC-16.
4. **Bidirectional link protocol:** half-duplex TDD scheduling, DATA/ACK, timeout and retransmission.
5. **Quantitative robustness testing:** known pseudorandom payloads, BER/PER versus SNR, timing errors, echo profiles and fading; throughput and retry statistics.

---

**Test principle:** Establish **device access → clean carrier detection → four-tone passband characterization → controlled impairments → repeatable results** before building higher-layer modem reliability logic.
