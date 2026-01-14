# Project Context: Scallop (Experimental Music Portfolio)

## 1. 项目概况
- **类型**: 个人作品集 / 数字博客 / 实验音乐
- **核心目标**: 展示音视频作品，发布文章/实验/笔记
- **当前状态**: 已上线 (Deployed)
- **Live URL**: [https://scallop-gdu.pages.dev](https://scallop-gdu.pages.dev)
- **Repo**: [GitHub (khipu-stack/scallop)](https://github.com/khipu-stack/scallop)

## 2. 技术栈 (Tech Stack)
- **框架**: Astro (v4+), 使用 `npm create astro@latest -- --template minimal` 初始化。
- **语言**: HTML, CSS (Raw/Global), JavaScript (Minimal), Markdown.
- **构建工具**: Vite (Astro 内置).
- **包管理器**: npm.
- **部署**: Cloudflare Pages (连接 GitHub 自动构建).
  - Framework Preset: `Astro`
  - Build Command: `npm run build`
  - Output Dir: `dist`

## 3. 设计系统 (Visual Design System)
- **风格**: Raw Brutalism / Terminal.
- **布局**: 自定义布局组件 `src/layouts/RawLayout.astro`。
- **色彩规范**:
  - 背景: `#050505` (接近纯黑).
  - 正文: `#d4d4d4` (浅灰，高可读性).
  - 强调色/骨架: `#00ff41` (绿，用于 H1-H3, 链接, 边框).
  - 边框: 硬边框，无圆角 (`border-radius: 0`)，`1px solid #333` 或 `#00ff41`。
- **字体**: Monospace (`Courier New`, `PingFang SC`).
- **排版**:
  - `max-width: 750px`.
  - 图片自适应 (`max-width: 100%`) 并带有深灰边框。

## 4. 文件结构与内容策略
- **src/layouts/RawLayout.astro**: 全局布局模版，包含 `<slot />` 和全局 CSS (`<style is:global>`)。
- **src/pages/index.astro**: 首页，硬编码的导航和简介。
- **src/pages/music.astro**: 音乐页。使用 `<iframe>` 嵌入 Bandcamp 播放器。
- **src/pages/notes/**: 写作系统。
  - `*.md` 文件: 实际的文章内容，Frontmatter 指定 `layout`.
  - `index.astro`: 使用 `Astro.glob` 抓取所有 `.md` 文件并生成列表。
- **public/images/**: 存放静态图片，Markdown 中通过 `/images/filename.jpg` 引用。

## 5. 开发工作流
1. 本地修改代码/添加 Markdown 文件。
2. 终端运行 `npm run dev` 预览。
3. Git 提交: `git push origin main` 。
4. Cloudflare Pages 自动触发构建并发布。
