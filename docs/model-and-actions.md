# Model and action contract

The model combines a Hugging Face vision encoder (the released configuration
uses SigLIP) with a diffusion/flow-matching Transformer (DiT) action model.
The checkpoint configuration is authoritative for action width, horizon,
vision encoder, modality settings, and inference parameters.

The public model card describes approximately 493M parameters and 256x256 RGB
input. The client also resizes screenshots to 256x256. The Hugging Face image
processor may then perform additional resizing, conversion, or normalization;
those values come from the configured processor rather than hard-coded runtime
constants.

## Temporal behavior

The server defaults `--ctx` to `1`, matching the README statement that the
released model sees only the last frame. This is a reactive policy with no
persistent memory, planner, automatic retry, goal detector, or episode
controller.

## Actions

The public dataset card describes 17 boolean buttons plus two 2D joysticks:

```text
17 button values + 2 left-stick values + 2 right-stick values = 21 values
```

The repository’s shared token list uses controller-oriented names and includes
`RIGHT_BOTTOM`, `RIGHT_LEFT`, `RIGHT_RIGHT`, and `RIGHT_UP`. Those names are
not all implemented by the gamepad adapter. The checkpoint tokenizer and game
mapping therefore matter more than assuming a generic 21-column layout.

Internally, continuous action values are decoded in `[0, 1]`; joystick values
are mapped to `[-1, 1]` and button channels are thresholded at `0.5`.
Joystick coordinates use the conventional orientation where negative Y is up.

## Preprocessing

The client captures RGB, converts through NumPy/OpenCV, resizes to 256x256,
converts back to RGB, and wraps the result as a PIL image. The server passes it
to `AutoImageProcessor`. Keep crop, aspect ratio, color order, and UI
visibility consistent with the live client.

## Compatibility checklist

For a modified checkpoint verify `action_dim`, `action_horizon`, token order,
game mapping, vision encoder/processor, context length, CUDA dtype support,
and that every output action is supported by the virtual gamepad adapter.
