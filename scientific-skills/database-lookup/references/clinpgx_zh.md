# PharmGKB（临床药物基因组学）

## 基础URL
```
https://api.pharmgkb.org/v1/data/
```

## 认证
只读访问无需API密钥。

## 关键端点

### 通用搜索
```
GET https://api.pharmgkb.org/v1/search?q={term}&page=0&size=10
```

### 基因数据
```
GET /gene?symbol={symbol}
```
示例：`/gene?symbol=CYP2D6`

响应包含：id, symbol, chromosome, hasGuideline, hasClinicalAnnotation, cpicGene

### 药物数据
```
GET /drug?name={name}
```
示例：`/drug?name=warfarin`

响应包含：id, name, genericNames, tradeNames, rxNormId, atcCodes

### 临床注释（药物-基因相互作用）
```
GET /clinicalAnnotation?gene={symbol}&drug={name}&level={level}
```

证据等级：`1A`, `1B`, `2A`, `2B`, `3`, `4`

示例：
```
/clinicalAnnotation?gene=CYP2C19&drug=clopidogrel&level=1A
```

响应包含：level, gene, drug, phenotype, significance, variants, url

### CPIC/DPWG指南
```
GET /guideline?gene={symbol}&drug={name}&source=CPIC
```

### 药代动力学通路
```
GET /pathway?drug={name}
```

### 药品标签（FDA, EMA）
```
GET /drugLabel?drug={name}&source=FDA
```

## 速率限制
无硬性公开限制。请合理使用。批量数据通过PharmGKB下载页面获取。
