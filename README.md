# sloped — 组件库项目

面向 AI 生成页面的组件库工作区。核心工作流：**design.md（规范源）→ 组件库页面（实现/验收）→ 快照生成业务页**。

## 目录职责

| 路径 | 作用 | 维护方式 |
|---|---|---|
| `组件库设计规范.html` | 运行文件：组件库页面，双击即开，三区分区（tokens/组件/演示） | 改样式只动 `<style>` 内 `:root`；新增组件按四联同步 |
| `design.md` | AI 规范源：契约、tokens、注册表、页面骨架、准入规则 | 与组件库同步维护，冲突以 HTML 实现为准 |
| `组件补全计划.md` | 对照 Ant Design/Semi 的组件缺口分析与 7 批入库计划 | 按批次执行，完成一批更新本表版本 |
| `references/` | 线上截图原料区：图 + 同名 `.md` 描述 + `_index.md` 索引 | 新增截图必配描述并登记索引 |
| `pages/` | AI 生成的业务页面归档（自包含单文件） | 生成后归档，命名含业务主题 |

## 快速开始

1. 双击 `组件库设计规范.html` 浏览组件与规范；
2. 让 AI 生成页面时，提供 `design.md` + 本文件路径，按 design.md §0 快照流程执行；
3. 新增线上截图 → 放 `references/` → 按 `_spec-template.md` 写同名描述 → 登记 `_index.md`。

## 当前版本

v1.4.3（2026-11-24）：两项修复——① v1.4.2 outline 按钮语义修正：`.btn.outline` 由 danger 描边（红）改为主色描边（蓝），对齐 Semi；新增 `.outline.danger` 保留危险描边。**破坏性变更**：旧用法需加 `.danger` 修饰符或改用 `.btn.danger`。② axe-core 无障碍审计修复：`pre.code` 自动设 `tabindex="0"`，键盘可滚动查看代码块。全页面 WCAG 2.x AA 级通过：0 violations / 19 passes。详见 design.md §6 changelog。

> AI生成
