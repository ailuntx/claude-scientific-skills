# 机器学习

## 概述

PathML 为计算病理学提供全面的机器学习能力，包括用于细胞核检测与分割的预训练模型、PyTorch 集成的训练工作流、公共数据集访问以及基于 ONNX 的推理部署。该框架无缝桥接图像预处理与深度学习，实现端到端的病理学机器学习流程。

## 预训练模型

PathML 包含用于细胞核分析的最新预训练模型：

### HoVer-Net

**HoVer-Net**（水平垂直网络）同步执行细胞核实例分割与分类。

**架构：**
- 包含三个预测分支的编码器-解码器结构：
  - **核像素（NP）** - 细胞核区域的二值分割
  - **水平-垂直（HV）** - 到细胞核质心的距离图
  - **分类（NC）** - 细胞核类型分类

**细胞核类型：**
1. 上皮细胞
2. 炎症细胞
3. 结缔/软组织细胞
4. 死亡/坏死细胞
5. 背景

**用法：**
```python
from pathml.ml import HoVerNet
import torch

# 加载预训练模型
model = HoVerNet(
    num_types=5,  # 细胞核类型数量
    mode='fast',  # 'fast' 或 'original'
    pretrained=True  # 加载预训练权重
)

# 如果可用则移至GPU
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model = model.to(device)

# 在组织切片上进行推理
tile_image = torch.from_numpy(tile.image).permute(2, 0, 1).unsqueeze(0).float()
tile_image = tile_image.to(device)

with torch.no_grad():
    output = model(tile_image)

# 输出包含：
# - output['np']: 核像素预测
# - output['hv']: 水平-垂直图
# - output['nc']: 分类预测
```

**后处理：**
```python
from pathml.ml import hovernet_postprocess

# 将模型输出转换为实例分割
instance_map, type_map = hovernet_postprocess(
    np_pred=output['np'],
    hv_pred=output['hv'],
    nc_pred=output['nc']
)

# instance_map: 每个细胞核具有唯一ID
# type_map: 每个细胞核分配类型 (1-5)
```

### HACTNet

**HACTNet**（分层细胞类型网络）执行分层细胞核分类并支持不确定性量化。

**特性：**
- 分层分类（从粗粒度到细粒度类型）
- 预测不确定性估计
- 在数据不平衡场景下性能更优

```python
from pathml.ml import HACTNet

# 加载模型
model = HACTNet(
    num_classes_coarse=3,
    num_classes_fine=8,
    pretrained=True
)

# 推理
output = model(tile_image)
coarse_pred = output['coarse']  # 粗粒度类别
fine_pred = output['fine']  # 特定细胞类型
uncertainty = output['uncertainty']  # 预测置信度
```

## 训练工作流

### 数据集准备

PathML 提供 PyTorch 兼容的数据集类：

**TileDataset:**
```python
from pathml.ml import TileDataset
from pathml.core import SlideDataset

# 从处理后的玻片创建数据集
tile_dataset = TileDataset(
    slide_dataset,
    tile_size=256,
    transform=None  # 可选数据增强变换
)

# 访问组织切片
image, label = tile_dataset[0]
```

**DataModule 集成：**
```python
from pathml.ml import PathMLDataModule

# 创建训练/验证/测试划分
data_module = PathMLDataModule(
    train_dataset=train_tile_dataset,
    val_dataset=val_tile_dataset,
    test_dataset=test_tile_dataset,
    batch_size=32,
    num_workers=4
)

# 与 PyTorch Lightning 配合使用
trainer = pl.Trainer(max_epochs=100)
trainer.fit(model, data_module)
```

### 训练 HoVer-Net

在自定义数据上训练 HoVer-Net 的完整工作流：

