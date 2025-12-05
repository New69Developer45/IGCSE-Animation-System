# Implementation Patterns & Consistency Rules

## Overview

These patterns ensure all AI agents and developers write compatible, consistent code that works together seamlessly. Following these patterns prevents conflicts in naming, structure, formats, and processes.

**Critical Conflict Points Addressed:** 5 major categories where implementation could vary

## 1. Naming Patterns (Python PEP 8 Standard)

**Module Names:** `snake_case`
```python
✅ workflow_engine.py, manim_codegen.py, batch_manager.py
❌ WorkflowEngine.py, manimCodegen.py
```

**Class Names:** `PascalCase`
```python
✅ WorkflowEngine, PromptExecutor, CLIReviewer, ManimCodeGenerator
❌ workflow_engine, Prompt_Executor
```

**Function/Method Names:** `snake_case`
```python
✅ execute_workflow(), extract_question_from_pdf(), generate_manim_code()
❌ executeWorkflow(), ExtractQuestion()
```

**Variable Names:** `snake_case`
```python
✅ question_id, workflow_state, assessment_result
❌ questionId, WorkflowState
```

**Constants:** `UPPER_SNAKE_CASE`
```python
✅ BRAND_BLACK = "#000000", MAX_RETRY_ATTEMPTS = 3, API_TIMEOUT_SECONDS = 60
❌ brandBlack, MaxRetryAttempts
```

**JSON Keys:** `snake_case` (consistent with Python)
```python
✅ {"question_id": "PHY_001", "current_stage": "DESIGN"}
❌ {"questionId": "...", "CurrentStage": "..."}
```

**Enum Conventions:** `PascalCase` class, `UPPER_CASE` values
```python
class WorkflowStage(Enum):
    ASSESSMENT = "assessment"
    DESIGN = "design"
    GENERATION = "generation"
```

## 2. Structure Patterns

**Test Organization:**
```
tests/
├── unit/              # test_workflow_engine.py, test_extractor.py
├── integration/       # test_full_workflow.py
└── fixtures/          # sample_questions.json, mock_responses.py
```
- Separate `tests/` directory at project root
- Prefix all test files with `test_`

**Module Organization:**
- Each component is separate module with `__init__.py`
- Clear separation of concerns per module

**Utility Organization:**
```
src/igcse_animation/utils/
├── error_handling.py   # Shared error utilities
├── file_io.py          # File operations
├── logging.py          # Logging setup
└── validators.py       # Data validation
```

**Configuration Organization:**
```
src/igcse_animation/config/
├── brand_config.py       # BRAND_BLACK, BRAND_YELLOW, etc.
├── api_config.py         # API keys, endpoints
├── rendering_config.py   # Video specs
└── workflow_config.py    # Stage settings
```

**Import Pattern:** Always absolute imports
```python
✅ from igcse_animation.utils.error_handling import StageError
❌ from ..utils.error_handling import StageError
```

**Prompt Template Organization:**
- Location: `src/igcse_animation/prompts/`
- Format: LangChain PromptTemplate objects in Python files
- Version controlled, testable

**Data Directory (Never commit to git):**
```
data/
├── inbox/           # Input PDFs
├── extracted/       # Extraction results
├── questions/       # Validated questions
├── workflows/       # Workflow state
├── generated_code/  # Manim code
├── outputs/         # Rendered videos
└── logs/            # Log files
```

## 3. Format Patterns

**LLM Response Standard (All Prompts):**
```python
{
    "status": "success" | "error",
    "result": {
        # Prompt-specific data
    },
    "metadata": {
        "execution_time": 2.3,
        "tokens_used": 1523,
        "model": "claude-3-5-sonnet-20241022"
    },
    "error": {  # Only if status == "error"
        "message": "Human-readable error",
        "code": "ERROR_CODE",
        "details": {}
    }
}
```

**Workflow State Checkpoint Format:**
```python
{
    "question_id": "PHY_2023_P1_Q15",
    "batch_id": "batch_001",
    "current_stage": "GENERATION",
    "created_at": "2025-12-04T10:30:00Z",
    "updated_at": "2025-12-04T10:35:23Z",

    "assessment_result": {},
    "design_spec": {},
    "generated_code": None,
    "validation_results": None,
    "rendered_files": None,
    "review_status": None,

    "total_cost": 0.23,
    "cost_breakdown": {"assessment": 0.08, "design": 0.15},

    "error_log": [],
    "retry_count": 0
}
```

