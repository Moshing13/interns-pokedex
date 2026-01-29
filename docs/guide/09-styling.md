# 09 - Styling with CSS

Now let's make our Pokedex look amazing with CSS! We'll create a classic Pokedex-inspired design.

## File Location

Create `public/css/style.css`. This file will be served as a static file.

## CSS Structure Overview

Our stylesheet is organized into sections:

1. Reset & Variables
2. Layout (Container, Header, Footer)
3. Search & Filters
4. Pokemon Grid & Cards
5. Pokemon Detail Page
6. Error Page
7. Responsive Design

## Part 1: Reset and CSS Variables

```css
/* Reset and Base Styles */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  /* Pokedex Colors */
  --pokedex-red: #dc0a2d;
  --pokedex-dark-red: #a00020;
  --pokedex-cream: #f0f0e8;
  --pokedex-screen: #98cb98;
  --text-dark: #1a1a2e;
  --text-light: #ffffff;
  --shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 25px rgba(0, 0, 0, 0.2);

  /* Pokemon Type Colors */
  --type-normal: #A8A878;
  --type-fire: #F08030;
  --type-water: #6890F0;
  --type-electric: #F8D030;
  --type-grass: #78C850;
  --type-ice: #98D8D8;
  --type-fighting: #C03028;
  --type-poison: #A040A0;
  --type-ground: #E0C068;
  --type-flying: #A890F0;
  --type-psychic: #F85888;
  --type-bug: #A8B820;
  --type-rock: #B8A038;
  --type-ghost: #705898;
  --type-dragon: #7038F8;
  --type-dark: #705848;
  --type-steel: #B8B8D0;
  --type-fairy: #EE99AC;
}

body {
  font-family: 'Roboto', sans-serif;
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
  min-height: 100vh;
  color: var(--text-dark);
}
```

### Understanding CSS Variables

CSS variables (custom properties) let you reuse values:

```css
:root {
  --my-color: #ff0000;  /* Define */
}

.element {
  color: var(--my-color);  /* Use */
}
```

## Part 2: Layout Structure

```css
/* Pokedex Container */
.pokedex-container {
  max-width: 1400px;
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* Main Content */
.main-content {
  flex: 1;
  padding: 30px;
  background: var(--pokedex-cream);
}
```

## Part 3: Header with Pokedex Styling

```css
/* Header */
.pokedex-header {
  background: var(--pokedex-red);
  padding: 20px 30px;
  display: flex;
  align-items: center;
  gap: 30px;
  border-bottom: 8px solid var(--pokedex-dark-red);
  box-shadow: var(--shadow-lg);
}

.header-left {
  display: flex;
  align-items: center;
  gap: 15px;
}

/* The big blue light */
.big-light {
  width: 60px;
  height: 60px;
  background: linear-gradient(135deg, #7df9ff 0%, #00bfff 50%, #0080ff 100%);
  border-radius: 50%;
  border: 4px solid white;
  box-shadow:
    inset -5px -5px 15px rgba(0, 0, 0, 0.3),
    inset 5px 5px 15px rgba(255, 255, 255, 0.5),
    0 0 20px rgba(125, 249, 255, 0.5);
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%, 100% {
    box-shadow: inset -5px -5px 15px rgba(0, 0, 0, 0.3),
                inset 5px 5px 15px rgba(255, 255, 255, 0.5),
                0 0 20px rgba(125, 249, 255, 0.5);
  }
  50% {
    box-shadow: inset -5px -5px 15px rgba(0, 0, 0, 0.3),
                inset 5px 5px 15px rgba(255, 255, 255, 0.5),
                0 0 35px rgba(125, 249, 255, 0.8);
  }
}

/* Small indicator lights */
.small-lights {
  display: flex;
  gap: 8px;
}

.small-light {
  width: 16px;
  height: 16px;
  border-radius: 50%;
  border: 2px solid rgba(255, 255, 255, 0.8);
}

.small-light.red { background: #ff5555; box-shadow: 0 0 8px #ff5555; }
.small-light.yellow { background: #ffdd55; box-shadow: 0 0 8px #ffdd55; }
.small-light.green { background: #55ff55; box-shadow: 0 0 8px #55ff55; }

/* Logo */
.logo {
  font-family: 'Press Start 2P', cursive;
  font-size: 28px;
  color: white;
  text-shadow: 3px 3px 0 var(--pokedex-dark-red);
  letter-spacing: 2px;
}
```

