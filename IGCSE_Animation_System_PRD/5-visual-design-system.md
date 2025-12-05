# 5. Visual Design System

## 5.1 Frame Composition

**Standard Layout Template:**
```
┌─────────────────────────────────┐
│  [WATERMARK: Top Right]         │
│                                 │
│  [TITLE/QUESTION: Top 20%]      │
│                                 │
│  [MAIN VISUAL: Center 50%]      │
│                                 │
│  [ANNOTATIONS: Bottom 30%]      │
│                                 │
└─────────────────────────────────┘
```

**Aspect Ratio Configurations:**

1. **Square (1:1) - 1080x1080px**
   - Primary for Facebook feed
   - Balanced composition, no element cropping
   - Safe area: Inner 90% of frame

2. **Vertical (9:16) - 1080x1920px**
   - Primary for TikTok, Facebook Reels
   - Top 25%: Question text
   - Middle 50%: Main visual
   - Bottom 25%: Answer/explanation

3. **Portrait (4:5) - 1080x1350px**
   - Secondary for Facebook/Instagram feed
   - Compromise between square and vertical
   - Similar layout to 1:1 with more vertical space

## 5.2 Animation Timing Standards

**Pacing Guidelines:**
- **Intro/Branding:** 1-2 seconds (fade in/out)
- **Question Display:** 4-8 seconds (allow reading time)
- **Each Explanation Step:** 8-15 seconds
- **Answer Reveal:** 3-5 seconds
- **Outro/CTA:** 2-3 seconds

**Transition Standards:**
- **Between sections:** 0.5-0.8 seconds
- **Element appearance:** FadeIn over 0.5-1.0 seconds
- **Element emphasis:** Scale(1.0 → 1.15) over 0.3 seconds
- **Element removal:** FadeOut over 0.3-0.5 seconds

**Animation Principles (Disney's 12 Principles Adapted):**
1. **Ease In/Out:** Use rate functions (smooth_start, smooth_end)
2. **Anticipation:** Brief pause before major transitions
3. **Staging:** Direct attention to one element at a time
4. **Follow Through:** Elements don't stop abruptly
5. **Timing:** Critical information gets more screen time

## 5.3 Text Rendering Guidelines

**Readability Standards:**
- Minimum font size: 18px (for mobile viewing)
- Line height: 1.3-1.5x font size
- Maximum line length: 50 characters
- Text-to-background contrast ratio: Minimum 7:1 (WCAG AAA)

**Mathematical Notation:**
```python
# Always use MathTex for equations, Tex for mixed text
equation = MathTex(r"v^2 = u^2 + 2as", color=BRAND_WHITE)

# Units in upright font, variables in italic
formula = MathTex(r"F = ma", r"\text{ (in N)}", color=BRAND_WHITE)

# Chemical equations
reaction = MathTex(r"\ce{2H2 + O2 -> 2H2O}", color=BRAND_WHITE)
```

## 5.4 Visual Hierarchy System

**Priority Levels:**
1. **Primary (Attention Grabber):** Yellow highlight, largest size, animated entry
2. **Secondary (Context):** White, medium size, static or subtle animation
3. **Tertiary (Supporting):** White at 70% opacity, smallest size, background element

**Example Hierarchy:**
```
PRIMARY:     "What is the velocity at t = 5s?" (Yellow, 28px, Write animation)
SECONDARY:   Graph axes and trajectory (White, standard weight, Create animation)
TERTIARY:    Grid lines and minor labels (White 70%, thin weight, static)
```

---
