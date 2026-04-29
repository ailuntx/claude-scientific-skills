```markdown
---
name: generate-image
description: 使用AI模型（FLUX、Nano Banana 2）生成或编辑图像。适用于通用图像生成场景，包括照片、插画、艺术作品、视觉素材、概念艺术等非技术图表。流程图、电路图、技术示意图等请改用 scientific-schematics 技能。
license: MIT 许可证
compatibility: 需要 OpenRouter API 密钥
metadata:
    skill-author: K-Dense Inc.
---

# 图像生成

使用 OpenRouter 图像生成模型（包括 FLUX.2 Pro 和 Gemini 3.1 Flash Image Preview）创建和编辑高质量图像。

## 使用场景

**本技能适用于：**
- 照片级写实图像
- 艺术插画与美术作品
- 概念艺术与视觉创意
- 演示文稿或文档的视觉素材
- 图像编辑与修改
- 任何通用图像生成需求

**技术图表请改用 scientific-schematics：**
- 流程图与过程示意图
- 电路图与电气原理图
- 生物通路与信号传导图
- 系统架构图
- CONSORT 流程图与方法学图示
- 任何技术/原理示意图

## 快速入门

通过 `scripts/generate_image.py` 脚本生成或编辑图像：

```bash
# 生成新图像
python scripts/generate_image.py "山脉上壮丽的日落"

# 编辑现有图像
python scripts/generate_image.py "将天空改为紫色" --input photo.jpg
```

生成/编辑的图像将保存为当前目录下的 `generated_image.png`。

## API 密钥配置

**重要提示**：脚本需配置 OpenRouter API 密钥。运行前请检查：

1. 在项目目录或上级目录查找 `.env` 文件
2. 确认 `.env` 文件中包含 `OPENROUTER_API_KEY=<密钥>`
3. 若未找到密钥，需指导用户：
   - 创建 `.env` 文件并写入 `OPENROUTER_API_KEY=你的API密钥`
   - 或设置环境变量：`export OPENROUTER_API_KEY=你的API密钥`
   - 密钥申请地址：https://openrouter.ai/keys

脚本会自动检测 `.env` 文件，并在密钥缺失时提供明确错误提示。

## 模型选择

**默认模型**：`google/gemini-3.1-flash-image-preview`（高质量推荐）

**支持生成与编辑的模型**：
- `google/gemini-3.1-flash-image-preview` - 高质量，支持生成+编辑
- `black-forest-labs/flux.2-pro` - 快速高效，支持生成+编辑

**仅支持生成的模型**：
- `black-forest-labs/flux.2-flex` - 经济快速，质量低于 Pro 版

选择依据：
- **质量优先**：选用 gemini-3.1-flash-image-preview 或 flux.2-pro
- **编辑需求**：选用 gemini-3.1-flash-image-preview 或 flux.2-pro（均支持编辑）
- **成本优先**：生成场景选用 flux.2-flex

## 常用命令模式

### 基础生成
```bash
python scripts/generate_image.py "你的提示词"
```

### 指定模型
```bash
python scripts/generate_image.py "太空中的猫" --model "black-forest-labs/flux.2-pro"
```

### 自定义输出路径
```bash
python scripts/generate_image.py "抽象艺术" --output artwork.png
```

### 编辑现有图像
```bash
python scripts/generate_image.py "将背景改为蓝色" --input photo.jpg
```

### 指定模型编辑
```bash
python scripts/generate_image.py "给人像添加太阳镜" --input portrait.png --model "black-forest-labs/flux.2-pro"
```

### 自定义编辑输出
```bash
python scripts/generate_image.py "移除图片中的文字" --input screenshot.png --output cleaned.png
```

### 批量生成
多次运行脚本指定不同提示词或输出路径：
```bash
python scripts/generate_image.py "图像1描述" --output image1.png
python scripts/generate_image.py "图像2描述" --output image2.png
```

## 脚本参数

- `prompt` (必填)：图像生成描述或编辑指令文本
- `--input` 或 `-i`：待编辑图像的输入路径（启用编辑模式）
- `--model` 或 `-m`：OpenRouter 模型ID（默认：google/gemini-3.1-flash-image-preview）
- `--output` 或 `-o`：输出文件路径（默认：generated_image.png）
- `--api-key`：OpenRouter API 密钥（覆盖.env文件）

## 应用案例

### 科研文档
```bash
# 生成论文概念图
python scripts/generate_image.py "免疫治疗剂攻击癌细胞的显微视图，科学插画风格" --output figures/immunotherapy_concept.png

# 创建演示文稿配图
python scripts/generate_image.py "突出突变位点的DNA双螺旋结构，现代科学可视化风格" --output slides/dna_mutation.png
```

### 演示文稿与海报
```bash
# 标题页背景
python scripts/generate_image.py "含分子纹理的蓝白抽象背景，专业演示风格" --output slides/background.png

# 海报主视觉
python scripts/generate_image.py "配备现代仪器的实验室环境，写实风格，光线充足" --output poster/hero.png
```

### 通用视觉内容
```bash
# 网站/文档配图
python scripts/generate_image.py "现代办公室中团队围绕电子白板协作的专业场景" --output docs/team_collaboration.png

# 营销素材
python scripts/generate_image.py "发光神经网络构成的未来派AI大脑概念图" --output marketing/ai_concept.png
```

## 错误处理

脚本针对以下情况提供明确报错：
- API 密钥缺失（含配置指引）
- API 错误（含状态码）
- 异常响应格式
- 依赖缺失（requests 库）

运行失败时请根据错误提示解决问题后重试。

## 注意事项

- 图像以 base64 编码数据 URL 返回，自动保存为 PNG 格式
- 脚本兼容 OpenRouter 不同模型的 `images` 和 `content` 响应格式
- 生成时间因模型而异（通常 5-30 秒）
- 编辑时输入图像以 base64 编码发送至模型
- 支持的输入格式：PNG、JPEG、GIF、WebP
- 成本参考：https://openrouter.ai/models

## 图像编辑技巧

- 明确修改需求（例如"将天空改为日落色调"优于"编辑天空"）
- 尽可能指定图像中的具体元素
- 使用清晰详细的编辑指令可获得最佳效果
- Gemini 3.1 Flash Image Preview 和 FLUX.2 Pro 均通过 OpenRouter 支持图像编辑

## 技能协同

- **scientific-schematics**：用于技术图表/流程图/电路图/通路图
- **generate-image**：用于照片/插画/艺术作品/视觉概念
- **scientific-slides**：结合本技能创建视觉丰富的演示文稿
- **latex-posters**：调用本技能生成海报视觉元素与主图
```
