# ComfyMuse — AI-Powered Prompt Engineering Skill

> Transform natural language descriptions into production-ready ComfyUI prompts & workflows.
> Full English base with AI real-time translation. `lang_select` is the only bilingual UI entry.

---

## 1. Invocation

```
/ComfyMuse <natural language description>
/ComfyMuse config
```

- `/ComfyMuse <description>` → starts the full prompt engineering pipeline
- `/ComfyMuse config` → force re-enter configuration flow
- Natural triggers (when no slash command system exists): "生成图", "画图", "生成照片", "文生图", "AI画图", "帮我画", "生成一张", "画一张", "生成个图", "我想生成", "帮我生成图"

---

## 2. Configuration Flow (First Run Only / `/ComfyMuse config`)

### 2.1 Check user_profile.md

**Path**: `Config/user_profile.md`

| Condition | Action |
|-----------|--------|
| Exists | Read lang/platform/comfyui_path/obsidian_path → read Obsidian/ComfyMuse/Core.md preferences → enter classification |
| Missing | Enter configuration flow below |

### 2.2 Language Decision (affects entire flow)

| lang | Behavior |
|------|----------|
| `zh-CN` | AI translates all UI text, questions, options to Chinese in real time. Obsidian naming uses Chinese. |
| `en` | AI uses English throughout. Obsidian naming uses English. |

### 2.3 Configuration Questions

#### ASK Round 1 (4 questions, one screen)

| # | Question | Options |
|---|----------|---------|
| Q1 | Language | [中文] [English] (`lang_select` bilingual display) |
| Q2 | Platform | [ComfyUI] [通用文生图 / Universal] |
| Q3 | Memory Limit | Input box / [100] [500] [999] [Unlimited] |
| Q4 | Recommendation Count | Input box / [3] [5] [10] [Unlimited] |
| Q5 | Obsidian Vault Path | Input box / [🔍 AI Find for Me] |

- **AI Find for Me** → show drive picker [C:\] [D:\] [E:\] [F:\] [Not listed → type manually] → scan for folders containing `.obsidian/` → single match = auto-select, multiple = user picks, none = try another drive / manual / skip

#### ASK Round 2 (ComfyUI only)

| Condition | Action |
|-----------|--------|
| platform=ComfyUI | Ask for ComfyUI installation path → manual input (validate existence) OR [🔍 AI Find for Me] |
| AI Find | Drive picker → scan for ComfyUI (look for main.py or ComfyUI directory) → single/multiple/none handling |

#### Step 5: Full Scan (AI Auto-Execute, user invisible)

```
① Hardware detection
   ├── Windows: wmic path win32_VideoController get name,adapterram
   ├── Memory: wmic memorychip get capacity
   └── Map to tier (see 2.4)
② Check ComfyUI version (main.py / comfy dir structure)
③ Scan models (checkpoints/) + VAEs + CLIP → tag each: node/param/content
④ Fitness check + version recommendation + model supplement (see 2.4)
⑤ Download with speed monitor → 7z (3 sources) → Git conversion on first update
   *Update check at startup: first time convert to Git, then git pull*
```

**7z Sources** (fallback chain):
1. GitHub Official: `https://github.com/ip7z/7zip/releases`
2. Official Site: `https://www.7-zip.org/download.html`
3. NJU Mirror: `https://mirror.nju.edu.cn/7zip/`

**Download Monitor Rules**:
- Speed < 1 MB/s → switch mirror
- Still slow → give user direct download link
- Stuck > 30s → declare dead link → give direct link

#### ASK Round 3: Completion

```
[Start] [Exit]
```

### 2.4 Hardware Detection + Version Recommendation

