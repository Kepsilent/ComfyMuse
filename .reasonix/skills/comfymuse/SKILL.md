---
name: comfymuse
description: 跨平台 AI 生图提示词生成引擎。当用户说"生成一张图/帮我画/文生图/我想生成一张图片"等任何想要生成图片的自然语言时，自动激活本 Skill。
runAs: inline
---

# ComfyMuse

> 跨平台 AI 生图提示词生成引擎 —— 你说人话，它出提示词
> Version: 2.1.0

**触发方式**：除了 `/ComfyMuse` 命令，用户说以下自然语言也会激活本 Skill：
- "我想生成一张图"
- "帮我画一个..."
- "文生图..."
- "生成一张照片"
- 其他包含"生成图/画图/文生图"等关键词的句子

**语言规则**：本文档所有分类数据（风格/类型/场景/细则/维度）均为英文。当 lang=zh-CN 时，AI 需将英文数据实时翻译为中文呈现给用户。语言包仅保留 UI 按钮文字。

**数据读取优先级**：
1. 本文件 `八、分类数据库` — 稳定版，永不动
2. `NSFW_Patch.md` — 可选，存在时加载 NSFW 追加内容
3. `Addons-SFW.md` — 运行时自动创建，存放 SFW 自定义细则/维度
4. `Addons-NSFW.md` — 运行时自动创建，存放 NSFW 自定义细则/维度（存在 NSFW_Patch.md 时才会产生）

**Community contribution**: Addons-SFW.md can be submitted to the main repo at https://github.com/wanjie/ComfyMuse. Addons-NSFW.md can be submitted to HuggingFace (link in NSFW_Patch.md). Valuable additions get merged into the stable release.

---

**ASK 交互方式**：

每轮选项规则：
1. AI 优先从 SKILL.md 题库（分类数据库/维度参考/NSFW 补丁）中选取匹配的选项 → token 成本低
2. 题库没有的 → AI 自主生成，并保存回对应分类数据库（按场景/类型归属），越用越聪明
3. 每轮选项下方标注：⚠️ 或在上方「输入你的答案」框自行补充

存回规则：SFW 新增内容 → 写入 Addons-SFW.md；NSFW 新增内容 → 写入 Addons-NSFW.md。文件不存在时由 AI 自动创建。

---

## 一、完整流程总览

> * 仅 Ship: SKILL.md + NSFW_Patch.md，其余文件（Config/、Versions/、Obsidian 同步等）均为运行时生成。*

```
/ComfyMuse <描述>          ← 自然语言描述想生成的画面
/ComfyMuse config          ← 调出配置界面
        │
        ▼
┌──────────────────────────────────────────────────────────────┐
│ ① 检查配置：.reasonix/skills/comfymuse/user_profile.md  │
│  ├─ 存在 → 读偏好 → 进 ③分类选择                          │
│  ├─ 不存在 → 走 ②配置流程                                │
│  └─ obsidian_path 为空 → 偏好用默认值                     │
└──────────────────────────────────────────────────────────────┘
        │
        ▼
┌──────────────────────────────────────────────────────────────┐
│ ② 配置流程（仅无配置时或 /config 时触发）                    │
│  ASK第1轮: 语言/平台/记忆上限/推荐条数                       │
│  → 平台=ComfyUI → ASK第2轮: 路径                           │
│  → 路径不存在 → ASK第3轮: [我自己找] [让AI帮我找]           │
│  → 全量扫描 + 适配检查 + 版本推荐 → 写入 user_profile.md      │
│  → ASK: [Start] [Exit]                                        │
└──────────────────────────────────────────────────────────────┘
        │
        ▼
┌──────────────────────────────────────────────────────────────┐
┌──────────────────────────────────────────────────────────────┐
│ ③ 分类与追问（统一算法，含分类选择+方案匹配+维度追问）      │
│  第1轮: 模式（纯净版跳过） 第2-n轮: 风格/类型/场景/细则/维度  │
│  交互由顶部ASK规则管理，细则加载NSFW_Patch.md追加项          │
└──────────────────────────────────────────────────────────────┘
│  第1步：有记忆→[①用上次] [②改上次] [③不重复]              │
│         无记忆→AI生成4-5个推荐 [multi-select]                │
│  第2步：[✅无补充] [✏️需要补充→引导在上方输入框填写]        │
│         补充内容→AI识别→分SFW/NSFW→存 Addons-SFW.md 或 Addons-NSFW.md（AI自动创建）  │
└──────────────────────────────────────────────────────────────┘
        │
        ▼
┌──────────────────────────────────────────────────────────────┐
│ ④ 确认文本（标签概览+完整画面描述）                          │
│  [✅没问题] [✏️哪里有问题→多选分类+其他→逐个修改→重新确认]  │
└──────────────────────────────────────────────────────────────┘
        │ 点✅没问题
        ▼
┌──────────────────────────────────────────────────────────────┐
│ ⑤ 查平台（同步） │
│  ├─ platform=ComfyUI     → 走 A 分支                         │
│  └─ platform=通用文生图  → 走 B 分支                         │
└──────────────────────────────────────────────────────────────┘
        │
   ┌────┴────┐
   ▼         ▼
 ┌─────┐  ┌─────────────────────────┐
 │A分支│  │B分支：通用文生图          │
 │     │  │ 按语言转提示词·输出到屏幕  │
 │增量扫描│                            │
 │硬件定档│                            │
 │更新检查│                            │
 │模型选择│                            │

 │API提交│                            │
 │→轮询等待│                            │
 │版本写入│                            │
 │ASK满意? ←MUST NOT SKIP              │
 │（多选要改的→逐个问→改→重跑）        │
 │←不满意改 ←（循环直到满意）          │
 │→命名+备份+删临时文件                │
 │→同步Obsidian/ComfyMuse/             │
 └─────┘  └─────────────────────────┘
```

