# Starter Template & Foundation Architecture

## Primary Technology Domain

**Python AI Orchestration + Code Generation + Animation Rendering System**

This is a unique hybrid system combining multi-LLM orchestration, automated code generation, and video rendering. No conventional starter template applies directly.

## Foundation Architecture Decision

**Selected Approach: Modern Python with Lightweight Orchestration**

**Rationale:**
- Academic accuracy requirements demand simplicity and debuggability
- Well-defined 6-stage workflow doesn't require heavy orchestration initially
- Python 3.12 + Poetry 2.0 provides modern, maintainable foundation
- LangChain prompt management without LangGraph complexity
- Custom state machine offers full control for mission-critical workflow

## Technology Stack

### 1. Project Structure & Dependency Management

**Poetry 2.0 (Released January 5, 2025)**

- Modern Python dependency management with `pyproject.toml`
- Automatic virtual environment management
- Integration with Black, Flake8, MyPy
- Python 3.12+ recommended for new systems

**Initialization:**
```bash
# Install Poetry
curl -sSL https://install.python-poetry.org | python3 -

# Create project
poetry new igcse-animation-system --name igcse_animation --src

# Add dependencies
poetry add manim py3dmol rdkit vpython matplotlib numpy scipy pillow
poetry add langchain langchain-anthropic langchain-openai
poetry add --group dev black flake8 mypy pytest
```

**Project Structure:**
```
igcse_animation_system/
├── pyproject.toml                    # Dependencies, tools, metadata
├── src/
│   └── igcse_animation/
│       ├── __init__.py
│       ├── orchestrator/             # Master orchestrator
│       │   ├── __init__.py
│       │   ├── workflow_engine.py    # Custom state machine
│       │   └── state_manager.py      # Workflow state tracking
│       ├── prompts/                  # 6 specialized prompts (LangChain templates)
│       │   ├── __init__.py
│       │   ├── assessment.py         # Question suitability scoring
│       │   ├── design.py             # Animation planning
│       │   ├── generation.py         # Manim code generation
│       │   ├── validation.py         # Quality assurance
│       │   └── review_interface.py   # Human review guidance
│       ├── generation/               # Code generation engine
│       │   ├── __init__.py
│       │   ├── manim_codegen.py      # Manim scene generator
│       │   ├── library_integrator.py # py3Dmol, VPython, Matplotlib
│       │   └── templates/            # Code templates
│       ├── rendering/                # Multi-format rendering pipeline
│       │   ├── __init__.py
│       │   ├── format_renderer.py    # Square/Vertical/Portrait
│       │   └── subtitle_generator.py # SRT file generation
│       ├── validation/               # Quality assurance framework
│       │   ├── __init__.py
│       │   ├── academic_validator.py # IGCSE spec compliance
│       │   ├── brand_validator.py    # Visual identity checks
│       │   ├── code_validator.py     # Black/Flake8/MyPy
│       │   └── esl_validator.py      # ESL accessibility checks
│       ├── review/                   # Human review interface
│       │   ├── __init__.py
│       │   └── approval_workflow.py  # Human-in-the-loop
│       ├── config/                   # Configuration management
│       │   ├── __init__.py
│       │   ├── brand_config.py       # Colors, fonts, layout
│       │   └── rendering_config.py   # Video specs
│       └── utils/                    # Shared utilities
│           ├── __init__.py
│           └── error_handling.py     # Fallback decision trees
├── tests/
│   ├── unit/
│   ├── integration/
│   └── fixtures/
├── data/
│   ├── questions/                    # Input questions
│   ├── mark_schemes/                 # IGCSE marking guides
│   └── outputs/                      # Generated videos
├── docs/
└── README.md
```

### 2. Multi-LLM Prompt Management

**LangChain PromptTemplate**

**Why LangChain (not LangGraph):**
- Your 6 prompts are well-defined and mostly sequential
- LangChain `PromptTemplate` provides reusable, versioned prompts
- Simpler than LangGraph for initial implementation
- Easy migration path to LangGraph if workflows become more complex

**Implementation Approach:**
```python
from langchain.prompts import PromptTemplate

# Example: Assessment Prompt Template
assessment_prompt = PromptTemplate(
    input_variables=["question_text", "subject", "topic"],
    template="""
    # IGCSE Animation System - Question Assessment Specialist

    Evaluate this IGCSE exam question for animation suitability:

    Subject: {subject}
    Topic: {topic}
    Question: {question_text}

    [... full prompt from PRD Section 12.3 ...]
    """
)
```