| GPU | VRAM | Recommended Version |
|-----|------|-------------------|
| NVIDIA | ≥ 10 GB | Standard CUDA |
| NVIDIA | 8–10 GB | Standard CUDA (lower resolution) |
| NVIDIA | 6–8 GB | Standard CUDA (SD1.5 recommended) |
| NVIDIA | 4–6 GB | ComfyUI Portable |
| NVIDIA | < 4 GB | ComfyUI Portable (upgrade recommended) |
| AMD | any | DirectML Edition |
| Intel | any | OpenVINO Edition |
| No GPU | RAM ≥ 32 GB | CPU Edition |
| No GPU | RAM < 32 GB | Cannot run locally |

**Tier mapping for API params**:

| Tier | VRAM Range | Max Steps | Max Resolution |
|------|-----------|-----------|----------------|
| Ultra | ≥ 16 GB | unlimited | 2048+ |
| High | 10–16 GB | ≤ 40 | 1536 |
| Medium | 6–10 GB | ≤ 30 | 1024 |
| Low | < 6 GB | ≤ 25 | 768 |

---

## 3. Classification &追问 Pipeline

### 3.1 Unified Algorithm

Each round loads: user description + long-term preferences + Cases/ match results + previous selections.

**Round 1**: Mode selection
- NSFW_Patch.md exists on disk → show [Safe] [Adult] (mode_sfw / mode_nsfw)
- NSFW_Patch.md absent → skip, default to SFW

**Rounds 2–n**: AI decides order and number of rounds from:
- Style → Type → Scene → Detail → Dimension
- Elastic rule: more user description → fewer rounds; simpler description → more rounds
- Each round: AI recommends options from classification database → user picks → "Anything else to add?" → if yes → guide to input box → AI extracts keywords → save to Addons-SFW.md or Addons-NSFW.md

### 3.2 Interaction Rules (ASK system, applies globally)

1. AI优先从题库选取选项 → saves tokens
2. 题库没有时 → AI自主生成，并保存回题库 (gets smarter over time)
3. Each option set includes hint: ⚠️ or type your own answer above
4. Generated new content → SFW存Addons-SFW.md / NSFW存Addons-NSFW.md

### 3.3 SFW Classification Database

#### Style

| Category | Prompt Keywords |
|----------|----------------|
| Realistic | realistic photo, natural lighting, detailed skin texture |
| Anime | anime style, cel shading, clean lines |
| Semi-Realistic | semi-realistic, detailed, painterly |
| 3D | 3D render, octane render, soft lighting |
| Oil Painting | oil painting, brush strokes, canvas texture |
| Watercolor | watercolor, wash, soft edges |
| Sketch | pencil sketch, hand-drawn, line art |
| Pixel | pixel art, 8-bit, retro game |
| Cyberpunk | cyberpunk, neon lights, futuristic city |

#### Type

| Category | Prompt Keywords |
|----------|----------------|
| Campus | school uniform, campus, student |
| Daily | casual, everyday, natural |
| Portrait | portrait, close-up, facial focus |
| Fashion | fashion, haute couture, runway |
| Fantasy | fantasy, magical, ethereal |
| Artistic | artistic, creative, abstract |
| Sports | sportswear, athletic, dynamic |
| Cute | cute, kawaii, adorable |
| Night | night scene, evening, dark |

#### Scene

Classroom, Library, Playground, Dormitory, Bedroom, Street, Outdoor, Cafe, Balcony, Rooftop, Convention, Studio, Park, Hallway, Living Room, Kitchen, Window Side, Street Snap, Fitting Room, Castle, Forest, Art Studio, Workshop, Dance Room, Swimming Pool, Garden, Dessert Shop, Neon Street, Bar, Overpass, Indoor

#### Detail Tags (★ = NSFW-extensible)

| Tag | Description |
|-----|-------------|
| ★Exposure | exposure level (see NSFW extension) |
| ★Voyeur | voyeur perspective |
| ★Teasing | teasing pose or expression |
| ★Half-Nude | partially undressed |
| ★Accidental | accidental exposure |
| Reading | reading a book/phone |
| Studying | studying at desk |
| Sleeping | sleeping, eyes closed |
| Eating | eating food |
| Walking | walking, strolling |
| Running | running, jogging |
| Dancing | dancing, moving |
| Singing | singing, microphone |

