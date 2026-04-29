# 地理空间数据的机器学习

面向遥感与空间分析的机器学习和深度学习应用指南。

## 传统机器学习

### 用于土地覆盖分类的随机森林

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report, confusion_matrix
import rasterio
from rasterio.features import rasterize
import geopandas as gpd
import numpy as np

def train_random_forest_classifier(raster_path, training_gdf):
    """训练用于图像分类的随机森林模型。"""

    # 加载影像
    with rasterio.open(raster_path) as src:
        image = src.read()
        profile = src.profile
        transform = src.transform

    # 提取训练数据
    X, y = [], []

    for _, row in training_gdf.iterrows():
        mask = rasterize(
            [(row.geometry, 1)],
            out_shape=(profile['height'], profile['width']),
            transform=transform,
            fill=0,
            dtype=np.uint8
        )
        pixels = image[:, mask > 0].T
        X.extend(pixels)
        y.extend([row['class_id']] * len(pixels))

    X = np.array(X)
    y = np.array(y)

    # 分割数据
    X_train, X_val, y_train, y_val = train_test_split(
        X, y, test_size=0.2, random_state=42, stratify=y
    )

    # 训练模型
    rf = RandomForestClassifier(
        n_estimators=100,
        max_depth=20,
        min_samples_split=10,
        min_samples_leaf=4,
        class_weight='balanced',
        n_jobs=-1,
        random_state=42
    )
    rf.fit(X_train, y_train)

    # 验证
    y_pred = rf.predict(X_val)
    print("分类报告:")
    print(classification_report(y_val, y_pred))

    # 特征重要性
    feature_names = [f'Band_{i}' for i in range(X.shape[1])]
    importances = pd.DataFrame({
        'feature': feature_names,
        'importance': rf.feature_importances_
    }).sort_values('importance', ascending=False)

    print("\n特征重要性:")
    print(importances)

    return rf

# 全图分类
def classify_image(model, image_path, output_path):
    with rasterio.open(image_path) as src:
        image = src.read()
        profile = src.profile

    image_reshaped = image.reshape(image.shape[0], -1).T
    prediction = model.predict(image_reshaped)
    prediction = prediction.reshape(image.shape[1], image.shape[2])

    profile.update(dtype=rasterio.uint8, count=1)
    with rasterio.open(output_path, 'w', **profile) as dst:
        dst.write(prediction.astype(rasterio.uint8), 1)
```

### 支持向量机

```python
from sklearn.svm import SVC
from sklearn.preprocessing import StandardScaler

def svm_classifier(X_train, y_train):
    """用于遥感数据的SVM分类器。"""

    # 特征缩放
    scaler = StandardScaler()
    X_train_scaled = scaler.fit_transform(X_train)

    # 训练SVM
    svm = SVC(
        kernel='rbf',
        C=100,
        gamma='scale',
        class_weight='balanced',
        probability=True
    )
    svm.fit(X_train_scaled, y_train)

    return svm, scaler

# 多类别分类
def multiclass_svm(X_train, y_train):
    from sklearn.multiclass import OneVsRestClassifier

    scaler = StandardScaler()
    X_train_scaled = scaler.fit_transform(X_train)

    svm_ovr = OneVsRestClassifier(
        SVC(kernel='rbf', C=10, probability=True),
        n_jobs=-1
    )
    svm_ovr.fit(X_train_scaled, y_train)

    return svm_ovr, scaler
```

## 深度学习

### 使用TorchGeo的卷积神经网络

```python
import torch
import torch.nn as nn
import torchgeo.datasets as datasets
import torchgeo.models as models
from torch.utils.data import DataLoader

# 定义CNN
class LandCoverCNN(nn.Module):
    def __init__(self, in_channels=12, num_classes=10):
        super().__init__()
        self.encoder = nn.Sequential(
            nn.Conv2d(in_channels, 64, 3, padding=1),
            nn.BatchNorm2d(64),
            nn.ReLU(),
            nn.MaxPool2d(2),

            nn.Conv2d(64, 128, 3, padding=1),
            nn.BatchNorm2d(128),
            nn.ReLU(),
            nn.MaxPool2d(2),

            nn.Conv2d(128, 256, 3, padding=1),
            nn.BatchNorm2d(256),
            nn.ReLU(),
            nn.MaxPool2d(2),
        )

        self.decoder = nn.Sequential(
            nn.ConvTranspose2d(256, 128, 2, stride=2),
            nn.BatchNorm2d(128),
            nn.ReLU(),

            nn.ConvTranspose2d(128, 64, 2, stride=2),
            nn.BatchNorm2d(64),
            nn.ReLU(),

            nn.ConvTranspose2d(64, num_classes, 2, stride=2),
        )

    def forward(self, x):
        x = self.encoder(x)
        x = self.decoder(x)
        return x

