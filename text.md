---
layout: default
title: "Azulejo Diffusion Reconstruction"
---

<!-- Hero Section -->
<div align="center">

# Azulejo Diffusion Reconstruction

Macao Portuguese Tiles × Diffusion Models  
澳门葡式 Azulejo × 扩散模型数字重建

**Year:** 2026

<br>

<img src="images/hero_azulejo_grid.png" alt="Azulejo diffusion reconstruction gallery" width="80%">

</div>

---

## Project at a Glance

- **Type:** Master’s thesis project  
- **Theme:** Digital reconstruction and style analysis of Macao Azulejo tiles  
- **Keywords:** Stable Diffusion 1.5, LoRA, ControlNet, structure-aware generation, cultural style metrics  
- **Role:** Full-stack research (data, modeling, metrics, visualization, user interviews)

> _“How can a generic diffusion model learn to respect the strict symmetry and tiling rules of Azulejo,  
> while still preserving its unique blue–white cultural style?”_

---

## 1. What This Project Is About

Macao’s Portuguese-style **Azulejo** tiles appear on façades and street-name signs.  
They are highly structured and highly cultural:

- strong **diagonal / mirror symmetry** and **periodic tiling**  
- distinctive **blue–white glaze** and color palette  
- in Macao, often combined with **Chinese–Portuguese bilingual text**

In this project I:

1. Built a **diffusion-based pipeline** to reconstruct and generate Azulejo patterns.  
2. Designed a set of **multi-level metrics** to evaluate structure, texture, color and cultural style consistency.  
3. Collected **subjective feedback** to see when AI-generated tiles “feel” like real Azulejo.

---

## 2. Visual Highlights – Before & After

### Img2img: Diagonal-Symmetric Azulejo Reconstruction

<div align="center">

<table>
  <tr>
    <td align="center"><b>Original</b></td>
    <td align="center"><b>Generated (SD1.5 + LoRA + UNet)</b></td>
  </tr>
  <tr>
    <td><img src="images/before_after_1.png" alt="Azulejo before after 1" width="100%"></td>
    <td><img src="images/before_after_2.png" alt="Azulejo before after 2" width="100%"></td>
  </tr>
</table>

</div>

- 输入是带轻微噪声 / 退化的 Azulejo 斜对称图案  
- 输出在**保持对称与拼缝连续**的前提下，恢复纹理细节和色彩氛围

（你可以继续在下面加更多 pair，对应更多街牌或不同图案）

---

## 3. Overall System – From Data to Evaluation

<div align="center">
<img src="images/fig_overall_framework.png" alt="Overall research framework" width="95%">
</div>

**How to read this diagram:**

1. **Data & Input**  
   - Azulejo photos → cropping & alignment  
   - Symmetry annotations, tiling masks, edge maps  

2. **Structure-Aware Diffusion Model**  
   - Text / image as content input  
   - Structural conditions (symmetry, tiling, edges)  
   - Generates structure-preserving Azulejo images  

3. **Objective Evaluation & Style Space**  
   - Extract GLCM, Lab color, CLIP features  
   - Compute SSIM / ΔE / texture & color distances / CSCI  
   - Visualize everything in a “style feature space”  

4. **Subjective Experiment & Human Alignment**  
   - User ratings and pairwise comparisons  
   - Learn human-aligned style distance  

5. **Generalization**  
   - Extend to other cultural patterns, mosaics, textiles, etc.

---

## 4. Structure-Aware Diffusion (Method Core)

Although the master’s work mainly uses existing components (Stable Diffusion 1.5 + LoRA + ControlNet + UNet shrink),  
I organize them into a **structure-aware pipeline** that explicitly respects Azulejo geometry.

<div align="center">
<img src="images/fig_structure_aware_diffusion.png" alt="Structure-aware diffusion model" width="95%">
</div>

**Key ideas in this model:**

- **Content input:** text or image (img2img).  
- **Structural conditions:**  
  - symmetry axes  
  - tiling masks (periodic grid)  
  - edges / lineart  
- Conditions are injected into **different UNet layers**:  
  - encoder → local edges and contours  
  - middle layers → global symmetry & layout  
  - decoder → detail refinement & seamless boundaries  

The goal is that the model does not just “roughly resemble” a tile, but actually **knows it is drawing something symmetric and tilable**.

