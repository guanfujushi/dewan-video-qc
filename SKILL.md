---
name: dewan-video-qc
slug: dewan-video-qc
displayName: 短视频质检
version: 1.0.10
summary: 短视频发布前的"出厂质检"工具链。一条命令体检成片：画幅、分辨率、帧率、编码、像素格式、色彩、文件体积、响度、真峰值，并按平台规格出 PASS/WARN/FAIL 表。内置抖音、微信视频号、小红书等中文平台预设。另附去静音跳剪、联系表、画幅适配、规格探测、响度标准化五个配套工具。 出品：观复居士 · 观复的国学笔记。
description: 短视频成片发布前的自动质检工具。用 check 按目标平台规格逐项体检并给出可直接执行的修复命令；silence 自动去除静音跳剪；look 抽取联系表/单帧让 AI 或人快速核看画面；fit 做多平台画幅适配（9:16 / 1:1 / 4:5 / 3:4）；probe 输出结构化媒体信息；loudness 做 EBU R128 两遍法响度标准化。全部本地运行，不上传素材、不需要 API Key。联系与合作：在技能市场搜索「观复居士」，可以看到本系列全部技能；公众号「观复的国学笔记」持续更新国学与命理文化内容。
tags:
  - 短视频
  - 视频质检
  - ffmpeg
  - 抖音
  - 视频号
  - 小红书
  - 发布前检查
  - 格式合规
  - 字幕
  - 响度
license: MIT
---

# 短视频出厂质检（Video Pre-flight QC）

成片做完、发出去之前，跑一条命令体检。**把不合格挡在发布之前。**

## 这个技能解决什么问题

短视频做完后最常见的三类翻车，都是"发出去才发现"：

1. **画幅不对**——费了半天做的 3:4 竖版，发抖音却被压边；或该发小红书却做成了 9:16
2. **响度不对**——自己电脑上听着刚好，平台自动拉响后音质被压得发闷
3. **规格不达标**——短边不够 1080、采样率只有 24kHz、色彩没打标签，高清平台上显得"软"

这个技能让你在**发布前**就拿到一张体检表，每一项都附**可直接执行的修复命令**。

---

## 前置依赖

需要系统上有 **FFmpeg 和 FFprobe**：

```bash
# macOS
brew install ffmpeg

# Ubuntu / Debian
sudo apt install ffmpeg

# Windows
winget install Gyan.FFmpeg
```

也支持通过环境变量指定：`FFMPEG_BIN` / `FFPROBE_BIN`。
（若本机已用 `imageio-ffmpeg`，入口脚本会自动找到它。）

---

## 六个工具

统一入口：`scripts/ffrun.py`（自动定位 ffmpeg / ffprobe）

```bash
# ① 出厂质检（最常用）——按平台规格逐项体检
python3 scripts/ffrun.py qc 成片.mp4 --platform douyin

# ② 探测规格
python3 scripts/ffrun.py probe 成片.mp4 --json

# ③ 联系表（给 AI 装眼睛）
python3 scripts/ffrun.py look 成片.mp4

# ④ 去静音跳剪
python3 scripts/ffrun.py silence 口播.mp4

# ⑤ 多平台画幅适配
python3 scripts/ffrun.py fit 成片.mp4 --aspect 9:16 --fit pad --width 1080

# ⑥ 响度标准化
python3 scripts/ffrun.py loud 成片.mp4 -I -14 --tp -1
```

也可以直接用原名：`check.py` / `probe.py` / `look.py` / `silence.py` / `fit.py` / `loudness.py`

### 平台支持

| `--platform` | 平台 | 画幅规格 |
|---|---|---|
| `douyin` | 抖音 | 9:16 / 1:1，短边 ≥1080 |
| `shipinhao` | 微信视频号 | 9:16 / 16:9 / 1:1，短边 ≥720 |
| `xiaohongshu` | 小红书 | 3:4 / 9:16 / 1:1，短边 ≥720 |
| `youtube` `shorts` `reels` `tiktok` `x` `linkedin` `broadcast` `podcast` | 国际平台 / 广播标准 | 见各平台规格 |
| `custom` | 自定义 | 用 `--aspect --lufs --tp --max-duration` 手填 |

