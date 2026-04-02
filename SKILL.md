---
name: nano-banano-pro
description: Generate and edit images using Gemini 3 Pro Image Preview (gemini-3-pro-image-preview) API. Use when the user wants to create images, edit images, generate infographics, visualizations, or any image generation task. Supports multi-turn editing, up to 14 reference images, 4K resolution, Google Search grounding, and advanced text rendering.
allowed-tools: Read, Write, Edit, Bash, Glob
---

# Nano Banano Pro (Gemini 3 Pro Image Preview)

Generate and edit professional images using Google's Gemini 3 Pro Image model.

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
  model: 'gemini-3-pro-image-preview',
  contents: 'Your prompt here',
  config: {
    responseModalities: ['TEXT', 'IMAGE'],
    imageConfig: {
      aspectRatio: '16:9',  // Options: '1:1', '4:3', '3:4', '16:9', '9:16', '5:4'
      imageSize: '2K',      // Options: '1K', '2K', '4K' (must be uppercase)
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

## Multi-Turn Conversational Editing

Use chat sessions to iteratively edit images:

```javascript
const ai = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });

const chat = ai.chats.create({
  model: "gemini-3-pro-image-preview",
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

## Using Reference Images (Up to 14)

Mix reference images for the final output:
- Up to 6 object images (high-fidelity inclusion)
- Up to 5 human images (character consistency)

```javascript
const contents = [
  { text: 'An office group photo of these people making funny faces.' },
  { inlineData: { mimeType: "image/jpeg", data: base64Image1 } },
  { inlineData: { mimeType: "image/jpeg", data: base64Image2 } },
  { inlineData: { mimeType: "image/jpeg", data: base64Image3 } },
];

const response = await ai.models.generateContent({
  model: 'gemini-3-pro-image-preview',
  contents: contents,
  config: {
    responseModalities: ['TEXT', 'IMAGE'],
    imageConfig: { aspectRatio: '5:4', imageSize: '2K' },
  },
});
```

## Google Search Grounding

Generate images based on real-time data (weather, stocks, events):

```javascript
const response = await ai.models.generateContent({
  model: 'gemini-3-pro-image-preview',
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
| `imageSize` | `'1K'`, `'2K'`, `'4K'` | Must be uppercase! |
| `aspectRatio` | `'1:1'`, `'4:3'`, `'3:4'`, `'16:9'`, `'9:16'`, `'5:4'` | |
| `responseModalities` | `['TEXT', 'IMAGE']` | Required for image output |
| `tools` | `[{googleSearch: {}}]` | Enable real-time data grounding |

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
- Advanced text rendering: legible text for infographics, menus, diagrams
- Google Search grounding: real-time data visualization
- Thinking mode: reasoning through complex prompts
- Up to 14 reference images
- Multi-turn conversational editing

## Common Use Cases

1. **Infographics**: Create educational visuals with accurate text
2. **Product mockups**: Generate professional marketing assets
3. **Data visualization**: Charts and graphs from real-time data
4. **Character consistency**: Maintain same characters across images
5. **Style transfer**: Apply artistic styles to concepts
6. **Iterative refinement**: Edit and refine through conversation

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
- **Choose a font**: Describe typography style or name the font (e.g., "bold, white, sans-serif font" or "Century Gothic 12px")
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

| Spec | Nano Banana Pro (Gemini 3 Pro Image) | Nano Banana 2 (Gemini 3.1 Flash Image) |
|------|--------------------------------------|----------------------------------------|
| Input tokens | 65,536 max | 131,072 max |
| Output tokens | 32,768 max | 32,768 max |
| Resolutions | 1K, 2K, 4K | 0.5K, 1K, 2K, 4K |
| Aspect ratios | 1:1, 3:2, 2:3, 3:4, 4:3, 4:5, 5:4, 9:16, 16:9, 21:9 | All of Pro + 1:4, 4:1, 1:8, 8:1 |
| Reference images | Up to 14 | Up to 14 |
| Image formats | PNG, JPEG, WebP, HEIC, HEIF | PNG, JPEG, WebP, HEIC, HEIF |
| Knowledge cutoff | January 2025 | January 2025 |
| Live data | Google Search grounding | Google Search grounding |
| Safety | C2PA Content Credentials + SynthID watermark | C2PA Content Credentials + SynthID watermark |
