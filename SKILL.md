---
name: nano-banano-pro
description: Generate and edit images with Google's Nano Banana 2.1 (gemini-nano-banana-2.1), the default since October 2026, with Nano Banana Pro (gemini-3-pro-image) as an option. Use when the user wants to create images, edit images, generate infographics, ads, posters, social graphics, visualizations, or any image generation task. Supports multi-turn editing, up to 14 reference images, character consistency, 1K/2K/4K output, ultra-wide aspect ratios, Google Search grounding, configurable thinking, and strong text rendering including Hebrew.
allowed-tools: Read, Write, Edit, Bash, Glob
---

# Nano Banana 2.1 (Gemini Image Generation)

Generate and edit professional images with Google's Gemini image models. The default model is **Nano Banana 2.1** (`gemini-nano-banana-2.1`). It's better, cheaper and supports more formats than Nano Banana Pro.

## Which model to use

| | **Nano Banana 2.1** (default) | Nano Banana Pro |
|---|---|---|
| Model code | `gemini-nano-banana-2.1` | `gemini-3-pro-image` (`gemini-3-pro-image-preview` still works) |
| Price per 2K image | ~$0.05 (+ ~$0.01 thinking) | $0.134 |
| Price per 4K image | $0.113 | $0.24 |
| Speed at 2K | ~25 s (≈15 s with thinking `MINIMAL`) | ~25 s |
| Ultra-wide 1:4, 4:1, 1:8, 8:1 | Yes | No (HTTP 400) |
| Thinking level | `MINIMAL` / `MEDIUM` (default) / `HIGH` | Fixed |

Batch mode halves both prices. Source: https://ai.google.dev/gemini-api/docs/pricing

**Use 2.1 for everything by default.** In a 34-case test (Hebrew marketing, edge cases, creative work, fonts, long text, text placement, character consistency), 2.1 won 16 cases, Pro won 3 and 15 were ties. Switch to Pro only when you need a layout followed very literally: 2.1 sometimes "art-directs", for example by turning a flat poster into a mockup on a wall.

## Setup

1. Set your API key as an environment variable:
   ```bash
   # Windows (Command Prompt)
   set GEMINI_API_KEY=your-api-key-here

   # Windows (PowerShell)
   $env:GEMINI_API_KEY="your-api-key-here"

   # Linux/macOS
   export GEMINI_API_KEY=your-api-key-here
   ```

2. Install the SDK:
   ```bash
   npm install @google/genai
   ```

## Quick Start - Single Image Generation

```javascript
import { GoogleGenAI } from "@google/genai";
import * as fs from "node:fs";

const ai = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });

const response = await ai.models.generateContent({
  model: 'gemini-nano-banana-2.1',
  contents: 'Your prompt here',
  config: {
    responseModalities: ['TEXT', 'IMAGE'],
    imageConfig: {
      aspectRatio: '16:9',  // see the aspect ratio list below
      imageSize: '2K',      // '1K', '2K', '4K' (must be uppercase)
    },
  },
});

// Save the generated image
for (const part of response.candidates[0].content.parts) {
  if (part.text) {
    console.log(part.text);
  } else if (part.inlineData) {
    const buffer = Buffer.from(part.inlineData.data, "base64");
    fs.writeFileSync("output.png", buffer);
  }
}
```

## Thinking Level (2.1 only)

2.1 reasons before drawing. Pick the level per job:

| Level | Time at 2K | Use for |
|---|---|---|
| `MINIMAL` | ~15 s | Drafts, quick variations, simple scenes |
| `MEDIUM` (default) | ~25 s | Most work |
| `HIGH` | 25–50 s | Text-heavy images (especially Hebrew), exact counts, hands, complex instructions |

```javascript
config: {
  responseModalities: ['TEXT', 'IMAGE'],
  imageConfig: { aspectRatio: '9:16', imageSize: '2K' },
  thinkingConfig: { thinkingLevel: 'HIGH' },
}
```

