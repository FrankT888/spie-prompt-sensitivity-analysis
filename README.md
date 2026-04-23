# Benchmarking prompt design choices for spatial reasoning and aircraft classification in overhead imagery: toward world model grounding

This repository contains the dataset, experimental results, and analysis code supporting the following publication:

> Tanner, F, "Evaluating Prompt Design Choices for Object Detection, Counting, and Classification in Overhead Imagery," in *Proc. SPIE Defense + Commercial Sensing*, 2026.

## Summary

This paper examines how prompt design choices influence multimodal large language model (MLLM) behavior during object detection, counting, and classification tasks in overhead imagery. Using a balanced 300-image subset from the RarePlanes satellite imagery dataset (713 ground-truth aircraft across five classes), we evaluated three frontier MLLMs -- Claude Opus 4.6, Gemini 3 Pro, and GPT-5.4 -- across three progressively richer prompt conditions:

1. **Simple Prompt** -- a global task-scoped instruction with five-class ontology definitions and a JSON output schema.
2. **GSD-Augmented Prompt** -- extends the simple prompt with ground sample distance (0.30 m/pixel) and explicit size-bracket guidance mapping pixel-span ranges to class categories.
3. **Visual Ontology Prompt** -- extends the GSD-augmented prompt with one representative reference image chip per class as few-shot visual conditioning.

Key findings:
- **Gemini 3 Pro** achieved the highest overall performance across all conditions (up to 92.0% count accuracy, 72.7% classification accuracy, 73.2% localization accuracy).
- **GPT-5.4** was the most prompt-sensitive model, with a cumulative +17.4 percentage point improvement in classification accuracy from the simple to visual ontology condition.
- **Claude Opus 4.6** showed perception-level gains from prompt enrichment but suffered a 21.3% structured-output failure rate under the visual ontology condition, highlighting a tradeoff between prompt complexity and operational reliability.

## Repository Structure

```
dataset/                    # 300-image balanced RarePlanes subset
  annotations.json          # COCO-format annotations (709 objects)
  balanced_dataset_manifest.csv  # Per-image class labels and attributes
  *.png                     # 512x512 satellite image tiles

experiment-results/         # Model evaluation results (3 models x 3 conditions)
  {model}_{condition}.csv   # Per-image predictions, accuracies, costs, and raw LLM responses

prompts/                    # Full prompt text for each experimental condition
  simple.md                 # Condition 1: Simple global task-scoped instruction
  gsd-text.md               # Condition 2: GSD-augmented prompt with size-bracket guidance
  gsd-images.md             # Condition 3: Visual ontology prompt with per-class reference images
  reference_images/         # Per-class reference image chips used by gsd-images.md
    105_104001003108D900_tile_47.png   # Class 0: Small Civil Transport/Utility
    106_104001003D8DB300_tile_99.png   # Class 1: Medium Civil Transport/Utility
    31_10400100443CFD00_tile_817.png   # Class 2: Large Civil Transport/Utility
    55_1040010049CD5600_tile_308.png   # Class 3: Military Transport/Utility/AWAC
    128_104001004215BF00_tile_1888.png # Class 4: Military Fighter/Interceptor/Attack
    annotated_ground_truth/   # Same tiles with RarePlanes GeoJSON role labels overlaid
    annotated_prompt_claimed/ # Same tiles with the class labels asserted in gsd-images.md

charts/                     # Confusion matrix figures
```

The five reference image chips in `prompts/reference_images/` are the per-class visual exemplars attached to the Visual Ontology Prompt (Condition 3). Four come from the RarePlanes `real/test/PS-RGB_tiled/` split and one (Class 4 Military Fighter) comes from `real/train/PS-RGB_tiled/` — the latter because there is no clean single-class non-eval-overlap fighter tile in the test split. None of the five reference tiles is also present in the 300-image evaluation subset under `dataset/`. The per-chip bounding-box metadata and class-to-tile mapping are recorded inline in `prompts/gsd-images.md`.

Two annotated renderings of each reference chip are provided for inspection:

- `annotated_ground_truth/` — bounding boxes drawn from the authoritative RarePlanes GeoJSON polygons, labeled with the GeoJSON `role` field.
- `annotated_prompt_claimed/` — identical boxes labeled with the class name asserted by `prompts/gsd-images.md` (the label the MLLM was shown).

Rendering is produced by `scripts/render_reference_annotations.py`, which projects GeoJSON WGS84 polygons into pixel space using each tile's GDAL `.aux.xml` GeoTransform sidecar and also copies source PNGs into `reference_images/`. Comparing the two annotated subfolders side-by-side makes any disagreement between RarePlanes ground truth and the prompt's claimed exemplar class visible; for the current reference set, the two renderings match for all five tiles.

## Dataset Attribution

This dataset is derived from the RarePlanes dataset created by CosmiQ Works/In-Q-Tel, available at https://www.iqt.org/library/the-rareplanes-dataset. The original dataset and underlying imagery are licensed under Creative Commons Attribution 4.0 International (CC BY 4.0). This derived subset and evaluation results are also released under CC BY 4.0. Original citation: Shermeyer, J., Hossler, T., Van Etten, A., Hogan, D., Lewis, R., and Kim, D., "RarePlanes: Synthetic Data Takes Flight," WACV 2021.
