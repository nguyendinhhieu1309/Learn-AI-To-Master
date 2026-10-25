# Computer Vision Worked Lab: Images, Labels, Metrics, and Failure Analysis

Computer vision is not only a model architecture problem. Image decoding, color space, resizing, label transforms, patient or scene leakage, and task-specific evaluation can change the result more than a deeper backbone.

## Learning objectives

- Design a reproducible image-data pipeline.
- Keep images and geometric labels synchronized through transforms.
- Select metrics for classification, detection, and segmentation.
- Analyze visual errors and compare against a simple baseline.

## Prerequisites

Complete the [image pipeline lesson](05-image-pipelines-and-vision-training-workflow.md) and [detection and segmentation lesson](04-detection-segmentation-and-error-analysis.md).

## 1. Write the image contract

Record the following before training:

- one image represents what unit: frame, patient, product, camera, or scene;
- channel order, numeric range, color space, and image orientation;
- image width and height conventions;
- annotation format and whether coordinates are absolute or normalized;
- the event or entity used for train/validation/test grouping;
- the target metric and the deployment operating point.

A file that loads successfully can still be wrong: EXIF rotation, BGR/RGB confusion, inconsistent units, or a mask resized with a smoothing interpolator can corrupt labels without raising an error.

## 2. Transform images and labels together

If an image is horizontally flipped, its bounding boxes and segmentation masks must be flipped too. If a crop removes pixels, boxes must be clipped or removed according to a documented rule. A transform that changes the image but not its labels creates false supervision.

For a horizontal flip of an image with width $W$ using continuous image-edge coordinates, a box $(x_{\min},y_{\min},x_{\max},y_{\max})$ maps to:

$$
(W-x_{\max},\,y_{\min},\,W-x_{\min},\,y_{\max}).
$$

Pixel-index conventions may use $W-1$ instead. The important requirement is consistency with the annotation convention.

Use smooth interpolation for natural-image intensities when appropriate. Use nearest-neighbor interpolation for categorical masks so that class IDs are not replaced by fractional values.

## 3. Split at the true independent unit

If frames come from the same video, split by video or capture session. If images come from the same patient, split by patient. If the goal is future performance, split by time. Randomly splitting nearby frames can produce nearly duplicated scenes in train and test.

Inspect:

- filename and hash duplicates;
- near-duplicate images;
- class counts by split;
- source camera, site, patient, video, or scene overlap;
- annotation density and empty-label frequency.

## 4. Metrics answer different questions

### Classification

Accuracy is useful when classes and error costs are balanced. For imbalanced data, also report per-class precision/recall, confusion matrix, and the threshold used. If scores drive decisions, evaluate probability calibration.

### Object detection

For predicted and reference boxes $A$ and $B$:

$$
\operatorname{IoU}(A,B)=\frac{|A\cap B|}{|A\cup B|}.
$$

If box A covers $[0,10]\times[0,10]$ and box B covers $[5,15]\times[0,10]$, the intersection area is 50, the union area is $100+100-50=150$, and IoU is $1/3$.

Average precision summarizes a precision-recall curve after matching predictions to ground-truth objects. State the IoU matching rule, class averaging, confidence treatment, and whether evaluation uses a single threshold or a range. A detection count without matching rules is not reproducible.

Non-maximum suppression removes overlapping lower-scored boxes under a selected overlap threshold. It can reduce duplicate detections but may suppress nearby valid objects. The threshold is part of the evaluated system.

### Segmentation

For one class, Dice and IoU are:

$$
\operatorname{Dice}=\frac{2|P\cap G|}{|P|+|G|},
\qquad
\operatorname{IoU}=\frac{|P\cap G|}{|P\cup G|}.
$$

Dice emphasizes overlap and can look high for large regions while small structures are missed. Report class-wise scores and explain how empty masks are handled.

## 5. Baselines and transfer learning

Start with a simple, reproducible baseline:

1. verify label alignment with visual overlays;
2. train a small model or use a frozen pretrained representation;
3. establish validation metrics and failure categories;
4. fine-tune only after the input pipeline and split are trusted.

Pretrained weights introduce assumptions about image domain, color channels, resolution, and normalization. Record the checkpoint source and preprocessing contract. Do not compare a pretrained model with a baseline that receives different evaluation images.

## 6. Error-analysis worksheet

For each error, save:

- image identifier and split;
- ground truth and prediction overlay;
- confidence or score;
- error type: missed object, duplicate, wrong class, boundary error, domain shift, corrupt label;
- whether a human reviewer agrees with the annotation;
- whether the issue is fixable by data, model, threshold, or product design.

Review examples across the whole validation set and by important slices. Looking only at the most dramatic failures creates selection bias.

## Practice

1. A segmentation mask uses IDs 0, 1, and 2. Why should a smooth resize be avoided for the mask?
2. Images are extracted every tenth frame from 20 videos. The target is future-video performance. How should you split?
3. A detector's precision improves after raising its confidence threshold while recall drops. Is the model necessarily better?
4. A classifier has high overall accuracy but misses nearly every positive case. Which metrics reveal the issue?
5. A pretrained model and a baseline use different image normalization. Why is the comparison invalid?

## Answer checks

1. Interpolation can create fractional or unintended class IDs. Use a label-preserving interpolation method.
2. Split by video, with all frames from one video assigned to only one partition; use a time-based protocol if future periods are the claim.
3. No. The operating point changed. Compare the full precision-recall behavior and select a threshold from the intended costs.
4. Positive-class recall, precision, confusion matrix, and the class support.
5. The input distribution differs, so measured performance mixes model quality with preprocessing differences.

## Mini-project

Train or adapt an image classifier on a small, licensed dataset. Include a group-aware split, a simple baseline, a visual label audit, a fixed validation protocol, per-class metrics, 12 reviewed error examples, and an inference contract. If you choose detection or segmentation, explain its matching or mask-empty rules.

## Common mistakes

- Randomly splitting neighboring video frames.
- Transforming images without transforming boxes or masks.
- Reporting only aggregate accuracy or mAP without protocol details.
- Treating pretrained normalization as an implementation detail.
- Fixing difficult labels after viewing test performance.
