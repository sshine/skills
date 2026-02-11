# TresJS Best Practices

This project uses TresJS (v5+), a Vue 3 framework for creating declarative Three.js experiences.

## Quick Reference

### Installation & Setup

Always configure Vite with TresJS template options:

```typescript
import { templateCompilerOptions } from '@tresjs/core'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue({ ...templateCompilerOptions })],
})
```

### Performance

**Use `shallowRef` for Three.js objects** - Three.js objects have deep structures that don't benefit from Vue reactivity:

```typescript
// Good
const meshRef = shallowRef<THREE.Mesh | null>(null)

// Avoid - unnecessary overhead
const meshRef = ref<THREE.Mesh | null>(null)
```

**Use appropriate render modes** on `<TresCanvas>`:
- `render-mode="always"` - Continuous (default, use for animations)
- `render-mode="on-demand"` - Auto-invalidation (use for mostly static scenes)
- `render-mode="manual"` - Full control (advanced use cases)

### Component Patterns

**Separate canvas from experience:**

```vue
<!-- App.vue -->
<TresCanvas>
  <Experience />
</TresCanvas>

<!-- Experience.vue -->
<template>
  <TresPerspectiveCamera :position="[5, 5, 5]" />
  <TresMesh>
    <TresBoxGeometry :args="[2, 2, 2]" />
    <TresMeshStandardMaterial color="#00ff00" />
  </TresMesh>
</template>
```

**Always position cameras away from origin:**

```vue
<!-- Bad - camera at scene center -->
<TresPerspectiveCamera />

<!-- Good -->
<TresPerspectiveCamera :position="[7, 7, 7]" :look-at="[0, 0, 0]" />
```

### Animation

**Use `useLoop` with delta for frame-rate independence:**

```typescript
import { useLoop } from '@tresjs/core'

const { onBeforeRender } = useLoop()

onBeforeRender(({ delta, elapsed }) => {
  // delta: time since last frame
  if (mesh.value) {
    mesh.value.rotation.x += delta * 0.5 // frame-rate independent
  }
})
```

### Asset Loading

**Use `useLoader` composable (v5+ reactive API):**

```typescript
import { useLoader } from '@tresjs/core'
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader'

const { state: model, isLoading, error, progress } = useLoader(
  GLTFLoader,
  '/models/scene.gltf'
)
```

**Wrap async components in Suspense:**

```vue
<Suspense>
  <template #default><ModelComponent /></template>
  <template #fallback><LoadingSpinner /></template>
</Suspense>
```

### Component Naming

All Three.js classes use `Tres` prefix:
- `THREE.PerspectiveCamera` → `<TresPerspectiveCamera />`
- `THREE.Mesh` → `<TresMesh />`
- `THREE.BoxGeometry` → `<TresBoxGeometry />`

### Constructor Arguments

Use `:args` prop for constructor parameters:

```vue
<!-- new THREE.BoxGeometry(2, 2, 2) -->
<TresBoxGeometry :args="[2, 2, 2]" />

<!-- new THREE.PerspectiveCamera(75, 1, 0.1, 1000) -->
<TresPerspectiveCamera :args="[75, 1, 0.1, 1000]" />
```

### Events (v5+)

Use native DOM event names:

```vue
<TresMesh
  @click="handleClick"
  @pointerdown="handleDown"
  @pointerup="handleUp"
  @pointermove="handleMove"
  @pointerenter="handleEnter"
  @pointerleave="handleLeave"
/>

<!-- On canvas for clicks outside objects -->
<TresCanvas @pointermissed="handleMissed">
```

**Note:** Only the first intersected element triggers pointer events (performance optimization).

### Shadows

Enable shadows at canvas level, then on objects and lights:

```vue
<TresCanvas shadows>
  <TresDirectionalLight :cast-shadow="true" />
  <TresMesh :cast-shadow="true" :receive-shadow="true">
    <!-- geometry/material -->
  </TresMesh>
</TresCanvas>
```

### Extending the Component Catalogue

Add custom Three.js classes:

```typescript
import { extend } from '@tresjs/core'
import { OrbitControls } from 'three/addons/controls/OrbitControls'

extend({ OrbitControls })
```

Then use: `<TresOrbitControls />`

### Memory Management

**For `<primitive>` objects, handle disposal manually:**

```vue
<primitive
  :object="customObject"
  :dispose="customDisposeFunction"
/>
```

TresJS does NOT auto-dispose primitive resources.

### Development Helpers

Use during development, remove in production:

```vue
<TresAxesHelper :args="[5]" />
<TresGridHelper :args="[10, 10]" />
<TresDirectionalLight v-light-helper />
```

## Common Gotchas

1. **Default camera position is [0,0,0]** - Always set camera position
2. **Component order matters** - Place controls after camera:
   ```vue
   <TresPerspectiveCamera :position="[5, 5, 5]" />
   <TresOrbitControls /> <!-- Must come after camera -->
   ```
3. **WebGL context props are non-reactive** - Props like `antialias`, `alpha`, `depth` cannot change after canvas creation
4. **Memory leak on mobile** - Known issue with unmounting `TresCanvas` on mobile browsers
5. **useTexture moved to Cientos** - In v5, texture loading moved to `@tresjs/cientos`

## v5 Breaking Changes

### Event Names
- `@pointer-down` → `@pointerdown`
- `@pointer-up` → `@pointerup`
- `@after-render` → `@render`

### Removed Composables
- `useTresReady` → Use `@ready` event
- `useRenderLoop` → Use `useLoop`
- `useTexture` → Moved to `@tresjs/cientos`

### useLoader API Change
```typescript
// Old (v4)
const model = await useLoader(GLTFLoader, url)

// New (v5)
const { state: model, isLoading, error } = useLoader(GLTFLoader, url)
```

## Ecosystem

- **@tresjs/cientos** - Ready-to-use helpers (useGLTF, useTexture, Text3D, Environment, etc.)
- **@tresjs/post-processing** - Post-processing effects
- **@tresjs/nuxt** - Nuxt module with auto-imports

## Resources

- [Documentation](https://docs.tresjs.org/)
- [TresCanvas API](https://docs.tresjs.org/api/components/tres-canvas)
- [Cookbook](https://docs.tresjs.org/cookbook/)
- [Cientos Components](https://cientos.tresjs.org/)
