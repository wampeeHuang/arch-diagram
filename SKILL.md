---
name: arch-diagram
description: |
  架构图生成。三步流水线：结构定义(节点·边·Phase分组) → 规则引擎(色板·等宽·间距固化) → CSS Grid + SVG 渲染输出自包含 HTML。

  Use this skill when the user says: "生成架构图", "画架构图", "更新架构图", "做一张系统图", "流程图", "生成 diagram", "把这个流程画出来", "做一张架构图", "重建 system-diagram"。

  Also trigger when: the user is describing a multi-step workflow or system architecture and wants a visual diagram; the user references an existing skill's flow and wants it visualized; the user says a diagram "looks bad" and wants a redesigned version.
---

# arch-diagram · 架构图生成

> 不猜布局，不算图论。用 CSS Grid 做空间排版，用 SVG 叠边线。
> 设计规则固化为代码常量，零审美判断。

## 能力边界（诚实边界）

| 边界 | 说明 |
|------|------|
| **只管渲染** | 输入是已定义好的节点+边+分组。skill 不推断、不补全、不自创节点 |
| **网格布局** | 所有节点吸附 CSS Grid。不支持自由坐标、不支持绝对定位、不支持重叠 |
| **4 色硬限制** | gate(红) / decision(橙) / success(绿) / output(紫) + process(白) + term(深红)。不加色、不混色、不渐变 |
| **流向约束** | 主边从上到下(→↓)，回退边自动识别(目标在上方=虚线红色绕左侧)。不处理环形依赖、不自创路由 |
| **单图单出** | 一次调用生成一张图。不批量、不拼接、不生成多视图 |
| **不自决内容** | 节点文本、Phase 分组、边标签均由调用方提供。skill 不判断"该不该有这条边" |
| **输出格式单一** | 只出独立 HTML 文件。不出 SVG/PNG/PDF/Mermaid/D2 |
| **不读源码** | skill 不分析项目代码、不推断架构。需要的信息必须在定义 JSON 里写明 |

## 三步流水线

```
结构定义(JSON) → 规则引擎(代码常量) → HTML 输出
```

### Step 1: 结构定义

写一个 JSON，描述三件事：节点是什么、谁连谁、怎么分行分组。

完整 schema 见 `references/definition-schema.md`。

最小示例：

```json
{
  "title": "我的架构图",
  "phases": [
    {
      "id": "p1",
      "num": "1",
      "label": "输入",
      "title": "数据入口",
      "phaseBg": "#FEF2F2",
      "dotColor": "#EF4444",
      "rows": [
        ["nodeA"],
        ["nodeB", "nodeC"]
      ]
    }
  ],
  "nodes": {
    "nodeA": { "shape": "rect", "color": "gate", "text": "节点A\n说明文字" },
    "nodeB": { "shape": "diamond", "color": "decision", "text": "判断?" },
    "nodeC": { "shape": "rect", "color": "success", "text": "通过" }
  },
  "edges": [
    { "from": "nodeA", "to": "nodeB", "label": "触发" },
    { "from": "nodeB", "to": "nodeC", "label": "是" }
  ]
}
```

### Step 2: 规则引擎

以下规则已固化为代码常量，不可配置、不可覆盖：

| 规则 | 实现 |
|------|------|
| **色板** | 6 类颜色(gate/decision/success/process/output/term)，每类 bg+border+text 三色 |
| **同相等宽** | 每个 Phase 内 `grid-template-columns: repeat(N, 1fr)`，N = 最大列数 |
| **间距下限** | Phase 间 32px，节点间 16px |
| **大号编号** | Phase 标题区 36px mono 数字作为视觉锚点 |
| **回退边降权** | 目标节点在源节点上方 → 自动橙色虚线+绕右侧 cubic bezier |
| **反馈边** | `style: "feedback"` → 灰色虚线+更低透明度+绕右侧 |
| **箭头** | 正向灰色箭头，loop 橙色箭头，feedback 灰色箭头（3 组独立 marker） |
| **节点形状** | rect(圆角矩形)/diamond(菱形 clip-path)/hexagon(六边形)/page(折角矩形) |

### Step 3: 输出

1. 读 `assets/renderer.html` 模板
2. 把定义 JSON 替换 `__DEFINITION__`，标题替换 `__TITLE__`
3. 写入目标路径（通常是 skill 的 `references/system-diagram.html`）
4. 浏览器打开验证

## 工作流程

当用户要求生成或重建架构图时：

1. **收集信息** — 如果用户已有明确流程，直接写定义。如果不确定，先确认节点/Phase/边再写定义
2. **写定义 JSON** — 按 schema 写。不确定的字段留默认值（shape 默认 rect，color 默认 process）
3. **组装 HTML** — 读 `assets/renderer.html`，替换占位符，写入目标路径
4. **验证** — `Start-Process` 在浏览器打开，检查节点位置、边线走向、颜色是否正确
5. **修正** — 如果布局不对，调整 Phase 的 rows 分组，重新生成

## 颜色速查

| color | 用途 | bg | border | text |
|-------|------|-----|--------|------|
| `gate` | 门禁/阻断 | `#FEF2F2` | `#EF4444` | `#B91C1C` |
| `decision` | 判断/分支 | `#FFF7ED` | `#F97316` | `#9A3412` |
| `success` | 通过/确认 | `#ECFDF5` | `#10B981` | `#065F46` |
| `process` | 中性步骤 | `#FFFFFF` | `#CBD5E1` | `#1E293B` |
| `output` | 产出/报告 | `#F5F3FF` | `#8B5CF6` | `#5B21B6` |
| `term` | 终止/否决 | `#FEF2F2` | `#DC2626` | `#991B1B` |
| `start` | 入口 | `#F8FAFC` | `#94A3B8` | `#475569` |

## Phase 背景色约定

| 场景 | phaseBg |
|------|---------|
| 红色警戒区（门禁/阻断/终止） | `#FEF2F2` |
| 中性处理区（评审/评分/盘查） | `#FBFBFB` |
| 紫色输出区（报告/文件产出） | `#F5F3FF` |
| 入口区 | 不留背景（继承 body） |

## 参考文件

- `references/definition-schema.md` — 完整 JSON schema，所有字段和可选值
- `assets/renderer.html` — HTML 模板，CSS Grid + SVG 渲染引擎
