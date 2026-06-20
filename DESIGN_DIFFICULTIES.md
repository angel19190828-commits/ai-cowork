# AI COWORK — 设计困难实录
*以对话形式还原，中文*

---

## 困难 1 · 背景大数字被裁切

**问题出现**

> **Angel:** Section 背景大数字 1 2 3 被砍掉了，2 and 3 的底部都没有了

背景数字 `font-size: 44vw`，`.section` 有 `overflow: hidden`，数字底部超出容器被裁掉。

**第一次尝试（错误）**

> **Claude:** 把 `.section` 改成 `overflow-x: hidden`，这样只裁左右，不裁上下。

> **Angel:** 你是傻子吗

结果：浏览器规范里 `overflow-x: hidden` 会强制把 `overflow-y` 变成 `auto`，每个 section 都出现了滚动条。立即 rewind。

**正确解法**

保持 `overflow: hidden` 不动，改用 `translateY` 把数字往上移，让底部不超出容器：

```css
.section-bg-num {
  transform: translateY(-51%);
}
```

**微调过程**

> **Angel:** 数字往下移了2%  
> **Angel:** 再下移4%  
> **Angel:** 再下移5%

最终落在 `-51%`，通过小步迭代找到视觉平衡点。

---

## 困难 2 · 卡片侧边竖排文字与横线交叠

**问题出现**

> **Angel:** 侧边竖排文字跟卡片横线交叠（附截图，红圈标注两处溢出）

`.side-label` 用 `position: absolute; top: 0; bottom: 0` 撑满整个 `.card-body`，但文字内容太长，在 `writing-mode: vertical-rl` 下溢出到 `.card-title` 的下边线和 `.card-foot` 的上边线之外。

**第一次尝试（被 rewind）**

> **Claude:** 给 `.side-label` 加 `background: rgba(244,242,239,0.92)` 和 `z-index: 2`，让标签盖住横线。

> **Angel:** 错错错 rewind，不要了

**第二次尝试（部分接受）**

把 `.side-label` 从 `.card` 直接子元素移到 `.card-body` 内部，让 `position: absolute` 相对 `.card-body` 定位，并给 `.card-body` 加 `overflow: hidden`：

```html
<!-- 之前 -->
<div class="card">
  <span class="side-label l">BUILT FOR BUILDERS</span>
  <div class="card-body"></div>
  <span class="side-label r">AI WORKSPACE INFRASTRUCTURE</span>
</div>

<!-- 之后 -->
<div class="card">
  <div class="card-body">
    <span class="side-label l">BUILT FOR BUILDERS</span>
    <span class="side-label r">AI WORKSPACE INFRASTRUCTURE</span>
  </div>
</div>
```

这次 Angel 接受了结构改动，但这个问题的最终视觉调整留给 Angel 自己手动微调。

---

## 困难 3 · 卡片底部信息区太挤

**问题出现**

> **Angel:** （截图附注）more space，底部这个 section 可以更高但整体 card size 不能变

`.card-foot` 内有 barcode、PURCHASED 标签、5条数据 + ingredients 文字，padding 只有 `0.8vw`，视觉上偏挤。

**解法**

从 `.card-body` 的 height 里借出 1.2vw 给 `.card-foot` 的 padding，保持整体卡片高度不变：

```css
/* 之前 */
.card-body { height: 8vw; }
.card-foot { padding: 0.8vw 1.4vw; }

/* 之后 */
.card-body { height: 6.8vw; }
.card-foot { padding: 1.4vw 1.4vw; }
```

但这次改动随后被 rewind，Angel 决定暂不处理。

---

## 困难 4 · Hero Headline 加载时粗体闪烁（FOUT）

**问题出现**

> **Angel:** 我发现每次打开网站，Hero Headline 总是先变成粗体然后再变回细体，我不喜欢这个，帮我修

原因：Google Fonts 用了 `display=swap`，浏览器先用系统字体（粗体）渲染，Inter 加载完成后再替换。

**第一次尝试（造成更大问题）**

> **Claude:** 在 CSS 给 `.word-inner` 加 `transform: translateY(105%)` 预藏文字，同时用 `document.fonts.ready` 等字体加载完再触发 GSAP 动画。

> **Angel:** 你瞎啊，啥也没有了（截图显示整个 Hero Headline 消失）

问题是 `document.fonts.ready` 的等待时间太长，字体还没加载完，headline 就一直是隐藏状态。

**第二次尝试（部分 rewind）**

移除 `document.fonts.ready` wrapper，只保留 CSS 预藏：

```css
.hero-headline .word-inner { 
  display: block; 
  transform: translateY(105%); 
}
```

> **Angel:** fuck just rewind I dont care the glitch anymore

最终选择不处理这个问题，恢复原状。

