# Architecture and data flow

NitroGen is a split runtime: the model server performs GPU inference, while
the game client runs beside the Windows game and owns capture and input.

```text
Windows game -> DXCam/window capture -> GamepadEnv -> play.py
             -> PIL RGB frame -> pickle over ZeroMQ
             -> serve.py -> InferenceSession.predict()
             -> image processor -> vision encoder -> DiT flow matching
             -> tokenizer/decoder -> action over ZeroMQ
             -> virtual gamepad -> Windows game
```

## Responsibilities

`scripts/play.py` parses the target executable, creates `GamepadEnv`, captures
an observation, resizes it to 256x256 RGB, sends it to `ModelClient`, and
applies the returned action. It is a reactive loop: there is no planner,
replay buffer, score monitor, or episode policy.

`nitrogen/game_env.py` is the Windows boundary. It finds a process with
`psutil`, locates a matching window with `pywinctl`, captures it using DXCam,
and drives a virtual controller through `vgamepad`. `xspeedhack` is used for
process-related timing/injection support.

`nitrogen/inference_client.py` is a synchronous ZeroMQ REQ client. It pickles
one observation, waits for one reply, and unpickles the action.

`scripts/serve.py` creates one ZeroMQ REP socket and one `InferenceSession`.
The session validates the checkpoint, loads the configured Hugging Face vision
encoder and processor, reconstructs NitroGen, moves it to CUDA, and decodes
each request.

## Design consequences

- A network hop adds latency to every frame; the REQ/REP client cannot
  pipeline frames.
- The server is a trusted-process component; do not expose its port publicly.
- The model predicts controller actions, not keyboard/mouse events.
- The default context is one frame. More context requires a compatible
  checkpoint/configuration and does not itself create planning or memory.
