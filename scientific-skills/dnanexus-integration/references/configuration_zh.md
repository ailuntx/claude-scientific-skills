# DNAnexus 应用配置与依赖管理

## 概述

本指南涵盖通过 dxapp.json 元数据配置应用，以及管理系统包、Python 库和 Docker 容器等依赖项。

## dxapp.json 结构

`dxapp.json` 是 DNAnexus 应用和应用小程序的配置文件，用于定义元数据、输入输出、执行要求和依赖项。

### 最小化示例

```json
{
  "name": "my-app",
  "title": "我的分析应用",
  "summary": "对输入文件执行分析",
  "dxapi": "1.0.0",
  "version": "1.0.0",
  "inputSpec": [],
  "outputSpec": [],
  "runSpec": {
    "interpreter": "python3",
    "file": "src/my-app.py",
    "distribution": "Ubuntu",
    "release": "24.04"
  }
}
```

## 元数据字段

### 必填字段

```json
{
  "name": "my-app",           // 唯一标识符（小写字母、数字、连字符、下划线）
  "title": "我的应用",         // 人类可读名称
  "summary": "单行描述",
  "dxapi": "1.0.0"            // API 版本
}
```

### 可选元数据

```json
{
  "version": "1.0.0",         // 语义化版本（应用必需）
  "description": "详细描述...",
  "developerNotes": "实现说明...",
  "categories": [             // 用于应用发现
    "读段比对",
    "变异检测"
  ],
  "details": {                // 自定义元数据
    "contactEmail": "dev@example.com",
    "upstreamVersion": "2.1.0",
    "citations": ["doi:10.1000/example"],
    "changelog": {
      "1.0.0": "初始版本"
    }
  }
}
```

## 输入规范

定义输入参数：

```json
{
  "inputSpec": [
    {
      "name": "reads",
      "label": "输入读段",
      "class": "file",
      "patterns": ["*.fastq", "*.fastq.gz"],
      "optional": false,
      "help": "包含测序读段的 FASTQ 文件"
    },
    {
      "name": "quality_threshold",
      "label": "质量阈值",
      "class": "int",
      "default": 30,
      "optional": true,
      "help": "最低碱基质量分数"
    },
    {
      "name": "reference",
      "label": "参考基因组",
      "class": "file",
      "patterns": ["*.fa", "*.fasta"],
      "suggestions": [
        {
          "name": "人类 GRCh38",
          "project": "project-xxxx",
          "path": "/references/human_g1k_v37.fasta"
        }
      ]
    }
  ]
}
```

### 输入类型

- `file` - 文件对象
- `record` - 记录对象
- `applet` - 小程序引用
- `string` - 文本字符串
- `int` - 整型数值
- `float` - 浮点数值
- `boolean` - 布尔值
- `hash` - 键值映射
- `array:class` - 指定类型的数组

### 输入选项

- `name` - 参数名称（必需）
- `class` - 数据类型（必需）
- `optional` - 是否可选（默认：false）
- `default` - 可选参数的默认值
- `label` - 界面显示名称
- `help` - 描述文本
- `patterns` - 文件名匹配模式（针对文件）
- `suggestions` - 预定义参考数据
- `choices` - 允许值（针对字符串/数字）
- `group` - 界面分组

## 输出规范

定义输出参数：

```json
{
  "outputSpec": [
    {
      "name": "aligned_reads",
      "label": "比对读段",
      "class": "file",
      "patterns": ["*.bam"],
      "help": "包含比对读段的 BAM 文件"
    },
    {
      "name": "mapping_stats",
      "label": "比对统计",
      "class": "record",
      "help": "包含比对统计数据的记录"
    }
  ]
}
```

## 运行规范

定义应用执行方式：

```json
{
  "runSpec": {
    "interpreter": "python3",        // 或 "bash"
    "file": "src/my-app.py",         // 入口脚本
    "distribution": "Ubuntu",
    "release": "24.04",
    "version": "0",                  // 发行版版本
    "execDepends": [                 // 系统包
      {"name": "samtools"},
      {"name": "bwa"}
    ],
    "bundledDepends": [              // 捆绑资源
      {"name": "scripts.tar.gz", "id": {"$dnanexus_link": "file-xxxx"}}
    ],
    "assetDepends": [                // 资产依赖
      {"name": "asset-name", "id": {"$dnanexus_link": "record-xxxx"}}
    ],
    "systemRequirements": {
      "*": {
        "instanceType": "mem2_ssd1_v2_x4"
      }
    },
    "headJobOnDemand": true,
    "restartableEntryPoints": ["main"]
  }
}
```

## 系统要求

### 实例类型选择

```json
{
  "systemRequirements": {
    "main": {
      "instanceType": "mem2_ssd1_v2_x8"
    },
    "process": {
      "instanceType": "mem3_ssd1_v2_x16"
    }
  }
}
```