```python
import torch
import torch.nn as nn
from torch.utils.data import DataLoader
from pathml.ml import HoVerNet
from pathml.ml.datasets import PanNukeDataModule

# 1. 准备数据
data_module = PanNukeDataModule(
    data_dir='path/to/pannuke',
    batch_size=8,
    num_workers=4,
    tissue_types=['乳腺', '结肠']  # 特定组织类型
)

# 2. 初始化模型
model = HoVerNet(
    num_types=5,
    mode='fast',
    pretrained=False  # 从头训练或使用 pretrained=True 进行微调
)

# 3. 定义损失函数
class HoVerNetLoss(nn.Module):
    def __init__(self):
        super().__init__()
        self.mse_loss = nn.MSELoss()
        self.bce_loss = nn.BCEWithLogitsLoss()
        self.ce_loss = nn.CrossEntropyLoss()

    def forward(self, output, target):
        # 核像素分支损失
        np_loss = self.bce_loss(output['np'], target['np'])

        # 水平-垂直分支损失
        hv_loss = self.mse_loss(output['hv'], target['hv'])

        # 分类分支损失
        nc_loss = self.ce_loss(output['nc'], target['nc'])

        # 组合损失
        total_loss = np_loss + hv_loss + 2.0 * nc_loss
        return total_loss, {'np': np_loss, 'hv': hv_loss, 'nc': nc_loss}

criterion = HoVerNetLoss()

# 4. 配置优化器
optimizer = torch.optim.Adam(
    model.parameters(),
    lr=1e-4,
    weight_decay=1e-5
)

scheduler = torch.optim.lr_scheduler.ReduceLROnPlateau(
    optimizer,
    mode='min',
    factor=0.5,
    patience=10
)

# 5. 训练循环
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model = model.to(device)

num_epochs = 100
for epoch in range(num_epochs):
    model.train()
    train_loss = 0.0

    for batch in data_module.train_dataloader():
        images = batch['image'].to(device)
        targets = {
            'np': batch['np_map'].to(device),
            'hv': batch['hv_map'].to(device),
            'nc': batch['type_map'].to(device)
        }

        optimizer.zero_grad()
        outputs = model(images)
        loss, loss_dict = criterion(outputs, targets)

        loss.backward()
        optimizer.step()

        train_loss += loss.item()

    # 验证
    model.eval()
    val_loss = 0.0
    with torch.no_grad():
        for batch in data_module.val_dataloader():
            images = batch['image'].to(device)
            targets = {
                'np': batch['np_map'].to(device),
                'hv': batch['hv_map'].to(device),
                'nc': batch['type_map'].to(device)
            }
            outputs = model(images)
            loss, _ = criterion(outputs, targets)
            val_loss += loss.item()

    scheduler.step(val_loss)

    print(f"Epoch {epoch+1}/{num_epochs}")
    print(f"  训练损失: {train_loss/len(data_module.train_dataloader()):.4f}")
    print(f"  验证损失: {val_loss/len(data_module.val_dataloader()):.4f}")

    # 保存检查点
    if (epoch + 1) % 10 == 0:
        torch.save({
            'epoch': epoch,
            'model_state_dict': model.state_dict(),
            'optimizer_state_dict': optimizer.state_dict(),
            'loss': val_loss,
        }, f'hovernet_checkpoint_epoch_{epoch+1}.pth')
```

### PyTorch Lightning 集成

PathML 模型与 PyTorch Lightning 集成实现简化训练：

```python
import pytorch_lightning as pl
from pathml.ml import HoVerNet
from pathml.ml.datasets import PanNukeDataModule

class HoVerNetModule(pl.LightningModule):
    def __init__(self, num_types=5, lr=1e-4):
        super().__init__()
        self.model = HoVerNet(num_types=num_types, pretrained=True)
        self.lr = lr
        self.criterion = HoVerNetLoss()

    def forward(self, x):
        return self.model(x)

    def training_step(self, batch, batch_idx):
        images = batch['image']
        targets = {
            'np': batch['np_map'],
            'hv': batch['hv_map'],
            'nc': batch['type_map']
        }
        outputs = self(images)
        loss, loss_dict = self.criterion(outputs, targets)

        # 记录指标
        self.log('train_loss', loss, prog_bar=True)
        for key, val in loss_dict.items():
            self.log(f'train_{key}_loss', val)

        return loss

    def validation_step(self, batch, batch_idx):
        images = batch['image']
        targets = {
            'np': batch['np_map'],
            'hv': batch['hv_map'],
            'nc': batch['type_map']
        }
        outputs = self(images)
        loss, loss_dict = self.criterion(outputs, targets)

        self.log('val_loss', loss, prog_bar=True)
        for key, val in loss_dict.items():
            self.log(f'val_{key}_loss', val)

        return loss

    def configure_optimizers(self):
        optimizer = torch.optim.Adam(self.parameters(), lr=self.lr)
        scheduler = torch.optim.lr_scheduler.ReduceLROnPlateau(
            optimizer, mode='min', factor=0.5, patience=10
        )
        return {
            'optimizer': optimizer,
            'lr_scheduler': {
                'scheduler': scheduler,
                'monitor': 'val_loss'
            }
        }

# 使用 PyTorch Lightning 训练
data_module = PanNukeDataModule(data_dir='path/to/pannuke', batch_size=8)
model = HoVerNetModule(num_types=5, lr=1e-4)

trainer = pl.Trainer(
    max_epochs=100,
    accelerator='gpu',
    devices=1,
    callbacks=[
        pl.callbacks.ModelCheckpoint(monitor='val_loss', mode='min'),
        pl.callbacks.EarlyStopping(monitor='val_loss', patience=20)
    ]
)

trainer.fit(model, data_module)
```

## 公共数据集

