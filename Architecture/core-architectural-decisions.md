# Core Architectural Decisions

## Decision Priority Analysis

**Critical Decisions (Block Implementation):**
1. ✅ Document Ingestion Pipeline - Claude Vision + CLI review
2. ✅ LLM API Integration - LangChain with cost tracking
3. ✅ Data Storage - File-based with structured folders
4. ✅ Error Handling - Checkpoint-based recovery
5. ✅ Deployment - Local development initially

**Important Decisions (Shape Architecture):**
1. ✅ Batch Processing - 7/14/21 questions per batch
2. ✅ Review Interfaces - CLI tools for extraction & final approval
3. ✅ State Management - JSON files per question workflow

**Deferred Decisions (Post-MVP):**
1. Cloud deployment (when scaling needed)
2. Parallel processing (when volume increases)
3. Database migration from files to SQLite (when >100 questions)
4. GPU-accelerated rendering (if performance bottleneck)

## 1. Document Ingestion Pipeline

**Decision:** Claude 3.5 Sonnet Vision API with CLI validation

**Input Format:** Official IGCSE past papers as PDFs or screenshots

**Batch Sizes:** 7, 14, or 21 questions per batch

**Rationale:**
- Official IGCSE past papers have consistent format
- Claude Vision excels at educational content extraction
- Same API/account as animation generation prompts
- CLI review fits Python-centric workflow
- Multiple choice questions with correct answers pre-identified

**Workflow:**
```
inbox/batch/ → Claude Vision Extraction → extracted/ → CLI Review → questions/ → Animation Pipeline
```

**Components:**
- `src/igcse_animation/ingestion/extractor.py` - Claude Vision integration
- `src/igcse_animation/ingestion/reviewer.py` - CLI review tool
- `src/igcse_animation/ingestion/batch_manager.py` - Batch processing logic

**Extraction Process:**
1. User places 7/14/21 PDF files or screenshots in `data/inbox/batch_XXX/`
2. Extraction script calls Claude Vision API for each file
3. Claude extracts: subject, question text, diagram description, options A-D, correct answer
4. Results saved to `data/extracted/batch_XXX/*.json`
5. CLI review tool presents each extraction for validation
6. Approved extractions move to `data/questions/` for processing

**CLI Review Tool Interface:**
```bash
$ python -m igcse_animation.ingestion.review batch_001

Reviewing batch_001: 7 questions extracted

Question 1/7: PHY_2023_P1_Q15
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Subject: Physics
Question: A ball is thrown at 30° to the horizontal...
Options:
  A: 5.1 m
  B: 10.2 m  ← Correct
  C: 15.3 m
  D: 20.4 m
Diagram: Yes (projectile trajectory)

[E]dit | [A]pprove | [S]kip | [Q]uit:
```

**Directory Structure:**
```
data/
├── inbox/
│   ├── batch_001/                  # User drops PDFs here
│   │   ├── question_1.pdf
│   │   └── ... (7, 14, or 21 files)
│   └── batch_002/
├── extracted/
│   └── batch_001/
│       ├── question_1_extracted.json
│       ├── question_1_diagram.png  # Extracted diagrams if present
│       └── ...
└── questions/                      # Validated, ready for animation
    ├── PHY_2023_P1_Q15.json
    └── ...
```

## 2. LLM API Integration & Cost Management

**Decision:** LangChain ChatAnthropic with built-in retry and cost tracking

**Model:** Claude 3.5 Sonnet (claude-3-5-sonnet-20241022)

**Rationale:**
- Already using LangChain for PromptTemplates
- Built-in retry logic (3 attempts, exponential backoff)
- Automatic cost tracking via callbacks
- Standardized error handling across all 6 prompts
- Easy fallback to alternative models if needed

**API Call Volume per Question:**
- Document Extraction: 1 call (Claude Vision)
- Assessment Prompt: 1 call
- Design Prompt: 1 call
- Code Generation Prompt: 1 call
- Validation Prompt: 1 call
- Review Interface Prompt: 1 call
- **Total: 6 calls per question**

**Batch Processing Costs:**
- 21 questions × 6 calls = 126 API calls per batch
- Cost tracking essential for budget management

**Implementation:**
```python
from langchain_anthropic import ChatAnthropic
from langchain.callbacks import get_openai_callback

class PromptExecutor:
    def __init__(self):
        self.llm = ChatAnthropic(
            model="claude-3-5-sonnet-20241022",
            max_retries=3,
            timeout=60,
            max_tokens=4096
        )

    def execute_with_tracking(self, prompt: str) -> tuple[dict, float]:
        """Execute prompt and return response + cost"""
        with get_openai_callback() as cb:
            response = self.llm.invoke(prompt)
            return response, cb.total_cost
```

