# 科学示意图生成器 - Nano Banana 2

**通过自然语言描述生成任意科学示意图**

Nano Banana 2 自动创建出版物级示意图——无需编码、无需模板、无需手动绘制。

## 快速入门

### 生成任意示意图

```bash
# 设置您的 OpenRouter API 密钥
export OPENROUTER_API_KEY='your_api_key_here'

# 生成任意科学示意图
python scripts/generate_schematic.py "CONSORT受试者流程图" -o figures/consort.png

# 神经网络架构
python scripts/generate_schematic.py "Transformer编码器-解码器架构" -o figures/transformer.png

# 生物通路
python scripts/generate_schematic.py "MAPK信号通路" -o figures/pathway.png
```

### 您将获得

- **最多两次迭代**（v1, v2）的渐进优化
- 每次迭代后的**自动质量评审**
- 包含评分与改进建议的**详细评审日志**（JSON格式）
- 符合科学出版标准的**即用型图像**

## 核心功能

### 迭代优化流程

1. **第一代生成**：根据描述创建初始示意图
2. **第一轮评审**：AI评估清晰度、标注、准确性和可访问性
3. **第二代生成**：基于评审意见改进
4. **第二轮评审**：提供具体反馈的二次评估
5. **第三代生成**：最终优化版本

### 自动质量标准

所有示意图自动遵循：
- 纯净白/浅色背景
- 高对比度确保可读性
- 清晰标注（最小10磅字体）
- 专业排版
- 色盲友好配色
- 元素间合理间距
- 按需添加比例尺、图例和坐标轴

## 安装指南

### AI生成环境配置

```bash
# 获取OpenRouter API密钥
# 访问：https://openrouter.ai/keys

# 设置环境变量
export OPENROUTER_API_KEY='sk-or-v1-...'

# 或添加到.env文件
echo "OPENROUTER_API_KEY=sk-or-v1-..." >> .env

# 安装Python依赖（如未安装）
pip install requests
```

## 使用示例

### 示例1：CONSORT流程图

```bash
python scripts/generate_schematic.py \
  "随机对照试验CONSORT受试者流程图。\
   资格评估(n=500)。\
   排除(n=150)：年龄<18岁(n=80)，拒绝参与(n=50)，其他原因(n=20)。\
   随机分组(n=350)：治疗组(n=175)与对照组(n=175)。\
   失访：分别15例与10例。\
   最终分析：160例与165例。" \
  -o figures/consort.png
```

**输出：**
- `figures/consort_v1.png` - 初始版本
- `figures/consort_v2.png` - 首次优化后
- `figures/consort_v3.png` - 最终版本
- `figures/consort.png` - 最终版本副本
- `figures/consort_review_log.json` - 详细评审日志

### 示例2：神经网络架构

```bash
python scripts/generate_schematic.py \
  "左侧编码器（输入嵌入、位置编码、多头注意力、前馈网络）与\
   右侧解码器（掩码注意力、交叉注意力、前馈网络）组成的Transformer架构。\
   显示编码器到解码器的交叉注意力连接。" \
  -o figures/transformer.png \
  --iterations 2
```

### 示例3：生物通路图

```bash
python scripts/generate_schematic.py \
  "MAPK信号通路：EGFR受体→RAS→RAF→MEK→ERK→细胞核。\
   标注各步骤磷酸化过程，不同激酶使用不同颜色。" \
  -o figures/mapk.png
```

### 示例4：系统架构图

```bash
python scripts/generate_schematic.py \
  "物联网系统框图：传感器（底部）→微控制器→\
   WiFi模块与显示屏（中部）→云服务器→移动应用（顶部）。\
   所有连接标注通信协议。" \
  -o figures/iot_system.png
```

## 命令行参数

```bash
python scripts/generate_schematic.py [选项] "描述文本" -o 输出.png

选项：
  --iterations N         AI优化迭代次数（默认：2，最大：2）
  --api-key KEY          OpenRouter API密钥（或使用环境变量）
  -v, --verbose          显示详细输出
  -h, --help             显示帮助信息
```

## Python API

```python
from scripts.generate_schematic_ai import ScientificSchematicGenerator

# 初始化
generator = ScientificSchematicGenerator(
    api_key="your_key",
    verbose=True
)

# 迭代生成示意图
results = generator.generate_iterative(
    user_prompt="CONSORT流程图",
    output_path="figures/consort.png",
    iterations=2
)

# 获取结果
print(f"最终评分: {results['final_score']}/10")
print(f"最终图像: {results['final_image']}")

# 查看迭代记录
for iteration in results['iterations']:
    print(f"迭代{iteration['iteration']}: {iteration['score']}/10")
    print(f"评审意见: {iteration['critique']}")
```

