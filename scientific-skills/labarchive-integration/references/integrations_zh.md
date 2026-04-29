# LabArchives 第三方集成

## 概述

LabArchives 与众多科学软件平台集成，以简化研究工作流程。本文档涵盖各支持平台的程序化集成方法、自动化策略及最佳实践。

## 集成分类

### 1. 实验方案管理

#### Protocols.io 集成

直接从 Protocols.io 导出实验方案至 LabArchives 电子实验记录本。

**应用场景：**
- 在实验记录本中统一实验操作流程
- 维护实验方案的版本控制
- 将实验方案与实验结果关联

**设置步骤：**
1. 在 LabArchives 设置中启用 Protocols.io 集成
2. 使用 Protocols.io 账户认证
3. 浏览并选择要导出的实验方案

**程序化方法：**
```python
# 将 Protocols.io 实验方案导出为 HTML/PDF
# 通过 API 上传至 LabArchives

def import_protocol_to_labarchives(client, uid, nbid, protocol_id):
    """将 Protocols.io 实验方案导入 LabArchives 条目"""
    # 1. 从 Protocols.io API 获取实验方案
    protocol_data = fetch_protocol_from_protocolsio(protocol_id)

    # 2. 在 LabArchives 创建新条目
    entry_params = {
        'uid': uid,
        'nbid': nbid,
        'title': f"Protocol: {protocol_data['title']}",
        'content': protocol_data['html_content']
    }
    response = client.make_call('entries', 'create_entry', params=entry_params)

    # 3. 添加实验方案元数据作为注释
    entry_id = extract_entry_id(response)
    comment_params = {
        'uid': uid,
        'nbid': nbid,
        'entry_id': entry_id,
        'comment': f"Protocols.io ID: {protocol_id}<br>Version: {protocol_data['version']}"
    }
    client.make_call('entries', 'create_comment', params=comment_params)

    return entry_id
```

**更新日期：** 2025年9月22日

### 2. 数据分析工具

#### GraphPad Prism 集成（8+版本）

直接从 Prism 导出分析结果、图表和图像至 LabArchives。

**应用场景：**
- 归档原始数据的统计分析
- 记录出版物中的图表生成过程
- 维护合规性分析审计追踪

**设置步骤：**
1. 安装 GraphPad Prism 8 或更高版本
2. 在 Prism 偏好设置中配置 LabArchives 连接
3. 使用文件菜单中的"导出至 LabArchives"选项

**程序化方法：**
```python
# 通过 API 将 Prism 文件上传至 LabArchives

def upload_prism_analysis(client, uid, nbid, entry_id, prism_file_path):
    """将 GraphPad Prism 文件上传至 LabArchives 条目"""
    import requests

    url = f'{client.api_url}/entries/upload_attachment'
    files = {'file': open(prism_file_path, 'rb')}
    params = {
        'uid': uid,
        'nbid': nbid,
        'entry_id': entry_id,
        'filename': os.path.basename(prism_file_path),
        'access_key_id': client.access_key_id,
        'access_password': client.access_password
    }

    response = requests.post(url, files=files, data=params)
    return response
```

**支持文件类型：**
- .pzfx (Prism 项目文件)
- .png, .jpg, .pdf (导出图表)
- .xlsx (导出数据表)

**更新日期：** 2025年9月8日

### 3. 分子生物学与生物信息学

#### SnapGene 集成

为分子生物学工作流程、质粒图谱和序列分析提供直接集成。

**应用场景：**
- 记录克隆策略
- 将质粒图谱与实验记录共同归档
- 将序列与实验结果关联

**设置步骤：**
1. 安装 SnapGene 软件
2. 在 SnapGene 偏好设置中启用 LabArchives 导出
3. 使用"发送至 LabArchives"功能

**支持文件格式：**
- .dna (SnapGene 文件)
- .gb, .gbk (GenBank 格式)
- .fasta (序列文件)
- .png, .pdf (质粒图谱导出文件)