## Part 4: Search and Type Filters

```css
/* Search Section */
.search-section {
  background: white;
  padding: 25px;
  border-radius: 15px;
  margin-bottom: 30px;
  box-shadow: var(--shadow);
  border: 3px solid #ddd;
}

.search-input-wrapper {
  display: flex;
  gap: 10px;
  max-width: 500px;
}

.search-input {
  flex: 1;
  padding: 15px 20px;
  font-size: 16px;
  border: 3px solid #ddd;
  border-radius: 30px;
  outline: none;
  transition: border-color 0.3s;
}

.search-input:focus {
  border-color: var(--pokedex-red);
}

.search-btn {
  padding: 15px 25px;
  background: var(--pokedex-red);
  color: white;
  border: none;
  border-radius: 30px;
  cursor: pointer;
  transition: background 0.3s, transform 0.2s;
}

.search-btn:hover {
  background: var(--pokedex-dark-red);
  transform: scale(1.05);
}

/* Type Filter Buttons */
.type-buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.type-btn {
  padding: 8px 16px;
  border-radius: 20px;
  text-decoration: none;
  font-size: 12px;
  font-weight: 600;
  text-transform: uppercase;
  color: white;
  background: #888;
  transition: transform 0.2s, box-shadow 0.2s;
}

.type-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.type-btn.active {
  box-shadow: 0 0 0 3px var(--text-dark);
}
```

## Part 5: Type Badge Colors

```css
/* Type Badge Colors */
.type-normal, .type-btn.type-normal { background: var(--type-normal); }
.type-fire, .type-btn.type-fire { background: var(--type-fire); }
.type-water, .type-btn.type-water { background: var(--type-water); }
.type-electric, .type-btn.type-electric { background: var(--type-electric); color: #333; }
.type-grass, .type-btn.type-grass { background: var(--type-grass); }
.type-ice, .type-btn.type-ice { background: var(--type-ice); color: #333; }
.type-fighting, .type-btn.type-fighting { background: var(--type-fighting); }
.type-poison, .type-btn.type-poison { background: var(--type-poison); }
.type-ground, .type-btn.type-ground { background: var(--type-ground); color: #333; }
.type-flying, .type-btn.type-flying { background: var(--type-flying); }
.type-psychic, .type-btn.type-psychic { background: var(--type-psychic); }
.type-bug, .type-btn.type-bug { background: var(--type-bug); }
.type-rock, .type-btn.type-rock { background: var(--type-rock); }
.type-ghost, .type-btn.type-ghost { background: var(--type-ghost); }
.type-dragon, .type-btn.type-dragon { background: var(--type-dragon); }
.type-dark, .type-btn.type-dark { background: var(--type-dark); }
.type-steel, .type-btn.type-steel { background: var(--type-steel); color: #333; }
.type-fairy, .type-btn.type-fairy { background: var(--type-fairy); }
```

## Part 6: Pokemon Grid

