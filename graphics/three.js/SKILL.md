---
name: three-js
description: A complete reference for using Three.js via CDN (no build tools required).  
  Current stable version: **r0.180.0** (May 2026)  
  Official docs: https://threejs.org/docs/  
  LLM-optimized reference: https://threejs.org/docs/llms.txt

license: Complete terms in LICENSE.txt
---
# Three.js Skill — CDN Reference & Usage Guide
## 1. CDN Setup (Import Maps — Preferred Pattern)

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Three.js Scene</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { overflow: hidden; background: #000; }
    canvas { display: block; }
  </style>
</head>
<body>
  <!-- Import Map: tells the browser how to resolve bare module specifiers -->
  <script type="importmap">
  {
    "imports": {
      "three": "https://cdn.jsdelivr.net/npm/three@0.180.0/build/three.module.js",
      "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.180.0/examples/jsm/"
    }
  }
  </script>

  <script type="module">
    import * as THREE from 'three';
    import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
    import { GLTFLoader }    from 'three/addons/loaders/GLTFLoader.js';

    // Your scene code here…
  </script>
</body>
</html>
```

> ⚠️ **Never use the legacy `<script src="three.min.js">` pattern** — it pollutes the global scope  
> and breaks tree-shaking. Import Maps work in all modern browsers (Chrome 89+, Firefox 108+, Safari 16.4+).

### Alternative: jsDelivr direct import (no importmap needed)

```js
import * as THREE from 'https://cdn.jsdelivr.net/npm/three@0.180.0/build/three.module.js';
import { OrbitControls } from 'https://cdn.jsdelivr.net/npm/three@0.180.0/examples/jsm/controls/OrbitControls.js';
```

---

## 2. Minimal Boilerplate Scene

```js
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

// --- Scene, Camera, Renderer ---
const scene    = new THREE.Scene();
const camera   = new THREE.PerspectiveCamera(75, innerWidth / innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer({ antialias: true });

renderer.setSize(innerWidth, innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
renderer.shadowMap.enabled = true;
document.body.appendChild(renderer.domElement);

camera.position.set(0, 2, 5);

// --- Lights ---
scene.add(new THREE.AmbientLight(0xffffff, 0.5));
const sun = new THREE.DirectionalLight(0xffffff, 1);
sun.position.set(5, 10, 5);
sun.castShadow = true;
scene.add(sun);

// --- Object ---
const mesh = new THREE.Mesh(
  new THREE.BoxGeometry(1, 1, 1),
  new THREE.MeshStandardMaterial({ color: 0x4fc3f7 })
);
mesh.castShadow = true;
scene.add(mesh);

// --- Controls ---
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;

// --- Resize ---
window.addEventListener('resize', () => {
  camera.aspect = innerWidth / innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(innerWidth, innerHeight);
});

// --- Render Loop ---
function animate() {
  requestAnimationFrame(animate);
  mesh.rotation.y += 0.01;
  controls.update();           // required when damping is enabled
  renderer.render(scene, camera);
}
animate();
```

---

## 3. Core Architecture

Three.js follows a **Scene Graph** pattern:

```
Scene
├── Camera (not rendered, just a perspective point)
├── Lights
│   ├── AmbientLight
│   └── DirectionalLight
└── Object3D / Group
    └── Mesh
        ├── BufferGeometry  (vertices, UVs, normals)
        └── Material        (appearance/shader)
```

Every visible object is a `Mesh = Geometry + Material`.  
Every entity in the scene inherits from `Object3D`.

---

## 4. Essential Classes

### 4.1 Scene
```js
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x111111);    // solid color
scene.background = new THREE.CubeTextureLoader().load([...]);  // skybox
scene.fog = new THREE.Fog(0x000000, 10, 100);    // linear fog
scene.fog = new THREE.FogExp2(0x000000, 0.02);   // exponential fog
scene.add(object);     // add any Object3D
scene.remove(object);  // remove it
```

---

### 4.2 Cameras
```js
// PerspectiveCamera(fov, aspect, near, far)
const camera = new THREE.PerspectiveCamera(75, innerWidth / innerHeight, 0.1, 1000);
camera.position.set(x, y, z);
camera.lookAt(0, 0, 0);   // or camera.lookAt(new THREE.Vector3(x,y,z))

// OrthographicCamera(left, right, top, bottom, near, far)
const w = innerWidth / innerHeight;
const ortho = new THREE.OrthographicCamera(-w, w, 1, -1, 0.1, 100);
```

| Property | Description |
|---|---|
| `fov` | Vertical field of view in degrees (50-75 typical) |
| `aspect` | Must be updated on resize |
| `near / far` | Clipping planes — keep ratio ≤ 10000 for precision |

---

### 4.3 Renderer
```js
const renderer = new THREE.WebGLRenderer({
  antialias: true,        // MSAA
  alpha: true,            // transparent background
  powerPreference: 'high-performance',
  canvas: document.querySelector('#canvas'),  // attach to existing canvas
});

renderer.setSize(innerWidth, innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio, 2));  // cap at 2 for perf
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;       // softer shadows
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1.0;
renderer.outputColorSpace = THREE.SRGBColorSpace;
```

**WebGPURenderer** (cutting edge, requires newer browser):
```js
import { WebGPURenderer } from 'three/addons/renderers/common/WebGPURenderer.js';
const renderer = new WebGPURenderer();
await renderer.init();
```

---

### 4.4 Geometries

| Class | Constructor | Use Case |
|---|---|---|
| `BoxGeometry` | `(w, h, d, wSeg, hSeg, dSeg)` | Cubes, walls |
| `SphereGeometry` | `(r, widthSeg, heightSeg)` | Balls, planets |
| `PlaneGeometry` | `(w, h, wSeg, hSeg)` | Floors, screens |
| `CylinderGeometry` | `(rTop, rBot, h, radSeg)` | Pillars, pipes |
| `TorusGeometry` | `(r, tube, radSeg, tubeSeg)` | Rings, donuts |
| `TorusKnotGeometry` | `(r, tube, tubSeg, radSeg, p, q)` | Decorative knots |
| `IcosahedronGeometry` | `(r, detail)` | Smooth spheres |
| `BufferGeometry` | custom | Custom meshes |

```js
// Custom geometry from vertices
const geo = new THREE.BufferGeometry();
const vertices = new Float32Array([
  0,  1, 0,   // vertex 0
 -1, -1, 0,   // vertex 1
  1, -1, 0,   // vertex 2
]);
geo.setAttribute('position', new THREE.BufferAttribute(vertices, 3));
geo.computeVertexNormals();  // required for lighting
```

---

### 4.5 Materials

| Material | Lighting | Use Case |
|---|---|---|
| `MeshBasicMaterial` | None | Flat color, wireframes, debug |
| `MeshLambertMaterial` | Diffuse | Fast, low-quality shading |
| `MeshPhongMaterial` | Phong | Specular highlights |
| `MeshStandardMaterial` | PBR | General purpose (recommended) |
| `MeshPhysicalMaterial` | PBR+ | Clearcoat, transmission, iridescence |
| `MeshToonMaterial` | Toon | Cel-shading |
| `ShaderMaterial` | Custom GLSL | Full control |
| `PointsMaterial` | None | Particle systems |
| `LineBasicMaterial` | None | Lines/wireframes |

```js
// Standard PBR material (most commonly used)
const mat = new THREE.MeshStandardMaterial({
  color:       0x4fc3f7,   // or '#4fc3f7' or new THREE.Color(r, g, b)
  metalness:   0.5,        // 0 = plastic, 1 = full metal
  roughness:   0.3,        // 0 = mirror, 1 = fully rough
  map:         texture,    // albedo texture
  normalMap:   normalTex,  // bump details
  envMap:      envTexture, // reflections
  transparent: true,
  opacity:     0.8,
  wireframe:   false,
  side:        THREE.FrontSide,  // FrontSide | BackSide | DoubleSide
});

// Update at runtime
mat.color.set(0xff0000);
mat.needsUpdate = true;  // required after changing certain properties
```

---

### 4.6 Lights

| Light | Description | Shadows |
|---|---|---|
| `AmbientLight` | Global, no direction | ❌ |
| `DirectionalLight` | Parallel rays, like the sun | ✅ |
| `PointLight` | Radiates from a point | ✅ |
| `SpotLight` | Cone-shaped beam | ✅ |
| `HemisphereLight` | Sky/ground gradient | ❌ |
| `RectAreaLight` | Area light (needs LTC maps) | ❌ |

```js
// Typical 3-point lighting setup
const ambient = new THREE.AmbientLight(0xffffff, 0.3);
scene.add(ambient);

const key = new THREE.DirectionalLight(0xffffff, 1.0);
key.position.set(5, 10, 5);
key.castShadow = true;
key.shadow.mapSize.set(2048, 2048);  // shadow resolution
key.shadow.camera.near = 0.1;
key.shadow.camera.far  = 50;
scene.add(key);

const fill = new THREE.PointLight(0x8888ff, 0.5, 20);
fill.position.set(-5, 3, -5);
scene.add(fill);

// Object must opt-in to shadows
mesh.castShadow    = true;
mesh.receiveShadow = true;
```

---

### 4.7 Mesh & Object3D

Every `Object3D` (Mesh, Group, Light, Camera…) has:

```js
// Position, Rotation, Scale
object.position.set(1, 2, 3);
object.position.x = 1;
object.rotation.y = Math.PI / 4;  // radians
object.scale.set(2, 2, 2);

// Hierarchy
const group = new THREE.Group();
group.add(mesh1, mesh2);    // mesh positions become relative to group
scene.add(group);

// Traversal
scene.traverse(obj => {
  if (obj.isMesh) obj.material.wireframe = true;
});

// Visibility & layers
mesh.visible = false;
mesh.layers.set(1);
camera.layers.enable(1);

// userData (custom metadata, no impact on rendering)
mesh.userData = { id: 'player', health: 100 };

// World position (ignores parent transforms)
const worldPos = new THREE.Vector3();
mesh.getWorldPosition(worldPos);
```

---

## 5. Textures & Loaders

### TextureLoader
```js
const loader = new THREE.TextureLoader();
const texture = loader.load(
  './texture.jpg',
  (tex) => console.log('loaded'),     // onLoad
  undefined,                           // onProgress (not supported)
  (err) => console.error(err)         // onError
);
texture.colorSpace  = THREE.SRGBColorSpace;  // required for color maps
texture.wrapS       = THREE.RepeatWrapping;
texture.wrapT       = THREE.RepeatWrapping;
texture.repeat.set(4, 4);
```

### LoadingManager (track multiple assets)
```js
const manager = new THREE.LoadingManager(
  () => console.log('All loaded'),
  (url, loaded, total) => console.log(`${loaded}/${total} — ${url}`),
  (url) => console.error(`Error: ${url}`)
);
const texLoader  = new THREE.TextureLoader(manager);
const gltfLoader = new GLTFLoader(manager);
```

### GLTFLoader (3D Models)
```js
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import { DRACOLoader } from 'three/addons/loaders/DRACOLoader.js';

const dracoLoader = new DRACOLoader();
dracoLoader.setDecoderPath('https://cdn.jsdelivr.net/npm/three@0.180.0/examples/jsm/libs/draco/');

const gltfLoader = new GLTFLoader();
gltfLoader.setDRACOLoader(dracoLoader);

gltfLoader.load('./model.glb', (gltf) => {
  const model = gltf.scene;
  model.position.set(0, 0, 0);
  // Traverse and enable shadows
  model.traverse(child => {
    if (child.isMesh) {
      child.castShadow = true;
      child.receiveShadow = true;
    }
  });
  scene.add(model);

  // Play animations
  const mixer = new THREE.AnimationMixer(model);
  gltf.animations.forEach(clip => mixer.clipAction(clip).play());
});
```

### Environment Maps (HDR reflections)
```js
import { RGBELoader } from 'three/addons/loaders/RGBELoader.js';

new RGBELoader().load('./env.hdr', (hdr) => {
  hdr.mapping = THREE.EquirectangularReflectionMapping;
  scene.environment = hdr;   // affects all PBR materials
  scene.background  = hdr;   // visible skybox
});
```

---

## 6. Controls (addons)

### OrbitControls (most common)
```js
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping  = true;    // smooth inertia
controls.dampingFactor  = 0.05;
controls.autoRotate     = true;
controls.autoRotateSpeed = 1.0;
controls.minDistance    = 2;
controls.maxDistance    = 50;
controls.maxPolarAngle  = Math.PI / 2;  // prevent going underground
controls.target.set(0, 1, 0);          // orbit around a point
controls.update();   // MUST call in animation loop when damping/autoRotate enabled
```

### Other Controls
```js
import { FlyControls }       from 'three/addons/controls/FlyControls.js';
import { PointerLockControls } from 'three/addons/controls/PointerLockControls.js';
import { TransformControls } from 'three/addons/controls/TransformControls.js';
import { FirstPersonControls } from 'three/addons/controls/FirstPersonControls.js';
```

---

## 7. Animation System

### requestAnimationFrame Loop
```js
const clock = new THREE.Clock();

function animate() {
  requestAnimationFrame(animate);

  const delta     = clock.getDelta();    // seconds since last frame
  const elapsed   = clock.getElapsedTime();  // total time

  // Use delta for frame-rate independent movement
  mesh.rotation.y += 1.0 * delta;  // 1 radian per second
  mesh.position.y  = Math.sin(elapsed) * 2;

  controls.update();
  renderer.render(scene, camera);
}
animate();
```

### AnimationMixer (GLTF / Keyframe Animations)
```js
const mixer = new THREE.AnimationMixer(model);
const action = mixer.clipAction(gltf.animations[0]);
action.play();
action.setLoop(THREE.LoopRepeat, Infinity);  // or LoopOnce, LoopPingPong
action.timeScale = 2.0;    // speed multiplier
action.weight    = 0.5;    // blend weight

// In animation loop:
mixer.update(delta);

// Crossfade between actions
action1.crossFadeTo(action2, 0.5, true);
```

### Tweening with GSAP (recommended for UI animations)
```js
import gsap from 'https://cdn.jsdelivr.net/npm/gsap@3/+esm';

gsap.to(mesh.position, { x: 5, duration: 2, ease: 'power2.inOut' });
gsap.to(mesh.rotation, { y: Math.PI * 2, duration: 3, repeat: -1, ease: 'none' });
gsap.to(camera.position, { z: 3, duration: 1.5, onUpdate: () => controls.update() });
```

---

## 8. Math Utilities

```js
// Vectors
const v  = new THREE.Vector3(1, 2, 3);
v.normalize();
v.multiplyScalar(5);
v.add(new THREE.Vector3(0, 1, 0));
const dist = v.distanceTo(other);
const dot  = v.dot(other);
const cross = new THREE.Vector3().crossVectors(v, other);

// Color
const c = new THREE.Color('#4fc3f7');
c.setHSL(0.6, 1.0, 0.5);   // hue (0-1), sat, lightness
c.lerp(otherColor, 0.5);    // blend

// Quaternion / Euler (rotation)
const euler = new THREE.Euler(0, Math.PI / 4, 0, 'XYZ');
mesh.quaternion.setFromEuler(euler);
mesh.quaternion.slerp(targetQuat, 0.05);  // smooth rotation

// Matrix4
const matrix = new THREE.Matrix4();
matrix.setPosition(1, 2, 3);
mesh.applyMatrix4(matrix);

// Raycasting (mouse picking)
const raycaster = new THREE.Raycaster();
const mouse = new THREE.Vector2();

window.addEventListener('click', (e) => {
  mouse.x =  (e.clientX / innerWidth)  * 2 - 1;
  mouse.y = -(e.clientY / innerHeight) * 2 + 1;
  raycaster.setFromCamera(mouse, camera);
  const hits = raycaster.intersectObjects(scene.children, true);
  if (hits.length > 0) console.log('Hit:', hits[0].object.name);
});

// MathUtils
THREE.MathUtils.clamp(value, 0, 1);
THREE.MathUtils.lerp(a, b, t);
THREE.MathUtils.degToRad(90);
THREE.MathUtils.randFloat(0, 1);
```

---

## 9. Particles

```js
// Basic particle system
const count = 5000;
const positions = new Float32Array(count * 3);
for (let i = 0; i < count * 3; i++) {
  positions[i] = (Math.random() - 0.5) * 20;
}

const geo = new THREE.BufferGeometry();
geo.setAttribute('position', new THREE.BufferAttribute(positions, 3));

const mat = new THREE.PointsMaterial({
  size:         0.05,
  sizeAttenuation: true,   // smaller when far
  color:        0xffffff,
  transparent:  true,
  alphaMap:     particleTexture,
  depthWrite:   false,     // prevents sorting artifacts
  blending:     THREE.AdditiveBlending,
});

const particles = new THREE.Points(geo, mat);
scene.add(particles);

// Animate individual particles
const posAttr = particles.geometry.attributes.position;
for (let i = 0; i < count; i++) {
  posAttr.setY(i, Math.sin(elapsed + i * 0.01));
}
posAttr.needsUpdate = true;  // REQUIRED after modifying
```

---

## 10. Post-Processing (EffectComposer)

```js
import { EffectComposer }    from 'three/addons/postprocessing/EffectComposer.js';
import { RenderPass }        from 'three/addons/postprocessing/RenderPass.js';
import { UnrealBloomPass }   from 'three/addons/postprocessing/UnrealBloomPass.js';
import { FilmPass }          from 'three/addons/postprocessing/FilmPass.js';
import { SSAOPass }          from 'three/addons/postprocessing/SSAOPass.js';

const composer = new EffectComposer(renderer);
composer.addPass(new RenderPass(scene, camera));

const bloom = new UnrealBloomPass(
  new THREE.Vector2(innerWidth, innerHeight),
  0.5,   // strength
  0.4,   // radius
  0.85   // threshold
);
composer.addPass(bloom);

// Replace renderer.render in animation loop:
composer.render();
```

---

## 11. Custom Shaders (ShaderMaterial)

```js
const mat = new THREE.ShaderMaterial({
  uniforms: {
    uTime:  { value: 0.0 },
    uColor: { value: new THREE.Color(0x4fc3f7) },
  },
  vertexShader: `
    uniform float uTime;
    varying vec2 vUv;
    void main() {
      vUv = uv;
      vec3 pos = position;
      pos.y += sin(pos.x * 3.0 + uTime) * 0.1;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
    }
  `,
  fragmentShader: `
    uniform vec3 uColor;
    varying vec2 vUv;
    void main() {
      gl_FragColor = vec4(uColor * vUv.x, 1.0);
    }
  `,
  transparent: false,
  side: THREE.DoubleSide,
});

// Update uniforms in animation loop:
mat.uniforms.uTime.value = elapsed;
```

---

## 12. Performance Tips

| Technique | Code |
|---|---|
| **Dispose geometry** | `geo.dispose(); mat.dispose(); texture.dispose();` |
| **InstancedMesh for repeated objects** | `new THREE.InstancedMesh(geo, mat, count)` |
| **Frustum culling** | Enabled by default; set `object.frustumCulled = true` |
| **Level of Detail** | `import { LOD } from 'three'` |
| **mergeGeometries** | `import { mergeGeometries } from 'three/addons/utils/BufferGeometryUtils.js'` |
| **Cap pixel ratio** | `renderer.setPixelRatio(Math.min(devicePixelRatio, 2))` |
| **Use BufferGeometry** | Always — never legacy Geometry |
| **Static objects** | `mesh.matrixAutoUpdate = false; mesh.updateMatrix()` |
| **Texture sizes** | Must be powers of 2 (512, 1024, 2048…) |

```js
// InstancedMesh — draw 10,000 cubes in one draw call
const iMesh = new THREE.InstancedMesh(geo, mat, 10000);
const dummy = new THREE.Object3D();
for (let i = 0; i < 10000; i++) {
  dummy.position.set(Math.random() * 40 - 20, 0, Math.random() * 40 - 20);
  dummy.updateMatrix();
  iMesh.setMatrixAt(i, dummy.matrix);
}
iMesh.instanceMatrix.needsUpdate = true;
scene.add(iMesh);
```

---

## 13. UI — HTML Overlay / Labels

```js
// CSS2DRenderer — HTML labels that track 3D positions
import { CSS2DRenderer, CSS2DObject } from 'three/addons/renderers/CSS2DRenderer.js';

const labelRenderer = new CSS2DRenderer();
labelRenderer.setSize(innerWidth, innerHeight);
labelRenderer.domElement.style.position = 'absolute';
labelRenderer.domElement.style.top = '0px';
labelRenderer.domElement.style.pointerEvents = 'none';
document.body.appendChild(labelRenderer.domElement);

const div = document.createElement('div');
div.textContent = 'Hello 3D World';
div.style.color = 'white';
const label = new CSS2DObject(div);
label.position.set(0, 1.5, 0);
mesh.add(label);

// In animation loop, also call:
labelRenderer.render(scene, camera);
```

---

## 14. Stats & Debugging

```js
// FPS counter
import Stats from 'three/addons/libs/stats.module.js';
const stats = new Stats();
document.body.appendChild(stats.dom);
// In loop: stats.update();

// GUI (dat.GUI / lil-gui)
import GUI from 'https://cdn.jsdelivr.net/npm/lil-gui@0.19/+esm';
const gui = new GUI();
const params = { color: '#4fc3f7', metalness: 0.5, speed: 1.0 };
gui.addColor(params, 'color').onChange(v => mat.color.set(v));
gui.add(params, 'metalness', 0, 1).onChange(v => mat.metalness = v);
gui.add(params, 'speed', 0, 5);

// Helper objects (visible in dev, remove in prod)
scene.add(new THREE.AxesHelper(5));           // X=red, Y=green, Z=blue
scene.add(new THREE.GridHelper(20, 20));
scene.add(new THREE.CameraHelper(camera));
scene.add(new THREE.DirectionalLightHelper(sun, 1));
scene.add(new THREE.PointLightHelper(fill, 0.3));
```

---

## 15. Common Addon Paths (jsDelivr)

```
BASE = https://cdn.jsdelivr.net/npm/three@0.180.0/examples/jsm/

Controls:
  controls/OrbitControls.js
  controls/FlyControls.js
  controls/PointerLockControls.js
  controls/TransformControls.js

Loaders:
  loaders/GLTFLoader.js
  loaders/DRACOLoader.js
  loaders/RGBELoader.js
  loaders/FontLoader.js
  loaders/OBJLoader.js
  loaders/FBXLoader.js
  loaders/SVGLoader.js

Post-Processing:
  postprocessing/EffectComposer.js
  postprocessing/RenderPass.js
  postprocessing/UnrealBloomPass.js
  postprocessing/SSAOPass.js
  postprocessing/FilmPass.js
  postprocessing/GlitchPass.js
  postprocessing/OutlinePass.js

Renderers:
  renderers/CSS2DRenderer.js
  renderers/CSS3DRenderer.js

Utils:
  utils/BufferGeometryUtils.js
  math/OctTree.js

Misc:
  objects/Sky.js
  objects/Water.js
  objects/Reflector.js
  geometries/TextGeometry.js
  helpers/VertexNormalsHelper.js
```

---

## 16. Quick Recipe: Responsive Full-Page Canvas

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Three.js App</title>
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
    body { overflow: hidden; background: #000; }
    canvas { display: block; width: 100vw; height: 100vh; }
  </style>
</head>
<body>
  <script type="importmap">
  {
    "imports": {
      "three": "https://cdn.jsdelivr.net/npm/three@0.180.0/build/three.module.js",
      "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.180.0/examples/jsm/"
    }
  }
  </script>
  <script type="module">
    import * as THREE from 'three';
    import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

    const scene    = new THREE.Scene();
    const camera   = new THREE.PerspectiveCamera(60, innerWidth / innerHeight, 0.1, 200);
    const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: false });
    renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
    renderer.setSize(innerWidth, innerHeight);
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.PCFSoftShadowMap;
    renderer.toneMapping = THREE.ACESFilmicToneMapping;
    renderer.outputColorSpace = THREE.SRGBColorSpace;
    document.body.appendChild(renderer.domElement);

    scene.background = new THREE.Color('#111827');

    camera.position.set(0, 3, 8);
    const controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;
    controls.target.set(0, 1, 0);

    // Lights
    scene.add(new THREE.AmbientLight('#ffffff', 0.4));
    const dir = new THREE.DirectionalLight('#ffffff', 1.5);
    dir.position.set(5, 10, 5);
    dir.castShadow = true;
    dir.shadow.mapSize.setScalar(2048);
    scene.add(dir);

    // Geometry
    const mesh = new THREE.Mesh(
      new THREE.TorusKnotGeometry(1, 0.3, 120, 20),
      new THREE.MeshStandardMaterial({ color: '#4fc3f7', metalness: 0.6, roughness: 0.2 })
    );
    mesh.castShadow = true;
    mesh.position.y = 1.5;
    scene.add(mesh);

    const floor = new THREE.Mesh(
      new THREE.PlaneGeometry(20, 20),
      new THREE.MeshStandardMaterial({ color: '#1f2937' })
    );
    floor.rotation.x = -Math.PI / 2;
    floor.receiveShadow = true;
    scene.add(floor);

    // Resize
    window.addEventListener('resize', () => {
      camera.aspect = innerWidth / innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(innerWidth, innerHeight);
    });

    // Loop
    const clock = new THREE.Clock();
    (function animate() {
      requestAnimationFrame(animate);
      const t = clock.getElapsedTime();
      mesh.rotation.y = t * 0.5;
      mesh.rotation.x = t * 0.3;
      controls.update();
      renderer.render(scene, camera);
    })();
  </script>
</body>
</html>
```

---

## 17. Key Links

| Resource | URL |
|---|---|
| Official Docs | https://threejs.org/docs/ |
| Manual / Tutorials | https://threejs.org/manual/ |
| Examples (300+) | https://threejs.org/examples/ |
| Editor (no-code) | https://threejs.org/editor/ |
| LLMs.txt reference | https://threejs.org/docs/llms.txt |
| GitHub | https://github.com/mrdoob/three.js |
| Discourse Forum | https://discourse.threejs.org/ |
| jsDelivr CDN | https://cdn.jsdelivr.net/npm/three@0.180.0/ |
| cdnjs | https://cdnjs.com/libraries/three.js |
| Bruno Simon's course | https://threejs-journey.com |