---

## 二、检查记忆

用户输入 `/ComfyMuse <描述>` 后，第一步：

```
检查 .reasonix/skills/comfymuse/user_profile.md 是否存在
├─ 存在 → 读配置（lang/platform/models）
│         → 取 obsidian_path → 读 Core.md → 取偏好
│         → 偏好+描述词 → AI推荐
│         → 直接进 ③分类选择
├─ 不存在 → 走 ②配置流程
           → 配置完成弹 ASK：[Start] [Exit]
           → 选 Start ➔ 进入 ③分类选择
           → 选 Exit ➔ 结束流程

/ComfyMuse config → 强制进 ②配置流程

If obsidian_path is empty or Core.md does not exist:
  → Default preferences (realistic, SFW), flow continues normally

### Update check (runs every ComfyMuse startup when platform=ComfyUI)

```
Check version:
  Read installed version from user_profile.md
  curl GitHub API for latest release -> parse tag_name
  Compare:
    same -> "ComfyUI is up to date"
    newer -> "New version vX.X.X available. Update?"
              [Update] [Skip]
    API fail -> "Cannot check. Visit: https://github.com/comfyanonymous/ComfyUI/releases"

If user confirms update:
  First update (convert to Git):
    1. Check if Git is installed
       Yes -> proceed
       No  -> download Git from official source -> install
    2. cd {comfyui_path}
    3. git init
    4. git remote add origin https://github.com/comfyanonymous/ComfyUI.git
    5. git fetch origin                    # get all version info
    6. git checkout {current_version_tag}   # pin current files first
       (This ensures user models/plugins/config are not affected)
    7. git pull                             # pull latest changes only
    8. Done -> "Migrated to Git. Future updates will use git pull"

  Subsequent updates:
    1. cd {comfyui_path}
    2. git pull
    3. Done

  Note: User's models/ stay untouched because:
    - models/ custom_nodes/ user/ are not in the official repo
    - Git only updates files it knows about
    - Everything else is preserved automatically
```

---

## 配置流程

当检测到无配置文件或用户输入 `/ComfyMuse config` 时执行。

### 第1轮 ASK（4问，一次性问完）

```
Q1: 语言 / Language
    下方固定选项：[中文] [English]      ◄── 唯一双语

Q2: 平台
    下方固定选项：[ComfyUI] [通用文生图]

Q3: 记忆上限 → 或在上方「输入你的答案」框自行设定数值
    下方固定选项：[100] [500] [999] [不限]

Q4: 推荐条数 → 或在上方「输入你的答案」框自行设定数值
    下方固定选项：[3] [5] [10] [不限]

Q5: Obsidian 知识库存放路径 → 引导用户在上方「输入你的答案」框填写路径
    下方固定选项：[🔍 AI 帮我找]
    点「AI帮我找」→ 检测系统盘符（C:, D:, E: 等）
    ↓
    弹 ASK 让用户点选盘符：
    [C:\] [D:\] [E:\] [F:\]  ← 不在列表中？直接在上方「输入你的答案」框填写
    ↓
    选定盘符后，AI 在该盘搜索含 .obsidian/ 隐藏目录的文件夹（Obsidian 仓库特征）
    ↓
    结果处理：
    ├─ 找到 1 个 → 直接选用该路径
    │
    ├─ 找到多个 → 列出让用户点选确认
    │             [D:\我的知识库] [D:\Obsidian\笔记库] [都不是，重新搜索]
    │
    └─ 没找到 → "在{D}盘没有找到 Obsidian 仓库，请尝试以下方法："
               "① 换个盘符重新搜索"
               "② 自己去 Obsidian 客户端点开仓库设置，复制路径"
               "③ 跳过此设置，后续手动配置"
```

### 第2轮 ASK（仅平台=ComfyUI时触发）

```
Q1: ComfyUI 安装在哪个路径？ → 引导用户点击上方「输入你的答案」框填写
    下方单一选项：[🔍 AI 帮我找]
    点「AI帮我找」→ 检测盘符 → 弹用户点选盘符 → AI在该盘搜索 ComfyUI
    提示：ComfyUI 根目录应有 models/ 目录
```

### 第3步：路径校验

```
用户输入路径（来自上方输入框）
    ↓
路径是否存在？
├─ 存在 → 执行全量扫描
│
└─ 不存在 → "你提供的路径不存在"
            → 引导用户重新在上方输入框填写正确路径
```

### 第4步：AI 帮我找

```
用户点「🔍 AI 帮我找」
    ↓
检测系统有哪些盘符（C:, D:, E: 等）
    ↓
弹 ASK 让用户点选盘符：
[C:\] [D:\] [E:\] [F:\]  ← 不在列表中？直接在上方「输入你的答案」框填写
    ↓
选定盘符后，AI 在该盘根目录搜索：
  执行 ls 命令搜索名称含 ComfyUI 的文件夹
  没有直接命中 → 搜索含有 models/checkpoints/ 结构的目录
    ↓
结果处理：
├─ 找到 1 个 → 直接选用该路径
│
├─ 找到多个 → 列出让用户点选确认
│             [D:\ComfyUI\] [D:\AI\ComfyUI\] [都不是，重新搜索]
│
└─ 没找到 → "在{D}盘没有找到 ComfyUI 文件夹，请尝试以下方法："
           "① 换个盘符重新搜索"
           "② 自己去电脑里找到 ComfyUI 根目录（它应该有 models/ 文件夹）"
           "③ 从 ComfyUI 官网 https://github.com/comfyanonymous/ComfyUI 重新安装"
