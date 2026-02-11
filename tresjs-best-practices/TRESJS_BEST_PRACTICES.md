# TresJS Best Practices - Complete Reference

> Comprehensive guide for building 3D experiences with TresJS v5+ and Vue 3

## Table of Contents

- [Installation & Setup](#installation--setup)
- [Performance Optimization](#performance-optimization)
- [Component Usage](#component-usage)
- [Composables](#composables)
- [Animation Patterns](#animation-patterns)
- [Event Handling](#event-handling)
- [Memory Management](#memory-management)
- [Common Patterns](#common-patterns)
- [Anti-Patterns](#anti-patterns)
- [Migration Guide (v4 → v5)](#migration-guide-v4--v5)

---

## Installation & Setup

### Basic Installation

```bash
npm install @tresjs/core three
npm install -D @types/three
```

### Vite Configuration (Required)

**Critical:** Configure Vue compiler to prevent console warnings:

```typescript
// vite.config.ts
import { templateCompilerOptions } from '@tresjs/core'
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [
    vue({
      ...templateCompilerOptions
    }),
  ],
})
```

### Nuxt Setup

```bash
npm install @tresjs/nuxt
```

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@tresjs/nuxt'],
})
```

Benefits: auto-imports, better DX, automatic devtools integration.

### Additional Packages

```bash
# Helpers and components
npm install @tresjs/cientos

# Post-processing effects
npm install @tresjs/post-processing
```

---

## Performance Optimization

### 1. Use `shallowRef` for Three.js Objects

Three.js objects have complex nested structures. Vue's deep reactivity system creates unnecessary overhead.

```typescript
// ✅ Good - shallow reactivity
const meshRef = shallowRef<THREE.Mesh | null>(null)
const materialRef = shallowRef<THREE.Material | null>(null)

// ❌ Bad - deep reactivity overhead
const meshRef = ref<THREE.Mesh | null>(null)
```

**Why:** Three.js objects contain matrices, geometries, and materials with hundreds of properties. Deep reactivity tracking is wasteful since most updates happen imperatively.

### 2. Render Modes

Choose the appropriate render mode for your scene:

```vue
<!-- Continuous rendering (default) - use for animations -->
<TresCanvas render-mode="always">

<!-- On-demand rendering - use for mostly static scenes -->
<TresCanvas render-mode="on-demand">

<!-- Manual rendering - advanced control -->
<TresCanvas render-mode="manual">
```

**On-demand mode** automatically invalidates when props change, saving significant performance for static scenes.

**Manual mode** requires calling `invalidate()` or `advance()` from `useTres()`:

```typescript
const { invalidate, advance } = useTres()

// Single frame render
invalidate()

// Advance by specific delta
advance(16.67) // ~60fps frame
```

### 3. Tree-Shaking

Import only components you need. TresJS supports tree-shaking:

```typescript
// Bundle only includes used components
import { TresMesh, TresPerspectiveCamera } from '@tresjs/core'
```

### 4. Event Performance

**v5+ optimization:** Only the first intersected element triggers pointer events, significantly improving performance in complex scenes with overlapping geometries.

### 5. Device Pixel Ratio

Control rendering resolution:

```vue
<!-- Lower DPR for better performance -->
<TresCanvas :dpr="[1, 1.5]">

<!-- Full device resolution (default) -->
<TresCanvas :dpr="[1, 2]">
```

Array format: `[min, max]` - clamps to device capabilities.

---

## Component Usage

### TresCanvas - The Root Component

Main entry point for 3D scenes:

```vue
<TresCanvas
  clear-color="#000000"
  :alpha="true"
  :antialias="true"
  shadows
  :shadow-map-type="PCFSoftShadowMap"
  :tone-mapping="ACESFilmicToneMapping"
  :tone-mapping-exposure="1"
  render-mode="always"
  :window-size="true"
>
  <Experience />
</TresCanvas>
```

#### Non-Reactive Props (WebGL Context)

**Cannot be changed after canvas creation:**
- `alpha` - Transparent background
- `antialias` - Edge smoothing
- `depth` - Depth buffer
- `stencil` - Stencil buffer
- `logarithmicDepthBuffer` - Better depth precision
- `preserveDrawingBuffer` - For screenshots
- `failIfMajorPerformanceCaveat` - Graceful degradation
- `useLegacyLights` - Compatibility mode

**Plan these settings upfront!**

#### Reactive Props

Can be changed anytime:
- `clearColor` / `clearAlpha` - Background
- `shadows` / `shadowMapType` - Shadow settings
- `dpr` - Pixel ratio
- `toneMapping` / `toneMappingExposure` - Color grading
- `renderMode` - Rendering strategy
- `windowSize` - Fullscreen mode

### Component Naming Convention

All Three.js classes use `Tres` prefix:

| Three.js | TresJS Component |
|----------|-----------------|
| `THREE.PerspectiveCamera` | `<TresPerspectiveCamera />` |
| `THREE.Mesh` | `<TresMesh />` |
| `THREE.BoxGeometry` | `<TresBoxGeometry />` |
| `THREE.MeshStandardMaterial` | `<TresMeshStandardMaterial />` |
| `THREE.DirectionalLight` | `<TresDirectionalLight />` |
| `THREE.Group` | `<TresGroup />` |

### The `:args` Prop

Pass constructor arguments as an array:

```vue
<!-- new THREE.BoxGeometry(width, height, depth) -->
<TresBoxGeometry :args="[2, 2, 2]" />

<!-- new THREE.PerspectiveCamera(fov, aspect, near, far) -->
<TresPerspectiveCamera :args="[75, 1, 0.1, 1000]" />

<!-- new THREE.SphereGeometry(radius, widthSegments, heightSegments) -->
<TresSphereGeometry :args="[1, 32, 32]" />
```

### Prop Mapping

All Three.js properties work as Vue props:

```vue
<TresMesh
  :position="[0, 1, 0]"
  :rotation="[0, Math.PI / 4, 0]"
  :scale="[1, 1, 1]"
  :visible="isVisible"
  :cast-shadow="true"
  :receive-shadow="true"
/>

<TresMeshStandardMaterial
  color="#ff0000"
  :roughness="0.5"
  :metalness="0.8"
  :opacity="0.5"
  :transparent="true"
/>
```

**TresJS automatically converts:**
- Kebab-case to camelCase (`cast-shadow` → `castShadow`)
- Arrays to Vector3/Euler (`[x, y, z]` → `new THREE.Vector3(x, y, z)`)
- Hex strings to Colors (`"#ff0000"` → `new THREE.Color(0xff0000)`)

### Template Refs

Access underlying Three.js objects:

```vue
<script setup lang="ts">
import { shallowRef } from 'vue'
import type { Mesh } from 'three'

const meshRef = shallowRef<Mesh | null>(null)

onMounted(() => {
  if (meshRef.value) {
    console.log(meshRef.value.geometry)
  }
})
</script>

<template>
  <TresMesh ref="meshRef">
    <TresBoxGeometry />
    <TresMeshStandardMaterial />
  </TresMesh>
</template>
```

### The `attach` Prop

For advanced parent-child relationships:

```vue
<!-- Automatic attachment (works for most cases) -->
<TresMesh>
  <TresBoxGeometry /> <!-- Attached as geometry -->
  <TresMeshStandardMaterial /> <!-- Attached as material -->
</TresMesh>

<!-- Multiple materials -->
<TresMesh>
  <TresBoxGeometry />
  <TresMeshStandardMaterial attach="material-0" />
  <TresMeshStandardMaterial attach="material-1" />
</TresMesh>

<!-- Custom attachment -->
<TresCamera>
  <TresOrthographicCamera attach="left" />
  <TresPerspectiveCamera attach="right" />
</TresCamera>
```

**Piercing syntax** for deep attachments (kebab-case):

```vue
<primitive :object="customObject">
  <TresMaterial attach="some-deep-property" />
</primitive>
```

### Extending the Component Catalogue

Add custom Three.js classes or addon classes:

```typescript
import { extend } from '@tresjs/core'
import { OrbitControls } from 'three/addons/controls/OrbitControls'
import { Water } from 'three/addons/objects/Water'

extend({ OrbitControls, Water })
```

Then use as components:

```vue
<TresOrbitControls :args="[camera, canvas]" />
<TresWater />
```

---

## Composables

### `useLoop` - Animation Core

Primary composable for render loop integration:

```typescript
import { useLoop } from '@tresjs/core'

const { onBeforeRender, resume, pause, isActive } = useLoop()

// Register callback (runs every frame)
onBeforeRender(({ delta, elapsed, clock }) => {
  // delta: seconds since last frame (frame-rate independence)
  // elapsed: total seconds since start
  // clock: Three.js Clock instance

  if (mesh.value) {
    mesh.value.rotation.y += delta // ~60fps = 0.016/frame
  }
})

// Control loop
pause()  // Stop rendering
resume() // Resume rendering
console.log(isActive.value) // Check state
```

**Best Practice:** Always use `delta` parameter for frame-rate independence:

```typescript
// ❌ Bad - speed varies by device
onBeforeRender(() => {
  mesh.value.rotation.x += 0.01
})

// ✅ Good - consistent across all frame rates
onBeforeRender(({ delta }) => {
  mesh.value.rotation.x += delta * 0.5
})
```

**Priority parameter** (optional):

```typescript
// Higher numbers run first
onBeforeRender(() => { /* runs first */ }, 10)
onBeforeRender(() => { /* runs second */ }, 5)
onBeforeRender(() => { /* runs last */ })  // default: 0
```

### `useLoader` - Asset Loading

**v5+ Reactive API:**

```typescript
import { useLoader } from '@tresjs/core'
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader'
import { TextureLoader } from 'three'

// Returns reactive state
const {
  state: model,    // Loaded resource (null until loaded)
  isLoading,       // Boolean loading state
  error,           // Error object if failed
  progress         // Loading progress (0-1)
} = useLoader(GLTFLoader, '/models/scene.gltf')

// With options
const { state: texture } = useLoader(
  TextureLoader,
  '/textures/diffuse.jpg',
  (loader) => {
    // Configure loader
    loader.setCrossOrigin('anonymous')
  }
)
```

**Multiple resources:**

```typescript
const { state: models } = useLoader(GLTFLoader, [
  '/models/car.gltf',
  '/models/tree.gltf',
  '/models/building.gltf'
])
```

**Usage in template:**

```vue
<script setup>
const { state: model, isLoading } = useLoader(GLTFLoader, '/scene.gltf')
</script>

<template>
  <primitive v-if="!isLoading && model" :object="model.scene" />
</template>
```

**With Suspense:**

```vue
<Suspense>
  <template #default>
    <ModelComponent />
  </template>
  <template #fallback>
    <LoadingIndicator />
  </template>
</Suspense>
```

**Migration Note (v4 → v5):**

```typescript
// Old (v4) - Promise-based
const model = await useLoader(GLTFLoader, '/model.gltf')

// New (v5) - Reactive state
const { state: model, isLoading, error } = useLoader(GLTFLoader, '/model.gltf')
```

### `useTres` - Context Access

Access global TresJS context:

```typescript
import { useTres } from '@tresjs/core'

const {
  camera,        // Current camera (ref)
  renderer,      // WebGL renderer (ref)
  scene,         // Three.js scene (ref)
  sizes,         // Canvas dimensions
  invalidate,    // Request single render
  advance,       // Advance by delta
  extend,        // Add custom components
  controls       // Current controls (if any)
} = useTres()
```

**Manual rendering example:**

```typescript
const { invalidate } = useTres()

function handleUserInput() {
  // Update scene
  mesh.value.position.x += 1

  // Request re-render
  invalidate()
}
```

### `useTexture` (from Cientos)

**Moved to `@tresjs/cientos` in v5:**

```typescript
import { useTexture } from '@tresjs/cientos'

const { map, normalMap, roughnessMap } = await useTexture({
  map: '/textures/diffuse.jpg',
  normalMap: '/textures/normal.jpg',
  roughnessMap: '/textures/roughness.jpg',
})
```

### `useGLTF` (from Cientos)

Enhanced GLTF loading with Draco support:

```typescript
import { useGLTF } from '@tresjs/cientos'

const { scene, animations, asset } = await useGLTF('/models/scene.gltf', {
  draco: true // Enable Draco compression
})
```

---

## Animation Patterns

### Simple Animations - `useLoop`

Best for continuous, simple animations:

```typescript
const { onBeforeRender } = useLoop()

onBeforeRender(({ delta, elapsed }) => {
  // Rotation
  mesh.value.rotation.y += delta

  // Oscillation
  mesh.value.position.y = Math.sin(elapsed) * 2

  // Bounce
  mesh.value.scale.setScalar(1 + Math.sin(elapsed * 4) * 0.1)
})
```

### Complex Animations - GSAP

For sequences, timelines, easing:

```bash
npm install gsap
```

```typescript
import gsap from 'gsap'

// Simple tween
gsap.to(mesh.value.position, {
  x: 5,
  duration: 2,
  ease: 'power2.inOut'
})

// Timeline
const tl = gsap.timeline()
tl.to(mesh.value.position, { y: 2, duration: 1 })
  .to(mesh.value.rotation, { y: Math.PI, duration: 1 }, '<')
  .to(mesh.value.scale, { x: 2, y: 2, z: 2, duration: 0.5 })

// Advanced features
gsap.to(meshes.value, {
  y: 5,
  stagger: 0.1, // Delay between each
  repeat: -1,   // Infinite
  yoyo: true    // Reverse
})
```

**Benefits over useLoop:**
- Automatic frame rate optimization
- Rich easing functions
- Timeline sequencing
- Better performance for complex sequences

### Model Animations

For animated GLTF models:

```typescript
import { useGLTF, useAnimations } from '@tresjs/cientos'

const { scene, animations } = await useGLTF('/animated-model.gltf')
const { actions, mixer } = useAnimations(animations, scene)

// Play animation
actions.value['Walk']?.play()

// Update mixer in loop
const { onBeforeRender } = useLoop()
onBeforeRender(({ delta }) => {
  mixer.value?.update(delta)
})
```

---

## Event Handling

### Pointer Events (v5+)

Use native DOM event names:

```vue
<TresMesh
  @click="handleClick"
  @pointerdown="(event) => console.log('down', event)"
  @pointerup="(event) => console.log('up', event)"
  @pointermove="handleMove"
  @pointerenter="() => isHovered = true"
  @pointerleave="() => isHovered = false"
  @contextmenu="handleRightClick"
/>
```

**Event object:**

```typescript
interface TresEvent {
  intersect: Intersection    // Three.js intersection
  event: PointerEvent       // DOM event
  stopPropagating: () => void
}

function handleClick(e: TresEvent) {
  console.log('Clicked object:', e.intersect.object)
  console.log('World position:', e.intersect.point)
  console.log('Face:', e.intersect.face)

  e.stopPropagating() // Prevent parent handlers
}
```

### Canvas Events

Special events on `TresCanvas`:

```vue
<TresCanvas
  @click="handleCanvasClick"
  @pointermissed="handleClickOutside"
  @ready="onSceneReady"
>
```

- `@pointermissed` - Click on canvas but not on any object
- `@ready` - Scene fully initialized

### Hover State Example

```vue
<script setup lang="ts">
const isHovered = ref(false)
const color = computed(() => isHovered.value ? '#ff0000' : '#00ff00')
</script>

<template>
  <TresMesh
    @pointerenter="isHovered = true"
    @pointerleave="isHovered = false"
  >
    <TresBoxGeometry />
    <TresMeshStandardMaterial :color="color" />
  </TresMesh>
</template>
```

**Performance Note:** In v5+, only the first intersected element triggers events, significantly improving performance in complex scenes.

---

## Memory Management

### Automatic Disposal

TresJS automatically disposes:
- Geometries
- Materials
- Textures (when using TresJS loaders)
- Render targets

**When components unmount**, resources are cleaned up.

### Manual Disposal - `<primitive>`

For custom Three.js objects passed via `<primitive>`, disposal is **NOT automatic**:

```typescript
const customMesh = new THREE.Mesh(
  new THREE.BoxGeometry(),
  new THREE.MeshStandardMaterial()
)

function disposeCustomMesh() {
  customMesh.geometry.dispose()
  customMesh.material.dispose()
}
</script>

<template>
  <primitive
    :object="customMesh"
    :dispose="disposeCustomMesh"
  />
</template>
```

### Textures

```typescript
const texture = new THREE.TextureLoader().load('/texture.jpg')

onBeforeUnmount(() => {
  texture.dispose()
})
```

### Known Issues

**Mobile memory leak:** There's a known issue where unmounting `TresCanvas` may not properly dispose all internal objects on mobile browsers. Be cautious in SPAs that frequently mount/unmount 3D scenes.

**Workaround:** Keep canvas mounted and toggle content inside instead of unmounting entire canvas.

---

## Common Patterns

### Scene Structure

**Separate Canvas from Experience:**

```vue
<!-- App.vue -->
<template>
  <TresCanvas v-bind="canvasProps">
    <Experience />
  </TresCanvas>
</template>

<script setup lang="ts">
const canvasProps = {
  clearColor: '#1a1a1a',
  shadows: true,
  alpha: false,
}
</script>
```

```vue
<!-- Experience.vue -->
<template>
  <TresPerspectiveCamera :position="[5, 5, 5]" :look-at="[0, 0, 0]" />
  <TresDirectionalLight :position="[3, 3, 3]" :intensity="1" />
  <TresAmbientLight :intensity="0.5" />

  <Scene />
</template>
```

**Why:** Separation of concerns, easier testing, cleaner code.

### Camera Setup

```vue
<TresPerspectiveCamera
  :args="[75, 1, 0.1, 1000]"
  :position="[7, 7, 7]"
  :look-at="[0, 0, 0]"
/>
```

**Common gotcha:** Default position is `[0, 0, 0]` (inside scene), resulting in invisible viewpoint.

### Lighting

**Basic three-point lighting:**

```vue
<!-- Key light -->
<TresDirectionalLight
  :position="[5, 5, 5]"
  :intensity="1"
  :cast-shadow="true"
/>

<!-- Fill light -->
<TresDirectionalLight
  :position="[-3, 1, -5]"
  :intensity="0.4"
/>

<!-- Ambient (base illumination) -->
<TresAmbientLight :intensity="0.3" />
```

### Shadows

```vue
<TresCanvas shadows :shadow-map-type="PCFSoftShadowMap">
  <TresDirectionalLight
    :position="[3, 3, 3]"
    :cast-shadow="true"
    :shadow-mapSize-width="2048"
    :shadow-mapSize-height="2048"
    :shadow-camera-top="10"
    :shadow-camera-bottom="-10"
    :shadow-camera-left="-10"
    :shadow-camera-right="10"
  />

  <TresMesh :cast-shadow="true" :receive-shadow="true">
    <TresBoxGeometry />
    <TresMeshStandardMaterial />
  </TresMesh>

  <!-- Ground plane -->
  <TresMesh :rotation="[-Math.PI / 2, 0, 0]" :receive-shadow="true">
    <TresPlaneGeometry :args="[20, 20]" />
    <TresMeshStandardMaterial />
  </TresMesh>
</TresCanvas>
```

### Controls

**OrbitControls** - Must come after camera:

```vue
<script setup>
import { OrbitControls } from '@tresjs/cientos'
</script>

<template>
  <TresPerspectiveCamera :position="[5, 5, 5]" />
  <OrbitControls />  <!-- Must be after camera -->
</template>
```

**TransformControls:**

```vue
<script setup>
import { TransformControls } from '@tresjs/cientos'

const meshRef = shallowRef()
</script>

<template>
  <TresMesh ref="meshRef">
    <TresBoxGeometry />
    <TresMeshStandardMaterial />
  </TresMesh>

  <TransformControls :object="meshRef" mode="translate" />
</template>
```

### Development Helpers

Use during development, remove for production:

```vue
<template>
  <TresAxesHelper :args="[5]" />
  <TresGridHelper :args="[10, 10]" />

  <TresDirectionalLight v-light-helper />
  <TresSpotLight v-light-helper />

  <TresMesh>
    <TresBoxGeometry />
    <TresMeshStandardMaterial v-edges />
  </TresMesh>
</template>
```

---

## Anti-Patterns

### ❌ Using `ref` for Three.js Objects

```typescript
// DON'T
const mesh = ref<THREE.Mesh>()

// DO
const mesh = shallowRef<THREE.Mesh | null>(null)
```

**Why:** Deep reactivity overhead for complex objects.

### ❌ Forgetting Camera Position

```vue
<!-- DON'T - camera at origin -->
<TresPerspectiveCamera />

<!-- DO -->
<TresPerspectiveCamera :position="[5, 5, 5]" />
```

### ❌ Ignoring Frame-Rate Independence

```typescript
// DON'T
onBeforeRender(() => {
  mesh.value.rotation.x += 0.01
})

// DO
onBeforeRender(({ delta }) => {
  mesh.value.rotation.x += delta * 0.5
})
```

### ❌ Controls Before Camera

```vue
<!-- DON'T -->
<TresOrbitControls />
<TresPerspectiveCamera :position="[5, 5, 5]" />

<!-- DO -->
<TresPerspectiveCamera :position="[5, 5, 5]" />
<TresOrbitControls />
```

### ❌ Modifying Non-Reactive Props

```vue
<!-- DON'T - antialias can't change after creation -->
<TresCanvas :antialias="dynamicValue">

<!-- DO - set once -->
<TresCanvas :antialias="true">
```

### ❌ Not Disposing Primitives

```vue
<!-- DON'T -->
<primitive :object="customObject" />

<!-- DO -->
<primitive :object="customObject" :dispose="disposeFunction" />
```

---

## Migration Guide (v4 → v5)

### Event Names

```vue
<!-- Old (v4) -->
<TresMesh
  @pointer-down="handleDown"
  @pointer-up="handleUp"
/>

<!-- New (v5) -->
<TresMesh
  @pointerdown="handleDown"
  @pointerup="handleUp"
/>
```

**Changes:**
- `@pointer-down` → `@pointerdown`
- `@pointer-up` → `@pointerup`
- `@pointer-move` → `@pointermove`
- `@after-render` → `@render`
- `@before-render` → `@before-loop`

### useLoader API

```typescript
// Old (v4)
const model = await useLoader(GLTFLoader, '/model.gltf')

// New (v5)
const { state: model, isLoading, error } = useLoader(GLTFLoader, '/model.gltf')
```

### Removed Composables

| Old (v4) | New (v5) |
|----------|----------|
| `useTresReady()` | Use `@ready` event on `<TresCanvas>` |
| `useRenderLoop()` | Use `useLoop()` |
| `useTresEventManager()` | Built into event system |
| `useTexture()` | Moved to `@tresjs/cientos` |
| `useSeek()` | Use `useGraph()` |

### ESM Only

v5 is ESM-only. Update build configuration if using UMD.

---

## Resources

### Official Documentation
- [TresJS Docs](https://docs.tresjs.org/)
- [API Reference](https://docs.tresjs.org/api/)
- [Cookbook](https://docs.tresjs.org/cookbook/)

### Ecosystem
- [Cientos](https://cientos.tresjs.org/) - Component library
- [Post-Processing](https://tresjs.org/guide/post-processing.html)
- [Nuxt Module](https://github.com/Tresjs/nuxt)

### Community
- [GitHub](https://github.com/Tresjs/tres)
- [Discord](https://discord.gg/UCr96AQmWn)

### Three.js Resources
- [Three.js Documentation](https://threejs.org/docs/)
- [Three.js Examples](https://threejs.org/examples/)
