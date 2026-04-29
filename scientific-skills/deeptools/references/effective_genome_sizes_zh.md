# 有效基因组大小

## 定义

有效基因组大小指"可映射"基因组的长度——即测序读段能够唯一映射的区域。该指标对许多deepTools命令中的标准化处理至关重要。

## 重要性

- 是RPGC标准化(`--normalizeUsing RPGC`)的必要参数
- 影响覆盖度计算的准确性
- 必须与数据处理方法匹配（过滤与未过滤读段）

## 计算方法

1. **非N碱基法**：统计基因组序列中非N核苷酸的数量
2. **唯一可映射性**：特定大小下可被唯一映射的区域（可考虑编辑距离）

## 常见生物体参考值

### 非N碱基法

| 生物体 | 基因组版本 | 有效大小 | 完整命令 |
|----------|----------|----------------|--------------|
| 人类 | GRCh38/hg38 | 2,913,022,398 | `--effectiveGenomeSize 2913022398` |
| 人类 | GRCh37/hg19 | 2,864,785,220 | `--effectiveGenomeSize 2864785220` |
| 小鼠 | GRCm39/mm39 | 2,654,621,837 | `--effectiveGenomeSize 2654621837` |
| 小鼠 | GRCm38/mm10 | 2,652,783,500 | `--effectiveGenomeSize 2652783500` |
| 斑马鱼 | GRCz11 | 1,368,780,147 | `--effectiveGenomeSize 1368780147` |
| 果蝇 | dm6 | 142,573,017 | `--effectiveGenomeSize 142573017` |
| 线虫 | WBcel235/ce11 | 100,286,401 | `--effectiveGenomeSize 100286401` |
| 线虫 | ce10 | 100,258,171 | `--effectiveGenomeSize 100258171` |

### 人类(GRCh38)按读长分类

经质量过滤的读段，数值随读长变化：

| 读长 | 有效大小 |
|-------------|----------------|
| 50bp | ~27亿 |
| 75bp | ~28亿 |
| 100bp | ~28亿 |
| 150bp | ~29亿 |
| 250bp | ~29亿 |

### 小鼠(GRCm38)按读长分类

| 读长 | 有效大小 |
|-------------|----------------|
| 50bp | ~23亿 |
| 75bp | ~25亿 |
| 100bp | ~26亿 |

## 在deepTools中的应用

有效基因组大小最常用于以下场景：

### bamCoverage的RPGC标准化
```bash
bamCoverage --bam input.bam --outFileName output.bw \
    --normalizeUsing RPGC \
    --effectiveGenomeSize 2913022398
```

### bamCompare的RPGC标准化
```bash
bamCompare -b1 treatment.bam -b2 control.bam \
    --outFileName comparison.bw \
    --scaleFactorsMethod RPGC \
    --effectiveGenomeSize 2913022398
```

### computeGCBias / correctGCBias
```bash
computeGCBias --bamfile input.bam \
    --effectiveGenomeSize 2913022398 \
    --genome genome.2bit \
    --fragmentLength 200 \
    --biasPlot bias.png
```

## 参数选择指南

**常规分析**：使用参考基因组的非N碱基法数值  
**过滤数据**：若采用严格质量过滤或移除多重映射读段，建议使用读长特异性数值  
**不确定时**：采用保守的非N碱基法数值——适用性更广  

## 常用快捷参数

deepTools部分场景支持简写值：
- `hs` 或 `GRCh38`: 2913022398
- `mm` 或 `GRCm38`: 2652783500
- `dm` 或 `dm6`: 142573017
- `ce` 或 `ce10`: 100286401

具体支持情况请查阅对应deepTools版本的文档。

## 自定义参数计算

针对自定义基因组或组装版本，计算非N碱基数：
```bash
# 使用faCount(UCSC工具)
faCount genome.fa | grep "total" | awk '{print $2-$7}'

# 使用seqtk
seqtk comp genome.fa | awk '{x+=$2}END{print x}'
```

## 参考文献

获取最新有效基因组大小及详细计算方法：
- deepTools文档：https://deeptools.readthedocs.io/en/latest/content/feature/effectiveGenomeSize.html
- ENCODE参考基因组详情文档