```

### Step 5: Full Scan (AI Agent Auto-Execute)

```
Full scan executed once path is valid. AI uses tools in this order:

1. Hardware detection (runs first; stops if hardware is insufficient):
   Run nvidia-smi to detect GPU model and VRAM
   Run wmic to detect system RAM
   Classify by VRAM:

   | Tier | VRAM | Max Steps | Max Resolution | Compatible Models |
   |:---:|:----:|:-------:|:---------:|---------|
   | Ultra | >=16GB | 60 | 2048x2048 | All |
   | High | 8-12GB | 40 | 1024x1216 | SDXL/Pony/ZImage |
   | Medium | 4-6GB | 30 | 1024x1024 | SDXL Lite/SD1.5 |
   | Low | <4GB | 25 | 768x768 | SD1.5 |

   Save result to user_profile.md -> ##Hardware:
   ##Hardware
   gpu: NVIDIA GeForce RTX 4090   # GPU model
   vram: 24 GB                    # VRAM
   vram_tier: Ultra               # Ultra>=16 / High 8-12 / Medium 4-6 / Low<4
   ram: 32 GB                     # System RAM
   Re-detect on every A branch startup.

2. Check ComfyUI version:
   Verify main.py / comfy/ directory exists in ComfyUI root
   Check version number if available

3. Scan directories + model analysis:
   Use ls/glob to scan {comfyui_path}/models/ subdirs:
   checkpoints/  vae/  clip/  text_encoders/  loras/  controlnet/  unet/  diffusion_models/

   For each .safetensors in checkpoints/, extract full tags:

   Node tags (define workflow wiring):
     node_checkpoint: CheckpointLoaderSimple
     node_clip: CLIPLoader + node_clip_name + node_clip_type (analyze text_encoders/)
     node_vae: VAELoader + node_vae_name (analyze vae/)
     node_latent: EmptyLatentImage / EmptyFlux2LatentImage / EmptySD3LatentImage + resolution
     node_encode: CLIPTextEncode (SDXL uses dual encoders)
     node_prompt_format: standard / score_prefix (Pony)

   Param tags (inferred):
     params: {steps, cfg, sampler, scheduler} inferred from file size and arch

   Content tags (from filename analysis):
     style: realistic / anime / semi-realistic / text-render
     safety: sfw / nsfw
     use_case: portrait / cosplay / general / text

   Dependency check:
     Use glob to verify VAE/CLIP files exist in vae/ and text_encoders/
     Write file paths into corresponding tags

4. Fitness check + smart recommendations:
   Based on hardware tier + scanned model list:

   a) Hardware detection:
      (1) GPU + VRAM (one command):
          Run: wmic path win32_VideoController get name,adapterram
          Parse output:
            NVIDIA found        -> NVIDIA GPU, read VRAM from adapterram (bytes -> GB)
            AMD / Radeon found  -> AMD GPU, read VRAM
            Intel / UHD / Iris  -> Intel integrated GPU
                                 -> if VRAM empty, mark as "shared memory"
            None of above       -> no dedicated GPU

      (2) System RAM:
          Run: wmic memorychip get capacity
          Sum all sticks, convert bytes -> GB

   b) Version recommendation (full coverage table):
       GPU Type | VRAM / RAM     | Recommended Version          | Notes
       ---------|----------------|-----------------------------|------
       NVIDIA   | >= 10GB        | Standard CUDA               | All models, no limits
       NVIDIA   | 8 - 10GB       | Standard CUDA               | SDXL lower resolution
       NVIDIA   | 6 - 8GB        | Standard CUDA               | Prefer SD1.5, or use Portable
       NVIDIA   | 4 - 6GB        | ComfyUI Portable            | SD1.5 primarily
       NVIDIA   | < 4GB          | ComfyUI Portable            | Recommend hardware upgrade
       AMD      | any            | DirectML edition            | -
       Intel    | any            | OpenVINO edition            | Including shared memory
       No GPU   | RAM >= 32GB    | CPU edition                 | Works but slow
       No GPU   | RAM < 32GB     | Cannot run locally          | Suggest cloud service

   c) First-time install (zip download + extract + no model config):
      (1) AI downloads with speed monitoring:
          curl -L -o {user_chosen_path}/ComfyUI.7z {url}
          Monitor every 10%:
            speed > 5MB/s -> continue
            speed < 1MB/s -> switch to mirror
              mirror also slow -> give direct link to user
            stuck > 30s -> connection lost -> give direct link
          Check initial speed: if < 1MB/s from start -> switch mirror immediately

      (2) User chooses destination:
          ASK: "Where to install?"
          [Desktop] [D:/AI/] [Custom path]
          AI creates directory

      (3) 7z extraction:
          Check if 7z is installed:
            Yes -> 7z x {file} -o{destination}
            No  -> download 7z from official sources:
                   GitHub: https://github.com/ip7z/7zip/releases
                   Official: https://www.7-zip.org/download.html
                   Mirror (community): https://mirror.nju.edu.cn/7zip/
                -> AI picks one -> install 7z -> extract

      (4) AI completes install:
          Extract to destination -> organize files
          Report: "Installation complete. ComfyUI installed at: {path}"
          Save path to user_profile.md

      (5) If AI download fails at any point:
          Give user direct download link, let them download manually
          User says "I saved it to D:/downloads/"
          AI takes over: check 7z -> extract -> organize -> done

   d) Model fitness check
      Compare each model against hardware tier limits:
        Ultra -> all models OK
        High -> flag >12GB models as may exceed VRAM
        Medium -> flag SDXL/Pony/ZImage as consider SD1.5 alternative
        Low -> flag >4GB models as not recommended
      Show flags alongside model list

   e) Supplementary recommendations
      Based on tier + existing models + missing dependencies:
      e.g. SDXL VAE: sdxl_vae.safetensors (missing)

