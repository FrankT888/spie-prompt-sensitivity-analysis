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

charts/                     # Confusion matrix figures
```

## Dataset Attribution

This dataset is derived from the RarePlanes dataset created by CosmiQ Works/In-Q-Tel, available at https://www.iqt.org/library/the-rareplanes-dataset. The original dataset and underlying imagery are licensed under Creative Commons Attribution 4.0 International (CC BY 4.0). This derived subset and evaluation results are also released under CC BY 4.0. Original citation: Shermeyer, J., Hossler, T., Van Etten, A., Hogan, D., Lewis, R., and Kim, D., "RarePlanes: Synthetic Data Takes Flight," WACV 2021.
