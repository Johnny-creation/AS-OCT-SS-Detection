# AS-OCT 巩膜突关键点检测 - YOLOv11 Pose

基于YOLOv11的前节OCT (Anterior Segment OCT) 图像巩膜突关键点检测项目。

## 项目概述

本项目用于自动检测AS-OCT图像中的**巩膜突(Scleral Spur)**位置，这是眼科临床中重要的解剖标志点，用于:
- 房角评估 (Angle Assessment)
- 青光眼诊断 (Glaucoma Diagnosis)
- 白内障手术规划 (Cataract Surgery Planning)

## 数据集说明

### 数据统计
- **总样本数**: 717 张AS-OCT图像
- **图像尺寸**: 2135 x 1868
- **标注格式**: LabelMe JSON

### 类别分布
数据包含4个原始类别（合并训练为单一类别 'asoct'）:

| 类别 | 样本数 | 占比 |
|------|--------|------|
| Cataract (白内障) | 380 | 53.0% |
| Normal (正常) | 50 | 7.0% |
| PACG (原发性闭角型青光眼) | 148 | 20.6% |
| PACG_Cataract (青光眼+白内障) | 139 | 19.4% |

### 标注内容

#### 关键点 (Keypoints) - 用于训练
- **left_scleral_spur**: 左侧巩膜突位置
- **right_scleral_spur**: 右侧巩膜突位置

#### 辅助区域 (用于边界框计算)
- **lens**: 晶状体轮廓
- **nucleus**: 晶状体核轮廓
- **left_iris**: 左侧虹膜轮廓
- **right_iris**: 右侧虹膜轮廓
- **anterior_chamber**: 前房轮廓

## 环境配置

### 系统要求
- Python 3.8+
- CUDA 11.8+ (GPU版本，可选但强烈推荐)
- 8GB+ GPU显存 (推荐)

### 安装步骤

#### 方式A: 使用venv

```bash
# 1. 创建虚拟环境
python -m venv venv
venv\Scripts\activate  # Windows
# source venv/bin/activate  # Linux/Mac

# 2. 升级pip
pip install --upgrade pip

# 3. 安装PyTorch (选择对应CUDA版本)
# CPU版本:
pip install torch torchvision torchaudio

# GPU版本 (CUDA 11.8):
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118

# GPU版本 (CUDA 12.1):
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121

# 4. 安装Ultralytics和依赖
pip install -e .
pip install scikit-learn
```

#### 方式B: 使用conda (推荐)

```bash
# 1. 创建环境
conda create -n yolo11 python=3.11 -y
conda activate yolo11

# 2. 安装PyTorch
# GPU版本 (CUDA 12.1):
conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia -y

# 3. 安装Ultralytics和依赖
pip install -e .
pip install scikit-learn
```

### 验证安装

```python
import torch
from ultralytics import YOLO

print(f"PyTorch: {torch.__version__}")
print(f"CUDA可用: {torch.cuda.is_available()}")
if torch.cuda.is_available():
    print(f"GPU: {torch.cuda.get_device_name(0)}")
```

## 快速开始

### 步骤1: 数据转换

将LabelMe格式转换为YOLO Pose格式:

```bash
python convert_asoct_to_yolo_pose.py
```

**输出结构**:
```
datasets/ASOCT_YOLO/
├── images/
│   ├── train/  # 训练集 ~574张 (80%)
│   └── val/    # 验证集 ~143张 (20%)
└── labels/
    ├── train/  # 训练集标注
    └── val/    # 验证集标注
```

**YOLO标注格式**:
```
<class_id> <x_center> <y_center> <width> <height> <kp1_x> <kp1_y> <kp1_v> <kp2_x> <kp2_y> <kp2_v>
```
- 所有坐标归一化到 [0, 1]
- `kpX_v`: 0=未标注, 1=遮挡, 2=可见

### 步骤2: 训练模型

```bash
python train_asoct_pose.py
```

**训练配置**:
- **模型**: YOLOv11s-pose (Small, 推荐)
- **轮数**: 150 epochs
- **图像大小**: 640
- **批次大小**: 16
- **学习率**: 0.01 → 0.0001
- **预训练**: 使用COCO-pose预训练权重

