# Image Pipelines and Vision Training Workflow

Vision models can fail because of image decoding, coordinates, color channels, or split design long before architecture becomes the limiting factor.

## 1. Image representation

An image tensor usually has height, width, and channel axes. Libraries differ between HWC and CHW layout and between RGB and BGR channel order. Pixel values may be integers in $[0,255]$ or floats in $[0,1]$. Normalization constants must match the pretrained model or training configuration.

Inspect decoded images after every transform. Check orientation metadata, alpha channels, grayscale handling, corrupt files, and label-to-filename mapping. Save a small visual audit set with boxes, masks, or keypoints drawn on top.

## 2. Resize, crop, and coordinate transforms

Resizing changes pixel coordinates and object scale. Cropping changes the origin and may truncate objects. Letterboxing preserves aspect ratio but introduces padding. For boxes, apply the same geometric transform to all four corners and then recompute bounds. For masks, use nearest-neighbor interpolation.

Document whether normalized coordinates use image width/height or width-minus-one/height-minus-one, and whether box maxima are inclusive or exclusive. A one-pixel convention mismatch can significantly affect small-object evaluation.

## 3. Data splitting for vision

Split by the independent source: patient, video, scene, camera, acquisition session, or geographic site. Adjacent video frames and multiple crops from one source are correlated. Keep all related material in one partition. For temporal deployment, train on earlier data and validate/test on later periods.

## 4. Augmentation with a physical meaning

Horizontal flips, crops, color jitter, blur, and noise can improve robustness if they represent plausible deployment variation. Avoid transformations that change semantic meaning: flipping traffic signs, medical laterality, text, or robot direction can create invalid training examples. For detection, track boxes that become too small or disappear after a crop.

## 5. Transfer learning workflow

Start with a pretrained backbone when the task and image domain are reasonably related. Replace the task head, freeze the backbone briefly if useful, then fine-tune with a lower learning rate. Ensure normalization layers use the intended mode. Compare against training from scratch only when enough data and compute justify it.

## 6. Evaluate with error slices

Report the benchmark metric and its exact configuration, then inspect examples by object size, occlusion, lighting, camera, geography, and class. Measure end-to-end latency at target resolution. A model that meets mAP offline may still fail on edge hardware due to memory, frame-rate, or thermal limits.

## 7. Reproducible image experiments

Store dataset version, file hashes or stable IDs, split IDs, resize policy, augmentation config, class map, normalization, model checkpoint, and evaluation settings. Visualize the same examples across model versions. Keep test data locked during architecture and threshold tuning.

## 8. Exercises

1. A pretrained model expects RGB CHW floats but receives BGR HWC bytes. What symptoms and checks would you expect?
2. Why should adjacent frames from the same video not be randomly distributed across train and test?
3. A horizontal flip improves validation accuracy on handwritten text recognition. What potential issue should be checked before accepting the result?
4. Why should a mask be resized with nearest-neighbor interpolation?

## References

- [Torchvision transforms](https://pytorch.org/vision/stable/transforms.html)
- [OpenCV image processing](https://docs.opencv.org/4.x/d2/d96/tutorial_py_table_of_contents_imgproc.html)
