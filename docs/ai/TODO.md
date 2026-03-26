# 性能优化 TODO

基于性能瓶颈分析结果，按优先级排序的优化任务清单。

---

## 优先级说明

- **P0 (Critical)**: 严重影响性能或可能导致系统崩溃，必须立即修复
- **P1 (High)**: 明显影响用户体验，应尽快处理
- **P2 (Medium)**: 有优化空间，可在后续迭代中处理
- **P3 (Low)**: 锦上添花型优化

---

## 后端优化任务

### P0 - 数据库查询优化

- [ ] **分页查询实现**
  - 文件: `server/api/photos/index.get.ts`
  - 问题: 全表查询无分页，一次性加载所有照片数据
  - 方案: 添加 cursor/limit 分页，默认每页 50-100 条
  - 影响: 首屏加载时间，内存占用

- [ ] **字段选择优化**
  - 文件: `server/api/photos/index.get.ts`
  - 问题: `select()` 返回所有字段，包括大型 EXIF JSON
  - 方案: 明确指定所需字段，列表页不返回完整 EXIF
  - 影响: 响应体大小，传输时间

- [ ] **缓存层引入**
  - 文件: `server/api/photos/index.get.ts`
  - 问题: 每次请求都直接查询数据库
  - 方案: 添加 Redis/Nitro 缓存，TTL 5-10 分钟
  - 影响: 数据库压力，重复查询

### P0 - 图片处理管道优化

- [ ] **并行阶段处理**
  - 文件: `server/services/pipeline-queue/manager.ts`
  - 问题: 7 个处理阶段串行执行，CPU 利用率低
  - 方案: 独立阶段并行化（缩略图生成与 EXIF 提取可同时进行）
  - 影响: 任务吞吐量，上传后处理延迟

- [ ] **Worker Threads 迁移**
  - 文件: `server/services/image/processor.ts`, `server/services/image/thumbnail.ts`
  - 问题: Sharp 处理在主线程执行，阻塞事件循环
  - 方案: 使用 Node.js Worker Threads 隔离图像处理
  - 影响: API 响应性，并发处理能力

- [ ] **Sharp 实例池化**
  - 文件: `server/services/image/thumbnail.ts`
  - 问题: 每次处理新建 Sharp 实例，初始化开销大
  - 方案: 实现 Sharp 实例池，复用处理管道
  - 影响: 内存分配，处理延迟

### P1 - 队列系统优化

- [ ] **事件驱动替代轮询**
  - 文件: `server/services/pipeline-queue/worker-pool.ts`
  - 问题: 固定 2 秒轮询数据库，空闲时浪费资源
  - 方案: 使用 LISTEN/NOTIFY 或 Redis Pub/Sub 实现事件通知
  - 影响: 数据库负载，任务响应延迟

- [ ] **背压机制实现**
  - 文件: `server/services/pipeline-queue/worker-pool.ts`
  - 问题: 任务堆积时仍按固定速率处理，可能 OOM
  - 方案: 添加队列长度监控，超过阈值时暂停新增任务
  - 影响: 系统稳定性，内存使用

- [ ] **连接池管理**
  - 文件: `server/services/pipeline-queue/worker-pool.ts`
  - 问题: 所有工作器共享单一数据库连接
  - 方案: 配置 Drizzle/Better-SQLite3 连接池
  - 影响: 高并发下的数据库吞吐

### P2 - 静态资源优化

- [ ] **HTTP 缓存头配置**
  - 文件: `server/routes/image/[...key].get.ts`
  - 问题: 无 `Cache-Control`、`ETag` 头
  - 方案: 根据文件内容哈希设置长期缓存（immutable）
  - 影响: 重复下载，带宽消耗

- [ ] **响应压缩启用**
  - 文件: `server/routes/image/[...key].get.ts`
  - 问题: 未启用 gzip/brotli 压缩
  - 方案: Nitro 配置压缩中间件
  - 影响: 传输体积

- [ ] **格式自适应**
  - 文件: `server/routes/image/[...key].get.ts`
  - 问题: 直接返回原始格式（可能是 HEIC）
  - 方案: 根据 Accept 头自动转码为 WebP/JPEG
  - 影响: 浏览器兼容性，渲染性能

---

## 前端优化任务

### P0 - 瀑布流虚拟滚动

- [ ] **虚拟列表实现**
  - 文件: `app/components/masonry/Root.vue`
  - 问题: 无虚拟滚动，一次性渲染所有照片
  - 方案: 使用 `vue-virtual-scroller` 或自研虚拟滚动，只渲染可见区域
  - 影响: DOM 节点数量，滚动帧率

