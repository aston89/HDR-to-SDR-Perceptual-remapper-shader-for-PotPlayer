# HDR to SDR Perceptual remapper shader (PotPlayer, MPC and similar other player who support the usage of pixer shader filters)
This shader is a **perceptual HDR to SDR remapper** designed to run inside compatible HLSL-based players for users who doesnt have an HDR monitor like IPS/VA/TN, without relying on HDR metadata (ST2084/ST2086/ST2094) or uselessy complex combination of external controls.

---

## Core goal
> **Make HDR content look natural, readable and well-balanced on SDR displays**, while preserving:
* color integrity
* scene luminance hierarchy
* smooth highlight rolloff
* artifact-free output (no banding, no harsh clipping)

All of this without requiring manual tuning via sliders or toggles.

---

## Purpose
When HDR content is displayed on SDR screens, common issues include:
* clipped highlights (blown-out whites)
* compressed or muddy midtones
* unstable or oversaturated colors
* reliance on manual tone mapping settings

This shader takes a different approach:
> Instead of technically converting HDR → SDR, it **reinterprets luminance distribution perceptually**, producing a visually coherent result.

---

## How It Works
The shader operates per-frame, with **no HDR metadata and no temporal memory**:

### 1. Linearization (Gamma Handling)
Input RGB values are converted to linear space:
* enables correct luminance operations
* avoids non-linear distortion during tone mapping

### 2. Local Luminance Estimation
For each pixel:
* luminance is computed (Rec.709)
* neighboring pixels are sampled

This produces:
> **localY (local luminance estimate)**, used to guide adaptive behavior

### 3. Adaptive Exposure
Exposure is dynamically adjusted based on scene content:
* darker areas are slightly boosted
* brighter areas are slightly compressed

This **mimics** a simplified form of:
> perceptual adaptation / pseudo dynamic tone mapping

### 4. Filmic Tone Mapping
A modified ACES-like curve is applied:
* compresses highlights smoothly
* avoids hard clipping
* preserves detail in bright regions

this is the core of HDR to SDR perceptual transformation

### 5. Luminance / Chrominance Separation
After tone mapping:
* luminance drives contrast
* color is processed separately

This prevents:
* hue shifts
* unnatural saturation artifacts

### 6. Adaptive Saturation
Saturation is modulated based on brightness:
* preserved in midtones
* slightly reduced in highlights

prevents “neon” colors and improves realism

### 7. Highlight Rolloff
Bright regions are gently desaturated and compressed:
* maintains detail
* avoids harsh clipping

### 8. Dithering
A subtle dithering step is applied:
* reduces banding
* improves gradient smoothness
* especially useful in 8-bit pipelines

### 9. Output Conversion
Final values are converted back to display gamma (~2.2):
* ready for SDR rendering
* compatible with most displays

---

## Design Philosophy
This shader is **perceptual, not technical**.

It does NOT attempt to:
* reconstruct original HDR intent exactly
* use metadata-driven tone mapping

Instead:
> It produces a visually coherent SDR image based on human perception principles.

This makes it:
* robust in “blind” pipelines
* stable across different content
* free from manual configuration overhead

---

## Important technical notes !

### 1 Full Range Pipeline
This shader assumes:
* input is already **full range (0–255)**

If applied to limited range (16–235):
* blacks may appear lifted
* a prior range remap is required

### 2 Pipeline Placement
Best results when applied:
* after color conversion (in potplayer, shaders operate only after yuv/rgb conversion wich is ready for renderer)
* before any aggressive post-processing (pre-resize)

### 3 Not a True HDR Renderer
This shader does NOT implement:
* ST2084 (PQ decoding)
* ST2086 metadata
* ST2094 dynamic metadata

it's a **perceptual approximation** !

### 4 Behavior with already SDR Content
If used on On SDR input:
* may slightly enhance contrast and clarity
* may alter original artistic intent
* not the primary use case and not recommended.

### 5 Display Dependency
Works best on:
* SDR displays (gamma ~2.2) 
* moderate brightness setups
* well-calibrated IPS / VA / TN panels

May require adjustments on:
* very bright displays
* high-contrast panels

### 6 PotPlayer Limitations
While PotPlayer provides built-in HDR handling options (ST2084/ST2086/ST2094-based processing, inside configuration/video/pixel-shader), these methods are generally **conventional and parameter-driven rather than perceptually adaptive**.

In practice, this means:
* they often rely on static assumptions about display capabilities
* they may partially depend on metadata that is not always present or reliable
* they expose sliders (nits) that require **manual tuning by eye**

As a result, achieving a balanced image typically involves trial-and-error, and outcomes can vary significantly depending on content and display characteristics.

In particular, dynamic approaches such as **ST2094-like processing** (when available) **are not implemented in a fully accurate or scene-aware manner within the player**. Instead of true per-scene or per-frame adaptation, they may behave closer to **global remapping functions with limited responsiveness**, leading to:
* overly dark or overly bright scenes
* loss of detail in shadows or highlights
* inconsistent color and contrast behavior

This shader aims to solve this problem without the need for constant manual adjustment.

Before using this shader : 
> open settings in potplayer and go into video, then inside pixel-shader tab deselect the two checkboxes correlated to STMPE functions to avoid multiple useless and destructive remapping.

### 7  Usage in other players like MPC and forks
If you want to use this shader in other players like MPC, kmplayer, mpc mods or forks like mpc-hc, check in their pixel shader folders wich extensions are they using, for mpc you have to swap the .txt extension in .hlsl and it should work.
To use this shader in potplayer, go into his program folder and put the .txt file inside the px-shader folder (C:\Program Files\PotPlayer\PxShader)

---

## Expected Results
When properly used:
* deep but readable blacks
* clear and natural midtones
* controlled highlights
* rich but not oversaturated colors
* overall “filmic” appearance

This shader is a practical balance between:
* visual quality
* simplicity
* pipeline independence

It is not a technically perfect HDR conversion, but:
> a stable and perceptually coherent reinterpretation of HDR content for SDR environments.

And in real-world usage, that often delivers the best results.
