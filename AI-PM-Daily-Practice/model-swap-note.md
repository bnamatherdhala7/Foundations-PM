# Model Swap Note

Product studied:

## Assumptions baked into today's design
| Component | Assumes model limit | What I'd delete/simplify if the model were 2x better |
|-----------|--------------------|-------------------------------------------------------|
|           |                    |                                                       |

## Design principles so a better model drops in
- Eval harness first, so a swap is a rerun, not a rewrite
- Thin scaffolding; avoid hardcoded routing around weaknesses
- Model-agnostic interfaces