- [ ] **统计计算优化**
  - 文件: `app/components/masonry/Root.vue`
  - 问题: `photoStats` computed 遍历全量数据 4 次
  - 方案: 使用 Web Worker 异步计算，或改为懒加载统计
  - 影响: 主线程阻塞，响应延迟

- [ ] **可见性检测优化**
  - 文件: `app/components/masonry/Root.vue`
  - 问题: `updateDateRange` 每次滚动都重新计算日期和城市
  - 方案: 防抖处理，或增量更新可见集合
  - 影响: 滚动性能，CPU 占用

### P1 - WebGL 图像查看器优化

- [ ] **大图片异步纹理上传**
  - 文件: `packages/webgl-image/src/core/WebGLImageViewerEngine.ts`
  - 问题: `createTexture` 同步处理超大图片，阻塞主线程
  - 方案: 使用 `gl.texStorage2D` + `gl.texSubImage2D` 分块上传，或离屏 Canvas 在 Worker 中预处理
  - 影响: 查看大图时 UI 冻结

- [ ] **瓦片创建优化**
  - 文件: `packages/webgl-image/src/core/WebGLImageViewerEngine.ts`
  - 问题: `createTiles` 双重循环同步创建 Canvas 上下文
  - 方案: 瓦片懒加载（只创建可见瓦片），或使用 `createImageBitmap` 解码后直接上传 GPU
  - 影响: 初始加载时间，内存峰值

- [ ] **渲染节流优化**
  - 文件: `packages/webgl-image/src/core/WebGLImageViewerEngine.ts`
  - 问题: 拖拽时使用 16ms throttle，动画用 rAF，可能 GPU 过载
  - 方案: 动态调整节流间隔，检测 GPU 帧率自适应降级
  - 影响: 低性能设备的电池消耗

- [ ] **图片加载延迟启动**
  - 文件: `app/libs/image-loader-manager.ts`
  - 问题: `setTimeout(..., 300)` 延迟 300ms 才开始加载图片
  - 方案: 移除延迟，或使用 requestIdleCallback 在空闲时加载
  - 影响: 图片切换时的感知延迟

- [ ] **WebGL 上下文复用**
  - 文件: `app/components/photo/ProgressiveImage.vue`
  - 问题: `v-if` 条件渲染导致 WebGL 上下文频繁销毁重建
  - 方案: 使用 `v-show` 或透明度控制，复用 WebGL 实例
  - 影响: 图片切换时的初始化开销

### P1 - LivePhoto 处理器优化

- [ ] **事件驱动替代轮询**
  - 文件: `app/composables/useLivePhotoProcessor.ts`
  - 问题: `setInterval` 100ms 轮询等待处理完成
  - 方案: 使用 Promise + 状态订阅模式
  - 影响: 不必要的 CPU 循环，电池消耗

- [ ] **有界缓存实现**
  - 文件: `app/composables/useLivePhotoProcessor.ts`
  - 问题: `processedLivePhotos` Map 无大小限制
  - 方案: 实现 LRU 缓存，最大 50 条，配合内存压力监听
  - 影响: 内存泄漏风险，长时间使用后的页面崩溃

- [ ] **真实视频转码**
  - 文件: `app/composables/useLivePhotoProcessor.ts`
  - 问题: 仅重新包装 Blob 为 MP4 MIME 类型，无实际转码
  - 方案: 使用 FFmpeg.wasm 客户端转码，或后端预转码存储
  - 影响: iOS/Android 视频播放兼容性

### P2 - 组件渲染优化

- [ ] **Swiper 虚拟滚动修复**
  - 文件: `app/components/photo/Viewer.vue`
  - 问题: `slides-per-view="1"` 导致 Virtual 模块缓存失效
  - 方案: 调整缓存策略，预加载前后 3 张图片
  - 影响: 大图切换时的白屏

- [ ] **ThumbHash 计算缓存**
  - 文件: `app/components/ui/ThumbHash.vue`
  - 问题: `computed` 中每次渲染都执行 `thumbHashToDataURL`
  - 方案: 使用 `shallowRef` 缓存 DataURL，hash 变化时才重新计算
  - 影响: 列表滚动时的重复计算

- [ ] **组件状态瘦身**
  - 文件: `app/components/photo/Viewer.vue`
  - 问题: 单个 Viewer 实例维护 20+ 个 ref
  - 方案: 合并相关状态，使用组合式函数拆分逻辑
  - 影响: 内存占用，实例创建开销

- [ ] **图片缓存容量扩容**
  - 文件: `app/libs/image-loader-manager.ts`
  - 问题: LRUCache 只缓存 6 张图片，快速切换时命中率低
  - 方案: 增大到 20-30 张，或根据设备内存动态调整
  - 影响: 重复浏览时的加载速度

