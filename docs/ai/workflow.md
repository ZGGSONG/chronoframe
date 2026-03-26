# 开发流程规则

## 修改前

- 涉及 2 个以上文件的修改：先用 EnterPlanMode 制定计划
- 涉及数据库 schema 变更：先检查现有迁移文件命名规则

## 修改后（必须执行）

```bash
pnpm lint && pnpm build
```

两项必须全部通过。

## 依赖管理

- 禁止直接修改 `package.json` 添加依赖 —— 使用 `pnpm add <pkg>`
- 工作区包依赖使用 `workspace:*` 协议

## 数据库文件

- 文件位置：`data/app.sqlite3`
- WAL 模式已启用
