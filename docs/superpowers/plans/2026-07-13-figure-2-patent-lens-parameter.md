# Figure 2 Patent Lens Parameter Diagram Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the manually drawn manuscript Figure 2 with a reproducible, annotated rendering of held-out patent design `patent_000972`.

**Architecture:** A new script in `optics_data_modder_2` selects and validates the exact patent row, reuses the repository's lens renderer and geometry helpers, overlays symbolic annotations, and writes PDF/SVG/PNG plus a provenance manifest into the paper repository. The paper then references the vector PDF while retaining the original manual PNG.

**Tech Stack:** Python 3 in `/home/manideep/miniforge3/envs/rayoptics/bin/python`, pandas, NumPy, Matplotlib, pytest, LaTeX/latexmk, Poppler.

## Global Constraints

- Select exactly `dataset_row_id=patent_000972`; never substitute a fallback row.
- Use zero-based stop annotation `i=0` and example surface `j=7`.
- Do not add rays, image plane, axes, ticks, title, or legend.
- Do not modify or delete `figures/lens_params_def.png`.
- Emit deterministic PDF, SVG, and PNG files plus a JSON provenance manifest.
- Use the optics repository's required `rayoptics` environment without installing dependencies.
- Preserve unrelated untracked files in both repositories.

## File Structure

- `optics_data_modder_2/experiment_scripts/2026_07_13/make_patent_lens_parameter_figure.py`: source selection, validation, geometry, rendering, output hashing, and CLI.
- `optics_data_modder_2/tests/test_make_patent_lens_parameter_figure.py`: focused behavioral and deterministic-output tests.
- `optics_data_modder_2/notes/commits/2026-07-13/add-programmatic-patent-lens-figure.md`: required commit note for the optics-repo implementation commit.
- `paper_v4/figures/lens_params_def_programmatic/*`: generated review artifacts and provenance manifest.
- `paper_v4/sections/dataset_section.tex`: notation, PDF include path, and caption.

---

### Task 1: Define and test the generator contract

**Files:**
- Create: `optics_data_modder_2/tests/test_make_patent_lens_parameter_figure.py`
- Create: `optics_data_modder_2/experiment_scripts/2026_07_13/make_patent_lens_parameter_figure.py`

**Interfaces:**
- Produces: `load_patent_row(path: Path, dataset_row_id: str) -> pd.Series`
- Produces: `validate_patent_row(row: pd.Series, example_surface: int) -> dict[str, int | str]`
- Produces: `surface_geometry(row: pd.Series, surface_index: int) -> tuple[np.ndarray, np.ndarray]`
- Produces: `make_figure(row: pd.Series, example_surface: int) -> matplotlib.figure.Figure`
- Produces: `write_outputs(row: pd.Series, source_csv: Path, output_dir: Path) -> dict[str, Any]`

- [ ] **Step 1: Write failing source-selection and validation tests**

Create tests that load the module with `importlib.util`, write minimal CSV fixtures, and assert:

```python
assert module.load_patent_row(csv_path, "patent_000972")["dataset_row_id"] == "patent_000972"
with pytest.raises(RuntimeError, match="exactly one"):
    module.load_patent_row(duplicate_csv, "patent_000972")
assert module.validate_patent_row(valid_row, example_surface=7) == {
    "dataset_row_id": "patent_000972",
    "surface_count": 9,
    "element_count": 4,
    "stop_index": 0,
    "example_surface": 7,
}
```

- [ ] **Step 2: Run the focused tests and confirm RED**

Run:

```bash
/home/manideep/miniforge3/envs/rayoptics/bin/python -m pytest \
  tests/test_make_patent_lens_parameter_figure.py -q
```

Expected: collection or assertion failure because the generator API is not implemented.

- [ ] **Step 3: Implement minimal selection, normalization, and validation**

Implement exact-row selection, missing-material-to-air normalization, numeric checks for active radii/thicknesses/semi-diameters, one-based `sto` to zero-based stop conversion, and glass-interval counting. Validation must reject a missing source, duplicate/missing IDs, invalid stop, unavailable surfaces 7/8, and a non-glass interval after surface 7.

- [ ] **Step 4: Run focused tests and confirm GREEN**

Run the Task 1 pytest command. Expected: all Task 1 tests pass with no warnings from the new code.

### Task 2: Render deterministic annotated artifacts

**Files:**
- Modify: `optics_data_modder_2/tests/test_make_patent_lens_parameter_figure.py`
- Modify: `optics_data_modder_2/experiment_scripts/2026_07_13/make_patent_lens_parameter_figure.py`
- Create: `optics_data_modder_2/notes/commits/2026-07-13/add-programmatic-patent-lens-figure.md`

