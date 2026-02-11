# TresJS Best Practices

A skill reference for building 3D experiences with TresJS v5+ and Vue 3.

## What is This?

This repository contains curated best practices, patterns, and anti-patterns for working with [TresJS](https://tresjs.org/) - a declarative Vue 3 framework for Three.js.

## What is TresJS?

TresJS transforms Three.js development by providing:

- **Declarative Syntax** - Use Vue components instead of imperative Three.js code
- **Vue Reactivity** - Leverage Vue's reactivity system for dynamic 3D experiences
- **Better DX** - TypeScript support, Vue DevTools integration, HMR
- **Ecosystem** - Rich collection of helpers, components, and post-processing effects

### Example

Instead of this (vanilla Three.js):

```javascript
const scene = new THREE.Scene()
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
const renderer = new THREE.WebGLRenderer()
const geometry = new THREE.BoxGeometry()
const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 })
const mesh = new THREE.Mesh(geometry, material)
scene.add(mesh)
camera.position.z = 5
```

Write this (TresJS):

```vue
<TresCanvas>
  <TresPerspectiveCamera :position="[0, 0, 5]" />
  <TresMesh>
    <TresBoxGeometry />
    <TresMeshBasicMaterial color="#00ff00" />
  </TresMesh>
</TresCanvas>
```

## Quick Start

### Installation

You can replace `npm` with any alternatives like `pnpm`, `deno` or `bun`.

```bash
npm install @tresjs/core three
npm install -D @types/three
```

### Vite Configuration

```typescript
import { templateCompilerOptions } from '@tresjs/core'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue({ ...templateCompilerOptions })],
})
```

### Your First Scene

```vue
<script setup lang="ts">
import { TresCanvas } from '@tresjs/core'
</script>

<template>
  <TresCanvas clear-color="#000">
    <TresPerspectiveCamera :position="[5, 5, 5]" />
    <TresMesh>
      <TresBoxGeometry />
      <TresMeshStandardMaterial color="#00ff00" />
    </TresMesh>
    <TresAmbientLight :intensity="0.5" />
    <TresDirectionalLight :position="[3, 3, 3]" />
  </TresCanvas>
</template>
```

## Core Concepts

### Performance
- Use `shallowRef` for Three.js objects
- Choose appropriate render modes (`always`, `on-demand`, `manual`)
- Leverage tree-shaking

### Animation
- Use `useLoop` for simple animations with `delta` parameter
- Use GSAP for complex timelines and sequences

### Asset Loading
- Use `useLoader` composable (v5 reactive API)
- Wrap async components in `<Suspense>`

### Events
- Native DOM event names (`@click`, `@pointerdown`, etc.)
- Only first intersected element triggers events (performance)

## Common Gotchas

1. **Always position cameras away from origin** - Default is `[0,0,0]`
2. **Use `shallowRef` not `ref`** for Three.js objects
3. **Frame-rate independence** - Always use `delta` parameter
4. **Component order** - Controls must come after camera
5. **WebGL context props** - Non-reactive, set once at creation

## v5 Breaking Changes

- Event names: `@pointer-down` → `@pointerdown`
- `useLoader` returns reactive state instead of promise
- `useTexture` moved to `@tresjs/cientos`
- ESM only (no UMD)

## Resources

- [TresJS Documentation](https://docs.tresjs.org/)
- [TresJS GitHub](https://github.com/Tresjs/tres)
- [Cientos (Helpers)](https://cientos.tresjs.org/)
- [Three.js Docs](https://threejs.org/docs/)
