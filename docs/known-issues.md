# Known Issues & Future Work

[← Back to README](../README.md)

These issues were found while documenting the original 2019 notebook. They are recorded here so results can be read correctly and the project can be improved. The notebook itself has not been changed.

## Known issues

### 1. Feature 1 is not a ratio

```python
image = np.empty(len(image_files), dtype=object)   # array of images
...
ratio_of_green = cv2.countNonZero(mask) / (image.size / 3)
```

`image.size` is the **number of images in the folder**, not the number of pixels in the current image. Feature 1 is therefore `green_pixels × 3 / n_images`, which explains why it ranges from about 100 to 650 instead of 0 to 1. It also makes **feature 2 negative**, and both features change if the number of images in a folder changes.

**Fix:**
```python
img = image[n]
ratio_of_green = cv2.countNonZero(mask) / (img.shape[0] * img.shape[1])
```

### 2. Labels depend on folder listing order

```python
mark = 0
for i in os.listdir(path):
    ...
    mark += 1
```

Each class folder gets its label from the order `os.listdir` returns, and that order is not guaranteed. In the saved notebook run, `diseased` got label 0. The provided `training.csv` uses `healthy = 0`. `plot_graph_features` always assumes `0 = healthy`, so the plot legend can be wrong. Accuracy is not affected.

**Fix:** set the labels explicitly, for example `labels = {"healthy": 0, "diseased": 1}`.

### 3. Re-running feature extraction corrupts the CSV

- The CSV is opened in **append** mode (`"a"`), so a second run duplicates every row.
- `os.listdir(path)` also returns `training.csv`, and the loop then tries to list it as a folder of images.

**Fix:** write to a fresh file, and only iterate over sub-directories (`if os.path.isdir(...)`).

### 4. ROC curve probability column

```python
probab = classifi.predict_proba(feature_test)[:, 0]   # P(class 0)
roc_curve(label_test, probab)                          # default pos_label = 1
```

`roc_curve` treats class **1** as positive, but the score passed in is the probability of class **0**. The standard form is `predict_proba(...)[:, 1]`. Check this before quoting an AUC.

### 5. Results are not reproducible

`train_test_split` and `SVC(probability=True)` are called without `random_state`, so accuracy and the confusion matrix change on every run (94.1% in the notebook, 96.6% in the report).

### 6. Hard-coded absolute paths

The notebook uses `os.chdir("E://Mtech programs/...")`. See [usage.md](usage.md#4-fix-the-paths-in-the-notebook).

### 7. Augmented duplicates can leak across the split

The dataset contains rotated and flipped copies of the same leaf (`_flipTB`, `_180deg`, …). With a random split, copies of one leaf can land in both the training and the test set, which can inflate test accuracy. Splitting by original leaf ID would give a more honest estimate.

### 8. Features are not scaled

Feature 1 (hundreds) and feature 4 (below 1) are on very different scales, and SVMs are sensitive to scale. Adding a `StandardScaler` in a `Pipeline` is recommended.

## Future work

- Fix issues 1–5 above and re-report the metrics with cross-validation.
- Add per-class **precision, recall, F1** and **ROC-AUC**.
- Try an **RBF kernel** and tune `C`/`gamma` with `GridSearchCV`.
- Implement the **local SVM (kSVM)** described in the report: k-means clustering, then one SVM per cluster.
- Add richer features, such as colour histograms, more GLCM properties (homogeneity, energy, correlation), and lesion count and area.
- Extend to **multi-class** classification of disease types (fungal / bacterial / viral) using more of the 38 PlantVillage classes.
- Compare against a CNN baseline (for example, transfer learning with MobileNet or ResNet).
- Move the notebook logic into a reusable Python module with a `predict(image_path)` function.

dataset.md
See [known issue #2](known-issues.md#2-labels-depend-on-folder-listing-order).

- [Known issues & future work](docs/known-issues.md): limitations found in the code, and ideas for extending it

methodology.md
> ⚠️ The divisor is meant to be the pixel count, but `image` here is the *array of images*, not the current image. See [known issues](known-issues.md#1-feature-1-is-not-a-ratio). As a result, the stored values are in the hundreds rather than between 0 and 1.

results.md


> See [known issue #4](known-issues.md#4-roc-curve-probability-column) about which probability column feeds the curve. The AUC is not computed in the notebook; `roc_auc_score` is imported but never called.



([known issue #2](known-issues.md#2-labels-depend-on-folder-listing-order))

usage.md

   The plotting function assumes `0 = healthy`. If your output says otherwise, the plot colours are swapped (see [known issue #2](../docs/known-issues.md#2-labels-depend-on-folder-listing-order)). The accuracy is unaffected.