**Interfaces:**
- Consumes: validated patent row and surface geometry from Task 1.
- Produces: stable PDF/SVG/PNG artifacts and manifest dictionary.

- [ ] **Step 1: Add failing rendering and determinism tests**

Tests must assert that `make_figure` contains the six approved symbolic labels and that two independent calls to `write_outputs` produce identical SHA-256 values for PDF, SVG, and PNG:

```python
labels = {text.get_text() for text in figure.axes[0].texts}
assert {
    r"Radius of curvature $R_j$",
    r"Thickness $t_j$",
    r"Material $j$",
    r"Semi-diameter $h_j$",
    r"Surface $j$",
    r"Surface $i$: aperture stop",
}.issubset(labels)
assert first["output_sha256"] == second["output_sha256"]
```

- [ ] **Step 2: Run the rendering tests and confirm RED**

Run the focused pytest file. Expected: failures because rendering/output functions are absent.

- [ ] **Step 3: Implement the annotated figure and deterministic writers**

Reuse `plot_lens_assembly` for the base lens. Remove renderer-added axes/title/legend, calculate annotation coordinates with `get_surf_yz_read_data`, and add the approved callouts. Set `svg.hashsalt`, fixed PDF metadata, stable PNG metadata, and stable JSON ordering. The manifest must record source/output paths and hashes, row identity, counts, indices, renderer path/revision, runtime versions, and no sampling or filtering beyond exact row selection.

- [ ] **Step 4: Run tests and the real generator**

Run:

```bash
/home/manideep/miniforge3/envs/rayoptics/bin/python -m pytest \
  tests/test_make_patent_lens_parameter_figure.py -q
/home/manideep/miniforge3/envs/rayoptics/bin/python \
  experiment_scripts/2026_07_13/make_patent_lens_parameter_figure.py \
  --output-dir /home/manideep/PARA/PARA/1_projects/lens_data_revive/reports_and_papers/lensgen_paper/paper_v4/figures/lens_params_def_programmatic
```

Expected: tests pass; four stable artifacts are written; manifest identifies `patent_000972`, 9 surfaces, 4 elements, `i=0`, and `j=7`.

- [ ] **Step 5: Write the required optics-repo commit note and commit only task files**

Commit subject: `feat: add programmatic patent lens parameter figure`

Commit body must include:

```text
Commit-Note: notes/commits/2026-07-13/add-programmatic-patent-lens-figure.md
```

### Task 3: Integrate and visually verify Figure 2

**Files:**
- Modify: `paper_v4/sections/dataset_section.tex`
- Modify: `paper_v4/docs/superpowers/specs/2026-07-13-figure-2-patent-lens-parameter-design.md`
- Create: `paper_v4/figures/lens_params_def_programmatic/figure_2_patent_lens_parameters.{pdf,svg,png}`
- Create: `paper_v4/figures/lens_params_def_programmatic/figure_2_patent_lens_parameters_manifest.json`

**Interfaces:**
- Consumes: deterministic artifacts from Task 2.
- Produces: manuscript Figure 2 and a reviewable preview folder.

- [ ] **Step 1: Update manuscript notation, include path, and caption**

Change `Semi-diameter $j$` to `Semi-diameter $h_j$`, point `\includegraphics` to the generated PDF, and state in the caption that the geometry is a representative held-out patent-derived assembly with symbolic annotations.

- [ ] **Step 2: Verify generated formats visually**

Render the PDF with `/usr/bin/pdftoppm` and the SVG with an available local SVG renderer. Inspect those renderings and the native PNG for clipping, overlap, broken symbols, insufficient contrast, or unreadable labels. Adjust only the generator, rerun focused tests, and regenerate if defects are visible.

- [ ] **Step 3: Build and inspect the paper**

Run:

```bash
latexmk -pdf -interaction=nonstopmode -halt-on-error template.tex
```

Locate Figure 2 using `pdftotext`, render its page with `/usr/bin/pdftoppm`, and inspect the final manuscript-scale result.

- [ ] **Step 4: Verify exact scope and commit the paper changes**

Run focused tests again, `git diff --check` in both repositories, inspect both status/diffs, and confirm the original manual PNG hash is unchanged. Commit only the plan/spec correction, manuscript edit, and four generated files with subject `docs: replace Figure 2 with patent lens rendering`.

- [ ] **Step 5: Report the review artifact**

Show the generated PNG inline and link the review folder, generator, manifest, and manuscript source. Report requested-item status as implemented/verified/not implemented/intentionally skipped.
