# fono-voice

**Pre-converted, version-pinned ONNX Runtime (`.ort`) voice models for the
[Fono](https://github.com/bogdanr/fono) voice stack — and for anyone else who
wants ready-to-run `.ort` text-to-speech models.**

This repository hosts **no source code and no model weights in its git tree.**
The models live as **release assets**, grouped by the ONNX Runtime ABI they were
built for. The git tree contains only this README and a small machine-readable
[`manifest.json`](manifest.json) that indexes every hosted file with its
SHA-256, size, and upstream provenance.

---

## What is this, and why does it exist?

Modern neural TTS voices (Piper, Kokoro, …) are distributed as plain ONNX
(`.onnx`) graphs. ONNX Runtime can also load an **optimized flatbuffer format,
`.ort`**, which:

- loads faster (graph optimizations are baked in ahead of time), and
- is the **only** format a **minimal / mobile** ONNX Runtime build can load —
  such builds drop the `.onnx` (protobuf) graph parser entirely to stay small.

The catch: **`.ort` is coupled to the ONNX Runtime version** that produced it.
A `.ort` file made with onnxruntime 1.24.2 is meant to be consumed by an
onnxruntime 1.24.x runtime. There is **no public hub that hosts `.ort` files** —
HuggingFace's `rhasspy/piper-voices`, `hexgrad/Kokoro`, etc. all ship `.onnx`.

`fono-voice` fills that gap: it takes the upstream `.onnx` voices, converts them
to `.ort` with a **pinned** onnxruntime, checksums everything, and publishes the
result so a small static binary can fetch a model and run it with **zero
conversion tooling on the user's machine** (no Python, no `onnxruntime.tools`).

## Repository layout

- **Releases**, tagged by ABI: `ort-<onnxruntime-version>` (e.g.
  [`ort-1.24.2`](https://github.com/bogdanr/fono-voice/releases/tag/ort-1.24.2)).
  Each release contains, per voice:
  - `<voice>.ort` — the converted model;
  - `<voice>.onnx.json` — the Piper config sidecar (sample rate, phoneme map,
    inference scales) needed to drive the model;
  - `SHA256SUMS` — checksums for every asset in that release.
- **`manifest.json`** (in the tree) — the canonical index: every voice, every
  ABI version, with `sha256`, `size`, the upstream `.onnx` URL and *its* SHA-256,
  and the license. Machine-readable; this is what tooling should parse.

When the pinned onnxruntime version bumps, a **new** `ort-<version>` release is
minted; older releases stay published so existing installs keep working. Models
are re-converted **only** on such a bump (or when a voice is added) — not on any
particular schedule.

## Using these models — outside Fono

You don't need Fono to use these. A `.ort` file is a standard ONNX Runtime
artifact. **Match the runtime version to the release tag** (`ort-1.24.2` →
onnxruntime 1.24.x).

### Download

```sh
TAG=ort-1.24.2
BASE=https://github.com/bogdanr/fono-voice/releases/download/$TAG
curl -fLO $BASE/ro_RO-mihai-medium.ort
curl -fLO $BASE/ro_RO-mihai-medium.onnx.json
curl -fLO $BASE/SHA256SUMS
sha256sum --check --ignore-missing SHA256SUMS   # verify
```

### Run it (Python)

```python
# pip install onnxruntime==1.24.2   # must match the release ABI
import onnxruntime as ort, numpy as np, json

cfg = json.load(open("ro_RO-mihai-medium.onnx.json"))
sess = ort.InferenceSession("ro_RO-mihai-medium.ort")

# Piper VITS single-speaker signature: phoneme ids -> f32 PCM @ sample_rate.
# Map text -> phoneme ids with espeak-ng using cfg["phoneme_id_map"], then:
ids = np.array([[ ... ]], dtype=np.int64)         # [1, T] phoneme ids
lengths = np.array([ids.shape[1]], dtype=np.int64)
scales = np.array([0.667, 1.0, 0.8], dtype=np.float32)  # noise, length, noise_w
audio = sess.run(None, {"input": ids, "input_lengths": lengths, "scales": scales})[0]
# audio is f32 PCM at cfg["audio"]["sample_rate"] (22050 for this voice)
```

> Loading a minimal-build runtime? Disable graph optimizations when creating the
> session (the `.ort` is already optimized), e.g. set
> `SessionOptions.graph_optimization_level = ORT_DISABLE_ALL`.

### Run it (Rust, via the [`ort`](https://crates.io/crates/ort) crate)

```rust
let session = ort::session::Session::builder()?
    .with_optimization_level(ort::session::builder::GraphOptimizationLevel::Disable)?
    .commit_from_file("ro_RO-mihai-medium.ort")?;
// feed input / input_lengths / scales as above
```

### What you still need for Piper voices

A `.ort` is just the acoustic model. To go from **text** to audio you also need:

1. the **`.onnx.json`** sidecar (here in the release), and
2. an **espeak-ng** phonemizer to turn text into the phoneme ids the model
   expects (`phoneme_type: "espeak"`). Any espeak-ng binding works; Fono uses
   the pure-Rust [`espeak-ng`](https://crates.io/crates/espeak-ng) crate.

## How the models are produced (reproducibility)

Every `.ort` is derived from a published upstream `.onnx`, recorded in
`manifest.json` (`source.onnx_url` + `source.onnx_sha256`). Conversion uses
Fono's [`scripts/gen-ort-models.sh`](https://github.com/bogdanr/fono/blob/main/scripts/gen-ort-models.sh),
which wraps onnxruntime's `convert_onnx_models_to_ort` with operator-type
reduction:

```sh
pip install onnxruntime==1.24.2
MODELS_DIR=voice-models OUT_DIR=voice-models/ort sh gen-ort-models.sh
```

You can reproduce any artifact: download the upstream `.onnx` at the pinned URL,
verify its SHA-256 against the manifest, run the converter with the matching
onnxruntime version, and compare.

## Licensing

This repository **redistributes** third-party models; each carries **its own
upstream license**, recorded per-voice in `manifest.json`:

| Family | License | Upstream |
|---|---|---|
| Piper voices | **GPL-3.0-or-later** | [`rhasspy/piper-voices`](https://huggingface.co/rhasspy/piper-voices) |
| Kokoro | Apache-2.0 | [`hexgrad/Kokoro-82M`](https://huggingface.co/hexgrad/Kokoro-82M) |
| Silero / Zipformer / KWS (planned) | Apache-2.0 / MIT | per model |

Conversion to `.ort` is a format change only; it does not alter the model's
license. The **original text** in this repo (this README, `manifest.json`,
and any helper scripts) is licensed **GPL-3.0-or-later** to match Fono.

If you redistribute these files, keep the upstream attribution and license.

## Requesting or contributing a voice

Open an issue with the upstream `.onnx` URL (ideally from `rhasspy/piper-voices`
or another OSI/GPL-compatible source). Llama- and Gemma-family models are **not**
accepted as their licenses are not OSI-approved.

## Relationship to Fono

Fono's downloader reads a small catalog pinned in the
[main repo](https://github.com/bogdanr/fono) and fetches the matching `.ort` +
config from the release here, verifying the SHA-256 from that catalog. The base
URL is configurable, so a fork or a self-hoster can point Fono at their own
mirror of these assets instead.
