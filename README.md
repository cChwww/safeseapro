# 🌊 基于扩散模型的海上遇险目标图像生成与增强研究

![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)
![PyTorch](https://img.shields.io/badge/PyTorch-Compatible-orange.svg)
![Stable Diffusion](https://img.shields.io/badge/Stable_Diffusion-v1.5-blueviolet.svg)
![YOLOv5](https://img.shields.io/badge/YOLO-v5-green.svg)
![Status](https://img.shields.io/badge/Status-Graduation_Project-brightgreen.svg)

## 📖 项目简介

本项目为2026届计算机科学与技术专业本科毕业设计课题。主要针对复杂海洋环境下，遇险目标（落水人员、救生筏、船只等）训练数据匮乏、长尾分布严重的问题，提出了一种**融合空间软缓冲和弱特征锚定的局部可控扩散生成方法**。

项目基于 Stable Diffusion 1.5 和 ControlNet，能够在绝对保留“前景遇险目标”物理结构的前提下，根据文本提示词动态生成强风浪、浓雾、夜间等复杂背景海况。最终通过将合成数据加入 YOLOv5 的训练中，显著提升了目标检测模型在极端海况下的检测精度、泛化能力与鲁棒性。

---

## 🗺️ 技术路线 (Technical Route)

本项目的数据增强与生成框架主要由三大核心模块协同构成，突破了传统硬掩码局部重绘带来的“前景侵蚀”与“贴纸效应”。

<div align="center">
  <img src="reademe_image/tech_route.jpg" width="800" alt="项目技术路线图"/>
  <p><em>图：基于扩散模型的海上遇险目标图像生成总体框架</em></p>
</div>

1. **基于边界缓冲的掩码保护机制**：通过形态学腐蚀与高斯平滑，将传统二值硬掩码转化为具有连续灰度梯度的软掩码，在潜在空间锁定前景目标，防止背景特征侵蚀。
2. **基于自适应结构提取的条件控制网络 (ControlNet)**：针对海上目标低对比度的特性，自适应调整 Canny 双阈值，提取完整弱边缘，提供强鲁棒性的结构先验约束。
3. **掩码引导的逐步融合扩散**：在反向去噪的全时间步中，利用软掩码灰度权重进行同频加噪回填与像素级特征插值，实现前景与生成背景的像素级无缝融合。

---

## 📊 实验结果展示 (Results & Visualization)

> **⚠️ 注意**：由于生成数据集较为庞大，本仓库的 `inputs/` 和 `outputs/` 文件夹下仅提供了**部分样例数据**用于演示运行流程。

### 1. 图像生成与增强对比
通过扩散模型，我们成功将单一平稳海况下的目标，迁移到了多种复杂海况中，且目标结构未发生畸变：

| 原始输入图像 (Input) | 生成增强图像 (Generated - 复杂海况) |
| :---: | :---: |
| <img src="reademe_image/example_input_1.jpg" width="350" alt="原图1"/> | <img src="reademe_image/example_output_1.jpg" width="350" alt="生成图1"/> |
| <img src="reademe_image/example_input_2.jpg" width="350" alt="原图2"/> | <img src="reademe_image/example_output_2.jpg" width="350" alt="生成图2"/> |

### 2. YOLOv5 下游检测性能提升
在 `yolov5/runs/train/` 目录下提供了基线（Baseline）与加入合成数据后（Augmented）的训练结果对比。从训练日志和验证结果可以明显看出：
* **泛化性显著提升**：在引入生成数据后，模型面对复杂海浪干扰的特征提取能力更强。全局 mAP@0.5 从 79.7% 提升至 **81.8%**。
* **误检率大幅下降**：“落水人员”的相对误检率降低了 12.3%，“救生设备”等微小目标的相对误检率更是下降了 31%。

| 基线模型检测结果 (safesea_baseline_1400) | 增强模型检测结果 (safesea_baseline_14001) |
| :---: | :---: |
| <img src="reademe_image/yolo_baseline.jpg" width="350" alt="基线检测"/> | <img src="reademe_image/yolo_augmented.jpg" width="350" alt="增强后检测"/> |

---

## 📂 目录结构

```text
📦 project_root
 ┣ 📂 inputs/               # 原始输入数据（由于数据量大，仅展示部分样例）
 ┃ ┣ 📂 images/             # 原始图像 (如 172.jpg, 393.jpg...)
 ┃ ┣ 📂 labels/             # YOLO 格式标注文件 (如 172.txt...)
 ┃ ┗ 📂 masks/              # 预处理生成的目标掩码图 (如 172_mask.png...)
 ┣ 📂 outputs/              # 扩散模型推理输出结果
 ┃ ┗ 📂 images/             # 生成增强后的图像样例 (如 172_s3.jpg...)
 ┣ 📂 scripts/              # 实验核心脚本
 ┃ ┣ 📜 compare_1.py        # 实验对比与数据评估脚本
 ┃ ┣ 📜 image_generation.py # 批量自动化生成脚本 (循环调用编辑流水线)
 ┃ ┗ 📜 text_editing_stable_diffusion_1.py # 核心：软掩码+Canny+SD扩散生成代码
 ┣ 📂 yolov5/               # YOLOv5 下游任务工作区
 ┃ ┣ 📂 runs/train/         # 训练日志与权重存放区
 ┃ ┃ ┣ 📂 safesea_baseline_1400/  # 基线组训练结果与权重 (仅使用真实数据)
 ┃ ┃ ┗ 📂 safesea_baseline_14001/ # 增强组训练结果与权重 (加入本文生成数据，泛化性提升)
 ┃ ┣ 📜 baseline.yaml       # 基线模型训练配置文件
 ┃ ┗ 📜 augement.yaml       # 增强数据集训练配置文件
 ┗ 📂 reademe_image/        # 存放本文档 (README) 所需的各种展示图片
