---
name: tresjs-best-practices
description: Best practices for TresJS - Building 3D experiences with Vue 3 and Three.js
metadata:
  tags: tresjs, vue, three.js, 3d, webgl, graphics
---

## When to use

Use this skill whenever you are working with TresJS code - a declarative Vue 3 framework for building Three.js 3D experiences. This provides comprehensive patterns, best practices, and anti-patterns for TresJS v5+.

## What is TresJS?

TresJS transforms Three.js development by providing:
- **Declarative Syntax** - Use Vue components instead of imperative Three.js code
- **Vue Reactivity** - Leverage Vue's reactivity system for dynamic 3D experiences
- **Better DX** - TypeScript support, Vue DevTools integration, HMR
- **Ecosystem** - Rich collection of helpers, components, and post-processing effects

## How to use

This skill includes several reference documents:

### [TRESJS_BEST_PRACTICES.md](./TRESJS_BEST_PRACTICES.md)
Complete reference guide covering:
- **Installation & Setup** - Vite configuration, Nuxt integration, required packages
- **Performance Optimization** - shallowRef usage, render modes, tree-shaking, device pixel ratio
- **Component Usage** - TresCanvas props, naming conventions, template refs, the attach prop
- **Composables** - useLoop, useLoader, useTres, useTexture, useGLTF
- **Animation Patterns** - Simple animations with useLoop, complex timelines with GSAP
- **Event Handling** - Pointer events, canvas events, hover states
- **Memory Management** - Automatic disposal, manual cleanup for primitives
- **Common Patterns** - Scene structure, camera setup, lighting, shadows, controls
- **Anti-Patterns** - What NOT to do and why
- **Migration Guide** - v4 to v5 breaking changes and updates

### [CHEATSHEET.md](./CHEATSHEET.md)
Quick reference for common TresJS patterns:
- Installation snippets
- Basic scene structure
- Component naming conventions
- Performance rules
- Animation examples
- Asset loading
- Event handling
- Shadows and lighting
- Controls setup
- Common gotchas

### [README.md](./README.md)
Overview of TresJS concepts with quick start guide and core concepts summary.

## Key Topics

**Performance:**
- Always use `shallowRef` for Three.js objects (not `ref`)
- Choose appropriate render modes (`always`, `on-demand`, `manual`)
- Leverage tree-shaking by importing only needed components

**Animation:**
- Use `useLoop` with `delta` parameter for frame-rate independence
- Use GSAP for complex timelines and sequences

**Common Gotchas:**
- Always position cameras away from origin (default is [0,0,0])
- Use `shallowRef` not `ref` for Three.js objects
- Always use `delta` parameter for frame-rate independence
- Component order matters - controls must come after camera
- WebGL context props are non-reactive - set once at creation

**v5 Breaking Changes:**
- Event names changed: `@pointer-down` → `@pointerdown`
- `useLoader` returns reactive state instead of promise
- `useTexture` moved to `@tresjs/cientos`
- ESM only (no UMD)

## Resources

- [TresJS Documentation](https://docs.tresjs.org/)
- [Cientos (Helpers)](https://cientos.tresjs.org/)
- [Three.js Docs](https://threejs.org/docs/)
