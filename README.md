# AS-OCT 巩膜突关键点检测

YOLOv11 Pose 模型用于自动检测AS-OCT图像中左右巩膜突的位置。

## 数据集准备

### 1. 目录结构

将数据放在以下位置：

```
./
└── datasets/
    ├── Cataract/
    │   ├── Original Images/       # 原始图片 (*.jpg)
    │   └── Annotated Images/      # LabelMe标注 (*.json)
    ├── Normal/
    │   ├── Original Images/
    │   └── Annotated Images/
    ├── PACG/
    │   ├── Original Images/
    │   └── Annotated Images/
    └── PACG_Cataract/
        ├── Original Images/
        └── Annotated Images/
```

### 2. 标注格式

LabelMe JSON 文件需包含：
- **关键点**: `left_scleral_spur`, `right_scleral_spur` (必须)
- **辅助区域**: `lens`, `nucleus`, `iris`, `anterior_chamber` (用于边界框)

### 3. 预训练权重

**自动下载**: 首次训练时自动下载预训练权重到项目根目录

**手动下载** (可选):
```bash
# 下载到项目根目录
wget https://github.com/ultralytics/assets/releases/download/v8.3.0/yolo11x-pose.pt
# 或其他模型: yolo11n-pose.pt, yolo11s-pose.pt, yolo11m-pose.pt, yolo11l-pose.pt
```

权重文件位置：
```
./
├── yolo11n-pose.pt   # 自动下载到项目根目录
├── yolo11s-pose.pt
├── yolo11m-pose.pt
├── yolo11l-pose.pt
└── yolo11x-pose.pt
```

## 快速开始

### 1. 安装环境

```bash
conda create -n yolo11 python=3.11 -y
conda activate yolo11
conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia -y
pip install -e .
pip install scikit-learn matplotlib
```

### 2. 数据转换

将LabelMe格式转换为YOLO格式：

```bash
python convert_asoct_to_yolo_pose.py
```

**输出位置**: `datasets/ASOCT_YOLO/`
```
datasets/ASOCT_YOLO/
├── images/
│   ├── train/  # 训练集图片
│   └── val/    # 验证集图片
└── labels/
    ├── train/  # 训练集标注 (*.txt)
    └── val/    # 验证集标注 (*.txt)
```

### 3. 训练

修改 `train_asoct_pose.py` 选择模型：

```python
model_name = 'yolo11x-pose.pt'  # 可选: n/s/m/l/x
```

开始训练：

```bash
python train_asoct_pose.py
```

**训练输出**: `runs/pose/asoct_yolo11x/`
```
runs/pose/asoct_yolo11x/
├── weights/
│   ├── best.pt      # 最佳模型（验证集最优）
│   └── last.pt      # 最后一轮模型
├── results.png      # 训练曲线
├── confusion_matrix.png
└── val_batch*.jpg   # 验证集可视化
```

### 4. 评估

```bash
# YOLO标准指标
python validate_asoct_pose.py

# 像素距离评估（关键）
python evaluate_pixel_distance.py

# 可视化误差
python visualize_prediction_errors.py
```

**评估输出**:
- `runs/pose/evaluation_results.png` - 像素距离分布图表
- `runs/pose/visualize_errors/` - 误差可视化图片（绿色圆圈=真实，蓝色叉=预测）

### 5. 预测

```bash
python predict_asoct_pose.py
```

## 损失函数配置

在 `train_asoct_pose.py` 中调整损失权重：

```python
'box': 0,        # 边界框损失
'cls': 1,        # 分类损失
'pose': 15.0,    # 关键点损失（主要优化）
'kobj': 1.0,     # 关键点置信度
```

## 模型选择

| 模型 | 参数 | 精度 | 速度 |
|------|------|------|------|
| yolo11n-pose | 2.9M | ★★ | ⚡⚡⚡ |
| yolo11s-pose | 9.9M | ★★★ | ⚡⚡ |
| yolo11m-pose | 21M | ★★★★ | ⚡ |
| yolo11l-pose | 26M | ★★★★ | ⚡ |
| yolo11x-pose | 59M | ★★★★★ | ⚡ |

## 评估指标

### 医学关键指标

| 等级 | MPD | PCK@10px | 临床适用性 |
|------|-----|----------|-----------|
| 优秀 | <5px | >95% | 临床诊断 |
| 良好 | <10px | >90% | 辅助诊断 |
| 可用 | <20px | >80% | 研究初筛 |

- **MPD**: 平均像素距离
- **PCK@10px**: 10像素内准确率

### YOLO指标

- **mAP50-95 (pose)**: >0.80 为良好

## Python API

```python
from ultralytics import YOLO

# 加载模型
model = YOLO('runs/pose/asoct_yolo11x/weights/best.pt')

# 预测
results = model.predict('image.jpg', conf=0.25)

# 获取关键点
keypoints = results[0].keypoints.xy[0]  # [2, 2]
left_spur = keypoints[0]   # [x, y]
right_spur = keypoints[1]  # [x, y]
```

## 完整目录结构

```
./                                     # 项目根目录
│
├── datasets/                          # 数据集目录
│   ├── Cataract/                      # 原始数据（需要提供）
│   │   ├── Original Images/           # 图片文件
│   │   └── Annotated Images/          # JSON标注
│   ├── Normal/                        # 原始数据（需要提供）
│   │   ├── Original Images/
│   │   └── Annotated Images/
│   ├── PACG/                          # 原始数据（需要提供）
│   │   ├── Original Images/
│   │   └── Annotated Images/
│   ├── PACG_Cataract/                 # 原始数据（需要提供）
│   │   ├── Original Images/
│   │   └── Annotated Images/
│   ├── asoct-pose.yaml                # 数据集配置
│   └── ASOCT_YOLO/                    # 转换后数据（自动生成）
│       ├── images/
│       │   ├── train/                 # 训练集
│       │   └── val/                   # 验证集
│       └── labels/
│           ├── train/                 # 训练标签
│           └── val/                   # 验证标签
│
├── runs/pose/                         # 训练输出（自动生成）
│   └── asoct_yolo11x/
│       ├── weights/
│       │   ├── best.pt               # 最佳模型
│       │   └── last.pt               # 最后模型
│       ├── results.png               # 训练曲线
│       └── ...
│
├── yolo11x-pose.pt                   # 预训练权重（自动下载）
│
├── convert_asoct_to_yolo_pose.py     # 数据转换脚本
├── train_asoct_pose.py               # 训练脚本
├── validate_asoct_pose.py            # YOLO验证
├── predict_asoct_pose.py             # 预测脚本
├── evaluate_pixel_distance.py        # 像素距离评估
└── visualize_prediction_errors.py    # 误差可视化
```


## 命令行使用

```bash
# 训练
yolo pose train data=datasets/asoct-pose.yaml model=yolo11x-pose.pt epochs=150

# 验证
yolo pose val model=runs/pose/asoct_yolo11x/weights/best.pt data=datasets/asoct-pose.yaml

# 预测
yolo pose predict model=runs/pose/asoct_yolo11x/weights/best.pt source=path/to/images

# 导出
yolo export model=runs/pose/asoct_yolo11x/weights/best.pt format=onnx
```

---

**许可证**: AGPL-3.0
**文档**: [Ultralytics Docs](https://docs.ultralytics.com)
