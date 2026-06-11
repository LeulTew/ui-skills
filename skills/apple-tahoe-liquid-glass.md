---
name: apple-tahoe-liquid-glass
description: Implements Apple-style liquid glass refracting buttons and UI components using WebP normal displacement maps, SVG filters, a 10-layer specular reflection shadow stack, chromatic aberration rainbow filters, and interactive click transitions & hold gestures.
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
    scale="0.32" 
    xChannelSelector="R" 
    yChannelSelector="G" 
  />
  ```
- **Scale Optimization**: Adjust the `scale` based on the bounding box size of the element:
  - For small interactive elements (e.g., buttons, navigation switchers, active slider pills), keep the `scale` between `0.30` and `0.35` for highly active, visible refraction.
  - For large containers and layout cards, keep the `scale` between `0.08` and `0.12` to prevent massive distortion of child text content while retaining beautiful edge refraction.

### 2. Chromatic Aberration (Rainbow Refraction) on Active Drag
- To achieve a realistic light splitting (rainbow refraction) when interactive lenses are in active motion or dragged:
  1. Define a secondary filter separating the channels: Red, Green, and Blue.
  2. Displace each color channel with a slightly different scale (e.g. Red scale `0.45`, Green scale `0.32`, Blue scale `0.19`).
  3. Keep only the respective color channel for each using `<feColorMatrix>`.
  4. Recombine them back using additive blending via `<feBlend mode="screen">`.
- This ensures that flat areas remain sharp and aligned, while the normal map borders separate into a natural rainbow color fringe during active gesture movement:
  ```xml
  <filter id="displacement-rainbow" primitiveUnits="objectBoundingBox">
    <feImage href={WEBP_MAP} preserveAspectRatio="none" result="map" />
    <feDisplacementMap in="SourceGraphic" in2="map" scale="0.45" xChannelSelector="R" yChannelSelector="G" result="red_displaced" />
    <feColorMatrix in="red_displaced" type="matrix" values="1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0" result="red_only" />
    
    <feDisplacementMap in="SourceGraphic" in2="map" scale="0.32" xChannelSelector="R" yChannelSelector="G" result="green_displaced" />
    <feColorMatrix in="green_displaced" type="matrix" values="0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 1 0" result="green_only" />
    
    <feDisplacementMap in="SourceGraphic" in2="map" scale="0.19" xChannelSelector="R" yChannelSelector="G" result="blue_displaced" />
    <feColorMatrix in="blue_displaced" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 1 0" result="blue_only" />
    
    <feBlend in="red_only" in2="green_only" mode="screen" result="rg" />
    <feBlend in="rg" in2="blue_only" mode="screen" />
  </filter>
  ```

### 3. CSS Backdrop Filter Configuration
- **CRITICAL**: The CSS `backdrop-filter` property must reference the custom SVG filter without any blur token to preserve transparency and prevent frostiness (e.g., `backdrop-filter: url(#filter-id) saturate(150%);`).
- Controlling Transparency: The backing color opacity should remain extremely low (e.g., `1.5%` opacity in light mode, `6%` opacity in dark mode) to act as a crystal clear lens layer.

### 4. Pill capsule roundness
- For Apple-style navigation widgets, buttons, and active tabs, use a fully circular capsule border-radius:
  - Container: `border-radius: 9999px;`
  - Active Lens: `border-radius: 9999px;`
- This ensures elegant rounded aesthetics, especially in desktop layouts.

### 5. Isolated Lens Layer (Preventing Text Ghosting)
- To prevent text or icons from distorting or ghosting when refracted, the glass lens must be placed in a separate layer (`-z-10` or `position: absolute`) that contains no child content.
- The button text/label floats on top, completely clean and crisp.

### 6. Specular Box Shadow Stack
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

### 7. Interactive Gestures and Snapping transitions
- **Hold & Move Expansion (Apple-style Ballooning)**: During active touch, hold, or drag-glide events, expand the active glass lens outward especially vertically to simulate water tension and bubble ballooning (e.g., `transform: scale(1.04, 1.20)`). Under active slide movement, dynamically swap the standard refraction filter with the chromatic aberration filter to project the rainbow effect.
- **Click Transitions**: When switching options, trigger a temporary transitioning state that stretches the glass indicator along the movement axis and snaps it back (e.g. keyframes animating scale from `1` to `1.15` and back).
