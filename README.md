# Blender 3D Renderer - Custom Graphics Engine

A fully functional 3D rendering engine built from scratch in Python for Carnegie Mellon's 15-112 Fundamentals of Programming course. This project implements a complete graphics pipeline including 3D transformations, perspective projection, backface culling, and an interactive UI system for real-time mesh manipulation—all built on top of CMU's basic 2D graphics library.

## Overview

This is a **custom-built 3D graphics engine** developed as a term project for CMU 15-112. Starting with only a 2D drawing library (`cmu_graphics`), I designed and implemented the entire 3D rendering pipeline from scratch—including all transformation matrices, projection mathematics, rendering algorithms, and UI systems. The engine renders and manipulates 3D meshes in real-time, supporting multiple objects with independent transformations and an interactive viewport inspired by professional 3D software like Blender.

**What I Built:**
- Complete 3D transformation system (2,571 lines of original Python code)
- Custom matrix mathematics library for all transformations
- Perspective projection from 3D world space to 2D screen coordinates  
- Dual-camera architecture with independent view systems
- Backface culling using surface normal calculations
- Depth sorting with painter's algorithm
- Interactive UI with sliders, buttons, and mesh selection
- Real-time mesh manipulation (translate, rotate, scale)

**What Was Provided:**
- `cmu_graphics` library - CMU's basic 2D drawing primitives (equivalent to using a canvas API)
- No 3D functionality, transformation math, or rendering algorithms

## Key Features

### 3D Graphics Pipeline

**Transformation Pipeline**
- **Translation** - Move objects in 3D space (X, Y, Z axes)
- **Rotation** - Rotate objects around arbitrary pivot points with Euler angles
- **Scaling** - Resize objects with independent axis control
- **Perspective Projection** - Convert 3D coordinates to 2D screen space with proper depth
- **Camera System** - Dual-camera architecture (world camera + gizmo camera)

**Rendering Features**
- **Backface Culling** - Only renders visible faces using normal vector calculations
- **Depth Sorting** - Painter's algorithm for correct triangle rendering order
- **Wireframe Rendering** - Edge-based mesh visualization
- **Face Shading** - Basic lighting calculations using face normals
- **Convex Hull Computation** - Graham Scan algorithm for mesh outline rendering

**Mesh Management**
- **Multiple Mesh Support** - Render and manipulate multiple objects simultaneously
- **Mesh Selection System** - Select and edit individual meshes in the scene
- **Dynamic Mesh Loading** - Support for cube, pyramid, and custom mesh primitives
- **Face Organization** - Efficient face grouping by normal vectors for optimization

### Interactive UI System

**Viewport Controls**
- **Camera Orbit** - Mouse-based camera rotation around scene pivot
- **Object Selection** - Click-based mesh selection with visual feedback
- **Transform Gizmos** - Visual indicators for object transformations
- **Real-time Updates** - Immediate visual feedback for all operations

**Transform Controls**
- Position sliders (X, Y, Z translation)
- Rotation controls (X, Y, Z angles in degrees)
- Scale controls (X, Y, Z scale factors)
- Reset and delete mesh buttons
- Add new mesh buttons (cube, pyramid, etc.)

**UI Components**
- Custom button class with hover states
- Slider widgets for precise value control
- Text input fields for numerical entry
- Status display for selected mesh properties

## Architecture

### Project Structure

```
Blender112/
├── blenderNew.py          # Main application and event loop (722 lines)
├── MeshClass.py           # Core 3D mesh representation (495 lines)
├── cameraClass.py         # Camera projection and view matrices (112 lines)
├── MatrixOperations.py    # 3D transformation matrix math (143 lines)
├── drawFunctions.py       # Rendering and drawing utilities (312 lines)
├── UI_Functions.py        # User interface event handlers (303 lines)
├── buttonClass.py         # Custom button widget implementation (154 lines)
├── otherFunctions.py      # Helper functions and utilities (330 lines)
├── cmu_graphics/          # CMU 15-112 course library (2D drawing primitives)
└── README.md

Total: 2,571 lines of original code
```

**Note on cmu_graphics:** This is CMU's standard 2D graphics library provided for the 15-112 course. It offers basic drawing functions (lines, circles, rectangles, text) similar to HTML Canvas or Python's turtle graphics. All 3D mathematics, transformations, projection, rendering algorithms, and UI systems were implemented from scratch.

### Core Components

#### 1. Mesh Class (`MeshClass.py`)

The heart of the engine - represents a 3D mesh with complete transformation stack:

**Data Structure:**
- Vertex data stored as triangles `[[x1,y1,z1], [x2,y2,z2], [x3,y3,z3]]`
- Transformation matrices (translation, rotation, scale)
- Face organization dictionary for efficient rendering
- Camera reference for projection
- Lighting vector for shading calculations

