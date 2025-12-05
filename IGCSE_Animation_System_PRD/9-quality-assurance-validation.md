# 9. Quality Assurance & Validation

## 9.1 Automated Quality Checks

**Pre-Render Validation:**
```python
class QualityValidator:
    """
    Automated quality checks before rendering
    """
    
    def __init__(self, question_data: dict):
        self.data = question_data
        self.errors = []
        self.warnings = []
    
    def validate(self) -> Tuple[bool, List[str], List[str]]:
        """
        Run all validation checks
        Returns: (is_valid, errors, warnings)
        """
        self._check_required_fields()
        self._check_color_usage()
        self._check_text_length()
        self._check_timing_constraints()
        self._check_brand_compliance()
        
        is_valid = len(self.errors) == 0
        return is_valid, self.errors, self.warnings
    
    def _check_required_fields(self):
        """Ensure all required fields present"""
        required = ['subject', 'topic', 'question_text', 'correct_answer', 'explanation']
        missing = [f for f in required if f not in self.data]
        if missing:
            self.errors.append(f"Missing required fields: {', '.join(missing)}")
    
    def _check_color_usage(self):
        """Verify only brand colors are used"""
        allowed_colors = ['#000000', '#FFFFFF', '#FFFF00', '#006600']
        if 'colors' in self.data:
            invalid = [c for c in self.data['colors'] if c not in allowed_colors]
            if invalid:
                self.errors.append(f"Non-brand colors used: {', '.join(invalid)}")
    
    def _check_text_length(self):
        """Ensure text fits readability constraints"""
        if 'on_screen_text' in self.data:
            for text_element in self.data['on_screen_text']:
                if len(text_element) > 100:
                    self.warnings.append(f"Text may be too long: '{text_element[:30]}...'")
    
    def _check_timing_constraints(self):
        """Validate video duration"""
        if 'total_duration' in self.data:
            duration = self.data['total_duration']
            if duration < 60:
                self.warnings.append(f"Video is short ({duration}s), consider adding context")
            elif duration > 120:
                self.errors.append(f"Video exceeds maximum duration (120s): {duration}s")
    
    def _check_brand_compliance(self):
        """Verify logo and watermark presence"""
        if not self.data.get('watermark_present', False):
            self.errors.append("Watermark not configured")
        
        if not self.data.get('logo_present', False):
            self.warnings.append("Logo not present in outro")
```

**Post-Render Validation:**
```python
import cv2

class PostRenderValidator:
    """
    Validate rendered video files
    """
    
    @staticmethod
    def validate_video(video_path: str) -> dict:
        """
        Check rendered video properties
        Returns validation report
        """
        report = {
            'valid': True,
            'issues': []
        }
        
        # Open video file
        cap = cv2.VideoCapture(video_path)
        
        # Check resolution
        width = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
        height = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
        
        if width != 1080:
            report['issues'].append(f"Width should be 1080, got {width}")
            report['valid'] = False
        
        # Check duration
        fps = cap.get(cv2.CAP_PROP_FPS)
        frame_count = int(cap.get(cv2.CAP_PROP_FRAME_COUNT))
        duration = frame_count / fps
        
        if duration < 60 or duration > 120:
            report['issues'].append(f"Duration out of range: {duration:.1f}s")
            report['valid'] = False
        
        # Check frame rate
        if fps not in [30, 60]:
            report['issues'].append(f"Unexpected frame rate: {fps}")
            report['valid'] = False
        
        cap.release()
        
        return report
    
    @staticmethod
    def check_subtitle_sync(video_path: str, srt_path: str) -> bool:
        """
        Verify subtitle timing matches video duration
        """
        # Parse SRT file
        with open(srt_path, 'r', encoding='utf-8') as f:
            srt_content = f.read()
        
        # Extract last timestamp
        timestamps = re.findall(r'(\d{2}:\d{2}:\d{2},\d{3})', srt_content)
        if not timestamps:
            return False
        
        last_timestamp = timestamps[-1]
        # Convert to seconds and compare with video duration
        # Implementation...
        
        return True
```

## 9.2 Difficulty Assessment System

