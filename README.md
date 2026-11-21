# hmos-appgallery-poster

给 [DevEco Code](https://developer.huawei.com/consumer/cn/deveco-code/)（DevEco Studio 的 AI 编码助手）写的 skill：一句话为 HarmonyOS 应用生成华为应用市场（AppGallery）详情页的宣传图 / 介绍截图 / 上架海报。

对着 DevEco Code 说"帮我做宣传图"，它会自己把整条流水线跑完。

## 它能干什么

1. **联网核对规范** —— 先搜一遍 AppGallery 最新的图片尺寸、比例、大小、数量限制和审核红线，确认无误才动手（内置的基线规格表只是兜底，每次任务都会联网复核）
2. **模拟器截图** —— 自动构建、安装、启动应用，用 hdc 逐页截屏收集产品截图素材
3. **AI 生图（模拟人类绘画）** —— 调 Pollinations 免费生图（无需 API Key）画背景：写提示词 → 看图 → 换 seed 迭代，内置 7 种风格库（渐变流体 / 极简几何 / 3D 卡通 / 自然风光 / 国潮插画 / 暗黑霓虹 / 柔和弥散光）按应用类型选型
4. **合成产品介绍图** —— 用 Python + Pillow 画出鸿蒙手机轮廓（圆角机身 + 高亮描边），嵌入模拟器截图、排好卖点文案，导出符合 AGC 规格的成品
5. **可选可画精修** —— 配置了 Canva（可画）等设计类 MCP 连接器时，可直接套用其内置模板精修，不必从零绘制

默认产出一套"标准宣传图"：手机端竖版 5 张（1080×1920 / 2340 / 2400）+ 平板端横向 + PC 端 16:9。

## 安装

**方式一：clone 到 DevEco Code 的全局 skills 目录**

```bash
# macOS / Linux
git clone https://github.com/Huangzihengcmd/hmos-appgallery-poster.git ~/.config/deveco/skills/hmos-appgallery-poster
```

```powershell
# Windows PowerShell
git clone https://github.com/Huangzihengcmd/hmos-appgallery-poster.git "$env:USERPROFILE\.config\deveco\skills\hmos-appgallery-poster"
```

**方式二：clone 到任意目录（建议非系统盘），在项目 `deveco.json` 里注册**

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "skills": {
    "paths": ["E:\\your\\path\\deveco-skills"]
  }
}
```

安装后重启 DevEco Code 生效。

## 使用

对话里提到"宣传图 / 宣传页 / 上架图 / 商店截图 / 应用市场截图 / 应用介绍图"等关键词即自动触发，或直接说：

> 帮我的鸿蒙应用做一套应用市场宣传图

产物输出在 `<项目根>/promotion/` 下：

```
promotion/
  plan.md          # 卖点、规格核实记录、风格选型
  shots/           # 模拟器原始截图
  bg/              # AI 生成背景（多风格、多 seed）
  final/phone/     # 手机端成品
  final/tablet/    # 平板端成品（若适用）
  final/pc/        # PC 端成品（若适用）
  manifest.md      # 交付清单
```

## 已知坑（2026-11 实测）

- **Pollinations 免费通道限流**：返回 503 就等 12 秒重试，最多 5 次一般能成
- **`nologo=true` 参数已收费**（返回 402 Payment Required），不要带，用默认模型即可
- 生图提示词务必带 `no text, no watermark`——画里的文字交给合成阶段排版，生图模型画的文字必乱

## License

[MIT](./LICENSE)