5. Results display:
   Scan complete! Found {n} models:
   - darkBeast -> ZImageTurbo (12GB) [realistic,NSFW]
   ...

6. Reports:
   Fitness report: 2 models flagged for your tier (Medium)
   Suggestions: install sdxl_vae.safetensors, consider SD1.5 models
```
~ Model params are not read from preset tables. AI infers from file size/filename/hardware tier during scan and writes to user_profile.md. SKILL.md does not hardcode any model config data. ~
~ Config flow must write at this step, otherwise next startup treats as unconfigured. ~

### 第6步：配置完成

```
弹 ASK：[Config done! Start?]
        [Start] [Exit]
```

## 三、分类与追问（统一算法）

所有分类选择、有效方案匹配、维度追问统一为一套流程：
AI 每轮根据以下信息生成选项：
- **用户输入的描述词**（提取关键词）
- **用户长期记忆中的偏好**
- **上轮已选内容**
- **{obsidian_path}/ComfyMuse/Cases/ 匹配结果**（全部/部分/单维匹配，取 top N）

第1轮：模式（仅NSFW_Patch存在时显示，纯净版跳过）
第2-n轮：风格/类型/场景/细则/维度，顺序和轮次由 AI 自主决定
每轮交互方式由顶部全局 ASK 规则统一管理
所有维度问完 → 自动进入确认文本

### 可用维度参考（AI 自主选用，非强制）

| 维度 Key | 说明（英文打底，AI 翻译） | 示例问题 |
|----------|--------------------------|---------|
| Subject | 主体 | 单人 / 双人 / 多人？性别？ |
| Build | 体型 | 苗条 / 丰满 / 肌肉 / 娇小？ |
| Clothing | 服装 | 校服 / 便服 / 制服 / 泳装？ |
| Hair | 发型 | 黑长直 / 马尾 / 双马尾 / 短发？ |
| Expression | 表情 | 微笑 / 害羞 / 惊讶 / 冷漠？ |
| Pose | 姿势 | 站立 / 坐姿 / 弯腰 / 躺卧？ |
| Angle | 视角 | 正面 / 侧面 / 背后 / 俯视？ |
| Lighting | 光照 | 窗光 / 顶光 / 逆光 / 霓虹灯？ |
| Environment | 环境 | 室内 / 室外 / 具体陈设？ |
| Atmosphere | 氛围 | 温馨 / 暧昧 / 紧张 / 梦幻？ |
| Quality | 画质 | 4K / 胶片 / 手机拍摄 / 高清？ |



### 用户自定义细则（长期记忆）

AI 识别用户补充的内容可作为新细则时：
  SFW → 写入 Addons-SFW.md；NSFW → 写入 Addons-NSFW.md。文件不存在时 AI 自动创建。
## 四、确认文本

所有维度追问完成后，组装确认文本。

### 展示内容（两部分）

```
┌─────────────────────────────────────────────────────────┐
│ 📋 确认你的画面设定                                      │
│                                                          │
│ 【标签概览】                                              │
│ 模式：成人内容  风格：写实  类型：校园                     │
│ 场景：教室      细则：★露出+★挑逗                         │
│ 主体：单人/女   服装：校服/真空  发型：黑长直              │
│ 表情：羞涩脸红  姿势：弯腰      视角：背后中景            │
│ 景深：背景虚化  光照：窗光侧光  环境：黑板课桌            │
│ 氛围：暧昧紧张  画质：4K                                  │
│                                                          │
│ 【完整画面描述】                                          │
│ 一个穿着日式校服的17岁女高中生独自在教室里，弯着腰捡     │
│ 起掉落的粉笔，白色衬衫尾部从裙子里扯出，黑色长发垂落     │
│ 在脸侧，脸颊泛红。午后阳光从窗户斜射进来，在课桌上       │
│ 投下金色的光影，教室后排散落着几本书。                     │
│                                                          │
│ [✅ 没问题] [✏️ 哪里有问题]                               │
└─────────────────────────────────────────────────────────┘
```

### 点"哪里有问题"

```
用户点「哪里有问题」→ 弹 ASK（multiSelect: true），展示本轮已选的分类：
[模式] [风格] [类型] [场景] [细则] [服饰] [发型] [姿势] [视角] [其他]

流程：
1. 用户多选要修改的分类 + 可勾【其他】
2. AI 按勾选顺序逐个重走对应选择流程
3. 【其他】最后处理：
   ├─ 用户输入自然语言 → AI 解析并直接修改方案
   ├─ AI 发现描述不清晰 → 弹 ASK 选项供用户点选澄清
   └─ 用户提出新分类/细则 → AI 自动新增到对应题库
4. 全部修改完毕 → 重新展示确认 → 循环直到用户点「没问题」
```

### 点"没问题"

进入下一步：保存记忆 + 查平台（同步执行）。

---

## 五、保存记忆 + 查平台

### 查平台前的收尾

```
不写入本地 history。所有有效方案仅保存到 Obsidian Cases。
```

### 查平台

```
读 user_profile.md 中的 platform 字段
├─ platform = ComfyUI      → 走 A 分支（本地模型）
└─ platform = 通用文生图    → 走 B 分支（云端）
```

---

## 六、A 分支 — ComfyUI

### 6.1 增量扫描

```
扫 models/checkpoints/ 目录 → 获取当前磁盘模型列表
读 user_profile.md 中已记录的模型列表
双向对比：

