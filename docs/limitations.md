# Limitations and failure modes

The current project is a research runtime with a narrow supported path.

## Model and product

The README states that the current model sees only the last frame. It does not
currently provide long-horizon planning, end-to-end episode play, online
self-improvement, or reliable play on completely unseen games. The public model
card reports better suitability for gamepad-oriented games than
mouse/keyboard-heavy games.

## Runtime

- Inference is hard-coded to CUDA and uses bfloat16 autocast.
- Gameplay asserts Windows and depends on DXCam, pywinctl, pywin32, vgamepad,
  xspeedhack, and a compatible desktop/game.
- The game must be running with a discoverable playable window.
- Multiple matching processes are reduced to the first after a warning; there
  is no PID-selection flag.
- Window selection uses exact title equality and is fragile for dynamic titles.
- Episode termination and cleanup are limited.
- Unsupported action names can be silently ignored by the gamepad mapping.

## Reliability and security

- The server binds unauthenticated TCP by default.
- Python pickle is used for requests and replies; the endpoint must be trusted.
- There is no schema negotiation, encryption, authentication, health check, or
  structured error reply.
- Malformed requests and inference exceptions can terminate the server.
- The synchronous protocol cannot pipeline frames.
- No automated test suite or CI coverage protects the critical Windows, CUDA,
  capture, controller, or protocol paths.

These points are either explicit in the README or direct consequences of the
current source. Public dataset and paper results are not guarantees that this
local runtime handles every game or machine.