## 提示词设计技巧

### 明确布局要求
✓ "垂直流向的流程图，自上而下"  
✓ "左侧编码器、右侧解码器的架构图"  
✗ "做个示意图"（过于模糊）

### 包含量化细节
✓ "神经网络：输入层(784)，隐藏层(128)，输出层(10)"  
✓ "流程图：筛选n=500，排除n=150，随机分组n=350"  
✗ "一些数字"（不具体）

### 指定视觉风格
✓ "简洁线条的极简框图"  
✓ "包含蛋白质结构的详细生物通路图"  
✓ "采用工程符号的技术示意图"

### 要求特定标注
✓ "所有箭头标注激活/抑制关系"  
✓ "每个框内显示层维度"  
✓ "通过时间戳展示进程"

### 声明色彩需求
✓ "使用色盲友好配色"  
✓ "兼容灰度显示的设计"  
✓ "按功能分色：蓝色=输入，绿色=处理，红色=输出"

## 评审日志格式

每次生成产生JSON格式评审日志：

```json
{
  "user_prompt": "CONSORT受试者流程图...",
  "iterations": [
    {
      "iteration": 1,
      "image_path": "figures/consort_v1.png",
      "prompt": "完整生成提示词...",
      "critique": "评分：7/10。问题：字体过小...",
      "score": 7.0,
      "success": true
    },
    {
      "iteration": 2,
      "image_path": "figures/consort_v2.png",
      "score": 8.5,
      "critique": "显著改进。遗留问题..."
    },
    {
      "iteration": 3,
      "image_path": "figures/consort_v3.png",
      "score": 9.5,
      "critique": "优秀。达到出版标准"
    }
  ],
  "final_image": "figures/consort_v3.png",
  "final_score": 9.5,
  "success": true
}
```

## 选择Nano Banana 2的理由

**简单描述需求——Nano Banana 2自动实现：**

- ✓ **高效**：分钟级出图
- ✓ **易用**：自然语言描述（无需编程）
- ✓ **优质**：自动评审与优化
- ✓ **通用**：支持所有示意图类型
- ✓ **出版级**：即时输出高质量结果

**只需描述您的示意图，即可自动生成。**

## 故障排除

### API密钥问题

```bash
# 检查密钥设置
echo $OPENROUTER_API_KEY

# 临时设置
export OPENROUTER_API_KEY='your_key'

# 永久设置（添加到~/.bashrc或~/.zshrc）
echo 'export OPENROUTER_API_KEY="your_key"' >> ~/.bashrc
```

### 导入错误

```bash
# 安装requests库
pip install requests

# 使用包管理器安装
pip install -r requirements.txt
```

### 生成失败

```bash
# 使用详细模式查看错误
python scripts/generate_schematic.py "示意图" -o out.png -v

# 检查API状态
curl https://openrouter.ai/api/v1/models
```

### 低质量评分

若迭代评分持续低于7/10：
1. 增加提示词具体性
2. 补充布局和标注细节
3. 明确视觉要求
4. 增加迭代次数：`--iterations 2`

## 测试验证

运行测试脚本：

```bash
python test_ai_generation.py
```

测试范围包括：
- 文件结构
- 模块导入
- 类初始化
- 错误处理
- 提示词工程
- 封装脚本

## 成本说明

OpenRouter模型使用费用：
- **Nano Banana 2**：输入Token约$2/百万，输出Token约$12/百万

典型示意图成本：
- 简单示意图（1次迭代）：$0.05-0.15
- 复杂示意图（2次迭代）：$0.10-0.30

## 示例图库

完整示例参见SKILL.md，包含：
- CONSORT流程图
- 神经网络架构（Transformer、CNN、RNN）
- 生物通路图
- 电路示意图
- 系统架构图
- 功能框图

## 技术支持

问题咨询流程：
1. 查阅SKILL.md获取详细文档
2. 运行test_ai_generation.py验证环境
3. 使用详细模式(-v)查看错误
4. 分析review_log.json获取质量反馈

## 许可协议

本工具属于scientific-writer套件，许可信息详见主仓库。
