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

1. **SLAM** ✅ — feature extraction, feature matching, pose estimation, mapping, and loop closure over TUM RGB-D sequences.
2. **Detection + tracking** — YOLOv8/YOLO11 detections passed through ByteTrack to assign persistent track IDs.
3. **Fusion** — each tracked object's pixel position is un-projected to 3D using depth and camera intrinsics, then transformed into the map frame using the SLAM camera pose for that frame.
4. **Evaluation** — SLAM accuracy (ATE, RPE), tracking quality (MOTA/IDF1 where ground truth is available), and real-time performance (FPS, per-frame latency).

## Results so far

### SLAM (`01_slam.ipynb`)

Run on TUM RGB-D `freiburg1_desk` (573 RGB-D frames, associated via TUM's standard timestamp-matching tool).

- **571 / 573 frames** successfully tracked (one brief tracking loss mid-sequence, recovered via relocalization — a normal SLAM failure mode, not a pipeline bug)
- **Sparse map**: ~7,800 triangulated 3D points, forming a coherent planar cluster consistent with the desk surface in the scene
- **Estimated trajectory**: ~2.2m × 1.2m × 1.3m spatial extent, consistent with handheld camera motion around a desk
- Outputs saved as `poses.json`, `trajectory.txt` (TUM format), and `map_points.npy` — file-based by design, ready for the fusion stage and eventual FastAPI wrapping
- Quantitative accuracy (ATE/RPE against ground truth) deferred to `05_evaluation.ipynb`, where trajectories are properly Sim(3)-aligned before comparison

![SLAM trajectory and sparse map]([docs/slam_trajectory_preview.png](https://github.com/FaridRash/slam-track-fusion/blob/main/Output/01_slam.jpg))
*Estimated camera trajectory (blue) and sparse 3D map points (gray) on freiburg1_desk. Ground truth (green) shown unaligned — see note above.*

## Tech stack

- Python, PyTorch
- pyslam (visual SLAM — ORB2 features, DBoW3 loop closing)
- Ultralytics YOLO + ByteTrack (detection + tracking)
- ONNX Runtime (inference benchmarking)
- TUM RGB-D dataset (`freiburg1_desk`)

## Status

SLAM stage complete and verified on `freiburg1_desk`. Moving to detection + tracking next.

## Roadmap

- [x] SLAM trajectory + map on TUM RGB-D
- [ ] SLAM accuracy scored with ATE/RPE (`05_evaluation.ipynb`)
- [ ] Detection + tracking on video, validated visually and (where applicable) with MOTA/IDF1
- [ ] Fusion of tracked object positions into the SLAM map
- [ ] Real-time benchmarking (FPS, latency, ONNX vs native inference)
- [ ] Optional: port to embedded hardware (Jetson) as a v2 extension

## Author

Ali Rashidi (Farid) — Computer Vision / Deep Learning Engineer
[LinkedIn](https://linkedin.com/in/faridrash) · [Portfolio](https://faridrash.vercel.app)
