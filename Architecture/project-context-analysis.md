# Project Context Analysis

## Requirements Overview

**Functional Requirements:**

The IGCSE Animation System transforms exam questions into educational animations through a sophisticated AI-orchestrated pipeline:

1. **Multi-Prompt Orchestration Architecture**
   - Master Orchestrator coordinates 6 specialized AI agents
   - Question Assessment (suitability scoring, visualization feasibility)
   - Animation Design (pedagogical planning, narrative structure)
   - Code Generation (Manim scene creation, library integration)
   - Quality Validation (compliance checks, accuracy verification)
   - Review Interface (human oversight and approval workflows)

2. **Animation Generation Pipeline**
   - Manim-based scene construction with strict brand compliance
   - Multi-library integration: py3Dmol (chemistry molecules), VPython (3D physics), Matplotlib (graphs)
   - Modular code architecture with helper functions and configuration management
   - Type-hinted, documented code following Black/Flake8/MyPy standards

3. **Multi-Format Rendering System**
   - Platform-specific video outputs: Square (1:1), Vertical (9:16), Portrait
   - Automated SRT subtitle generation synchronized to animations
   - File naming conventions for distribution workflow
   - Quality settings: 1080p, 60fps for production

4. **Quality Assurance Automation**
   - Automated validation checks across workflow stages
   - Difficulty assessment scoring system
   - Fallback decision trees for edge cases
   - Academic accuracy verification against IGCSE specifications

5. **Human Review Interface**
   - Approval workflow for AI-generated content
   - Feedback loop integration for continuous improvement
   - Multi-part question handling with human guidance escalation

**Non-Functional Requirements:**

1. **Academic Accuracy (Critical):** 100% alignment with IGCSE exam board specifications and marking schemes
2. **Brand Identity Compliance:** Strict adherence to "Sophisticated Precision" aesthetic
   - Color palette: #000000 (black), #FFFFFF (white), #FFFF00 (yellow), #006600 (green)
   - Monospaced typography for mathematical precision
   - Watermark and logo placement standards
3. **Production Performance:** Target rendering time < 10 minutes per question for scalability
4. **Code Quality Standards:** Black formatting, Flake8 linting, MyPy type checking enforced
5. **ESL Accessibility:** Simple sentence structure (15-20 words max), visual support for all explanations
6. **Social Media Engagement:** High completion rates, professional quality distinguishing content from competitors

**Scale & Complexity:**

- **Primary domain:** AI Orchestration + Generative Code + Animation Rendering (Multi-domain)
- **Complexity level:** **Enterprise**
- **Estimated architectural components:**
  - Prompt Management System (6 specialized prompts)
  - Workflow State Machine (6-stage pipeline)
  - Code Generation Engine (Manim + 3 external libraries)
  - Rendering Pipeline (multi-format output)
  - Quality Assurance Framework (validation + scoring)
  - Human Review Interface
  - Asset Management (questions, mark schemes, outputs)

## Technical Constraints & Dependencies

**Hard Constraints:**
- Python-based implementation (Manim requirement)
- Manim Community Edition as primary animation framework
- Brand visual identity system (non-negotiable color/typography standards)
- IGCSE curriculum alignment (external standard)
- Platform-specific video format requirements (TikTok/Facebook specifications)

**Library Dependencies:**
- Manim (core animation engine)
- py3Dmol + RDKit (organic chemistry 3D molecular visualization)
- VPython (3D physics simulations with collision detection)
- Matplotlib (data visualization with brand styling)
- NumPy/SciPy (mathematical computations)
- PIL/Pillow (image frame manipulation)

**Integration Challenges:**
- Coordinating multiple specialized Python libraries with different rendering paradigms
- Converting 3D library outputs (py3Dmol, VPython) into Manim-compatible 2D frame sequences
- Managing render pipelines across different output formats from single source
- Maintaining brand consistency across multiple visualization libraries

## Cross-Cutting Concerns Identified

1. **Multi-LLM Coordination:**
   - State management across 6 specialized prompts
   - Context passing between orchestration stages
   - Error propagation and recovery across prompt boundaries
   - Prompt versioning and consistency

2. **Quality Assurance Throughout Pipeline:**
   - Academic accuracy validation at every stage
   - Brand compliance checks automated where possible
   - Code quality enforcement (linting, type checking)
   - Output validation before human review

3. **Error Handling & Fallback Strategies:**
   - Visualization difficulty scoring triggers fallback decisions
   - Multi-part question routing to human guidance
   - Code generation retry logic with corrections
   - Graceful degradation for unsupported question types

4. **Performance Optimization:**
   - Rendering efficiency (<10 min target)
   - Caching complex computations
   - Batch operations for visual elements
   - Memory management for large datasets

5. **Human-in-the-Loop Integration:**
   - Review interface for approval workflows
   - Feedback loop for continuous improvement
   - Escalation triggers for edge cases
   - Progress tracking and workflow visibility

6. **Code Generation Consistency:**
   - Ensuring AI-generated Manim code follows standards
   - Template-based code structure enforcement
   - Configuration-driven customization (no magic numbers)
   - Comprehensive documentation generation

7. **Brand Compliance Automation:**
   - Color palette enforcement at code generation
   - Typography standards validation
   - Watermark/logo placement verification
   - Visual hierarchy system adherence
