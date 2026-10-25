# Images and Classical Vision

## Learning objectives

Read image arrays, understand how simple filters change them, and use classical vision operations before reaching for a learned model.

## 1. How a digital image is represented

A grayscale image is a 2D array with shape `(height, width)`. A color image is often `(height, width, channels)`, with 3 channels (RGB or BGR) or 4 (RGBA). Values may be integers from 0–255 or floats in `[0,1]`. Always check the library's color order; OpenCV commonly loads BGR while many plotting tools expect RGB.

Image coordinates usually use `x` across columns and `y` down rows. Array indexing is often `[row, column] = [y, x]`. This convention is a frequent source of transposed coordinates.

## 2. Color and intensity

RGB represents additive color channels. HSV separates hue, saturation, and value; it can make thresholding by color more intuitive under some conditions. Grayscale conversion combines channels, but it discards color information. Normalize or resize images consistently between training and inference.

## 3. Filters and edges

A convolution filter computes a weighted neighborhood around each pixel. A blur averages nearby values and reduces high-frequency detail/noise. Edge filters respond to rapid intensity changes. Thresholding turns a grayscale image into a binary mask. Morphological erosion shrinks foreground regions; dilation expands them; opening/closing combine these operations to remove small noise or fill gaps.

These methods are fast and interpretable, but fixed thresholds can fail under changing lighting, exposure, or camera conditions.

## 4. Geometric transformations

Resize, crop, rotate, and perspective transforms change pixel coordinates. When transforming an image for a supervised task, transform its label too: boxes and masks must remain aligned. For classification, random crops or flips are useful augmentation only if those changes preserve the label (a left/right flip may invalidate a direction label).

## Mini example

For a 640-by-480 image, the NumPy array shape is often `(480, 640, 3)`. Pixel `[10, 20]` means row 10, column 20. Its displayed coordinate is typically `(x=20, y=10)`. If a box is recorded as `(x_min, y_min, x_max, y_max)`, do not index it as if it were `(row_min, col_min, ...)` without converting.

## Small OpenCV example

```python
import cv2

image_bgr = cv2.imread("image.jpg")
if image_bgr is None:
    raise FileNotFoundError("Could not read image.jpg")

gray = cv2.cvtColor(image_bgr, cv2.COLOR_BGR2GRAY)
blurred = cv2.GaussianBlur(gray, (5, 5), 0)
edges = cv2.Canny(blurred, threshold1=60, threshold2=140)
```

The thresholds are example settings, not universal values. If the output is poor, inspect the grayscale and blurred images and tune using representative lighting conditions.

## Practice

Take an image, display each channel, grayscale version, blurred version, and edge map. Write down one detail that disappears and one that becomes clearer in each transformation.

## Common mistakes

- Mixing BGR and RGB channels.
- Confusing image width/height with array row/column order.
- Resizing images but forgetting to scale bounding-box coordinates.
- Applying augmentation that changes the meaning of the label.
