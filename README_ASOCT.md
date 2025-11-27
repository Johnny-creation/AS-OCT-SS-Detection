# AS-OCT 巩膜突关键点检测

YOLOv11 Pose 模型用于自动检测AS-OCT图像中左右巩膜突的位置。

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

### 3. 训练

修改 `train_asoct_pose.py` 选择模型：

```python
model_name = 'yolo11x-pose.pt'  # 可选: n/s/m/l/x
```

开始训练：

```bash
python train_asoct_pose.py
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

## 项目文件

```
├── datasets/
│   ├── Cataract/              # 原始数据
│   ├── Normal/
│   ├── PACG/
│   ├── PACG_Cataract/
│   ├── asoct-pose.yaml        # 配置
│   └── ASOCT_YOLO/            # 转换后数据
│
├── convert_asoct_to_yolo_pose.py
├── train_asoct_pose.py
├── validate_asoct_pose.py
├── predict_asoct_pose.py
├── evaluate_pixel_distance.py      # 像素距离评估
└── visualize_prediction_errors.py  # 误差可视化
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
