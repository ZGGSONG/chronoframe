---
paths:
  - "server/api/**"
  - "server/services/**"
---

# Server 规则

## 存储操作

API 路由中使用 `useStorageProvider(event)`：

```typescript
const provider = useStorageProvider(event)
const buffer = await provider.get(key)
const object = await provider.create(key, buffer, contentType)
```

## 服务目录结构

- `server/services/image/` — 图像处理（EXIF、缩略图、ThumbHash）
- `server/services/storage/` — 存储提供商（s3/local/openlist）
- `server/services/pipeline-queue/` — 异步处理队列

## API 路由

- 文件路径自动生成路由（Nitro 自动导入）
- 使用 `requireUserSession(event)` 进行认证检查
