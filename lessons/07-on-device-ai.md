# 07 — On-device AI: quantization reality and the "unsupported hardware" myth

From shipping a local-LLM Android app (Localyze), a three-platform desktop port, and a
Snapdragon-NPU research recipe. This is the domain where public documentation is most wrong and
only device-measured numbers count.

## The headline finding: "unsupported" usually means "nobody shipped the artifact"

Conventional wisdom (forums, docs, tutorials) said LLMs on Qualcomm's Hexagon NPU require
Snapdragon 8 Gen 2+. The hardware support for 8 Gen 1 (Hexagon v69) had existed in ExecuTorch
source for over a year — **what was missing was a single published `.pte` file targeting v69.**
When one appeared (uploaded quietly, 3 downloads), it unblocked the entire 8 Gen 1 install base:
measured **31.3 tok/s, 107ms TTFT** on a OnePlus 10 Pro that "couldn't run LLMs on NPU."

Rule: before accepting "hardware X can't do Y," separate *hardware capability* from *artifact
availability*. Check whether anyone has actually compiled for the target, not whether forums say
it works.

## Measured limits that no datasheet tells you

- **GPU delegates can be float-only.** A 1.5B model quantized to int8 on disk de-quantized to
  FP16 at engine init — 1.47GiB on disk became 2.5–3GB in VRAM and OOM'd a 0.5GB-free GPU.
  Disk size ≠ runtime size; know your delegate's compute dtype.
- **NPU throughput can be worse than GPU.** INT8 on the v69 HTP measured **0.42 tok/s** vs the
  GPU path's 7–10 tok/s for the same model family. "Runs on NPU" and "faster on NPU" are
  different claims; benchmark before architecting around the accelerator.
- **Quantization pipelines fail in mundane ways:** toolchains rejecting specific ONNX ops,
  host-RAM OOM during quantization, dtype support gaps (INT4 rejected, INT8 only), buffer-name
  collisions on graph splits. Budget for the pipeline, not just the model.
- **KV-cache is the memory lever:** cutting cache 8192→2048 tokens (a 75% reduction) is what made
  mid-range GPUs viable — most mobile chats never need the full window.

## Shipping rules for local-model products

1. **The CPU fallback masks every GPU/NPU failure.** The app "works" while silently pinning a core
   at 100%. Log which backend actually ran, surface it in settings, and treat unexpected fallback
   as an error to fix, not a save.
2. **Preflight the hardware, don't try-catch it.** A compatibility probe (arch + driver + available
   memory) before engine init beats crashing into an allocation failure. Refuse loudly with a
   hardware-fit warning rather than degrading silently.
3. **Small models need guardrails at the output boundary:** a repetition-loop detector (fuzzy
   similarity over a sliding window, cancel mid-stream) and a post-processor that strips
   harness/markup leaks. Put them at the single choke point, not at every call site.
4. **Test on the worst device you claim to support** — the launch-day disaster (52.6s for "Say
   hello") was a model/device pairing nobody had run once.
5. **Model downloads are a product surface:** resumable (HTTP Range), pausable, background, with
   checksums from a manifest keyed by detected hardware. Users judge the app by its first
   five minutes, which is the download.