**模型选择**:
| 模型 | 速度 | 精度 | 参数量 | 推荐场景 |
|------|------|------|--------|----------|
| yolo11n-pose | 最快 | 低 | 2.9M | 快速原型验证 |
| yolo11s-pose | 快 | 中 | 9.9M | **生产环境推荐** |
| yolo11m-pose | 中 | 高 | 21.0M | 高精度需求 |
| yolo11l-pose | 慢 | 很高 | 26.2M | 研究/离线处理 |
| yolo11x-pose | 最慢 | 最高 | 58.9M | 最高精度要求 |

**训练输出**:
```
runs/pose/asoct_yolo11s/
├── weights/
│   ├── best.pt   # 最佳模型
│   └── last.pt   # 最后一轮
├── results.png   # 训练曲线
├── confusion_matrix.png
└── val_batch*.jpg
```

### 步骤3: 验证模型

```bash
python validate_asoct_pose.py
```

**评估指标**:
- **mAP50-95 (pose)**: 主要关注指标
- **mAP50 (pose)**: IoU=0.5时的精度
- **mAP50 (box)**: 边界框检测精度

### 步骤4: 预测图片

```bash
python predict_asoct_pose.py
```

**自定义预测**:
```python
from ultralytics import YOLO

# 加载模型
model = YOLO('runs/pose/asoct_yolo11s/weights/best.pt')

# 预测单张图片
results = model.predict('path/to/image.jpg', save=True, conf=0.25)

# 获取关键点坐标
for r in results:
    keypoints = r.keypoints.xy  # shape: [num_detections, 2, 2]
    boxes = r.boxes.xyxy        # 边界框
    conf = r.boxes.conf         # 置信度

    # 第一个检测的关键点
    if len(keypoints) > 0:
        left_spur = keypoints[0][0]   # [x, y]
        right_spur = keypoints[0][1]  # [x, y]
        print(f"Left Scleral Spur: {left_spur}")
        print(f"Right Scleral Spur: {right_spur}")
```

## 高级配置

### 调整训练参数

编辑 `train_asoct_pose.py` 中的配置:

```python
config = {
    'epochs': 200,        # 增加训练轮数
    'batch': 8,          # GPU内存不足时减小
    'imgsz': 800,        # 增大输入尺寸提升精度
    'lr0': 0.005,        # 调整初始学习率
    'patience': 100,     # 增大早停耐心值
    'pose': 15.0,        # 增大关键点损失权重
}
```

### GPU内存优化

遇到 `CUDA out of memory` 错误:

```python
# 方法1: 减小批次大小
'batch': 8,  # 或 4, 2

# 方法2: 减小图像尺寸
'imgsz': 480,

# 方法3: 使用更小的模型
model = YOLO('yolo11n-pose.pt')

# 方法4: 禁用AMP
'amp': False,
```

### 数据增强

YOLOv11自动应用增强（可调整）:
```python
'hsv_h': 0.015,     # HSV色调增强
'hsv_s': 0.7,       # HSV饱和度
'hsv_v': 0.4,       # HSV明度
'degrees': 0.0,     # 旋转角度 (±度)
'translate': 0.1,   # 平移比例
'scale': 0.5,       # 缩放比例
'shear': 0.0,       # 剪切
'perspective': 0.0, # 透视变换
'flipud': 0.0,      # 上下翻转
'fliplr': 0.5,      # 左右翻转
'mosaic': 1.0,      # Mosaic增强
'mixup': 0.0,       # MixUp增强
```

## 命令行使用

```bash
# 训练
yolo pose train data=datasets/asoct-pose.yaml model=yolo11s-pose.pt epochs=150 imgsz=640 batch=16

# 验证
yolo pose val model=runs/pose/asoct_yolo11s/weights/best.pt data=datasets/asoct-pose.yaml

# 预测
yolo pose predict model=runs/pose/asoct_yolo11s/weights/best.pt source='path/to/images'

# 导出ONNX
yolo export model=runs/pose/asoct_yolo11s/weights/best.pt format=onnx
```

## 项目文件结构

