# Reels Prototype Engine

A lightweight, config-driven web engine for rapidly prototyping slot-style reel mechanics. Built as a design + prototyping exercise focused on flexible architecture: swap reel size, symbols, paylines, and feature triggers entirely through a JSON config — no code changes required.

**[Play the live demo →](https://DoloressD.github.io/reels-prototype-engine/)**

## What it does

- Config-driven reel grid (reel count, row count, symbol sets, weighted reel strips)
- Staggered spin animation
- Payline evaluation with win highlighting
- Scatter detection and feature trigger hooks (e.g. free spins)
- Live debug panel with forced-result input and win-rate simulation (100 / 1,000 / 10,000 spin runs)
- Three swappable configs included: Classic 3×3, Standard 5×3, Wide 6×4

## Why

The goal was iteration speed: let a designer or stakeholder test a new reel layout, symbol set, or feature idea by editing config, not code. See [DESIGN.md](./DESIGN.md) for the full architecture writeup — system breakdown, phased build plan, risks, and a self-critique of what's strong and what's still underspecified.

## Tech

Single-file HTML/CSS/JS, no build step, no dependencies. Open `index.html` directly or visit the live demo link above.
