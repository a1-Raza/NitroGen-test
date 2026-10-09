# Runtime and inference protocol

## Lifecycle

```bash
python scripts/serve.py <checkpoint>
python scripts/play.py --process Game.exe
```

The server loads the checkpoint before binding its socket. If the checkpoint
contains a game mapping, `InferenceSession.from_ckpt()` prints game choices
and prompts for a game ID; an empty response selects unconditional inference.
Automation must account for this prompt.

The server binds a ZeroMQ `REP` socket to `tcp://*:<port>` (default `5555`)
and handles one request at a time.

## Wire behavior

The client sends a pickled observation, normally a PIL RGB image. The server
unpickles it, calls `InferenceSession.predict()`, pickles the action, and
sends it back. There is no handshake, authentication, encryption, schema
negotiation, timeout policy, or structured error protocol.

Python pickle can execute code during deserialization. Use a firewall or an
isolated trusted network; never expose this endpoint to an untrusted network.

## Checkpoint loading

`load_model()` uses `torch.load(..., map_location="cpu", weights_only=False)`
and expects `ckpt_config` and `model` entries. The validated configuration
contains model, tokenizer, and modality settings. The configured vision
encoder and image processor are loaded through Hugging Face
`from_pretrained()` calls, so first startup may require network access or a
warm cache.

The model is evaluated on CUDA. Inference allocates CUDA tensors and uses
bfloat16 autocast. There is no CPU fallback or device CLI flag.

## Inference stages

1. `predict()` applies the configured image processor.
2. The vision encoder produces image tokens.
3. `NitrogenTokenizer` assembles image and action tokens.
4. The DiT flow-matching path performs denoising; the loader currently forces
   eight inference timesteps.
5. The decoder maps continuous joystick values and button logits to actions.

Malformed requests, preprocessing errors, CUDA failures, inference exceptions,
and serialization errors are not converted into structured replies and can
terminate the server. A client may then hang or receive a socket error.

## Server flags

| Flag | Default | Effect |
| --- | ---: | --- |
| `ckpt` | required | Local checkpoint path |
| `--port` | `5555` | TCP ZeroMQ port |
| `--old-layout` | false | Legacy layout option |
| `--cfg` | `1.0` | Classifier-free guidance scale |
| `--ctx` | `1` | Image context length |