├─ 磁盘有、记录无 → 新增
│   → 判断架构 → 检测VAE/CLIP → 打标签 → 加入列表
│
├─ 磁盘有、记录有 → 不变，保留原有配置
│
└─ 记录有、磁盘无 → 已删除
    → 从 user_profile.md 中移除该模型

更新 user_profile.md 模型列表为最新磁盘状态。
注：增量扫描在每次 A 分支启动时执行。
```

### 6.2 模型选择

```
模型选择逻辑：
AI 根据用户当前的风格标签从 user_profile.md 模型列表中过滤
NSFW_Patch 存在时，允许 nsfw 标签的模型通过过滤
过滤后由 AI 自行决定展示数量和交互方式（遵循顶部统一 ASK 规则）
```

### 6.3 查该模型完整配置 + 画质档位覆盖

从 user_profile.md 中读取该模型的：
  arch: 架构
  requires_vae: VAE路径
  requires_clip: CLIP路径
  params: 推荐参数（steps/cfg/sampler/scheduler/size）

从 ##Hardware 读取当前档位：
  根据档位覆盖 params 的上限：
    Ultra 档 → 不限制，使用模型默认参数
    High 档 → steps ≤ 40, size ≤ 1024×1216
    Medium 档 → steps ≤ 30, size ≤ 1024×1024
    Low 档 → steps ≤ 25, size ≤ 768×768

### 6.4 输出：生成图片 + 保存方案

模型配置读取完毕，进入输出环节。



在当前项目目录创建 `Workbench_v{N}.md`，记录本次生成的提示词和参数。

```markdown
# Workbench

日期：20260617
模式：成人内容
风格：写实
类型：校园
场景：教室
细则：★露出 + ★挑逗

## 正向提示词
1girl, chinese, high school girl, school uniform, short skirt...

## 反向提示词
（AI 根据正向提示词动态生成）

## 参数
模型：darkBeast_dbzit9DIMRclaw（ZImageTurbo）
Steps: 40, CFG: 1, Sampler: euler, Scheduler: simple
Size: 1024x1024
```

#### 第2步：组装 API 格式并提交到 ComfyUI

AI 根据模型架构直接拼装 API JSON（不依赖外部模板文件），替换正向提示词和反向提示词 → POST 到 `http://127.0.0.1:8188/prompt`。

**提示词组装规则**：分类/维度的标签（style/type/scene/detail 等）本身已是英文，直接使用。用户确认的完整画面描述（中文）翻译为英文后，与标签组合为最终正向提示词。反向提示词 AI 根据正向提示词内容动态生成，不预设模板。

**标签拼装规则**（无硬编码）：
每个模型在扫描时已打好完整标签（见全量扫描），标签包含所有拼装所需字段：
node_checkpoint / node_clip / node_clip_name / node_clip_type
node_vae / node_vae_name / node_latent / node_latent_size
node_encode / node_prompt_format / params (steps/cfg/sampler/scheduler)
style / safety / use_case

AI 读取标签直接组装 JSON，不需要对照表或硬编码规则。
curl -s -X POST http://127.0.0.1:8188/prompt \
  -H "Content-Type: application/json" \
  -d '{"prompt": { "4": {"class_type": "CheckpointLoaderSimple", "inputs": {"ckpt_name": "darkBeast_dbzit9DIMRclaw.safetensors"}}, ... }}'
```

返回示例：`{"prompt_id": "29580262-38f6-..."}`

#### 第3步：智能轮询（按步数估算等待时间）

```
AI 从 user_profile.md 读取该模型的 params.steps：
  估算：steps × 1.2秒 = 预计总时长
  等待：预计总时长 × 70% → 首次查询
  首次查询 → 未完成 → 每 2 秒查询一次
  超时：max(预计总时长 × 2, 120秒) → 提示超时
```

从返回值取文件名：`outputs["9"]["images"][0]["filename"]`

#### 第4步：拷贝图片 + 自动打开

从 `{comfyui_path}/output/{filename}` 拷贝到版本目录，命名为 `Preview_v{N}.png`。
打开方式：执行 `open_preview.bat "{版本目录}/Preview_v{N}.png"`（脚本位置见下文，弹出前台）
告知用户：图片已生成！（按用户语言翻译）

**open_preview.bat 说明**：
脚本需创建在 skill 目录下：`.reasonix/skills/comfymuse/open_preview.bat`
首次使用此命令前，AI 用 write_file 创建该文件，内容为：
```
@echo off
start "" "%*"
```
如果文件已存在，直接调用，不再重复创建。

#### 第5步
#### 第6步：确认【MUST NOT SKIP — 每次生成完必须弹】

```
ASK：[✅ 满意，保存方案] [✏️ 不满意，修改方案]（multiSelect: false, 单选）

⚠️ 此 ASK 每次生成完必须弹出，不可省略、不可跳过。

- 选不满意 → 弹 ASK（multiSelect: true）：
  [姿势] [画质] [人物细节] [场景] [其他]
  允许多选，一次勾选所有要改的地方
         → AI 识别选中的问题 → 逐个询问具体改法
         → 更新工作台版本文件 → 重新 API 提交 → 生成新版本
         → ⚠️ 再次弹本确认 ASK（循环，直到用户选满意）