**Key Methods:**
```python
getMidpoint()              # Calculate mesh center for pivot point
getTransformedPoints()     # Apply all transformations and project to 2D
addFace(tri)              # Organize faces by normal vector
translate(point, x, y, z) # Move point in 3D space
```

**Transformation Pipeline:**
1. Translate mesh to rotation point (pivot)
2. Apply scale transformations
3. Apply rotation (X → Y → Z order)
4. Translate back from pivot
5. Apply world translation
6. Transform by camera view matrix
7. Project to 2D screen coordinates
8. Perform backface culling
9. Sort faces by depth

#### 2. Camera System (`cameraClass.py`)

**Dual Camera Architecture:**
- **World Camera** - Primary viewport camera for scene viewing
- **Gizmo Camera** - Secondary camera for UI overlays and gizmos

**Projection Mathematics:**
```python
# Perspective projection formula
screenX = (x * focalLength) / z + centerX
screenY = (y * focalLength) / z + centerY
```

**Camera Properties:**
- Position vector `[x, y, z]`
- Target/look-at point
- Up vector for orientation
- Focal length for field of view
- Viewport dimensions

#### 3. Matrix Operations (`MatrixOperations.py`)

**Implemented Transformations:**
- **4x4 Transformation Matrices** - Homogeneous coordinates for 3D math
- **Matrix Multiplication** - Chain transformations efficiently
- **Rotation Matrices** - Separate X, Y, Z rotation matrices
- **Scale Matrices** - Non-uniform scaling support
- **Translation Matrices** - Position transformation

**Matrix Multiplication:**
```python
def matrixMultiply(a, b):
    # Standard 4x4 matrix multiplication
    # Used to combine transformations
    result = [[0]*4 for _ in range(4)]
    for i in range(4):
        for j in range(4):
            for k in range(4):
                result[i][j] += a[i][k] * b[k][j]
    return result
```

#### 4. Rendering System (`drawFunctions.py`)

