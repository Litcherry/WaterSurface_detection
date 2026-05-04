# WaterSurface_detection：基于改进 YOLOv5s 的水面垃圾检测模型

该项目针对水面环境复杂（反光、波纹干扰）、现有机器人功耗大及误检率高等痛点，开发了一套深度学习检测模型，并成功部署于树莓派（Raspberry Pi）边缘端。

## 核心改进：SimAM 注意力机制集成

在本项目中，我们针对 YOLOv5s 的检测头（Head）进行了优化，引入了 **SimAM（Simple Parameter-Free Attention Module）**。

*   **无参数化特性**：SimAM 是一种不需要增加额外参数量的注意力机制，通过计算神经元的重要性来推导 3D 权重，非常适合资源受限的树莓派平台。
*   **架构优化**：在 P3、P4、P5 检测层之前分别挂载 SimAM 模块，增强了模型在复杂水面背景下提取垃圾特征的能力。

### 模型配置预览 (SimAM Head v3)
```yaml
# 部分 head 配置代码
head:
  [ ... ]
   # --- SimAM 挂载层 ---
   [17, 1, SimAM, [256]], # 24 (输入来自第 17 层)
   [20, 1, SimAM, [512]], # 25 (输入来自第 20 层)
   [23, 1, SimAM, [1024]], # 26 (输入来自第 23 层)
 
   [[24, 25, 26], 1, Detect, [nc, anchors]], # 27 检测层输出
```

## 实验结果与性能分析

通过在 Roboflow 开源水面垃圾数据集上的训练对比，改进后的模型在不增加计算负担的前提下，实现了各项指标的全面提升。

### 核心指标对比

| 模型版本                 | mAP50      | mAP50-95   | Precision (精确率) |
| :----------------------- | :--------- | :--------- | :----------------- |
| YOLOv5s (Baseline)       | 0.592      | 0.376      | 0.736              |
| **YOLOv5s-SimAM (Ours)** | **0.608**  | **0.384**  | **0.754**          |
| **提升幅度 (相对)**      | **+2.70%** | **+2.13%** | **+2.45%**         |

### 深度分析：0.7% 之外的含金量
虽然数值上的绝对增长看似较小，但在目标检测领域，尤其是不破坏预训练权重的前提下，实现正向增长标志着架构实验的成功：
1.  **精确率显著提升**：Precision 从 0.736 提升至 0.754，意味着模型“认错”概率大大降低。这在实际应用中能有效避免机器人将浪花或反光误判为垃圾，从而节省能耗。
2.  **难检类别突破**：在“纸箱（kardus-kertas）”与“塑料袋（kantong-plastik）”等类别上均观察到了稳步增长，提升了模型在恶劣光照下的鲁棒性。

## 部署与应用

*   **跨平台部署**：模型已通过 ONNX 格式完成转换，并在 **Raspberry Pi** 上优化了推理流程。
*   **实物集成**：算法作为核心感知模块，集成了北斗定位与太阳能电力管理系统，服务于实物装置。

## 快速上手

### 环境要求
```bash
git clone https://github.com/Litcherry/WaterSurface_detection.git
cd WaterSurface_detection
pip install -r requirements.txt
```

### 运行推理
```bash
python detect.py --weights weights/best.pt --source your_video_or_image_path
```

## 致谢
- [Ultralytics YOLOv5](https://github.com/ultralytics/yolov5)
- [Roboflow Datasets](https://roboflow.com/)
- [SimAM 原理参考](https://github.com/ZjjGo/SimAM)
