# ComfyMuse

> 跨平台 AI 生图提示词生成引擎 —— 你说人话，它出提示词

---

## 🚀 一键式傻瓜安装

### 一句话搞定

支持：Reasonix · Claude Code · Cursor · Gemini CLI · GitHub Copilot · Windsurf · Cline · Aider …

直接复制下面这行话，丢给你的 AI Agent，就装好了：

```
强制重装这个项目，删除旧版本后重新安装最新版：https://github.com/Kepsilent/ComfyMuse
```

> ⚠️ 重要：AI 不得跳过安装流程，必须删除旧版本后重新下载覆盖。详见 `INSTALL.md`。

Agent 会自动读取 `AGENTS.md` → 按 `INSTALL.md` 步骤执行。当前会话立刻生效。

---

## 📖 项目介绍

ComfyMuse 是一个 AI 辅助的提示词生成引擎。你不需要学 prompt 语法，只需用自然语言描述想生成的画面（比如"一个女孩在教室"），它会引导你一步步完善细节，最终**直接调用 ComfyUI API 自动生图**，图片直接出现在你的对话目录。

**支持平台**：ComfyUI（API 直连） · 小云雀 · 即梦 · 谷歌生图 · OpenAI 生图 · 通用文生图

### 核心功能

| 功能 | 说明 |
|------|------|
| 🗣️ 自然语言输入 | 说人话就行，不用学 prompt 语法 |
| 🧭 智能分类引导 | AI 根据你的描述推荐风格/类型/场景/细则 |
| 🧠 弹性维度追问 | 你描述得越细，问得越少；越简单，问得越多 |
| 📝 工作台 | 修改提示词不丢失上下文 |
| 🎨 API 自动生图 | ComfyUI 平台一键提交 → 轮询等待 → 图片就绪 |
| 💾 Obsidian 知识库 | 满意的方案自动存入 Obsidian，越用越聪明 |
| 🏷️ 自动命名 | 图片按 日期_场景_细则_风格 自动归档 |
| 🔄 双语支持 | 中文/英文界面切换 |
| 🔞 NSFW 支持 | 完整版含成人内容分类（仅 HuggingFace 发布） |

### 怎么用

```
/ComfyMuse 一个女孩在教室里看书
```

然后跟着引导走：
1. 选模式（安全/成人）
2. 选风格（写实/动漫/油画...）
3. 选类型（校园/人像/幻想...）
4. 选场景（教室/图书馆/操场...）
5. 选细则（听课/自习/看书...）
6. AI 弹性追问细节（服装/发型/表情/姿势...）
7. 确认文本 → 自动提交 ComfyUI → 预览图.png 出现在文件面板
8. 满意 → 自动命名存入 Obsidian 知识库

每次生成完它会记住你的偏好，越用越顺手。

### 配置

```
/ComfyMuse config
```

设置语言、平台、ComfyUI 路径、Obsidian 路径、记忆数量等。

---

## 项目结构

```
ComfyMuse/
├── README.md              # 本文档
├── AGENTS.md              # AI Agent 发现入口
├── INSTALL.md             # AI 自动安装指令
├── CLAUDE.md              # Claude Code 路由指引
├── LICENSE                # MIT 协议
├── NSFW_Patch.md           # NSFW 细则补丁（仅完整版）
├── Config/                 # 配置文件（首次使用时自动生成）
├── skill.json              # Reasonix 注册文件
└── .reasonix/skills/comfymuse/
    └── SKILL.md            # ⭐ 技能本体（完整流程+分类+架构规则）
```

---

## 关于 NSFW 版本

本仓库为标准版（SFW only），不含成人内容。

**需要完整版（含 NSFW 支持）？**
→ 前往 HuggingFace 获取：[huggingface.co/wanjie647/ComfyMuse](https://huggingface.co/wanjie647/ComfyMuse)

完整版额外提供 NSFW 模式下的成人内容细则（露出/自慰等），并通过 NSFW_Patch.md 扩展分类系统。

---

## License

MIT
