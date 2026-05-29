# IMA 知识库 API 调用指南

## 凭证配置

### 获取凭证

1. 打开 https://ima.qq.com/agent-interface
2. 获取 **Client ID** 和 **Api Key**
3. 配置环境变量：

```bash
export IMA_OPENAPI_CLIENTID="your_client_id"
export IMA_OPENAPI_APIKEY="your_api_key"
```

### 凭证预检

每次调用前检查：

```bash
if [ -z "$IMA_OPENAPI_CLIENTID" ] || [ -z "$IMA_OPENAPI_APIKEY" ]; then
  echo "缺少 IMA 凭证，请配置环境变量"
  exit 1
fi
```

---

## 知识库参数

- 笔记知识库ID：`fWOyEuKiNhpDMhD7xdanzFFPVsw4iAkeA0XCQoPVNfA=`
- 核心笔记目录：`folder8dc8010c0788bef5`（实体老板跃迁）
- 终稿笔记本：`folderc8ed84dc3028d711`（已发布文章）
- 草稿笔记本：`folder5ec3d92b6f5680ae`（实体店草稿）
- 工具提示词目录：`folder739f76ddae6dc6e0`（常用工具/提示词）

---

## API 调用模板

### 辅助函数

```bash
ima_api() {
  local endpoint="$1" body="$2"
  curl -s -X POST "https://ima.qq.com/openapi/note/v1/$endpoint" \
    -H "ima-openapi-clientid: $IMA_OPENAPI_CLIENTID" \
    -H "ima-openapi-apikey: $IMA_OPENAPI_APIKEY" \
    -H "Content-Type: application/json" \
    -d "$body"
}
```

### 搜索笔记（推荐）

```bash
# 按正文内容搜索
ima_api "search_note_book" '{
  "search_type": 1,
  "query_info": {"content": "抖音获客"},
  "start": 0,
  "end": 10
}'

# 按标题搜索
ima_api "search_note_book" '{
  "search_type": 0,
  "query_info": {"title": "复购"},
  "start": 0,
  "end": 10
}'
```

### 浏览笔记本

```bash
# 列出笔记本
ima_api "list_note_folder_by_cursor" '{"cursor": "0", "limit": 20}'

# 浏览指定笔记本的笔记
ima_api "list_note_by_folder_id" '{
  "folder_id": "folderc8ed84dc3028d711",
  "cursor": "",
  "limit": 20
}'
```

### 读取笔记内容

```bash
# 获取笔记正文（纯文本格式）
ima_api "get_doc_content" '{
  "doc_id": "笔记ID",
  "target_content_format": 0
}'
```

---

## 响应解析

### 搜索结果结构

```json
{
  "code": 0,
  "data": {
    "docs": [{
      "doc": {
        "basic_info": {
          "docid": "笔记ID",
          "title": "标题",
          "summary": "摘要",
          "folder_id": "笔记本ID",
          "folder_name": "笔记本名称"
        }
      },
      "highlight_info": {
        "doc_title": ["<em>高亮词</em>"]
      }
    }],
    "is_end": true
  }
}
```

### 提取笔记ID和标题

```bash
ima_api "search_note_book" '{"search_type": 1, "query_info": {"content": "关键词"}, "start": 0, "end": 10}' | \
python3 -c "
import sys, json
data = json.load(sys.stdin)
for doc in data.get('data', {}).get('docs', []):
    info = doc.get('doc', {}).get('basic_info', {})
    print(f\"{info.get('title')}|{info.get('docid')}\")
"
```

---

## 常见错误

| 错误码 | 含义 | 处理方式 |
|-------|------|---------|
| 200002 | skill auth failed | 凭证失效，重新获取 apiKey |
| 100001 | 参数错误 | 检查 JSON 格式 |
| 100005 | 无权限 | 确认操作的是自己的笔记 |
| 100006 | 笔记已删除 | 告知用户笔记不存在 |

---

## 使用流程

1. 用户提问 → 搜索知识库获取相关内容
2. 解析搜索结果 → 获取笔记ID
3. 读取笔记正文 → 提取关键信息
4. 联网补充 → 获取最新案例和数据
5. 整合输出 → 给出可落地方案

---

## 注意事项

- 标题搜索用 `search_type: 0`
- 正文搜索用 `search_type: 1`（推荐）
- 时间字段为 Unix 毫秒时间戳
- 内容必须 UTF-8 编码
