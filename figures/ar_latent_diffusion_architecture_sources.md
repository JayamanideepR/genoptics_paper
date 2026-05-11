# AR Latent Diffusion Architecture Figure Source Map

Date: 2026-05-11

Figure files:

- `figures/ar_latent_diffusion_architecture.pdf`
- `figures/ar_latent_diffusion_architecture.svg`
- `figures/ar_latent_diffusion_architecture.png`

Generation script:

- `/home/manideep/PARA/PARA/1_projects/lens_data_revive/code/optics_data_modder_2/experiment_scripts/2026_05_11/ar_latent_diffusion_architecture_figure.py`

## Figure Intent

The figure presents the proposed optics-native autoregressive latent diffusion architecture as a lens-prescription-native model, not merely a generic diffusion model. The intended paper story is:

1. Lens prescriptions are ordered, variable-surface-count optical systems.
2. The model converts them into surface-major, typed surface-parameter tokens.
3. A transformer encoder and learned attention pooling compress valid optical tokens into latent tokens.
4. A conditional diffusion prior is trained over those latent tokens using the vendored `smalldiffusion` implementation.
5. Generated latent tokens are decoded autoregressively back into mixed continuous/categorical lens-prescription parameters, with stop prediction and valid-surface masking before downstream post-processing and ray tracing.

## Evidence Mapping

| Figure element | Local evidence |
|---|---|
| Variable-length lens prescription input | `src/gen_data_processing_utils.py:14-58` parses `surfcount`, `sto`, and `s00_*` surface columns into `(num_samples, max_surfcount, num_surf_params)` tensors. |
| Parameter-type and surface-position tokenization | `src/gen_data_processing_utils.py:66-371` defines parameter-type embeddings, surface-index/Fourier/progress embeddings, `CompositeTokenizer`, and `token_valid` masks. |
| Transformer encoder and learned latent tokens | `src/model_utils.py:244-327` defines the encoder transformer and learned-query latent attention pooling. |
| Latent-conditioned autoregressive decoder | `src/model_utils.py:442-620` defines `LensLatentARModel`, separate encoder/decoder tokenizers, latent pooling, causal decoder with cross-attention, and output heads. |
| Teacher-forced training and stop handling | `src/model_utils.py:626-768` computes parameter losses over valid tokens and stop-surface losses using regression/classification/ordinal heads. |
| Generation from latent tokens | `src/model_utils.py:775-919` implements `generate_from_latent(...)`, autoregressive surface-parameter generation, token-valid masking, and stop prediction. |
| Conditional latent diffusion prior | `smalldiffusion/src/smalldiffusion/model.py:273-352` defines `FiLMConditionalMLP`; `smalldiffusion/src/smalldiffusion/diffusion.py:44-48` defines `ScheduleLogLinear`; `diffusion_model.ipynb` imports `training_loop`, `samples`, `ScheduleLogLinear`, and `FiLMConditionalMLP`. |
| Paper narrative alignment | `paper_v4/sections/abstract.tex:4-6`, `sections/introduction.tex:12-14`, `sections/background.tex:15`, and `sections/new_model_arch.tex:1-43` describe the optics-native conditional latent diffusion contribution. |
| Note alignment | `optics_data_modder_2/notes/reviews/2026-05-07-publication-intent-and-paper-section-plan-consolidated.md:45-66` and `2026-05-06-subagent-publication-review-comments.md:169-198` identify the strongest claim as surface-token representation plus latent AR diffusion, while noting that diffusion is a separate prototype path. |

## Scope And Caveats

- The figure is a methods/architecture schematic, not a result plot.
- The AR latent model file itself does not implement diffusion internally; it exposes latent tokens for downstream diffusion.
- The conditional diffusion path is currently represented by notebooks/prototype artifacts using vendored `smalldiffusion`, not yet by a clean integrated training pipeline.
- The figure should not be used to claim superiority over conditional tabular baselines unless those baselines are actually run.
