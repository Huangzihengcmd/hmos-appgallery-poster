---
name: hmos-appgallery-poster
description: 为 HarmonyOS 应用制作应用市场宣传图/宣传页/上架海报/商店截图/详情页推广图（AppGallery promotional images）。Use when 用户提到 宣传图、宣传页、宣传海报、上架图、上架海报、商店截图、应用市场截图、应用市场详情页图、AppGallery 推广素材、应用介绍图、产品介绍图、生图、宣传背景图。
---

# 鸿蒙应用市场宣传图制作

为当前 HarmonyOS 工程生成一套可直接上传 AppGallery Connect 的宣传图（应用详情页顶部轮播的介绍截图/宣传大图）。

## 总原则

1. **背景插画禁止用代码画**。像人类设计师一样：写提示词 → 调免费生图模型 → 看图 → 不满意换 seed/改提示词重来（见第 3 步）。代码只用于合成（拼图、圆角、文字排版、缩放导出）。
2. **规范先联网核实，再动手**。本文件内嵌的基线规格可能过时，每次任务先执行第 1 步。
3. **默认产出"标准宣传图套装"**：手机端竖版 5 张（上限 8）+ 平板端横向 2-3 张 + PC 端 16:9 两张。用户另有要求时按用户要求。
4. 所有产物写入 `<项目根>/promotion/`，不要污染工程源码目录。
5. 下载素材、生图等临时文件放项目目录，不要写入系统盘的 Temp 目录。

## 工作流程

### 第 0 步：分析应用

读以下文件提炼信息（不要问用户，自己读代码）：

- `AppScope/app.json5`：bundleName、icon、label
- `entry/src/main/module.json5`：abilities、权限、设备类型（是否支持 tablet/pc）
- `AppScope/resources/base/element/string.json` 与 `resources/zh/`：应用名
- `entry/src/main/resources/base/profile/main_pages.json` + `pages/` 下各页面：功能清单
- `resources/base/media/` 下 icon：用取色思路定品牌主色（可直接肉眼读图判定主色调）

产出 `promotion/plan.md`，包含：应用名、一句话定位、3-5 个核心卖点（每张宣传图对应一个卖点）、目标人群、品牌主色 hex、风格选型。

### 第 1 步：联网核实规范（必做）

用 websearch / webfetch 搜索并核对最新要求，搜索词示例：

- `AppGallery 宣传图 尺寸 规格 应用信息`
- `华为应用市场 应用介绍图片 截图 要求`
- `developer.huawei.com 主题设计指导及规范`（HarmonyOS 设计规范）
- `AppGallery Connect 图片规格 上架`

以华为官方文档（developer.huawei.com / AppGallery Connect 帮助中心）为准。若官方页 JS 渲染抓不到，用上述关键词搜二手权威来源交叉验证。

**基线规格表（2026-11 核对版，仍需联网复核）**：

| 素材 | 尺寸/比例 | 格式与限制 |
| --- | --- | --- |
| 手机端宣传图（竖版） | 1080×1920 / 1080×2340 / 1080×2400（9:16 ~ 9:19.5） | JPG/PNG，单张 ≤2MB，3-8 张 |
| 平板端宣传图（横版） | 2560×1440（16:9）或 3:2 | 同上 |
| PC 端宣传图 | 1920×1080（16:9） | 同上 |
| 应用市场图标 | 216×216（AGC 后台）/ 512×512 | PNG，圆角由后台裁剪 |
| HarmonyOS 桌面图标前景/背景 | 1024×1024 | PNG，透明，禁止自带圆角边距阴影 |

**审核红线**：图中不得出现 iOS / iPhone / 苹果商店 / 竞品应用商店字样或设备外观；不得出现与实际功能不符的虚假宣传；不要出现国旗、政治敏感元素；截图中不要暴露隐私信息。

将核实结果记入 `promotion/plan.md`。

### 第 2 步：模拟器截图（产品介绍图素材）

1. `devecocli devices` 检查设备；无设备则启动模拟器。
2. 若工程未构建：`devecocli build`；然后 `devecocli run --skip-build` 安装启动。
3. 逐页截图（**默认用 hdc 截屏，不要用 verify_ui**，UI 校验工具仅在用户明确要求 UI 验证时使用）：