**结论：** 这个困难没有找到 Angel 满意的解法。视觉 bug 存在，但修复代价（影响 headline 可见性）比 bug 本身更难接受。

---

## 困难 5 · Receipt 行为：替换 vs 累加

**问题出现**

最初实现：每次进入新 section，receipt 内容被替换成新产品。

> **Angel:** 不对，应该像真实收据打印机一样，每访问一个产品就往下加一行，不要清除之前的。

**解法**

用 `Set` 记录已经打印过的项目，append 而不是 replace：

```javascript
const printedItems = new Set()

function printReceiptLine(i) {
  if (printedItems.has(i)) return  // 已打印过，跳过
  printedItems.add(i)
  if (printedItems.size === 1) recLinesEl.innerHTML = ''  // 清除 hint
  
  const line = document.createElement('div')
  line.innerHTML = `${cfg[i].item}<br><span>${cfg[i].ref}</span>`
  recLinesEl.appendChild(line)  // 累加，不替换
}
```

第一次实现就被接受，没有 rewind。

---

## 困难 6 · 摄像头 Demo 手部追踪

**问题出现**

> **Angel:** 摄像头 demo 我希望可以有互动，在 web camera 基础上加上 hand tracking，并且 hand 可以移动我设置的 PNG

**技术挑战**

- MediaPipe Hands 是 CDN 加载，需要正确的 `locateFile` 配置
- 视频显示是镜像的（`scaleX(-1)`），所以 landmark 坐标也要镜像：`x = (1 - landmark.x) * width`
- canvas 上需要同时叠加：视频画面 + 骨骼线 + PNG 图标
- 捏合手势检测：拇指尖（landmark 4）与食指尖（landmark 8）的距离 `< canvas宽 * 0.05`

**解法关键：**

```javascript
// 食指尖带动 PNG
const tip = handPts[8]
const px = (1 - tip.x) * w  // 镜像修正
const py = tip.y * h

// 捏合检测
const pinchDist = Math.hypot(mx(thumb) - px, my(thumb) - py)
const isPinching = pinchDist < w * 0.05
```

实现过程顺利，没有重大 rewind。

---

## 困难 7 · Camera 电影感运动

**两次尝试，两次 rewind**

**第一次：**
> **Angel:** 我想加一个 cinematic camera movement，从稍远处慢慢推进再回到正常位置

> **Claude:** 实现了 dolly in + settle back，起始位置 `(0.55, 6.4, 11.2)`

> **Angel:** rewind

**第二次：**
> **Angel:** 幅度太小，偏移感加2%

> **Claude:** 调整了起始点和推进幅度

> **Angel:** rewind

**结论：** Camera position 对 Angel 来说是非常敏感的区域。即使是他自己提出的需求，实际看到效果后也选择了 rewind。这个功能目前不在线。

---

## 困难 8 · work1.html 404 错误

**问题出现**

> **Angel:** http://localhost:3000/work1 这个还是连不到我的 HTML

原因：Python `http.server` 只能处理精确路径，`/work1` 不能自动匹配 `work1.html`。

**解法**

换用 `npx serve`，它支持扩展名省略：

```json
// .claude/launch.json
{
  "runtimeExecutable": "npx",
  "runtimeArgs": ["serve", "-p", "3000", "."]
}
```

`/work1` → 自动路由到 `work1.html`。

---

## 困难 9 · GitHub Pages 路径错误

**问题出现**

部署后，点击卡片跳转到 `/work1.html` 出现 404。

原因：GitHub Pages 把网站放在子路径 `/ai-cowork/` 下，`/work1.html` 会解析为根域名下的路径，不是子文件夹内。

**解法**

所有链接改为相对路径：

```html
<!-- 错误 -->
<div class="card-flip" data-work="/work1.html">

<!-- 正确 -->
<div class="card-flip" data-work="work1.html">
```

---

## 总结

| 困难 | 解法 | 结果 |
|------|------|------|
| 背景数字裁切 | translateY 微调替代 overflow 修改 | ✅ 解决 |
| 侧边标签溢出 | 移入 card-body + overflow:hidden | ✅ 结构改好，视觉待调 |
| 底部信息区太挤 | 从 card-body 借高度给 card-foot | ↩️ Rewind |
| Hero headline FOUT | — | ❌ 无满意解法，保留 bug |
| Receipt 替换 vs 累加 | Set + append 模式 | ✅ 一次成功 |
| 手部追踪 MediaPipe | 镜像修正 + 骨骼叠加 | ✅ 成功 |
| Camera 电影感运动 | — | ↩️ 两次 Rewind |
| work1.html 404 | npx serve 替代 Python | ✅ 解决 |
| GitHub Pages 路径 | 改为相对路径 | ✅ 解决 |

---

*整理自 AI COWORK ® STUDIO 完整构建对话 · 2026*
