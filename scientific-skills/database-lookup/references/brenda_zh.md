# BRENDA 酶数据库 (SOAP API)

## 重要提示：BRENDA 使用 SOAP 协议而非 REST。需安装 Python 的 `zeep` 库。

## SOAP 端点
```
https://www.brenda-enzymes.org/soap/brenda_zeep.wsdl
```

## 认证
需在 https://www.brenda-enzymes.org/register.php 免费注册。  
每次调用需传递凭证（邮箱 + SHA-256 哈希密码）。

## 核心 SOAP 方法

所有方法均需基础参数：`email`、`password`（SHA-256哈希值）和 `ecNumber`。

| 方法 | 描述 |
|--------|-------------|
| `getKmValue` | 米氏常数 (Km) |
| `getTurnoverNumber` | 转换数 (kcat) |
| `getKcatKmValue` | 催化效率 (kcat/Km) |
| `getKiValue` | 抑制常数 (Ki) |
| `getIc50Value` | IC50 值 |
| `getSpecificActivity` | 比活性 |
| `getPhOptimum` | 最适 pH |
| `getTemperatureOptimum` | 最适温度 |
| `getSubstrate` | 底物 |
| `getProduct` | 产物 |
| `getInhibitors` | 抑制剂 |
| `getCofactor` | 辅因子 |
| `getOrganism` | 来源生物体 |
| `getReaction` | 反应方程式 |
| `getSequence` | 蛋白质序列 |
| `getDisease` | 相关疾病 |

## 参数语法
采用 `字段名*值` 格式。空值表示返回全部。

```
ecNumber*1.1.1.1           # 必需：EC 编号
organism*Homo sapiens      # 可选：按生物体筛选
substrate*ethanol          # 可选：按底物筛选
kmValue*                   # 返回字段（空值=全部）
```

## Python 示例
```python
import hashlib
from zeep import Client

client = Client("https://www.brenda-enzymes.org/soap/brenda_zeep.wsdl")
email = "your@email.com"
password = hashlib.sha256("your_password".encode()).hexdigest()

# 获取乙醇脱氢酶的 Km 值
result = client.service.getKmValue(
    email, password,
    "ecNumber*1.1.1.1", "organism*Homo sapiens",
    "kmValue*", "substrate*", "literature*"
)
```

## 响应格式
返回以 `!`（记录分隔符）和 `#`/`*`（字段分隔符）解析的字符串，需手动解析。

## 速率限制
未公布具体限制。SOAP 响应通常需 1-5 秒。请合理使用——此为免费学术服务。

## 本技能注意事项
由于 BRENDA 使用 SOAP（非 REST），调用需编写并执行含 `zeep` 的 Python 脚本。请使用 Bash 而非 WebFetch 运行脚本。