PathML 提供便捷的公共病理数据集访问：

### PanNuke 数据集

**PanNuke** 包含来自 19 种组织类型的 7,901 个组织学图像块，包含 5 种细胞类型的细胞核标注。

```python
from pathml.ml.datasets import PanNukeDataModule

# 加载 PanNuke 数据集
pannuke = PanNukeDataModule(
    data_dir='path/to/pannuke',
    batch_size=16,
    num_workers=4,
    tissue_types=None,  # 使用所有组织类型，或指定列表
    fold='all'  # 'fold1', 'fold2', 'fold3' 或 'all'
)

# 访问数据加载器
train_loader = pannuke.train_dataloader()
val_loader = pannuke.val_dataloader()
test_loader = pannuke.test_dataloader()

# 批次结构
for batch in train_loader:
    images = batch['image']  # 形状: (B, 3, 256, 256)
    inst_map = batch['inst_map']  # 实例分割图
    type_map = batch['type_map']  # 细胞类型图
    np_map = batch['np_map']  # 核像素图
    hv_map = batch['hv_map']  # 水平-垂直距离图
    tissue_type = batch['tissue_type']  # 组织类别
```

**可用组织类型：**
乳腺、结肠、前列腺、肺、肾、胃、膀胱、食管、宫颈、肝、甲状腺、头颈部、睾丸、肾上腺、胰腺、胆管、卵巢、皮肤、子宫

### TCGA 数据集

访问癌症基因组图谱数据集：

```python
from pathml.ml.datasets import TCGADataModule

# 加载 TCGA 数据集
tcga = TCGADataModule(
    data_dir='path/to/tcga',
    cancer_type='BRCA',  # 乳腺癌
    batch_size=32,
    tile_size=224
)
```

### 自定义数据集集成

为 PathML 工作流创建自定义数据集：

```python
from torch.utils.data import Dataset
import numpy as np
from pathlib import Path

class CustomPathologyDataset(Dataset):
    def __init__(self, data_dir, transform=None):
        self.data_dir = Path(data_dir)
        self.image_paths = list(self.data_dir.glob('images/*.png'))
        self.transform = transform

    def __len__(self):
        return len(self.image_paths)

    def __getitem__(self, idx):
        # 加载图像
        image_path = self.image_paths[idx]
        image = np.array(Image.open(image_path))

        # 加载对应标注
        annot_path = self.data_dir / 'annotations' / f'{image_path.stem}.npy'
        annotation = np.load(annot_path)

        # 应用变换
        if self.transform:
            image = self.transform(image)

        return {
            'image': torch.from_numpy(image).permute(2, 0, 1).float(),
            'annotation': torch.from_numpy(annotation).long(),
            'path': str(image_path)
        }

# 在 PathML 工作流中使用
dataset = CustomPathologyDataset('path/to/data')
dataloader = DataLoader(dataset, batch_size=16, shuffle=True, num_workers=4)
```

## 数据增强

应用增强技术提升模型泛化能力：

```python
import albumentations as A
from albumentations.pytorch import ToTensorV2

# 定义增强流程
train_transform = A.Compose([
    A.RandomRotate90(p=0.5),
    A.Flip(p=0.5),
    A.ColorJitter(brightness=0.2, contrast=0.2, saturation=0.2, hue=0.1, p=0.5),
    A.GaussianBlur(blur_limit=(3, 7), p=0.3),
    A.ElasticTransform(alpha=1, sigma=50, alpha_affine=50, p=0.3),
    A.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225]),
    ToTensorV2()
])

val_transform = A.Compose([
    A.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225]),
    ToTensorV2()
])

# 应用于数据集
train_dataset = TileDataset(slide_dataset, transform=train_transform)
val_dataset = TileDataset(val_slide_dataset, transform=val_transform)
```

## 模型评估

### 评估指标

使用病理学专用指标评估模型性能：

```python
from pathml.ml.metrics import (
    dice_coefficient,
    aggregated_jaccard_index,
    panoptic_quality
)

#

```python
model = HoVerNet(num_types=5, pretrained=True)
model.eval()

# 创建虚拟输入
dummy_input = torch.randn(1, 3, 256, 256)

# 导出为ONNX格式
torch.onnx.export(
    model,
    dummy_input,
    'hovernet_model.onnx',
    export_params=True,
    opset_version=11,
    input_names=['input'],
    output_names=['np_output', 'hv_output', 'nc_output'],
    dynamic_axes={
        'input': {0: 'batch_size'},
        'np_output': {0: 'batch_size'},
        'hv_output': {0: 'batch_size'},
        'nc_output': {0: 'batch_size'}
    }
)
```

### ONNX运行时推理

```python
import onnxruntime as ort
import numpy as np

