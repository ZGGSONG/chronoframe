# AI 开发指南

> 本文档供 AI 助手开发时参考

## 快速命令

```bash
pnpm dev              # 开发服务器
pnpm build            # 生产构建（修改后必须执行）
pnpm lint             # ESLint（提交前必须执行）
pnpm db:generate      # 生成迁移文件
pnpm db:migrate       # 执行迁移
```

## 文档导航

| 文档 | 内容 |
|-----|------|
| [backend.md](./backend.md) | 后端开发规范（数据库、API、存储） |
| [frontend.md](./frontend.md) | 前端开发规范（组件、自动导入） |
| [workflow.md](./workflow.md) | 开发流程规则 |

## 项目结构

```
app/                    # 前端（Vue/Nuxt）
server/
  api/                  # API 路由
  database/schema.ts    # Drizzle schema
  services/             # 业务逻辑
packages/webgl-image/   # WebGL 包
shared/types/           # 共享类型
```

## 禁止修改

- `packages/webgl-image/dist/`
- `data/`
- `pnpm-lock.yaml`

## 修改前检查

- 涉及 2 个以上文件：先用 EnterPlanMode 制定计划
- 涉及数据库 schema：先检查迁移文件命名规则

## 环境变量

- `NUXT_STORAGE_PROVIDER`: `s3` | `local` | `openlist`
- `NUXT_SESSION_PASSWORD`: 32 字符随机字符串
