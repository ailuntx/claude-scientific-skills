# 科学示意图 - 快速参考

**工作原理：** 描述您的示意图 → Nano Banana 2 自动生成

## 初始设置（一次性）

```bash
# 从 https://openrouter.ai/keys 获取 API 密钥
export OPENROUTER_API_KEY='sk-or-v1-your_key_here'

# 添加到 shell 配置文件实现持久化
echo 'export OPENROUTER_API_KEY="sk-or-v1-your_key"' >> ~/.bashrc  # 或 ~/.zshrc
```

## 基础用法

```bash
# 描述您的示意图，Nano Banana 2 将创建它
python scripts/generate_schematic.py "您的示意图描述" -o output.png

# 完成！自动执行：
# - 迭代优化（3轮）
# - 质量审查与改进
# - 生成可直接发表的输出
```

## 常见示例

### CONSORT 流程图
```bash
python scripts/generate_schematic.py \
  "CONSORT流程：筛选 n=500，排除 n=150，随机分组 n=350" \
  -o consort.png
```

### 神经网络
```bash
python scripts/generate_schematic.py \
  "包含编码器和解码器堆栈的 Transformer 架构" \
  -o transformer.png
```

### 生物通路
```bash
python scripts/generate_schematic.py \
  "MAPK通路：EGFR → RAS → RAF → MEK → ERK" \
  -o mapk.png
```

### 电路图
```bash
python scripts/generate_schematic.py \
  "包含1kΩ电阻和10µF电容的运算放大器电路" \
  -o circuit.png
```

## 命令选项

| 选项 | 描述 | 示例 |
|--------|-------------|---------|
| `-o, --output` | 输出文件路径 | `-o figures/diagram.png` |
| `--iterations N` | 优化轮数 (1-2) | `--iterations 2` |
| `-v, --verbose` | 显示详细输出 | `-v` |
| `--api-key KEY` | 提供 API 密钥 | `--api-key sk-or-v1-...` |

## 提示词技巧

### ✓ 优质提示（具体明确）
- "CONSORT流程图：筛选(n=500)、排除(n=150)、随机分组(n=350)"
- "Transformer架构：左侧6层编码器，右侧解码器，带交叉注意力连接"
- "MAPK信号通路：受体→RAS→RAF→MEK→ERK→细胞核，标注每次磷酸化"

### ✗ 避免（过于模糊）
- "做个流程图"
- "神经网络"
- "通路示意图"

## 输出文件

输入 `diagram.png` 将生成：
- `diagram_v1.png` - 第一轮迭代
- `diagram_v2.png` - 第二轮迭代  
- `diagram_v3.png` - 最终迭代
- `diagram.png` - 最终版本副本
- `diagram_review_log.json` - 质量评分与改进建议

## 审查日志

```json
{
  "iterations": [
    {
      "iteration": 1,
      "score": 7.0,
      "critique": "良好起点。字体过小..."
    },
    {
      "iteration": 2,
      "score": 8.5,
      "critique": "显著改进。存在轻微间距问题..."
    },
    {
      "iteration": 3,
      "score": 9.5,
      "critique": "优秀。达到发表标准。"
    }
  ],
  "final_score": 9.5
}
```

## Python API

```python
from scripts.generate_schematic_ai import ScientificSchematicGenerator

# 初始化
gen = ScientificSchematicGenerator(api_key="your_key")

# 生成
results = gen.generate_iterative(
    user_prompt="示意图描述",
    output_path="output.png",
    iterations=2
)

# 检查质量
print(f"评分: {results['final_score']}/10")
```

## 故障排除

### API 密钥未找到
```bash
# 检查是否设置
echo $OPENROUTER_API_KEY

# 设置密钥
export OPENROUTER_API_KEY='your_key'
```

### 导入错误
```bash
# 安装 requests
pip install requests
```

### 质量评分低
- 使提示词更具体
- 包含布局细节（左到右，上到下）
- 明确标注要求
- 增加迭代次数：`--iterations 2`

## 测试

```bash
# 验证安装
python test_ai_generation.py

# 应显示："6/6 测试通过"
```

## 成本

单张示意图典型成本（最多2轮迭代）：
- 简单（1轮迭代）：$0.05-0.15
- 复杂（2轮迭代）：$0.10-0.30

## Nano Banana 2 工作原理

**只需用自然语言描述您的示意图：**
- ✓ 无需编程
- ✓ 无需模板
- ✓ 无需手动绘制
- ✓ 自动质量审查
- ✓ 直接可发表的输出
- ✓ 支持任意示意图类型

**描述需求，自动生成结果。**

## 获取帮助

```bash
# 显示帮助
python scripts/generate_schematic.py --help

# 调试用详细模式
python scripts/generate_schematic.py "示意图" -o out.png -v
```

## 快速入门清单

- [ ] 设置 `OPENROUTER_API_KEY` 环境变量
- [ ] 运行 `python test_ai_generation.py`（应通过6/6测试）
- [ ] 尝试：`python scripts/generate_schematic.py "测试示意图" -o test.png`
- [ ] 检查输出文件（test_v1.png, v2, v3, review_log.json）
- [ ] 阅读 SKILL.md 获取详细文档
- [ ] 查看 README.md 中的示例

## 资源

- 完整文档：`SKILL.md`
- 详细指南：`README.md`
- 实现细节：`IMPLEMENTATION_SUMMARY.md`
- 示例脚本：`example_usage.sh`
- 获取 API 密钥：https://openrouter.ai/keys