```powershell
hdc shell "snapshot_display -f /data/local/tmp/shot_1.jpeg"
hdc file recv /data/local/tmp/shot_1.jpeg .\promotion\shots\shot_1.jpeg
```

4. 按卖点规划截图：每个核心功能页至少一张；导航到对应页面后再截下一张。
5. 截图若非 1080 宽，在 plan.md 记录实际分辨率，合成时等比缩放。

### 第 3 步：AI 生图——模拟人类绘画（多风格背景）

**方式 A（默认，免费无 Key）：Pollinations 直连**

URL 形式（prompt 需 URL 编码、用英文）：

```
https://image.pollinations.ai/prompt/{encoded_prompt}?width=1024&height=2048&seed={n}
```

**2026-11 实测注意**：

- **不要带 `nologo=true`**，该参数已转为付费（返回 402 Payment Required）；`model=flux` 也不要带，用默认模型即可。
- 免费通道限流，首次请求常见 503（提示 Retry after 10s），**等 12 秒重试，最多 5 次一般能成**。

```powershell
$prompt = [uri]::EscapeDataString("soft gradient fluid abstract background, deep blue and cyan, smooth flowing shapes, no text, no watermark")
foreach ($i in 1..5) {
  Start-Sleep -Seconds 12
  try {
    Invoke-WebRequest -Uri "https://image.pollinations.ai/prompt/$prompt`?width=1024&height=2048&seed=42" -OutFile .\promotion\bg\style1_s42.png -TimeoutSec 120
    break
  } catch { }
}
```

人类式迭代：每种风格生成 2-3 个 seed，逐张查看（读图判断质量），不满意就改提示词或换 seed 重来，不要一次生成几十张。

**方式 B（降级，用户要求"像人一样操作网页"时）：浏览器自动化**

若 deveco.json 配置了浏览器类 MCP（如 `@playwright/mcp`），则打开免费生图站（pollinations.ai 等），模拟人类：输入提示词 → 等待出图 → 下载到 `promotion/bg/`。没有该 MCP 则直接用方式 A。

**方式 C：环境里已配置的生图 MCP/连接器**（如可画连接器自带的 AI 生图），存在则优先于 A。

**风格库**（按应用类型在 plan.md 选 3-5 种，每种对应 1-2 张宣传图）：

| 风格 | 适用 | 提示词关键词 |
| --- | --- | --- |
| 渐变流体 | 工具/科技/AI | soft gradient fluid abstract, smooth flowing shapes |
| 极简几何 | 效率/商务 | minimalist flat design, geometric shapes, clean, pastel |
| 3D 卡通渲染 | 儿童/教育/休闲 | cute 3d cartoon render, soft lighting, playful |
| 自然风光摄影 | 生活/健康/旅行 | natural landscape photography, golden hour, bokeh |
| 国潮插画 | 内容/文化 | chinese traditional illustration, modern guochao style |
| 暗黑霓虹 | 游戏/电竞 | dark neon glow, cyberpunk, dramatic lighting |
| 柔和弥散光 | 社交/情感 | soft diffuse light, dreamy pastel, warm |

**提示词铁律**：必须包含 `no text, no watermark` —— 画里的文字交给第 4 步排版，生图模型的文字必乱。配色围绕品牌主色写。

### 第 4 步：合成产品介绍图（勾画鸿蒙手机轮廓 + 文案）

用 Python + Pillow 合成（无则 `pip install pillow`）。每张竖版图结构：

```
┌─────────────────┐  1080×2400
│  应用名（大标题）   │  ← 标题区：应用名 + 一句话卖点，主色/白字
│  一句话卖点        │
│ ╭─────────────╮ │
│ │ 鸿蒙手机轮廓   │ │  ← 机身：品牌主色渐变圆角矩形（r≈140px），
│ │ ┌─────────┐ │ │     描边高光，内嵌第 2 步截图（同样圆角裁切）
│ │ │ 模拟器截图│ │ │
│ │ └─────────┘ │ │
│ ╰─────────────╯ │
│  icon + 应用名    │  ← 底部品牌区
└─────────────────┘
```

最小可用骨架（按需改参数，不要照抄硬套）：

```python
from PIL import Image, ImageDraw, ImageFilter, ImageFont

W, H = 1080, 2400
bg = Image.open("bg/style1_s42.png").convert("RGB")
bg = bg.resize((W, H))  # 背景铺满，必要时先 crop 再 resize 保持比例

