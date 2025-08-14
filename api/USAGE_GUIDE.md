# Cobalt API 使用指南

## 本地构建和运行

### 1. 安装依赖
```bash
# 使用 pnpm 安装依赖（推荐）
pnpm install

# 或使用 npm
npm install
```

### 2. 启动 API 服务器
```bash
# 默认端口 9000
npm start

# 使用自定义端口（例如 9001）
API_PORT=9001 API_URL=http://localhost:9001 npm start

# 后台运行
API_PORT=9001 API_URL=http://localhost:9001 npm start > server.log 2>&1 &
```

## API 使用方法

### 基本请求格式

**端点:** `POST /`

**请求头:**
```
Accept: application/json
Content-Type: application/json
```

### 请求参数

| 参数 | 类型 | 说明 | 默认值 |
|------|------|------|--------|
| `url` | string | 视频URL（必需） | - |
| `downloadMode` | string | 下载模式：auto/audio/mute | auto |
| `videoQuality` | string | 视频质量：max/4320/2160/1440/1080/720/480/360/240/144 | 1080 |
| `audioFormat` | string | 音频格式：best/mp3/ogg/wav/opus | mp3 |
| `audioBitrate` | string | 音频比特率：320/256/128/96/64/8 (kbps) | 128 |
| `filenameStyle` | string | 文件名样式：classic/pretty/basic/nerdy | basic |
| `disableMetadata` | boolean | 是否禁用元数据 | false |

### 支持的平台

- YouTube（视频、音乐、Shorts）
- TikTok（视频、图片、音频）
- Twitter/X
- Instagram（Reels、照片、视频）
- Facebook（公开视频）
- Reddit（GIF、视频）
- Bilibili
- SoundCloud
- Twitch Clips
- Vimeo
- 以及更多...

### 使用示例

#### 1. 使用 cURL
```bash
# 下载 YouTube 视频
curl -X POST http://localhost:9001/ \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://www.youtube.com/watch?v=VIDEO_ID",
    "videoQuality": "720",
    "audioFormat": "mp3"
  }'

# 只下载音频
curl -X POST http://localhost:9001/ \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://www.youtube.com/watch?v=VIDEO_ID",
    "downloadMode": "audio",
    "audioFormat": "mp3",
    "audioBitrate": "320"
  }'
```

#### 2. 使用 JavaScript/Node.js
```javascript
async function downloadVideo(videoUrl) {
    const response = await fetch('http://localhost:9001/', {
        method: 'POST',
        headers: {
            'Accept': 'application/json',
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            url: videoUrl,
            videoQuality: '1080',
            audioFormat: 'mp3'
        })
    });

    const result = await response.json();
    
    if (result.status === 'tunnel' || result.status === 'redirect') {
        console.log('Download URL:', result.url);
        console.log('Filename:', result.filename);
        
        // 下载文件
        const fileResponse = await fetch(result.url);
        const buffer = await fileResponse.arrayBuffer();
        // 保存 buffer 到文件...
    }
}
```

#### 3. 使用 Python
```python
import requests
import json

def download_video(video_url):
    api_url = "http://localhost:9001/"
    
    headers = {
        "Accept": "application/json",
        "Content-Type": "application/json"
    }
    
    data = {
        "url": video_url,
        "videoQuality": "720",
        "audioFormat": "mp3"
    }
    
    response = requests.post(api_url, headers=headers, json=data)
    result = response.json()
    
    if result["status"] in ["tunnel", "redirect"]:
        download_url = result["url"]
        filename = result["filename"]
        
        # 下载文件
        file_response = requests.get(download_url)
        with open(filename, 'wb') as f:
            f.write(file_response.content)
        
        print(f"Downloaded: {filename}")
    else:
        print(f"Error: {result}")

# 使用示例
download_video("https://www.youtube.com/watch?v=dQw4w9WgXcQ")
```

### 响应格式

#### 成功响应
```json
{
    "status": "tunnel",
    "url": "http://localhost:9001/tunnel?id=...",
    "filename": "video_title.mp4"
}
```

#### 错误响应
```json
{
    "status": "error",
    "error": {
        "code": "error_code",
        "message": "Error description"
    }
}
```

## 高级配置

### 环境变量

| 变量名 | 说明 | 默认值 |
|--------|------|--------|
| `API_PORT` | API 端口 | 9000 |
| `API_URL` | API URL | - |
| `DURATION_LIMIT` | 最大视频时长（秒） | 10800 |
| `RATELIMIT_MAX` | 速率限制（每分钟请求数） | 20 |

### 运行测试
```bash
# 运行内置测试
npm test

# 运行自定义测试脚本
node test-download.js
```

## 注意事项

1. **速率限制**: API 有速率限制，默认每分钟 20 个请求
2. **视频时长**: 默认最大支持 3 小时（10800 秒）的视频
3. **代理设置**: 可通过 `HTTP_PROXY` 和 `HTTPS_PROXY` 环境变量设置代理
4. **隧道生命周期**: 下载链接有效期为 90 秒，需要及时下载

## 故障排除

### 端口被占用
```bash
# 检查端口占用
lsof -i :9000

# 使用其他端口
API_PORT=9001 API_URL=http://localhost:9001 npm start
```

### 依赖安装失败
```bash
# 清理缓存后重试
pnpm store prune
pnpm install
```

### API 返回错误
- 检查视频 URL 是否正确
- 确认视频是公开可访问的
- 查看 server.log 获取详细错误信息