- 选没问题 → 进入保存

循环示例：
  生成 v1 → ASK → 不满意 → 改 → 生成 v2 → ASK → 不满意 → 改
  → 生成 v3 → ASK → 满意 ✓ → 保存

引导规则：
  每轮 ASK 后 → AI 说"请点击选项，或直接在对话框告诉我你想怎么改"
  ASK 选项无法覆盖的需求 → 引导用户在对话框自然描述
  AI 收到描述 → 解析 → 更新工作台版本文件 → 重新提交 API → 再次弹 ASK
```

#### 第7步：保存到知识库

**自动命名规则**（按用户语言）：
  lang=zh-CN → `[日期]_[场景中文]_[细则中文]_[风格中文]`
  lang=en    → `[date]_[scene]_[detail]_[style]`

例如：`20260617_classroom_exposure_realistic`

```
① 最终版本文件已在版本目录中：Workbench_v{N}.md + Preview_v{N}.png
②  cp 到 Obsidian（文件名按用户语言决定：zh-CN用中文命名，en用英文命名）：
   cp Workbench_v{N}.md → {obsidian_path}/ComfyMuse/Cases/{命名}.md
   cp Preview_v{N}.png → {obsidian_path}/ComfyMuse/Images/{命名}.png

③ 清理ComfyUI output/ 临时图：
   rm {comfyui_path}/output/ComfyMuse_*.png

③ 跳过本地备份（有效方案仅存 Obsidian）

④ 如果 obsidian_path 已配置：
   写入 Obsidian 笔记（含图片嵌入）：
   → {obsidian_path}/ComfyMuse/Cases/{按用户语言命名}.md

   Obsidian 目录结构：
   {obsidian_path}/ComfyMuse/
   ├── Core.md
   ├── Cases/          ← .md 方案文件
   │   └── 20260621_漫展_露出_写实.md
   └── Images/         ← .png 图片文件
       └── 20260621_漫展_露出_写实.png

   内容格式（YAML frontmatter + 嵌入图片 + 提示词）：
   ---
   date: 2026-06-17
   场景: 教室
   细则: 露出, 偷窥
   风格: 写实
   模式: NSFW
   model: darkBeast_dbzit9DIMRclaw
   tags: [NSFW, realistic, exposure]
   ---
   
   ![[20260617_classroom_exposure_realistic.png]]
   
   ## 正向提示词
   1girl, chinese, 17 years old, schoolgirl...
   
   ## 反向提示词
   worst quality, bad anatomy...
   
   ## 参数
   Steps: 40, CFG: 1, 模型: DarkBeast

⑤ 如果 obsidian_path 已配置 → 更新 Obsidian Core：
   → {obsidian_path}/ComfyMuse/Core.md（没有则创建）

⑥ 同步 user_profile：
   将最新磁盘模型扫描结果写回 .reasonix/skills/comfymuse/user_profile.md

⑦ 告知用户：
   "方案已保存 → Obsidian ComfyMuse/Cases/"
   "图片 → Obsidian ComfyMuse/Images/"
```

<!-- 8.5 表已删除：模型参数全部通过扫描标签获取，不硬编码 -->

## 七、B 分支 — 通用文生图

```
直接按用户语言生成提示词：
  lang=zh-CN → 中文详细描述
  lang=en    → 英文详细描述

