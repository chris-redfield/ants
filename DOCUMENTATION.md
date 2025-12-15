# Ant Colony Simulation - Complete Documentation

A visual simulation demonstrating emergent swarm intelligence through simple individual rules that create complex collective behavior.

---

## Table of Contents

1. [Overview](#overview)
2. [Features](#features)
3. [Architecture](#architecture)
4. [HTML Structure](#html-structure)
5. [CSS Styling](#css-styling)
6. [JavaScript Documentation](#javascript-documentation)
   - [Canvas Setup](#canvas-setup)
   - [Configuration Constants](#configuration-constants)
   - [Simulation State](#simulation-state)
   - [Core Functions](#core-functions)
   - [Ant Class](#ant-class)
   - [Rendering Functions](#rendering-functions)
   - [Event Handlers](#event-handlers)
   - [Game Loop](#game-loop)
7. [Algorithms](#algorithms)
8. [How It All Works Together](#how-it-all-works-together)

---

## Overview

This is an interactive visual ant colony simulation built as a single-page web application. It models realistic ant colony behavior where individual ants follow simple rules that create complex collective behavior through chemical communication (pheromones).

**Key Concepts:**
- **Emergent Behavior**: Complex patterns arising from simple individual rules
- **Swarm Intelligence**: Collective problem-solving without central coordination
- **Pheromone Communication**: Chemical trails for navigation and information sharing

---

## Features

| Feature | Description |
|---------|-------------|
| **Pheromone System** | Two trail types: home (blue) and food (orange) with decay and diffusion |
| **Pheromone Sensing** | 3-point sensory system (left/center/right) for trail following |
| **Interactive Controls** | Click to spawn food, adjust ant count (20-500), control speed |
| **Two Ant Styles** | Classic simple ant vs. detailed termite rendering |
| **Real-time Stats** | Ant count, food collected, active sources, colony size |
| **Organic Food Shapes** | Irregular polygons with 3-8 sides in various colors |
| **3D Colony Mound** | Multi-layer gradient ant hill with entrance hole |
| **Responsive Design** | Works on mobile and desktop devices |

---

## Architecture

### Technology Stack

- **HTML5**: Structure and semantic markup
- **CSS3**: Modern styling with gradients, flexbox, backdrop blur
- **Vanilla JavaScript**: Core simulation logic (no dependencies)
- **HTML5 Canvas API**: Multi-layer rendering system

### Three-Layer Canvas System

```
┌─────────────────────────────────────┐
│         ant-canvas (z-index: 3)     │  ← Ants, colony, food sources
├─────────────────────────────────────┤
│     pheromone-canvas (z-index: 2)   │  ← Pheromone trails
├─────────────────────────────────────┤
│        bg-canvas (z-index: 1)       │  ← Grass background texture
└─────────────────────────────────────┘
```

This separation allows:
- Background to be drawn once (performance optimization)
- Pheromones to update independently with pixel manipulation
- Ants to be drawn on top without clearing trails

---

## HTML Structure

### Document Head (Lines 1-230)
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Ant Colony Simulation</title>
    <style>/* CSS styles */</style>
</head>
```

### Body Structure (Lines 231-299)

| Element | Purpose |
|---------|---------|
| `<header>` | Title and subtitle |
| `#game-container` | Main container for canvas wrapper |
| `#canvas-wrapper` | Holds all three canvas layers |
| `.stats` | Real-time statistics display (top-left) |
| `.legend` | Color legend for visual elements (top-right) |
| `.instructions` | User instructions |
| `.controls` | Control panel with sliders and buttons |

### Canvas Elements
```html
<canvas id="bg-canvas"></canvas>        <!-- Background grass texture -->
<canvas id="pheromone-canvas"></canvas> <!-- Pheromone trails -->
<canvas id="ant-canvas"></canvas>       <!-- Colony, ants, food -->
```

### Control Elements
```html
<input type="range" id="ant-slider" min="20" max="500" value="150">   <!-- Ant count -->
<input type="range" id="speed-slider" min="1" max="5" value="2">      <!-- Simulation speed -->
<input type="range" id="food-slider" min="0" max="15" value="5">      <!-- Food sources -->
<button id="toggle-trails">On</button>                                 <!-- Trail visibility -->
<button id="toggle-style">Ant</button>                                 <!-- Ant rendering style -->
<button id="reset-btn">↺</button>                                     <!-- Reset simulation -->
```

---

## CSS Styling

### Color Scheme
```css
Primary Background: #1a1a2e (dark blue-gray)
Accent Color:       #f4a261 (orange)
Grass Color:        #2d5a27 (green)
Text Color:         #e0e0e0 (light gray)
```

### Key Styling Features

**Controls Panel** (Lines 87-102):
- Fixed position at bottom center
- Semi-transparent background with backdrop blur
- Orange accent border

**Responsive Breakpoints** (Lines 196-228):
- `@media (max-width: 600px)` - Mobile adjustments for padding, font sizes

---

## JavaScript Documentation

### Canvas Setup
**Location:** Lines 301-307

```javascript
const bgCanvas = document.getElementById('bg-canvas');
const pheromoneCanvas = document.getElementById('pheromone-canvas');
const antCanvas = document.getElementById('ant-canvas');
const bgCtx = bgCanvas.getContext('2d');
const pheromoneCtx = pheromoneCanvas.getContext('2d');
const antCtx = antCanvas.getContext('2d');
```

Three separate canvas elements with their 2D rendering contexts for layered drawing.

---

### Configuration Constants
**Location:** Lines 309-334

| Constant | Value | Description |
|----------|-------|-------------|
| `CELL_SIZE` | `4` | Pheromone grid cell size in pixels |
| `ANT_SPEED` | `2` | Pixels per frame movement |
| `PHEROMONE_DEPOSIT` | `100` | Base pheromone strength when depositing |
| `PHEROMONE_DECAY` | `0.995` | General decay rate (not used directly) |
| `PHEROMONE_DIFFUSE` | `0.1` | Diffusion rate to neighboring cells |
| `WANDER_STRENGTH` | `0.3` | Random movement factor |
| `PHEROMONE_FOLLOW_STRENGTH` | `0.8` | How strongly ants follow trails |

---

### Simulation State
**Location:** Lines 314-326

```javascript
let ants = [];           // Array of Ant objects
let foods = [];          // Array of food source objects
let colony = {           // Colony object
    x: 0,                // Center X position
    y: 0,                // Center Y position
    radius: 25,          // Colony radius
    foodStored: 0        // Total food collected
};
let homePheromones = []; // Float32Array - trails back to colony
let foodPheromones = []; // Float32Array - trails to food sources
let showTrails = true;   // Toggle trail visibility
let simulationSpeed = 2; // Updates per frame (1-5)
let targetAntCount = 150;// Desired ant population
let targetFoodCount = 5; // Desired food sources
let antStyle = 'classic';// 'classic' or 'detailed'
let colonyGranules = []; // Stored granule positions for colony rendering
```

---

### Core Functions

#### `resize()`
**Location:** Lines 335-360
**Purpose:** Handles canvas sizing and pheromone grid initialization

```javascript
function resize() {
    // 1. Calculate dimensions based on container
    const maxWidth = Math.min(container.clientWidth - 40, 900);
    const maxHeight = Math.min(container.clientHeight - 40, 700);

    // 2. Snap to cell size for clean pheromone grid
    width = Math.floor(maxWidth / CELL_SIZE) * CELL_SIZE;
    height = Math.floor(maxHeight / CELL_SIZE) * CELL_SIZE;

    // 3. Calculate grid dimensions
    gridWidth = Math.floor(width / CELL_SIZE);
    gridHeight = Math.floor(height / CELL_SIZE);

    // 4. Resize all canvases
    // 5. Initialize pheromone grids as Float32Array
    // 6. Position colony at center
    // 7. Draw background
}
```

**Key Details:**
- Canvas dimensions snapped to `CELL_SIZE` multiples for clean grid alignment
- Pheromone grids use `Float32Array` for performance with continuous values
- Colony always centered in canvas

---

#### `drawBackground()`
**Location:** Lines 362-398
**Purpose:** Renders static grass texture on background canvas

```javascript
function drawBackground() {
    // 1. Fill base grass color (#2d5a27)

    // 2. Add 500 random texture patches
    //    - Lighter green: rgba(60, 120, 50, ...)
    //    - Darker green:  rgba(20, 60, 15, ...)
    //    - Elliptical shapes for organic look

    // 3. Add 200 grass blade hints
    //    - Short angled lines pointing upward
}
```

**Rendering Details:**
- Uses ellipses with random rotation for natural patches
- Grass blades drawn at near-vertical angles with slight variation

---

#### `init()`
**Location:** Lines 708-743
**Purpose:** Initializes/resets the entire simulation

```javascript
function init() {
    resize();

    // Create ants at colony position with random spread
    ants = [];
    for (let i = 0; i < targetAntCount; i++) {
        const angle = Math.random() * Math.PI * 2;
        const dist = Math.random() * colony.radius;
        ants.push(new Ant(
            colony.x + Math.cos(angle) * dist,
            colony.y + Math.sin(angle) * dist
        ));
    }

    // Create initial food sources
    foods = [];
    for (let i = 0; i < targetFoodCount; i++) {
        spawnRandomFood();
    }

    // Reset colony food counter
    colony.foodStored = 0;

    // Generate static colony dirt granules
    colonyGranules = [];
    // ... (30 granules with random positions/sizes/colors)
}
```

---

#### `createFood(x, y)`
**Location:** Lines 654-693
**Purpose:** Creates a food source at specified coordinates

```javascript
function createFood(x, y) {
    // 1. Generate random polygon (3-8 sides)
    const sides = 3 + Math.floor(Math.random() * 6);
    const baseRadius = 12 + Math.random() * 12;

    // 2. Create irregular vertices
    const vertices = [];
    for (let i = 0; i < sides; i++) {
        const angle = (i / sides) * Math.PI * 2 - Math.PI / 2;
        const r = baseRadius * (0.7 + Math.random() * 0.6);
        vertices.push({ x: Math.cos(angle) * r, y: Math.sin(angle) * r });
    }

    // 3. Select random food type
    const foodTypes = [
        { color: '#4ade80', highlight: '#86efac', name: 'leaf' },
        { color: '#fbbf24', highlight: '#fcd34d', name: 'seed' },
        { color: '#f87171', highlight: '#fca5a5', name: 'berry' },
        { color: '#a78bfa', highlight: '#c4b5fd', name: 'flower' },
        { color: '#fb923c', highlight: '#fdba74', name: 'crumb' },
    ];

    // 4. Push food object to array
    foods.push({
        x, y, radius: baseRadius, vertices, sides,
        amount: 20 + Math.floor(Math.random() * 40),
        maxAmount: 60, color, highlight, rotation
    });
}
```

**Food Object Structure:**
```javascript
{
    x: number,           // X position
    y: number,           // Y position
    radius: number,      // Base radius (12-24)
    vertices: [{x, y}],  // Polygon points
    sides: number,       // Number of sides (3-8)
    amount: number,      // Current food units (20-60)
    maxAmount: number,   // Maximum (60)
    color: string,       // Main color
    highlight: string,   // Highlight color
    rotation: number     // Random rotation angle
}
```

---

#### `spawnRandomFood()`
**Location:** Lines 695-706
**Purpose:** Creates food at random valid location

```javascript
function spawnRandomFood() {
    const margin = 80;
    let x, y, attempts = 0;

    // Find position not too close to colony
    do {
        x = margin + Math.random() * (width - margin * 2);
        y = margin + Math.random() * (height - margin * 2);
        attempts++;
    } while (
        Math.sqrt((x - colony.x) ** 2 + (y - colony.y) ** 2) < 100
        && attempts < 20
    );

    createFood(x, y);
}
```

**Constraints:**
- 80px margin from edges
- Minimum 100px from colony center
- Maximum 20 attempts before placing anyway

---

#### `updatePheromones()`
**Location:** Lines 745-782
**Purpose:** Handles pheromone decay and diffusion

```javascript
function updatePheromones() {
    const newHome = new Float32Array(gridWidth * gridHeight);
    const newFood = new Float32Array(gridWidth * gridHeight);

    const HOME_DECAY = 0.998;  // Slower decay - trails persist longer
    const FOOD_DECAY = 0.994;  // Faster decay - encourages exploration

    for (let y = 0; y < gridHeight; y++) {
        for (let x = 0; x < gridWidth; x++) {
            const idx = y * gridWidth + x;

            // Self contribution (90%)
            let homeSum = homePheromones[idx] * (1 - PHEROMONE_DIFFUSE);
            let foodSum = foodPheromones[idx] * (1 - PHEROMONE_DIFFUSE);

            // Neighbor contribution (10% split among 8 neighbors)
            for (let dy = -1; dy <= 1; dy++) {
                for (let dx = -1; dx <= 1; dx++) {
                    if (dx === 0 && dy === 0) continue;
                    // Add neighbor's contribution
                    homeSum += neighbor * PHEROMONE_DIFFUSE / 8;
                    foodSum += neighbor * PHEROMONE_DIFFUSE / 8;
                }
            }

            // Apply decay
            newHome[idx] = homeSum * HOME_DECAY;
            newFood[idx] = foodSum * FOOD_DECAY;
        }
    }

    // Swap buffers
    homePheromones = newHome;
    foodPheromones = newFood;
}
```

**Decay Rates:**
- Home trails: 0.998 (decays ~0.2% per frame) - persistent for return navigation
- Food trails: 0.994 (decays ~0.6% per frame) - fades faster to encourage new exploration

---

#### `update()`
**Location:** Lines 947-981
**Purpose:** Main simulation update tick

```javascript
function update() {
    // Run multiple times based on speed setting
    for (let i = 0; i < simulationSpeed; i++) {
        // 1. Update all ants
        for (let ant of ants) {
            ant.update();
        }

        // 2. Update pheromones (decay/diffuse)
        updatePheromones();

        // 3. Remove depleted food sources
        foods = foods.filter(f => f.amount > 0);

        // 4. Maintain target food count
        while (foods.length < targetFoodCount) spawnRandomFood();
        while (foods.length > targetFoodCount) foods.pop();

        // 5. Maintain target ant count
        while (ants.length < targetAntCount) {
            // Spawn new ant at colony
        }
        while (ants.length > targetAntCount) ants.pop();
    }
}
```

---

### Ant Class
**Location:** Lines 400-652

#### Constructor
```javascript
class Ant {
    constructor(x, y) {
        this.x = x;                              // X position
        this.y = y;                              // Y position
        this.angle = Math.random() * Math.PI * 2; // Movement direction
        this.hasFood = false;                    // Carrying food flag
        this.foodColor = null;                   // Color of carried food
        this.pheromoneTimer = 0;                 // Deposit timing
        this.wanderAngle = 0;                    // Accumulated wander
        this.distanceTraveled = 0;               // Distance since state change
        this.maxPheromoneStrength = PHEROMONE_DEPOSIT; // Starting strength
    }
}
```

#### `update()` Method
**Location:** Lines 413-542
**Purpose:** Core ant AI - state machine with two states

**State Machine:**
```
┌─────────────┐     Found Food      ┌─────────────┐
│  SEARCHING  │ ──────────────────► │  RETURNING  │
│  (hasFood   │                     │  (hasFood   │
│   = false)  │ ◄────────────────── │   = true)   │
└─────────────┘   Reached Colony    └─────────────┘
```

**SEARCHING State (Lines 463-511):**
```javascript
// 1. Check for nearby food
for (let food of foods) {
    const dist = distance(this, food);
    if (dist < food.radius * 0.7 && food.amount > 0) {
        this.hasFood = true;
        this.foodColor = food.color;
        food.amount--;
        this.angle = Math.atan2(colony.y - this.y, colony.x - this.x);
        break;
    }
}

// 2. If no food found, follow pheromones or wander
if (!this.hasFood) {
    // Sense pheromones at three angles
    const left = sensePheromone(foodPheromones, angle - π/4, 12);
    const center = sensePheromone(foodPheromones, angle, 12);
    const right = sensePheromone(foodPheromones, angle + π/4, 12);

    if (maxStrength > 2) {
        // Follow strongest pheromone direction
    } else {
        // Random exploration
    }
}
```

**RETURNING State (Lines 435-462):**
```javascript
// 1. Calculate distance to colony
const dx = colony.x - this.x;
const dy = colony.y - this.y;
const dist = Math.sqrt(dx * dx + dy * dy);

// 2. Check if reached colony
if (dist < colony.radius + 5) {
    this.hasFood = false;
    this.foodColor = null;
    colony.foodStored++;
    this.angle += Math.PI; // Turn around
    this.distanceTraveled = 0;
} else {
    // Steer towards colony with slight wander
    const targetAngle = Math.atan2(dy, dx);
    this.angle += angleDiff * 0.2;
    this.angle += random(-0.05, 0.05);
}
```

**Pheromone Deposition (Lines 415-433):**
```javascript
// Every 3 frames
if (this.pheromoneTimer > 2) {
    const strength = max(10, PHEROMONE_DEPOSIT - distanceTraveled * 0.3);

    if (this.hasFood) {
        // Leave FOOD trail (for others to find food)
        foodPheromones[idx] += strength;
    } else {
        // Leave HOME trail (to find way back)
        const homeStrength = max(20, PHEROMONE_DEPOSIT * 1.5 - distToColony * 0.2);
        homePheromones[idx] += homeStrength;
    }
}
```

**Movement & Collision (Lines 514-541):**
```javascript
// Move in current direction
this.x += Math.cos(this.angle) * ANT_SPEED;
this.y += Math.sin(this.angle) * ANT_SPEED;
this.distanceTraveled += ANT_SPEED;

// Bounce off walls with margin
const margin = 8;
if (this.x < margin) {
    this.x = margin;
    this.angle = Math.PI - this.angle + random(-0.25, 0.25);
}
// ... similar for other edges
```

#### `sensePheromone(pheromones, angle, distance)` Method
**Location:** Lines 544-556

```javascript
sensePheromone(pheromones, angle, distance) {
    let total = 0;
    // Sample at intervals of 4 pixels
    for (let d = 4; d <= distance; d += 4) {
        const sx = this.x + Math.cos(angle) * d;
        const sy = this.y + Math.sin(angle) * d;
        const gx = Math.floor(sx / CELL_SIZE);
        const gy = Math.floor(sy / CELL_SIZE);
        if (inBounds(gx, gy)) {
            total += pheromones[gy * gridWidth + gx];
        }
    }
    return total;
}
```

**How It Works:**
- Samples pheromone values along a ray from ant position
- Samples every 4 pixels up to `distance` (12 pixels = 3 samples)
- Returns total accumulated pheromone strength

#### `draw(ctx)` Method
**Location:** Lines 558-651

**Classic Style (Lines 562-583):**
```javascript
// Simple brown ant
ctx.fillStyle = '#5c4033';

// Body - single oval
ctx.ellipse(0, 0, 5, 3, 0, 0, Math.PI * 2);

// Head - circle
ctx.arc(5, 0, 2, 0, Math.PI * 2);

// Food (if carrying)
if (this.hasFood) {
    ctx.fillStyle = this.foodColor;
    ctx.arc(-5, 0, 2.5, 0, Math.PI * 2);
}
```

**Detailed Style (Lines 584-648):**
```javascript
// Dark brown termite-style
const antColor = '#3d2817';

// Animated legs (6 total)
const legWiggle = Math.sin(this.distanceTraveled * 0.5) * 0.2;
for (let i = -1; i <= 1; i++) {
    // Draw left and right legs
}

// Three-part body: Abdomen, Thorax, Head
ctx.ellipse(-3, 0, 3, 2.2, ...);   // Abdomen
ctx.ellipse(1, 0, 2, 1.5, ...);    // Thorax
ctx.ellipse(4, 0, 1.8, 1.5, ...);  // Head

// Curved antennae
ctx.quadraticCurveTo(7, -2, 8, -1);
ctx.quadraticCurveTo(7, 2, 8, 1);
```

---

### Rendering Functions

#### `drawPheromones()`
**Location:** Lines 784-822
**Purpose:** Renders pheromone trails using pixel manipulation

```javascript
function drawPheromones() {
    pheromoneCtx.clearRect(0, 0, width, height);
    if (!showTrails) return;

    // Create ImageData for direct pixel access
    const imageData = pheromoneCtx.createImageData(width, height);
    const data = imageData.data;

    for (let gy = 0; gy < gridHeight; gy++) {
        for (let gx = 0; gx < gridWidth; gx++) {
            const home = homePheromones[idx];
            const food = foodPheromones[idx];

            if (home > 1 || food > 1) {
                // Fill 4x4 pixel cell
                for (let py = 0; py < CELL_SIZE; py++) {
                    for (let px = 0; px < CELL_SIZE; px++) {
                        // Calculate pixel index
                        const pixelIdx = (pixelY * width + pixelX) * 4;

                        // Blend colors:
                        // Home = Blue (77, 166, 255)
                        // Food = Orange (255, 180, 50)
                        data[pixelIdx] = homeBlue.r + foodOrange.r;
                        data[pixelIdx + 1] = homeBlue.g + foodOrange.g;
                        data[pixelIdx + 2] = homeBlue.b + foodOrange.b;
                        data[pixelIdx + 3] = alpha;
                    }
                }
            }
        }
    }

    pheromoneCtx.putImageData(imageData, 0, 0);
}
```

**Color Mapping:**
| Trail Type | RGB Values | Visual |
|------------|------------|--------|
| Home | (77, 166, 255) | Blue |
| Food | (255, 180, 50) | Orange |

---

#### `drawAnts()`
**Location:** Lines 824-938
**Purpose:** Renders colony, food sources, and ants

**Colony Mound (Lines 828-880):**
```javascript
// 1. Draw 6 gradient layers for 3D effect
for (let i = 5; i >= 0; i--) {
    const gradient = createRadialGradient(...);
    // Sandy/dirt colors with varying brightness
    gradient.addColorStop(0, lighterBrown);
    gradient.addColorStop(0.6, darkerBrown);
    gradient.addColorStop(1, transparent);
    ctx.arc(colony.x, colony.y - offset, radius, 0, Math.PI * 2);
}

// 2. Draw stored dirt granules
for (let granule of colonyGranules) {
    ctx.arc(granule.x, granule.y, granule.size, ...);
}

// 3. Draw dark entrance hole with gradient
const holeGradient = createRadialGradient(...);
holeGradient.addColorStop(0, 'rgba(20, 15, 10, 1)');  // Dark center
holeGradient.addColorStop(1, transparent);
ctx.ellipse(colony.x, colony.y, radius * 0.6, radius * 0.45, ...);
```

**Food Sources (Lines 882-932):**
```javascript
for (let food of foods) {
    if (food.amount > 0) {
        // Scale based on remaining amount
        const scale = 0.4 + (food.amount / food.maxAmount) * 0.6;

        ctx.save();
        ctx.translate(food.x, food.y);
        ctx.rotate(food.rotation);
        ctx.scale(scale, scale);

        // 1. Glow effect (radial gradient)
        // 2. Main polygon shape
        // 3. Highlight circle
        // 4. Outline stroke

        ctx.restore();
    }
}
```

**Ants (Lines 934-937):**
```javascript
for (let ant of ants) {
    ant.draw(antCtx);
}
```

---

#### `updateStats()`
**Location:** Lines 940-945
**Purpose:** Updates DOM statistics display

```javascript
function updateStats() {
    document.getElementById('ant-count').textContent = ants.length;
    document.getElementById('food-collected').textContent = colony.foodStored;
    document.getElementById('food-sources').textContent = foods.filter(f => f.amount > 0).length;
    document.getElementById('colony-size').textContent = Math.floor(1 + colony.foodStored / 50);
}
```

**Colony Size Calculation:**
- Starts at 1
- Increases by 1 for every 50 food collected

---

### Event Handlers
**Location:** Lines 995-1046

#### Canvas Click/Touch (Lines 996-1012)
```javascript
// Click to add food
antCanvas.addEventListener('click', (e) => {
    const rect = antCanvas.getBoundingClientRect();
    const x = e.clientX - rect.left;
    const y = e.clientY - rect.top;
    createFood(x, y);
});

// Touch support for mobile
antCanvas.addEventListener('touchstart', (e) => {
    e.preventDefault();
    // Same coordinate calculation
    createFood(x, y);
});
```

#### Slider Controls (Lines 1014-1024)
```javascript
// Ant count slider
document.getElementById('ant-slider').addEventListener('input', (e) => {
    targetAntCount = parseInt(e.target.value);
});

// Speed slider
document.getElementById('speed-slider').addEventListener('input', (e) => {
    simulationSpeed = parseInt(e.target.value);
});

// Food slider
document.getElementById('food-slider').addEventListener('input', (e) => {
    targetFoodCount = parseInt(e.target.value);
});
```

#### Toggle Buttons (Lines 1026-1035)
```javascript
// Trail visibility toggle
document.getElementById('toggle-trails').addEventListener('click', (e) => {
    showTrails = !showTrails;
    e.target.textContent = showTrails ? 'On' : 'Off';
    e.target.classList.toggle('active', showTrails);
});

// Ant style toggle
document.getElementById('toggle-style').addEventListener('click', (e) => {
    antStyle = antStyle === 'classic' ? 'detailed' : 'classic';
    e.target.textContent = antStyle === 'classic' ? 'Ant' : 'Termite';
});
```

#### Reset Button (Lines 1037-1041)
```javascript
document.getElementById('reset-btn').addEventListener('click', () => {
    homePheromones.fill(0);  // Clear all pheromones
    foodPheromones.fill(0);
    init();                   // Reinitialize simulation
});
```

#### Window Resize (Lines 1043-1046)
```javascript
window.addEventListener('resize', () => {
    resize();
    drawBackground();
});
```

---

### Game Loop
**Location:** Lines 983-993

```javascript
function render() {
    drawPheromones();
    drawAnts();
    updateStats();
}

function gameLoop() {
    update();   // Simulation logic
    render();   // Draw everything
    requestAnimationFrame(gameLoop);  // Schedule next frame
}

// Start
init();
gameLoop();
```

**Frame Rate:** ~60 FPS (browser's refresh rate via `requestAnimationFrame`)

---

## Algorithms

### Pheromone Following Algorithm

Ants use a three-sensor system to follow pheromone trails:

```
           [Left Sensor]
              ↖ (π/4)

    [Ant] ──→ [Center Sensor]

              ↘ (π/4)
           [Right Sensor]
```

**Decision Logic:**
1. If `maxStrength > 2` (pheromones detected):
   - If center strongest: continue forward with minor wander
   - If left strongest: turn left (`wanderAngle -= 0.8`)
   - If right strongest: turn right (`wanderAngle += 0.8`)
2. If no pheromones: random exploration (`wanderAngle += random * 0.6`)

### Pheromone Diffusion Algorithm

Uses a simple blur/diffusion kernel:

```
[1/8] [1/8] [1/8]
[1/8] [0.9] [1/8]  × decay_rate
[1/8] [1/8] [1/8]
```

- 90% of value stays in cell
- 10% spreads equally to 8 neighbors
- Result multiplied by decay rate (0.998 or 0.994)

### Pheromone Strength Decay by Distance

```javascript
// Food trails: strength decreases as ant travels
strength = max(10, PHEROMONE_DEPOSIT - distanceTraveled * 0.3)

// Home trails: strength decreases with distance from colony
homeStrength = max(20, PHEROMONE_DEPOSIT * 1.5 - distToColony * 0.2)
```

This creates stronger trails near important locations (colony and food sources).

---

## How It All Works Together

### Initialization Flow
```
1. resize()
   ├── Calculate canvas dimensions
   ├── Initialize pheromone Float32Arrays
   └── Position colony at center

2. init()
   ├── Create ant population at colony
   ├── Spawn food sources
   └── Generate colony granules

3. gameLoop()
   └── Start animation loop
```

### Main Loop Flow (60 FPS)
```
┌─────────────────────────────────────────────────────────────┐
│                       gameLoop()                             │
├─────────────────────────────────────────────────────────────┤
│  update() × simulationSpeed                                 │
│  ├── For each ant: ant.update()                             │
│  │   ├── Deposit pheromones                                 │
│  │   ├── State machine (SEARCHING/RETURNING)                │
│  │   ├── Movement                                           │
│  │   └── Wall collision                                     │
│  ├── updatePheromones() (decay + diffuse)                   │
│  ├── Remove depleted food                                   │
│  ├── Maintain food count                                    │
│  └── Maintain ant count                                     │
├─────────────────────────────────────────────────────────────┤
│  render()                                                   │
│  ├── drawPheromones() → pheromone-canvas                    │
│  ├── drawAnts() → ant-canvas                                │
│  │   ├── Colony mound                                       │
│  │   ├── Food sources                                       │
│  │   └── Individual ants                                    │
│  └── updateStats() → DOM                                    │
├─────────────────────────────────────────────────────────────┤
│  requestAnimationFrame(gameLoop)                            │
└─────────────────────────────────────────────────────────────┘
```

### Emergent Behavior

The simulation demonstrates **stigmergy** - indirect coordination through environmental modification:

1. **Trail Formation**: Random exploring ants eventually find food
2. **Trail Reinforcement**: Successful ants leave strong food trails
3. **Positive Feedback**: More ants follow strong trails, making them stronger
4. **Natural Decay**: Old/unused trails fade, adapting to food depletion
5. **Network Optimization**: Over time, efficient paths emerge naturally

No ant has global knowledge - each follows simple local rules, yet the colony exhibits intelligent collective behavior.

---

## File Summary

| Section | Lines | Description |
|---------|-------|-------------|
| HTML Head | 1-230 | Document setup, CSS styles |
| HTML Body | 231-299 | DOM structure, controls |
| Canvas Setup | 301-307 | Canvas element references |
| Constants | 309-334 | Configuration values |
| resize() | 335-360 | Canvas/grid initialization |
| drawBackground() | 362-398 | Grass texture rendering |
| Ant class | 400-652 | Ant AI and rendering |
| createFood() | 654-693 | Food source creation |
| spawnRandomFood() | 695-706 | Random food placement |
| init() | 708-743 | Simulation initialization |
| updatePheromones() | 745-782 | Decay and diffusion |
| drawPheromones() | 784-822 | Trail rendering |
| drawAnts() | 824-938 | Main canvas rendering |
| updateStats() | 940-945 | DOM statistics update |
| update() | 947-981 | Main simulation tick |
| render() | 983-987 | Render orchestration |
| gameLoop() | 989-993 | Animation loop |
| Event handlers | 995-1046 | User interaction |
| Bootstrap | 1048-1050 | init() and gameLoop() calls |

**Total: 1054 lines** (HTML + CSS + JavaScript in single file)
