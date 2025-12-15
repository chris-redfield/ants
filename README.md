# Ant Colony Simulation

A visual simulation demonstrating emergent swarm intelligence. Watch ants form trails, find food, and compete across multiple colonies!

## Features

- **Menu System** - Choose Single Colony or Multiple Colonies mode (2-4 colonies)
- **Colony Colors** - Orange, Blue, Green, Purple with matching pheromone trails
- **Pheromone System** - Per-colony trails with decay and diffusion
- **Interactive Controls** - Click to add food, adjust ant count, speed, and more
- **Two Ant Styles** - Classic simple ant or detailed termite rendering
- **Real-time Stats** - FPS, ant count, food collected
- **Browser Zoom** - Zoom in and scroll to navigate

## How to Play

1. Open `index.html` in a browser
2. Select **Single Colony** or **Multiple Colonies**
3. For multi-colony, choose 2-4 colonies
4. Click **Start Simulation**
5. Click anywhere to add food sources
6. Watch the emergent behavior!

## Controls

| Control | Action |
|---------|--------|
| Click on canvas | Add food source |
| Ant slider | Adjust ant count (balanced across colonies) |
| Speed slider | Simulation speed (1-5x) |
| Food slider | Target food sources |
| Trails button | Toggle pheromone visibility |
| Style button | Toggle ant/termite rendering |
| Reset button | Clear and restart |
| Browser zoom | Zoom in/out, scroll to navigate |

## Game Modes

### Single Colony
- One colony in the center
- Classic brown ants and orange trails

### Multiple Colonies
- 2-4 colonies in corners
- Each colony has unique colors and independent pheromone trails
- Colonies compete for shared food sources

## Technical Overview

### Architecture
- **Single HTML file** with embedded CSS and JavaScript
- **Three-layer canvas system**: background, pheromones, ants
- **No dependencies** - vanilla JavaScript

### Performance Optimizations
- **ImageData + offscreen canvas** for pheromone rendering
- **Per-colony pheromone grids** with efficient diffusion algorithm
- **FPS counter** to monitor performance

### Key Algorithms
- **Pheromone following**: 3-sensor system (left/center/right)
- **Pheromone diffusion**: 90% self, 10% spread to neighbors
- **Decay rates**: Home trails persist longer than food trails

## Emergent Behavior

The simulation demonstrates **stigmergy** - indirect coordination through environmental modification:

1. Ants explore randomly and find food
2. Successful ants leave strong pheromone trails
3. More ants follow strong trails (positive feedback)
4. Old trails decay, adapting to food depletion
5. Efficient paths emerge naturally

No ant has global knowledge - complex colony behavior emerges from simple individual rules.

## Browser Compatibility

- Modern browsers with Canvas API
- Touch support for mobile
- Browser zoom + scroll for navigation
