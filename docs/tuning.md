# Tuning guide

Tune one variable at a time and record end-to-end latency, visible control
quality, GPU memory, and failure rate. There is no built-in benchmark.

## Server controls

`--ctx` is the image context length. The default `1` matches the released
model. Increasing it requires a compatible checkpoint and can increase memory
and latency; it does not create planning or persistent memory.

`--cfg` is classifier-free guidance scale, default `1.0`. Higher values may
strengthen conditioning but can amplify unstable actions. Compare values on
the same fixed clips.

The loader uses eight flow-matching inference timesteps by default. The server
exposes this as `--steps`; for example:

```bash
python scripts/serve.py models/ng.pt --steps 4
```

Fewer steps reduce denoising compute approximately linearly, but may reduce
action quality. Compare 8, 4, and 2 steps on the same gameplay clips before
using a lower value. The default remains eight for compatibility.

Per-request shape and timing logs are disabled by default because they are on
the hot path. Use `--verbose-inference` when diagnosing a run rather than
enabling it during latency measurements.

`--port` changes only connectivity. `--old-layout` selects a legacy layout
option and should only be used with a compatible checkpoint; verify decoded
action order.

## Safe experiment order

1. Baseline with `ctx=1`, `cfg=1.0`, and eight steps.
2. Measure capture-to-action latency and VRAM.
3. Compare small CFG changes on fixed clips.
4. Change timestep count only after confirming denoising dominates latency.
5. Experiment with context only after checking tensor shapes and checkpoint
   modality settings.
6. Change mappings only with a saved before/after action trace.

The client’s 256x256 resize, aspect-ratio policy, color order, and capture
frequency are also behavior controls. Change them only with matching processor
and checkpoint assumptions. Tuning cannot add planning, keyboard/mouse support,
termination detection, unseen-game adaptation, or missing controller mappings.
