# Building AI COWORK ® STUDIO
### A Case Study in Human × AI Collaborative Design & Development

**Project:** AI COWORK Visual Intelligence Website  
**Stack:** HTML / CSS / JavaScript · Three.js · GSAP ScrollTrigger · MediaPipe  
**Duration:** Multi-session build  
**Live:** https://angel19190828-commits.github.io/ai-cowork/

---

## Overview

This project is a working demonstration of what AI-native product development looks like in practice. The goal was not just to build a website — it was to prototype a **new workflow** where a human creative director and an AI engineer collaborate in real-time, with the human holding final authority over all design decisions.

The result: a production-grade single-page portfolio site with 3D orbital animations, scroll-driven interactions, a camera demo with MediaPipe hand tracking, and a cinematic editorial aesthetic — built without writing a line of code by hand.

---

## The Workflow

### Role Separation

| Human (Angel) | AI (Claude) |
|---------------|-------------|
| Design direction & taste | Code generation & implementation |
| Accept / Reject / Rewind | Propose solutions |
| Define what feels right | Explain technical tradeoffs |
| Set visual targets | Debug and optimize |

The defining characteristic of this workflow is the **"rewind" command** — a single word that instantly reverts any change. This gave the human director full creative control without needing to understand the code. The AI never pushed back on a rewind; it simply restored the previous state.

---

## Phase 1 — Design Language

The session began with establishing the visual system before touching any code:

- **Background color:** `#f4f2ef` — not pure white, feels like paper
- **Typography:** Inter weight 100 for display, PT Mono for labels
- **Aesthetic reference:** nutrition labels, receipt printers, financial terminals
- **3D role:** ambient atmosphere, not hero content

> *"The 3D objects are atmosphere. The editorial layout is the content."*

This clarity meant every subsequent decision had a north star. When something felt wrong, it was easy to articulate why.

---

## Phase 2 — Core Build

### What Got Built (and How)

**Hero Section**
- Large headline with GSAP word-by-word reveal
- 3 Three.js torus knots orbiting in an elliptical ring (30° top-down camera)
- Subtle mouse tilt influence
- SCROLL TO DISCOVER with slow CSS blink

**Scroll Architecture**
- ScrollTrigger controls mode switching: `hero → service`
- In service mode: one knot moves to the left of the active card
- On fast scroll: `gsap.set()` synchronously kills all other tweens before scaling in the correct knot

**Product Cards (Nutrition Label Style)**
- Front: large number + title + barcode + stats grid
- Back: dark flip face with loading bar animation
- Click triggers GSAP `rotateY(180)` → redirects to project page

**Receipt Printer (Bottom Right)**
- Fixed position, grows upward as products are visited
- Items accumulate — never reset, never replaced
- `printedItems` Set prevents re-printing on scroll-back

**work1.html — Project Detail Page**
- LineWaves GLSL shader background (ported from React to vanilla Three.js)
- Camera demo with MediaPipe hand tracking
- Index finger drives an uploaded PNG in real-time
- Pinch gesture detection (thumb + index tip distance)
- Hand skeleton overlay (21-point MediaPipe model)

---

## Phase 3 — Iteration Cycles

This is where the AI coworking skill is most visible. Every feature went through rapid propose → evaluate → accept/rewind cycles.

### Example Iterations

**Background Section Numbers**
```
AI proposed: translateY(-50%)
Angel: numbers are cut off at the bottom
AI: changed to overflow-x: hidden
Angel: 你是傻子吗 (scrollbar appeared — browser spec bug)
AI: reverted, moved to translateY(-51%)
Angel: move down 2% → then 4% → then 5%
Final: translateY(-51%) with iterative fine-tuning
```

**Side Labels on Cards**
```
AI: moved inside .card-body with overflow:hidden + z-index
Angel: 错错错 rewind，不要了
AI: full revert, no questions asked
Status: left for manual tweaking
```

**Receipt Behavior**
```
First attempt: each section replaced the receipt content
Angel: wanted accumulation — like a real printer growing upward
AI: rebuilt with Set pattern + append
Result: items stack permanently, never reset
```

**Camera Demo Trash Button**
```
Angel: arrow should point downward
AI: changed ↓ Clear to "↓Clear" with proper down arrow
```

