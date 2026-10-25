# Cement Bag Detection using YOLOv8

A computer vision project for detecting cement bags in images and conveyor video using a custom-trained YOLOv8 model. Detection is the first stage of a planned cement bag tracking and counting system.

## Project status

- Custom dataset prepared in YOLO format with training, validation, and test splits.
- Model training completed and `best.pt` checkpoint generated.
- Evaluation results and prediction examples are pending verification.
- Object tracking, line-crossing counting, and multi-camera integration are planned.

No detection accuracy, counting accuracy, or real-time performance claim is made until measured on representative data.

## Technology

- Python
- Ultralytics YOLOv8
- PyTorch
- Custom bounding-box annotations in YOLO format

## Directory structure

```text
cbd/
├── datasets/
│   ├── train/
│   │   ├── images/
│   │   └── labels/
│   ├── valid/
│   │   ├── images/
│   │   └── labels/
│   ├── test/
│   │   ├── images/
│   │   └── labels/
│   └── data.yaml
├── runs/
│   └── detect/
│       └── cement_bag_v1/
│           └── weights/
│               ├── best.pt
│               └── last.pt
└── README.md
```

Training output folders may differ. Update the example model paths to match your actual checkpoint location. Dataset images and trained weights may need to be obtained separately if they are excluded from the repository.

## Setup

Open a terminal in the project root. These commands target Windows PowerShell with Python 3.11 installed.

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install ultralytics
```

Check whether PyTorch can use an NVIDIA GPU:

```powershell
python -c "import torch; print('PyTorch:', torch.__version__); print('CUDA available:', torch.cuda.is_available())"
```

GPU training requires a compatible NVIDIA driver and CUDA-enabled PyTorch installation. If CUDA is unavailable, use `device=cpu` in the commands below. CPU training can take considerably longer.

## Dataset configuration

For a single class, the dataset YAML should identify class `0` as `cement_bag`. Preserve the class order used during annotation if your dataset contains multiple classes.

Example `datasets/data.yaml`:

```yaml
# Replace with the absolute path to your datasets directory.
path: C:/path/to/cbd/datasets
train: train/images
val: valid/images
test: test/images

names:
  0: cement_bag
```

Each image's label file uses the same base filename, with one row per bounding box:

```text
class_id x_center y_center width height
```

Coordinates are normalized to the image dimensions, with values between 0 and 1. Keep related frames from the same recording in the same split to reduce data leakage. Reserve test data for final evaluation rather than tuning.

## Train a baseline model

The following example fine-tunes pretrained YOLOv8n weights for up to 100 epochs:

```powershell
yolo detect train model=yolov8n.pt data=datasets/data.yaml epochs=100 imgsz=640 batch=8 device=0 workers=0 project=runs/detect name=cement_bag_v1
```

For CPU training, replace `device=0` with `device=cpu` and consider `batch=4`. Reduce the batch size if training runs out of memory. The initial pretrained weights download requires internet access.

The example configuration uses 640-pixel training images. `workers=0` is a conservative starting setting for Windows. Actual completed epochs and settings should be checked in the run's `args.yaml` and training logs.

The run saves `best.pt`, selected using validation fitness, and `last.pt`, the latest checkpoint. Reusing a run name may create a numbered output folder.

## Evaluate

Evaluate validation performance while developing the model:

```powershell
yolo detect val model=runs/detect/cement_bag_v1/weights/best.pt data=datasets/data.yaml split=val
```

After finalizing the model, evaluate the held-out test split:

```powershell
yolo detect val model=runs/detect/cement_bag_v1/weights/best.pt data=datasets/data.yaml split=test
```

Record the measured results below. Values have not yet been supplied for this project.

| Metric | Test result |
| --- | --- |
| Precision | Pending |
| Recall | Pending |
| mAP@0.50 | Pending |
| mAP@0.50:0.95 | Pending |
| Inference speed on target hardware | Pending |

Review missed bags, false detections, and duplicate boxes alongside the metrics. Detection metrics do not establish counting accuracy.

## Run predictions

### Test images

```powershell
yolo detect predict model=runs/detect/cement_bag_v1/weights/best.pt source=datasets/test/images conf=0.25 save=True
```

### Conveyor video

```powershell
yolo detect predict model=runs/detect/cement_bag_v1/weights/best.pt source="C:/path/to/conveyor_video.mp4" conf=0.25 save=True
```

### Webcam preview

```powershell
yolo detect predict model=runs/detect/cement_bag_v1/weights/best.pt source=0 conf=0.25 show=True
```

The terminal prints the output directory for saved predictions. The confidence threshold of `0.25` is a starting value; choose the operating threshold using validation results.

## Planned counting pipeline

1. Detect bags in each video frame.
2. Track detections across frames with persistent object IDs.
3. Count directional crossings of a configured line.
4. Compare automatic counts against manually counted videos.
5. Add per-camera reporting and evaluate multi-camera operation.

These stages are future work. Per-frame detection totals must not be summed to produce a bag count, because the same bag appears in multiple frames.

## Limitations

Performance must be tested under site conditions, including dust, changing lighting, motion blur, overlapping bags, and different camera angles. FPS depends on hardware, model size, input resolution, and the complete processing pipeline. Conveyor control and relay integration are outside the current implementation.

## Repository preparation

- Add measured evaluation results and representative prediction images.
- Document dataset source, size, annotation method, and redistribution permissions.
- Record the tested Python and package versions for reproducibility.
- Exclude `.venv/`, temporary caches, and private footage from version control.
- Distribute large weights and datasets through an appropriate download link, Git LFS, or release asset when needed.
- Add a project license after checking dataset and dependency terms.

## References

- [Ultralytics YOLOv8 documentation](https://docs.ultralytics.com/models/yolov8/)
- [Ultralytics training documentation](https://docs.ultralytics.com/modes/train/)
- [Ultralytics licensing](https://www.ultralytics.com/license)

## Author

Deva Sahayaraj — [GitHub](https://github.com/DevaSahayaraj)
