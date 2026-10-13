# xsai · 产品宣传视频制作

**`xsai-promo-video`** —— 只用代码 + 真实产品截图，做出一支合格的产品宣传片。不用剪辑软件，不用拉时间轴。

这套流程的价值不在「做出一个片子」，而在**片子上的每个判断都能被一条命令证伪**：
「动效差点意思」变成帧间差分的一个数，「像 PPT」变成镜头数与镜头内运动的计数，「卡点不准」变成毫秒级的偏移量。

```text
① 定验收线 → ② 写创作稿 → ③ 备真实素材 → ④ 拆解参考片
      → ⑤ 搭建合成 → ⑥ 渲染 → ⑦ 对抗性验收   （不过线 → 回到 ⑤）
```

---

## 一、演示视频

> **演示成片 / Demo film**（点击封面在新标签打开，或直接播放下方视频）

<p align="center">
  <a href="https://github.com/user-attachments/assets/d57bfc55-5acf-4f7f-b2da-78c640df4a79">
    <img src="assets/demo-poster.jpg" alt="xsai · 产品宣传视频制作 — 演示成片封面" width="100%">
  </a>
</p>

https://github.com/user-attachments/assets/d57bfc55-5acf-4f7f-b2da-78c640df4a79

*59.9s · 1920×1080 · 30fps — 直接下载：[demo/xsai-promo-video-demo.mp4](demo/xsai-promo-video-demo.mp4)*

> **说明：本视频使用 DeepSeek 7 轮对话产出。**
>
> 从定验收标准、写创作稿、选素材与卡点，到改文案、改配色、改结尾节奏，全程由 7 轮对话驱动完成，
> 没有手工剪辑，也没有打开过任何剪辑软件。

---

## 二、它和「随便生成一段视频」有什么区别

| | 常见做法 | 这套流程 |
|---|---|---|
| 好坏标准 | 靠感觉，反复返工 | 先写验收标准（加权维度 + 硬否决项），再动一帧 |
| 画面素材 | 生成的假界面、编的数字 | 真实产品截图、真实数据 |
| 动效 | 「再加点效果」 | 先量出来是镜头太长、还是镜头内没东西在动 |
| 卡点 | 肉眼对齐 | 从音乐源文件算出真实脉冲，全片吸附 |
| 验收 | 看一遍觉得还行 | `audit_video.py` 逐项测，退出码 0/1 |

---

## 三、安装

这是一个标准 skill 目录，直接克隆到 skills 路径即可：

```bash
# 用 Codex / Claude Code 的 agent skills 目录
git clone https://github.com/lianying1716/xsai-promo-video.git \
  ~/.agents/skills/xsai-promo-video
```

克隆后把整个仓库当作 `xsai-promo-video` 这个技能使用，入口是 `SKILL.md`。

### 依赖

- **Node**（`npx hyperframes`）——默认渲染器。流程本身与框架无关，任何能输出 MP4 的渲染器都行。
- **Python 3.9+**，需要 `numpy`、`scipy`、`librosa`、`Pillow`。
- **ffmpeg / ffprobe** 在 `PATH` 里。
- 可选：一个 OpenAI 兼容的多模态模型，供 `vision_teardown.py` 做语义拆解
  （设置 `VISION_ENDPOINT` / `VISION_MODEL` / `VISION_API_KEY`）。

---

## 四、五分钟上手

```bash
# 1. 把 starter 拷成你自己的片子
cp -r assets/starter/my-film ~/my-film && cd ~/my-film

# 2. 先证明工具链是通的（开箱就是 0 error）
npx hyperframes@0.8.147 check
npx hyperframes@0.8.147 render -o renders/smoke.mp4 --crf 24

# 3. 换成你自己的文案 / 真实封面 / BGM，然后走完六步
```

starter 里已经写好并验证过的模式：设计令牌、真实封面卡片、「生长式」入场、三层摄影机、结尾留白。

---

## 五、六步流程

### ① 定验收线（最容易被跳过，也最贵）

先写 `ACCEPTANCE.md`：一句话定义「优秀」，加权评分维度，加**硬否决项**（触犯任意一条直接不通过）。
模板见 `references/acceptance-rubric.md`。

### ② 写创作稿

产出一句话核心主张 + 逐镜文案表 + 视觉语言 + 分镜表。
开场 3 秒必须用**观众自己的话**回答「我为什么要在乎」；每个镜头只干一件事。
见 `references/creative-brief.md`。

### ③ 备真实素材

