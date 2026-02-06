# MonetGPT 提示词文档 (Prompt Documentation)

本文档详细说明了 MonetGPT 代码库中调用模型时使用的所有提示词。

This document details all the prompts used when calling models in the MonetGPT codebase.

---

## 目录 (Table of Contents)

1. [推理阶段提示词 (Inference Prompts)](#推理阶段提示词-inference-prompts)
   - [系统提示词 (System Prompt)](#系统提示词-system-prompt)
   - [分析提示词 (Analysis Prompt)](#分析提示词-analysis-prompt)
   - [JSON生成提示词 (JSON Generation Prompt)](#json生成提示词-json-generation-prompt)
2. [数据集生成提示词 (Dataset Generation Prompts)](#数据集生成提示词-dataset-generation-prompts)
   - [Puzzle 1: 单操作分析](#puzzle-1-单操作分析)
   - [Puzzle 2: 多版本比较](#puzzle-2-多版本比较)
   - [Puzzle 3: 综合编辑计划](#puzzle-3-综合编辑计划)

---

## 推理阶段提示词 (Inference Prompts)

推理阶段使用两轮对话来生成图像编辑建议。第一轮进行分析，第二轮生成具体的JSON参数。

The inference stage uses a two-round conversation to generate image editing suggestions. The first round performs analysis, and the second round generates specific JSON parameters.

**位置 (Location):** `inference/core.py`

### 系统提示词 (System Prompt)

```
You are a helpful advanced image-editing assistant with expertise in Adobe Lightroom.
```

这是所有推理请求使用的系统级提示词。

This is the system-level prompt used for all inference requests.

### 分析提示词 (Analysis Prompt)

**模板 (Template):**

```
Analyze the provided image and develop a professional-grade editing plan using operations available in Adobe Lightroom to address issues in {operation_desc}. 

Your task is to identify all visual issues in the image and propose precise, optimized adjustments to address issues in {short_operation}. 

Create a professional editing plan for this photo with a list of the **optimal** adjustments needed to address the identified issues.

For each adjustment, follow this format:
Adjustment: [Mention the adjustment that needs to be made. Eg: **Adjustment:** The whites need to be greatly reduced**.]
Issue: [Explain the specific issue in the image, focusing on how it negatively impacts the photo, use as much context from the image as possible.]
Solution: [Describe how the adjustment will resolve the issue]

You should ensure that applying these adjustments will lead to an **optimal** image with balanced adjustment values specifically tuned for this image.

{style_instruction}

{extra_instruction}
```

**参数说明 (Parameters):**

- `operation_desc`: 操作类型的详细描述，包含可用的操作列表
  - Detailed description of the operation type, including the list of available operations
- `short_operation`: 操作类型的简短名称
  - Short name of the operation type
- `style_instruction`: 风格指导（balanced/vibrant/retro）
  - Style guidance (balanced/vibrant/retro)
- `extra_instruction`: 用户提供的额外指令
  - Additional instructions provided by the user

**操作类型 (Operation Types):**

1. **white-balance-tone-contrast**: 曝光、对比度和色调范围调整
   - Exposure, Contrast & Tonal Range Adjustments
   - 可用操作 (Available): Blacks, Contrast, Highlights, Shadows, Whites, Exposure

2. **color-temperature**: 白平衡和全局饱和度
   - White Balance & Global Saturation
   - 可用操作 (Available): Temperature, Tint, Saturation

3. **hsl**: 选择性颜色调整
   - Selective Color Adjustments
   - 可用操作 (Available): 各种色相、亮度、饱和度调整 (Various Hue, Luminance, Saturation adjustments)

**风格说明 (Style Instructions):**

- **balanced（平衡）**: 追求平衡的编辑，保持自然、真实的外观
  - Aim for balanced edits, maintaining a natural, true-to-life look
  
- **vibrant（鲜艳）**: 追求鲜艳和有冲击力的颜色，适合社交媒体
  - Aim for vibrant and punchy colors, suitable for social media
  
- **retro（复古）**: 追求怀旧的复古氛围，柔和、褪色的色彩
  - Aim for a nostalgic retro vibe with muted tones and soft colors

### JSON生成提示词 (JSON Generation Prompt)

**模板 (Template):**

```
Based on the editing plan and the original image, tell the **optimal** adjustment values needed to edit this photo in JSON format. All adjustment values are scaled between -100 and +100. You must ensure that the final edited image has **optimal** adjustment values to look like an **optimal image**.

{intensity_legend}

{style_instruction}

{extra_instruction}
```

**强度图例 (Intensity Legend):**

```
The following legend can be used to map the intensity values from your previous answer:
1-12: Very Slight
13-24: Slight
25-36: Mild
37-48: Moderate
49-60: Noticeable
61-72: Significant
73-84: Very Significant
85-100: Extremely Intense
```

---

## 数据集生成提示词 (Dataset Generation Prompts)

数据集生成使用不同的提示词为三种类型的谜题生成推理解释。

Dataset generation uses different prompts to generate reasoning explanations for three types of puzzles.

**位置 (Location):** `dataset/query_llm.py`

### Puzzle 1: 单操作分析

**用途 (Purpose):** 教授单个图像修饰操作
- Teaching individual retouching operations

**系统提示词 (System Prompt):**

```
You are a helpful assistant with expertise in image editing using Lightroom. You will compare the images side by side (images are stitched side by side so you can DEFINITELY do this)
```

**用户提示词 (User Prompt):**

```
Here's a stitched pair of images (original and edited). Analyze the changes and provide the following:
Question: An expert image editor has applied the operation {operation} with value {value} in Lightroom. The value ranges from -100 to +100.
Justify with reasoning.

Follow this template to answer your question.
1. The editing operation(s) applied.
2. The value of each adjustment.
3. Your step-by-step reasoning.


Use the following legend to describe the degree of adjustment:
1–12: Very Slight
13–24: Slight
25–36: Mild
37–48: Moderate
49–60: Noticeable
61–72: Significant
73–84: Very Significant
85–100: Extremely Intense

Example:
1. Operation(s) Applied: Saturation Adjustment.
2. Value of Adjustment: +25% saturation.

Reasoning:
Step 1: Observed that the colors in the edited image appear more vibrant compared to the original image.
Step 2: Identified that this increase in vibrancy is uniform across all colors, pointing to a saturation adjustment rather than a change in individual color channels.
Step 3: Compared the vibrancy to reference images with known saturation adjustments, approximating the change to +25%.

Summary:
The operation applied was a 'Saturation Adjustment,' with an approximate value of +25%.
```

### Puzzle 2: 多版本比较

**用途 (Purpose):** 教授参数值关系
- Teaching parameter value relationships

**系统提示词 (System Prompt):**

```
You are a helpful assistant with expertise in image editing using Lightroom. You will analyze the stitched images and provide detailed reasoning.
```

**用户提示词 (User Prompt):**

```
A source image has been edited in Lightroom where the `{operation}` has been adjusted to create 4 versions of the image: a, b, c, and d (from left to right and it's labelled as well). The stitched image represents these 4 versions in a random order of adjusted values.

Tasks:
1. **Sorting and Reasoning**:
   - The sorted order of the images from lowest to highest `{operation}` adjustment is: **`{order}`** (this is the answer).
   - Justify this order by explaining why each image has a lesser or greater level of `{operation}` compared to the others. When describing the reasoning:
     - Include as much context about the image as possible, such as how highlights, shadows, textures, colors, or other elements are affected by the adjustment.
     - Avoid generic or vague statements like "It balances the highlights." Instead, pinpoint exact issues caused by under- or over-adjustment (e.g., loss of detail in shadows, unnatural brightness, washed-out tones, etc.).

 **Image `{index}`** is the optimal image.

2. **Justification for optimal Image `{index}`**:
   - Explain why image **`{index}`** has the optimal level of `{operation}`.
   - Focus on how this level of adjustment specifically improves the image's visual quality while preserving critical details. Mention issues observed in other images and how the optimal level avoids those problems.
   - Address specific visual elements affected by the adjustment and why this level is optimal for maintaining balance and naturalness.

### Sample Response: (**You must follow the below structure/format**)

Sorted Ordering:
The sorted order of the images from lowest to highest Contrast adjustment is: c, a, b, d.

Justification:
Image c: This image has the least Contrast, as indicated by its very flat tonal range. The shadows appear washed out and lack depth, making the entire scene feel lifeless. For example, the tree bark in the foreground has almost no discernible texture, and the clouds in the sky are soft and undefined.
Image a: This image demonstrates moderate Contrast, with noticeable improvements. The tree bark reveals more texture, and the clouds in the sky show better definition. However, it still does not achieve the depth and vibrancy seen in b.
Image b: This image strikes the perfect balance of Contrast. The highlights in the clouds are bright but retain subtle textures, while the shadows in the tree bark are deep but still preserve intricate patterns and grooves. This level of Contrast enhances the image's tonal range without overemphasizing differences, creating a natural and visually appealing result.
Image d: This image has the highest level of Contrast, resulting in over-emphasized tonal differences. The shadows are overly dark, obscuring details in the tree bark and turning it into a uniform black mass. Similarly, the highlights in the clouds are blown out, losing subtle textures and making the sky appear unnaturally harsh.

Why Image b is optimal:
The operation applied is Contrast, and image b has the optimal level of adjustment.
This level of Contrast achieves a balanced tonal range that enhances visual quality while preserving critical details:
Shadows: The shadows are rich and deep, adding a sense of depth to the image, but they retain fine details, such as the individual grooves and cracks in the tree bark.
Highlights: The highlights are bright enough to make the clouds in the sky stand out, but they still maintain subtle gradients and textures, avoiding a washed-out or overly bright appearance.

Comparison to Other Images:
Compared to c, image b avoids the flat, lifeless appearance caused by low Contrast, where shadows and highlights lack differentiation.
Compared to d, image b avoids the harsh tonal extremes that result in loss of detail in both the highlights and shadows.
Image b is optimal because it reveals the intricate textures of the tree bark while maintaining the subtle gradients in the clouds, ensuring both shadow and highlight details are preserved. This balance enhances the scene's depth and vibrancy, creating a visually striking image that remains true to the natural appearance of the original scene.
```

### Puzzle 3: 综合编辑计划

**用途 (Purpose):** 教授完整的修饰工作流程
- Teaching complete retouching workflows

**系统提示词 (System Prompt):**

```
You are an advanced image-editing assistant with expertise in Adobe Lightroom. You have the ability to analyze a stitched pair of images (original on the left, edited on the right). Your task is to produce a single set of instructions that identifies issues in the original image and provides justified adjustments to address them.

When describing your solutions, follow these rules:

1. **Do not refer to the present edited photo in a direct manner; instead, phrase any improvements in future tense (e.g., "the edited image will have...")**.
2. Provide detailed justifications for each suggested adjustment, linking them to the specific issues found in the original image.
3. Describe adjustment intensities using a descriptive legend (Very Slight, Slight, Mild, Moderate, etc.) instead of numerical values.
4. Avoid vague or generic statements—be precise about what is wrong in the original image and how the adjustment solves that issue.
5. Base your justifications on visible changes seen in the stitched comparison.
```

**用户提示词 (User Prompt):**

```
Examine the side-by-side comparison image (original on the left, edited on the right) in detail to understand every single change the professional photographer made. 

We will be adressing issues in {operation}.

Identify all the issues in the original image that prompted these adjustments, being specific and descriptive (e.g., blown-out highlights, lack of contrast, color cast, mood issues, distracting background elements, etc.). Explain why each element might pose a problem or need adjusting.

{editing_goal_summary}

After identifying these issues, create a list of the adjustments needed to address them. Use this format for each adjustment:

[Start your answer by saying the category of operations it addresses. In this case: {short_operation}]

**Adjustment:** [Mention the adjustment that needs to be made, phrased as an instruction. E.g.: "The whites need to be greatly reduced."]  
**Issue:** [Explain the specific issue in the original image, focusing on how it negatively impacts the photo. Be explicit about the visual problem, referencing the original photo's elements—no vague statements or incomplete references.]  
**Solution:** [Describe how the adjustment will resolve the issue, phrased in a future-oriented tone. For instance, say "the edited image will have more balanced highlights" rather than "in the edited image, the highlights are balanced." **Make sure that the change is observable in the edited images**]

Add a very short summary at the end justifying how it meets the editing goal (balanced vs punchy colors) the expert editor was aiming for.

Include as much context as possible about the original and edited versions, referencing notable visual features that stand out or become altered in the final edit (such as lighting nuances, tonal ranges, color balance, focal elements, expressions, textures, or compositional details). 
Connect each edit to the specific elements in the original photo that need improving (for example, a color cast in the subject's shirt or underexposed shadows in the background) and describe precisely how the chosen adjustments resolve these issues.
**Make sure that the change is observable in the edited images**

The actual adjustments from the professional editor are given below. (All adjustments range from -100 to +100, where higher absolute values signify more intensity.)

Actual adjustments:
{settings}

These adjustments specifically address operations for {operation}.

When describing the degree of each change, do not mention the numerical values themselves. Instead, use the following legend to characterize intensity:
1-12: Very Slight
13-24: Slight
25-36: Mild
37-48: Moderate
49-60: Noticeable
61-72: Significant
73-84: Very Significant
85-100: Extremely Intense

Avoid generic or vague statements like "It balances the highlights." Instead, clearly pinpoint the exact issue (e.g., overexposure in specific areas, color casting on the subject's face) and explain how the adjustment solves that problem. Also avoid uncertainty—base your justifications on the most probable reasons for each adjustment and reference the visible changes in the side-by-side comparison to confirm your reasoning. Additionally, do not explicitly refer to the present edited photo; use a future-oriented tone for the solution description (e.g., "the edited image will have deeper, more vibrant colors," rather than "in the edited image, the colors are deeper and more vibrant.").

Example of an adjustment:

**Adjustment:** The highlights need to be significantly reduced.  
**Issue:** In the original image, the highlights, particularly on the surface of the water and parts of the woman's dress, are too bright and verge on being blown out. These overexposed regions lack detail and create a visual distraction.  
**Solution:** By significantly reducing the highlights, the details in the brightest parts of the image will be recovered. The overexposed areas of the water will gain texture, the details in the dress will become more visible, and the overall image will have a more balanced exposure.

**Start your answer like below:
Editing Goal: {editing_goal_summary}**

Followed by [adjustments]

**Do not write any summary at the end and do not mention {short_operation} for every single adjustment, only mention it once at the beginning.**
```

---

## 配置文件 (Configuration Files)

所有提示词配置存储在以下文件中：

All prompt configurations are stored in the following files:

- **推理配置 (Inference Config)**: `configs/inference_config.yaml`
  - 包含操作类型、风格指令、强度图例
  - Contains operation types, style instructions, intensity legend

- **数据集配置 (Dataset Config)**: `configs/dataset_config.yaml`
  - 包含API设置、模型名称、路径配置
  - Contains API settings, model name, path configurations

---

## 使用的模型 (Models Used)

1. **推理阶段 (Inference)**: 
   - 本地部署的 MonetGPT 模型
   - Locally deployed MonetGPT model
   - API端点 (API endpoint): `http://localhost:8000/v1`
   - 模型名称 (Model name): `test`

2. **数据集生成 (Dataset Generation)**:
   - Google Gemini 2.0 Flash
   - API端点 (API endpoint): `https://generativelanguage.googleapis.com/v1beta/openai/`

---

## 关键设计原则 (Key Design Principles)

1. **两阶段推理 (Two-Stage Inference)**:
   - 第一阶段：分析问题并提出编辑计划
   - 第二阶段：生成具体的JSON参数值
   - Stage 1: Analyze issues and propose editing plan
   - Stage 2: Generate specific JSON parameter values

2. **强度描述 (Intensity Description)**:
   - 使用描述性术语而非数值（如"Moderate"而非"40"）
   - 帮助模型更好地理解调整的程度
   - Use descriptive terms instead of numbers
   - Helps model better understand adjustment degrees

3. **上下文丰富性 (Context Richness)**:
   - 提示词要求详细描述图像内容和问题
   - 避免模糊或通用的陈述
   - Prompts require detailed description of image content and issues
   - Avoid vague or generic statements

4. **风格控制 (Style Control)**:
   - 支持多种编辑风格（balanced, vibrant, retro）
   - 可通过配置灵活调整
   - Support multiple editing styles
   - Flexible adjustment through configuration

---

## 示例使用 (Example Usage)

### 推理示例 (Inference Example)

```python
from inference.core import StagedEditingPipeline

pipeline = StagedEditingPipeline()
result = pipeline.process_image(
    image_path="input.jpg",
    output_base_path="output/result",
    style="balanced",  # 或 "vibrant" 或 "retro"
    extra_instructions={
        "white-balance-tone-contrast": "Focus on recovering shadow details"
    }
)
```

### 数据集生成示例 (Dataset Generation Example)

```bash
# 生成 Puzzle 1 的推理
python dataset_cli.py query 1 0 -1

# 生成 Puzzle 2 的推理
python dataset_cli.py query 2 0 -1

# 生成 Puzzle 3 的推理
python dataset_cli.py query 3 0 -1
```

---

## 总结 (Summary)

MonetGPT 使用精心设计的提示词系统来：

1. 引导模型进行专业级图像分析
2. 生成可解释的编辑建议
3. 通过谜题训练提高模型的操作理解能力
4. 支持多种编辑风格和自定义指令

MonetGPT uses a carefully designed prompt system to:

1. Guide model for professional-level image analysis
2. Generate explainable editing suggestions
3. Improve model's operation understanding through puzzle training
4. Support multiple editing styles and custom instructions

所有提示词都强调：
- 精确性和可解释性
- 避免过度或不足的调整
- 保持图像的自然外观
- 基于可观察的视觉变化

All prompts emphasize:
- Precision and explainability
- Avoiding over- or under-adjustment
- Maintaining natural image appearance
- Based on observable visual changes
