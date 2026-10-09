# Setup and operations

## Requirements

- Windows for the game client, capture, and virtual gamepad.
- A running game that accepts a compatible virtual gamepad.
- Python compatible with the package metadata; maintainers tested Windows 11
  with Python 3.12 or newer.
- A CUDA-capable inference machine with adequate memory.
- Network access or a local Hugging Face cache for the configured vision model.

The package declares Python `>=3.10`, but that is a packaging floor rather than
a guarantee that every Windows dependency works on every 3.10 installation.

## Install and checkpoint

```bash
git clone https://github.com/MineDojo/NitroGen.git
cd NitroGen
pip install -e .
hf download nvidia/NitroGen ng.pt --local-dir models
```

The runtime does not download checkpoints itself. Pass the actual local path
to the server.

## Launch

On the inference host:

```bash
python scripts/serve.py models/ng.pt
```

On the Windows game host:

```bash
python scripts/play.py --process Game.exe
```

The process argument is the exact executable name, including `.exe`, not a
launcher display name. Start the game first and keep its playable window
visible with a stable title. Resolve any server game-selection prompt before
starting the client.

## Troubleshooting

| Symptom | Likely cause | Next check |
| --- | --- | --- |
| Server fails during load | Bad checkpoint, missing cache, invalid config | Inspect the first traceback |
| CUDA/device error | Driver, GPU, dtype, or VRAM issue | Check CUDA visibility and free VRAM |
| Server appears hung | Waiting for game selection | Read its prompt |
| Game not found | Process mismatch or game not running | Compare Task Manager name |
| Window not found | Dynamic title or hidden/minimized window | Make the playable window visible |
| No controls | Virtual pad unsupported or mapping mismatch | Test controller compatibility |
| Client hangs after server error | REQ/REP request was not completed | Restart server and inspect traceback |
| High latency | Capture, network, preprocessing, or denoising | Measure each stage |

There is no health endpoint, metrics exporter, structured error response, or
automatic recovery loop.
