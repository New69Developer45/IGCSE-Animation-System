# 3. Design Philosophy & Constraints

## Core Design Principles

### 3.1 "Sophisticated Precision" Aesthetic
The visual identity must convey:
- **Academic Authority:** Professional, university-level presentation quality
- **Technical Precision:** Mathematical/scientific accuracy in all visualizations
- **Modern Sophistication:** Contemporary design that avoids "clip art" or "cartoonish" aesthetics
- **Visual Calm:** Deliberate pacing, uncluttered layouts, intentional white space (or dark space)

### 3.2 Brand Identity System

**Approved Brand Colors:**
```python
BRAND_BLACK = "#000000"    # Primary background for most content
BRAND_WHITE = "#FFFFFF"    # Primary text and diagrams
BRAND_YELLOW = "#FFFF00"   # Highlights, accents, key information
BRAND_GREEN = "#006600"    # Correct answers, success states, growth
```

**Color Usage Guidelines:**
- **Background:** Primarily `BRAND_BLACK` for the "blackboard 2.0" effect
- **Text:** `BRAND_WHITE` for maximum contrast and readability
- **Emphasis:** `BRAND_YELLOW` for highlighting critical information, key terms, examiner tips
- **Validation:** `BRAND_GREEN` for correct answers, checkmarks, success indicators
- **Avoid:** Never use red, blue, or other colors outside the brand palette
- **Opacity:** Use opacity variations (0.3-0.7) for de-emphasizing elements, not color changes

**Typography:**
- **Primary Font:** Monospaced fonts (Consolas, Courier New, Monaco) for precision feel
- **Mathematical Notation:** LaTeX/TeX rendering via Manim's Tex and MathTex classes
- **Hierarchy:** Font size differences (not color) to establish information hierarchy
  - Headers: 32-38px
  - Body text: 20-24px
  - Annotations: 16-18px
  - Watermark: 14-16px

**Logo Placement:**
- Watermark: Top right corner at 50% opacity
- Outro: Full opacity, centered
- Size: Logo should not exceed 15% of frame width

### 3.3 Instructional Design Constraints

**Language Accessibility (ESL Considerations):**
- Use simple, direct sentence structures
- Avoid idioms, colloquialisms, or culturally specific references
- Define technical terms on first use
- Use visual aids to support verbal explanations
- Limit text on screen to 15-20 words maximum per frame
- Provide 2-3 seconds minimum reading time per text element

**Cognitive Load Management:**
- **Segmenting:** Break complex problems into 3-5 discrete steps
- **Pre-training:** Introduce unfamiliar concepts before applying them
- **Signaling:** Use yellow highlights to direct attention to relevant information
- **Coherence:** Remove extraneous information; every visual element must serve the explanation

**Common Misconception Integration:**
- Identify 1-2 common errors students make on each question type
- Briefly show why the misconception fails (10-15 seconds max)
- Use contrasting visual treatment (e.g., fading out incorrect approach)
- Language: "Students often think... but this fails because..."

### 3.4 Exam Authenticity Standards

**Question Presentation:**
- Replicate IGCSE paper formatting as closely as possible
- Maintain original question numbering if provided
- Preserve diagram proportions and spatial relationships
- Use identical notation conventions (e.g., vector notation, chemical equations)

**Mark Scheme Alignment:**
- Explanations must match official IGCSE mark scheme language where appropriate
- Highlight "keywords" that examiners look for in student responses
- Show the logical structure examiners expect in answers

---