#### Dimension Reference

| Dimension | Questions |
|-----------|----------|
| Subject | Age? Gender? Number of characters? |
| Build | Slim / Athletic / Curvy / Petite |
| Clothing | What are they wearing? Color? Style? |
| Hair | Length? Color? Style? |
| Expression | Happy / Serious / Surprised / Shy / Blank |
| Pose | Standing / Sitting / Lying / Leaning / Bending |
| Angle | Front view / Side view / Back view / Top-down / Low angle |
| Lighting | Natural / Studio / Backlit / Neon / Candlelight |
| Environment | Indoor / Outdoor / Time of day / Weather |
| Atmosphere | Cozy / Energetic / Mysterious / Romantic / Lonely |

### 3.4 NSFW Extension Loading

**Condition**: `NSFW_Patch.md` exists on disk at project root.

**Loading order**: `skill.md (base)` → `NSFW_Patch.md (extension)` → `Addons-*.md (user additions)`

**追加维度 (Additional Dimensions, NSFW only)**:

| Dimension Key | Description | Example Options |
|---------------|-------------|-----------------|
| ★Exposure | Exposure level | Fully nude / Partially hidden / See-through / Clothes half off |
| ★Pubic | Pubic hair status | Shaven / Trimmed / Natural / Brazilian wax |
| ★Contact | Body contact | Masturbating / Toys / Bondage / No contact |
| ★Situation | Situation type | Accidental exposure / Teasing / Voyeur / Forced |

**NSFW Detail Extension by Scene** (loaded from NSFW_Patch.md):
- Classroom: ★exposure, ★masturbation, ★half-nude, ★undressing, ★teasing, ★toys, ★voyeur, ★rear-entry
- (and 27+ more scenes defined in NSFW_Patch.md)

### 3.5 Addons Storage

| Content Type | Storage File |
|-------------|-------------|
| SFW user additions | `Addons-SFW.md` |
| NSFW user additions | `Addons-NSFW.md` |

**Data reading priority**: `skill.md` (base) → `NSFW_Patch.md` (patch) → `Addons-*` (user extensions)

---

## 4. Confirmation & Review

### 4.1 Display Format

```
┌─ 📋 Confirm Your Scene ───────────────────────────┐
│   [Tags] SFW / Realistic / Campus / Classroom      │
│   [Details] Female / 17 / Uniform / Black long hair │
│   [Description] ...                                │
│   [✅ Looks good] [✏️ Something's wrong]           │
└────────────────────────────────────────────────────┘
```

### 4.2 Modification Loop

