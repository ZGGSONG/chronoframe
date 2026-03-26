---
paths:
  - "server/database/**"
  - "server/utils/db.ts"
  - "drizzle.config.ts"
---

# Database 规则

## 访问方式

仅使用 `server/utils/db.ts`：

```typescript
import { useDB, tables, eq } from '~~/server/utils/db'
await useDB().select().from(tables.photos).where(eq(tables.photos.id, id))
```

禁止直接导入 `drizzle-orm` 或 `better-sqlite3`。

## Schema 变更流程

1. 修改 `server/database/schema.ts`
2. 执行 `pnpm db:generate` 生成迁移
3. 禁止手动修改已有迁移文件名
4. 生产环境只使用迁移，不使用 `db:push`

## 数据库文件

- 文件位置：`data/app.sqlite3`
- WAL 模式已启用
