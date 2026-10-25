# Computer Vision: Worked Problem Set

Practice matching image representations, labels, metrics, and splits to the deployment task.

## 1. Image tensor layout

A color image is 240 high, 320 wide, with three channels. Give channel-last and channel-first shapes for a batch of 16.

### Solution

Channel-last: [16,240,320,3]. Channel-first: [16,3,240,320]. Check the framework's expected layout.

## 2. Resize and aspect ratio

A 640 by 480 image is resized directly to 224 by 224. What changes geometrically?

### Solution

The 4:3 image is distorted to a 1:1 ratio. Alternatives include preserving ratio with padding, resizing then cropping, or letterboxing. Transform boxes and masks consistently, and match training/inference preprocessing.

## 3. Bounding-box intersection over union

Box A spans x=[0,4], y=[0,4]. Box B spans x=[2,6], y=[0,4]. Use continuous coordinates. Find IoU.

### Solution

Each area is 16. Intersection is 2×4=8. Union is 16+16-8=24. IoU is 8/24=1/3.

## 4. Non-maximum suppression

Two same-class boxes have IoU 0.82. Scores are 0.91 and 0.74. NMS threshold is 0.5. Which is retained?

### Solution

Keep the higher-scoring box and suppress the lower-scoring one because overlap exceeds the threshold. Behavior also depends on class handling, score threshold, ties, and hard versus soft NMS.

## 5. Cost-sensitive threshold

False negatives cost ten times as much as false positives. Assuming calibrated probabilities and only these costs, derive the positive decision threshold.

### Solution

Predict positive when C_FP(1-p) < C_FN p, so p > C_FP/(C_FP+C_FN). With C_FN=10 C_FP, threshold is 1/11, about 0.091. Validate calibration and operational constraints.

## 6. Accuracy versus recall

A test set has 980 negatives and 20 positives. A model predicts all cases negative. Find accuracy and positive recall.

### Solution

Accuracy is 98%; positive recall is 0%. Use confusion matrices, per-class metrics, precision-recall analysis, and a task-reviewed operating point.

## 7. Segmentation IoU and Dice

For one class, TP=70, FP=10, FN=20 pixels. Find IoU and Dice.

### Solution

IoU=TP/(TP+FP+FN)=70/100=0.70. Dice=2TP/(2TP+FP+FN)=140/170, about 0.824.

## 8. Mean average precision

What does average precision summarize, and why should a detection mAP result state IoU thresholds and class averaging?

### Solution

Average precision summarizes a precision-recall curve under a defined matching rule. Benchmarks average over classes and IoU thresholds differently. Scores are not comparable unless matching, thresholds, class treatment, and ignored annotations are specified.

## 9. Split by patient, scene, or camera

A dataset contains video frames from 50 locations. Random frame splitting places adjacent frames in train and validation. What is the risk?

### Solution

Near-duplicate frames share backgrounds, lighting, and objects, inflating validation. Group by sequence or location when testing unseen scenes. If deployment targets future frames from known scenes, use a temporal split and state that target.

## 10. Augmentation and labels

A horizontal flip is applied to an image with left/right labels and boxes. Which labels need transformation?

### Solution

Mirror box x coordinates. Transform keypoints and remap directional labels when meaning changes. Object identity may stay fixed; left/right or text-reading labels may not. Augmentation is valid only if the transformed sample preserves the task label.

## 11. Preprocessing leakage

You compute normalization statistics using train and validation images. Why is that leakage?

### Solution

Validation influenced fitted preprocessing. Compute statistics on training data only and freeze them for validation, test, and deployment. Fixed external constants are a separate documented choice.

## 12. Error analysis by object size

A detector has low recall for small objects and strong recall for large ones. What should you inspect?

### Solution

Break down by object size, class, occlusion, source, and distance. Inspect annotations, resize policy, feature resolution, assignment strategy, threshold, and NMS. Change one factor and compare the same slices.

## Review checklist

- Are image and label transforms synchronized?
- Do splits reflect new patients, locations, cameras, or time?
- Are per-class and size-based errors visible?
- Are mAP matching rules documented?
- Does the threshold reflect calibrated risk and operational capacity?
- Are normalization statistics fit on training data only?