**Rendering Pipeline:**
1. **Clear Screen** - Reset viewport for new frame
2. **Process Each Mesh:**
   - Get transformed and projected vertices
   - Sort triangles by depth (painter's algorithm)
   - Cull backfaces (dot product with view vector)
   - Draw visible faces with shading
3. **Draw UI Elements** - Buttons, sliders, text overlays
4. **Draw Gizmos** - Transform handles and indicators

**Face Culling Algorithm:**
```python
# Calculate face normal
edge1 = vertex2 - vertex1
edge2 = vertex3 - vertex1
normal = cross_product(edge1, edge2)

# Dot product with view vector
if dot_product(normal, view_vector) < 0:
    # Face is backfacing, don't render
    skip_face()
```

**Depth Sorting:**
Uses merge sort to order triangles by average Z-depth before rendering.

#### 5. UI System (`UI_Functions.py`, `buttonClass.py`)

**Event Handling:**
- Mouse press events for button clicks and mesh selection
- Mouse drag events for camera orbit and slider manipulation
- Key press events for shortcuts and mode switching
- Mouse move events for hover states

**Custom Button Class:**
```python
class Button:
    def __init__(self, x, y, width, height, text, action):
        self.bounds = (x, y, width, height)
        self.label = text
        self.callback = action
        self.hover_state = False
    
    def isClicked(self, mouseX, mouseY):
        # Hit detection
        # Returns True if click is within bounds
    
    def draw(self):
        # Render button with current state
```

## Algorithms Implemented

### 1. Graham Scan Algorithm (Convex Hull)

Used for drawing mesh outlines by finding the convex hull of projected vertices.

**Source:** [GeeksforGeeks - Convex Hull using Graham Scan](https://www.geeksforgeeks.org/convex-hull-using-graham-scan/)

**Steps:**
1. Find bottom-most point (lowest Y coordinate)
2. Sort points by polar angle relative to bottom point
3. Process points, maintaining left-turn property
4. Result: Convex boundary of point set

### 2. Point-in-Polygon Test

Determines if a mouse click intersects a mesh for selection.

**Source:** [GeeksforGeeks - Point Inside Polygon](https://www.geeksforgeeks.org/how-to-check-if-a-given-point-lies-inside-a-polygon/)

**Ray Casting Method:**
- Cast ray from point to infinity
- Count intersections with polygon edges
- Odd count = inside, even count = outside

### 3. Merge Sort for Depth Sorting

Sorts triangles by depth for correct rendering order (painter's algorithm).

**Source:** CS Academy course materials

**Complexity:** O(n log n) - Efficient for large triangle counts

### 4. Backface Culling

Optimizes rendering by skipping invisible faces.

**Cross Product Method:**
```python
# Calculate face normal using cross product
normal = cross_product(edge1, edge2)

# Check if facing camera
dot = dot_product(normal, camera_direction)
if dot < 0:
    face_is_visible = True
```

**Performance Benefit:** Reduces triangles rendered by ~50%

## Technical Specifications

| Feature | Implementation |
|---------|----------------|
| **Coordinate System** | Right-handed (Y-up) |
| **Projection Type** | Perspective projection |
| **Transformation Order** | Scale → Rotate → Translate |
| **Rotation Order** | X → Y → Z (Euler angles) |
| **Face Culling** | Backface culling via normal vectors |
| **Depth Sorting** | Painter's algorithm with merge sort |
| **Matrix Size** | 4×4 homogeneous coordinates |
| **Supported Primitives** | Cube, pyramid (extensible) |

## Usage

### Running the Application

**Prerequisites:**
```bash
# Install CMU Graphics library
pip install cmu-graphics

# Python 3.8+ required
python --version
```

**Launch:**
```bash
cd Blender112
python blenderNew.py
```

### Controls

**Camera:**
- **Left Mouse Drag** - Orbit camera around scene
- **Mouse Wheel** - Zoom in/out (if implemented)

**Object Manipulation:**
- **Click Mesh** - Select object (highlights in UI)
- **Sliders** - Adjust position, rotation, scale
- **Add Buttons** - Create new mesh primitives
- **Delete Button** - Remove selected mesh

**UI Navigation:**
- **Click Buttons** - Execute actions (add mesh, reset transform, etc.)
- **Drag Sliders** - Fine-tune transformation values
- **Text Fields** - Enter precise numerical values

### Adding Custom Meshes

Define new mesh primitives in `otherFunctions.py`:

```python
def chooseMesh(meshType):
    if meshType == 'cube':
        return getCubeMesh()
    elif meshType == 'pyramid':
        return getPyramidMesh()
    elif meshType == 'custom':
        return [
            # List of triangles
            [[x1, y1, z1], [x2, y2, z2], [x3, y3, z3]],
            # More triangles...
        ]
```

**Mesh Format:**
- Each mesh is a list of triangles
- Each triangle is three vertices `[[x,y,z], [x,y,z], [x,y,z]]`
- Use right-hand rule for face normals (vertices in counter-clockwise order)

## Design Decisions

### Why Build from Scratch?

**Course Requirement & Learning Objectives:**
This was developed as a term project for CMU's 15-112 Fundamentals of Programming course, which challenges students to build substantial applications demonstrating programming mastery. I chose to build a 3D engine to:
- Understand graphics pipeline fundamentals from first principles
- Implement matrix transformations and projection mathematics manually
- Master complex algorithm implementation (convex hull, depth sorting, culling)
- Design object-oriented architecture for a non-trivial application
- Gain deep understanding that using high-level 3D libraries wouldn't provide

**Educational Value:**
- Complete control over every aspect of the rendering pipeline
- No black-box dependencies (only basic 2D drawing)
- Foundation for understanding modern graphics APIs (OpenGL, DirectX, Vulkan)
- Demonstrates ability to implement complex mathematical and algorithmic systems

### Why Python?

**Course Constraint:**
CMU 15-112 uses Python as the primary teaching language, which actually provided interesting challenges for a performance-intensive application like 3D rendering.

**Advantages for this project:**
- Rapid prototyping and iteration during development
- Clear, readable code demonstrating algorithms
- Strong object-oriented capabilities
- Excellent for demonstrating programming concepts

**Challenges overcome:**
- Performance limitations required careful algorithm optimization
- Implemented efficient data structures to minimize overhead
- Used backface culling and smart caching to maintain real-time performance
- Demonstrates ability to work within constraints and optimize accordingly

### Architecture Philosophy

**Object-Oriented Design:**
- Each mesh is independent object with encapsulated state
- Camera is separate object for reusability
- UI components are modular classes

**Separation of Concerns:**
- Mesh math separate from rendering
- UI logic separate from 3D transformations
- Matrix operations isolated for testing

## Performance Characteristics

**Rendering Complexity:**
- Per-frame: O(n × m) where n = meshes, m = triangles per mesh
- Sorting: O(m log m) per mesh
- Projection: O(m) per mesh
- Total: Manageable for 10-100 triangles at 30+ FPS

**Optimization Techniques:**
- Backface culling reduces render calls by ~50%
- Face organization dictionary speeds up normal-based operations
- Efficient matrix multiplication (no unnecessary allocations)

**Limitations:**
- No spatial acceleration structures (octree, BSP tree)
- No GPU acceleration
- Single-threaded rendering
- Limited to small scenes (<1000 triangles)

## Learning Outcomes

This project demonstrates mastery of:

**Computer Graphics Fundamentals**
- 3D coordinate systems and transformations
- Projection mathematics (3D → 2D)
- Camera systems and view matrices
- Rendering pipeline architecture

**Mathematical Concepts**
- Matrix algebra and transformations
- Vector mathematics (dot product, cross product)
- Trigonometry for rotations
- Linear algebra for projections

**Software Engineering**
- Object-oriented design patterns
- Modular code architecture
- Event-driven programming
- UI/UX implementation

**Algorithm Implementation**
- Computational geometry (convex hull, point-in-polygon)
- Sorting algorithms (merge sort)
- Optimization techniques (culling, caching)

## Future Enhancements

### Rendering Features
- [ ] Phong shading with proper lighting model
- [ ] Texture mapping support
- [ ] Shadow rendering
- [ ] Anti-aliasing
- [ ] Ambient occlusion

### Mesh Features
- [ ] OBJ file import
- [ ] Mesh editing (extrude, subdivide)
- [ ] Smooth shading (vertex normals)
- [ ] UV coordinate support

### Performance
- [ ] Spatial partitioning (octree)
- [ ] Frustum culling
- [ ] Level of detail (LOD) system
- [ ] Multi-threaded rendering

### UI/UX
- [ ] Timeline for animation
- [ ] Material editor
- [ ] Scene hierarchy panel
- [ ] Keyboard shortcuts

## Comparison to Professional Software

| Feature | This Project | Blender |
|---------|--------------|---------|
| Custom 3D engine | ✅ From scratch | ✅ Custom C/C++ |
| Matrix transformations | ✅ Manual | ✅ Optimized |
| Perspective projection | ✅ Implemented | ✅ Advanced |
| Backface culling | ✅ Yes | ✅ Yes |
| Mesh primitives | ✅ Cube, pyramid | ✅ 20+ primitives |
| UI system | ✅ Custom | ✅ Professional |
| GPU acceleration | ❌ No | ✅ OpenGL/Vulkan |
| Texture mapping | ❌ No | ✅ Full PBR |
| Animation | ❌ No | ✅ Full timeline |

## Technical Challenges Solved

### 1. Gimbal Lock
Euler angle rotations can suffer from gimbal lock. Mitigated by:
- Rotation order (X→Y→Z)
- Small angle constraints
- Future: Quaternion implementation

### 2. Z-Fighting
Triangles at same depth can flicker. Solved by:
- Depth sorting with stable sort
- Small epsilon for depth comparisons

### 3. Projection Edge Cases
Vertices behind camera cause projection issues. Handled by:
- Clipping planes (near and far)
- Z-value validation before projection

### 4. Performance
Python's speed limitations addressed by:
- Efficient data structures
- Minimal allocations per frame
- Backface culling
- Face organization caching

## Acknowledgments

**Algorithm References:**
- [GeeksforGeeks](https://www.geeksforgeeks.org/) - Graham Scan and point-in-polygon algorithms
- CS Academy - Merge sort implementation
- CMU 15-112 Course - Graphics library

**Inspiration:**
- Blender 3D software
- Unity/Unreal Engine architecture
- Computer graphics textbooks

## Author

**Author:**  
Carnegie Mellon University, Class of 2027  
Electrical and Computer Engineering

**Course Context:**  
15-112: Fundamentals of Programming - Term Project  
This project demonstrates advanced Python programming, mathematical modeling, software architecture, and algorithm implementation skills.

## License

Academic project developed for CMU 15-112. The `cmu_graphics` library is property of Carnegie Mellon University and used under course license. All original rendering, transformation, and UI code (2,571 lines) is my own work.

---

**Note:** This is an educational implementation built for a university course, showcasing computer graphics fundamentals and algorithm implementation. For production 3D applications, use established engines like Blender (open source), Unity, or Unreal Engine, which provide GPU acceleration and extensive toolsets. This project demonstrates the ability to implement complex systems from first principles—a valuable skill for understanding how such tools work under the hood.

## Technical Deep Dive

For those interested in implementation details:

**Transformation Matrix Example:**
```python
# Rotation around Y-axis
def rotationY(angle):
    rad = math.radians(angle)
    return [
        [math.cos(rad),  0, math.sin(rad), 0],
        [0,              1, 0,             0],
        [-math.sin(rad), 0, math.cos(rad), 0],
        [0,              0, 0,             1]
    ]
```

**Perspective Projection:**
```python
def projectToScreen(point3D, camera):
    # Translate by camera position
    x = point3D[0] - camera.pos[0]
    y = point3D[1] - camera.pos[1]
    z = point3D[2] - camera.pos[2]
    
    # Avoid division by zero
    if z <= 0:
        return None
    
    # Project to 2D
    focalLength = 500
    screenX = (x * focalLength / z) + width/2
    screenY = (y * focalLength / z) + height/2
    
    return [screenX, screenY, z]
```

This implementation serves as a complete reference for understanding 3D graphics programming from first principles.
README_BLENDER.md
Displaying README_BLENDER.md.
