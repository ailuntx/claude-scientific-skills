# Open Notebook 示例

## 完整研究流程

此示例展示了一个完整的研究流程：创建笔记本、添加来源、生成笔记、与AI聊天以及跨材料搜索。

```python
import requests
import time

BASE_URL = "http://localhost:5055/api"


def complete_research_workflow():
    """使用 Open Notebook 的端到端研究流程"""

    # 1. 创建研究笔记本
    notebook = requests.post(f"{BASE_URL}/notebooks", json={
        "name": "Drug Resistance in Cancer",
        "description": "实体肿瘤耐药机制综述"
    }).json()
    notebook_id = notebook["id"]
    print(f"已创建笔记本: {notebook_id}")

    # 2. 从URL添加来源
    urls = [
        "https://www.nature.com/articles/s41568-020-0281-y",
        "https://www.cell.com/cancer-cell/fulltext/S1535-6108(20)30211-8",
    ]

    source_ids = []
    for url in urls:
        source = requests.post(f"{BASE_URL}/sources", data={
            "url": url,
            "notebook_id": notebook_id,
            "process_async": "true"
        }).json()
        source_ids.append(source["id"])
        print(f"已添加来源: {source['id']}")

    # 3. 等待处理完成
    for source_id in source_ids:
        while True:
            status = requests.get(
                f"{BASE_URL}/sources/{source_id}/status"
            ).json()
            if status.get("status") in ("completed", "failed"):
                break
            time.sleep(5)
        print(f"来源 {source_id}: {status['status']}")

    # 4. 创建聊天会话并提问
    session = requests.post(f"{BASE_URL}/chat/sessions", json={
        "notebook_id": notebook_id,
        "title": "耐药机制"
    }).json()

    answer = requests.post(f"{BASE_URL}/chat/execute", json={
        "session_id": session["id"],
        "message": "实体肿瘤的主要耐药机制有哪些？",
        "context": {"include_sources": True, "include_notes": True}
    }).json()
    print(f"AI回复: {answer}")

    # 5. 跨材料搜索
    results = requests.post(f"{BASE_URL}/search", json={
        "query": "efflux pump resistance mechanism",
        "search_type": "vector",
        "limit": 5
    }).json()
    print(f"找到 {results['total']} 条搜索结果")

    # 6. 创建总结性人工笔记
    note = requests.post(f"{BASE_URL}/notes", json={
        "title": "耐药机制总结",
        "content": "文献中的关键发现...",
        "note_type": "human",
        "notebook_id": notebook_id
    }).json()
    print(f"已创建笔记: {note['id']}")


if __name__ == "__main__":
    complete_research_workflow()
```

## 文件上传示例

```python
import requests

BASE_URL = "http://localhost:5055/api"


def upload_research_papers(notebook_id, file_paths):
    """向笔记本上传多篇研究论文"""
    for path in file_paths:
        with open(path, "rb") as f:
            response = requests.post(
                f"{BASE_URL}/sources",
                data={
                    "notebook_id": notebook_id,
                    "process_async": "true",
                },
                files={"file": (path.split("/")[-1], f)},
            )
        if response.status_code == 200:
            print(f"已上传: {path}")
        else:
            print(f"失败: {path} - {response.text}")


# 使用示例
upload_research_papers("notebook:abc123", [
    "papers/study_1.pdf",
    "papers/study_2.pdf",
    "papers/supplementary.docx",
])
```

## 播客生成示例