**常用实例类型**：
- `mem1_ssd1_v2_x2` - 2核, 3.9 GB 内存
- `mem1_ssd1_v2_x4` - 4核, 7.8 GB 内存
- `mem2_ssd1_v2_x4` - 4核, 15.6 GB 内存
- `mem2_ssd1_v2_x8` - 8核, 31.2 GB 内存
- `mem3_ssd1_v2_x8` - 8核, 62.5 GB 内存
- `mem3_ssd1_v2_x16` - 16核, 125 GB 内存

### 集群规格

分布式计算配置：

```json
{
  "systemRequirements": {
    "main": {
      "clusterSpec": {
        "type": "spark",
        "version": "3.1.2",
        "initialInstanceCount": 3,
        "instanceType": "mem1_ssd1_v2_x4",
        "bootstrapScript": "bootstrap.sh"
      }
    }
  }
}
```

## 区域选项

跨区域部署应用：

```json
{
  "regionalOptions": {
    "aws:us-east-1": {
      "systemRequirements": {
        "*": {"instanceType": "mem2_ssd1_v2_x4"}
      },
      "assetDepends": [
        {"id": "record-xxxx"}
      ]
    },
    "azure:westus": {
      "systemRequirements": {
        "*": {"instanceType": "azure:mem2_ssd1_x4"}
      }
    }
  }
}
```

## 依赖管理

### 系统包 (execDepends)

运行时安装 Ubuntu 包：

```json
{
  "runSpec": {
    "execDepends": [
      {"name": "samtools"},
      {"name": "bwa"},
      {"name": "python3-pip"},
      {"name": "r-base", "version": "4.0.0"}
    ]
  }
}
```

包通过 `apt-get` 从 Ubuntu 仓库安装。

### Python 依赖

#### 方案1：通过 execDepends 安装 pip

```json
{
  "runSpec": {
    "execDepends": [
      {"name": "python3-pip"}
    ]
  }
}
```

在应用脚本中：
```python
import subprocess
subprocess.check_call(["pip", "install", "numpy==1.24.0", "pandas==2.0.0"])
```

#### 方案2：使用 requirements 文件

创建 `resources/requirements.txt`：
```
numpy==1.24.0
pandas==2.0.0
scikit-learn==1.3.0
```

在应用中：
```python
subprocess.check_call(["pip", "install", "-r", "requirements.txt"])
```

### 捆绑依赖

将自定义工具或库包含在应用中：

**文件结构**：
```
my-app/
├── dxapp.json
├── src/
│   └── my-app.py
└── resources/
    ├── tools/
    │   └── custom_tool
    └── scripts/
        └── helper.py
```

在应用中访问资源：
```python
import os

# 资源位于上级目录
resources_dir = os.path.join(os.path.dirname(__file__), "..", "resources")
tool_path = os.path.join(resources_dir, "tools", "custom_tool")

# 运行捆绑工具
subprocess.check_call([tool_path, "arg1", "arg2"])
```

### 资产依赖

资产是可跨应用共享的预构建依赖包。

#### 使用资产

```json
{
  "runSpec": {
    "assetDepends": [
      {
        "name": "bwa-asset",
        "id": {"$dnanexus_link": "record-xxxx"}
      }
    ]
  }
}
```

资产在运行时挂载，通过环境变量访问：
```python
import os
asset_dir = os.environ.get("DX_ASSET_BWA")
bwa_path = os.path.join(asset_dir, "bin", "bwa")
```

#### 创建资产

创建资产目录：
```bash
mkdir bwa-asset
cd bwa-asset
# 安装软件
./configure --prefix=$PWD/usr/local
make && make install
```

构建资产：
```bash
dx build_asset bwa-asset --destination=project-xxxx:/assets/
```

## Docker 集成

### 使用 Docker 镜像

```json
{
  "runSpec": {
    "interpreter": "python3",
    "file": "src/my-app.py",
    "distribution": "Ubuntu",
    "release": "24.04",
    "systemRequirements": {
      "*": {
        "instanceType": "mem2_ssd1_v2_x4"
      }
    },
    "execDepends": [
      {"name": "docker.io"}
    ]
  }
}
```

在应用中使用 Docker：
```python
import subprocess

# 拉取 Docker 镜像
subprocess.check_call(["docker", "pull", "biocontainers/samtools:v1.9"])

# 在容器中运行命令
subprocess.check_call([
    "docker", "run",
    "-v", f"{os.getcwd()}:/data",
    "biocontainers/samtools:v1.9",
    "samtools", "view", "/data/input.bam"
])
```

### 以 Docker 为基础镜像

完全在 Docker 中运行的应用：

```json
{
  "runSpec": {
    "interpreter": "bash",
    "file": "src/wrapper.sh",
    "distribution": "Ubuntu",
    "release": "24.04",
    "execDepends": [
      {"name": "docker.io"}
    ]
  }
}
```

## 访问权限要求

