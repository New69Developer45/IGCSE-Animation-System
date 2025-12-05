# 8. Platform Optimization Requirements

## 8.1 Multi-Format Rendering System

**Automated Aspect Ratio Generation:**
```python
class MultiFormatRenderer:
    """
    Render same content in multiple aspect ratios
    Automatically adjust composition for each format
    """
    
    FORMATS = {
        'square': {'width': 1080, 'height': 1080, 'ratio': '1:1'},
        'vertical': {'width': 1080, 'height': 1920, 'ratio': '9:16'},
        'portrait': {'width': 1080, 'height': 1350, 'ratio': '4:5'}
    }
    
    def render_all_formats(self, scene_class, output_dir="./outputs/"):
        """Render scene in all formats with appropriate adjustments"""
        for format_name, specs in self.FORMATS.items():
            # Configure Manim for this format
            config.pixel_width = specs['width']
            config.pixel_height = specs['height']
            config.frame_width = 8  # Maintain consistent coordinate system
            config.frame_height = 8 * specs['height'] / specs['width']
            
            # Render
            scene = scene_class()
            scene.render()
            
            # Move output to format-specific directory
            output_file = f"{output_dir}/{format_name}/{scene_class.__name__}.mp4"
            shutil.move(scene.renderer.file_writer.movie_file_path, output_file)
```

**Layout Adaptation Strategy:**
```python
def adapt_layout_to_aspect_ratio(format_type: str) -> dict:
    """
    Return layout parameters optimized for each format
    """
    layouts = {
        'square': {
            'title_y': 3.2,
            'main_visual_scale': 1.0,
            'options_y': -2.8,
            'spacing': 0.4
        },
        'vertical': {
            'title_y': 7.5,
            'main_visual_scale': 0.85,
            'options_y': -6.5,
            'spacing': 0.5
        },
        'portrait': {
            'title_y': 5.0,
            'main_visual_scale': 0.95,
            'options_y': -4.5,
            'spacing': 0.45
        }
    }
    return layouts[format_type]
```

## 8.2 Subtitle Generation (SRT Files)

**Automated Caption Creation:**
```python
import datetime

class SubtitleGenerator:
    """Generate SRT subtitle files from scene timing"""
    
    def __init__(self):
        self.subtitles = []
        self.current_index = 1
        
    def add_subtitle(self, text: str, start_time: float, end_time: float):
        """Add a subtitle entry"""
        self.subtitles.append({
            'index': self.current_index,
            'start': self._format_time(start_time),
            'end': self._format_time(end_time),
            'text': text
        })
        self.current_index += 1
    
    def _format_time(self, seconds: float) -> str:
        """Convert seconds to SRT time format (HH:MM:SS,mmm)"""
        td = datetime.timedelta(seconds=seconds)
        hours, remainder = divmod(td.seconds, 3600)
        minutes, seconds = divmod(remainder, 60)
        milliseconds = td.microseconds // 1000
        return f"{hours:02}:{minutes:02}:{seconds:02},{milliseconds:03}"
    
    def export(self, filename: str):
        """Export subtitles to SRT file"""
        with open(filename, 'w', encoding='utf-8') as f:
            for sub in self.subtitles:
                f.write(f"{sub['index']}\n")
                f.write(f"{sub['start']} --> {sub['end']}\n")
                f.write(f"{sub['text']}\n\n")

# Usage in scene
class CaptionedScene(Scene):
    def construct(self):
        captions = SubtitleGenerator()
        
        # Scene timing
        question_text = "What is the acceleration?"
        self.play(Write(question_text))
        captions.add_subtitle("What is the acceleration?", 0.0, 2.5)
        
        explanation = "We use Newton's second law"
        self.play(Write(explanation))
        captions.add_subtitle("We use Newton's second law", 2.5, 5.0)
        
        # Export after scene completes
        captions.export("scene_captions.srt")
```

## 8.3 Platform-Specific Optimizations

**TikTok Optimization:**
```python
class TikTokOptimizer:
    """
    TikTok-specific content optimizations
    """
    
    @staticmethod
    def add_hook_moment(scene, timestamp=1.5):
        """
        Add a visual hook in first 3 seconds (TikTok retention)
        """
        hook = Text(
            "Can you solve this? 🤔",
            font_size=32,
            color=BRAND_YELLOW,
            weight=BOLD
        )
        scene.play(FadeIn(hook, scale=1.3), run_time=0.5)
        scene.wait(timestamp)
        scene.play(FadeOut(hook), run_time=0.3)
    
    @staticmethod
    def add_loop_point(scene):
        """
        Create seamless loop point (TikTok autoplay)
        Hold final frame for 1 second before loop
        """
        scene.wait(1.0)
    
    @staticmethod
    def optimize_pacing():
        """
        TikTok pacing guidelines:
        - Hook: 0-3s
        - Question: 3-10s
        - Explanation: 10-50s
        - Answer: 50-60s
        - CTA: 60-65s
        """
        return {
            'hook': (0, 3),
            'question': (3, 10),
            'explanation': (10, 50),
            'answer': (50, 60),
            'cta': (60, 65)
        }
```

**Facebook Optimization:**
```python
class FacebookOptimizer:
    """
    Facebook-specific content optimizations
    """
    
    @staticmethod
    def add_sound_optional_notice(scene):
        """
        85% of Facebook videos are watched without sound
        Ensure text conveys all information
        """
        notice = Text(
            "🔇 Sound not required",
            font_size=14,
            color=BRAND_WHITE,
            weight=LIGHT
        ).set_opacity(0.5)
        notice.to_corner(DL, buff=0.2)
        return notice
    
    @staticmethod
    def add_engagement_prompt(scene, timestamp=55):
        """
        Add comment/share prompt near end
        """
        prompt = Text(
            "💬 Comment your answer below!",
            font_size=22,
            color=BRAND_YELLOW
        )
        scene.add(prompt)
        scene.wait(timestamp)
        scene.play(FadeIn(prompt, shift=UP*0.3))
    
    @staticmethod
    def optimize_for_feed():
        """
        Facebook feed optimization:
        - First 3 seconds: Clear value proposition
        - Middle: Substantive content
        - End: Clear CTA
        """
        return {
            'value_prop': (0, 3),
            'content': (3, 57),
            'cta': (57, 60)
        }
```

## 8.4 File Naming Convention

**Standardized Output Structure:**
```
outputs/
├── square_1x1/
│   ├── [Subject]_[Topic]_[QuestionID]_square.mp4
│   ├── [Subject]_[Topic]_[QuestionID]_square.srt
├── vertical_9x16/
│   ├── [Subject]_[Topic]_[QuestionID]_vertical.mp4
│   ├── [Subject]_[Topic]_[QuestionID]_vertical.srt
├── portrait_4x5/
│   ├── [Subject]_[Topic]_[QuestionID]_portrait.mp4
│   ├── [Subject]_[Topic]_[QuestionID]_portrait.srt
└── thumbnails/
    ├── [Subject]_[Topic]_[QuestionID]_thumb.png
```

**Example:**
```
Physics_Mechanics_ProjectileMotion_Q15_square.mp4
Chemistry_OrganicChem_IsomerIdentification_Q07_vertical.mp4
Biology_Genetics_PunnettSquare_Q23_portrait.srt
```

---
