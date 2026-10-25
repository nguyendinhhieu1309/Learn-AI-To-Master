# Deep Vision Tasks

## Learning objectives

Distinguish classification, detection, and segmentation; understand the metrics each uses; and plan error analysis around visual labels.

## 1. Classification

Image classification returns one or more category scores for a whole image. A single-label task chooses among mutually exclusive classes; a multi-label task may assign several labels at once. Class balance and label ambiguity matter. Inspect errors by category, image quality, lighting, and acquisition source.

## 2. Object detection

Detection predicts object classes and bounding boxes. A box is often `[x_min, y_min, x_max, y_max]`. **Intersection over Union (IoU)** is

$$
IoU = \frac{\text{area of overlap}}{\text{area of union}}.
$$

An IoU threshold determines whether a predicted box matches a ground-truth box. Non-maximum suppression (NMS) removes duplicate overlapping predictions. Average precision summarizes a precision-recall curve; mean average precision (mAP) averages AP across classes and sometimes across IoU thresholds. Always state the evaluation convention because mAP variants differ.

## 3. Segmentation

Semantic segmentation assigns a class to each pixel. Instance segmentation also distinguishes separate objects of the same class. Mean IoU averages per-class overlap. A high image-level accuracy can coexist with poor boundary quality or failure on small objects, so inspect masks visually and report per-class results.

## 4. Data and labels

Images from the same video, site, or person should usually stay within one split. Otherwise, near-duplicates may leak. Labeling rules should define occlusion, truncation, ambiguous classes, and small objects. Measure annotation agreement where labels are subjective. Document resolution, camera, lighting, and permitted data use.

## 5. Augmentation and deployment

Augmentation can improve robustness when it resembles valid variations: mild brightness, crop, rotation, or noise. It cannot replace representative real evaluation data. At deployment, verify color order, normalization, resizing, and thresholds match training. Measure latency and memory on the target device.

Treat deployment differences as part of the vision problem: rolling shutter, motion blur, low light, compression, glare, and changed camera placement can all shift the input distribution. Measure these conditions explicitly where they matter.

## Practice

For a detector, choose 20 false positives and 20 false negatives. Group them by causes such as tiny object, blur, occlusion, confusing background, or incorrect annotation. Pick one improvement and state how you would measure whether it helped without reusing the test set for tuning.

## Common mistakes

- Comparing detection scores without specifying IoU and class averaging.
- Evaluating near-duplicate video frames in both train and test.
- Reporting only aggregate mAP and ignoring per-class failures.
- Resizing or normalizing differently at inference than during training.
