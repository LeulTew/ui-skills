---
name: apple-tahoe-liquid-glass
description: Implements Apple-style liquid glass refracting buttons and UI components using WebP normal displacement maps, SVG filters, and a 10-layer specular reflection shadow stack.
---

# Apple Tahoe Liquid Glass Component Skill

Use this skill to implement highly realistic, refracting liquid glass buttons and panels that interact with underlying background elements and images.

## Core Architectural Rules

### 1. SVG Displacement Refraction (Chrome/Edge Support)
- Define a bounding box filter with `primitiveUnits="objectBoundingBox"`.
- Use a WebP normal displacement map (base64 encoded) loaded inside `<feImage>` with `preserveAspectRatio="none"`.
- Chain it to `<feGaussianBlur>` (for pre-refraction smoothing, set to `stdDeviation="0.01"`) and `<feDisplacementMap>` with `scale="0.5"`, `xChannelSelector="R"`, and `yChannelSelector="G"`.

### 2. Backdrop Filter Blur (Safari Fallback)
- For browsers that do not fully support custom SVG displacement filters, provide a fallback using `backdrop-filter: blur(8px) saturate(150%)`.

### 3. Isolated Lens Layer (Preventing Text Ghosting)
- To prevent text or icons from distorting or ghosting when refracted, the glass lens must be placed in a separate layer (`-z-10` or `position: absolute`) that contains no child content.
- The button text/label floats on top, completely clean and crisp.

### 4. Specular Box Shadow Stack
Create realistic glass thickness and reflections using a 10-layered shadow stack:
```css
box-shadow: 
  inset 0 0 0 1px rgba(255, 255, 255, 0.1), /* Rim outline */
  inset 1.8px 3px 0px -2px rgba(255, 255, 255, 0.9), /* Specular light source reflection */
  inset -2px -2px 0px -2px rgba(255, 255, 255, 0.8), /* Bottom rim bounce */
  inset -3px -8px 1px -6px rgba(255, 255, 255, 0.6), /* Bottom specular glow */
  inset -0.3px -1px 4px 0px rgba(0, 0, 0, 0.12),
  inset -1.5px 2.5px 0px -2px rgba(0, 0, 0, 0.20),
  inset 0px 3px 4px -2px rgba(0, 0, 0, 0.20),
  inset 2px -6.5px 1px -4px rgba(0, 0, 0, 0.10),
  0px 1px 5px 0px rgba(0, 0, 0, 0.10),
  0px 6px 16px 0px rgba(0, 0, 0, 0.08); /* Heavy glass shadow depth */
```

### 5. High-Transparency Backing
- To showcase real refraction rather than a frosted overlay, the background color must be highly translucent. Use `rgba(255, 255, 255, 0.05)` (light mode) and `rgba(15, 16, 20, 0.15)` (dark mode) to keep the glass see-through.