**程序化工作流：**
```python
def upload_snapgene_file(client, uid, nbid, entry_id, snapgene_file):
    """上传 SnapGene 文件及预览图"""
    # 上传主 SnapGene 文件
    upload_attachment(client, uid, nbid, entry_id, snapgene_file)

    # 生成并上传预览图（需 SnapGene CLI）
    preview_png = generate_snapgene_preview(snapgene_file)
    upload_attachment(client, uid, nbid, entry_id, preview_png)
```

#### Geneious 集成

从 Geneious 导出生物信息学分析至 LabArchives。

**应用场景：**
- 归档序列比对和系统发育树
- 记录 NGS 分析流程
- 将生物信息学工作流与湿实验关联

**支持导出内容：**
- 序列比对
- 系统发育树
- 组装报告
- 变异检测结果

**文件格式：**
- .geneious (Geneious 文档)
- .fasta, .fastq (序列数据)
- .bam, .sam (比对文件)
- .vcf (变异文件)

### 4. 计算型笔记本

#### Jupyter 集成

将 Jupyter 笔记本嵌入 LabArchives 条目，实现可复现的计算研究。

**应用场景：**
- 记录数据分析工作流
- 归档计算实验
- 关联代码、结果与说明文档

**工作流：**

```python
def export_jupyter_to_labarchives(notebook_path, client, uid, nbid):
    """将 Jupyter 笔记本导出至 LabArchives"""
    import nbformat
    from nbconvert import HTMLExporter

    # 加载笔记本
    with open(notebook_path, 'r') as f:
        nb = nbformat.read(f, as_version=4)

    # 转换为 HTML
    html_exporter = HTMLExporter()
    html_exporter.template_name = 'classic'
    (body, resources) = html_exporter.from_notebook_node(nb)

    # 在 LabArchives 创建条目
    entry_params = {
        'uid': uid,
        'nbid': nbid,
        'title': f"Jupyter Notebook: {os.path.basename(notebook_path)}",
        'content': body
    }
    response = client.make_call('entries', 'create_entry', params=entry_params)

    # 将原始 .ipynb 文件作为附件上传
    entry_id = extract_entry_id(response)
    upload_attachment(client, uid, nbid, entry_id, notebook_path)

    return entry_id
```

**最佳实践：**
- 导出包含输出结果（导出前执行全部单元格）
- 附加 environment.yml 或 requirements.txt 文件
- 在注释中添加执行时间戳和系统信息

### 5. 临床研究

#### REDCap 集成

临床数据采集与 LabArchives 集成，满足研究合规性和审计追踪要求。

**应用场景：**
- 将临床数据采集与实验记录本关联
- 维护监管合规的审计追踪
- 记录临床试验方案及修订

**集成方式：**
- 通过 REDCap API 将数据导出至 LabArchives 条目
- 纵向研究的自动化数据同步
- HIPAA 合规数据处理

**示例工作流：**
```python
def sync_redcap_to_labarchives(redcap_api_token, client, uid, nbid):
    """将 REDCap 数据同步至 LabArchives"""
    # 获取 REDCap 数据
    redcap_data = fetch_redcap_data(redcap_api_token)

    # 创建 LabArchives 条目
    entry_params = {
        'uid': uid,
        'nbid': nbid,
        'title': f"REDCap Data Export {datetime.now().strftime('%Y-%m-%d')}",
        'content': format_redcap_data_html(redcap_data)
    }
    response = client.make_call('entries', 'create_entry', params=entry_params)

    return response
```

**合规特性：**
- 符合 21 CFR Part 11 规范
- 审计追踪维护
- 数据完整性验证

### 6. 科研出版

#### Qeios 集成

预印本与同行评审的科研出版平台集成。

**应用场景：**
- 将研究成果导出至预印本服务器
- 记录出版工作流程
- 将已发表文章与实验记录本关联

**工作流：**
- 从 LabArchives 导出格式化条目
- 提交至 Qeios 平台
- 维护记录本与出版物间的双向链接

#### SciSpace 集成

文献管理与引用集成。

**应用场景：**
- 将参考文献与实验步骤关联
- 在记录本中维护文献综述
- 生成报告参考文献

**功能：**
- 从 SciSpace 导入引文至 LabArchives
- PDF 批注同步
- 参考文献管理

## 集成的 OAuth 认证

LabArchives 新第三方集成现采用 OAuth 2.0 认证。

