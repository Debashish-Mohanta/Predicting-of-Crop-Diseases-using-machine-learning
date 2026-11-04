# Methodology

[← Back to README](../README.md)

## Objective

Classify a leaf as **healthy** or **diseased** from its image, using a small set of visual features and a classical machine-learning classifier. The project does not use a deep network.

## Pipeline

```
┌──────────────┐   ┌──────────────┐   ┌────────────────────┐   ┌─────────────┐   ┌────────────┐
│ Image        │──►│ Image        │──►│ Feature extraction │──►│ Linear SVM  │──►│ Prediction │
│ preprocessing│   │ segmentation │   │ (4 features → CSV) │   │ classifier  │   │ 0 / 1      │
└──────────────┘   └──────────────┘   └────────────────────┘   └─────────────┘   └────────────┘
```

All of this is in the notebook [`MT19211_MT19214.ipynb`](../MT19211_MT19214.ipynb).

### Step 1: Load the image

`cv2.imread()` loads each image as a NumPy array in **BGR** channel order (OpenCV's default).

### Step 2: Convert colour spaces

- **BGR → HSV** (`cv2.COLOR_BGR2HSV`). In HSV, *hue* carries the colour on its own, separate from brightness. That makes it much easier to isolate "green" than in BGR, where colour and luminance are mixed together.
- **BGR → Grayscale** (`cv2.COLOR_BGR2GRAY`), used for the shape and texture features.

### Step 3: Feature extraction

The function `feature_extraction(file_path, image_files, mark, output_file)` computes four features for every image in a folder. It appends one row per image to a CSV file in the form `label, f1, f2, f3, f4,`.

#### Feature 1: Green region

```python
mask = cv2.inRange(image_hsv, (30, 0, 0), (70, 255, 255))
ratio_of_green = cv2.countNonZero(mask) / (image.size / 3)
feature1 = np.round(ratio_of_green, 2)
```

The mask keeps pixels with hue between 30 and 70 (OpenCV hue runs 0–179, so this is roughly the green band). Disease shows up as yellow, brown or black lesions that fall outside this band, so diseased leaves tend to have **less** green.


#### Feature 2: Non-green region

```python
feature2 = 1 - feature1
```

The complement of feature 1. Because it is a linear function of feature 1, it gives the classifier no extra information.

#### Feature 3: Leaf perimeter

```python
gray = cv2.GaussianBlur(image_gray, (3, 3), 0)
_, biny = cv2.threshold(gray, 0, 255, cv2.THRESH_OTSU | cv2.THRESH_BINARY_INV)
contours, _ = cv2.findContours(biny, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
feature3 = sum(cv2.arcLength(c, True) for c in contours) / 150
```

1. A 3×3 Gaussian blur removes pixel noise.
2. **Otsu's method** chooses the threshold automatically and splits the leaf (and any dark spots) from the background.
3. The code finds the external contours and adds up their closed lengths.

A damaged or spotty leaf produces a more irregular boundary and extra small contours, which changes the total perimeter. The value is divided by 150 to scale it down.

#### Feature 4: Texture contrast (GLCM)

```python
glcm = greycomatrix(image_gray, [1, 2], [0, np.pi/2, 3*np.pi/4])
contrast = greycoprops(glcm, "contrast")    # shape (2 distances, 3 angles)
feature4 = contrast.sum() / 20000
```

A **Grey-Level Co-occurrence Matrix** counts how often pairs of grey levels occur at a given distance and angle. Its *contrast* property is large when neighbouring pixels differ a lot. The feature adds up contrast over 2 distances × 3 angles, which captures how textured the image is.

### Step 4: Build the feature matrix

`label_featurematrix(input_file)` reads the CSV and names its columns `label, feature1 … feature4`. It drops the empty trailing column created by the trailing comma. It returns:

- the full DataFrame
- the label Series
- the feature DataFrame (4 columns)

`plot_graph_features(input_file, x, y)` draws a scatter plot of any two features, coloured by class.

### Step 5: Train/test split

```python
train_test_split(features_data, label, test_size=0.2)
```

The split is 80/20, which gives **474 training** and **119 test** samples.

### Step 6: Classifier, a Support Vector Machine

```python
svm.SVC(kernel='linear', C=1.0, probability=True)
```

- **Linear kernel**: the SVM finds the hyperplane in the 4-D feature space with the widest margin between the two classes.
- **C = 1.0**: the default regularisation, which balances a wide margin against misclassified training points.
- **probability=True**: turns on Platt scaling, so `predict_proba` is available for the ROC curve.

Training a kernel SVM costs between **O(n²)** and **O(n³)** in the number of samples. That is not a concern for 474 samples.

> The project report also shows pseudo-code for a *local SVM (kSVM)*, which clusters the data with k-means and trains one SVM per cluster. The notebook does **not** implement it; kSVM is listed in the report as background and a possible extension.

### Step 7: Evaluation

- Accuracy (`classifier.score`, `accuracy_score`)
- Confusion matrix (`confusion_matrix`)
- ROC curve (`roc_curve`, using `predict_proba`)

Numbers and plots are in [results.md](results.md).