铁律：**产品有真界面就截真界面，有真数据就用真数字。**
先把演示数据做全真，否则成片全是占位。
Playwright 用 `deviceScaleFactor=2` 采集；「活着」的动作要录成真视频，不能用静图。
见 `references/asset-production.md`。

### ④ 拆解参考片（两遍）

至少拆 **3 部**大厂片（一部是风格，三部才是规律）：

- **量化**：`references/reference-teardown.md` —— 镜头数/时长、镜头内运动、卡点偏移。
- **语义**：`references/multimodal-teardown.md` —— 带时间戳的关键帧板喂多模态模型，跑 7 个视角，
  最后一步 `improve_ours` 把参考片和你的片子同屏对比，直接产出改进清单。

**跨片复现的四条规律**收在 `references/design-rules.md`，动手前先读，比重新推导便宜。

### ⑤ 搭建合成

一条主时间线；场景显式声明起止；**一切卡点吸附到音乐的「真实脉冲」**，不要相信标注的 BPM；
数字用逐帧插值，不要「每拍 set 一次」；每个场景都要有环境运动层。
HUD 层（角标、标注）不要跟着摄影机变换走，否则边缘文字会被推出画布。

### ⑥ 渲染

```bash
npx hyperframes@0.8.147 check                       # 必须 0 error
npx hyperframes@0.8.147 render -o renders/xxx.mp4 --crf 20
```

全屏缓慢缩放的片子，CRF 给太低体积会翻几倍；CRF 20 一般观感无差。

### ⑦ 对抗性验收（真正的核心）

不要问「好不好看」，要问**「哪一帧能证明它是死的 / 假的 / 错的」**，然后去量。

```bash
python3 scripts/audit_video.py renders/xxx.mp4 \
  --total 59.9 --grid-json assets/beat-grid.json --bgm assets/bgm.mp3 \
  --countup "t0:t1:x:y:w:h" --playback "..." --scale-windows "..." \
  --json evidence/audit.json        # 退出码 0 = 过线，1 = 未过
```

它会测：规格、开场音频能量比、分块运动（**frozen 与 low-motion 分开报**）、空/黑帧、
卡点偏移、推近倍数（区域内模板匹配）、真视频是否在播、数字是否连续。

细节与阈值见 `references/audit-method.md`。

---

## 六、目录结构

```text
xsai-promo-video/
├── SKILL.md                     技能入口：流程、硬规矩、按需读哪份文档
├── README.md                    英文说明
├── README.zh-CN.md              本文件（中文说明）
├── demo/                        演示成片
├── references/
│   ├── acceptance-rubric.md     验收标准（维度 + 硬否决项）
│   ├── creative-brief.md        创作稿 / 分镜 / 视觉语言格式
│   ├── asset-production.md      真实素材怎么做、怎么不被做假
│   ├── design-rules.md          五部参考片提炼出的四条规律（附测量数据）
│   ├── multimodal-teardown.md   多模态语义拆解（含调用格式与两个坑）
│   ├── reference-teardown.md    量化拆解
│   ├── audit-method.md          每一项检查、阈值，以及「指标本身会骗人」
│   └── pipeline-checklist.md    最短上手清单
├── scripts/
│   ├── audit_video.py           对抗性验收器（退出码 0/1 + JSON 证据）
│   ├── make_beat_grid.py        从音乐里算真实脉冲节拍网格
│   ├── make_frame_sheets.py     视频 → 带时间戳的关键帧板
│   ├── vision_teardown.py       关键帧板 → 7 视角语义拆解
│   └── teardown_video.py        六维量化拆解（结构/运动/文字/声音/卡点/配色）
└── assets/
    ├── fonts/                   JetBrains Mono（OFL）
    └── starter/my-film/         可直接运行的 12s 骨架（开箱 0 error）
```

---

## 七、四条规律（一句话版）

1. **主角是人，或人的一个动作，不是界面本身。**
2. **每个镜头里都得有东西在动**（摄影机 + 环境 + 内容，三层）。
3. **一屏一个主张，结尾要停得住**（2–4 秒近静帧）。
4. **配色是封闭系统**：低饱和底 + 一个承载含义的强调色。

完整证据与做法见 `references/design-rules.md`。

---

## 八、许可

代码与文档：**MIT**，见 `LICENSE`。
第三方素材：见 `THIRD_PARTY_NOTICES.md`。

本仓库不包含任何第三方产品截图、专有字体或商业音乐；starter 里的占位封面与测试音为开放素材，使用前请替换。
