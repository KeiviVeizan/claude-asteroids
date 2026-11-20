# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Asteroids clone in plain HTML5 Canvas. No dependencies, no bundler, no build step, no tests, no linter. The README and in-game text are in Spanish; keep new user-facing text and comments consistent with that.

## Running

Open `index.html` directly in a browser, or serve locally with `npx serve .` (http://localhost:3000). Deployed as a static site (GitHub Pages).

## Architecture

All logic lives in `game.js` (loaded by `index.html`, which provides `<canvas id="canvas">`). Rendering uses a fixed 800×600 logical size (`W`, `H`).

- **Entities** (`Ship`, `Asteroid`, `Bullet`, `Particle`) are classes with `update(dt)` and `draw()`; each has a `dead` flag. Dead entities are removed by filtering the arrays (`bullets`, `asteroids`, `particles`) after update, not by splicing mid-iteration.
- **Global state**: `ship`, entity arrays, `score`, `lives`, `level`, and a `state` string: `'playing' | 'dead' | 'gameover'`. `update()` branches on `state` first; `'dead'` is a 2s respawn timer (`deadTimer`) during which asteroids and particles keep moving, and `'gameover'` waits for Space to call `initGame()`.
- **Toroidal world**: positions wrap via `wrap(v, max)`. Particles are the exception and don't wrap.
- **Input**: `keys` holds held state; `justPressed` plus `pressed(code)` gives edge-triggered, consume-on-read presses (used for shooting and restart, so Space must be re-pressed per shot).
- **Asteroid sizes** are 1–3, indexed into the parallel arrays `RADII`, `SPEEDS`, `POINTS`. `split()` produces two asteroids of `size - 1`. Note that `POINTS` rewards smaller asteroids more.
- **Collisions** are circle checks with `dist()`. Ship-vs-asteroid uses `a.radius * 0.82` for leniency and is skipped while `ship.invincible > 0` (3s after any `reset()`, including level start).
- **Game loop**: `requestAnimationFrame` with `dt` in seconds, clamped to 0.05 to avoid large jumps after tab switches. A level is cleared when `asteroids.length === 0`, then `nextLevel()` spawns `3 + level` large asteroids outside a safe radius around screen center.
