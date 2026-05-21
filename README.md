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

## Cloning

This is a submodule. From the main repo:

```bash
git clone --recurse-submodules https://github.com/cohostai/cohost-translator.git
# or if already cloned:
git submodule update --init --recursive
```

Git LFS pulls the actual binaries on checkout. If you only see small pointer files, run `git lfs pull`.

## Runtime download

End users do not clone this repo. The app downloads these files at first launch via `DownloadConfig.kt` using LFS media URLs:

```
https://media.githubusercontent.com/media/cohostai/models/main/<filename>
```

## License

Models derived from `vinai/PhoWhisper-small` are subject to the upstream license. The original PhoWhisper paper: [PhoWhisper: Automatic Speech Recognition for Vietnamese](https://arxiv.org/abs/2406.02555).
