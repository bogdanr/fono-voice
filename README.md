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

## Available voices

The [`ort-1.24.2`](https://github.com/bogdanr/fono-voice/releases/tag/ort-1.24.2)
release ships **42 [Piper](https://github.com/rhasspy/piper) voices across 38
languages** — one default voice per language (a few languages carry extra
regional variants, e.g. `en_US`/`en_GB`, `es_ES`/`es_AR`/`es_MX`,
`nl_NL`/`nl_BE`). All are single-architecture Piper VITS models and load on the
same minimal runtime.

It also ships **6 [Kokoro](https://huggingface.co/hexgrad/Kokoro-82M) English
voices** (3 female + 1 male American, 1 female + 1 male British). Unlike Piper,
all Kokoro voices share **one** model (`kokoro-v1.0-q8f16.ort`) and select a
speaker through a small per-voice **`.style.bin`** voice pack (a `510×256` f32
tensor, ~522 KB) — so adding a Kokoro voice costs one style pack, not a whole
model. The style packs are byte-identical to the upstream
[`onnx-community/Kokoro-82M-v1.0-ONNX`](https://huggingface.co/onnx-community/Kokoro-82M-v1.0-ONNX)
voice tensors.

`⚠️` marks a **non-commercial** license — fine for personal/offline use, but
check the upstream terms before any commercial deployment. The authoritative,
machine-readable license for every voice is in [`manifest.json`](manifest.json).

| Voice | Locale | Language | Quality | License |
|---|---|---|---|---|
| `ar_JO-kareem-medium` | ar_JO | Arabic | medium | See upstream MODEL_CARD |
| `bg_BG-dimitar-medium` | bg_BG | Bulgarian | medium | CC0-1.0 |
| `ca_ES-upc_ona-medium` | ca_ES | Catalan | medium | CC-BY-SA-4.0 |
| `cs_CZ-jirka-medium` | cs_CZ | Czech | medium | CC0-1.0 |
| `cy_GB-gwryw_gogleddol-medium` | cy_GB | Welsh | medium | See upstream MODEL_CARD |
| `da_DK-talesyntese-medium` | da_DK | Danish | medium | CC0-1.0 |
| `de_DE-thorsten-medium` | de_DE | German | medium | CC0-1.0 |
| `en_GB-alan-medium` | en_GB | English | medium | See upstream MODEL_CARD |
| `en_US-amy-medium` | en_US | English | medium | See upstream MODEL_CARD |
| `es_AR-daniela-high` | es_AR | Spanish | high | CC-BY-SA-4.0 |
| `es_ES-davefx-medium` | es_ES | Spanish | medium | CC0-1.0 |
| `es_MX-ald-medium` | es_MX | Spanish | medium | CC0-1.0 |
| `eu_ES-antton-medium` | eu_ES | Basque | medium | CC-BY-4.0 |
| `fa_IR-amir-medium` | fa_IR | Farsi | medium | CC0-1.0 |
| `fi_FI-harri-medium` | fi_FI | Finnish | medium | CC0-1.0 |
| `fr_FR-siwis-medium` | fr_FR | French | medium | CC-BY-4.0 |
| `hi_IN-pratham-medium` | hi_IN | Hindi | medium | CC-BY-NC-SA-4.0 ⚠️ |
| `hu_HU-anna-medium` | hu_HU | Hungarian | medium | CC0-1.0 |
| `it_IT-paola-medium` | it_IT | Italian | medium | See upstream MODEL_CARD |
| `ka_GE-natia-medium` | ka_GE | Georgian | medium | See upstream MODEL_CARD |
| `kk_KZ-issai-high` | kk_KZ | Kazakh | high | CC-BY-4.0 |
| `ku_TR-berfin_renas-medium` | ku_TR | Kurmanji Kurdish | medium | CC-BY-NC-4.0 ⚠️ |
| `lb_LU-marylux-medium` | lb_LU | Luxembourgish | medium | CC-BY-NC-SA-4.0 ⚠️ |
| `lv_LV-aivars-medium` | lv_LV | Latvian | medium | CC0-1.0 |
| `nl_BE-nathalie-medium` | nl_BE | Dutch | medium | CC0-1.0 |
| `nl_NL-alex-medium` | nl_NL | Dutch | medium | CC0-1.0 |
| `no_NO-talesyntese-medium` | no_NO | Norwegian | medium | CC0-1.0 |
| `pl_PL-darkman-medium` | pl_PL | Polish | medium | CC0-1.0 |
| `pt_PT-tugão-medium` | pt_PT | Portuguese | medium | CC0-1.0 |
| `ro_RO-mihai-medium` | ro_RO | Romanian | medium | CC0-1.0 |
| `ru_RU-denis-medium` | ru_RU | Russian | medium | CC0-1.0 |
| `sk_SK-lili-medium` | sk_SK | Slovak | medium | CC0-1.0 |
| `sl_SI-artur-medium` | sl_SI | Slovenian | medium | CC-BY-4.0 |
| `sq_AL-edon-medium` | sq_AL | Albanian | medium | CC0-1.0 |
| `sr_RS-serbski_institut-medium` | sr_RS | Serbian | medium | CC-BY-NC-SA-4.0 ⚠️ |
| `sv_SE-alma-medium` | sv_SE | Swedish | medium | CC-BY-4.0 |
| `sw_CD-lanfrica-medium` | sw_CD | Swahili | medium | See upstream MODEL_CARD |
| `tr_TR-dfki-medium` | tr_TR | Turkish | medium | CC-BY-NC-SA-4.0 ⚠️ |
| `uk_UA-ukrainian_tts-medium` | uk_UA | Ukrainian | medium | CC0-1.0 |
| `ur_PK-fasih-medium` | ur_PK | Urdu | medium | MIT |
| `vi_VN-vais1000-medium` | vi_VN | Vietnamese | medium | CC-BY-4.0 |
| `zh_CN-chaowen-medium` | zh_CN | Chinese | medium | CC0-1.0 |

### Kokoro (English)

All share `kokoro-v1.0-q8f16.ort` and carry a per-voice `.style.bin` voice pack.
Licensed **Apache-2.0** (upstream [`hexgrad/Kokoro-82M`](https://huggingface.co/hexgrad/Kokoro-82M)).

| Voice | Locale | Gender | Style pack |
|---|---|---|---|
| `af_heart` | en_US | female | `af_heart.style.bin` |
| `af_bella` | en_US | female | `af_bella.style.bin` |
| `af_nicole` | en_US | female | `af_nicole.style.bin` |
| `am_michael` | en_US | male | `am_michael.style.bin` |
| `bf_emma` | en_GB | female | `bf_emma.style.bin` |
| `bm_lewis` | en_GB | male | `bm_lewis.style.bin` |

> Some languages that Piper offers are intentionally **omitted** here because
> their best voice uses ONNX control-flow operators (`If`) that the current
> minimal runtime is not built with: Greek, Indonesian, Icelandic, Malayalam,
> Nepali, Brazilian Portuguese (European `pt_PT` is included), and Telugu. They
> can be added once the runtime's operator set is extended.

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
upstream license**, taken from the voice's upstream `MODEL_CARD` (Piper) or
model card (Kokoro) and recorded per-voice in `manifest.json`. There is **no
single license** — the current `ort-1.24.2` voices break down as:

| License | Voices | Commercial use |
|---|---|---|
| CC0-1.0 (public domain) | 21 | yes |
| CC-BY-4.0 | 6 | yes, with attribution |
| CC-BY-SA-4.0 | 2 | yes, with attribution + share-alike |
| Apache-2.0 | 6 | yes, with attribution |
| MIT | 1 | yes |
| CC-BY-NC-SA-4.0 | 4 | **non-commercial only** ⚠️ |
| CC-BY-NC-4.0 | 1 | **non-commercial only** ⚠️ |
| See upstream MODEL_CARD | 7 | check the linked dataset URL |

The six Apache-2.0 voices are the Kokoro English voices; the 42 others are
Piper. The five `⚠️` non-commercial voices are listed in the voice table above.
The upstream families:

| Family | Upstream |
|---|---|
| Piper voices | [`rhasspy/piper-voices`](https://huggingface.co/rhasspy/piper-voices) |
| Kokoro voices | [`hexgrad/Kokoro-82M`](https://huggingface.co/hexgrad/Kokoro-82M) — Apache-2.0 |
| Silero / Zipformer / KWS (planned) | per model (Apache-2.0 / MIT) |

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
