---
name: apple-tahoe-liquid-glass
description: Implements Apple-style liquid glass refracting buttons and UI components using WebP normal displacement maps, SVG filters, a 10-layer specular reflection shadow stack, and interactive click transitions & hold gestures.
---

# Apple Tahoe Liquid Glass Component Skill

Use this skill to implement highly realistic, refracting liquid glass buttons, toggles, and panels that interact with underlying background elements and images.

## Core Architectural Rules

### 1. SVG Displacement Refraction (Clear & See-Through)
- Define a bounding box filter with `primitiveUnits="objectBoundingBox"`.
- Use a WebP normal displacement map (base64 encoded) loaded inside `<feImage>` with `preserveAspectRatio="none"`.
- **ZERO FROSTINESS REFRACTION**: Direct coordinate displacement should be performed on the sharp background to simulate light bending without frosty blur. Feed `in="SourceGraphic"` directly into the displacement map filter:
  ```xml
  <feDisplacementMap 
    id="disp" 
    in="SourceGraphic" 
    in2="map" 
    scale="0.05" 
    xChannelSelector="R" 
    yChannelSelector="G" 
  />
  ```
- Keep the `scale` between `0.04` and `0.06` for clean, high-fidelity refraction without pixelation or visual glitches.

### 2. CSS Backdrop Filter Configuration
- **CRITICAL**: The CSS `backdrop-filter` property must reference the custom SVG filter without any blur token to preserve transparency and prevent frostiness (e.g., `backdrop-filter: url(#filter-id) saturate(150%);`).
- Controlling Transparency: The backing color opacity should remain extremely low (e.g., `1.5%` opacity in light mode, `6%` opacity in dark mode) to act as a crystal clear lens layer.

### 3. Pill capsule roundness
- For Apple-style navigation widgets, buttons, and active tabs, use a fully circular capsule border-radius:
  - Container: `border-radius: 9999px;`
  - Active Lens: `border-radius: 9999px;`
- This ensures elegant rounded aesthetics, especially in desktop layouts.

### 4. Isolated Lens Layer (Preventing Text Ghosting)
- To prevent text or icons from distorting or ghosting when refracted, the glass lens must be placed in a separate layer (`-z-10` or `position: absolute`) that contains no child content.
- The button text/label floats on top, completely clean and crisp.

### 5. Specular Box Shadow Stack
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

### 6. Interactive Gestures and Snapping transitions
- **Hold Gestures**: During active drag or press events, apply a slight scale squish (e.g., `transform: scale(0.94, 0.94)`) and a subtle fluid pulse animation to simulate liquid tension.
- **Click Transitions**: When switching options, trigger a temporary transitioning state that stretches the glass indicator along the movement axis and snaps it back (e.g. keyframes animating scale from `1` to `1.15` and back).
