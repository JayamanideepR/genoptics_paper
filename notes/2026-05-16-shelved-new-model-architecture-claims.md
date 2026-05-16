# Shelved New-Model Architecture Claims

Date: 2026-05-16

This note records the new generative-model architecture content removed from the active `paper_v4` manuscript when the paper scope was narrowed to the optics dataset, tuned/untuned representation, tabular generative-model benchmarks, and ray-tracing-based post-processing.

The version that still includes these claims is preserved at:

- Git tag: `paper-v4-with-new-model-claims-2026-05-16`
- Archive branch: `archive/paper-v4-with-new-model-claims-2026-05-16`
- Snapshot commit: `777b08c9aaa361286fa0ae2df13d019fe6f4b4fb`

The full shelved section remains in `sections/new_model_arch.tex`, but it is no longer included by `template.tex`.

## Why It Was Removed From This Paper

For the current paper, the main evidence is focused on:

1. a large-scale optical prescription dataset,
2. a tuned tabular/parametric representation for optical systems,
3. benchmarking off-the-shelf unconditional generative models,
4. ray-tracing-based post-processing for improving generated samples.

The new optics-native architecture is still potentially valuable, but it is better treated as a separate paper because it requires a dedicated experimental section, architecture ablations, conditional-generation metrics, and a cleaner implementation story.

## Manuscript Claims Removed

### Abstract

The abstract previously described five contributions. The fifth contribution was:

> an optics-native conditional latent diffusion architecture that represents lens prescriptions as ordered surface-parameter tokens for target-conditioned generation.

The abstract also included this model summary:

> The proposed conditional model combines a transformer-based autoencoder that embeds variable-length lens assemblies into a fixed-length latent space with a conditional diffusion model trained over the resulting latent representations.

The active abstract now stops at the dataset, representation, benchmark suite, and post-processing pipeline.

### Introduction

The introduction previously argued that the generative-model architecture itself must be adapted to the optical prescription representation. The removed framing was:

> the generative model architecture must be adapted to respect the structure and constraints of this representation.

The contribution list previously ended with:

> Finally, we introduce an optics-native conditional latent diffusion architecture that tokenizes lens prescriptions as ordered surface-parameter sequences, embeds parameter type and surface position, and generates target-conditioned designs through a learned latent representation.

The paper-organization paragraph previously said the paper would conclude with the proposed conditional architecture. The active manuscript now concludes with limitations and future work instead.

### Background

The background section previously framed the representation question as a choice among:

- generic tabular rows,
- structured surface-indexed tables,
- ordered optical design objects.

The removed architecture-level motivation was:

> It also motivates a second, architecture-level representation in which prescriptions are treated as variable-length sequences of ordered surface-parameter tokens with explicit surface-position, parameter-type, and valid-surface information.

The active paper now keeps only the tuned tabular representation motivation.

### Benchmark And Evaluation Framework

The evaluation section previously reserved a separate metric family for the proposed conditional model. The removed text was:

> Conditional target satisfaction is treated separately for the proposed conditional model.

and:

> For conditional generation, target satisfaction becomes a separate metric family.

The removed metric-table row was:

| Metric family | Quantities | Stage | Purpose |
| --- | --- | --- | --- |
| Conditional target satisfaction | EFL, EPD, and field-condition error relative to requested targets | Conditional model | Target-directed generation accuracy |

This can be reintroduced in a future conditional-model paper after the conditional experiment design is finalized.

## Shelved Architecture Content

### Proposed Model Name

Working section title:

> Proposed Conditional Generative Model

Potential paper-level framing:

> An optics-native conditional latent diffusion architecture for target-conditioned optical prescription generation.

### Core Motivation

The proposed model was motivated by the limitations of fixed-width tabular representations:

- optical prescriptions are ordered assemblies of surfaces,
- the same curvature, thickness, or material value has different meaning at different surface indices,
- lens designs have variable numbers of active surfaces,
- flattened columns make surface order, masking, and adjacency structure implicit,
- inactive padded surfaces should not be treated as ordinary missing data.

The intended solution was to represent each prescription as an ordered sequence of optics-aware tokens and learn a latent generative model over that sequence.

### Three-Part Framework

The shelved architecture had three main components:

1. Composite tokenizer: maps surface-parameter values into token sequences.
2. Transformer autoencoder: compresses and reconstructs optical prescriptions.
3. Conditional latent diffusion model: samples the learned latent space conditioned on target optical specifications.

### Component Table Content

The removed table summarized the model components as follows:

| Component | Role | Optics-specific design choice |
| --- | --- | --- |
| Composite tokenizer | Converts lens prescriptions into token sequences | Each token carries parameter value, parameter type, and surface-position information |
| Transformer encoder | Maps the prescription sequence into contextual token states | Learns interactions within a surface and across ordered surfaces |
| Attention pooling | Compresses encoder states into a fixed-length latent code | Handles variable active surface count through masked aggregation |
| Autoregressive decoder | Reconstructs the prescription from the latent code | Decodes in surface-major order with causal masking and inactive-surface loss masking |
| Conditional diffusion prior | Generates latent codes for requested design conditions | Uses continuous conditioning through FiLM-modulated denoising layers |
| Post-processing layer | Converts decoded parameters back to valid optical prescriptions | Applies inverse representation transforms and optional physics-based correction |