img = bg.copy()
draw = ImageDraw.Draw(img)

# 鸿蒙手机轮廓
frame_w, frame_h = 760, 1560
fx, fy = (W - frame_w) // 2, 560
body = Image.new("RGB", (frame_w, frame_h), (20, 20, 25))
mask = Image.new("L", (frame_w, frame_h), 0)
ImageDraw.Draw(mask).rounded_rectangle([0, 0, frame_w, frame_h], radius=110, fill=255)
img.paste(body, (fx, fy), mask)

shot = Image.open("shots/shot_1.jpeg").convert("RGB")
shot = shot.resize((frame_w - 24, int((frame_w - 24) * shot.height / shot.width)))
if shot.height > frame_h - 24:
    shot = shot.crop((0, 0, shot.width, frame_h - 24))
smask = Image.new("L", shot.size, 0)
ImageDraw.Draw(smask).rounded_rectangle([0, 0, shot.width, shot.height], radius=98, fill=255)
img.paste(shot, (fx + 12, fy + 12), smask)
draw.rounded_rectangle([fx, fy, fx + frame_w, fy + frame_h], radius=110, outline=(255, 255, 255), width=4)

# 文案（中文字体：Windows 用 msyh.ttc，装了 HarmonyOS Sans 则优先）
font_big = ImageFont.truetype("C:/Windows/Fonts/msyh.ttc", 88)
font_sub = ImageFont.truetype("C:/Windows/Fonts/msyh.ttc", 46)
draw.text((W // 2, 300), "应用名", font=font_big, fill=(255, 255, 255), anchor="mm")
draw.text((W // 2, 410), "一句话卖点", font=font_sub, fill=(230, 230, 230), anchor="mm")

img.save("final/phone/poster_1.png", optimize=True)
```

要求：

- 平板/PC 版改为横构图：截图居左或居右，文案区对侧，分辨率按第 1 步核实结果。
- 导出后逐张检查：单张 ≤2MB（超了转 JPG quality=88 或降采样）、文字不压主体、圆角自然。
- 文案从 plan.md 的卖点提炼，短句化（≤12 字/行），禁止虚假功能描述。

### 第 5 步：可选增强——网页绘图工具（可画连接器等）

可画（Canva）自带大量内置图形资产：现成模板（App 推广/手机海报类）、形状、插画贴纸、图标、字体排版。AI 借助可画连接器可以直接套用这些内置图形精修，**不必从零绘制**。

1. 检查项目 `deveco.json` / 全局配置的 `mcp` 段是否有可画（canva）、Figma、MasterGo、Pixso 等设计类连接器。
2. **有可画**：搜索其内置模板（关键词 "app promotion" / "手机海报"）→ 把第 3 步 AI 背景与第 2 步截图替换进模板占位区 → 用内置图形微调排版与文案 → 导出回 `promotion/final/`。利用的是可画的现成设计资产，速度快且专业。
3. **无**：跳过本步，主流程（免费生图 + 合成）已足够；交付时告知用户可配置可画连接器获得模板精修能力。

**定位**：本步是锦上添花的精修增强。宣传图的主产出始终以第 3 步免费生图（Pollinations）+ 第 4 步代码合成为准，可画仅当连接器存在且用户需要更精致的模板化排版时使用。

### 第 6 步：自检与交付

对 `promotion/final/` 每张图核对：

- [ ] 分辨率 = 第 1 步核实的规格（尺寸、比例）
- [ ] 格式 JPG/PNG，单张 ≤2MB
- [ ] 手机端 3-8 张，每张对应一个卖点
- [ ] 无 iOS/苹果/竞品字样，无水印，无乱码文字
- [ ] 中文文案无错别字，与应用实际功能一致

目录结构：

```
promotion/
  plan.md          # 卖点、规格核实记录、风格选型
  shots/           # 模拟器原始截图
  bg/              # AI 生成背景（多风格、多 seed）
  final/phone/     # 手机端成品
  final/tablet/    # 平板端成品（若适用）
  final/pc/        # PC 端成品（若适用）
  manifest.md      # 交付清单：文件名、尺寸、大小、对应卖点
```

交付时向用户报告：成品路径、每张图的尺寸与对应卖点、以及上架 AppGallery Connect 时应上传的位置（应用信息 → 介绍图片/宣传图）。
