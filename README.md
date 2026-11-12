# STVMGeo

**Cross-modal UAV localization with topological–semantic maps and visual representations**

[English](README.md) | [简体中文](README.zh-CN.md)

> **Release status:** This page provides a preview of the STVMGeo project. The source code will be released after the associated paper is accepted. Installation instructions, model downloads, and reproducible experiment configurations will accompany the code release.

## Overview

STVMGeo estimates a UAV's **2D position and heading** by matching an RGB query image against a local OpenStreetMap (OSM) representation. It combines raster-map features, an OSM-derived topological–semantic graph, and dense visual features from a frozen DINOv3 encoder.

The method operates within a supplied local map or a search region defined by a coarse location prior. Ground-truth poses are used for training and evaluation, but are not required as inference inputs.

The implementation builds on VecMapLocNet and extends its image-to-map matching pipeline with graph-based map representations and learned visual–semantic matching.

## Localization examples

![STVMGeo localization examples with image-to-map overlays, ground-truth poses, and predicted poses](docs/assets/stvmgeo-localization.png)

**Selected localization examples on UAV2MAP.** Each panel overlays a UAV image on its local map using the estimated pose. Black arrows indicate ground-truth poses; colored arrows indicate STVMGeo predictions. The numbers report position error in meters and heading error in degrees.

The four rows contain five examples each, grouped into mutually exclusive accuracy categories:

| Row | Position and heading error |
| :-- | :-- |
| a | Both ≤ 3 m and ≤ 3° |
| b | Both ≤ 5 m and ≤ 5°, excluding row a |
| c | Both ≤ 10 m and ≤ 10°, excluding rows a and b |
| d | Position error > 10 m or heading error > 10° |

These selected cases illustrate successful and unsuccessful estimates; their proportions do not represent dataset-level recall. The overlays use an estimated display scale and feathered image boundaries for qualitative inspection, rather than calibrated orthorectification. Map data © [OpenStreetMap contributors](https://www.openstreetmap.org/copyright); UAV imagery is from UAV2MAP.

## Method

1. **Encode the UAV image.** Extract RGB matching features and dense DINOv3 features from the query image.
2. **Represent the local map.** Encode the OSM raster with a CNN and construct a topological–semantic graph from road connectivity, road attributes, and associated point-of-interest and area information.
3. **Fuse map representations.** Apply local message passing and global attention to the graph, interpolate node features onto a dense grid using inverse-distance weighting (IDW), and fuse them with raster features through a residual connection.
4. **Learn cross-modal matching features.** Use the visual–semantic matching module to produce query and map embeddings. The final STVMGeo configuration includes additional hard-negative fine-tuning of the matching module.
5. **Estimate the pose.** Evaluate candidate positions and headings and decode the pose from the maximum joint matching score. Position and heading are taken from the same hypothesis.

### DINOv3 and semantic predictions

The main configuration uses the general-purpose **DINOv3 ViT-L/16 (LVD-1689M)** backbone as a frozen dense feature extractor. It remains part of inference; cached features can be reused only when they correspond to the same image and backbone configuration.

Semantic predictions shown in the visualizations are produced by the learned matching head. DINOv3 itself is not treated as an off-the-shelf semantic segmentation model. Mask2Former and SAM2 belong to comparison or auxiliary visualization configurations and are not required by the main online DINOv3 inference path.

## Dataset and map preparation

Experiments use **UAV2MAP**, introduced with [VecMapLocNet](https://doi.org/10.1016/j.isprsjprs.2025.04.009). The original data provide UAV RGB images, OSM map rasters, pose annotations, and map resolution information.

This project additionally parses the corresponding OSM vector data to construct topological–semantic graphs. It also prepares feature and semantic caches for the relevant training and comparison configurations. **These graphs and caches are project-generated extensions, not original UAV2MAP fields.**

The public release is planned to include preparation scripts and data-format documentation. The original dataset and third-party model weights should be obtained from their respective providers; this project does not bundle them in the preview.

## Experimental configurations

The following names distinguish the baseline and the configurations used in the study:

| Configuration | Description |
| :-- | :-- |
| VecMapLocNet | Reference image-to-map localization baseline |
| B (RGB + STG) | RGB baseline extended with the topological–semantic graph (STG) branch |
| B + DINOv2 | B with DINOv2-based visual features |
| B + Mask2Former | B with Mask2Former-based semantic information |
| B + SAM2 | B with SAM2-derived mask information |
| B + DINOv3-SAT | B with the satellite-pretrained DINOv3 backbone |
| B + DINOv3-GEN | B with the general-purpose DINOv3 backbone, before the final hard-negative fine-tuning stage |
| STVMGeo | Final online DINOv3 configuration with the learned matching module and additional hard-negative fine-tuning |

The release will document the checkpoint, preprocessing, and evaluation settings for each configuration. Configuration names alone do not imply identical training budgets.

## Evaluation

Evaluation uses paired UAV queries and local maps, with position and heading errors computed from the same decoded joint pose. The reported analyses cover:

- **Position recall:** thresholds of 1, 3, 5, and 10 m, including longitudinal and lateral error analyses.
- **Heading recall:** thresholds of 1°, 3°, 5°, and 10°.
- **Joint recall:** both position and heading satisfy (3 m, 3°), (5 m, 5°), or (10 m, 10°).
- **Error distributions:** median, interquartile range, and percentile summaries.
- **Large-error fractions:** position errors above 50/100 m and heading errors above 30°/50°.
- **Efficiency:** per-module timing and parameter counts, with the measured scope specified separately from end-to-end latency.

Detailed quantitative results and the exact evaluation protocol will accompany the paper and code release.

## Planned release

The release after paper acceptance is planned to include:

- [ ] Model implementation and inference entry point.
- [ ] OSM graph construction and dataset preparation tools.
- [ ] Training and evaluation configurations.
- [ ] Project checkpoints or download instructions, subject to upstream model terms.
- [ ] Environment setup and reproducible usage examples.
- [ ] Visualization scripts and experiment metadata.

The current page is a project preview, not an installation guide. Runnable commands will be added after the release package has been validated.

## Citation

Paper details and a BibTeX entry will be added when available.

## Acknowledgments

We acknowledge the authors and contributors of:

- [VecMapLocNet and UAV2MAP](https://doi.org/10.1016/j.isprsjprs.2025.04.009) for the baseline and dataset.
- [DINOv3](https://github.com/facebookresearch/dinov3) for dense visual representations.
- [GraphGPS](https://github.com/rampasek/GraphGPS) for the local–global graph encoder design used as a basis for the map graph branch.
- [OpenStreetMap](https://www.openstreetmap.org/copyright) for map data.

Third-party datasets, code, and pretrained models remain subject to their respective terms. The license for the released project code will be specified with the code release.