# 训练
def train_model(train_loader, val_loader, num_epochs=50):
    device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
    model = LandCoverCNN().to(device)

    criterion = nn.CrossEntropyLoss()
    optimizer = torch.optim.Adam(model.parameters(), lr=0.001)

    for epoch in range(num_epochs):
        model.train()
        train_loss = 0

        for images, labels in train_loader:
            images, labels = images.to(device), labels.to(device)

            optimizer.zero_grad()
            outputs = model(images)
            loss = criterion(outputs, labels)
            loss.backward()
            optimizer.step()

            train_loss += loss.item()

        # 验证
        model.eval()
        val_loss = 0
        with torch.no_grad():
            for images, labels in val_loader:
                images, labels = images.to(device), labels.to(device)
                outputs = model(images)
                loss = criterion(outputs, labels)
                val_loss += loss.item()

        print(f'轮次 {epoch+1}/{num_epochs}, 训练损失: {train_loss:.4f}, 验证损失: {val_loss:.4f}')

    return model
```

### 用于语义分割的U-Net

```python
class UNet(nn.Module):
    def __init__(self, in_channels=4, num_classes=5):
        super().__init__()

        # 编码器
        self.enc1 = self.conv_block(in_channels, 64)
        self.enc2 = self.conv_block(64, 128)
        self.enc3 = self.conv_block(128, 256)
        self.enc4 = self.conv_block(256, 512)

        # 瓶颈层
        self.bottleneck = self.conv_block(512, 1024)

        # 解码器
        self.up1 = nn.ConvTranspose2d(1024, 512, 2, stride=2)
        self.dec1 = self.conv_block(1024, 512)

        self.up2 = nn.ConvTranspose2d(512, 256, 2, stride=2)
        self.dec2 = self.conv_block(512, 256)

        self.up3 = nn.ConvTranspose2d(256, 128, 2, stride=2)
        self.dec3 = self.conv_block(256, 128)

        self.up4 = nn.ConvTranspose2d(128, 64, 2, stride=2)
        self.dec4 = self.conv_block(128, 64)

        # 最终层
        self.final = nn.Conv2d(64, num_classes, 1)

    def conv_block(self, in_ch, out_ch):
        return nn.Sequential(
            nn.Conv2d(in_ch, out_ch, 3, padding=1),
            nn.BatchNorm2d(out_ch),
            nn.ReLU(inplace=True),
            nn.Conv2d(out_ch, out_ch, 3, padding=1),
            nn.BatchNorm2d(out_ch),
            nn.ReLU(inplace=True)
        )

    def forward(self, x):
        # 编码器
        e1 = self.enc1(x)
        e2 = self.enc2(F.max_pool2d(e1, 2))
        e3 = self.enc3(F.max_pool2d(e2, 2))
        e4 = self.enc4(F.max_pool2d(e3, 2))

        # 瓶颈层
        b = self.bottleneck(F.max_pool2d(e4, 2))

        # 带跳跃连接的解码器
        d1 = self.dec1(torch.cat([self.up1(b), e4], dim=1))
        d2 = self.dec2(torch.cat([self.up2(d1), e3], dim=1))
        d3 = self.dec3(torch.cat([self.up3(d2), e2], dim=1))
        d4 = self.dec4(torch.cat([self.up4(d3), e1], dim=1))

        return self.final(d4)
```

### 使用孪生网络进行变化检测

```python
class SiameseNetwork(nn.Module):
    """用于变化检测的孪生网络。"""

    def __init__(self):
        super().__init__()
        self.feature_extractor = nn.Sequential(
            nn.Conv2d(3, 32, 3, padding=1),
            nn.BatchNorm2d(32),
            nn.ReLU(),
            nn.MaxPool2d(2),

            nn.Conv2d(32, 64, 3, padding=1),
            nn.BatchNorm2d(64),
            nn.ReLU(),
            nn.MaxPool2d(2),

            nn.Conv2d(64, 128, 3, padding=1),
            nn.BatchNorm2d(128),
            nn.ReLU(),
        )

        self.classifier = nn.Sequential(
            nn.Conv2d(256, 128, 3, padding=1),
            nn.ReLU(),
            nn.Conv2d(128, 64, 3, padding=1),
            nn.ReLU(),
            nn.Conv2d(64, 2, 1),  # 二分类：变化/未变化
        )

    def forward(self, x1, x2):
        f1 = self.feature_extractor(x1)
        f2 = self.feature_extractor(x2)

        # 拼接特征
        diff = torch.abs(f1 - f2)
        combined = torch.cat([f1, f2, diff], dim=1)

        return self.classifier(combined)
```

## 图神经网络

### 用于空间数据的PyTorch Geometric

```python
import torch
from torch_geometric.data import Data
from torch_geometric.nn import GCNConv

# 创建空间图
def create_spatial_graph(points_gdf, k_neighbors=5):
    """使用k近邻从点数据创建图。"""

    from sklearn.neighbors import NearestNeighbors

    coords = np.array([[p.x, p.y] for p in points_gdf.geometry])

    # 寻找k近邻
    nbrs = NearestNeighbors(n_neighbors=k_neighbors).fit(coords)
    distances, indices = nbrs.kneighbors(coords)

    # 创建边索引
    edge_index = []
    for i, neighbors in enumerate(indices):
        for j in neighbors:
            edge_index.append([i, j])

    edge_index = torch.tensor(edge_index, dtype=torch.long).t().contiguous()

    # 节点特征
    features = points_gdf.drop('geometry', axis=1).values
    x = torch.tensor(features, dtype=torch.float)

    return Data(x=x, edge_index=edge_index)

# 用于空间预测的GCN
class SpatialGCN(torch.nn.Module):
