# Dataset and training context

The repository is an inference/runtime project. It does not contain the game
environments or the full training pipeline.

The public Hugging Face dataset contains action annotations for public
gameplay videos split into 20-second chunks. Chunks can contain raw and
processed Parquet actions plus metadata such as source video, timestamps,
game, controller type, and bounding boxes. The dataset card describes 17
boolean buttons and two joystick vectors in `[-1, 1]`, and warns that
paper reproduction requires additional filtering such as IDLE-frame filtering.

The website describes controller-overlay localization with template/keypoint
matching followed by a hybrid classification/segmentation model. It reports
approximately 0.84 joystick R2 and 0.96 button frame accuracy. The dataset
card reports roughly 30,000 annotated videos and 15B frames; the website
describes filtered analysis as roughly 40,000 hours across over 1,000 games.
These are external descriptions, not measurements of this checkout.

The runtime contract is simply:

```text
RGB frame(s) -> action horizon -> gamepad values
```

The local tokenizer and game mapping are checkpoint-sensitive; a generic
21-value dataset row cannot be passed to the virtual controller without
checking token order and decoding.

The dataset card lists CC BY-NC 4.0. The model card lists the NVIDIA One-Way
Noncommercial License and notes the SigLIP2 backbone’s Apache 2.0 license.
Review current upstream terms before redistribution or commercial use.