---

## 跨端优化任务

### P2 - 缓存策略统一

- [ ] **前端缓存层**
  - 方案: IndexedDB 存储已加载的缩略图，Service Worker 拦截图片请求
  - 影响: 重复浏览时的加载速度

- [ ] **后端缓存层**
  - 方案: CDN 边缘缓存 + Redis 响应缓存
  - 影响: 服务端负载，全球访问速度

### P3 - HEIC 处理优化

- [ ] **后端预转换**
  - 方案: 上传时后台转换为 WebP 多分辨率版本
  - 影响: 前端兼容性，实时转换 CPU 消耗

- [ ] **前端渐进加载**
  - 方案: 先加载 ThumbHash 占位，再加载低分辨率预览，最后原图
  - 影响: 感知加载速度

---

## 性能监控任务

- [ ] **前端性能埋点**
  - 首屏加载时间 (FCP, LCP)
  - 交互响应延迟 (INP)
  - 内存使用监控
  - 帧率监控（瀑布流滚动、WebGL 渲染）

- [ ] **后端性能埋点**
  - API 响应时间 P50/P95/P99
  - 图片处理管道各阶段耗时
  - 队列长度和等待时间
  - 数据库查询慢日志

- [ ] **性能回归测试**
  - Lighthouse CI 集成
  - 大数量级照片（1万+）压力测试
  - 低性能设备（4G 网络、低端 Android）测试

---

## 预估收益

| 优化项 | 预期收益 | 实施难度 |
|--------|---------|---------|
| 数据库分页 | 首屏加载 -80% | 低 |
| 虚拟滚动 | DOM 节点 -90%，滚动流畅 | 中 |
| Worker Threads | CPU 利用率 +200% | 中 |
| 缓存层 | 重复请求 -95% | 低 |
| WebGL 异步化 | 大图查看无卡顿 | 高 |
| 事件驱动队列 | 数据库负载 -70% | 中 |

---

*最后更新: 2026-03-26*

---

## 图片切换卡顿专项分析

### 问题描述

部署后图片查看器在切换图片时出现明显卡顿，UI 响应延迟，影响用户体验。

### 根因分析

| 原因 | 占比 | 详细说明 |
|------|------|----------|
| **WebGL 纹理同步创建** | 60% | Worker 加载图片后，在主线程同步执行 `createTexture`/`createTiles`，大图分块时阻塞渲染 |
| **加载延迟启动** | 20% | `image-loader-manager.ts` 中 300ms `setTimeout` 延迟才开始加载 |
| **LivePhoto 同步处理** | 15% | 切换时立即调用 `convertMovToMp4`，视频转换阻塞 UI |
| **缓存容量不足** | 5% | LRU 只缓存 6 张，快速切换时频繁重新加载 |

### 关键代码位置

```typescript
// 1. WebGL 同步纹理创建 (packages/webgl-image/src/core/WebGLImageViewerEngine.ts:220-261)
// Worker 回调后在主线程同步创建纹理/瓦片
this.worker.onmessage = (event) => {
  const { imageBitmap } = event.data.payload;
  // 同步执行，阻塞主线程
  if (shouldUseTiles) {
    this.createTiles(imageBitmap);  // 双重循环创建 Canvas
  } else {
    this.createTexture(imageBitmap); // 同步上传 GPU
  }
}

// 2. 300ms 加载延迟 (app/libs/image-loader-manager.ts:74)
this.timer = setTimeout(async () => {
  // 开始 XMLHttpRequest 加载
}, 300)

// 3. LivePhoto 同步处理 (app/components/photo/Viewer.vue:258-275)
watch(() => props.currentIndex, () => {
  // 立即处理，无延迟
  processCurrentLivePhoto() // 内含 await convertMovToMp4
})

// 4. 缓存容量过小 (app/libs/image-loader-manager.ts:32)
new LRUCache<string, ImageLoaderCacheResult>(6, ...) // 仅 6 张
```

### 优化优先级

**立即实施 (5分钟见效):**
1. 移除/减少 300ms 加载延迟
2. 增大 LRU 缓存到 20-30 张
3. LivePhoto 使用 `requestIdleCallback` 延迟处理

**短期实施 (需要测试):**
4. `createTiles` 改为异步分帧执行
5. WebGL 组件使用 `v-show` 替代 `v-if` 复用上下文
6. 预加载前后各 1-2 张图片

**长期实施 (架构改动):**
7. WebGL 纹理创建移入 Worker (OffscreenCanvas)
8. 分块瓦片懒加载（只创建可见瓦片）
