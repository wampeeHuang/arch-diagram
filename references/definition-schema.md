# 定义 JSON Schema

架构图定义文件描述三件事：节点是什么、谁连谁、怎么分行分组。

## 顶层

```json
{
  "title": "图标题",
  "phases": [...],
  "nodes": {...},
  "edges": [...]
}
```

| 字段 | 类型 | 必需 | 说明 |
|------|------|------|------|
| `title` | string | ✅ | 图标题，显示在左上角 |
| `phases` | Phase[] | ✅ | Phase 列表，从上到下排列 |
| `nodes` | {id: Node} | ✅ | 所有节点定义，key 为节点 ID |
| `edges` | Edge[] | ✅ | 所有边定义 |

## Phase

| 字段 | 类型 | 必需 | 说明 |
|------|------|------|------|
| `id` | string | ✅ | 唯一标识 |
| `num` | string | ✅ | 大号数字，如 `"0"`、`"1"`。36px mono 字体作为视觉锚点 |
| `label` | string | ✅ | 小号标签，如 "自检门禁"。10px uppercase |
| `title` | string | ❌ | 短标题，如 "框架自我审计"。14px，显示在 label 下方 |
| `phaseBg` | string | ✅ | CSS 背景色，如 `#FEF2F2` |
| `dotColor` | string | ✅ | 数字和标签颜色 |
| `rows` | string[][] | ✅ | 二维数组，每行是节点 ID 列表。`null` 表示空单元格 |

## Node

| 字段 | 类型 | 必需 | 默认 | 说明 |
|------|------|------|------|------|
| `shape` | string | ❌ | `rect` | rect / diamond / hexagon / page |
| `color` | string | ❌ | `process` | gate / decision / success / process / output / term / start |
| `text` | string | ✅ | — | 节点文字，`\n` 换行 |

## Edge

| 字段 | 类型 | 必需 | 默认 | 说明 |
|------|------|------|------|------|
| `from` | string | ✅ | — | 源节点 ID |
| `to` | string | ✅ | — | 目标节点 ID |
| `label` | string | ❌ | — | 边标签文字，`\n` 换行 |
| `style` | string | ❌ | `solid` | solid / loop / feedback |

### edge style 语义

| style | 渲染效果 | 使用场景 |
|-------|---------|---------|
| `solid` | 灰色实线 `#D4D4D4`，灰色箭头 | 正向流程 |
| `loop` | 橙色虚线 `#F97316`，橙色箭头，自动绕右侧 cubic bezier | 回退/重试边（目标在源上方时自动触发） |
| `feedback` | 灰色虚线 `#A3A3A3`，灰色箭头，低透明度，绕右侧 | 自迭代闭环（手动标记） |
