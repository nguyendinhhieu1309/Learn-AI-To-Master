# Video, Tracking, and Deployment

## Learning objectives

Process a video as a time-ordered stream, distinguish detection from tracking, and measure a vision system under its real compute and camera constraints.

## 1. Video is a sequence

A video contains frames with timestamps and a frame rate. Processing every frame costs more than sampling frames. If an object moves quickly, sparse sampling may miss it or make its apparent motion too large. Motion blur, rolling shutter, and dropped frames affect results. Preserve timestamps instead of assuming every frame interval is exact.

## 2. Detection vs. tracking

A detector answers “what objects are in this frame?” A tracker tries to maintain an identity over time. A simple tracking-by-detection pipeline runs a detector per frame, then associates boxes using overlap, appearance, motion prediction, or a combination. Occlusion, crowded scenes, similar objects, camera motion, and missed detections cause identity switches.

Evaluate both detection and temporal identity behavior. Inspect track fragments and ID switches, not just per-frame box scores. Tracking metrics such as IDF1 or HOTA summarize different aspects; state the metric and version used.

## 3. Temporal information

Optical flow estimates apparent pixel motion between frames. It can help detect motion or stabilize video but is sensitive to texture, lighting changes, and large displacements. A temporal model can use multiple frames, but it needs enough representative video and careful split design; frames from the same recording should generally stay in the same split.

## 4. Real-time budgets

Measure capture-to-result latency, not only neural-network inference time. Include image decode, resize, preprocessing, transfer to accelerator, inference, postprocessing, and communication. Report median and tail latency plus dropped-frame rate. A model that averages 20 ms but occasionally takes 400 ms may be unsuitable for a real-time loop.

On constrained hardware, consider smaller input resolution, model quantization, batching trade-offs, or a lighter architecture. Re-evaluate quality after every compression or runtime change.

## 5. Deployment checklist

- Camera and runtime use the same color channel order, normalization, crop, and resize.
- Camera calibration matches the active lens, focus, and resolution for geometric tasks.
- Thresholds are selected with validation data and recorded with the model.
- Performance is checked under expected lighting, blur, occlusion, and camera motion.
- A stale, missing, or low-confidence prediction has a defined fallback.

## Practice

Choose a short video. Compare detection on every frame with detection every fifth frame. Record runtime, missed events, and how association changes. Explain why the faster option may or may not be acceptable for the intended application.

## Common mistakes

- Randomly splitting neighboring frames across training and test sets.
- Treating each frame prediction as an independent track.
- Reporting only average inference time.
- Compressing a model without measuring quality changes on difficult examples.