输出到屏幕（可复制）
结束
```

---

## 八、分类数据库（Category Database）

All data in English. AI translates to user's language at runtime.

### 8.1 风格 / Styles (9)

| Key | Label |
|-----|-------|
| realistic | Realistic |
| anime | Anime |
| semi-realistic | Semi-Realistic |
| 3d | 3D |
| oil-painting | Oil Painting |
| watercolor | Watercolor |
| sketch | Sketch |
| pixel | Pixel |
| cyberpunk | Cyberpunk |

### 8.2 类型 / Types (9)

| Key | Label |
|-----|-------|
| campus | Campus |
| daily-life | Daily Life |
| portrait | Portrait |
| fashion | Fashion |
| fantasy | Fantasy |
| artistic | Artistic |
| sports | Sports |
| cute | Cute |
| night | Night |

### 8.3 场景 / Scenes — Campus

| Key | Label | SFW Details |
|-----|-------|-------------|
| classroom | Classroom | listening, self-study, exam, recess, cleaning, writing on board, raising hand, passing notes, sleeping at desk |
| library | Library | reading, research, self-study, borrowing books, organizing shelves, searching, note-taking, using computer |
| playground | Playground | running, exercise, PE class, flag raising, sports day, basketball, sitting on bleachers, jump rope |
| dormitory | Dormitory | resting, reading, phone, chatting, making bed, snacks, homework, mirror |
| hallway | Hallway | walking, chatting, waiting, looking out window, cleaning, leaning on railing, looking back, carrying books |
| rooftop | Rooftop | overlooking, breeze, chatting, lunch, photos, arms out, sitting, sunset watching |
| school-gate | School Gate | waiting, after school, chatting, bicycle, umbrella, looking back, waving goodbye |

### 8.4 场景 / Scenes — Daily Life

| Key | Label | SFW Details |
|-----|-------|-------------|
| living-room | Living Room | watching TV, reading, resting, chatting, tea, phone, lying on sofa, petting, unboxing |
| bedroom | Bedroom | sleeping, resting, reading, phone, makeup, music, changing clothes, mirror, stretching |
| kitchen | Kitchen | cooking, washing dishes, tea, chopping, organizing, tasting, wiping, opening fridge |
| balcony | Balcony | hanging laundry, sunbathing, reading, watering plants, overlooking, leaning, folding clothes, blowing bubbles |
| cafe | Cafe | coffee, reading, chatting, laptop, waiting, chin resting, window watching, writing |
| street | Street | walking, shopping, waiting bus, bicycle, photos, looking back, crossing, carrying bags |
| park | Park | walking, picnic, photos, running, dog walking, bench sitting, swing, feeding pigeons, kite |

### 8.5 场景 / Scenes — Portrait

| Key | Label | SFW Details |
|-----|-------|-------------|
| studio | Studio | posing, outfit change, makeup, waiting, side smile, looking back, chin rest, hair fix, hands on hips, mirror |
| outdoor | Outdoor | walking, standing in flowers, looking back, sitting on grass, eyes closed, hair flip, squatting, running, spinning |
| indoor | Indoor | sofa sitting, wall leaning, mirror, reading, lying down, kneeling, cross-legged, side lying, hugging pillow |
| window-side | Window Side | looking out, leaning, sunbathing, coffee, opening window, chin rest rain watching, curtains, reaching for rain |

### 8.6 场景 / Scenes — Fashion

| Key | Label | SFW Details |
|-----|-------|-------------|
| street-snap | Street Snap | walking, looking back, wall leaning, crossing, hands on hips, sunglasses adjustment, hair flip, side glance |
| runway | Runway | catwalk, pose, turn, bow, hands on hips, hair flip, head up, blowing kiss |
| fitting-room | Fitting Room | trying clothes, mirror, adjusting, selfie, zipping, side view, comparing, exiting |
| fashion-studio | Fashion Studio | posing, styling change, touch-up, shooting, hair flip, silhouette, crouch, looking back |

### 8.7 场景 / Scenes — Fantasy

| Key | Label | SFW Details |
|-----|-------|-------------|
| castle | Castle | balcony standing, stair walking, throne sitting, overlooking, curtsy, candle lighting, railing look back, door opening |
| forest | Forest | walking, sitting on roots, flower picking, looking up at light, barefoot wading, tree leaning, pushing branches, chasing fireflies |
| sky | Sky | flying, sitting on clouds, floating, falling, wings spread, reaching for light, eyes closed, somersault |
| dungeon | Dungeon | exploring, torch holding, fighting, resting, wall sneaking, shield raising, spell casting, map reading |

### 8.8 场景 / Scenes — Artistic

| Key | Label | SFW Details |
|-----|-------|-------------|
| art-studio | Art Studio | painting, mixing colors, viewing, thinking, brush gesturing, stepping back, wiping sweat, washing brushes |
| gallery | Gallery | viewing, pausing, photos, discussing, pointing at artwork, looking up at large painting, bench sitting, guidebook |
| workshop | Workshop | creating, sculpting, designing, organizing, welding, sanding, measuring, safety goggles |

### 8.9 场景 / Scenes — Sports

| Key | Label | SFW Details |
|-----|-------|-------------|
| gym | Gym | basketball, warm-up, rest, watching game, dribbling, shooting, water break |
| swimming-pool | Swimming Pool | swimming, poolside rest, diving, warm-up, swim cap, pool edge, backstroke |
| dance-room | Dance Room | practice, stretching, mirror practice, rest, splits, spin, leg press, tiptoe |
| sports-field | Sports Field | running, exercise, stretching, soccer, jump rope, sit-ups, floor sitting |

### 8.10 场景 / Scenes — Cute

| Key | Label | SFW Details |
|-----|-------|-------------|
| cute-bedroom | Bedroom | hugging plushie, selfie, snacks, reading, lying on bed, peace sign, comics, headphones |
| garden | Garden | watering, chasing butterflies, swing, flower picking, squatting, smelling flowers, spinning, blowing dandelion |
| dessert-shop | Dessert Shop | eating dessert, photos, cake selection, chatting, spoon licking, feeding, window browsing, cream squeezing |
| amusement-park | Amusement Park | carousel, ice cream, photos, queue, headband, balloon, screaming, ferris wheel |

### 8.11 场景 / Scenes — Night

| Key | Label | SFW Details |
|-----|-------|-------------|
| neon-street | Neon Street | walking, photos, wall leaning, neon watching, clear umbrella, looking back, crowd passing, tying shoelaces |
| bar | Bar | drinking, chatting, music, dancing, glass swirling, forehead resting, karaoke, bar leaning |
| night-rooftop | Night Rooftop | night view, breeze, drinking, chatting, edge sitting, pointing at lights, cuddling, star gazing |
| overpass | Overpass | traffic watching, photos, breeze, waiting, leaning down, umbrella, railing leaning, waving |
## 九、错误处理

| 场景 | 处理方式 |
|------|---------|
| user_profile.md 不存在 | 视为首次使用，走配置流程 |
| user_profile.md 损坏或为空 | 视为首次使用，走配置流程 |
| 用户输入与任何分类不匹配 | 不做推荐，直接展示全部选项 |
| AI 推荐结果不在可用选项内 | 丢弃该推荐，用默认推荐顶替 |
| 历史记录匹配失败 | 跳过记忆匹配，维度追问从零生成 |
| 用户中途说"返回/重来/修改" | 回到上一步或对应维度轮次，保持已填内容 |
| 用户说"取消/算了/不做了" | 结束流程，不保存记忆 |
| ComfyUI 路径不存在 | 弹ASK：[自己找] [让AI帮我找] → 让AI找：检测盘符→用户选盘→AI搜索→列出路径确认 |
| AI搜索ComfyUI路径失败 | 告知未找到，提供：①换盘重搜 ②手动查找 ③官网下载链接 |
| 增量扫描无新增 | 直接进入模型选择，不告知用户 |
| 模型架构无法判断 | 默认按 SDXL 输出，告知用户"该模型架构未知，已按 SDXL 格式输出" |
| 工作流模板缺失 | 只输出提示词，告知用户"工作流模板缺失，已输出提示词文本" |
| ComfyUI API 无响应（连接失败） | 告知用户"ComfyUI 未启动，请打开 ComfyUI 后重试"；等用户输入"重试"再提交 |
| ComfyUI API 返回 error（节点错误） | 告知具体错误信息 → 尝试更换模型或回退到通用模板参数 |
| ComfyUI 生成超时（>120秒） | 告知"生成超时" → 建议降低分辨率或更换模型 |
| 生成图片文件名读取失败 | 告知用户"图片已生成但无法自动获取"，提示去 output/ 目录手动查看 |

---

## 十、语言包

> 所有 Key 的值均为英文。AI 按用户语言实时翻译。
> 唯一例外：`lang_select` 显示 `语言 / Language`（双语入口）

### 10.1 Language Pack (en)

| Key | Text |
|-----|------|
| lang_select | 语言 / Language |
| platform_select | Platform |
| platform_comfyui | ComfyUI |
| platform_universal | Universal |
| memory_limit | Memory limit |
| memory_custom | Custom |
| recommend_count | Recommend count |
| all_display | Unlimited |
| more | More |
| continue_generate | Start |
| exit | Exit |
| use_last | Use last |
| modify_last | Modify |
| dont_reuse | Fresh |
| no_supplement | Done |
| need_supplement | Add |
| confirm_title | Confirm |
| confirm_ok | OK |
| confirm_modify | Fix |
| model_random | Random |
| model_more | More |
| path_not_exist | Path not found |
| path_guide | ComfyUI root has models/ folder |
| find_myself | I will find |
| let_ai_find | AI search |
| config_complete | Config done! Start? |
| api_not_running | ComfyUI not running |
| api_timeout | Generation timed out |
| scan_complete | Scan complete |
| image_generated | Image ready |
| image_saved_to | Saved |
| workbench | Workbench |
| preview_image | Preview |
| versions_folder | Versions |
| rollback_help | Enter version to restore (e.g. v2) |
| obsidian_path | Obsidian vault path |
| core_config | Core |
| cases_folder | Cases |
| mode_sfw | Safe |
| mode_nsfw | Adult |


---
## 十一、文件操作指南

### 读取
- 分类数据：本文档第十章内置
- NSFW 补丁：检查 `NSFW_Patch.md` → 细则段+维度段
- 用户配置：`.reasonix/skills/comfymuse/user_profile.md`（纯配置）
- 偏好数据：`{obsidian_path}/Core.md`（YAML frontmatter + 案例链接）
- Cases：`{obsidian_path}/ComfyMuse/Cases/*.md`
- Versions：`Versions/v{版本号}_{时间戳}/`（退档用）

### 写入
- `.reasonix/skills/comfymuse/user_profile.md`：纯配置（语言/平台/路径/模型/硬件）【配置流程必须写入】
- `Workbench_v{N}.md` + `Preview_v{N}.png`：版本文件，每次生成直接写入版本目录
- `{命名用用户语言: Versions(英文) / 版本记录(中文)}/v{版本号}_{时间戳}/`：版本目录（含 md + png）
- `{obsidian_path}/ComfyMuse/Cases/[日期]_[场景]_[细则]_[风格].md`：用户确认满意后写入
- `{obsidian_path}/ComfyMuse/Images/[日期]_[场景]_[细则]_[风格].png`：最终图片同步到 Obsidian
- `{obsidian_path}/ComfyMuse/Core.md`：新建或更新Cases链接 + 偏好统计

### 不要操作
- 不要修改本文件自身
- 不要删除 `NSFW_Patch.md`
- 不要手动删除 `Versions(英文) / 版本记录(中文)` 目录下的归档文件

---

## 十二、Obsidian 文件结构

当 `obsidian_path` 已配置时，以下文件位于 Obsidian 仓库中：

```
{obsidian_path}/ComfyMuse/
├── Core.md              # 偏好 + 案例链接
├── Cases/               # 成功方案 .md
│   └── 20260621_漫展_露出_写实.md
└── Images/              # 成功方案 .png
    └── 20260621_漫展_露出_写实.png
```

**格式**：
```markdown
---
偏好风格: 写实
常用类型: 校园
常用模型: darkBeast_dbzit9DIMRclaw
updated: 2026-06-17
---

# Core

## Cases
- [[Cases/20260617_classroom_exposure_realistic]]
- [[Cases/20260618_dormitory_halfnude_anime]]

## 偏好
- 风格: 写实（最近偏好多）
- 类型: 校园
- 模型: DarkBeast (40 steps)
```

**读取方式**（AI 启动时）：
1. 读 frontmatter → 取 偏好风格/常用类型/常用模型
2. 读Cases链接列表 → 匹配当前场景

**更新规则**：
- 每次保存新案例 → 追加链接
- 统计最近 3 条案例 → 同类标签 ≥2 → 更新 frontmatter 偏好
- 如果 Core.md 不存在 → 新建

### Cases

每个成功案例存为一个文件 `{obsidian_path}/Cases/[日期]_[场景]_[细则]_[风格].md`。

**内容**：YAML frontmatter（标签+参数）+ 嵌入图片 + 正向/反向提示词 + 参数。

图片放在 Obsidian 根目录或 assets/ 下，文件名与案例名一致。
