# slam-track-fusion

Real-time visual SLAM and multi-object tracking fused into a single coordinate frame, benchmarked on TUM RGB-D.

## Overview

This project combines two perception pipelines that are usually built and evaluated separately:

- **Visual SLAM** — estimates the camera's trajectory and builds a 3D map from RGB-D frames, with no prior knowledge of the scene.
- **Object detection + tracking** — detects objects per frame and follows them across frames with a persistent ID.

The core contribution is the **fusion step**: projecting each tracked object's position into the SLAM map's coordinate frame using the camera pose at that instant, so the final output is one coherent map showing both where the camera went and where the moving objects it saw went, relative to each other.

This closes a recurring skill gap identified across ~68 computer vision / ML engineering job descriptions: 3D point clouds, SLAM, object tracking, and real-time processing.

## Architecture

```
TUM RGB-D dataset (RGB + depth frames)
        |
        +----------------------+----------------------+
        |                                              |
   Visual SLAM                              Detection + tracking
   (trajectory + 3D map)                    (YOLO + ByteTrack)
        |                                              |
        +----------------------+----------------------+
                               |
                            Fusion
              (project tracks into map frame)
                               |
                        Annotated map
              (trajectory + object tracks)
                               |
                          Evaluation
                (ATE vs ground truth, FPS)
```

## Pipeline stages

1. **SLAM** — feature extraction, feature matching, pose estimation, mapping, and loop closure over TUM RGB-D sequences.
2. **Detection + tracking** — YOLOv8/YOLO11 detections passed through ByteTrack to assign persistent track IDs.
3. **Fusion** — each tracked object's pixel position is un-projected to 3D using depth and camera intrinsics, then transformed into the map frame using the SLAM camera pose for that frame.
4. **Evaluation** — SLAM accuracy (ATE, RPE), tracking quality (MOTA/IDF1 where ground truth is available), and real-time performance (FPS, per-frame latency).

## Tech stack

- Python, PyTorch
- pyslam (visual SLAM)
- Ultralytics YOLO + ByteTrack (detection + tracking)
- ONNX Runtime (inference benchmarking)
- TUM RGB-D dataset (`freiburg1_desk`)

## Status

Early stage — dataset and environment setup in progress. This README will be updated as each pipeline stage is completed with results and benchmark numbers.

## Roadmap

- [ ] SLAM trajectory + map on TUM RGB-D, scored with ATE
- [ ] Detection + tracking on video, validated visually and (where applicable) with MOTA/IDF1
- [ ] Fusion of tracked object positions into the SLAM map
- [ ] Real-time benchmarking (FPS, latency, ONNX vs native inference)
- [ ] Optional: port to embedded hardware (Jetson) as a v2 extension

## Author

Ali Rashidi (Farid) — Computer Vision / Deep Learning Engineer
[LinkedIn](https://linkedin.com/in/faridrash) · [Portfolio](https://faridrash.vercel.app)
