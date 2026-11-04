# Usage

[← Back to README](../README.md)

## 1. Requirements

- Python **3.7–3.9** (the notebook was written with Python 3.7.4)
- Jupyter Notebook or JupyterLab
- The packages in [`requirements.txt`](../requirements.txt)

| Package | Used for |
|---------|----------|
| `opencv-python` (≥ 4) | Image loading, colour conversion, thresholding, contours |
| `scikit-image` (< 0.19) | GLCM texture features (`greycomatrix`, `greycoprops`) |
| `scikit-learn` | SVM, train/test split, metrics |
| `numpy`, `pandas` | Arrays and the feature table |
| `matplotlib` | Scatter plot and ROC curve |

> **Newer scikit-image?** Version 0.19 renamed `greycomatrix`/`greycoprops` to `graycomatrix`/`graycoprops`, and later versions removed the old names. On a newer version, change the import in the first notebook cell to:
> ```python
> from skimage.feature import graycomatrix as greycomatrix, graycoprops as greycoprops
> ```

## 2. Install

```bash
git clone https://github.com/Debashish-Mohanta/Predicting-of-Crop-Diseases-using-machine-learning.git
cd Predicting-of-Crop-Diseases-using-machine-learning

python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate

pip install -r requirements.txt
pip install notebook
```

## 3. Extract the dataset

```bash
python -m zipfile -e training-20191201T155348Z-001.zip .
```

This creates `training/` with `healthy/`, `diseased/` and `training.csv`.

## 4. Fix the paths in the notebook

The notebook was written on the authors' machine, and it changes directory to an absolute path:

```python
os.chdir("E://Mtech programs/ACB/Project/Final_codes/training/")
```

Replace every `E://Mtech programs/ACB/Project/Final_codes/training/` (in **cells 3 and 7**) with the path to **your** extracted `training/` folder. For example:

```python
path = "C:/Users/you/Predicting-of-Crop-Diseases-using-machine-learning/training/"
```

Keep the trailing slash. The code builds file paths as `path + folder_name`.

## 5. Run the notebook

Open the notebook with `jupyter notebook MT19211_MT19214.ipynb`. You can then take one of two routes.

### Option A: Use the provided features (fastest)

`training/training.csv` already contains features for all 593 images.

1. Run **cell 1** (imports) and **cell 2** (function definitions).
2. **Skip cell 3**, the feature-extraction loop.
3. Run the remaining cells in order to plot the features, split the data, train the SVM, evaluate it and draw the ROC curve.

### Option B: Recompute the features from the images

1. **Delete or move `training/training.csv` first.** Two things go wrong otherwise:
   - `feature_extraction` opens the CSV in **append** mode, so new rows are added after the old ones and every image appears twice.
   - Cell 3 lists everything inside `training/`, including the CSV, and fails when it treats the CSV as a folder of images.
2. Run every cell from the top.
3. Check the printed output of cell 3. It shows which label each folder received:
   ```
   'diseased' with label 0 having 266 numbers
   'healthy' with label 1 having 327 numbers
   ```


## 6. Evaluate on a separate test set

The notebook has commented-out code for a separate `test/` folder (cells 3 and 8). To use it:

1. Arrange your test images as `test/healthy/` and `test/diseased/`.
2. Uncomment the test block in cell 3, set its paths, and run it to produce `test.csv`.
3. Uncomment cell 8 to load `test.csv`, then call `classifi.predict(features_data_test)`.

## Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| `FileNotFoundError: 'E://Mtech programs/...'` | Hard-coded path | Change the paths as in step 4 |
| `NotADirectoryError: ... training.csv` | Cell 3 treats the CSV as a folder | Remove `training.csv` before re-extracting (Option B), or skip cell 3 (Option A) |
| `ImportError: cannot import name 'greycomatrix'` | scikit-image ≥ 0.19 | Use the import alias above, or `pip install "scikit-image<0.19"` |
| `ValueError: not enough values to unpack` at `findContours` | OpenCV 3.x returns 3 values | Install OpenCV ≥ 4 |
| `ValueError: Length mismatch` when naming CSV columns | CSV rows have the wrong number of fields | Regenerate the CSV from scratch (Option B) |
| Accuracy differs from run to run | Random train/test split | Pass `random_state=42` to `train_test_split` |
