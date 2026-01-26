<div align="center">
  <img src="./public/logo.svg" width="120" alt="Sat-Hunter Logo" />
  <h1>Sat-Hunter</h1>
  <p>
    <b>卫星底图下载器</b>
    <br/>
    基于 Vue 3 和 Leaflet 的在线卫星瓦片底图下载工具
  </p>
  
  <p>
    <a href="./LICENSE">
      <img src="https://img.shields.io/badge/license-Apache%202.0-blue.svg?style=flat-square" alt="License">
    </a>
    <img src="https://img.shields.io/badge/vue-3.x-42b883.svg?style=flat-square" alt="Vue 3">
    <img src="https://img.shields.io/badge/typescript-%5E5.0-3178C6.svg?style=flat-square" alt="TypeScript">
    <img src="https://img.shields.io/badge/vite-%5E7.0-646CFF.svg?style=flat-square" alt="Vite">
    <a href="https://github.com/knight-L/sat-hunter/pulls">
      <img src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square" alt="PRs Welcome">
    </a>
  </p>
</div>

## ✨ 主要功能

- **多源底图支持**：内置多种常用底图源，包括：
  - 高德卫星地图
  - Google 卫星地图
  - ESRI 影像 (Imagery, Firefly, Light/Dark Gray)
  - 地形图 (Hill Shade)
- **灵活的选区模式**：
  - **区域轮廓下载**：支持按行政区划（省、市、区/县）搜索并自动选中轮廓范围。
  - **矩形框选下载**：提供画图工具，支持在地图上自由绘制矩形框进行下载。
- **自定义缩放级别**：支持 6-18 级的缩放级别调整，满足不同精度的需求。
- **实时预览与估算**：
  - 实时显示选中区域的瓦片数量估算。
  - 大数据量警告（超过 1000 张瓦片提示，超过 1500 张限制下载），防止浏览器崩溃。
- **自动拼接下载**：后端自动获取所有切片并在前端 Canvas 中拼接，一键导出为图片。
- **主题适配**：
  - 支持 **暗色模式 (Dark Mode)**，根据系统设置自动切换或手动切换。

## 🛠️ 技术栈

- **前端框架**: [Vue 3](https://vuejs.org/) + [TypeScript](https://www.typescriptlang.org/)
- **构建工具**: [Vite](https://vitejs.dev/)
- **样式库**: [Tailwind CSS](https://tailwindcss.com/)
- **地图引擎**: [Leaflet](https://leafletjs.com/)
- **绘图插件**: [Leaflet-Geoman](https://geoman.io/leaflet-geoman)

## 🚀 快速开始

### 环境要求

- Node.js (推荐 v16+)
- pnpm (推荐) 或 npm / yarn

### 安装步骤

1. **克隆仓库**

   ```bash
   git clone https://github.com/your-username/sat-hunter.git
   cd sat-hunter
   ```

2. **安装依赖**

   ```bash
   pnpm install
   # 或者
   npm install
   ```

3. **启动开发服务器**

   ```bash
   pnpm dev
   # 或者
   npm run dev
   ```

4. **打开浏览器**

   访问终端输出的本地地址（通常是 `http://localhost:5173`）即可使用。

### 构建部署

构建生产环境版本：

```bash
pnpm build
```

构建产物将输出到 `dist` 目录，可部署至任何静态网站托管服务（如 Nginx, Vercel, Netlify）。

## 📖 使用指南

1. **选择底图**：在左侧设置面板底部，点击选择你需要的底图风格（如“高德卫星”或“Google 卫星”）。
2. **确定范围**：
   - **方式一（区域搜索）**：在“按区域轮廓下载”输入框中，输入城市名称（如“北京市”），选择对应的行政区。
   - **方式二（手动框选）**：点击地图左侧工具栏的黑色矩形图标（⬛），在地图上拖拽画出一个矩形框。
3. **调整精度**：拖动“缩放级别 (Zoom)”滑块。
   - 级别越高，地图越清晰，但文件体积和瓦片数量也会急剧增加。
   - 建议先关注“预计瓦片”数量，避免一次性下载过大区域。
4. **开始下载**：点击“开始下载”按钮。
   - 系统将自动下载所有瓦片并在本地拼接。
   - 进度条完成后，浏览器会自动弹出图片下载保存对话框。

## ⚠️ 注意事项

- **数据来源**：本工具使用的地图服务均来自公开的第三方瓦片服务（高德、Google、ESRI 等），仅供学习和科研使用。
- **跨域问题**：部分地图源可能存在跨域限制，本工具已配置 `crossOrigin: "anonymous"`，但仍可能受限于源站策略。
- **性能限制**：由于纯前端拼接图片，受限于浏览器内存，不建议一次性下载过大范围（建议瓦片数控制在 1000 张以内）。

## 📄 许可证

[Apache License 2.0](./LICENSE) License
