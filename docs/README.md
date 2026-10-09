# NitroGen documentation

This guide explains the checked-out NitroGen implementation, not just the
public model announcement. The source code is authoritative for runtime
behavior; the linked model card, dataset card, website, and paper provide
training and research context.

## Start here

1. [Architecture and data flow](architecture.md)
2. [Setup and operations](setup-and-operations.md)
3. [Runtime and inference protocol](runtime.md)
4. [Model and action contract](model-and-actions.md)
5. [Tuning guide](tuning.md)
6. [Limitations and failure modes](limitations.md)
7. [Dataset and training context](data-and-training-context.md)
8. [Development guide](development.md)

## Source map

| Area | Primary files |
| --- | --- |
| Server | `scripts/serve.py` |
| Game client | `scripts/play.py` |
| Checkpoint and prediction | `nitrogen/inference_session.py` |
| ZeroMQ client | `nitrogen/inference_client.py` |
| Windows environment | `nitrogen/game_env.py` |
| Action packing | `nitrogen/mm_tokenizers.py` |
| DiT/vision model | `nitrogen/flow_matching_transformer/` |
| Configuration | `nitrogen/cfg.py` |

Statements about flags, shapes, device placement, and failure behavior are
derived from this checkout. Training and dataset claims are external context.