申请特殊权限：

```json
{
  "access": {
    "network": ["*"],           // 互联网访问
    "project": "CONTRIBUTE",    // 项目写入权限
    "allProjects": "VIEW",      // 读取其他项目
    "developer": true           // 高级权限
  }
}
```

**网络访问**：
- `["*"]` - 完全互联网访问
- `["github.com", "pypi.org"]` - 指定域名

## 超时配置

```json
{
  "runSpec": {
    "timeoutPolicy": {
      "*": {
        "days": 1,
        "hours": 12,
        "minutes": 30
      }
    }
  }
}
```

## 示例：完整 dxapp.json

```json
{
  "name": "rna-seq-pipeline",
  "title": "RNA-Seq 分析流程",
  "summary": "比对 RNA-seq 读段并量化基因表达",
  "description": "使用 STAR 比对器和 featureCounts 的完整 RNA-seq 流程",
  "version": "1.0.0",
  "dxapi": "1.0.0",
  "categories": ["读段比对", "RNA-Seq"],

  "inputSpec": [
    {
      "name": "reads",
      "label": "FASTQ 读段",
      "class": "array:file",
      "patterns": ["*.fastq.gz", "*.fq.gz"],
      "help": "单端或双端 RNA-seq 读段"
    },
    {
      "name": "reference_genome",
      "label": "参考基因组",
      "class": "file",
      "patterns": ["*.fa", "*.fasta"],
      "suggestions": [
        {
          "name": "人类 GRCh38",
          "project": "project-reference",
          "path": "/genomes/GRCh38.fa"
        }
      ]
    },
    {
      "name": "gtf_file",
      "label": "基因注释 (GTF)",
      "class": "file",
      "patterns": ["*.gtf", "*.gtf.gz"]
    }
  ],

  "outputSpec": [
    {
      "name": "aligned_bam",
      "label": "比对读段 (BAM)",
      "class": "file",
      "patterns": ["*.bam"]
    },
    {
      "name": "counts",
      "label": "基因计数",
      "class": "file",
      "patterns": ["*.counts.txt"]
    },
    {
      "name": "qc_report",
      "label": "质控报告",
      "class": "file",
      "patterns": ["*.html"]
    }
  ],

  "runSpec": {
    "interpreter": "python3",
    "file": "src/rna-seq-pipeline.py",
    "distribution": "Ubuntu",
    "release": "24.04",

    "execDepends": [
      {"name": "python3-pip"},
      {"name": "samtools"},
      {"name": "subread"}
    ],

    "assetDepends": [
      {
        "name": "star-aligner",
        "id": {"$dnanexus_link": "record-star-asset"}
      }
    ],

    "systemRequirements": {
      "main": {
        "instanceType": "mem3_ssd1_v2_x16"
      }
    },

    "timeoutPolicy": {
      "*": {"hours": 8}
    }
  },

  "access": {
    "network": ["*"]
  },

  "details": {
    "contactEmail": "support@example.com",
    "upstreamVersion": "STAR 2.7.10a, Subread 2.0.3",
    "citations": ["doi:10.1093/bioinformatics/bts635"]
  }
}
```

## 最佳实践

1. **版本管理**：为应用使用语义化版本控制
2. **实例类型**：从小型实例开始，按需扩展
3. **依赖项**：清晰记录所有依赖项
4. **错误信息**：为无效输入提供有用的错误提示
5. **测试**：使用多种输入类型和规模进行测试
6. **文档**：编写清晰的描述和帮助文本
7. **资源**：捆绑常用工具避免重复下载
8. **Docker**：对复杂依赖链使用 Docker
9. **资产**：为跨应用共享的重型依赖创建资产
10. **超时设置**：根据预期运行时间设置合理超时
11. **网络访问**：仅申请必要的网络权限
12. **区域支持**：多区域应用使用 regionalOptions

## 常见模式

### 生物信息学工具

```json
{
  "inputSpec": [
    {"name": "input_file", "class": "file", "patterns": ["*.bam"]},
    {"name": "threads", "class": "int", "default": 4, "optional": true}
  ],
  "runSpec": {

"execDepends": [{"name": "tool-name"}],
    "systemRequirements": {
      "main": {"instanceType": "mem2_ssd1_v2_x8"}
    }
  }
}
```

### Python 数据分析

```json
{
  "runSpec": {
    "interpreter": "python3",
    "execDepends": [
      {"name": "python3-pip"}
    ],
    "systemRequirements": {
      "main": {"instanceType": "mem2_ssd1_v2_x4"}
    }
  }
}
```

### 基于 Docker 的应用

```json
{
  "runSpec": {
    "interpreter": "bash",
    "execDepends": [
      {"name": "docker.io"}
    ],
    "systemRequirements": {
      "main": {"instanceType": "mem2_ssd1_v2_x8"}
    }
  },
  "access": {
    "network": ["*"]
  }
}
```