```python
import requests
import time

BASE_URL = "http://localhost:5055/api"


def generate_research_podcast(notebook_id):
    """根据笔记本内容生成播客节目"""

    # 获取可用剧集和发言人配置
    # (需先在UI或API中配置)

    # 提交播客生成任务
    job = requests.post(f"{BASE_URL}/podcasts/generate", json={
        "notebook_id": notebook_id,
        "episode_profile_id": "episode_profile:default",
        "speaker_profile_ids": [
            "speaker_profile:host",
            "speaker_profile:expert"
        ]
    }).json()
    job_id = job["job_id"]
    print(f"播客生成已启动: {job_id}")

    # 轮询任务状态
    while True:
        status = requests.get(f"{BASE_URL}/podcasts/jobs/{job_id}").json()
        print(f"状态: {status.get('status', '处理中')}")
        if status.get("status") in ("completed", "failed"):
            break
        time.sleep(10)

    if status["status"] == "completed":
        # 下载音频
        episode_id = status["episode_id"]
        audio = requests.get(
            f"{BASE_URL}/podcasts/episodes/{episode_id}/audio"
        )
        with open("research_podcast.mp3", "wb") as f:
            f.write(audio.content)
        print("播客已保存为 research_podcast.mp3")


if __name__ == "__main__":
    generate_research_podcast("notebook:abc123")
```

## 自定义转换流程

```python
import requests

BASE_URL = "http://localhost:5055/api"


def create_and_run_transformations():
    """创建自定义转换并应用于内容"""

    # 创建方法学提取转换
    transform = requests.post(f"{BASE_URL}/transformations", json={
        "name": "extract_methods",
        "title": "提取方法",
        "description": "从论文中提取并结构化方法学内容",
        "prompt": (
            "从文本中提取方法学部分。"
            "按以下结构组织：研究设计、样本量、统计方法、"
            "关键变量。输出结构化Markdown格式。"
        ),
        "apply_default": False,
    }).json()

    # 获取适用的模型
    models = requests.get(f"{BASE_URL}/models", params={
        "model_type": "llm"
    }).json()
    model_id = models[0]["id"]

    # 执行转换
    result = requests.post(f"{BASE_URL}/transformations/execute", json={
        "transformation_id": transform["id"],
        "input_text": "我们进行了一项随机对照试验...",
        "model_id": model_id,
    }).json()
    print(f"提取的方法学:\n{result['output']}")


if __name__ == "__main__":
    create_and_run_transformations()
```

## 带筛选的语义搜索

```python
import requests

BASE_URL = "http://localhost:5055/api"


def advanced_search(notebook_id, query):
    """执行带筛选的语义搜索并获取AI解答"""

    # 获取特定笔记本的来源
    sources = requests.get(f"{BASE_URL}/sources", params={
        "notebook_id": notebook_id
    }).json()
    source_ids = [s["id"] for s in sources]

    # 限定在笔记本来源的向量搜索
    results = requests.post(f"{BASE_URL}/search", json={
        "query": query,
        "search_type": "vector",
        "limit": 10,
        "source_ids": source_ids,
        "min_similarity": 0.75,
    }).json()

    print(f"找到 {results['total']} 条结果:")
    for result in results["results"]:
        print(f"  - {result.get('title', '无标题')} "
              f"(相似度: {result.get('similarity', 'N/A')})")

    # 获取AI增强解答
    answer = requests.post(f"{BASE_URL}/search/ask/simple", json={
        "query": query,
    }).json()
    print(f"\nAI解答: {answer['response']}")


if __name__ == "__main__":
    advanced_search("notebook:abc123", "CRISPR基因编辑效率")
```

## 模型管理

```python
import requests

BASE_URL = "http://localhost:5055/api"


def setup_ai_models():
    """配置 Open Notebook 的AI模型"""

    # 检查可用供应商
    providers = requests.get(f"{BASE_URL}/models/providers").json()
    print(f"可用供应商: {providers}")

    # 发现供应商模型
    discovered = requests.get(
        f"{BASE_URL}/models/discover/openai"
    ).json()
    print(f"发现 {len(discovered)} 个OpenAI模型")

    # 同步模型使其可用
    requests.post(f"{BASE_URL}/models/sync/openai")

    # 自动分配默认模型
    requests.post(f"{BASE_URL}/models/auto-assign")

    # 检查当前默认设置
    defaults = requests.get(f"{BASE_URL}/models/defaults").json()
    print(f"默认模型: {defaults}")


if __name__ == "__main__":
    setup_ai_models()
```
