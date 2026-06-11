---
name: apple-tahoe-liquid-glass
description: Implements Apple-style liquid glass refracting buttons and UI components using WebP normal displacement maps, SVG filters, a 10-layer specular reflection shadow stack, and interactive click transitions & hold gestures.
---

# Apple Tahoe Liquid Glass Component Skill

Use this skill to implement highly realistic, refracting liquid glass buttons, toggles, and panels that interact with underlying background elements and images.

## Core Architectural Rules

### 1. SVG Displacement Refraction
- Define a bounding box filter with `primitiveUnits="objectBoundingBox"`.
- Use a WebP normal displacement map (base64 encoded) loaded inside `<feImage>` with `preserveAspectRatio="none"`.
- **CRITICAL FOR CHROMIUM REFRACTION**: You MUST include a `<feGaussianBlur in="SourceGraphic" stdDeviation="0.01" result="blur" />` to trigger the browser's backdrop copy pass.
- Chain the `<feDisplacementMap>` to read from the blurred result:
  ```xml
  <feGaussianBlur in="SourceGraphic" stdDeviation="0.01" result="blur" />
  <feDisplacementMap 
    id="disp" 
    in="blur" 
    in2="map" 
    scale="0.5" 
    xChannelSelector="R" 
    yChannelSelector="G" 
  />
  ```

### 2. CSS Backdrop Filter Configuration
- **CRITICAL**: The CSS `backdrop-filter` property must prefix the custom SVG filter with a standard blur to activate refraction (e.g. `backdrop-filter: blur(8px) url(#filter-id) saturate(150%);`).
- Controlling Frostiness: Frostiness is controlled strictly by the opacity of the glass backing color, not the filter. To keep the glass transparent and see-through, keep the backing color opacity extremely low (e.g., 1.5% in light mode, 6% in dark mode).

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

### 5. Interactive Gestures and Snapping transitions
- **Hold Gestures**: During active drag or press events, apply a slight scale squish (e.g., `transform: scale(0.94, 0.94)`) and a subtle fluid pulse animation to simulate liquid tension.
- **Click Transitions**: When switching options, trigger a temporary transitioning state that stretches the glass indicator along the movement axis and snaps it back (e.g. keyframes animating scale from `1` to `1.15` and back).
