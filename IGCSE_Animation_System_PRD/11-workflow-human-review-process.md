# 11. Workflow & Human Review Process

## 11.1 Production Workflow

**End-to-End Process:**
```
┌─────────────────────────────────────────────────────────────────┐
│ STAGE 1: Input & Assessment                                     │
├─────────────────────────────────────────────────────────────────┤
│ 1. Load question from past paper database                       │
│ 2. Run automated assessments (visualization, complexity, etc.)  │
│ 3. Decision: Proceed / Request Guidance / Suggest Alternative   │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ STAGE 2: Animation Design Planning                              │
├─────────────────────────────────────────────────────────────────┤
│ 1. Select animation strategy based on question type             │
│ 2. Generate scene outline (intro → explanation → answer)        │
│ 3. Identify required libraries (Manim, py3Dmol, VPython, etc.)  │
│ 4. Create timing breakdown (60-120s total)                      │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ STAGE 3: Code Generation                                        │
├─────────────────────────────────────────────────────────────────┤
│ 1. Generate Manim scene class                                   │
│ 2. Integrate brand colors and typography                        │
│ 3. Add watermark and logo elements                              │
│ 4. Implement examiner tips and misconception segments           │
│ 5. Run code quality checks (linting, type hints)                │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ STAGE 4: Pre-Render Validation                                  │
├─────────────────────────────────────────────────────────────────┤
│ 1. Validate scene configuration                                 │
│ 2. Check brand compliance                                       │
│ 3. Verify academic accuracy                                     │
│ 4. Confirm timing constraints                                   │
│ → If validation fails: Return to Stage 3 with corrections       │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ STAGE 5: Rendering                                              │
├─────────────────────────────────────────────────────────────────┤
│ 1. Render in all three aspect ratios (1:1, 9:16, 4:5)           │
│ 2. Generate thumbnail images                                    │
│ 3. Create subtitle files (.srt)                                 │
│ 4. Organize outputs in standardized directory structure         │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ STAGE 6: Post-Render Validation                                 │
├─────────────────────────────────────────────────────────────────┤
│ 1. Check video properties (resolution, duration, frame rate)    │
│ 2. Verify subtitle sync                                         │
│ 3. Test playback on mobile viewport                             │
│ → If validation fails: Return to Stage 3 or 5                   │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ STAGE 7: Human Review (MANDATORY)                               │
├─────────────────────────────────────────────────────────────────┤
│ 1. Review content against checklist (academic, brand, pedagogy) │
│ 2. Approve or request revisions                                 │
│ 3. If approved: Proceed to Stage 8                              │
│ 4. If revisions needed: Return to Stage 3 with specific notes   │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│ STAGE 8: Publishing Preparation                                 │
├─────────────────────────────────────────────────────────────────┤
│ 1. Add platform-specific optimizations (hooks, CTAs)            │
│ 2. Generate posting captions with hashtags                      │
│ 3. Schedule for optimal posting times                           │
│ 4. Archive source code and project files                        │
└─────────────────────────────────────────────────────────────────┘
```

## 11.2 Human Review Interface

**Review Dashboard Requirements:**
```python
class ReviewDashboard:
    """
    Interface for human reviewers to assess generated content
    """
    
    def display_review_item(self, animation_id: str):
        """
        Display animation with review checklist
        """
        # Load animation data
        data = self._load_animation(animation_id)
        
        # Display video preview (all aspect ratios)
        self._show_video_preview(data['videos'])
        
        # Display checklist
        checklist = self._generate_checklist()
        
        # Display automated assessment scores
        self._show_assessment_scores(data['assessments'])
        
        # Display source question and mark scheme
        self._show_source_material(data['question_data'])
        
        # Review options
        return self._get_reviewer_decision()
    
    def _generate_checklist(self) -> dict:
        """Generate interactive checklist"""
        return {
            'academic_accuracy': [
                'All calculations verified',
                'Equations balanced/correct',
                'Units and notation correct',
                'Explanation matches mark scheme'
            ],
            'brand_compliance': [
                'Only brand colors used',
                'Logo present in outro',
                'Watermark present throughout',
                'Typography standards met'
            ],
            'pedagogical_quality': [
                'Explanation is clear for ESL students',
                'Common misconceptions addressed',
                'Examiner tips included',
                'Key insights emphasized'
            ],
            'technical_quality': [
                'Animations smooth',
                'Timing appropriate',
                'All formats rendered correctly',
                'Subtitles synced'
            ]
        }
    
    def _get_reviewer_decision(self) -> dict:
        """
        Capture reviewer decision
        """
        decision = input("Decision (approve/revise/reject): ").lower()
        
        if decision == 'approve':
            return {'status': 'approved', 'next_stage': 'publishing'}
        
        elif decision == 'revise':
            notes = input("Revision notes: ")
            return {
                'status': 'needs_revision',
                'notes': notes,
                'next_stage': 'regenerate'
            }
        
        else:  # reject
            reason = input("Rejection reason: ")
            return {
                'status': 'rejected',
                'reason': reason,
                'next_stage': 'select_new_question'
            }
```

## 11.3 Feedback Loop Integration

**Learning from Reviews:**
```python
class ReviewLearningSystem:
    """
    Track reviewer feedback to improve automation
    """
    
    def __init__(self, log_file='review_feedback.json'):
        self.log_file = log_file
        self.feedback_data = self._load_feedback()
    
    def log_review(self, animation_id: str, review_data: dict):
        """Record review decision and notes"""
        entry = {
            'timestamp': datetime.now().isoformat(),
            'animation_id': animation_id,
            'decision': review_data['status'],
            'revision_notes': review_data.get('notes', ''),
            'question_metadata': review_data.get('question_data', {})
        }
        
        self.feedback_data.append(entry)
        self._save_feedback()
    
    def analyze_common_issues(self) -> dict:
        """
        Identify patterns in revision requests
        """
        revision_reasons = []
        for entry in self.feedback_data:
            if entry['decision'] == 'needs_revision':
                revision_reasons.append(entry['revision_notes'])
        
        # Categorize issues
        categories = {
            'academic_errors': 0,
            'brand_violations': 0,
            'timing_issues': 0,
            'clarity_problems': 0
        }
        
        for reason in revision_reasons:
            # Categorization logic...
            pass
        
        return {
            'total_reviews': len(self.feedback_data),
            'approval_rate': self._calculate_approval_rate(),
            'common_issues': categories,
            'recommendations': self._generate_improvement_recommendations(categories)
        }
    
    def _calculate_approval_rate(self) -> float:
        """Calculate percentage of first-pass approvals"""
        approved = sum(1 for e in self.feedback_data if e['decision'] == 'approved')
        return approved / len(self.feedback_data) * 100 if self.feedback_data else 0.0
    
    def _generate_improvement_recommendations(self, issue_categories: dict) -> List[str]:
        """Suggest system improvements based on review patterns"""
        recommendations = []
        
        if issue_categories['academic_errors'] > 5:
            recommendations.append("Enhance fact-checking validation in Stage 4")
        
        if issue_categories['timing_issues'] > 5:
            recommendations.append("Adjust automated pacing calculations")
        
        # Additional recommendations...
        
        return recommendations
```

---