```
d:\code\AS-OCT\oct\yolo\
├── datasets/
│   ├── Cataract/           # 原始数据
│   │   ├── Original Images/
│   │   └── Annotated Images/
│   ├── Normal/
│   ├── PACG/
│   ├── PACG_Cataract/
│   ├── ASOCT_YOLO/         # 转换后的YOLO格式
│   └── asoct-pose.yaml     # 数据集配置
├── convert_asoct_to_yolo_pose.py  # 数据转换脚本
├── train_asoct_pose.py            # 训练脚本
├── validate_asoct_pose.py         # 验证脚本
├── predict_asoct_pose.py          # 预测脚本
└── ASOCT_POSE_README.md           # 本文档
```

## 性能基准

在AS-OCT数据集上的预期性能:

| 模型 | mAP50-95 (pose) | 推理速度 | GPU显存 |
|------|-----------------|----------|---------|
| YOLOv11n-pose | ~0.80 | ~100 FPS | ~2GB |
| YOLOv11s-pose | ~0.85 | ~80 FPS | ~3GB |
| YOLOv11m-pose | ~0.88 | ~50 FPS | ~5GB |

*实际性能取决于数据质量和训练配置*

## 模型导出

训练完成后可导出为其他格式:

```bash
# ONNX (跨平台推荐)
yolo export model=runs/pose/asoct_yolo11s/weights/best.pt format=onnx

# TensorRT (NVIDIA GPU加速)
yolo export model=runs/pose/asoct_yolo11s/weights/best.pt format=engine device=0

# CoreML (Mac/iOS)
yolo export model=runs/pose/asoct_yolo11s/weights/best.pt format=coreml

# TFLite (移动端)
yolo export model=runs/pose/asoct_yolo11s/weights/best.pt format=tflite
```

## 常见问题

### 1. 找不到数据集
**错误**: `Dataset 'datasets/asoct-pose.yaml' not found`

**解决**:
- 确保运行了 `python convert_asoct_to_yolo_pose.py`
- 检查 `datasets/ASOCT_YOLO/` 目录是否存在
- 验证 `asoct-pose.yaml` 中的路径设置

### 2. 训练速度慢
**原因**: 使用CPU训练

**解决**:
```bash
# 检查CUDA
python -c "import torch; print(torch.cuda.is_available())"

# 应该输出 True
# 如果是 False，重新安装GPU版本的PyTorch
```

### 3. 关键点定位不准
**优化方法**:
1. 增加 `pose` 损失权重 (12.0 → 15.0)
2. 增大训练轮数 (150 → 200)
3. 使用更大的模型 (s → m)
4. 增大输入尺寸 (640 → 800)
5. 检查标注质量

### 4. GPU内存不足
**解决方案**:
```python
# 1. 减小batch
'batch': 4,

# 2. 减小图像尺寸
'imgsz': 480,

# 3. 使用更小模型
model = YOLO('yolo11n-pose.pt')
```

### 5. 验证精度低
**调试步骤**:
1. 可视化训练数据: `runs/pose/*/train_batch*.jpg`
2. 检查验证预测: `runs/pose/*/val_batch*.jpg`
3. 调整置信度阈值
4. 增加训练数据

## 医学应用场景

### 房角评估
- 自动测量房角宽度
- 识别窄角/闭角
- 辅助青光眼筛查

### 手术规划
- IOL位置预测
- 切口位置规划
- 手术风险评估

### 疾病监测
- 青光眼进展跟踪
- 术后随访评估
- 治疗效果监测

## 引用

```bibtex
@software{ultralytics_yolo11,
  title = {Ultralytics YOLO11},
  author = {Glenn Jocher and Jing Qiu},
  year = {2024},
  url = {https://github.com/ultralytics/ultralytics}
}
```

## 许可证

本项目基于 AGPL-3.0 许可证。详见 [LICENSE](LICENSE)。

## 技术支持

- Ultralytics文档: https://docs.ultralytics.com
- GitHub Issues: https://github.com/ultralytics/ultralytics/issues
- 论坛: https://community.ultralytics.com

## 更新日志

### v1.0.0 (2025-01-08)
- ✅ 支持4类别合并训练
- ✅ 717张AS-OCT图像
- ✅ 2个关键点检测
- ✅ YOLOv11s-pose基线模型
- ✅ 完整训练/验证/预测流程
