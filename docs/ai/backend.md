# 后端开发规范

## 项目结构

```
server/
  api/                # API 路由（自动导入）
  database/schema.ts  # Drizzle schema
  services/           # 业务逻辑
    image/            # 图像处理（EXIF、缩略图、ThumbHash）
    storage/          # 存储提供商（s3/local/openlist）
    pipeline-queue/   # 异步处理队列
  utils/db.ts         # 数据库入口
```

## 数据库访问

**强制规则**：只使用 `server/utils/db.ts`

```typescript
import { useDB, tables, eq, and, or } from '~~/server/utils/db'

// 查询
await useDB().select().from(tables.photos).where(eq(tables.photos.id, id))

// 插入
await useDB().insert(tables.photos).values(photoData)

// 更新
await useDB().update(tables.photos).set({ title: '新标题' }).where(eq(tables.photos.id, id))
```

**禁止**：直接导入 `drizzle-orm` 或 `better-sqlite3`

## Schema 变更流程

1. 修改 `server/database/schema.ts`
2. 执行 `pnpm db:generate` 生成迁移
3. **禁止**手动修改已有迁移文件名
4. 生产环境只用迁移，不使用 `db:push`

## 存储操作

API 路由中使用 `useStorageProvider(event)`：

```typescript
const provider = useStorageProvider(event)
const buffer = await provider.get(key)
const object = await provider.create(key, buffer, contentType)
```

## API 路由

- 文件路径自动生成路由（Nitro 自动导入）
- 使用 `requireUserSession(event)` 进行认证检查

## 提交前检查

```bash
pnpm lint && pnpm build
```

必须全部通过。
