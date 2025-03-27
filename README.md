# DeepSeek-backup-web
DeepSeek备份网页版

# DeepSeek Chat API 文档

## 基础信息
- 基础URL: `https://chat.deepseek.com`
- 认证方式: Bearer Token (通过`Authorization`头部传递)

## API端点

### 1. 获取聊天会话列表
```
GET /api/v0/chat_session/fetch_page?count={数量}
```

**请求头:**
```http
Authorization: Bearer <你的token>
Accept: */*
Referer: https://chat.deepseek.com/
User-Agent: Mozilla/5.0 (Linux; Android 10; K) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/132.0.0.0 Mobile Safari/537.36
```

**查询参数:**
- `count`: 要返回的会话数量

**响应示例:**
```json
{
  "code": 0,
  "data": {
    "biz_data": {
      "chat_sessions": [
        {
          "id": "会话ID",
          "title": "会话标题",
          "updated_at": 时间戳
        }
      ]
    }
  }
}
```

### 2. 获取会话消息历史
```
GET /api/v0/chat/history_messages?chat_session_id={会话ID}
```

**请求头:**
```http
Authorization: Bearer <你的token>
Accept: */*
Referer: https://chat.deepseek.com/a/chat/s/{会话ID}
User-Agent: Mozilla/5.0 (Linux; Android 10; K) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/132.0.0.0 Mobile Safari/537.36
```

**查询参数:**
- `chat_session_id`: 要查询的会话ID

**响应示例:**
```json
{
  "code": 0,
  "data": {
    "biz_data": {
      "messages": [
        {
          "content": "消息内容",
          "role": "user|assistant"
        }
      ]
    }
  }
}
```

## 使用流程
1. 首先调用`/api/v0/chat_session/fetch_page`获取所有会话列表
2. 对每个会话ID调用`/api/v0/chat/history_messages`获取详细消息
3. 将会话内容保存为`标题.txt`文件

## 完整示例代码
```bash
#!/bin/bash

# 获取所有会话
curl 'https://chat.deepseek.com/api/v0/chat_session/fetch_page?count=500' \
  -H 'Authorization: Bearer <你的token>' \
  -H 'User-Agent: Mozilla/5.0' > sessions.json

# 提取会话ID和标题
jq -r '.data.biz_data.chat_sessions[] | .id + " " + .title' sessions.json | while read id title; do
  # 获取会话历史
  curl "https://chat.deepseek.com/api/v0/chat/history_messages?chat_session_id=$id" \
    -H 'Authorization: Bearer <你的token>' \
    -H 'User-Agent: Mozilla/5.0' > "$title.json"
  
  # 转换为文本文件
  jq -r '.data.biz_data.messages[] | .role + ": " + .content' "$title.json" > "${title//\//_}.txt"
done
```

> 注意：将`<你的token>`替换为实际的Bearer Token，并确保已安装`jq`工具处理JSON数据。


```
import requests
import json
import os

# Configuration
BASE_URL = "https://chat.deepseek.com"
API_KEY = "Bearer vD7……………………"  # Replace with your actual Bearer token if needed
import requests
import json
import os
from urllib.parse import quote, unquote  # 新增 unquote 用于解码文件名

# 替换为实际的Bearer token（如果需要）
HEADERS = {
    'authority': 'chat.deepseek.com',
    'accept': '*/*',
    'accept-language': 'zh-CN,zh;q=0.9',
    'authorization': API_KEY,
    'referer': 'https://chat.deepseek.com/',
    'sec-ch-ua': '"Not A(Brand";v="8", "Chromium";v="132"',
    'sec-ch-ua-mobile': '?1',
    'sec-ch-ua-platform': '"Android"',
    'sec-fetch-dest': 'empty',
    'sec-fetch-mode': 'cors',
    'sec-fetch-site': 'same-origin',
    'user-agent': 'Mozilla/5.0 (Linux; Android 10; K) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/132.0.0.0 Mobile Safari/537.36',
    'x-app-version': '20241129.1',
    'x-client-locale': 'zh_CN',
    'x-client-platform': 'web',
    'x-client-version': '1.0.0-always'
}

def get_all_chat_sessions():
    """获取所有对话会话"""
    url = f"{BASE_URL}/api/v0/chat_session/fetch_page?count=500"
    response = requests.get(url, headers=HEADERS)
    
    if response.status_code == 200:
        data = response.json()
        return data.get('data', {}).get('biz_data', {}).get('chat_sessions', [])
    else:
        print(f"获取会话列表失败，状态码: {response.status_code}")
        return []

def get_chat_history(chat_session_id):
    """获取特定会话的完整历史记录（原始JSON）"""
    url = f"{BASE_URL}/api/v0/chat/history_messages?chat_session_id={chat_session_id}"
    response = requests.get(url, headers=HEADERS)
    
    if response.status_code == 200:
        return response.text  # 返回原始文本响应
    else:
        print(f"获取会话 {chat_session_id} 历史失败，状态码: {response.status_code}")
        return None

def save_raw_response(title, raw_data):
    """保存原始API响应到文件"""
    # 1. 先解码可能已经编码的标题（防止重复解码）
    decoded_title = unquote(title)
    # 2. 清理特殊字符，并重新编码（确保文件名安全）
    safe_title = quote(decoded_title.replace('/', '_'))[:100]  # 限制文件名长度
    # 3. 最终保存时解码，使文件名可读
    final_filename = unquote(safe_title) + ".json"
    
    with open(final_filename, 'w', encoding='utf-8') as f:
        f.write(raw_data)
    
    print(f"已保存: {final_filename}")

def main():
    # 创建输出目录（如果不存在）
    if not os.path.exists('raw_responses'):
        os.makedirs('raw_responses')
    os.chdir('raw_responses')
    
    # 获取所有对话会话
    sessions = get_all_chat_sessions()
    print(f"找到 {len(sessions)} 个对话会话")
    
    for session in sessions:
        session_id = session['id']
        title = session['title']
        print(f"\n正在处理: {title} (ID: {session_id})")
        
        # 获取原始聊天历史记录
        raw_response = get_chat_history(session_id)
        if raw_response:
            save_raw_response(title, raw_response)
        else:
            print(f"无法获取会话 {session_id} 的原始数据")

if __name__ == "__main__":
    main()
```
