# 7. Technical Implementation Specifications

## 7.1 Manim Configuration Standards

**Base Configuration (applies to all animations):**
```python
from manim import *
import numpy as np

class IGCSEQuestionBase(Scene):
    def setup(self):
        """Standard setup for all IGCSE animations"""
        # Brand colors
        self.BRAND_BLACK = "#000000"
        self.BRAND_WHITE = "#FFFFFF"
        self.BRAND_YELLOW = "#FFFF00"
        self.BRAND_GREEN = "#006600"
        
        # Background
        self.camera.background_color = self.BRAND_BLACK
        
        # Watermark setup
        self.watermark = self._create_watermark()
        
    def _create_watermark(self):
        """Create persistent watermark"""
        watermark = Text(
            "T.Y. Supplementary Centre",
            font_size=16,
            color=self.BRAND_WHITE,
            weight=LIGHT
        )
        watermark.set_opacity(0.5)
        watermark.to_corner(UR, buff=0.25)
        return watermark
```

**Render Quality Settings:**
```bash
# For development/preview (faster rendering)
manim -pql scene.py SceneName  # 480p, 15fps

# For production (high quality)
manim -pqh scene.py SceneName  # 1080p, 60fps

# For platform-specific rendering
manim -pqh --format=mp4 --fps=30 scene.py SceneName
```

## 7.2 Code Structure Requirements

**Modularity Principles:**
1. **Helper Functions:** Create reusable functions for common elements
2. **Scene Inheritance:** Use base classes for shared functionality
3. **Configuration Files:** Store constants in separate config files
4. **Documentation:** Docstrings for all classes and functions

**Example Modular Structure:**
```python
# config.py
class VisualConfig:
    BRAND_BLACK = "#000000"
    BRAND_WHITE = "#FFFFFF"
    BRAND_YELLOW = "#FFFF00"
    BRAND_GREEN = "#006600"
    
    FONT_SIZES = {
        'title': 32,
        'body': 24,
        'annotation': 18,
        'watermark': 16
    }

# helpers.py
def create_equation_highlight(equation, index, color=BRAND_YELLOW):
    """Highlight specific term in equation"""
    rect = SurroundingRectangle(equation[index], color=color, buff=0.1)
    return rect

def create_examiner_tip(tip_text):
    """Create standardized examiner tip box"""
    tip_box = VGroup(
        Rectangle(width=6, height=1, fill_color=BRAND_YELLOW, fill_opacity=0.2),
        Text(f"💡 Examiner Tip: {tip_text}", font_size=18)
    )
    return tip_box

# main_scene.py
from config import VisualConfig
from helpers import create_equation_highlight, create_examiner_tip

class PhysicsQuestion(Scene):
    # Implementation using imported helpers
```

## 7.3 Integration with External Libraries

**Chemistry: py3Dmol Integration**
```python
import py3Dmol
from rdkit import Chem
from rdkit.Chem import AllChem
import io
from PIL import Image

def generate_molecule_frame(smiles, angle=0):
    """
    Generate molecule visualization as PIL Image
    For integration into Manim ImageMobject
    """
    mol = Chem.MolFromSmiles(smiles)
    AllChem.EmbedMolecule(mol)
    AllChem.UFFOptimizeMolecule(mol)
    
    view = py3Dmol.view(width=800, height=600)
    view.addModel(Chem.MolToMolBlock(mol), 'mol')
    view.setStyle({'stick': {'colorscheme': 'Jmol'}, 
                   'sphere': {'radius': 0.3, 'colorscheme': 'Jmol'}})
    view.rotate(angle, 'y')
    view.zoomTo()
    
    # Render to PNG and return as PIL Image
    png_data = view.png()
    return Image.open(io.BytesIO(png_data))

# Usage in Manim
class MoleculeVisualization(Scene):
    def construct(self):
        # Generate frames at different rotation angles
        frames = [generate_molecule_frame("CCO", angle=i) 
                  for i in range(0, 360, 10)]
        
        # Create image sequence animation
        molecule_imgs = [ImageMobject(frame) for frame in frames]
        # Animate rotation...
```

**Physics: VPython for 3D Simulations**
```python
from vpython import *
import numpy as np

def create_3d_physics_scene(export_path="./frames/"):
    """
    Create 3D physics simulation and export frame sequence
    """
    scene = canvas(width=1080, height=1080, background=color.black)
    
    # Create objects
    ball = sphere(pos=vector(0, 5, 0), radius=0.5, color=color.yellow)
    floor = box(pos=vector(0, -1, 0), size=vector(10, 0.2, 10), color=color.white)
    
    # Physics parameters
    ball.velocity = vector(2, 0, 0)
    g = vector(0, -9.8, 0)
    dt = 0.01
    
    frame_count = 0
    for t in np.arange(0, 5, dt):
        rate(100)  # Limit to 100 calculations per second
        
        # Update physics
        ball.velocity += g * dt
        ball.pos += ball.velocity * dt
        
        # Export frame every 0.033s (30fps)
        if frame_count % 3 == 0:
            scene.capture(f"{export_path}/frame_{frame_count:04d}.png")
        frame_count += 1
    
    return frame_count

# Integrate frames into Manim
class PhysicsSimulation(Scene):
    def construct(self):
        frame_count = create_3d_physics_scene()
        
        # Load frames as image sequence
        animation = ImageSequence(f"./frames/frame_", 
                                  num_frames=frame_count//3,
                                  duration=5)
        self.play(animation)
```