# 加载ONNX模型
session = ort.InferenceSession('hovernet_model.onnx')

# 准备输入
input_name = session.get_inputs()[0].name
tile_image = preprocess_tile(tile)  # 归一化，转置为(1, 3, H, W)

# 运行推理
outputs = session.run(None, {input_name: tile_image})
np_output, hv_output, nc_output = outputs

# 后处理
inst_map, type_map = hovernet_postprocess(np_output, hv_output, nc_output)
```

### 批量推理流程

```python
from pathml.core import SlideData
from pathml.preprocessing import Pipeline
import onnxruntime as ort

def run_onnx_inference_pipeline(slide_path, onnx_model_path):
    # 加载切片
    wsi = SlideData.from_slide(slide_path)
    wsi.generate_tiles(level=1, tile_size=256, stride=256)

    # 加载ONNX模型
    session = ort.InferenceSession(onnx_model_path)
    input_name = session.get_inputs()[0].name

    # 对所有图块进行推理
    results = []
    for tile in wsi.tiles:
        # 预处理
        tile_array = preprocess_tile(tile.image)

        # 推理
        outputs = session.run(None, {input_name: tile_array})

        # 后处理
        inst_map, type_map = hovernet_postprocess(*outputs)

        results.append({
            'coords': tile.coords,
            'instance_map': inst_map,
            'type_map': type_map
        })

    return results

# 在切片上运行
results = run_onnx_inference_pipeline('slide.svs', 'hovernet_model.onnx')
```

## 迁移学习

在自定义数据集上微调预训练模型：

```python
from pathml.ml import HoVerNet

# 加载预训练模型
model = HoVerNet(num_types=5, pretrained=True)

# 初始训练时冻结编码器层
for name, param in model.named_parameters():
    if 'encoder' in name:
        param.requires_grad = False

# 仅微调解码器和分类头
optimizer = torch.optim.Adam(
    filter(lambda p: p.requires_grad, model.parameters()),
    lr=1e-4
)

# 训练几个周期
train_for_n_epochs(model, train_loader, optimizer, num_epochs=10)

# 解冻所有层以进行完整微调
for param in model.parameters():
    param.requires_grad = True

# 以更低的学习率继续训练
optimizer = torch.optim.Adam(model.parameters(), lr=1e-5)
train_for_n_epochs(model, train_loader, optimizer, num_epochs=50)
```

## 最佳实践

1. **优先使用预训练模型：**
   - 使用 pretrained=True 以获得更好的初始化
   - 在特定领域数据上微调

2. **应用适当的数据增强：**
   - 旋转、翻转以实现方向不变性
   - 颜色抖动以处理染色变化
   - 弹性变形以适应生物变异性

3. **监控多维度量指标：**
   - 分别跟踪检测、分割和分类性能
   - 使用领域特定指标（AJI、PQ）而不仅是标准准确率

4. **处理类别不平衡：**
   - 对稀有细胞类型使用加权损失函数
   - 对少数类进行过采样
   - 对困难样本使用焦点损失（Focal loss）

5. **在多样化组织类型上验证：**
   - 确保在不同组织上的泛化性
   - 在保留的解剖部位上测试

6. **优化推理效率：**
   - 导出为ONNX以实现更快部署
   - 批量处理图块以提高GPU利用率
   - 尽可能使用混合精度（FP16）

7. **定期保存检查点：**
   - 根据验证指标保留最佳模型
   - 保存优化器状态以便恢复训练

## 常见问题及解决方案

**问题：细胞核边界分割效果差**
- 使用HV图（水平-垂直）来分离接触的细胞核
- 增加HV损失项的权重
- 应用形态学后处理

**问题：相似细胞类型的误分类**
- 增加分类损失权重
- 添加分层分类（HACTNet）
- 为易混淆类别增加训练数据

**问题：训练不稳定或不收敛**
- 降低学习率
- 使用梯度裁剪：`torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)`
- 检查数据预处理问题

**问题：训练期间内存不足**
- 减小批量大小
- 使用梯度累积
- 启用混合精度训练：`torch.cuda.amp`

**问题：模型对训练数据过拟合**
- 增加数据增强
- 添加Dropout层
- 降低模型容量
- 基于验证损失使用早停

## 附加资源

- **PathML ML API：** https://pathml.readthedocs.io/en/latest/api_ml_reference.html
- **HoVer-Net论文：** Graham et al., "HoVer-Net: Simultaneous Segmentation and Classification of Nuclei in Multi-Tissue Histology Images," Medical Image Analysis, 2019
- **PanNuke数据集：** https://warwick.ac.uk/fac/cross_fac/tia/data/pannuke
- **PyTorch Lightning：** https://www.pytorchlightning.ai/
- **ONNX Runtime：** https://onnxruntime.ai/