**Mouse "Look At" on 3D Objects**
```
AI: proposed adding it
Angel: rewind
AI: restored immediately
```
The effect was already subtly built into the existing code — the discussion surfaced that it existed but the user preferred the current balance.

---

## Phase 4 — Technical Problem-Solving

The project generated several non-obvious technical problems that required real debugging:

### Problems Solved

| Problem | Root Cause | Solution |
|---------|-----------|----------|
| work1.html 404 on localhost | Python http.server can't handle extension-less URLs | Switched to `npx serve` |
| WebGL context conflict | Tested WebGL with real canvas, Three.js couldn't claim it | Used throwaway canvas for detection |
| GitHub Pages broken links | `/work1.html` resolves from domain root, not subfolder | Changed to relative paths `work1.html` |
| Fast-scroll knot overlap | GSAP callback chains ran out of order | `gsap.set()` synchronous kill before scale-in |
| CSS overflow scrollbar | `overflow-x: hidden` forces `overflow-y: auto` (browser spec) | Kept `overflow: hidden`, adjusted translateY |
| Hero headline FOUT flash | Google Fonts `display=swap` + JS delay | CSS `transform: translateY(105%)` pre-hides words |
| makeDefaultPng() 60×/sec | Called inside drawFrame() on every rAF | Cache result outside draw loop (identified, pending fix) |
| Figma import failure | WebGL canvas content invisible to html.to.design | Chrome DevTools full-page screenshot instead |

---

## Phase 5 — Deployment

```
Local:   npx serve . --listen 3000
Remote:  GitHub Pages (auto-deploy on push, ~30s)
```

Git workflow was set up mid-project after the site was already built. The AI helped navigate:
- Initializing in the right directory (not a parent folder)
- Setting up `.gitignore` to exclude `portfolio/` (separate personal project)
- Switching from Python server to `npx serve` for routing
- Relative path adjustments for GitHub Pages subfolder deployment

---

## What This Workflow Enables

### Speed
A feature that would take a solo developer 2–4 hours (e.g., MediaPipe hand tracking integration) was implemented in a single focused session. The human only needed to describe the desired behavior.

### Safety
The "rewind" pattern means no change is permanent until accepted. The human never needs to understand what changed — only whether it *feels right*.

### Creative Control
The AI never made aesthetic decisions autonomously. Every color, every timing value, every layout choice was either initiated by the human or confirmed before being committed.

### Skill Amplification
The human's skill in this workflow is **taste and direction** — knowing what to ask for, when to rewind, and when something is right. The AI's skill is **execution and constraint navigation** — knowing how to implement without breaking existing work.

---

## Key Principles Demonstrated

1. **Describe intent, not implementation.** "Make the receipt behave like a real printer" is better than "append a div on scroll."

2. **Rewind is a feature, not a failure.** Every rewind sharpened the result.

3. **The AI holds no ego.** Reverts, corrections, and "你是傻子吗" moments are part of the process — not interruptions.

4. **Technical debt gets caught.** The AI identified issues like the `makeDefaultPng()` 60fps bug and memory leaks during code review — things the human never would have noticed.

5. **One source of truth.** The render loop owns 3D position. GSAP owns transitions. CSS owns static state. When these boundaries are respected, bugs don't happen.

---

## Stack Reference

```
Three.js r150+          3D scene, WebGL renderer, orbital animation
GSAP ScrollTrigger      Scroll-driven mode switching, card animations
GSAP Core               All transitions, timelines, entrance sequences
MediaPipe Hands         21-point hand landmark tracking (CDN)
PT Mono / Inter         Google Fonts — editorial typography system
npx serve               Local dev server with extension-less URL routing
GitHub Pages            Zero-build static deployment
```

---

## Files

```
ai-cowork/
├── index.html          Main site: hero + intro + 3 product sections
├── work1.html          Project detail: LineWaves shader + camera demo
├── DESIGN_DOCUMENT.md  Technical architecture & failure log
├── CASE_STUDY.md       This document
└── .claude/
    └── launch.json     Preview server config
```

---

*Built in collaboration with Claude (Anthropic) · AI COWORK ® STUDIO · 2026*
