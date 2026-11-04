# Dataset

[← Back to README](../README.md)

## Source

The images come from the Kaggle **[New Plant Diseases Dataset](https://www.kaggle.com/vipoooool/new-plant-diseases-dataset)**, an augmented version of the PlantVillage dataset. The full dataset has about 87,000 RGB leaf images in 38 crop/disease classes, split 80/20 into training and validation sets, plus 33 test images.

This project uses a **two-class subset** of 593 images, included in this repository as `training-20191201T155348Z-001.zip` (about 10.5 MB).

| Class | Label in `training.csv` | Images | Notes |
|-------|:-----------------------:|-------:|-------|
| Healthy | `0` | 327 | File names contain `RS_HL` / `GH_HL` (PlantVillage healthy) |
| Diseased | `1` | 266 | File names contain `GCREC_Bact.Sp` / `UF.GRC_BS_Lab` (PlantVillage bacterial spot) |

The file-name codes match the PlantVillage **tomato** healthy and bacterial-spot classes. Some images are augmented copies of the same leaf, with suffixes such as `_flipTB`, `_180deg`, `_new30degFlipLR` and `_newPixel25`.

## Folder layout after extraction

```
training/
├── training.csv          # pre-computed features (see below)
├── healthy/              # 327 .JPG images
└── diseased/             # 266 .JPG images
```

## `training.csv` format

The file has no header row and one row per image. Each row ends with a trailing comma:

```
label,feature1,feature2,feature3,feature4,
0.000000,215.540000,-214.540000,14.402668,0.382980,
```

| Column | Meaning |
|--------|---------|
| `label` | `0` = healthy, `1` = diseased |
| `feature1` | Green-pixel measure (see [known issue #1](known-issues.md#1-feature-1-is-not-a-ratio) for why it is not between 0 and 1) |
| `feature2` | `1 − feature1` |
| `feature3` | Total contour perimeter ÷ 150 |
| `feature4` | Summed GLCM contrast ÷ 20 000 |
| *(empty)* | Created by the trailing comma; dropped when the CSV is loaded |

Average feature values per class in the provided CSV:

| Class | feature3 (perimeter) | feature4 (contrast) |
|-------|---------------------:|--------------------:|
| Healthy (0) | 25.1 | 0.41 |
| Diseased (1) | 12.9 | 0.05 |

Feature 4 separates the classes most strongly. About 200 of the 327 healthy images have contrast above 0.3, compared with none of the diseased images.

## Using a different dataset

Any folder of images works if it is organised as one sub-folder per class:

```
my_data/
├── class_a/
└── class_b/
```

The notebook gives the sub-folders labels `0, 1, …` **in the order `os.listdir` returns them**. That order is not guaranteed to be alphabetical, so check the label printed for each folder. 
