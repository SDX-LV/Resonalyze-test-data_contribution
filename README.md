# Resonalyze test data

Real acoustic measurements backing
[Resonalyze](https://github.com/DIMOSUS/Resonalyze)'s field validation:

- `bmw-f30/` — the current snapshot of the car: swept per-driver transfer
  functions, moving-microphone spatial averages of the same channels, and the
  tuned project that pairs the two;
- `array-experiments/` — a seven-position microphone array simulated with one
  microphone, in the same car;
- `nearfield/` — an older system's drivers measured at the cone.

Every impulse response here is stored in the current Resonalyze format
(version 8, sample arrays as base64 float32) at 48 kHz. How they got there,
and what that does and does not change, is at the end of this file.

## bmw-f30: the current snapshot of the car

Measurements made in a BMW F30 sedan. The front drivers use their factory
locations: tweeters in the door mirror sail panels, midrange drivers in the
doors, and woofers beneath the front seats. The subwoofer is installed in a
stealth enclosure in the trunk. The center pass-through in the rear seat
between the cabin and the trunk was open during the measurements.

System configuration:

- Hertz MP 28.3 Pro tweeters;
- Audison Prima AP 4 midrange drivers;
- HELIX Ci5 S200FM-S2 under-seat woofers;
- Focal P25F subwoofer;
- Trioma MOST-SPDIF Link;
- AMP Panacea v2;
- Alpine MRV-M500.

### Swept transfer functions

`subb.json`, `l bass.json`, `r bass.json`, `l mid.json`, `r mid.json`,
`l tw.json` and `r tw.json` — one loopback-referenced transfer measurement per
channel, captured 2026-08-15 at 96 kHz / 24 bit with four averaged runs,
synchronized-loopback timing and SPL calibrated against 114 dB at 1 kHz. The
subwoofer, woofers and midranges were swept 20 Hz to 20 kHz over 2.2 s; the
tweeters 800 Hz to 20 kHz over 1.13 s.

### Moving-microphone averages

`sub mmm.json`, `l bass mmm.json`, `r bass mmm.json`, `l mid mmm.json`,
`r mid mmm.json`, `l tw mmm.json` and `r tw mmm.json` — moving-microphone (MMM)
spatial averages of the same channels, captured 2026-08-25 with the DSP
bypassed, the microphone in continuous motion for the length of each capture.

They are `resonalyze-live-capture` documents (`method: MovingMic`): periodic
pink noise at 48 kHz, a 65536-sample sequence (1365 ms frames), rectangular
window, no overlap, infinite averaging with slope compensation, read on the
SPL scale anchored at 117.03 dB. Integration runs 28.7 to 60.1 s per channel
(21 to 44 averaged frames). All seven carry the same 90-degree
grazing-incidence microphone calibration curve.

Details worth knowing before reading the files:

- the tweeter captures were made behind a protective 48 dB-per-octave
  Butterworth high-pass at 1 kHz, and carry the
  `protectiveHighPassCorrectionDb` term that undoes it — up to +34.4 dB, and
  deliberately absent where the filter is too deep to invert. Their titles
  read `hpf-compensated`;
- the two under-seat woofer captures are titled `l sub mmm` / `r sub mmm`.
  The titles are a slip made at capture time; the file names and the session
  below are what pair them correctly;
- both subwoofer files, `subb.json` and `sub mmm.json`, sit exactly 13 dB
  below the level they were captured at: the samples, the stored preview and
  the recorded microphone level were all lowered by 13 dB, and the capture's
  title says so (`sub mmm -13dB`). The session below is tuned against the
  lowered pair.

### The tuned project

`virtual-dsp-session.json` is the current tune of this car (saved
2026-09-11): a four-way project for an AMP Panacea v1/v2 at 96 kHz under the
Symmetric Q convention, with every channel pairing its swept measurement with
its moving-microphone curve — what the hybrid magnitude view reads.

| channel | crossover | polarity | delay L / R | gain L / R | PEQ bands L / R |
| --- | --- | --- | --- | --- | --- |
| subwoofer | low-pass 70 Hz BW24 | normal | 0.67 ms | 0 dB | 3 |
| under-seat woofers | 70 Hz BW24 – 200 Hz BW36 | normal | 4.25 / 3.25 ms | −3 / −3 dB | 5 / 7 |
| midranges | 200 Hz BW36 – 1586 Hz BW48 | inverted | 12.72 / 10.99 ms | −5 / −2 dB | 14 / 13 |
| tweeters | high-pass 1900 Hz BW48 | inverted | 13.00 / 11.38 ms | −5 / −2 dB | 18 / 19 |

The left tweeter's bank also carries a −3 dB preamp. The target curve is a
custom one: a +16 dB bass shelf at 100 Hz, a −1.5 dB presence dip at 2.5 kHz
and a −3 dB treble shelf at 10 kHz.

The session stores each source twice, as an absolute path and relative to its
own folder. The absolute paths name this repository's folder on the author's
machine; anywhere else they miss, and Resonalyze falls back to the relative
path resolved against the folder the session was opened from. It is a
version 10 session file, which the current build reads and migrates on load.

## array-experiments: a seven-position microphone array, measured with one microphone

A multi-microphone array simulated by a single microphone. In the same BMW F30
on 2026-08-27, the **left** midrange and the **left** tweeter were each swept
eight times while one microphone was physically carried through seven
positions around the listening seat and then returned to the centre:

| file | microphone position |
| --- | --- |
| `mid center.json`, `tw center.json` | centre — the listening position |
| `mid R.json`, `tw R.json` | right |
| `mid RF.json`, `tw RF.json` | right forward |
| `mid RFF.json`, `tw RFF.json` | right further forward |
| `mid L.json`, `tw L.json` | left |
| `mid LF.json`, `tw LF.json` | left forward |
| `mid LFF.json`, `tw LFF.json` | left further forward |
| `mid center final.json`, `tw center final.json` | centre again, closing the run |

The spacing, which the measurement files themselves do not record: the side
positions sit 10 cm to the left and to the right of the centre, the forward
pair adds 10 cm forward to that, and the further-forward pair 20 cm. The
seven therefore span 20 cm across and 20 cm deep, the widest the tripod
would reach. The capsule pointed at the ceiling at every one of them, so the
drivers are at grazing incidence to it throughout.

A real array captures all seven positions in one sweep; here one microphone
stood in for all seven, one sweep at a time, which only holds if the cabin,
the rig and the levels did not move between the first sweep and the last.
The closing centre repeat is the check on exactly that: it is the same
position as the opening one, so whatever separates the two is the run's own
repeatability rather than the array.

Each file is a loopback-referenced transfer measurement captured at
96 kHz / 24 bit: a 1.037 s exponential sweep specified 20 Hz to 20 kHz
(achieved 14.6 Hz to 28.2 kHz), two averaged runs, synchronized-loopback
timing, SPL calibrated against 114 dB at 1 kHz. The tweeter runs sit behind
the same protective 48 dB-per-octave Butterworth high-pass at 1 kHz as the
tweeter captures in `bmw-f30/`; the midrange runs have none.

`mid MMM.json` and `tw MMM.json` are the moving-microphone captures of those
same two drivers, taken at 96 kHz, 41.0 s and 49.2 s of integration. They are
the reference the seven-position average is judged against: averaging the
seven sweeps lands within 0.63 dB RMS (midrange) and 0.49 dB RMS (tweeter) of
the moving-microphone curve at 1/3-octave smoothing, and 1.38 / 1.54 dB RMS
unsmoothed — the accuracy figures behind Resonalyze's microphone-array
support ([Resonalyze
PR #130](https://github.com/DIMOSUS/Resonalyze/pull/130)), computed on the
original 96 kHz files.

The two halves store their calibration differently, and it matters the moment
one is read against the other. An impulse response in this format is always
raw — no calibration is ever baked into one — so the seven position sweeps
hold what the microphone heard, uncorrected. A live capture is the other way
round: it bakes the correction into the drawn curve and records it in
`calibrationCorrectionDb`, which adding back returns the level as measured.
The curve in force throughout the session was the 0-degree on-axis one; the
sweep files of this vintage had no field in which to record that, so it is
stated here instead. Below about 4 kHz it makes no difference, that curve
being flat there; by 10 kHz it is worth 1.3 dB and by 20 kHz 9.7 dB. Read
the sweeps with the 0-degree curve selected, or take the correction back out
of the MMM curve, before comparing the two. Read that way, with both sides
on the 0-degree curve, the average of the seven positions and the
moving-microphone capture agree in tilt across the band — the check that the
two paths are on the same footing.

## nearfield: an older system's drivers at the cone

The seven drivers of an earlier 4-way stereo system measured with the
microphone right at each cone, on axis: `l mid.json`, `r mid.json`,
`l twr.json`, `r twr.json`, `l woof.json`, `r woof.json` and `woofer.json`,
the subwoofer. Loopback-referenced exponential sweeps captured 2026-07-05 at
44.1 kHz, four averaged runs, a 3.09 s sweep over 12 octaves up to the
capture's Nyquist (5.4 Hz to 22.05 kHz).

They were taken ten minutes after a listening-seat pass of the same drivers,
same car, same rig and settings. That seat pass was published here until
2026-09-15 and then removed as stale; the author keeps it offline. Paired with
it, these files give for every driver the same sweep with and without the
cabin's contribution, so excess phase and the first-arrival-to-reflection
ratio measured at the seat can be attributed to the room rather than to the
driver.

It is not anechoic, and should not be read as if it were. A microphone at the
cone inside a car still hears the door cavity, the mounting and the
boundaries around it: this is the driver as installed with the reverberant
field suppressed, not a free-field response. It is a true nearfield only
while the capsule-to-cone distance stays small against the wavelength and
against the radiator — the low end for the woofers, and close-mic on axis
rather than a gated free-field curve for the tweeters.

## Format and sample rate

On 2026-09-15 every impulse response was re-saved in format version 8 at
48 kHz — from 96 kHz for `bmw-f30/` and `array-experiments/`, from 44.1 kHz
for `nearfield/`. The impulse responses shrank from 1.1 GB to 137 MB. The
repository history was squashed at the same time, so the originals at their
captured rates are no longer published; the author keeps them offline,
the two subwoofer files among them at their captured level, 13 dB above the
ones here.

How the samples were carried over:

- the transfer impulse response is circular — its acausal part wraps to the
  far end of the buffer — so it was resampled as one period in the frequency
  domain: every bin below the new Nyquist kept as it was, bins above it
  dropped, the buffer length scaled with the rate;
- the sweep deconvolution response is not circular, and was resampled with
  a 90 dB Kaiser windowed-sinc converter;
- both were rescaled by the rate ratio, so the transfer function's level is
  unchanged (a discrete impulse response of one system scales with the sample
  period);
- the recorded peak indices were kept on the lobe they marked; the new
  sample grid occasionally puts a neighbouring lobe of near-equal height
  higher, which is not taken as a reason to move them;
- the coherence was re-gridded onto the new bins; above 22.05 kHz, where the
  `nearfield/` captures have nothing, it is zero;
- the history preview curve was rebuilt at 48 kHz.

Checked against the originals for every file: within a −20 to +300 ms
window around the recorded peak, magnitude agrees within 0.002 dB and phase
within 0.02° across each measured band up to 20 kHz; the peaks moved by at
most 15 µs, under one sample at 48 kHz. Every file loads through Resonalyze's
own loader and restores as a measurement.

What the conversion leaves alone:

- the sweep metadata still describes the sweep that was played, so the
  achieved band of the 96 kHz captures reaches 28.2 kHz, above the new
  Nyquist; the `nearfield/` files, which recorded their band only as an
  octave count keyed to their own Nyquist, now state it explicitly
  (5.383 Hz to 22.05 kHz), since the same count would describe a different
  sweep at 48 kHz;
- the SPL calibration of the 96 kHz captures still records the 96 kHz input
  it was taken on; the measurement time is kept in `measuredAtUtc`, and
  `savedAtUtc` is the date of the conversion;
- the moving-microphone captures are untouched: they are already in the
  current live-capture format, their spectra are stored only up to 24 kHz,
  and their recipe records the analyzer they were taken with, which no
  resampling can change.

## License

The measurement data, associated metadata/session files, and documentation
are licensed under [Creative Commons Attribution 4.0
International](LICENSE.md). When reusing or referring to the measurements,
please credit:

> Resonalyze test data by DIMOSUS,
> https://github.com/DIMOSUS/Resonalyze-test-data,
> licensed under CC BY 4.0.

Indicate if you modified or processed the data.
