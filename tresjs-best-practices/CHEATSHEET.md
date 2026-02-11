# TresJS Cheat Sheet

Quick reference for common TresJS patterns and APIs.

---

## Installation

```bash
npm install @tresjs/core three
npm install -D @types/three
npm install @tresjs/cientos  # helpers
```

**Vite config:**
```typescript
import { templateCompilerOptions } from '@tresjs/core'
vue({ ...templateCompilerOptions })
```

---

## Basic Scene

```vue
<TresCanvas clear-color="#000" shadows>
  <TresPerspectiveCamera :position="[5, 5, 5]" />
  <TresMesh :cast-shadow="true">
    <TresBoxGeometry :args="[2, 2, 2]" />
    <TresMeshStandardMaterial color="#ff0000" />
  </TresMesh>
  <TresAmbientLight :intensity="0.5" />
  <TresDirectionalLight :position="[3, 3, 3]" :cast-shadow="true" />
</TresCanvas>
```

---

## Component Naming

| Three.js | TresJS |
|----------|--------|
| `new THREE.PerspectiveCamera()` | `<TresPerspectiveCamera />` |
| `new THREE.Mesh()` | `<TresMesh />` |
| `new THREE.BoxGeometry()` | `<TresBoxGeometry />` |

**Constructor args:** `:args="[...]"`

---

## Performance Rules

```typescript
// ✅ Good - shallow ref
const mesh = shallowRef<Mesh | null>(null)

// ❌ Bad - deep ref
const mesh = ref<Mesh>()
```

**Render modes:**
- `render-mode="always"` - continuous (animations)
- `render-mode="on-demand"` - auto invalidate (static)
- `render-mode="manual"` - full control

---

## Animation

**Simple (useLoop):**
```typescript
const { onBeforeRender } = useLoop()

onBeforeRender(({ delta, elapsed }) => {
  mesh.value.rotation.y += delta  // frame-independent
})
```

**Complex (GSAP):**
```typescript
gsap.to(mesh.value.position, { x: 5, duration: 2 })
```

---

## Loading Assets

**v5 API:**
```typescript
import { useLoader } from '@tresjs/core'

const { state: model, isLoading, error } = useLoader(
  GLTFLoader,
  '/model.gltf'
)
```

**Textures (from Cientos):**
```typescript
import { useTexture } from '@tresjs/cientos'

const { map } = await useTexture({ map: '/texture.jpg' })
```

---

## Events

```vue
<TresMesh
  @click="handleClick"
  @pointerdown="handleDown"
  @pointerenter="isHovered = true"
  @pointerleave="isHovered = false"
/>

<TresCanvas @pointermissed="clickedOutside" />
```

**Event object:**
```typescript
function handleClick(e: TresEvent) {
  e.intersect.object  // clicked mesh
  e.intersect.point   // world position
  e.stopPropagating()
}
```

---

## Common Props

**Transform:**
```vue
:position="[x, y, z]"
:rotation="[x, y, z]"
:scale="[x, y, z]"
```

**Material:**
```vue
color="#ff0000"
:roughness="0.5"
:metalness="0.8"
:opacity="0.5"
:transparent="true"
```

**Mesh:**
```vue
:visible="true"
:cast-shadow="true"
:receive-shadow="true"
```

---

## Lights

**Ambient:**
```vue
<TresAmbientLight :intensity="0.5" />
```

**Directional:**
```vue
<TresDirectionalLight
  :position="[3, 3, 3]"
  :intensity="1"
  :cast-shadow="true"
/>
```

**Point:**
```vue
<TresPointLight
  :position="[0, 2, 0]"
  :intensity="1"
  :distance="10"
/>
```

**Spot:**
```vue
<TresSpotLight
  :position="[5, 5, 0]"
  :angle="Math.PI / 4"
  :cast-shadow="true"
/>
```

---

## Shadows

```vue
<TresCanvas shadows :shadow-map-type="PCFSoftShadowMap">
  <TresDirectionalLight :cast-shadow="true" />
  <TresMesh :cast-shadow="true" :receive-shadow="true">
    <!-- ... -->
  </TresMesh>
</TresCanvas>
```

---

## Controls (from Cientos)

```vue
<script setup>
import { OrbitControls } from '@tresjs/cientos'
</script>

<template>
  <TresPerspectiveCamera :position="[5, 5, 5]" />
  <OrbitControls />  <!-- AFTER camera -->
</template>
```

---

## Helpers (Dev Only)

```vue
<TresAxesHelper :args="[5]" />
<TresGridHelper :args="[10, 10]" />
<TresDirectionalLight v-light-helper />
```

---

## Extend Catalogue

```typescript
import { extend } from '@tresjs/core'
import { OrbitControls } from 'three/addons/controls/OrbitControls'

extend({ OrbitControls })
```

Then: `<TresOrbitControls />`

---

## Context Access

```typescript
const { camera, renderer, scene, invalidate } = useTres()
```

---

## Primitives

```vue
<primitive :object="customObject" :dispose="disposeFunction" />
```

**Note:** Manual disposal required!

---

## Template Refs

```vue
<script setup lang="ts">
const meshRef = shallowRef<Mesh | null>(null)
</script>

<template>
  <TresMesh ref="meshRef">
    <!-- ... -->
  </TresMesh>
</template>
```

---

## Common Gotchas

- ❌ Camera at origin → ✅ Always set `:position`
- ❌ `ref()` for Three.js → ✅ Use `shallowRef()`
- ❌ Fixed animation speed → ✅ Use `delta` parameter
- ❌ Controls before camera → ✅ Camera first, then controls
- ❌ Changing `antialias` → ✅ Set once at creation

---

## v5 Changes

| Old (v4) | New (v5) |
|----------|----------|
| `@pointer-down` | `@pointerdown` |
| `await useLoader()` | `const { state } = useLoader()` |
| `useTexture` (core) | `useTexture` (cientos) |
| `useTresReady()` | `@ready` event |

---

## TresCanvas Props

**Reactive:**
- `clear-color` / `clear-alpha`
- `shadows` / `shadow-map-type`
- `dpr`
- `tone-mapping` / `tone-mapping-exposure`
- `render-mode`

**Non-reactive (set once):**
- `alpha`
- `antialias`
- `depth`
- `logarithmic-depth-buffer`

---

## Cientos Highlights

```typescript
// Loaders
import { useGLTF, useFBX, useTexture } from '@tresjs/cientos'

// Components
import {
  OrbitControls,
  Text3D,
  Environment,
  ContactShadows,
  Stars
} from '@tresjs/cientos'
```

---

## Resources

- 📖 [Docs](https://docs.tresjs.org/)
- 🧰 [Cientos](https://cientos.tresjs.org/)
- 💬 [Discord](https://discord.gg/UCr96AQmWn)
- 📦 [GitHub](https://github.com/Tresjs/tres)