**Prompt Versioning Strategy:**
- Store prompts in `src/igcse_animation/prompts/` as Python modules
- Version control with Git
- Track prompt performance metrics for continuous improvement

### 3. Workflow Orchestration

**Custom Python State Machine**

**Why Custom (not Prefect/Temporal initially):**
- Simple, debuggable, full control
- Academic accuracy demands transparency
- No distributed execution needed initially
- Can migrate to Prefect/Temporal later if needed

**State Machine Design:**
```python
from enum import Enum
from dataclasses import dataclass

class WorkflowStage(Enum):
    ASSESSMENT = "assessment"
    DESIGN = "design"
    GENERATION = "generation"
    VALIDATION = "validation"
    RENDERING = "rendering"
    REVIEW = "review"
    COMPLETE = "complete"
    FAILED = "failed"

@dataclass
class WorkflowState:
    question_id: str
    current_stage: WorkflowStage
    assessment_result: dict | None = None
    design_spec: dict | None = None
    generated_code: str | None = None
    validation_results: dict | None = None
    rendered_files: list[str] | None = None
    review_status: str | None = None
    error_log: list[str] | None = None

class WorkflowEngine:
    """
    Manages 6-stage animation generation workflow with
    decision trees and fallback logic.
    """
    def execute(self, question: dict) -> WorkflowState:
        state = WorkflowState(
            question_id=question['id'],
            current_stage=WorkflowStage.ASSESSMENT
        )

        # Stage 1: Assessment
        state = self._run_assessment(state, question)
        if state.current_stage == WorkflowStage.FAILED:
            return state

        # Stage 2: Design
        state = self._run_design(state)
        # ... continue through stages

        return state
```

**Migration Path to Prefect (Future):**
When you need:
- Parallel question processing
- Retry logic and error recovery
- Workflow observability dashboard
- Distributed rendering across machines

Then migrate to Prefect with minimal refactoring:
```python
from prefect import flow, task

@task
def assess_question(question: dict) -> dict:
    # Same logic as custom state machine
    pass

@flow
def animation_workflow(question: dict):
    assessment = assess_question(question)
    design = create_design(assessment)
    # ... Prefect handles orchestration
```

## Architectural Decisions Provided by This Foundation

**Language & Runtime:**
- Python 3.12+ with type hints (MyPy enforced)
- Poetry 2.0 for dependency management
- Virtual environment isolation

**Prompt Management:**
- LangChain `PromptTemplate` for 6 specialized prompts
- Version-controlled prompt templates in Git
- Modular prompt design for independent updates

**Workflow Orchestration:**
- Custom Python state machine for transparent control
- Clear stage boundaries with state persistence
- Decision tree logic for edge cases and fallbacks
- Easy migration to Prefect when scale demands it

**Code Quality:**
- Black formatting (enforced in pyproject.toml)
- Flake8 linting (max-line-length=88)
- MyPy type checking (strict mode)
- Pytest for unit and integration tests

**Library Integration:**
- Manim as primary animation engine
- py3Dmol/RDKit for chemistry visualization
- VPython for 3D physics
- Matplotlib for graphs with brand styling
- All managed through Poetry dependencies

**Development Experience:**
- Poetry shell for isolated development
- Black/Flake8/MyPy pre-commit hooks
- Pytest with fixtures for reproducible tests
- Clear project structure with logical separation

## Implementation First Steps

1. **Initialize Project:**
   ```bash
   poetry new igcse-animation-system --name igcse_animation --src
   cd igcse-animation-system
   ```

2. **Configure pyproject.toml:**
   - Add all dependencies (Manim, LangChain, visualization libraries)
   - Configure Black, Flake8, MyPy in [tool] sections
   - Set Python version to >=3.12

3. **Create Directory Structure:**
   - Set up src/igcse_animation/ with 7 main modules
   - Initialize tests/ with unit and integration directories
   - Create data/ for questions, mark schemes, outputs

4. **Implement Core State Machine:**
   - Build WorkflowEngine with 6-stage execution
   - Implement fallback decision trees from PRD Section 10
   - Add error handling and logging

5. **Create Prompt Templates:**
   - Port 6 specialized prompts from PRD Section 12 to LangChain templates
   - Implement prompt versioning strategy
   - Connect prompts to Anthropic Claude API (or OpenAI)

**Note:** Project initialization is the foundation for all subsequent development. This should be the first epic in implementation planning.