**Automated Complexity Scoring:**
```python
class DifficultyAnalyzer:
    """
    Assess question difficulty and flag complex cases
    """
    
    COMPLEXITY_FACTORS = {
        'multiple_steps': 15,          # Multi-step calculations
        'abstract_concept': 20,        # Hard-to-visualize topics
        'advanced_math': 25,           # Complex equations
        '3d_visualization': 20,        # 3D models required
        'multiple_representations': 15, # Needs graphs + diagrams + text
        'long_explanation': 10         # >60s explanation needed
    }
    
    def assess_question(self, question_data: dict) -> dict:
        """
        Calculate difficulty score and determine if special handling needed
        """
        score = 0
        factors_present = []
        
        # Analyze question characteristics
        if self._requires_multiple_steps(question_data):
            score += self.COMPLEXITY_FACTORS['multiple_steps']
            factors_present.append('multiple_steps')
        
        if self._is_abstract_concept(question_data):
            score += self.COMPLEXITY_FACTORS['abstract_concept']
            factors_present.append('abstract_concept')
        
        # Additional checks...
        
        # Determine handling approach
        if score >= 50:
            flag = "COMPLEX - Request human guidance"
            auto_proceed = False
        elif score >= 30:
            flag = "MODERATE - Implement with extra review"
            auto_proceed = True
        else:
            flag = "STRAIGHTFORWARD - Standard implementation"
            auto_proceed = True
        
        return {
            'complexity_score': score,
            'factors': factors_present,
            'flag': flag,
            'auto_proceed': auto_proceed,
            'recommendations': self._generate_recommendations(factors_present)
        }
    
    def _requires_multiple_steps(self, data: dict) -> bool:
        """Check if question needs multi-step solution"""
        # Analysis logic
        pass
    
    def _is_abstract_concept(self, data: dict) -> bool:
        """Check if concept is hard to visualize"""
        abstract_keywords = ['entropy', 'electromagnetic induction', 
                            'quantum', 'equilibrium constant']
        return any(kw in data.get('topic', '').lower() for kw in abstract_keywords)
    
    def _generate_recommendations(self, factors: List[str]) -> List[str]:
        """Provide implementation recommendations based on complexity"""
        recommendations = []
        
        if 'abstract_concept' in factors:
            recommendations.append("Use analogies and concrete examples")
            recommendations.append("Break into simpler sub-concepts first")
        
        if '3d_visualization' in factors:
            recommendations.append("Consider multiple camera angles")
            recommendations.append("Use py3Dmol or VPython for realistic rendering")
        
        if 'multiple_steps' in factors:
            recommendations.append("Use clear segmentation between steps")
            recommendations.append("Add progress indicators (Step 1/4, etc.)")
        
        return recommendations
```

## 9.3 Human Review Checklist

**Pre-Publishing Review Protocol:**
```markdown
# Animation Review Checklist

# Academic Accuracy
- [ ] All numerical values verified against mark scheme
- [ ] Chemical equations balanced correctly
- [ ] Physics units and notation correct (SI units, vector notation)
- [ ] Biological terminology matches current IGCSE specifications
- [ ] Mathematical working shows logical progression

# Brand Compliance
- [ ] Only brand colors used (#000000, #FFFFFF, #FFFF00, #006600)
- [ ] Logo present in outro at full opacity
- [ ] Watermark present throughout at 50% opacity
- [ ] Font sizes meet minimum readability standards (18px+)

# Pedagogical Quality
- [ ] Explanation matches examiner's perspective
- [ ] Common misconceptions addressed (if applicable)
- [ ] Language appropriate for ESL students
- [ ] Key insights emphasized with yellow highlights
- [ ] Examiner tips included (max 2 per video)

# Technical Quality
- [ ] Smooth animations with no jarring transitions
- [ ] Timing allows for comfortable reading (2-3s per text element)
- [ ] Audio-free content is fully comprehensible from text alone
- [ ] All three aspect ratios rendered correctly
- [ ] Subtitle files (.srt) generated and synced

# Platform Optimization
- [ ] Duration within 60-120s range
- [ ] Hook moment present in first 3 seconds (TikTok)
- [ ] Engagement prompt included near end (Facebook)
- [ ] File naming follows convention
- [ ] Thumbnails generated for all formats

# Final Approval
Reviewer Name: _______________
Date: _______________
Approved: [ ] Yes  [ ] Needs Revision

Revision Notes (if applicable):
_________________________________
_________________________________
```

---
