# Development guide

Start at `scripts/play.py` for the client path and follow calls into
`GamepadEnv`, `ModelClient`, and image preprocessing. For server behavior,
start at `scripts/serve.py`, then follow `InferenceSession.from_ckpt()` and
`InferenceSession.predict()`. For shape/layout changes, inspect
`mm_tokenizers.py` together with `flow_matching_transformer/nitrogen.py`.

## Safe extension points

Checkpoint changes should update validation in `cfg.py` and
`inference_session.py` together: model, tokenizer, modality, action width,
horizon, and processor must agree.

Adding a controller action requires updating vocabulary, decoding/layout
logic, and every controller backend. Prefer explicit unsupported-action errors
over silently dropping a predicted channel.

Transport changes must update both `inference_client.py` and `serve.py`.
Prefer a versioned, schema-checked format and explicit error replies before
adding batching or remote execution.

Windows integration should keep process lookup, capture, and controller
injection separate so each can be tested independently. Add PID/window
selection controls rather than relying on the first process or exact title.

## Validation

The repository has no meaningful automated coverage for its critical path. For
changes, run Python compilation/import checks, CLI `--help` checks where
possible, synthetic tokenizer shape checks, and local fake-client/server
serialization checks. On Windows, separately test process lookup, capture,
controller output, and cleanup against a harmless target before a real game.

When changing a flag, action layout, checkpoint field, device policy, capture
size, or failure behavior, update the corresponding guide and README.
