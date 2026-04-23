# Optional API Contract

`Astro Doge` 默认是静态模板。

如果你想启用评论、留言板或 `/thoughts/new` 网页端发布，可以自己实现后端接口；前端页面已经约定好了请求地址和返回格式。

## 约定

- `404` / `405` 会被前端视为“功能未启用”
- 评论接口返回 `500` 且包含 `{"error":"Server configuration error"}` 时，前端会提示“API 已部署但环境变量未配置完成”
- 返回 JSON 即可，不要求必须使用某个部署平台

## `GET /api/comments?slug=<slug>`

用途：返回某篇文章或留言板的评论树。

成功响应示例：

```json
{
  "comments": [
    {
      "id": 101,
      "name": "Dogxi",
      "avatar": "https://weavatar.com/avatar/xxx?s=80&d=identicon",
      "website": "https://example.com",
      "content": "这是一条评论",
      "createdAt": "2026-04-23T09:00:00.000Z",
      "userAgent": "Mozilla/5.0",
      "isOwner": true,
      "replies": [
        {
          "id": 102,
          "name": "Alice",
          "avatar": "https://weavatar.com/avatar/yyy?s=80&d=identicon",
          "content": "这是一条回复",
          "createdAt": "2026-04-23T09:10:00.000Z",
          "replyTo": {
            "id": 101,
            "name": "Dogxi"
          }
        }
      ]
    }
  ],
  "count": 1
}
```

字段要求：

- `comments` 必填，数组即可
- 顶层项会被前端按时间再次排序
- `replies` 可选；如果你直接返回评论树，前端会按树结构渲染

## `POST /api/submit-comment`

用途：提交评论或回复。

请求体示例：

```json
{
  "slug": "my-first-post",
  "title": "我的第一篇文章",
  "name": "Dogxi",
  "email": "hi@example.com",
  "website": "https://example.com",
  "content": "你好，这里是评论内容",
  "replyToId": "101",
  "replyToName": "Alice",
  "ownerToken": "secret-token",
  "_gotcha": "",
  "userAgent": "Mozilla/5.0"
}
```

字段说明：

- `slug`、`title`、`name`、`content` 为核心字段
- `replyToId` / `replyToName` 用于回复
- `ownerToken` 用于博主身份校验
- `_gotcha` 是 honeypot 字段，留空即可

成功响应示例：

```json
{
  "success": true,
  "message": "评论提交成功"
}
```

错误响应建议：

```json
{
  "error": "评论太频繁，请稍后再试"
}
```

推荐状态码：

- `400` 参数不合法
- `401` / `403` Token 或博主身份校验失败
- `429` 频率限制
- `500` 服务端异常

## `POST /api/add-thought`

用途：供 `/thoughts/new` 页面在线创建一条新的碎碎念。

请求头：

```text
Authorization: Bearer <THOUGHT_API_TOKEN>
Content-Type: application/json
```

请求体示例：

```json
{
  "content": "今天把博客模板整理干净了。",
  "tags": ["dev", "note"],
  "name": "template-update"
}
```

成功响应示例：

```json
{
  "success": true,
  "message": "碎碎念创建成功",
  "data": {
    "filename": "23_template-update.md",
    "filePath": "src/content/thoughts/2026/04/23_template-update.md",
    "github": {
      "commitUrl": "https://github.com/owner/repo/commit/xxx",
      "fileUrl": "https://github.com/owner/repo/blob/main/src/content/thoughts/2026/04/23_template-update.md"
    }
  }
}
```

前端会直接读取这些字段：

- `data.filename`
- `data.github.commitUrl`
- `data.github.fileUrl`

如果这些字段缺失，成功提示里的链接区域会无法正常显示。

## 推荐变量命名

如果你自己实现这组 API，可以参考 `.env.example`：

- `SITE_URL`
- `GITHUB_TOKEN`
- `COMMENTS_REPO`
- `OWNER_NAME`
- `OWNER_EMAIL`
- `OWNER_TOKEN`
- `THOUGHT_API_TOKEN`
- `CONTENT_REPO`
- `CONTENT_BRANCH`
