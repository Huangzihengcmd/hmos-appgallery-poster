# 一句话生成鸿蒙应用市场宣传图 —— hmos-appgallery-poster 开源分享

## 项目地址

- **GitCode**：https://gitcode.com/gcw_4meGIUNQ/hmos-appgallery-poster
- **GitHub**：https://github.com/Huangzihengcmd/hmos-appgallery-poster

## 项目简介

**hmos-appgallery-poster** 是一个 DevEco Code（DevEco Studio 的 AI 编码助手）的 skill，专为 HarmonyOS 开发者打造。

只需在 DevEco Code 里说一句"帮我做宣传图"，它就会自动生成一整套可直接上传 AppGallery Connect 的应用市场宣传图/介绍截图/上架海报，无需打开 PS，无需设计师，无需任何费用。

## 解决什么问题

做过鸿蒙应用上架的朋友都懂：

- AGC 后台的图片规范（尺寸、比例、数量、大小限制）经常更新，稍不注意就被审核打回
- 一套完整的宣传图涉及手机竖版、平板横版、PC 端多个规格，手动截图再 P 图工作量巨大
- 个人开发者不会用设计工具，外包给设计师又要花不少钱

这个 skill 把整条流水线全自动化了。

## 工作流程

| 步骤 | 说明 |
| --- | --- |
| 1. 联网核对规范 | 先搜索 AppGallery 最新的图片规格和审核红线，确认无误才动手 |
| 2. 模拟器截图 | 自动构建、安装、启动应用，用 hdc 逐页截屏收集真实产品截图 |
| 3. AI 免费生图 | 调 Pollinations 生成背景（无需 API Key），写提示词 → 看图 → 换 seed 迭代，内置 7 种风格库 |
| 4. 合成介绍图 | 用 Python + Pillow 画出鸿蒙手机轮廓，嵌入截图、排好卖点文案，按 AGC 规格导出 |

**内置风格库**：渐变流体 / 极简几何 / 3D 卡通 / 自然风光 / 国潮插画 / 暗黑霓虹 / 柔和弥散光，按应用类型自动选型。

**默认产出**：手机竖版 5 张（1080×1920 / 2340 / 2400）+ 平板端横向 + PC 端 16:9，一句触发，一套到手。

## 安装方式

**方式一**：clone 到 DevEco Code 全局 skills 目录

```bash
# Windows
git clone https://gitcode.com/gcw_4meGIUNQ/hmos-appgallery-poster.git "%USERPROFILE%\.config\deveco\skills\hmos-appgallery-poster"
```

**方式二**：clone 到任意目录，在项目 `deveco.json` 里注册

```jsonc
{
  "skills": {
    "paths": ["你的skills目录路径"]
  }
}
```

安装后重启 DevEco Code 生效。

## 已知坑（都已写进文档）

1. **Pollinations 免费通道限流**：返回 503 就等 12 秒重试，最多 5 次基本能成
2. **`nologo=true` 参数已收费**（402），不要带，用默认模型即可
3. **生图提示词务必带 `no text, no watermark`**：画里的文字交给合成阶段排版，生图模型画的文字必乱

## 开源协议

MIT License，欢迎 Star、Fork、提 Issue。

如果你也在做鸿蒙应用开发，欢迎试试，觉得有用的话帮忙点个 Star ⭐