### Composite Tokenizer

Each optical prescription was converted into a sequence of parameter tokens. Each token represented:

- surface index,
- parameter type,
- parameter value.

The tokenizer used:

1. Value embedding:
   - numerical parameters embedded with learnable affine projections,
   - categorical parameters embedded with lookup tables.
2. Parameter-type embedding:
   - a learnable type embedding distinguishes curvature, thickness, material, aperture, and other optical quantities.
3. Surface-position embedding:
   - surface order encoded using relative Fourier features,
   - for surface `i` in a system with `N` active surfaces, the normalized position was `i / (N - 1)`,
   - sinusoidal Fourier features were projected to the model dimension.
4. Validity mask:
   - inactive padded surfaces were masked out of attention pooling and reconstruction loss.

The aperture stop was treated separately from ordinary surface parameters. The decoder predicted it using a dedicated stop-index head from the latent representation.

### Transformer Autoencoder

The autoencoder adapted a transformer encoder-decoder formulation for optical prescriptions:

- encoder consumes optics-aware tokens from the composite tokenizer,
- encoder captures intra-surface and inter-surface relationships along the optical axis,
- learned attention pooling compresses contextual token states into a fixed-dimensional latent representation,
- decoder reconstructs parameters autoregressively from the latent representation,
- decoder receives the latent representation through cross-attention,
- reconstruction order is surface-major and parameter-minor,
- causal masking prevents the decoder from seeing future output tokens during training,
- encoder-side and decoder-side token embeddings are not shared,
- teacher forcing is used during training,
- numerical parameters use reconstruction losses,
- categorical parameters use classification losses,
- inactive-surface positions are masked out of the loss,
- aperture stop prediction uses a separate stop-index loss.

The associated architecture figure was:

- `figures/ar_latent_diffusion_architecture.pdf`

The removed figure caption was:

> Proposed optics-native autoregressive latent diffusion architecture. Lens prescriptions are tokenized as ordered surface-parameter sequences, compressed into latent tokens by a transformer autoencoder, sampled with a conditional diffusion prior, and decoded autoregressively back into mixed continuous/categorical optical parameters.

### Conditional Latent Diffusion Model

The latent diffusion model operated on autoencoder latents:

- training prescriptions are encoded into fixed-dimensional latent representations,
- latents are flattened and paired with design conditions,
- planned conditions included effective focal length, entrance pupil diameter, and maximum field angle, but the final condition set was still marked as needing confirmation,
- the implementation plan used the `smalldiffusion` package for noise scheduling, training objective, and sampling,
- modifications were focused on continuous optical conditioning and denoising-network conditioning.

The conditioning mechanism used FiLM:

- condition values are embedded with a small neural network,
- FiLM layers modulate intermediate denoising features,
- this lets the diffusion prior shift the sampled latent distribution according to requested optical specifications.

The training/sampling procedure used classifier-free guidance:

- train with noise-prediction MSE over the diffusion schedule,
- randomly drop conditioning inputs and replace them with a null condition,
- at inference, combine conditional and unconditional predictions using a guidance scale,
- reshape generated latent samples back to the autoencoder latent format,
- decode autoregressively into optical parameters,
- pass decoded prescriptions through the inverse representation and post-processing pipeline.

## Future-Paper Requirements Before Reintroducing This Claim

Before this architecture should be claimed in a separate paper, the following evidence should be added:

1. A finalized conditional task definition:
   - requested conditions,
   - condition ranges,
   - train/test split policy,
   - target-satisfaction metrics.
2. A clean implementation description:
   - tokenizer,
   - autoencoder,
   - latent diffusion prior,
   - post-processing interface.
3. Architecture ablations:
   - with/without surface-position embedding,
   - with/without parameter-type embedding,
   - attention pooling vs simpler pooling,
   - autoregressive decoder vs non-autoregressive reconstruction,
   - FiLM conditioning vs concatenation or other conditioning.
4. Conditional-generation results:
   - EFL target error,
   - EPD target error,
   - maximum-field-angle target error,
   - feasibility and CodeV/LLESP success after post-processing,
   - optical performance after simulation.
5. Baselines:
   - unconditional tabular models as distribution-learning baselines,
   - optionally conditional baselines if the paper scope requires direct target-conditioned comparison.
6. Clear positioning:
   - current LensGen paper: dataset, representation, benchmarks, ray-tracing post-processing,
   - future architecture paper: optics-aware tokenization plus conditional latent generative modeling.

## Files Affected In The Active Paper

The new-model claims were removed from the active paper by editing:

- `sections/abstract.tex`
- `sections/introduction.tex`
- `sections/background.tex`
- `sections/generative_models_and_eval_framework_section.tex`
- `template.tex`

The shelved source content remains available in:

- `sections/new_model_arch.tex`

