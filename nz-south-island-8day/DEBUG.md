# DEBUG.md — 排错记录

> 新西兰南岛8天自驾环线 index.html 踩坑经验

## Bug #1：CSS变量全部失效 → 白屏

- **现象**：页面完全空白，无地图、无内容
- **原因**：CSS 第13行 `::root` 写成了双冒号（应为一个冒号 `:root`），导致所有 CSS 自定义属性未定义
- **修复**：`::root` → `:root`

## Bug #2：Leaflet CDN 国内被墙

- **现象**：`unpkg.com` 加载超时，地图不显示
- **原因**：`unpkg.com` 在国内访问受限
- **修复**：
  1. 先尝试 `cdn.bootcdn.net` — CORS 报错 `blocked: origin`
  2. 最终方案：下载 Leaflet 到本地 `lib/` 目录，完全自托管

## Bug #3：`Peter's` 单引号截断 JS 字符串

- **现象**：`Uncaught SyntaxError: Unexpected identifier 's'`
- **位置**：第327行 `detail` 字段内含 `Peter's Lookout`
- **原因**：JavaScript 单引号字符串内部的 ASCII 单引号 `'` 未被转义，提前结束了字符串
- **修复**：`Peter's` → `Peter\'s`
- **教训**：生成 JS 数据时，必须检查所有 detail/desc 等字符串字段中的单引号

## 经验总结

| 踩坑 | 教训 |
|------|------|
| CDN | 面向国内用户的页面，优先本地化所有外部依赖（CSS/JS/字体/瓦片） |
| 字符串 | 单引号字符串内的所有 `'` 需转义，双引号同理 |
| CSS | `:root` 是单冒号，`::before`/`::after` 才是双冒号 |