**File Naming Conventions:**
```python
# Questions: {SUBJECT}_{YEAR}_{PAPER}_{QUESTION}.json
"PHY_2023_P1_Q15.json"

# Workflow state: {question_id}_state.json
"PHY_2023_P1_Q15_state.json"

# Generated code: {question_id}.py
"PHY_2023_P1_Q15.py"

# Videos: {question_id}_{format}.mp4
"PHY_2023_P1_Q15_square.mp4"
"PHY_2023_P1_Q15_vertical.mp4"
"PHY_2023_P1_Q15_portrait.mp4"
"PHY_2023_P1_Q15.srt"
```

## 4. Process Patterns

**Standardized Error Handling:**
```python
from dataclasses import dataclass
from enum import Enum

class ErrorSeverity(Enum):
    WARNING = "warning"      # Non-blocking, log and continue
    ERROR = "error"          # Stage failed, checkpoint and stop
    CRITICAL = "critical"    # Workflow failed, cannot recover

@dataclass
class StageError:
    stage: str                    # "ASSESSMENT", "GENERATION", etc.
    severity: ErrorSeverity
    message: str                  # Human-readable message
    details: dict                 # Technical details
    retry_possible: bool
    timestamp: str

    def to_dict(self) -> dict:
        return {
            "stage": self.stage,
            "severity": self.severity.value,
            "message": self.message,
            "details": self.details,
            "retry_possible": self.retry_possible,
            "timestamp": self.timestamp
        }
```

**Error Handling in Workflow Stages:**
```python
try:
    # Execute stage logic
    result = self.generation_prompt.execute(question)
    state.generated_code = result['generated_code']
    state.current_stage = WorkflowStage.VALIDATION
    logger.info(f"Generation complete | question_id={question['id']}")
    return state

except Exception as e:
    error = StageError(
        stage="GENERATION",
        severity=ErrorSeverity.ERROR,
        message=f"Failed to generate code: {str(e)}",
        details={"exception_type": type(e).__name__, "question_id": question['id']},
        retry_possible=True,
        timestamp=datetime.utcnow().isoformat()
    )

    logger.error(f"Generation failed | {error.message}")
    state.error_log.append(error.to_dict())
    state.current_stage = WorkflowStage.FAILED
    return state
```

**Validation Pattern:**
```python
def validate_generated_code(code: str) -> Union[bool, List[str]]:
    """Validate Manim code. Returns True if valid, else list of errors."""
    errors = []

    # Check syntax
    try:
        compile(code, '<string>', 'exec')
    except SyntaxError as e:
        errors.append(f"Syntax error: {e}")

    # Check required imports
    if "from manim import" not in code:
        errors.append("Missing: 'from manim import *'")

    # Check Scene class
    if "class" not in code or "(Scene)" not in code:
        errors.append("Missing Scene class")

    return True if not errors else errors
```

**ESL Validation Pattern:**
```python
from dataclasses import dataclass

@dataclass
class ESLValidationResult:
    """ESL validation results for animation script"""
    passes: bool
    sentence_count: int
    long_sentences: list[dict]  # {sentence, word_count, line_number}
    avg_words_per_sentence: float
    complexity_score: float  # 0-100, lower is simpler
    recommendations: list[str]

def validate_esl_compliance(script: str) -> ESLValidationResult:
    """
    Validate script for ESL accessibility (15-20 words/sentence max).

    Checks:
    - Sentence length (target: 15-20 words max)
    - Average words per sentence
    - Generates recommendations for improvement
    """
    from igcse_animation.validation.esl_validator import ESLValidator

    validator = ESLValidator()
    result = validator.validate_script(script)

    if not result.passes:
        logger.warning(
            f"ESL validation failed | long_sentences={len(result.long_sentences)} | "
            f"avg_words={result.avg_words_per_sentence:.1f}"
        )

    return result
```

**Integration in Workflow:**
```python
# In ValidationStage.execute()
esl_result = self.esl_validator.validate_script(state.generated_script)
state.validation_results['esl'] = esl_result.to_dict()

if not esl_result.passes:
    state.validation_results['warnings'].append('ESL_LONG_SENTENCES')
    # Continue workflow but flag for human review
```

