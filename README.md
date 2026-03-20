# PPT Agent -- 专业演示文稿全流程 AI 生成助手

模仿万元/页级别 PPT 设计公司的完整工作流，输出高质量 HTML 演示文稿 + 可编辑矢量 PPTX。

## 工作流概览

```
需求调研 -> 资料搜集 -> 大纲策划 -> 策划稿 -> 风格+配图+HTML设计稿 -> 后处理(SVG+PPTX)
```

## 输出产物

| 文件 | 说明 |
|------|------|
| `preview.html` | 浏览器翻页预览（自动生成） |
| `presentation.pptx` | PPTX 文件，PPT 365 中右键"转换为形状"可编辑 |
| `svg/*.svg` | 单页矢量 SVG，可直接拖入 PPT |
| `slides/*.html` | 单页 HTML 源文件 |

## 环境依赖

### 必须

- **Node.js** >= 18（Puppeteer + dom-to-svg 需要）
- **Python** >= 3.8（脚本执行）
- **python-pptx**（PPTX 生成）

### 自动安装（首次运行自动处理）

- `puppeteer` -- Headless Chrome
- `dom-to-svg` -- DOM 转 SVG（保留 `<text>` 可编辑）
- `esbuild` -- 将 dom-to-svg 打包为浏览器 bundle

### 可选（降级方案）

- `pdf2svg` -- 当 dom-to-svg 不可用时的降级方案（文字变 path，不可编辑）
- `inkscape` -- SVG 转 EMF（备用）

### 一键安装

```bash
# Python 依赖
pip install python-pptx lxml Pillow

# Node.js 依赖（首次运行脚本时自动安装，也可手动提前安装）
npm install puppeteer dom-to-svg

# 可选：降级方案
sudo apt install pdf2svg
```

## 目录结构

```
ppt-agent-workflow-san/
  SKILL.md                    # 主工作流指令（Agent 入口）
  README.md                   # 本文件
  references/
    prompts.md                # 5 套 Prompt 模板
    style-system.md           # 8 种预置风格 + CSS 变量
    bento-grid.md             # 7 种布局规格 + 卡片类型
    method.md                 # 核心方法论
  scripts/
    html_packager.py          # 多页 HTML 合并为翻页预览
    html2svg.py               # HTML -> SVG（dom-to-svg，保留文字可编辑）
    svg2pptx.py               # SVG -> PPTX（OOXML 原生 SVG 嵌入）
```

## 脚本用法

### html_packager.py -- 合并预览

```bash
python3 scripts/html_packager.py <slides_dir> -o preview.html
```

### html2svg.py -- HTML 转 SVG

```bash
python3 scripts/html2svg.py <slides_dir> -o <svg_dir>
```

- 底层：Puppeteer + dom-to-svg（DOM 直接转 SVG，`<text>` 可编辑）
- 图片：自动读取 `<img>` 引用的文件转 base64 嵌入
- 降级：dom-to-svg 不可用时自动退回 Puppeteer PDF + pdf2svg

### svg2pptx.py -- SVG 转 PPTX

```bash
python3 scripts/svg2pptx.py <svg_dir> -o output.pptx --html-dir <slides_dir>
```

- SVG 以 OOXML `asvg:svgBlip` 扩展原生嵌入 PPTX
- 同时生成 PNG 回退图（兼容旧版 Office）
- PPT 365 中右键 -> "转换为形状" 可编辑文字和形状

## 技术架构

```
HTML slides
  |
  v
[Puppeteer] 打开 HTML -> [dom-to-svg] DOM 直接转 SVG
  |                         (保留 <text> 元素，文字可编辑)
  |                         (base64 内联图片)
  |                         (color -> fill 属性后处理)
  v
SVG files
  |
  v
[python-pptx + lxml] OOXML svgBlip 嵌入
  |                    (PNG 回退图兼容旧版 Office)
  v
presentation.pptx
```

## 触发方式

在 Claude 对话中，以下表达会触发此 Skill：

- "帮我做个 PPT" / "做一个关于 X 的演示"
- "做 slides" / "做幻灯片" / "做汇报材料"
- "把这篇文档做成 PPT"
- "做培训课件" / "做路演 deck"