**Data Visualization: Matplotlib Integration**
```python
import matplotlib.pyplot as plt
import matplotlib
matplotlib.use('Agg')  # Non-interactive backend

def create_graph_figure(x_data, y_data, title=""):
    """
    Create matplotlib figure with brand styling
    Return as PIL Image for Manim
    """
    fig, ax = plt.subplots(figsize=(8, 6), facecolor='#000000')
    ax.set_facecolor('#000000')
    
    # Plot with brand colors
    ax.plot(x_data, y_data, color='#FFFF00', linewidth=2)
    ax.set_xlabel('Time (s)', color='#FFFFFF', fontsize=14)
    ax.set_ylabel('Velocity (m/s)', color='#FFFFFF', fontsize=14)
    ax.set_title(title, color='#FFFFFF', fontsize=16)
    
    # Styling
    ax.tick_params(colors='#FFFFFF')
    ax.spines['bottom'].set_color('#FFFFFF')
    ax.spines['left'].set_color('#FFFFFF')
    ax.spines['top'].set_visible(False)
    ax.spines['right'].set_visible(False)
    ax.grid(True, alpha=0.2, color='#FFFFFF')
    
    # Convert to image
    fig.canvas.draw()
    img = Image.frombytes('RGB', fig.canvas.get_width_height(),
                          fig.canvas.tostring_rgb())
    plt.close(fig)
    return img

# Usage in Manim
class GraphQuestion(Scene):
    def construct(self):
        x = np.linspace(0, 10, 100)
        y = 5 * x - 0.5 * 9.8 * x**2
        
        graph_img = create_graph_figure(x, y, "Projectile Motion")
        graph_mobject = ImageMobject(graph_img)
        
        self.play(FadeIn(graph_mobject))
```

## 7.4 Code Quality Standards

**Linting and Formatting:**
```bash
# Use Black for consistent code formatting
black --line-length 88 scene.py

# Use Flake8 for linting
flake8 scene.py --max-line-length=88

# Use MyPy for type checking
mypy scene.py
```

**Type Hints:**
```python
from typing import List, Tuple, Optional
from manim import Scene, Mobject, VGroup

def create_answer_options(
    options: List[str],
    correct_index: int,
    position: np.ndarray = UP * 2
) -> Tuple[VGroup, Mobject]:
    """
    Create multiple choice options with correct answer indicator
    
    Args:
        options: List of answer choice strings
        correct_index: Index of correct answer (0-based)
        position: Position for option group
        
    Returns:
        Tuple of (option_group, correct_indicator)
    """
    # Implementation
    pass
```

**Error Handling:**
```python
class QuestionRenderer:
    def render_question(self, question_data: dict) -> bool:
        """
        Render question with comprehensive error handling
        
        Returns:
            True if successful, False if rendering failed
        """
        try:
            # Validate input data
            required_fields = ['subject', 'topic', 'question_text', 'answer']
            missing = [f for f in required_fields if f not in question_data]
            if missing:
                raise ValueError(f"Missing required fields: {missing}")
            
            # Attempt rendering
            scene = self._create_scene(question_data)
            scene.render()
            
            return True
            
        except Exception as e:
            self._log_error(f"Rendering failed: {str(e)}")
            return False
    
    def _log_error(self, message: str):
        """Log error to file for human review"""
        with open("render_errors.log", "a") as f:
            f.write(f"{datetime.now()}: {message}\n")
```

## 7.5 Performance Optimization

**Rendering Efficiency:**
```python
# Use VGroup for batch operations
elements = VGroup(*[Circle() for _ in range(100)])
self.play(FadeIn(elements))  # More efficient than individual FadeIn calls

# Cache complex computations
@lru_cache(maxsize=128)
def calculate_trajectory(v0, angle, g=9.8):
    """Cache trajectory calculations"""
    return lambda t: v0 * np.sin(angle) * t - 0.5 * g * t**2

# Limit animation resolution for preview
config.pixel_width = 854   # 480p for faster preview
config.pixel_height = 480
config.frame_rate = 15     # Lower frame rate for preview
```

**Memory Management:**
```python
class LargeDataScene(Scene):
    def construct(self):
        # Create and use data
        large_dataset = self.generate_data()
        visualization = self.visualize(large_dataset)
        self.play(Create(visualization))
        
        # Explicitly remove from scene after use
        self.remove(visualization)
        del large_dataset, visualization
        
        # Continue with next section
```

---
