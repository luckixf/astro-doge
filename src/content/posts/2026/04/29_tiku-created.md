---
title: 个人题库网站的搭建
description: codex编程流程&项目介绍&个人感触
date: 2026-04-29T17:11:11+08:00
slug: tiku-created
draft: false
---



- - -

前两天刷到了 [dogxi 的前端面试题网站 iFace](https://face.dogxi.me/)，觉得很有用，就想着 **fork** 过来，修改部署为自己的建工考试刷题网站。也想尝试自己修改完成一个[web项目](https://tiku.luckixf.top)~~（虽说99.96%的编程工作都是codex完成的，但我们俩真厉害）~~

### 原项目改造思路

原项目 iFace 本身是一个很适合改造的底子，**功能齐全**：`刷题、答案、解析，还有相适应的AI辅助功能，个性化的错题集设置`，同时界面排版整洁，ui交互舒适。

整体思路不是“从零写一个网站”，而是保留原项目比较好的骨架，再把内容层、题库层和刷题逻辑换成建工考试场景。~~（好吧，其实是我不想搞得太复杂，看到合适的就直接拿来改改用了）~~

大体思路可以拆分成几步:

1. 首先，我们需要将原项目进行重构，把里面前端面试有关的习题及AI有关话术全部清除 ~~（桀桀桀，感觉自己好邪恶）~~ 
2. 然后从某第三方题库软件下载相应习题，将 pdf 题库文件解析成 json，并把题目图片单独提取为 webp 静态资源
3. 最后将题目导入，对原网站经行一些适配性调整`重新设计题库结构，支持单选题、多选题、解答题；优化目录浏览、搜索、章节展开。`


---

### 技术栈

改造后项目为纯前端单页应用，核心技术栈如下：


>  核心框架 `React 19 + TypeScript 5.9`
>
>构建工具` Vite 7`
>
>样式方案 `Tailwind CSS 4`
>
>本地存储 `IndexedDB`_（存储用户刷题进度、错题、收藏、自定义题目等数据）_
>
>内容渲染 `react-markdown`_（渲染 Markdown 格式的题目解析）_
>
>体验增强 `vite-plugin-pwa`_（实现 PWA 缓存、离线访问、桌面端添加能力）_

内置题库以静态 JSON 文件存放于`public/questions/construction/`目录，与用户本地数据完全隔离，不依赖后端服务。
 
---

### 题库 JSON 结构设计

为适配前端统一渲染，设计了标准化的题库结构，兼容单选、多选、案例解答等全题型，核心结构如下：

```json
{
  "id": "highway-chapters-1-1-747c70e1-q001",
  "module": "公路工程管理与实务 · 1.1 路基施工",
  "difficulty": 1,
  "type": "single",
  "question": "题干内容……",
  "options": [
    { "key": "A", "text": "选项 A" },
    { "key": "B", "text": "选项 B" }
  ],
  "correctAnswers": ["A"],
  "answer": "## 正确答案\nA\n\n## 解析\n这里是解析内容……",
  "tags": ["公路工程管理与实务", "单选题"],
  "source": "章节精讲",
  "questionImages": [
    "/question-assets/construction/highway/chapters/example/q001-1.webp"
  ]
}
```

其中`type`字段区分题型：`single`（单选题）、`multiple`（多选题）、`essay`（解答 / 案例题），答案与解析统一使用 Markdown 格式存储，保证复杂内容的渲染效果。

目前内置题库规模：

- 总题量 7213 道，其中单选题 5038 道、多选题 2106 道、解答题 69 道
- 覆盖 3 个科目：公路工程管理与实务（1405 道）、建设工程法规及相关知识（3254 道）、建设工程施工管理（2554 道）
- 配套题库 JSON 文件 166 个，题目配图 2484 张



---

### PDF 转 JSON 离线生产管线

为避免浏览器端 PDF 解析的性能问题与兼容性风险，采用**离线构建**的方案处理题库源文件，仅将最终生成的结构化数据提交至仓库。

IFace项目树

```text
iFace/
├─ api/
│  └─ auth.js                         # Vercel Serverless Function，用于 GitHub OAuth 登录和云同步授权
│
├─ docs/
│  └─ screenshots/                    # 项目截图，用于 README 或博客展示
│
├─ local-pdf-sources/                 # 本地 PDF 原题目录，已被 .gitignore 忽略，不应提交到 GitHub
│  ├─ highway/                        # 公路工程管理与实务 PDF
│  ├─ regulations/                    # 建设工程法规及相关知识 PDF
│  ├─ management/                     # 建设工程施工管理 PDF
│  ├─ past-exams/                     # 历年真题 PDF
│  └─ mock-exams/                     # 模拟试卷 PDF
│
├─ public/
│  ├─ icons/                          # PWA 图标，不同尺寸用于浏览器、桌面快捷方式等
│  ├─ questions/
│  │  └─ construction/                # 内置建工题库 JSON
│  │     ├─ catalog.json              # 轻量题库目录，题库页优先加载它，避免一次性加载全部答案
│  │     ├─ highway/                  # 公路工程管理与实务题库
│  │     ├─ regulations/              # 建设工程法规及相关知识题库
│  │     └─ management/               # 建设工程施工管理题库
│  │
│  └─ question-assets/
│     └─ construction/                # 题目配图资源，由 PDF 管线裁剪并转成 WebP
│        ├─ highway/                  # 公路工程相关题图
│        ├─ regulations/              # 法规相关题图
│        └─ management/               # 施工管理相关题图
│
├─ reports/
│  ├─ construction_bank_audit.json    # 题库审计报告，记录扫描结果和问题明细
│  ├─ construction_bank_audit.md      # 人类可读版审计报告
│  └─ construction_bank_repairs.json  # 保守修复脚本生成的修复记录
│
├─ scripts/
│  ├─ audit_construction_json.py      # 审计生成后的 JSON 题库，不重跑 PDF 构建
│  ├─ repair_construction_json.py     # 对高置信度错误进行保守修复
│  ├─ md_to_json.py                   # Markdown 题目转 JSON 的辅助脚本
│  └─ release.sh                      # 发布辅助脚本
│
├─ src/
│  ├─ assets/                         # 前端静态资源
│  │
│  ├─ components/
│  │  ├─ layout/
│  │  │  ├─ Navbar.tsx                # 顶部导航栏，“建工刷题助手”等品牌入口
│  │  │  └─ SettingsDrawer.tsx        # 设置抽屉，包含 AI 配置、刷题偏好、导入导出、云同步等
│  │  │
│  │  └─ ui/
│  │     ├─ AIPanel.tsx               # 题目 AI 辅助面板，用于解析、追问、点评
│  │     ├─ MarkdownRenderer.tsx      # Markdown 渲染组件，用于显示答案、解析、解答题分段
│  │     └─ index.tsx                 # 通用 UI 组件导出
│  │
│  ├─ data/
│  │  └─ schema.ts                    # 题目 JSON 数据结构定义，包括 single、multiple、essay
│  │
│  ├─ generated/
│  │  └─ constructionBank.ts          # PDF 管线自动生成的题库索引，不建议手动修改
│  │
│  ├─ hooks/
│  │  └─ useQuestions.ts              # 题库加载 Hook，连接页面和题库加载逻辑
│  │
│  ├─ lib/
│  │  ├─ db.ts                        # IndexedDB 本地数据库封装
│  │  ├─ gistSync.ts                  # GitHub Gist 云同步逻辑
│  │  ├─ mdImport.ts                  # Markdown 导入解析逻辑
│  │  ├─ questionLoader.ts            # 内置题库和自定义题库加载核心，支持 catalog 按需加载
│  │  └─ questionVisibility.ts        # 题库分类隐藏、显示偏好等过滤逻辑
│  │
│  ├─ pages/
│  │  ├─ Dashboard.tsx                # 首页仪表盘，展示总体进度、今日目标、快捷入口
│  │  ├─ ImportPage.tsx               # 导入页面，支持 JSON / Markdown 自定义题库导入
│  │  ├─ Practice.tsx                 # 刷题页，承载先答后看、边看边记、背题模式
│  │  ├─ QuestionDetail.tsx           # 题目详情页，展示完整题干、选项、图片、答案和解析
│  │  ├─ QuestionList.tsx             # 题库目录页，按科目、章节、真题、模拟题组织 7000+ 题
│  │  └─ WeakPoints.tsx               # 错题和薄弱点页面
│  │
│  ├─ store/
│  │  ├─ useAIStore.ts                # AI 配置、模型参数、系统提示词等状态
│  │  ├─ useAuthStore.ts              # GitHub 登录、OAuth 状态、云同步身份信息
│  │  └─ useStudyStore.ts             # 学习进度、做题记录、错题、收藏、笔记等状态
│  │
│  ├─ types/
│  │  └─ index.ts                     # 项目公共类型定义
│  │
│  ├─ App.tsx                         # 应用主入口，配置页面路由和整体布局
│  ├─ main.tsx                        # React 挂载入口
│  └─ index.css                       # 全局样式和 Tailwind 引入
│
├─ tools/
│  └─ pdf-pipeline/
│     ├─ build_construction_bank.py   # PDF 转 JSON 核心脚本，负责文本提取、清洗、分题、裁图
│     └─ README.md                    # PDF 管线说明，强调原始 PDF 不应提交
│
├─ .env.example                       # 环境变量示例，主要用于 GitHub OAuth 和云同步
├─ .gitignore                         # Git 忽略规则，包含 local-pdf-sources、PDF、构建产物等
├─ biome.json                         # Biome 代码格式化和检查配置
├─ index.html                         # Vite HTML 入口
├─ package.json                       # 项目依赖和 npm scripts
├─ README.md                          # 项目说明文档
├─ tsconfig.json                      # TypeScript 总配置
├─ vercel.json                        # Vercel 部署配置
└─ vite.config.ts                     # Vite 配置，包含 React、Tailwind、PWA、缓存策略等
```

>`src/` 是前端源码核心目录。页面、组件、状态管理、题库加载、IndexedDB、本地导入、云同步等主要逻辑都在这里。简单说，用户实际看到和操作的网站，大部分都由 `src/` 里的代码控制。

>`public/questions/construction/` 是生成后的内置题库目录。这里存放的是 JSON，不是 PDF。为了避免一次加载 7000 多道完整题目，项目额外生成了一个 `catalog.json` 作为轻量目录，题库页先读它，进入题目详情或练习时再加载完整题目。

>`public/question-assets/construction/` 是题目图片目录。PDF 中的题图会被脚本裁剪出来，转成 WebP，再按科目和来源分类保存。前端不会一打开网站就加载全部图片，而是在题目真正显示时按需加载。

>`tools/pdf-pipeline/` 是离线 PDF 处理工具。它不是普通用户使用网站所必需的部分，而是维护者用来把本地 PDF 转换成 JSON 题库的工具。这个目录负责处理水印、OCR 错字、题目分段、答案解析、图片裁剪等复杂问题。

>`local-pdf-sources/` 是本地 PDF 原文件目录。这个目录不应该提交到 GitHub，因为 PDF 原题可能涉及版权问题。项目通过 `.gitignore` 忽略它，公开仓库里尽量只保留代码、JSON 结构化题库和必要图片资源。

>`scripts/` 是题库后处理和质量检查目录。PDF 转 JSON 之后，还需要用审计脚本检查题目 ID、答案选项、图片路径、OCR 残留、水印残留、目录一致性等问题。修复脚本只做比较保守的高置信度修正，避免把题目越修越离谱。

>`api/auth.js` 是云同步相关的服务端函数。因为 GitHub OAuth 不能只靠纯前端安全完成，所以这里用 Vercel Serverless Function 处理登录跳转和回调。没有配置它时，网站依然可以本地刷题，只是 GitHub 云同步不可用。

>`reports/` 是脚本生成的报告目录。比如题库审计结果会输出到这里，方便查看本次扫描了多少题、有没有重复 ID、有没有结构错误或高风险样本。这个目录更像是“质检记录”。


### 脚本核心能力

执行`python tools/pdf-pipeline/build_construction_bank.py`即可完成全流程处理，核心步骤：

1. 基于 PyMuPDF 提取 PDF 页面文本与图片区域
2. 通过正则识别题号、选项、答案、解析，自动区分题型
3. 清理水印、页眉页脚、广告片段与重复文本，修正高频 OCR 错字
4. 裁剪题干关联图片，转换为 WebP 格式并生成静态资源路径
5. 按科目、章节拆分 JSON 文件，同步生成轻量目录索引`catalog.json`


---

### 内容清洗与特殊题型处理

PDF 解析的核心痛点是内容污染与结构错乱，针对性做了多层清洗方案：

1. 固定水印、广告片段过滤，高相似度段落去重
2. 工程术语、里程桩号、数字符号的保守修复
3. 题干、答案、解析的边界二次校验，避免跨题内容粘连

针对无固定选项的案例解答题，采用「规则初切分 + 脚本审计异常 + AI 辅助校验」的方案，先按「问题 - 参考答案 - 解析」标记做初步分段，再通过脚本识别高风险异常样本，最终人工抽查校验，保证内容准确性。


---

### 前端加载优化

针对 7000 + 道题的全量加载卡顿问题，做了分层加载与全链路优化：

1. **双层加载架构**：首屏仅加载轻量`catalog.json`（仅含题目摘要、分类、题型，不含答案与图片），用户进入对应章节 / 题目时，再按需加载完整 JSON 文件
2. 搜索与筛选采用防抖处理，避免输入过程中频繁全量重算
3. 列表分页渲染、章节按需渲染，减少首屏 DOM 节点数量
4. PWA 缓存对 JSON 题库与图片资源采用差异化策略，避免更新被缓存拦截
5. 题库列表仅展示题目摘要，不暴露答案与解析，同时减少渲染开销


### Web 功能补完

改造过程中还发现，原项目有些设置项更像是预留入口，并没有完全变成真实行为。于是这次顺手把刷题偏好补齐了。（呜呜 X﹏X，又被作者坑了）

现在主要有三种模式：

`先答后看 先作答，再展开参考答案，适合正常练习。 边看边记 直接显示答案，同时在答案卡片内写笔记，适合复习整理。 背题模式 默认展开答案，不要求输入作答，用户只标记“没记住 / 有印象 / 已背会”。`

此外还处理了几个实际使用中很烦的小问题：

1. 移动端卡片排版不完整，重新调整了题库模块布局。
2. 题库列表里部分题目答案暴露，改为列表只展示摘要。
3. React 出现重复 key 警告，重新处理题目 ID 和列表 key。
4. 设置里的隐藏分类、刷题偏好真正影响题库页、练习页和弱项页。
5. 导出数据时，如果是移动端，会提示用户注意浏览器或 QQ 内置浏览器的下载路径。
6. 顶部导航、AI 提示词、页面文案全部从“前端面试”换成“建工刷题”。

### GitHub 云同步踩坑

原项目的 GitHub OAuth 方案在 Vercel 部署时存在环境变量读取问题 ~~哈哈 原作者也没在README里讲，就甩锅给她好了~~ ，重构为服务端处理方案：

- 服务端函数`/api/auth`读取环境变量，处理 OAuth 跳转与鉴权
- 需配置环境变量：`GITHUB_CLIENT_ID`、`GITHUB_CLIENT_SECRET`、`APP_ORIGIN`
- OAuth App 回调地址填写`https://tiku.luckixf.top/api/auth`
- 云同步仅同步用户学习记录与自定义题目，不重复上传内置题库内容



### 关于 Codex 的一点感受

第一次使用codex构建完整的项目，也是第一次真正意义上使用智能体agent。此前我所借助的ai模型，只能帮我生成一个两个代码文件，大多都是自己复制粘贴，然后再看着报错反复地问，慢慢修改……（想起来假期的时候让豆包帮我写的noenbot课表插件）。

之前我对于agent的理解仅仅停留在这是一个很厉害的工具的层面，却不知道怎么使用，怎么配置。

回忆自己第一次接触编程，接触代码的时候。是在大一的11月，借助助学贷款买到了人生的第一台电脑。开始对编程这块内容感到兴趣。

其实一开始我的志愿就是计算机专业的，无奈最后差两分被调剂到了土木 X﹏X悲。后来自己也没有把握住转专业的机会，现在再去似乎也有些来不及了吧。

真正开始实践编程写代码是在大一的下学期了。当时关注到了QQ聊天机器人的搭建，想到自己那是笨手笨脚的安装python，从别人分享的网盘里下载机器人程序（机器人的协议段还是早已停止维护的gocqhttp）。

再然后自己在b站找教程，才终于了解到了github平台，搭建了第一个意义上的QQ机器人[zhenxun_bot](https://github.com/zhenxun-org/zhenxun_bot)。

后来倒腾的就比较少了 ~~其实是玩我的世界去了（不是）~~ 。对于计算机，程序也慢慢有了自己的理解。

直到去年大二上学期，我想这是自己真正开始迈入编程领域，此前主要都是按着别人的教程文档来部署程序。

因为自己的QQ机器人经常被风控，而且部署在本地每天的维护也很麻烦。便在阿里云买了台云服务器，将自己的项目转移了上去。为此自学了不久的linux命令行操作。

此后经验逐渐成熟，也沉寂了一段时间。直到最近一周了，相继注册了自己的域名，开始部署自己的网站。

这两天借助Codex完成了当下项目的修改，重构。完成了人生第一个web编辑。

回顾自己这两年对于电脑应用的探索实践，程序的开发实践，虽然起步相较晚，但我相信自己不久之后就能不借助AI自己完成web开发。

过去无可挽回，未来可以改变。

>ᯠ  _  ̫  _ ̥ ᯄ ੭


---

当下项目完成的还可以，至于后面会不会继续折腾，大概就看我是在学习，还是又忍不住开始优化学习工具了 ~~骗你的，优化不了一点~~ 。