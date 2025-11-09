# AS-OCT 巩膜突检测 - 快速开始指南

## 🚀 5分钟快速上手

### 1️⃣ 环境配置 (2分钟)

```bash
# 激活虚拟环境
python -m venv venv
venv\Scripts\activate

# 安装依赖
pip install --upgrade pip
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
pip install -e .
pip install scikit-learn matplotlib
```

### 2️⃣ 数据转换 (1分钟)

```bash
python convert_asoct_to_yolo_pose.py
```

**期望输出**:
```
找到 717 个有效样本:
  Cataract            :  380 张
  Normal              :   50 张
  PACG                :  148 张
  PACG_Cataract       :  139 张

数据划分:
  训练集: 574 张 (80%)
  验证集: 143 张 (20%)
```

### 3️⃣ 开始训练 (等待时间)

```bash
python train_asoct_pose.py
```

**训练时间参考**:
- GPU (RTX 3090): ~2-3 小时
- GPU (RTX 4090): ~1-2 小时
- CPU: 不推荐（太慢）

### 4️⃣ 评估结果 (1分钟)

```bash
# YOLO标准指标
python validate_asoct_pose.py

# 像素距离评估（医学应用重点）
python evaluate_pixel_distance.py

# 可视化误差
python visualize_prediction_errors.py
```

## 📊 评估指标说明

### YOLO指标
- **mAP50-95 (pose)**: 综合精度指标，>0.80 为良好

### 医学指标（重点）
- **MPD (平均像素距离)**: 应 <10px
- **PCK@10px**: 应 >90%
- **PCK@20px**: 应 >95%

## ✅ 检查清单

训练前检查:
- [ ] 数据已转换: `datasets/ASOCT_YOLO/` 目录存在
- [ ] GPU可用: `python -c "import torch; print(torch.cuda.is_available())"` 输出 True
- [ ] 磁盘空间: 至少 10GB 可用

训练后检查:
- [ ] 模型文件: `runs/pose/asoct_yolo11s/weights/best.pt` 存在
- [ ] 训练曲线: `runs/pose/asoct_yolo11s/results.png` 可查看
- [ ] MPD < 15px (可接受范围)
- [ ] PCK@20px > 85%

## 🎯 下一步

1. **优化模型**: 调整超参数提升精度
2. **导出模型**: 转换为ONNX用于部署
3. **集成应用**: 集成到医学影像系统

## ❓ 常见问题

**Q: 训练很慢怎么办？**
A: 检查是否使用GPU。如果使用CPU，请安装GPU版本PyTorch。

**Q: 内存不足？**
A: 在 `train_asoct_pose.py` 中设置 `batch=8` 或 `batch=4`

**Q: MPD太大？**
A: 尝试增加训练轮数到200，或使用更大模型 `yolo11m-pose.pt`

## 📖 完整文档

详见 [ASOCT_POSE_README.md](ASOCT_POSE_README.md)