---

## 5. Measuring Cultural Style – CSCI & Style Feature Space

单看 PSNR / SSIM 不足以说明“像不像 Azulejo”。  
所以我把多种特征拼成一个 **style feature vector**：

- 纹理：多方向、多距离 GLCM 特征  
- 色彩：Lab 色彩直方图与 ΔE\*ab  
- 语义 / 构图：CLIP 图像嵌入、多层感知特征

然后降维到 2D，得到一个“**Azulejo 风格特征空间**”：

<div align="center">
<img src="images/fig_style_space_csci.png" alt="Azulejo style feature space and CSCI" width="95%">
</div>

在这个空间里：

- ⭐ = 参考原图  
- 不同颜色的点 = 不同生成方法（I0: SD1.5 / I1: SD1.5+LoRA / I2: +UNet / I3: +ControlNet）  
- 每种方法在原图附近形成一个**小簇**

对每个方法 \(m\)，我计算：

- **\(D_{\text{ref}}(m)\)**：该方法样本到参考图的平均距离 → 风格有多接近原图  
- **\(D_{\text{intra}}(m)\)**：簇内部的平均距离 → 风格是否稳定一致  

通过指数归一化，把它们映射到 \([0,1]\)，得到：

- \(\text{CSCI}_\text{ref}(m)\)（对参考图的一致性）  
- \(\text{CSCI}_\text{intra}(m)\)（方法内部的一致性）  
- 几何平均 → **CSCI（文化风格一致性指数）**

直观：  
> 「点离原图近、同时聚得紧」→ CSCI 高；  
> 「虽然结构好，但整体风格漂得很远」→ CSCI 会拉低。

---

## 6. Quantitative Results – What the Numbers Say

<div align="center">
<img src="images/fig_img2img_metrics.png" alt="Img2img quantitative metrics" width="95%">
</div>

（这里可以用你那张 8 小图拼在一起的指标图）

**Observation examples（可按你的实际结果微调）：**

- 在中等 denoise（例如 0.35）下，  
  - **SD1.5 + LoRA + ControlNet** 取得 **最高 SSIM 与最小接缝误差**；  
  - 但在风格特征空间中，簇离参考图略远，说明结构虽然好，风格略“过于规整”。  
- **SD1.5 + LoRA + UNet 收缩** 整体 CSCI 最高：  
  - 结构保持较好；  
  - 纹理和色彩更接近原图的手绘质感。

<div align="center">
<img src="images/fig_csci_bar.png" alt="CSCI bar chart" width="65%">
</div>

*Higher CSCI = closer to reference style and more internally consistent.*

---

## 7. User Study – When Do People Believe It?

为了验证指标是否“有人的感觉”，我做了一个小型主观实验：

- **参与者：**  
  - 设计相关学生  
  - 熟悉澳门街区的本地居民  
  - 一般观众  

- **评价维度（Likert 评分 + 访谈）：**  
  - 几何结构与对称感  
  - 釉面纹理与细节  
  - 色彩与氛围是否“像典型 Azulejo”  
  - 整体文化真实性 / 在地性  

**初步结论：**

- 人对 **对称破坏与拼缝** 极度敏感：  
  高 SSIM + 低 seam error 的图往往得分更高。  
- Lab 色彩分布与 ΔE\*ab 与“像不像 Azulejo 蓝”的主观感受有明显相关。  
- 预设的 CSCI 与“整体风格一致性”的主观评分有一定相关性 →  
  支持在博士阶段进一步发展为**人类对齐的风格距离函数**。

---

## 8. What’s Next

这个硕士项目是我后续博士计划的起点。下一步，我想在此基础上做：

- 真正的 **结构感知扩散架构**（显式引入对称 / 平铺先验，而不是简单 ControlNet）  
- 一套理论上更完整的 **CSCI 指标家族**，并通过大规模主观实验学习人类对齐的风格距离  
- 将方法扩展到其他 **图案型文化影像**：葡萄牙本土 Azulejo、马赛克、织物纹样等  

---

## 9. Download / Contact

- **Thesis PDF:** _link to be added_  
- **Code & scripts (planned):** _GitHub repo link_  

如果你对 **文化遗产 × 生成式模型 × 视觉评估** 感兴趣，欢迎联系我 😊  
