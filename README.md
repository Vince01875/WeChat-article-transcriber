[README.md](https://github.com/user-attachments/files/31867910/README.md)
# wechat-article-transcriber · 公众号全文转 Word

**Turn WeChat Official Account articles — especially image-only "long screenshot" research reports — into AI-readable Markdown and shareable Word (.docx). Charts are transcribed back into tables. Pure Python standard library, zero pip dependencies.**

把微信公众号文章（尤其是正文全是长截图的**全图片版**研究报告/付费专栏）逐页转录为：① 任何 AI 都能读懂的完整文字稿（`.md`，图表还原为 Markdown 表格）；② 可在微信直接转发的 Word 文档（`.docx`，宋体/Times、三线表、1.5 倍行距）。

> 本项目以 [Coze（扣子）Agent Skill](https://www.coze.cn) 形式封装，把「抓取 → 逐页视觉转录 → 整合 → 转 Word → 渲染校验」做成标准流程。抓取脚本与转换器均为纯 Python 标准库，不依赖 Agent 平台，可独立运行；视觉转录环节需要接入具备图片理解能力的 AI（Agent 或多模态模型）。

## 解决什么痛点

公众号里大量研究报告、深度长文是**全图片版**：正文是一整页一整页的长截图，复制不出一个字。这类文章：

- 没法直接喂给 AI 做摘要/分析（AI 读不了几十张长图的关系）；
- 想转发全文给同事，只能一张张发图，体验差；
- 手动敲字不现实，图表数据还会丢。

本技能让 AI 像人一样**逐页读图**，把文字逐句转录、把图表数据还原成表格，再一键转成规范排版的 Word。

## 工作流程

```
mp.weixin.qq.com 链接
      │
      ▼
① 抓取（extract_wechat_article.py，纯标准库）
   → images/img_01..N.jpg + article.md + meta.json
      │
      ▼
② 判断是否"全图片版"（正文非图片文字 <200 字且图片 ≥3 张）
      │
      ▼
③ 逐页视觉转录：分批（每批5~6张）调用多模态 AI
   · 逐句忠实转录，不概括、不遗漏、不改写
   · 图表数据还原为 Markdown 表格（数值/年份/公司名不丢）
   · 分页标记数必须 == 图片总数（页数对齐红线）
      │
      ▼
④ 整合为自包含 AI 全文稿 .md
   （开头"给AI阅读者说明"+元信息，正文按章节，文末免责声明+数据速查）
      │
      ▼
⑤ md_to_docx.py 转 Word（纯标准库手写 OOXML，无依赖无网络）
   → 宋体/Times New Roman、黑体标题、改良三线表、1.5倍行距
      │
      ▼
⑥ 渲染校验：soffice 转 PDF → pdftoppm 渲染 → 视觉抽查
   （乱码/表格溢出/标题层级三项必查）
```

## 为什么 Word 转换不用 python-docx

`md_to_docx.py` 用 `zipfile` 直接手写 OOXML，**零第三方依赖、零网络请求**，在任何只有 Python 3 的沙箱/离线环境都能跑，输出稳定一致。

## 使用方法

```bash
# 1. 抓取文章
python3 scripts/extract_wechat_article.py "<mp.weixin.qq.com/s/链接>" -o "<任务目录>"

# 2. （全图片版）用任意多模态 AI 按 SKILL.md 第3步的转录要求逐批读图，整合为 全文稿.md

# 3. 转 Word
python3 scripts/md_to_docx.py "<全文稿.md>" "<输出.docx>"
```

产物两份，都要交付：

- **`<标题>_全文稿.docx`** —— 给人看 / 微信直接转发；
- **`<标题>_AI全文稿.md`** —— 给 AI 读，自包含纯文本，图表为 Markdown 表格。

> 抓取脚本与姊妹项目 [wechat-article-extractor](https://github.com/) 相同；若只需提取图片和文字、不需要转录与 Word，用那个更轻量。

## 目录结构

```
wechat-article-transcriber/
├── SKILL.md                          # 技能主文档：完整7步流程与转录质量红线
├── README.md                         # 本文件
├── LICENSE                           # MIT 许可证
└── scripts/
    ├── extract_wechat_article.py     # 文章抓取（纯标准库）
    └── md_to_docx.py                 # Markdown → Word 转换器（纯标准库手写 OOXML）
```

## 作为 Coze Skill 使用

1. 下载本仓库 ZIP；
2. 在扣子技能管理中「上传 / 创建技能」，导入 ZIP；
3. 对 Agent 发送 `mp.weixin.qq.com/s/...` 链接，说明要"全文转 Word / 文字稿"即可，Agent 会自动完成抓取、转录、转换与校验。

转录与校验环节依赖 Agent 的图片理解能力；SKILL.md 中的转录提示词、页数对齐规则、渲染校验命令同样适用于手动配合任意多模态模型执行。

## 注意事项

- 仅处理**公开可访问**的文章；不用于绕过付费、隐私或平台规则。
- 转录稿务必声明"转录自截图、个别数字可能有 OCR 误差，关键数据以原图为准"。
- 文章已删除/触发反爬时脚本以退出码 3/4 明确报错，按提示处理，勿盲目重试。
- 标准图文型文章（文字可复制）无需逐页转录，直接抓取整理转 Word 即可。

## 许可证

[MIT License](./LICENSE) © 2026 Vinceli