**Cost Tracking Strategy:**
- Log cost per question in workflow state
- Generate per-batch cost summaries
- Monthly cost reports for budgeting
- Alert if batch exceeds expected cost threshold

**Error Handling:**
- LangChain automatically retries on network errors (3 attempts)
- Exponential backoff between retries
- Timeout after 60 seconds per call
- Graceful failure with checkpoint preservation

## 3. Data Storage & State Management

**Decision:** File-based storage with structured directory hierarchy

**Rationale:**
- Simple, no database setup required
- Easy to inspect and debug (human-readable JSON)
- Version control friendly
- Appropriate for initial volumes (dozens of questions)
- Clear migration path to SQLite when volume increases (>100 questions)

**Complete Directory Structure:**
```
data/
├── inbox/                          # Input PDFs/screenshots
│   ├── batch_001/
│   │   ├── question_1.pdf
│   │   └── ... (7, 14, or 21 files)
│   └── batch_002/
├── extracted/                      # Claude Vision output (needs validation)
│   └── batch_001/
│       ├── question_1_extracted.json
│       ├── question_1_diagram.png  # Extracted diagrams
│       └── ...
├── questions/                      # Validated, ready for processing
│   ├── PHY_2023_P1_Q15.json
│   ├── CHEM_2023_P2_Q08.json
│   └── ...
├── workflows/                      # Workflow state per question
│   ├── PHY_2023_P1_Q15_state.json
│   └── ...
├── generated_code/                 # Manim code before rendering
│   ├── PHY_2023_P1_Q15.py
│   └── ...
├── outputs/                        # Rendered videos and subtitles
│   ├── PHY_2023_P1_Q15_square.mp4
│   ├── PHY_2023_P1_Q15_vertical.mp4
│   ├── PHY_2023_P1_Q15_portrait.mp4
│   ├── PHY_2023_P1_Q15.srt
│   └── ...
└── logs/                          # Error logs, performance metrics
    ├── batch_001_log.json
    └── ...
```

**Workflow State Schema:**
```python
@dataclass
class WorkflowState:
    question_id: str
    batch_id: str
    current_stage: WorkflowStage
    created_at: str
    updated_at: str

    # Checkpoint data (saved after each successful stage)
    assessment_result: dict | None = None
    design_spec: dict | None = None
    generated_code: str | None = None
    validation_results: dict | None = None
    rendered_files: list[str] | None = None
    review_status: str | None = None

    # Cost tracking
    total_cost: float = 0.0
    cost_breakdown: dict = field(default_factory=dict)

    # Error tracking
    error_log: list[str] = field(default_factory=list)
    retry_count: int = 0
```

**Migration Path:**
- When volume exceeds ~100 questions
- Migrate to SQLite for faster queries across questions
- Retain file storage for generated code and videos
- Simple Python migration script converts JSON to SQLite

## 4. Error Handling & Workflow Recovery

**Decision:** Checkpoint-based recovery with selective auto-retry

**Rationale:**
- Don't lose expensive API call progress on failure
- Enable human intervention at specific failed stages
- Automatic retry for transient network errors (via LangChain)
- Preserve all intermediate results for debugging and cost analysis

**Checkpoint Strategy:**
- Save workflow state JSON after each successful stage
- State includes all prior stage results
- Can resume from any checkpoint after fixing errors

**Error Handling by Type:**

| Error Type | Auto-Retry | Checkpoint | Action |
|------------|-----------|------------|---------|
| API Timeout | Yes (3×) | Yes | LangChain retries, checkpoint if all fail |
| Network Error | Yes (3×) | Yes | LangChain retries, checkpoint if all fail |
| Validation Failure | No | Yes | Flag for human review |
| Code Generation Error | No | Yes | Log error, manual prompt adjustment needed |
| Rendering Crash | No | Yes | Investigate Manim error, fix and retry |
| Assessment Rejection | No | Log only | Question unsuitable, skip to next |

**Resume Capability:**
```bash
# Check failed workflows
$ python -m igcse_animation.orchestrator.status

Failed Workflows:
- CHEM_2023_P2_Q08: Failed at GENERATION stage
- BIO_2023_P1_Q12: Failed at VALIDATION stage

# Resume specific workflow
$ python -m igcse_animation.orchestrator.resume CHEM_2023_P2_Q08

Loading checkpoint: GENERATION stage (Assessment & Design preserved)
Resuming from Code Generation...
✓ Generation complete
✓ Validation complete
✓ Rendering complete
✓ Review ready

Workflow complete!
```

