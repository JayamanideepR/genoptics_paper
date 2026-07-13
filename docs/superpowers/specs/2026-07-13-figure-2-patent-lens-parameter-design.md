# Figure 2 Patent Lens Parameter Diagram Design

## Status

Approved in conversation on 2026-07-13. This document defines the implementation and verification contract for replacing the manually drawn Figure 2 with a reproducible rendering of a real patent-derived lens prescription.

## Goal

Recreate Figure 2 programmatically while preserving its educational purpose: define the per-surface quantities used by the dataset. The new diagram must use the true geometry and material intervals of one held-out patent-derived design, retain symbolic parameter labels, and remain legible at the manuscript's final figure width.

## Non-goals

- Do not add ray traces, spot diagrams, optical-performance plots, or an image plane.
- Do not redesign other manuscript figures.
- Do not delete or overwrite the original manual PNG.
- Do not alter the underlying patent prescription or select a generated/synthetic design.

## Source Design and Provenance

The figure will use `dataset_row_id=patent_000972` from:

`optics_data_modder_2/data/inbox/from_LLESP_publication_ready/optics_publication_artifacts_20260707/selected_designs/tabsyn_full_tuned_lr3e4_patent_condition_anchors_20260710/selected_patent_anchor_physical_rows.csv`

The row is a held-out patent test design with:

- sequence file: `patent_reference_00973.seq`
- parent patent identifier: `patent_000972`
- surface count: 9
- element count: 4 glass intervals
- aperture stop: zero-based surface 0 (`sto=1` in the source row's one-based stop field)

The generator must select this row by exact `dataset_row_id`, require exactly one match, and fail rather than substitute another design.

## Architecture

The generator will live with the source data and optical renderer in:

`optics_data_modder_2/experiment_scripts/2026_07_13/make_patent_lens_parameter_figure.py`

It will use the existing `lens_theory_scripts.visualization_utils.plot_lens_assembly` function for the base assembly and the same lens-geometry helpers for annotation coordinates. This keeps the optical geometry consistent with other LensGen publication figures and avoids a hidden import from the paper repository into a sibling checkout.

The script will accept an explicit paper output directory. It will not install packages, mutate source data, or rely on the shell's current working directory. It will suppress variable creation metadata in the visual files so repeated runs produce byte-identical PDF, SVG, and PNG outputs.

## Figure Composition

The base drawing will show the patent assembly to scale on a white background with:

- translucent blue glass fills;
- black optical-surface outlines;
- a light gray optical axis;
- a red aperture-stop line at zero-based surface 0;
- no axes, ticks, title, legend, ray traces, object plane, or image plane.

Zero-based surface 7, the first surface of the final glass element, will be the symbolic example surface `j`. It provides a clear glass interval and sufficient surrounding whitespace for callouts. The diagram will annotate:

- `Radius of curvature $R_j$`, pointing to surface 7;
- `Thickness $t_j$`, using a horizontal double-headed dimension arrow from surface 7 to surface 8;
- `Material $j$`, pointing into the glass interval after surface 7;
- `Semi-diameter $h_j$`, using a vertical double-headed dimension arrow from the optical axis to the upper edge of surface 7;
- `Surface $j$`, pointing to surface 7;
- `Surface $i$: aperture stop`, pointing to surface 0.

Labels will use a restrained two-accent palette: one color for surface/stop identification and one for parameter dimensions. Text size, arrow weight, and whitespace must remain readable when included at `0.85\linewidth`.

## Manuscript Integration

Generated artifacts will be written to a reviewable folder in the paper repository:

`figures/lens_params_def_programmatic/`

with stable names:

- `figure_2_patent_lens_parameters.pdf`
- `figure_2_patent_lens_parameters.svg`
- `figure_2_patent_lens_parameters.png`
- `figure_2_patent_lens_parameters_manifest.json`

`sections/dataset_section.tex` will be updated to use the vector PDF. The semi-diameter list item will define the approved notation `$h_j$`, and the caption will state that the geometry is a representative held-out patent-derived assembly. The original `figures/lens_params_def.png` will remain unchanged for comparison.

## Manifest and Traceability

The JSON manifest will record:

- source CSV absolute path and SHA-256;
- selected `dataset_row_id`, `parent_patent_id`, and sequence filename;
- active surface count, derived glass-element count, and stop index;
- symbolic annotation indices `i=0` and `j=7`;
- generator path and SHA-256;
- optical-renderer source path and source-repository Git commit;
- output paths, SHA-256 hashes, dimensions, and generation timestamp;
- plotting-library versions and Python executable.

Absolute paths are provenance metadata only; the design identity will also be captured by row identifiers and hashes.

## Validation and Error Handling

The generator will fail with a clear message if:

- the source CSV is missing;
- `patent_000972` has zero or multiple matches;
- active radius, thickness, or semi-diameter fields required for rendering are missing or invalid;
- a material value is neither a recognized glass name nor the dataset's missing-value representation for air;
- the stop index is outside the active surface range;
- surfaces 7 and 8 are unavailable;
- the material interval after surface 7 is not glass;
- any requested output cannot be written.

No fallback row or approximate hand-drawn geometry is permitted.

## Verification

Implementation is accepted only after all of the following checks pass:

1. Run the generator with `/home/manideep/miniforge3/envs/rayoptics/bin/python`, as required by the optics repository, without installing or updating dependencies.
2. Confirm the manifest identifies `patent_000972`, `patent_reference_00973.seq`, 9 surfaces, 4 elements, `i=0`, and `j=7`.
3. Regenerate twice and verify that the visual file hashes are identical after excluding timestamp-only manifest content.
4. Render the generated PDF and SVG to PNG and visually inspect all three outputs for clipping, overlaps, broken glyphs, weak contrast, or unreadable callouts.
5. Build `template.pdf` with the repository's LaTeX workflow and inspect the page containing Figure 2 at final manuscript scale.
6. Run `git diff --check` in every modified repository and review the exact staged file set before committing.

## Expected File Changes

### Optics source repository

- Add `experiment_scripts/2026_07_13/make_patent_lens_parameter_figure.py`.
- Add `tests/test_make_patent_lens_parameter_figure.py` with focused source-selection, validation, element-count, and deterministic-output tests.

### Paper repository

- Add the four generated files under `figures/lens_params_def_programmatic/`.
- Update `sections/dataset_section.tex` to reference the PDF, define `$h_j$`, and revise the caption.
- Keep `figures/lens_params_def.png` unchanged.

## Acceptance Criteria

- Figure 2 is generated from the exact audited patent row rather than manually approximated geometry.
- Every approved symbolic callout is present and unambiguous.
- The figure remains legible at manuscript scale.
- PDF, SVG, PNG, and manifest outputs are reproducible and traceable.
- The manuscript builds with the new PDF and retains the original manual asset for comparison.
