# 6. Animation Design Plans by Subject

## 6.1 Mathematics

### Question Type Categories:

**A. Algebraic Manipulation**
- **Visual Approach:** Show step-by-step term rearrangement
- **Animation Style:** Transform equations with highlighted changes
- **Duration:** 60-90s

**Example Implementation:**
```python
class AlgebraQuestion(Scene):
    def construct(self):
        # Initial equation
        eq1 = MathTex("3x", "+", "5", "=", "20")
        
        # Step 1: Subtract 5
        eq2 = MathTex("3x", "=", "20", "-", "5")
        arrow1 = MathTex(r"\text{subtract 5 from both sides}")
        
        # Step 2: Simplify
        eq3 = MathTex("3x", "=", "15")
        
        # Step 3: Divide by 3
        eq4 = MathTex("x", "=", "5")
        
        # Animate transformations with yellow highlights
```

**B. Geometry & Trigonometry**
- **Visual Approach:** Dynamic diagrams with angle/length annotations
- **Animation Style:** Geometric constructions that build progressively
- **Key Elements:**
  - Animated compass/ruler tools for constructions
  - Color-coded angles and sides
  - Pythagorean theorem visual proofs
  - Trigonometric ratio overlays

**C. Graph Analysis & Functions**
- **Visual Approach:** Interactive plotting with point tracking
- **Animation Style:** Function curves that draw themselves
- **Key Elements:**
  - Axes with clear scaling
  - Point-by-point plot construction
  - Tangent lines for gradient
  - Area under curve shading for integrals

**D. Statistics & Probability**
- **Visual Approach:** Animated data visualizations
- **Animation Style:** Bar charts, pie charts, probability trees that build step-by-step
- **Key Elements:**
  - Data point aggregation animations
  - Probability branch highlighting
  - Mean/median indicators on distributions

## 6.2 Physics

### Question Type Categories:

**A. Forces & Motion**
- **Visual Approach:** Vector diagrams with force arrows
- **Animation Style:** Objects moving along calculated trajectories
- **Key Elements:**
  - Free body diagrams with force vectors
  - Velocity/acceleration vectors updating in real-time
  - Parabolic motion paths for projectiles
  - Energy bar charts showing transformations

**Example (from your code):**
```python
# Projectile motion with trajectory
path = FunctionGraph(
    lambda x: x * np.tan(θ) - (g * x**2) / (2 * v₀**2 * np.cos(θ)**2),
    color=BRAND_YELLOW
)
ball = Dot(color=BRAND_WHITE)
self.play(MoveAlongPath(ball, path, run_time=3))
```

**B. Electricity & Magnetism**
- **Visual Approach:** Circuit diagrams with current flow animation
- **Animation Style:** Field lines, charge movement, magnetic domains
- **Key Elements:**
  - Animated current (electron drift or conventional flow)
  - Magnetic field lines from magnets
  - Induced magnetism in ferromagnetic materials
  - Voltage/current meters with dynamic readings

**C. Waves & Optics**
- **Visual Approach:** Sinusoidal wave animations
- **Animation Style:** Ray diagrams with reflection/refraction
- **Key Elements:**
  - Transverse/longitudinal wave propagation
  - Wavefront representations
  - Ray tracing through lenses/mirrors
  - Interference patterns

**D. Energy & Thermal Physics**
- **Visual Approach:** Particle motion and energy level diagrams
- **Animation Style:** Kinetic energy → temperature correlations
- **Key Elements:**
  - Particle speed distributions
  - Energy transfer visualizations
  - Phase change diagrams
  - Sankey diagrams for energy efficiency

## 6.3 Chemistry

### Question Type Categories:

**A. Atomic Structure & Bonding**
- **Visual Approach:** 3D molecular models using py3Dmol
- **Animation Style:** Electron shell filling, bond formation
- **Key Elements:**
  - Bohr model electron orbits
  - Lewis dot structures forming bonds
  - 3D ball-and-stick molecular models
  - Intermolecular force representations (hydrogen bonds, van der Waals)

**Example Using py3Dmol:**
```python
import py3Dmol

def create_molecule_visualization(smiles_string):
    """
    Generate 3D molecular structure from SMILES notation
    Render as frames for Manim integration
    """
    view = py3Dmol.view(width=800, height=600)
    view.addModel(Chem.MolToMolBlock(Chem.MolFromSmiles(smiles_string)), 'mol')
    view.setStyle({'stick': {}, 'sphere': {'radius': 0.3}})
    view.zoomTo()
    return view
```

**B. Chemical Reactions & Stoichiometry**
- **Visual Approach:** Particle-level reaction visualization
- **Animation Style:** Molecule collision and rearrangement
- **Key Elements:**
  - Reactants → products transformation
  - Mole ratio demonstrations with grouped particles
  - Balanced equation derivation
  - Limiting reagent visualization (one reactant consumed first)

**C. Periodic Trends**
- **Visual Approach:** Animated periodic table with property gradients
- **Animation Style:** Highlighting trends (size, electronegativity, ionization energy)
- **Key Elements:**
  - Color gradients across periods/groups
  - Atomic radius comparisons
  - Electron configuration filling order
  - Group/period property animations

**D. Acids, Bases & Salts**
- **Visual Approach:** pH scale, neutralization particle view
- **Animation Style:** H⁺/OH⁻ ion interactions
- **Key Elements:**
  - pH indicator color changes
  - Ion combination to form water
  - Salt crystal formation
  - Titration curve plotting

## 6.4 Biology

### Question Type Categories:

**A. Cell Structure & Organization**
- **Visual Approach:** Labeled cell diagrams with organelle zoom-ins
- **Animation Style:** Camera zoom into cell layers, organelle function animations
- **Key Elements:**
  - Plant vs animal cell comparisons
  - Organelle structure cutaways
  - Membrane transport (diffusion, osmosis, active transport)
  - Mitosis/meiosis cell division sequences

**Example Using VPython for 3D Cells:**
```python
from vpython import sphere, cylinder, vector, color

def create_cell_model():
    """
    Create 3D cell with organelles
    Export as image sequence for Manim
    """
    # Nucleus
    nucleus = sphere(pos=vector(0,0,0), radius=2, color=color.blue, opacity=0.7)
    
    # Mitochondria
    mito = sphere(pos=vector(3,0,0), radius=0.8, color=color.red, opacity=0.6)
    
    # Render multiple angles
    return rendered_frames
```

**B. Genetics & Inheritance**
- **Visual Approach:** Punnett squares, pedigree charts, DNA structure
- **Animation Style:** Allele combination animations, DNA replication
- **Key Elements:**
  - Chromosome diagrams with gene loci
  - Punnett square filling sequence
  - Dominant/recessive trait inheritance
  - DNA double helix with base pairing

**C. Ecology & Environment**
- **Visual Approach:** Food webs, population graphs, nutrient cycles
- **Animation Style:** Energy flow through trophic levels, cycle animations
- **Key Elements:**
  - Pyramid of numbers/biomass/energy
  - Carbon/nitrogen cycle with reservoirs
  - Population growth curves (exponential vs logistic)
  - Predator-prey oscillations

**D. Human Physiology**
- **Visual Approach:** Body system diagrams with flow indicators
- **Animation Style:** Blood flow through heart, digestive process, gas exchange
- **Key Elements:**
  - Heart cycle with valve movements
  - Lungs with alveoli gas exchange
  - Digestive system with enzyme action
  - Nervous system impulse transmission

---