In testing, `HIGH` fixed a Hebrew spelling mistake and a finger-count mistake that `MEDIUM` made.

## Multi-Turn Conversational Editing

Use chat sessions to iteratively edit images:

```javascript
const ai = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });

const chat = ai.chats.create({
  model: "gemini-nano-banana-2.1",
  config: {
    responseModalities: ['TEXT', 'IMAGE'],
    tools: [{googleSearch: {}}],
  },
});

// First message - generate initial image
let response = await chat.sendMessage({
  message: "Create a vibrant infographic about photosynthesis"
});

// Second message - edit the image
response = await chat.sendMessage({
  message: 'Update this infographic to be in Spanish',
  config: {
    responseModalities: ['TEXT', 'IMAGE'],
    imageConfig: {
      aspectRatio: '16:9',
      imageSize: '2K',
    },
  },
});
```

To edit an existing image file in one call, send the image with the instruction and say exactly what must stay the same:

```javascript
const response = await ai.models.generateContent({
  model: 'gemini-nano-banana-2.1',
  contents: [{ role: 'user', parts: [
    { text: 'Keep the shoe, splash, composition and colors identical. Replace the Hebrew text with English: "Run Further".' },
    { inlineData: { mimeType: 'image/png', data: fs.readFileSync('ad.png').toString('base64') } },
  ]}],
  config: { responseModalities: ['TEXT', 'IMAGE'], imageConfig: { aspectRatio: '1:1', imageSize: '2K' } },
});
```

## Using Reference Images (Up to 14)

Mix reference images for the final output:
- Up to 10 object images (high-fidelity inclusion)
- Up to 4 characters kept consistent

```javascript
const contents = [
  { text: 'An office group photo of these people making funny faces.' },
  { inlineData: { mimeType: "image/jpeg", data: base64Image1 } },
  { inlineData: { mimeType: "image/jpeg", data: base64Image2 } },
  { inlineData: { mimeType: "image/jpeg", data: base64Image3 } },
];

const response = await ai.models.generateContent({
  model: 'gemini-nano-banana-2.1',
  contents: contents,
  config: {
    responseModalities: ['TEXT', 'IMAGE'],
    imageConfig: { aspectRatio: '5:4', imageSize: '2K' },
  },
});
```

## Character Consistency

What held up across scenes, a two-person scene, a turnaround sheet and a 4-page storybook:

1. **Start from a clean reference**: a portrait on a plain white background, upper body, even light. Distinctive features (glasses, a scarf, a hairstyle) make the character easier to keep.
2. **List what must stay the same** in every prompt: "keep her face, curly auburn hair, round glasses, nose ring and mustard denim jacket identical".
3. **Several people**: refer to them by order ("the woman from the first reference and the man from the second").
4. **Series without a reference** (e.g. a children's book): use a chat session, describe the character fully in the first message, then say "same character, same style" on every following page.
5. **Turnaround sheets** work well: "four full-body views side by side on white: front, three-quarter, side profile, back".

## Hebrew and Other Text

Both models write Hebrew well, but mistakes grow with text length. What we learned from testing:

- **Put every text in quotes** and add: `CRITICAL: Render all Hebrew text crisply, right-to-left, spelled exactly as given, letter by letter. No other text.`
- **Short text** (headlines, buttons, menus, tables, FAQs) usually comes out perfect.
- **Proofread anything over about 30 words.** Long paragraphs can come back with a merged or doubled word. Use thinking `HIGH` for text-heavy Hebrew images.
- **Font names are a hint, not a guarantee.** "Frank Ruhl Libre" or "Heebo Black" gets you the right family (serif vs sans, weight) but not the exact font. Describe the look as well: "tall condensed bold display", "classic Hebrew book serif with thick-thin contrast", "thin hand-drawn marker".
- **Placement works**: name where each text goes ("round sticker in the top-LEFT corner", "lower-third bar on the bottom RIGHT", "curved along the top arc of the stamp"). Text on surfaces in a photo (signs, T-shirts, chalkboards, posters) also works.
- **Check punctuation in speech bubbles.** A question mark can land on the wrong side in RTL text.
- **Mixed Hebrew and English** (Heblish) works. Say that English words stay left-to-right inside the Hebrew line.

## Google Search Grounding

Generate images based on real-time data (weather, stocks, events):

```javascript
const response = await ai.models.generateContent({
  model: 'gemini-nano-banana-2.1',
  contents: 'Visualize the current weather forecast for San Francisco as a modern chart',
  config: {
    responseModalities: ['TEXT', 'IMAGE'],
    tools: [{googleSearch: {}}],
    imageConfig: { aspectRatio: '16:9', imageSize: '2K' },
  },
});

// Response includes groundingMetadata with searchEntryPoint and groundingChunks
```

## Key Configuration Options

| Option | Values | Notes |
|--------|--------|-------|
| `imageSize` | `'1K'`, `'2K'`, `'4K'` | Must be uppercase. 1K ≈ 15–20 s, 2K ≈ 25 s, 4K ≈ 60–80 s |
| `aspectRatio` | `'1:1'`, `'2:3'`, `'3:2'`, `'3:4'`, `'4:3'`, `'4:5'`, `'5:4'`, `'9:16'`, `'16:9'`, `'21:9'`, plus on 2.1 only `'1:4'`, `'4:1'`, `'1:8'`, `'8:1'` | Ultra-wide is seamless on 2.1 (good for website heroes and banners) |
| `responseModalities` | `['TEXT', 'IMAGE']` | Required for image output |
| `tools` | `[{googleSearch: {}}]` | Enable real-time data grounding |
| `thinkingConfig` | `{ thinkingLevel: 'MINIMAL' \| 'MEDIUM' \| 'HIGH' }` | 2.1 only |

## Accessing Thinking Process

The model uses "thinking" for complex prompts (enabled by default):

```javascript
for (const part of response.candidates[0].content.parts) {
  if (part.thought) {
    // This is an interim thought image (not charged)
    if (part.text) console.log('Thought:', part.text);
  } else {
    // This is the final output
    if (part.inlineData) {
      fs.writeFileSync('final.png', Buffer.from(part.inlineData.data, 'base64'));
    }
  }
}
```

## Thought Signatures (Multi-Turn)

When using chat/multi-turn, thought signatures are handled automatically by the SDK. If manually managing history, pass back `thought_signature` fields exactly as received.

## Model Capabilities

- High-resolution output: 1K, 2K, 4K
- Advanced text rendering: legible text for infographics, menus, diagrams, Hebrew
- Google Search grounding: real-time data visualization
- Configurable thinking: minimal, medium, high
- Up to 14 reference images, up to 4 consistent characters
- Multi-turn conversational editing
- Ultra-wide and ultra-tall formats

## Common Use Cases

1. **Infographics**: Create educational visuals with accurate text
2. **Product mockups**: Generate professional marketing assets
3. **Data visualization**: Charts and graphs from real-time data
4. **Character consistency**: Maintain same characters across images
5. **Style transfer**: Apply artistic styles to concepts
6. **Iterative refinement**: Edit and refine through conversation
7. **Localization**: Swap the text language in a finished ad while keeping the image

## Prompting Best Practices (Official Guide)

Source: https://cloud.google.com/blog/products/ai-machine-learning/ultimate-prompting-guide-for-nano-banana

### Core Rules

1. **Be specific**: Provide concrete details on subject, lighting, and composition
2. **Use positive framing**: Describe what you want, not what you don't want (e.g., "empty street" instead of "no cars")
3. **Control the camera**: Use photographic and cinematic terms like "low angle" and "aerial view"
4. **Iterate**: Refine images with follow-up prompts in a conversational manner
5. **Start with a strong verb**: Tell the model the primary operation to perform

### Five Prompting Frameworks

#### 1. Text-to-Image Generation (no references)
**Formula**: `[Subject] + [Action] + [Location/context] + [Composition] + [Style]`

Example: "[Subject] A striking fashion model wearing a tailored brown dress. [Action] Posing with a confident stance. [Location] A deep cherry red studio backdrop. [Composition] Medium-full shot, center-framed. [Style] Fashion editorial, medium-format analog film, pronounced grain, cinematic lighting."

#### 2. Multimodal Generation (with references)
**Formula**: `[Reference images] + [Relationship instruction] + [New scenario]`

Example: "Using the attached sketch as the structure and the attached fabric sample as the texture, transform this into a high-fidelity 3D armchair render. Place it in a sun-drenched, minimalist living room."

#### 3. Image Editing
- **Semantic masking (inpainting)**: Define a "mask" through text to edit a specific part. Be explicit about what to keep the same.
- **Style transfer**: Upload a photo and ask to recreate its content in a different artistic style.
- **Adding elements**: Upload a base image and an object image, tell the model to combine them.

#### 4. Real-Time Information (Google Search Grounding)
**Formula**: `[Source/Search request] + [Analytical task] + [Visual translation]`

Example: "Search for current weather in San Francisco. Use this data to create a miniature city-in-a-cup visualization embedded in a smartphone UI."

#### 5. Text Rendering & Localization
Rules for best typographic results:
- **Use quotes**: Enclose desired words in quotes (e.g., `"Happy Birthday"`)
- **Choose a font**: Describe the typography style, and optionally name the font (e.g., "bold, white, sans-serif font")
- **Translate and localize**: Write prompt in one language, specify target language for text output
- **Text-first hack**: When generating text-heavy images, first converse to generate text concepts, then ask for the image with that text

### Creative Director Techniques

#### Lighting
- Studio setups: "three-point softbox setup"
- Dramatic: "Chiaroscuro lighting with harsh, high contrast"
- Natural: "Golden hour backlighting creating long shadows"

#### Camera, Lens & Focus
- **Hardware**: GoPro (immersive/distorted), Fujifilm (authentic color), disposable camera (nostalgic flash)
- **Lens**: "wide-angle lens" (vast scale), "macro lens" (intricate details)
- **Depth**: "shallow depth of field (f/1.8)" for bokeh, "deep focus" for sharp throughout

#### Color Grading & Film Stock
- Nostalgic: "as if on 1980s color film, slightly grainy"
- Modern moody: "cinematic color grading with muted teal tones"
- Vibrant: "high saturation, Fujifilm Velvia film simulation"

#### Materiality & Texture
Don't just say "suit jacket" — say "navy blue tweed". Not "armor" — say "ornate elven plate armor, etched with silver leaf patterns". For mockups, specify the surface: "minimalist ceramic coffee mug".

### Tech Specs Reference

| Spec | Nano Banana 2.1 (default) | Nano Banana Pro |
|------|---------------------------|-----------------|
| Model code | `gemini-nano-banana-2.1` | `gemini-3-pro-image` |
| Input tokens | 131,072 max | 65,536 max |
| Output tokens | 32,768 max | 32,768 max |
| Inputs | Text, image, video, PDF | Text, image |
| Resolutions | 1K, 2K, 4K | 1K, 2K, 4K |
| Aspect ratios | 1:1, 2:3, 3:2, 3:4, 4:3, 4:5, 5:4, 9:16, 16:9, 21:9, 1:4, 4:1, 1:8, 8:1 | 1:1, 2:3, 3:2, 3:4, 4:3, 4:5, 5:4, 9:16, 16:9, 21:9 |
| Reference images | Up to 14 (4 characters, 10 objects) | Up to 14 |
| Thinking | Minimal / medium / high | Fixed |
| Live data | Google Search grounding | Google Search grounding |
| Safety | C2PA Content Credentials + SynthID watermark | C2PA Content Credentials + SynthID watermark |
