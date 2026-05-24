---
name: CreateImagePrompt
description: Generate a header image prompt for a blog post in this wiki. Use when the user asks for an "image prompt", "header image prompt", "cover image prompt", or invokes /CreateImagePrompt while a Blog/*.md or root-level tutorial is the focus of the conversation.
---

# Create Image Prompt

Generate a prompt for an external text-to-image generator (Midjourney, DALL-E, Stable Diffusion, etc.) that produces a header image for a blog post in this Inventor Automation Wiki.

## Inputs

- If the user passes a filename or path as an argument, use that post.
- Otherwise, use the file currently open in the IDE (typically reported via a system reminder). If multiple candidates exist, ask which post.

## What the prompt should describe

A wide horizontal header illustration that:

- Conveys the *subject* of the post (what the rule, add-in, or technique does) in one immediately readable scene.
- Uses an Autodesk Inventor flavour: technical drawings, exploded assemblies, gears, brackets, fasteners, isometric views, dimension lines, callouts. Only include CAD imagery that is relevant to the post.
- May include a faint background overlay suggesting code, automation logic, or schematic patterns when the post is about scripting or automation — kept soft and non-dominant.
- Reads cleanly at thumbnail size. One focal idea, not a collage.

## Style defaults

Unless the post strongly suggests otherwise, default to:

- Modern technical-illustration style with subtle depth and soft shadows. Professional, minimal, slightly glossy UI feel with clean high-contrast lighting.
- Restrained palette built strictly around the wiki's tone colours: deep navy `#0B0B3B` as background, warm orange `#d84719` for highlights, balloons, and key accents. White and light gray only as supporting neutral tones.
- Image size 1200x630 pixels.
- Autodesk and Autodesk Inventor branding elements are allowed. DO NOT include any other branding, and do not include the Inventor UI or any software screenshots. Avoid literal depictions of the software interface; instead, focus on the underlying concepts and techniques.

## Output format

Return a single fenced code block containing the prompt, ready to paste into an image generator. Structure the prompt in labeled sections, in this order:

1. **Opening scene** — one or two sentences describing the central scene and what it conveys (no label).
2. **Style:** illustration style, level of detail, finish (vector, glossy, depth, shadows).
3. **Include:** optional — any secondary motifs such as background overlays of code, schematics, or automation logic. Only include when relevant.
4. **Color palette:** exact hex values and how they are used.
5. **Lighting:** lighting character (contrast, glossiness, softness).
6. **Composition:** layout direction, focal placement, balance, suitability as a blog header.
7. **Mood:** 3–5 adjectives capturing the feeling (e.g. efficient, engineered, precise, reliable).
8. **Size:** 1200x630.

Aim for roughly 150–220 words total. Below the code block, add one short line describing the chosen focal idea or metaphor so the user can ask for a different direction if they want.

Do **not** create or modify files unless the user asks you to save the prompt somewhere. Do **not** attempt to call an image generator; only produce the text prompt.

## Variations to offer (only on request)

If the user asks for alternatives, propose at most three distinct directions, each with its own prompt:

- Literal: a clean depiction of the result the rule produces.
- Metaphorical: the visual pun direction.
- Before/after: a side-by-side contrasting the problem the post solves with the solution.

