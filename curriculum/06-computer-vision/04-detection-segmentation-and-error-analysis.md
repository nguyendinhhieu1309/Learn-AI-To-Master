# Detection, Segmentation, and Vision Error Analysis

Computer vision tasks differ in what they predict and how success is measured. Choosing the wrong target format or metric can invalidate a seemingly strong result.

## 1. Task and label definitions

- **Image classification** predicts one or more labels for an image.
- **Object detection** predicts a class and bounding box for each object.
- **Semantic segmentation** assigns a class to every pixel; object instances of the same class are merged.
- **Instance segmentation** assigns a mask to each individual object.
- **Keypoint estimation** predicts structured points such as joints or landmarks.

Write annotation instructions before labeling: class definitions, occlusion rules, minimum object size, truncated objects, ambiguous examples, and how to represent “unknown.” Measure annotator agreement and review disagreements. Inconsistent labels set a ceiling on model quality.

## 2. Bounding boxes and detection metrics

Represent an axis-aligned box as $(x_{min}, y_{min}, x_{max}, y_{max})$ or $(x_c,y_c,w,h)$, and document coordinate conventions. Normalize by image dimensions only if the model and data pipeline expect normalized coordinates.

Intersection over Union (IoU) for predicted box $B_p$ and true box $B_t$:

$$
\operatorname{IoU}(B_p,B_t) = \frac{|B_p \cap B_t|}{|B_p \cup B_t|}
$$

A predicted detection is typically matched to a ground-truth object when class agrees and IoU exceeds a threshold. Matching is one-to-one: duplicate predictions for one object become false positives. Average precision summarizes precision-recall behavior; mAP averages over classes and, in some conventions, multiple IoU thresholds. Always state the benchmark’s precise definition.

## 3. Segmentation metrics

For a class, intersection-over-union is $TP/(TP+FP+FN)$. Mean IoU averages class IoU values; decide whether to include absent classes. Dice/F1 is $2TP/(2TP+FP+FN)$ and is related to IoU by $Dice=2IoU/(1+IoU)$ for the same binary masks.

Pixel accuracy can be dominated by background. Report per-class scores, macro and frequency-weighted averages where useful, and the confusion matrix. For boundary-sensitive tasks, include a boundary metric; two masks with similar IoU can have very different edge quality.

## 4. Data pipeline and augmentation

The image and spatial annotation must receive consistent geometric transforms. If an image is flipped, rotate the box or mask as well; if resized, update coordinates. Use nearest-neighbor interpolation for categorical masks, since bilinear interpolation invents class IDs. Check color order (RGB/BGR), pixel range, orientation metadata, and train/validation preprocessing.

Useful augmentations may include crop, resize, flip, color jitter, blur, noise, and occlusion. Keep them realistic for deployment: horizontal flips are invalid when left/right direction matters, and aggressive crops can remove small objects. Validation and test images should not receive stochastic training augmentation.

## 5. Class imbalance and sampling

Rare classes can be ignored by the optimizer. Options include class-weighted losses, focal loss, balanced sampling, or collecting more rare examples. These methods alter optimization and sometimes calibration; evaluate the full deployment distribution and tune thresholds on validation data.

## 6. Error analysis

Build a review set of false positives, false negatives, poor localization, missed small objects, occlusion, lighting, camera, and site. Categorize failures and estimate how often each category occurs. Create targeted slices and check whether the proposed fix could harm another slice.

Inspect both annotation quality and deployment mismatch. A model failure may be caused by a broken resize, incorrect class index, missing nighttime examples, or the model itself. Visualize predictions overlaid on images at the same resolution used in evaluation.

## 7. Reliable vision evaluation

Split by video, scene, patient, camera, or site when adjacent frames or related images are correlated. A random image split may leak nearly identical frames. Keep the evaluation resolution, suppression settings, confidence threshold, and model checkpoint fixed. Measure latency and memory alongside quality if the target is edge or robotic deployment.

## 8. Exercises

1. A detector has high recall but low precision. What failure patterns and operating settings would you inspect?
2. A segmentation mask is resized with bilinear interpolation and now has fractional class IDs. What went wrong?
3. A video dataset has thousands of adjacent frames. Why is a random frame split optimistic?
4. Overall mIoU improves, but a safety-critical class worsens. How should the result be presented and acted on?

## References

- [COCO evaluation metrics](https://cocodataset.org/#detection-eval)
- [Torchvision detection models](https://pytorch.org/vision/stable/models.html#object-detection)
- [Albumentations: supported targets](https://albumentations.ai/docs/)
