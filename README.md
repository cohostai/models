# cohostai/models

ONNX model artifacts consumed by [cohost-translator](https://github.com/cohostai/cohost-translator) at runtime.

This repo is added as a git submodule at `models/` in the main project. Large binary files (`*.onnx`) are tracked with [Git LFS](https://git-lfs.com).

## Contents

### PhoWhisper-small (RTranslator layout, INT8)

Re-exported from [`vinai/PhoWhisper-small`](https://huggingface.co/vinai/PhoWhisper-small) via [`model/scripts/export_phowhisper.py`](https://github.com/cohostai/cohost-translator/blob/main/model/scripts/export_phowhisper.py) in the main repo. Used by the dual-engine STT path for Vietnamese recognition.

| File | Size | Purpose |
|---|---|---|
| `PhoWhisper_encoder.onnx` | 94 MB | Mel → encoder hidden states. MatMul-only INT8 (Conv-safe on CPU EP). |
| `PhoWhisper_cache_initializer.onnx` | 14 MB | Encoder K/V projection — runs once per utterance to seed cross-attention cache. All-eligible INT8. |
| `PhoWhisper_decoder.onnx` | 172 MB | Per-token decoder with `cache_position` input (transformers ≥4.46). All-eligible INT8. |

SHA256 checksums in [`PhoWhisper_SHA256SUMS`](./PhoWhisper_SHA256SUMS).

### QNN STT bundle (sm8750 / HTP v79)

The Android NPU path downloads a single release archive instead of cloning this repo or using Git LFS:

```
https://github.com/cohostai/models/releases/download/v0.1.16-npu-models/cohost-qnn-stt-sm8750.tar.bz2
```

Release: [`v0.1.16-npu-models`](https://github.com/cohostai/models/releases/tag/v0.1.16-npu-models)

| File | Size | SHA-256 |
|---|---:|---|
| `cohost-qnn-stt-sm8750.tar.bz2` | 1,011,011,518 bytes | `2629898429a2e194de1eefa6cc3ee32002ae5182aa901ac51b8e2b8ba87e0c53` |

Target device class:

- Qualcomm Snapdragon 8 Elite / `sm8750`
- Hexagon HTP v79
- ONNX Runtime QNN EP / QAIRT 2.42-compatible artifact set

Archive layout after extraction into the app `filesDir`:

```
qnn/
  encoder.onnx
  encoder_qairt_context.bin
  decoder.onnx
  decoder_qairt_context.bin
qnn_pho/
  encoder.onnx
  encoder_qairt_context.bin
  decoder.onnx
  decoder_qairt_context.bin
```

`qnn/` is the Whisper-small English path. `qnn_pho/` is the PhoWhisper-small Vietnamese path. Keep each `.bin` next to its matching `.onnx`; the EPContext wrappers load the context binaries by relative path.

## Cloning

This is a submodule. From the main repo:

```bash
git clone --recurse-submodules https://github.com/cohostai/cohost-translator.git
# or if already cloned:
git submodule update --init --recursive
```

Git LFS pulls the actual binaries on checkout. If you only see small pointer files, run `git lfs pull`.

## Runtime download

End users do not clone this repo. The app downloads runtime artifacts at first launch via `DownloadConfig.kt`.

The CPU PhoWhisper ONNX files use LFS media URLs:

```
https://media.githubusercontent.com/media/cohostai/models/main/<filename>
```

The NPU STT files use the GitHub release archive above. The app verifies the archive SHA-256, extracts it into `filesDir`, deletes the downloaded tarball, then treats the eight extracted QNN files as the required runtime outputs.

## License

Models derived from `vinai/PhoWhisper-small` are subject to the upstream license. The original PhoWhisper paper: [PhoWhisper: Automatic Speech Recognition for Vietnamese](https://arxiv.org/abs/2406.02555).