```css
/* Pokemon Grid */
.pokemon-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 25px;
}

.pokemon-card {
  background: white;
  border-radius: 20px;
  padding: 20px;
  text-decoration: none;
  color: var(--text-dark);
  transition: transform 0.3s, box-shadow 0.3s;
  box-shadow: var(--shadow);
  position: relative;
  overflow: hidden;
}

/* Colored header based on type */
.pokemon-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 100px;
  opacity: 0.3;
  border-radius: 20px 20px 0 0;
}

.type-bg-fire::before { background: var(--type-fire); }
.type-bg-water::before { background: var(--type-water); }
/* ... more type backgrounds ... */

.pokemon-card:hover {
  transform: translateY(-10px) scale(1.02);
  box-shadow: var(--shadow-lg);
}

.pokemon-image img {
  width: 140px;
  height: 140px;
  object-fit: contain;
  filter: drop-shadow(0 5px 10px rgba(0, 0, 0, 0.2));
  transition: transform 0.3s;
}

.pokemon-card:hover .pokemon-image img {
  transform: scale(1.1);
}

.type-badge {
  padding: 5px 12px;
  border-radius: 15px;
  font-size: 11px;
  font-weight: 600;
  text-transform: uppercase;
  color: white;
}
```

### Understanding CSS Grid

```css
.pokemon-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 25px;
}
```

This creates a responsive grid:
- `auto-fill`: Create as many columns as fit
- `minmax(220px, 1fr)`: Each column is at least 220px, max equal share
- `gap`: Space between items

## Part 7: Stat Bars

```css
.stat-row {
  display: flex;
  align-items: center;
  margin-bottom: 12px;
  gap: 15px;
}

.stat-name {
  width: 80px;
  font-weight: 600;
  font-size: 13px;
}

.stat-bar-container {
  flex: 1;
  height: 12px;
  background: #e8e8e8;
  border-radius: 6px;
  overflow: hidden;
}

.stat-bar {
  height: 100%;
  background: linear-gradient(90deg, var(--pokedex-red) 0%, #ff6b6b 100%);
  border-radius: 6px;
  transition: width 0.5s ease-out;
}
```

## Part 8: Responsive Design

```css
/* Tablet */
@media (max-width: 900px) {
  .pokemon-detail-content {
    grid-template-columns: 1fr;
  }
}

/* Mobile */
@media (max-width: 600px) {
  .pokedex-header {
    flex-direction: column;
    text-align: center;
  }

  .logo {
    font-size: 20px;
  }

  .pokemon-grid {
    grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
    gap: 15px;
  }

  .pokemon-card {
    padding: 15px;
  }

  .pokemon-image img {
    width: 100px;
    height: 100px;
  }
}
```

### Understanding Media Queries

```css
@media (max-width: 600px) {
  /* These styles apply when screen is 600px or narrower */
}
```

## Key CSS Techniques Used

### 1. Flexbox for Layout

```css
display: flex;
justify-content: center;  /* Horizontal alignment */
align-items: center;      /* Vertical alignment */
gap: 10px;               /* Space between items */
```

### 2. CSS Grid for Card Layout

```css
display: grid;
grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
```

### 3. Transitions for Smooth Effects

```css
transition: transform 0.3s, box-shadow 0.3s;
```

### 4. CSS Animations

```css
@keyframes pulse {
  0%, 100% { /* start/end state */ }
  50% { /* middle state */ }
}

.element {
  animation: pulse 2s infinite;
}
```

### 5. Pseudo-elements

```css
.card::before {
  content: '';
  /* Creates an element inside .card */
}
```

## Summary

| Section | Purpose |
|---------|---------|
| Variables | Reusable colors and values |
| Header | Pokedex-style navigation |
| Search | Input and filter buttons |
| Grid | Responsive card layout |
| Cards | Pokemon display with hover effects |
| Detail | Full Pokemon information page |
| Responsive | Mobile-friendly adjustments |

## Commit Your Progress

Commit your CSS styling:

```bash
git add .
git commit -m "feat: add CSS styling for pokedex theme

Full Name: Juan Dela Cruz
Umindanao: juan.delacruz@email.com"
```

> **Remember:** Replace the name and email with your own information.

## What's Next?

Let's add tests to ensure our code works correctly!

Next: [10 - Testing](./10-testing.md)