## 5. Logging Patterns

**Logging Setup:**
```python
import logging
from pathlib import Path

def setup_logging(log_dir: str = "data/logs"):
    Path(log_dir).mkdir(parents=True, exist_ok=True)

    logging.basicConfig(
        level=logging.INFO,
        format='%(asctime)s | %(name)s | %(levelname)s | %(message)s',
        datefmt='%Y-%m-%d %H:%M:%S',
        handlers=[
            logging.FileHandler(f'{log_dir}/workflow.log'),
            logging.StreamHandler()
        ]
    )
```

**Logging Usage:**
```python
logger = logging.getLogger(__name__)

# Log levels
logger.debug("Detailed diagnostics")              # DEBUG
logger.info("Workflow stage completed")           # INFO: Normal ops
logger.warning("Validation score below threshold") # WARNING: Potential issues
logger.error("Code generation failed")             # ERROR: Stage failures
logger.critical("API key invalid")                 # CRITICAL: System failures

# Structured log messages
logger.info(
    f"Assessment complete | question_id={q_id} | score={score} | "
    f"decision={decision} | duration={duration:.2f}s"
)
```

**Log File Organization:**
```
data/logs/
├── workflow.log   # All workflow operations
├── api.log        # API calls with costs
└── errors.log     # ERROR and CRITICAL only
```

## Enforcement Guidelines

**All AI Agents MUST:**

1. **Follow Python PEP 8** - `snake_case` modules/functions, `PascalCase` classes, `UPPER_SNAKE_CASE` constants

2. **Use absolute imports** - Never relative imports

3. **Return standardized JSON** - All prompts return `{status, result, metadata, error}`

4. **Use StageError class** - Never return error strings directly

5. **Log all operations** - Appropriate levels with structured context

6. **Save checkpoints** - After every successful stage

7. **Validate generated code** - Before proceeding to rendering

8. **Use type hints** - For all function signatures

**Pre-commit Checks:**
```bash
black src/ tests/              # Format code
flake8 src/ tests/ --max-line-length=88  # Style check
mypy src/                       # Type check
pytest tests/                   # Run tests
```

## Pattern Examples

**✅ Good Example - Complete Stage Implementation:**
```python
from datetime import datetime
import logging
from typing import Optional

from igcse_animation.orchestrator.workflow_engine import WorkflowState, WorkflowStage
from igcse_animation.prompts.assessment import ASSESSMENT_PROMPT
from igcse_animation.utils.error_handling import StageError, ErrorSeverity

logger = logging.getLogger(__name__)

class AssessmentStage:
    """Execute assessment stage"""

    def execute(self, state: WorkflowState, question: dict) -> WorkflowState:
        """Assess question suitability for animation"""
        try:
            response, cost = self.prompt_executor.execute(
                ASSESSMENT_PROMPT.format(**question)
            )

            if response['status'] != 'success':
                raise ValueError(f"Assessment failed: {response.get('error')}")

            state.assessment_result = response['result']
            state.current_stage = WorkflowStage.DESIGN
            state.total_cost += cost

            logger.info(f"Assessment complete | question_id={question['id']}")
            return state

        except Exception as e:
            error = StageError(
                stage="ASSESSMENT",
                severity=ErrorSeverity.ERROR,
                message=f"Assessment failed: {str(e)}",
                details={"question_id": question['id']},
                retry_possible=True,
                timestamp=datetime.utcnow().isoformat()
            )

            state.error_log.append(error.to_dict())
            state.current_stage = WorkflowStage.FAILED
            return state
```

**❌ Anti-Patterns to Avoid:**

```python
# Bad: Inconsistent naming
class workflow_Engine:              # Should be WorkflowEngine
    def ExecuteWorkflow(self):      # Should be execute_workflow
        questionID = "PHY_001"      # Should be question_id

# Bad: Relative imports
from ..orchestrator import engine   # Should use absolute

# Bad: Unstructured errors
def assess(question):
    return "Error: Invalid"         # Should use StageError

# Bad: Missing type hints
def execute_workflow(question):     # Should specify -> WorkflowState
    pass

# Bad: Inconsistent JSON
{"question_id": "...", "currentStage": "..."}  # Mixed snake/camelCase
```

---

**Implementation patterns complete!** These rules ensure consistency across all code, regardless of which AI agent or developer implements each component.