**Benefits:**
- Save API costs (don't re-run successful expensive stages)
- Faster iteration when debugging prompts
- Clear audit trail of successes and failures
- Human can intervene at exact failure point

## 5. Deployment & Execution Environment

**Decision:** Local development initially, cloud migration when scaling

**Phase 1: Local Development (Current)**

**Platform:** Developer's laptop/desktop
**Environment:** Poetry virtual environment (Python 3.12+)
**Processing:** Sequential batch processing (one question at a time)
**Storage:** Local filesystem

**Rationale:**
- Simple setup, no cloud infrastructure costs
- Full control for debugging and prompt refinement
- Direct access to rendered videos for review
- Can run overnight for large batches
- Perfect for validating system end-to-end

**Performance Expectations:**
- Document extraction: ~30 seconds per question
- Workflow execution: ~5-10 minutes per question (6 LLM calls)
- Rendering: ~5-10 minutes per question (Manim rendering 3 formats)
- Total: ~10-20 minutes per question
- Batch of 21 questions: ~3.5-7 hours

**Daily Workflow:**
```bash
# Activate environment
$ cd igcse-animation-system
$ poetry shell

# Extract batch from PDFs
$ python -m igcse_animation.ingestion.extract batch_001
Extracting 14 questions... Done in 7 minutes

# Review extractions
$ python -m igcse_animation.ingestion.review batch_001
(Interactive CLI review)

# Run animation workflow
$ python -m igcse_animation.orchestrator.run batch_001
Processing 14 questions... (can run overnight)

# Review final animations
$ python -m igcse_animation.review.approve batch_001
(Interactive CLI review with video playback)
```

**Phase 2: Cloud VM (Future - When Scaling)**

**Migration Triggers:**
- Processing >50 questions per week regularly
- Local machine performance becomes bottleneck
- Need for always-on availability
- Want to process multiple batches in parallel

**Target Platform:** AWS EC2, DigitalOcean Droplet, or similar
**Instance Type:** GPU-enabled for faster Manim rendering (optional)
**Processing:** Parallel batch processing across multiple questions
**Storage:** Cloud storage for videos, synced to local for distribution

**Estimated Monthly Costs (Phase 2):**
- Basic CPU instance: ~$20-40/month
- GPU instance (if rendering slow): ~$100-200/month
- Storage: ~$5-10/month
- Claude API: Variable based on question volume

## 6. Human Review Interfaces

**Decision:** CLI tools for both extraction validation and final animation approval

**Rationale:**
- Consistent command-line workflow
- Fast keyboard-driven review process
- No web UI development complexity
- Fits Python-centric development environment
- Easy integration with automation scripts

**Extraction Review CLI:**
```bash
$ python -m igcse_animation.ingestion.review batch_001

Reviewing batch_001: 7 questions extracted

Question 1/7: PHY_2023_P1_Q15
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Subject: Physics
Question: A ball is thrown at 30° to the horizontal...
Options:
  A: 5.1 m
  B: 10.2 m  ← Correct
  C: 15.3 m
  D: 20.4 m
Diagram: Yes (projectile trajectory)

[E]dit | [A]pprove | [S]kip | [Q]uit: A

✓ Approved → questions/PHY_2023_P1_Q15.json

Question 2/7: ...
```

**Animation Approval CLI:**
```bash
$ python -m igcse_animation.review.approve batch_001

Reviewing batch_001: 7 animations rendered

Animation 1/7: PHY_2023_P1_Q15
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Formats: ✓ Square (1:1)  ✓ Vertical (9:16)  ✓ Portrait
Subtitles: ✓ SRT generated
Validation: ✓ All automated checks passed
Cost: $0.47

[P]lay videos | [A]pprove | [R]egenerate | [S]kip | [Q]uit: P

Opening videos in default player...
(Plays all 3 formats sequentially or in parallel)

[A]pprove | [R]egenerate | [S]kip: A

✓ Approved → Ready for TikTok/Facebook distribution

Animation 2/7: ...
```

**Review Interface Features:**
- Keyboard shortcuts for fast navigation
- Inline editing for corrections
- Batch approval option for trusted extractions
- Video playback integration with system default player
- Cost display per animation
- Validation results summary

## Decision Impact Analysis

**Implementation Sequence (Epic Order):**

1. **Project Setup & Foundation** (Epic 1)
   - Poetry project initialization
   - Directory structure creation
   - Configuration management (API keys, paths, constants)
   - Brand configuration (colors, fonts, layout standards)

2. **Document Ingestion Pipeline** (Epic 2)
   - Claude Vision integration for PDF/screenshot extraction
   - Batch manager for 7/14/21 question processing
   - CLI review tool for extraction validation
   - Question data schema and JSON structure

3. **Core Workflow Engine** (Epic 3)
   - State machine implementation (6 stages)
   - Checkpoint system with JSON persistence
   - Resume functionality for failed workflows
   - Error logging and status tracking

4. **LLM Prompt Integration** (Epic 4)
   - LangChain setup with ChatAnthropic
   - Cost tracking system with callbacks
   - 6 specialized prompts as LangChain PromptTemplates:
     - Assessment, Design, Generation, Validation, Review Interface, (Master Orchestrator)
   - Error handling and retry logic
   - Prompt versioning strategy

5. **Code Generation Engine** (Epic 5)
   - Manim code generator with templates
   - Library integration layer (py3Dmol, VPython, Matplotlib)
   - Brand compliance code injection (colors, fonts, watermark)
   - Code validation (syntax, imports, Manim structure)

6. **Rendering Pipeline** (Epic 6)
   - Multi-format rendering (square, vertical, portrait)
   - SRT subtitle generation synchronized to animations
   - Platform-specific optimizations (TikTok/Facebook specs)
   - Output file naming conventions
   - Brand compliance validation post-render

7. **Review & Distribution** (Epic 7)
   - Animation approval CLI with video playback
   - Batch approval workflow
   - Distribution preparation (file organization, metadata)
   - Cost reporting per batch

**Cross-Component Dependencies:**

| Component | Depends On | Provides To |
|-----------|-----------|-------------|
| Document Ingestion | Claude Vision API, CLI libraries | Validated questions → Workflow Engine |
| Workflow Engine | State management, LLM Integration | Orchestration → All prompts |
| LLM Integration | LangChain, Claude API | Prompt execution → All 6 prompts |
| Prompt System | LLM Integration, Question data | Stage results → Code Generation |
| Code Generation | Prompt outputs, Manim templates | Python code → Rendering Pipeline |
| Rendering Pipeline | Generated code, Manim, external libs | Video files → Review Interface |
| Review Interface | Rendered videos, CLI tools | Approved content → Distribution |

**Critical Path for MVP:**
1. Project Setup → Document Ingestion → Workflow Engine → LLM Integration → Code Generation → Rendering → Review

**Parallel Development Opportunities:**
- CLI review tools can be developed independently
- Prompt templates can be refined in parallel
- Brand compliance validation can be built alongside rendering

**Testing Strategy per Component:**
- **Ingestion:** Test with sample IGCSE PDFs
- **Workflow:** Unit tests for state machine transitions
- **LLM Integration:** Mock Claude responses for cost-free testing
- **Code Generation:** Validate generated code syntax and structure
- **Rendering:** Test with small sample animations first
- **Review:** Manual validation initially, automate later

## External Dependencies

### Prompt Content Dependency

**Dependency:** IGCSE_Animation_System_PRD Section 12 (System Prompt Architecture)

**Why External:** Prompt content is product specification, not architectural decision. PRD is source of truth for WHAT prompts should achieve (business logic, pedagogical requirements, brand voice). Architecture defines HOW prompts are implemented (LangChain templates, versioning, execution).

**Architectural Bridge:** `docs/prompts-specification.md`

This companion document bridges PRD and Architecture:
- Defines input/output schemas for all 6 prompts (versioned, testable)
- Provides LangChain PromptTemplate structure (implementation-ready)
- References PRD Section 12 for full prompt content (single source of truth)
- Maps prompts to architectural components (orchestrator, workflow stages)

**Dependency Chain:**
```
IGCSE_Animation_System_PRD Section 12 (Business Logic)
    ↓
docs/prompts-specification.md (Schemas + Templates)
    ↓
Architecture Document (Implementation Decisions)
    ↓
src/igcse_animation/prompts/*.py (LangChain Code)
```

**Implementation Path:**
1. **Epic 4 Prerequisites:** Read PRD Section 12 for full prompt content
2. **Use prompts-specification.md for:**
   - Input variable definitions
   - Output JSON schemas
   - LangChain template structure
   - Integration points with workflow stages
3. **Implement as LangChain PromptTemplates** in `src/igcse_animation/prompts/`
4. **Follow patterns** defined in Implementation Patterns section

**Risk Mitigation:**
- ✓ Prompt schemas versioned in prompts-specification.md (stable contracts)
- ✓ Template structure prevents implementation drift
- ✓ Clear dependency chain documented (no surprises)
- ✓ Schemas enable stub implementations for testing without full PRD
- ⚠️ Requires PRD access during Epic 4 (LLM Prompt Integration)
- ⚠️ PRD changes to prompt logic require prompts-specification.md updates

**Migration Path:** If PRD becomes unavailable, prompts-specification.md contains complete schemas and template structure to create stub implementations for development and testing. Full prompt content can be reconstructed from business requirements.