| User Action | AI Response |
|-------------|-------------|
| [✅ Looks good] | Save memory → proceed to platform branch |
| [✏️ Something's wrong] | Show multi-select categories → user picks which to change → AI goes back to that round → modify → re-display → loop until ✅ |

---

## 5. Platform Branches

### 5.1 Branch A: ComfyUI

#### Incremental Scan (user invisible)

```
① Scan checkpoints/ directory
② Read user_profile.md model list
③ Diff:
   ├── Disk has / Record missing → new model: detect architecture + tag
   ├── Disk has / Record has → unchanged
   └── Record has / Disk missing → removed: delete from user_profile
④ Hardware detection → tier assignment
```

#### Model Selection (user visible)

```
AI filters models by current style tags → show options via ASK rules
← User selects model
```

#### Prompt Assembly (user invisible)

```
Positive prompt = tags (English directly) + scene description (translate zh→en if needed)
Negative prompt = AI dynamically generates based on positive content (no template)
```

#### API Submission (user invisible)

```
① Read model tags + tier from user_profile.md
② Override params by tier (Ultra unlimited / High ≤40 steps / Medium ≤30 steps / Low ≤25 steps)
③ Assemble JSON from tags (node_checkpoint/node_clip/node_vae/node_latent/params)
④ POST → curl -X POST http://127.0.0.1:8188/prompt → get prompt_id
```

#### Smart Polling (user invisible)

```
① Estimate time: steps × 1.2s
② First check at 70% of estimate
③ If not done → check every 2s, timeout = max(estimate×2, 120s)
④ Success → extract outputs[0].images[0].filename
⑤ Timeout (>120s) → suggest lower resolution or different model
⑥ Error → report specific error → retry with adjusted params
```

#### Preview (user visible)

```
① Copy from ComfyUI/output/ to version directory → Preview_v{N}.png
② Auto-open image
③ ASK: [✅ Satisfied, save] [✏️ Not satisfied, modify]
```

#### Modification Loop (user visible)

```
User picks ✏️ → multi-select: [Pose] [Quality] [Character] [Scene] [Other]
→ AI追问 one by one → update Workbench → re-run API → new Preview
→ ASK again → loop until satisfied
```

#### Save & Cleanup (user invisible → notify user)

```
① Final image → Obsidian/ComfyMuse/Images/{命名}.png
② Final description → Obsidian/ComfyMuse/Cases/{命名}.md
③ Write to Obsidian/ComfyMuse/Core.md (update preferences + case links)
④ Clean ComfyUI_output/*.png (temp files)
⑤ Sync user_profile model list
⑥ Notify user: "Saved → Obsidian ComfyMuse/"
```

**Naming convention**:
- lang=zh-CN → `YYYYMMDD_场景_细则_风格` (Chinese)
- lang=en → `YYYYMMDD_scene_detail_style` (English)

**Version archive**: each generation writes to `Versions/v{N}`. User says "go back to v2" → restore from Versions → re-run API.

### 5.2 Branch B: Universal Text-to-Image

```
lang=zh-CN → output Chinese prompt (copyable)
lang=en    → output English prompt (copyable)
End
```

---

## 6. Error Handling (Any Stage)

| Error | Handling |
|-------|----------|
| Config/user_profile corrupted | Re-run configuration flow |
| ComfyUI path invalid | Guide re-enter / AI search |
| AI can't find ComfyUI | Try another drive / manual / official install guide |
| ComfyUI API no response | "ComfyUI not running" + wait and retry |
| ComfyUI API returns error | Report error + retry with different params |
| Generation timeout (>120s) | Suggest lower resolution or different model |
| Image filename read failure | Guide user to check output/ directory |
| User says "cancel/stop/forget it" | End flow, don't save |
| User says "go back/re-do/modify" | Go back one step, preserve filled content |

---

## 7. Language Pack (English Base)

| Key | English |
|-----|---------|
| `lang_select` | 语言 / Language |
| `continue_generate` | Start |
| `config_complete` | Config done! Start? |
| `mode_sfw` | Safe |
| `mode_nsfw` | Adult |
| `confirm_title` | Confirm Your Scene |
| `confirm_ok` | Looks good |
| `confirm_edit` | Something's wrong |
| `satisfied_save` | Satisfied, save |
| `satisfied_modify` | Not satisfied, modify |
| `saved_notify` | Saved → Obsidian ComfyMuse/ |

> **Rule**: All UI text, questions, and options are stored in English. When `lang=zh-CN`, AI translates at runtime. `lang_select` is the only place where both languages appear together.

---

## 8. Data Files Overview

| File | Path | Purpose |
|------|------|---------|
| `skill.md` | project root | This file — complete skill definition |
| `NSFW_Patch.md` | project root | NSFW extension (optional download) |
| `Config/user_profile.md` | auto-generated | User config: lang, platform, hardware, models |
| `Addons-SFW.md` | auto-generated | SFW user custom additions |
| `Addons-NSFW.md` | auto-generated | NSFW user custom additions |
| `Versions/` | auto-generated | Version archive per generation |
| `Workbench.md` | auto-generated | Current draft workspace |
| Obsidian/ComfyMuse/Cases/ | user's vault | Saved successful cases |
| Obsidian/ComfyMuse/Images/ | user's vault | Saved generated images |
| Obsidian/ComfyMuse/Core.md | user's vault | Preference + case index |
