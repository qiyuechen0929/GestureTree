# GestureTree · 手势圣诞树

![banner](banner.svg)


> 纯前端 · 无构建 · 单文件 WebAR 手势交互应用

[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Three.js](https://img.shields.io/badge/Three.js-r128-000000.svg)](https://threejs.org/)
[![MediaPipe](https://img.shields.io/badge/MediaPipe-Hands-00e5ff.svg)](https://developers.google.com/mediapipe)
[![Status](https://img.shields.io/badge/Status-Demo-green.svg)](#)

## 简介

`GestureTree` 是一个基于浏览器摄像头的**手势驱动的 3D 圣诞装饰应用**。上传照片后，你的手掌就是遥控器——通过四种手势在**圣诞树、照片星系、单张特写、巨型艺术字**四种炫酷模式间自由切换：

- ✋ **张开手掌** → 照片星系爆发
- ✊ **握拳** → 修长悬浮圣诞树
- ☝️ **单指** → 单张照片特写（绝对静止）
- 👌 **OK 手势** → 巨型艺术字

项目采用 **MediaPipe Hands** 完成实时手部关键点追踪与手势识别，**Three.js** 负责 3D 渲染，搭配雪花粒子、礼物粒子、树顶星星与螺旋光带，营造浓郁的圣诞氛围。全部代码压缩在**单个 HTML 文件**中，无需任何构建工具即可运行。

## 功能特性

- 🎄 **手势四模切换**：张开 / 握拳 / 单指 / OK 四种手势，对应四种不同的展示模式
- 🌌 **照片星系**：将上传的照片炸裂成漫天漂浮的照片星云，随手势轻微倾斜
- 🎅 **修长悬浮树**：2400 个粒子（方块、彩球、相框、礼物）实时聚合成圣诞树形态
- 🖼️ **单张特写**：绝对静止地展示单张照片，粒子彻底冻结，聚焦满分
- ✨ **巨型艺术字**：金灿灿的 "Merry Christmas" 发光艺术字，霓虹描边
- 📷 **照片导入**：支持批量上传照片（建议 10+ 张，上限 30 张），全部在浏览器本地处理
- ❄️ **沉浸式圣诞氛围**：800 片雪花飘落、树顶旋转星星、金色螺旋光带、礼物粒子
- ⚡ **零构建单文件**：一个 HTML 搞定全部逻辑与样式，复制即用

## 技术栈

| 技术 | 用途 |
| --- | --- |
| [MediaPipe Hands](https://developers.google.com/mediapipe) | 实时手部关键点追踪、多手势识别 |
| [Three.js r128](https://threejs.org/) | 3D 场景构建与渲染（InstancedMesh 高性能粒子） |
| Canvas 2D | 程序化纹理生成（雪花、礼物、艺术字） |
| FileReader | 本地照片读取与纹理转换 |

## 快速开始

### 方式一：直接打开

将 `decorate the Christmas tree.html` 下载到本地，**双击用浏览器打开**即可。

> ⚠️ **注意**：手势识别依赖摄像头，请使用 **HTTPS** 环境或本地文件访问，并授予浏览器摄像头权限。若本地文件（`file://`）下摄像头无法调用，请使用方式二。

### 方式二：本地服务器运行

```bash
# Python
python -m http.server 8080

# 或 Node.js
npx serve .

# 然后访问
open http://localhost:8080
```

### 方式三：在线部署

将 HTML 文件直接拖入任意静态托管平台（GitHub Pages、Vercel、Netlify、Cloudflare Pages 等）即可上线。

## 使用说明

1. **导入照片**：点击左侧「导入多张照片」按钮，批量选择本地图片（建议 10 张以上，效果最佳）
2. **激活手势**：页面加载后，让手掌进入摄像头画面，左上角视窗会显示实时追踪状态
3. **切换模式**：
   - **张开手掌** → 照片星系爆发，照片在 3D 空间中漂浮旋转
   - **握拳** → 粒子聚合为悬浮圣诞树，树顶星星闪烁
   - **单指** → 随机单张照片特写，画面绝对静止
   - **OK 手势** → 巨型 "Merry Christmas" 艺术字
4. **控制视角**：手掌在画面中移动，3D 场景会跟随手势轻微旋转（特写模式除外）

## 项目结构

```
gesture-tree/
└── decorate the Christmas tree.html   # 全部代码（样式 + 逻辑 + 3D 场景），单文件即项目
```

## 核心机制

```
摄像头输入
   ↓ MediaPipe Hands 手部关键点追踪
   ↓ 手势识别（FIST / POINTING / OK / OPEN）
   ↓ 模式切换
   ├─ OPEN     → nebula 照片星系（粒子炸散 + 照片云展开）
   ├─ FIST     → tree  粒子聚合成圣诞树
   ├─ POINTING → photo 单张特写（粒子冻结）
   └─ OK       → text  巨型艺术字
   ↓ Three.js 粒子补间动画 + 渲染 → 屏幕
```

## 手势识别逻辑

项目通过手部关键点间的相对距离判断手指弯曲状态，进而识别四种手势：

```js
function detect(lm) {
    const d = (i1, i2) => Math.hypot(lm[i1].x-lm[i2].x, lm[i1].y-lm[i2].y);
    const o = (t, p) => d(t, 0) > d(p, 0);
    const i=o(8,6), m=o(12,10), r=o(16,14), p=o(20,18);
    if(!i&&!m&&!r&&!p) return 'FIST';        // 全握 → 圣诞树
    if(i && !m && !r && !p) return 'POINTING'; // 单指 → 照片特写
    if(d(4,8)<0.05&&m&&r&&p) return 'OK';     // OK 圈 → 艺术字
    if(i&&m&&r&&p) return 'OPEN';             // 张开 → 照片星系
    return 'UNKNOWN';
}
```

## 浏览器兼容性

- 支持 WebGL 与 `getUserMedia` 的现代浏览器（Chrome / Edge / Firefox / Safari）
- 建议在**桌面端**使用以获得最佳追踪体验
- 移动端可通过 HTTPS 访问，但需注意性能表现

## License

[MIT](LICENSE) © 2025 GestureTree Contributors

## 致谢

- [MediaPipe](https://developers.google.com/mediapipe) — 强大的跨平台机器学习解决方案
- [Three.js](https://threejs.org/) — 易用的 3D JavaScript 库

## 作者

ChenQiyue

---

最后更新：2026-08-06