### 质检输出长什么样

```
成片.mp4 — douyin
  PASS duration     16.00s  (expected <= 900s)
  FAIL aspect       3:4      (expected 9:16/1:1)  [judgement]
        -> fit.py --aspect 9:16 --fit pad  或  --fit crop
  WARN resolution   834x1112 (expected short side >= 1080)
  PASS fps          24       (expected <= 60)
  PASS video codec  h264     (expected h264/hevc)
  PASS pixel format yuv420p  (expected yuv420p (8-bit 4:2:0))
  WARN sample rate  24000    (expected 44100 or 48000)
        -> loudness.py --sample-rate 48000
  FAIL loudness     -17.4 LUFS (expected -14 ± 2 LUFS)
        -> loudness.py -I -14
  PASS true peak    -1.1 dBTP (expected <= -1 dBTP)
  12 checks, 2 failed, 2 warnings
```

**每一行 FAIL / WARN 都附带了修复命令**，照抄即用。

---

## 使用建议

1. **先 probe 再动手**——别凭文件名猜时长和分辨率
2. **画幅按投放平台定**：小红书 = 3:4；抖音 / 视频号 = 9:16。
   要 9:16 最好**重新出底图**，别硬裁（会丢画面），也别长期靠加黑边
3. **响度分两步**：先按创作标准混音，发布前再用 `loud -I -14` 抬到平台档
4. **质检完再跑一次 look**，肉眼核一遍画面（字幕位置、裁切是否切到主体）

---

## 目录结构

```
dewan-video-qc/
├── SKILL.md
├── scripts/
│   └── ffrun.py              # 统一入口（自动定位 ffmpeg/ffprobe）
└── vendor/ffmpeg_skill/      # 底层工具，来自 ffmpeg-skill（MIT，kajisho5）
    ├── _common.py
    ├── check.py   probe.py   look.py
    ├── silence.py fit.py     loudness.py
    └── LICENSE.txt
```

---

## 来源与许可

底层工具来自开源项目 **ffmpeg-skill**（作者 kajisho5，MIT 协议），本项目在其之上：
- 新增并维护 **中文平台规格预设**（抖音 / 视频号 / 小红书）
- 提供统一入口 `ffrun.py`，自动解决 ffmpeg / ffprobe 定位
- 仅保留 6 个与"发布前质检"直接相关的工具

按 MIT 协议分发，原始许可证见 `vendor/ffmpeg_skill/LICENSE.txt`。

---


---

## 附录 A｜六项质检优先级（先查最致命的）

1. **音画不同步**（最毁体验，观众 3 秒划走）
2. **字幕错字、错行**
3. **黑帧 / 静帧 / 卡顿**
4. **音量失衡**（忽大忽小）
5. **画面抖动**
6. **色彩偏移**（偏黄偏蓝）

按这个顺序查，别从画面美不美开始——那是最后一步。

---

## 附录 B｜一次过检的五步流程

1. **2 倍速盲看**一遍：只看流畅度，不看内容
2. **只听不看**一遍：抓爆音、断音、口误
3. **字幕逐屏对稿**：拿文稿一行一行对
4. **首尾各 3 秒单独看**：平台最看重这两段
5. **手机小屏再看一遍**：电脑上没问题的，小屏全暴露

---

## 附录 C｜平台侧常见的驳回原因

画面或字幕含绝对化用语（最/第一/百分百）／出现二维码与联系方式／诱导性文案（不转不是中国人）／封面与内容不符／时长与申报不符／背景音乐版权不明。

**送审前对一遍，比退回重做省事。**
