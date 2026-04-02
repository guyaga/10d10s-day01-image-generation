<p align="center">
  <img src="cover.jpg" alt="Day 03 — Image Generation with Gemini 3 Pro" width="100%">
</p>

<h1 align="center">Day 03 — Image Generation</h1>
<p align="center">
  <strong>10 Days 10 Skills</strong> · Claude Code Course by <a href="https://bestguy.ai">Guy Aga</a>
</p>
<p align="center">
  <img src="https://img.shields.io/badge/Service-Gemini%203%20Pro-E63B2E?style=flat-square" alt="Gemini 3 Pro">
  <img src="https://img.shields.io/badge/Skill-nano--banano--pro-111111?style=flat-square" alt="Skill">
  <img src="https://img.shields.io/badge/Level-Beginner-E8E4DD?style=flat-square&labelColor=111111" alt="Beginner">
</p>

---

## What is This?

This skill lets you **generate and edit professional images** directly from Claude Code using Google's Gemini 3 Pro model. No Photoshop, no design tools, no design skills needed — just describe what you want in words.

### What Can You Create?

- **Social media graphics** — Instagram posts, stories, LinkedIn banners
- **Infographics** — Educational visuals with accurate, readable text
- **Product mockups** — Professional marketing assets
- **Brand materials** — Logos, covers, presentations
- **Data visualizations** — Charts and graphs from real-time data
- **Photo editing** — Edit existing photos with text prompts

### Why Gemini 3 Pro?

| Feature | What It Means for You |
|---------|----------------------|
| **4K Resolution** | Print-quality images, not blurry thumbnails |
| **Text Rendering** | Actually readable text on images (most AI models fail at this) |
| **14 Reference Images** | Send your logo, photos, examples — the AI follows them |
| **Multi-Turn Editing** | Generate an image, then say "make the background blue" — it remembers |
| **Google Search Grounding** | Create visuals based on real-time data (weather, stocks, trends) |
| **Thinking Mode** | The model reasons through complex prompts for better results |

---

## Prerequisites

Before you start, make sure you have:

- [ ] **Claude Code** installed (Pro or Max subscription)
- [ ] **Node.js 20+** installed on your computer
- [ ] A **Google AI Studio** account (free)
- [ ] A **Gemini API Key** (free tier available)

---

## Step 1: Get Your API Key

1. Go to [Google AI Studio](https://aistudio.google.com/apikey)
2. Click **"Create API Key"**
3. Select or create a Google Cloud project
4. Copy your API key

> **Important:** The free tier gives you **enough requests to learn and practice**. You don't need to pay anything for this lesson.

---

## Step 2: Set Up Your Environment

### Option A: Environment Variable (Recommended)

```bash
# Windows (PowerShell)
$env:GEMINI_API_KEY="your-api-key-here"

# Windows (Command Prompt)
set GEMINI_API_KEY=your-api-key-here

# macOS / Linux
export GEMINI_API_KEY=your-api-key-here
```

### Option B: .env File

Create a `.env` file in your project folder:

```
GEMINI_API_KEY=your-api-key-here
```

---

## Step 3: Install the Skill

### In Claude Code:

The skill is already installed if you're following the course. If not:

```bash
# Navigate to your skills folder
cd ~/.claude/skills/

# Clone this skill
git clone https://github.com/guyaga/10d10s-day03-image-generation nano-banano-pro
```

### Install the SDK:

```bash
npm install @google/genai
```

---

## Step 4: Your First Image

The simplest way — just ask Claude Code:

```
Generate a professional social media post about AI tools 
using Swiss design style with red and black colors
```

Claude Code will use the skill automatically and generate the image for you.

### Behind the Scenes

Here's what the skill does (you don't need to write this code — Claude does it for you):

```javascript
import { GoogleGenAI } from "@google/genai";

const ai = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });

const response = await ai.models.generateContent({
  model: 'gemini-3-pro-image-preview',
  contents: 'Your prompt here',
  config: {
    responseModalities: ['TEXT', 'IMAGE'],
    imageConfig: {
      aspectRatio: '16:9',  // 1:1, 4:3, 3:4, 16:9, 9:16, 5:4
      imageSize: '2K',      // 1K, 2K, 4K
    },
  },
});
```

---

## Step 5: Using Reference Images

This is where it gets powerful. You can send your own images as references:

```
Generate a branded social media post using my logo from logo.png 
and my photo from me.jpg in Swiss design style
```

The skill supports **up to 14 reference images**:
- Up to 6 object images (logos, products, brand assets)
- Up to 5 human images (photos of people for consistency)

---

## Step 6: Multi-Turn Editing

Generate an image, then refine it with follow-up requests:

```
1. "Create a course cover image with bold typography"
2. "Make the title bigger and add a red accent line"
3. "Change the background to pure black"
4. "Add my logo in the top left corner"
```

Each edit builds on the previous result — like having a conversation with a designer.

---

## Configuration Options

| Option | Values | What It Does |
|--------|--------|-------------|
| `imageSize` | `1K`, `2K`, `4K` | Output resolution (always use 2K minimum) |
| `aspectRatio` | `1:1`, `4:3`, `3:4`, `16:9`, `9:16`, `5:4` | Image dimensions |
| `tools` | `[{googleSearch: {}}]` | Enables real-time data in images |

### Common Aspect Ratios

| Use Case | Ratio |
|----------|-------|
| Instagram Post | `1:1` or `4:3` |
| Instagram Story | `9:16` |
| YouTube Thumbnail | `16:9` |
| LinkedIn Banner | `16:9` |
| Presentation Slide | `16:9` |

---

## Tips for Better Results

1. **Be specific** — "A Swiss-design social media post with bold red typography on black background" beats "make me a cool image"
2. **Mention colors by hex** — "Use #E63B2E for accents" gives precise results
3. **Describe layout** — "Logo top-left, title center, photo on right side"
4. **Reference styles** — "Like a high-end tech conference banner"
5. **Always use 2K** — Never use 1K, the quality difference is huge

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "API key not found" | Make sure `GEMINI_API_KEY` is set in your environment or `.env` file |
| Blurry output | Change `imageSize` from `'1K'` to `'2K'` |
| Text not readable | Add "text must be perfectly legible" to your prompt |
| Wrong colors | Specify exact hex codes in the prompt |
| Generation fails | Check your API quota at [Google AI Studio](https://aistudio.google.com/) |

---

## Links

- [Google AI Studio — Get API Key](https://aistudio.google.com/apikey)
- [Gemini API Pricing](https://ai.google.dev/pricing)
- [Course Page — bestguy.ai](https://bestguy.ai/course/10-days-10-skills)
- [HTML Skill Guide (Hebrew)](https://bestguy.ai/course/guides/day03-image-generation.html)

---

<p align="center">
  <strong>10 Days 10 Skills</strong> — Claude Code Course<br>
  <a href="https://bestguy.ai">bestguy.ai</a> · Guy Aga &copy; 2026
</p>