**开发者 OAuth 流程：**

```python
def labarchives_oauth_flow(client_id, client_secret, redirect_uri):
    """实现 LabArchives 集成的 OAuth 2.0 流程"""
    import requests

    # 步骤1：获取授权码
    auth_url = "https://mynotebook.labarchives.com/oauth/authorize"
    auth_params = {
        'client_id': client_id,
        'redirect_uri': redirect_uri,
        'response_type': 'code',
        'scope': 'read write'
    }
    # 用户访问 auth_url 并授权

    # 步骤2：用授权码换取访问令牌
    token_url = "https://mynotebook.labarchives.com/oauth/token"
    token_params = {
        'client_id': client_id,
        'client_secret': client_secret,
        'redirect_uri': redirect_uri,
        'grant_type': 'authorization_code',
        'code': authorization_code  # 来自重定向
    }

    response = requests.post(token_url, data=token_params)
    tokens = response.json()

    return tokens['access_token'], tokens['refresh_token']
```

**OAuth 优势：**
- 比 API 密钥更安全
- 细粒度权限控制
- 长期集成的令牌刷新机制
- 可撤销访问权限

## 自定义集成开发

### 通用工作流

针对未官方支持的工具，可开发自定义集成：

1. **导出数据**：从源应用（API 或文件导出）
2. **格式转换**：转为 HTML 或支持的文件类型
3. **认证**：使用 LabArchives API 认证
4. **创建条目**：或上传附件
5. **添加元数据**：通过注释实现可追溯性

### 示例：自定义集成模板

```python
class LabArchivesIntegration:
    """LabArchives 自定义集成模板"""

    def __init__(self, config_path):
        self.client = self._init_client(config_path)
        self.uid = self._authenticate()

    def _init_client(self, config_path):
        """初始化 LabArchives 客户端"""
        with open(config_path) as f:
            config = yaml.safe_load(f)
        return Client(config['api_url'],
                     config['access_key_id'],
                     config['access_password'])

    def _authenticate(self):
        """获取用户 ID"""
        # 实现参考 authentication_guide.md
        pass

    def export_data(self, source_data, nbid, title):
        """导出数据至 LabArchives"""
        # 将数据转换为 HTML
        html_content = self._transform_to_html(source_data)

        # 创建条目
        params = {
            'uid': self.uid,
            'nbid': nbid,
            'title': title,
            'content': html_content
        }
        response = self.client.make_call('entries', 'create_entry', params=params)

        return extract_entry_id(response)

    def _transform_to_html(self, data):
        """将数据转换为 HTML 格式"""
        # 自定义转换逻辑
        pass
```

## 集成最佳实践

1. **版本控制**：记录生成数据的软件版本
2. **元数据保留**：包含时间戳、用户信息和处理参数
3. **文件格式标准**：优先使用开放格式（CSV、JSON、HTML）
4. **批量操作**：为批量上传实施速率限制
5. **错误处理**：采用指数退避的重试逻辑
6. **审计追踪**：记录所有 API 操作以确保合规
7. **测试**：在生产环境使用前在测试记录本中验证集成

## 集成故障排除

### 常见问题

**集成未在 LabArchives 显示：**
- 确认管理员已启用集成
- 如使用 OAuth 请检查权限设置
- 确保软件版本兼容

**文件上传失败：**
- 检查文件大小限制（通常单文件上限 2GB）
- 验证文件格式兼容性
- 确认存储配额充足

**认证错误：**
- 验证 API 凭证有效性
- 检查集成专用令牌是否过期
- 确认用户具备必要权限

### 集成支持

针对特定集成问题：
- 查阅软件供应商文档（如 GraphPad、Protocols.io）
- 联系 LabArchives 支持：support@labarchives.com
- 访问 LabArchives 知识库：help.labarchives.com

## 未来集成机会

可开发的潜在集成方向：
- 电子数据采集（EDC）系统
- 实验室信息管理系统（LIMS）
- 仪器数据系统（色谱、光谱）
- 云存储平台（Box、Dropbox、Google Drive）
- 项目管理工具（Asana、Monday.com）
- 经费管理系统

如需开发自定义集成，请联系 LabArchives 获取 API 合作机会。
