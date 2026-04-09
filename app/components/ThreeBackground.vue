<script setup lang="ts">
import * as THREE from 'three'

const canvasRef = ref<HTMLCanvasElement | null>(null)

onMounted(() => {
  if (!canvasRef.value) return

  const canvas = canvasRef.value
  let w = window.innerWidth
  let h = window.innerHeight

  // ── Renderer ──────────────────────────────────────────────
  const renderer = new THREE.WebGLRenderer({ canvas, antialias: true, alpha: true })
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  renderer.setSize(w, h)
  renderer.setClearColor(0x000000, 0)
  renderer.toneMapping = THREE.ACESFilmicToneMapping
  renderer.toneMappingExposure = 1.4

  // ── Scene & Camera ────────────────────────────────────────
  const scene = new THREE.Scene()
  scene.fog = new THREE.FogExp2(0x06060f, 0.016)

  const camera = new THREE.PerspectiveCamera(65, w / h, 0.1, 1000)
  camera.position.set(0, 4, 12)
  camera.lookAt(0, 0, 0)

  // ── Neon palette ──────────────────────────────────────────
  const C_CYAN   = 0x00ffff
  const C_PURPLE = 0x7b2fff
  const C_GREEN  = 0x00ff88
  const C_PINK   = 0xff007f
  const C_GOLD   = 0xffd700

  const neonPalette = [
    new THREE.Color(C_CYAN),
    new THREE.Color(C_PURPLE),
    new THREE.Color(C_GREEN),
    new THREE.Color(C_PINK),
  ]

  // ── Lights ────────────────────────────────────────────────
  const ambientLight = new THREE.AmbientLight(0x030318, 2)
  scene.add(ambientLight)

  const cyanLight = new THREE.PointLight(C_CYAN, 80, 30)
  cyanLight.position.set(-4, 4, 2)
  scene.add(cyanLight)

  const purpleLight = new THREE.PointLight(C_PURPLE, 60, 25)
  purpleLight.position.set(5, -2, -3)
  scene.add(purpleLight)

  const pinkLight = new THREE.PointLight(C_PINK, 50, 20)
  pinkLight.position.set(0, 6, 4)
  scene.add(pinkLight)

  // ── Star Particles ────────────────────────────────────────
  const PARTICLE_COUNT = 2800
  const positions = new Float32Array(PARTICLE_COUNT * 3)
  const colors    = new Float32Array(PARTICLE_COUNT * 3)

  for (let i = 0; i < PARTICLE_COUNT; i++) {
    positions[i * 3]     = (Math.random() - 0.5) * 80
    positions[i * 3 + 1] = (Math.random() - 0.5) * 50
    positions[i * 3 + 2] = (Math.random() - 0.5) * 80

    const c = neonPalette[Math.floor(Math.random() * neonPalette.length)]!
    colors[i * 3]     = c.r
    colors[i * 3 + 1] = c.g
    colors[i * 3 + 2] = c.b
  }

  const particleGeo = new THREE.BufferGeometry()
  particleGeo.setAttribute('position', new THREE.BufferAttribute(positions, 3))
  particleGeo.setAttribute('color',    new THREE.BufferAttribute(colors, 3))

  const particleMat = new THREE.PointsMaterial({
    size: 0.14,
    vertexColors: true,
    transparent: true,
    opacity: 0.9,
    blending: THREE.AdditiveBlending,
    depthWrite: false,
    sizeAttenuation: true,
  })

  const particles = new THREE.Points(particleGeo, particleMat)
  scene.add(particles)

  // ── Streaking / falling neon particles ───────────────────
  const STREAK_COUNT = 70
  const streakPositions  = new Float32Array(STREAK_COUNT * 3)
  const streakVelocities: number[] = []
  const streakColors     = new Float32Array(STREAK_COUNT * 3)

  for (let i = 0; i < STREAK_COUNT; i++) {
    streakPositions[i * 3]     = (Math.random() - 0.5) * 70
    streakPositions[i * 3 + 1] = 20 + Math.random() * 20
    streakPositions[i * 3 + 2] = (Math.random() - 0.5) * 70
    streakVelocities.push(-(0.08 + Math.random() * 0.18))
    const sc = neonPalette[Math.floor(Math.random() * neonPalette.length)]!
    streakColors[i * 3]     = sc.r
    streakColors[i * 3 + 1] = sc.g
    streakColors[i * 3 + 2] = sc.b
  }
  const streakGeo = new THREE.BufferGeometry()
  streakGeo.setAttribute('position', new THREE.BufferAttribute(streakPositions, 3))
  streakGeo.setAttribute('color',    new THREE.BufferAttribute(streakColors,    3))
  const streakMat = new THREE.PointsMaterial({
    size: 0.25,
    vertexColors: true,
    transparent: true,
    opacity: 0.9,
    blending: THREE.AdditiveBlending,
    depthWrite: false,
  })
  const streaks = new THREE.Points(streakGeo, streakMat)
  scene.add(streaks)

  // ── Central torus knot ────────────────────────────────────
  const torusKnotGeo = new THREE.TorusKnotGeometry(2.0, 0.5, 220, 32, 3, 5)
  const torusKnotMat = new THREE.MeshPhongMaterial({
    color: C_CYAN,
    emissive: new THREE.Color(C_CYAN),
    emissiveIntensity: 0.7,
    wireframe: true,
    transparent: true,
    opacity: 0.4,
  })
  const torusKnot = new THREE.Mesh(torusKnotGeo, torusKnotMat)
  torusKnot.position.set(0, 1, 0)
  scene.add(torusKnot)

  const innerGeo = new THREE.TorusKnotGeometry(1.5, 0.22, 140, 18, 3, 5)
  const innerMat = new THREE.MeshPhongMaterial({
    color: C_PURPLE,
    emissive: new THREE.Color(C_PURPLE),
    emissiveIntensity: 1.0,
    transparent: true,
    opacity: 0.22,
    blending: THREE.AdditiveBlending,
  })
  const innerKnot = new THREE.Mesh(innerGeo, innerMat)
  innerKnot.position.set(0, 1, 0)
  scene.add(innerKnot)

  // ── Pulsing neon rings ────────────────────────────────────
  const rings: THREE.Mesh[] = []
  const ringDefs = [
    { radius: 3.5, tube: 0.045, color: C_CYAN,   y: 1,  tilt: 0.0 },
    { radius: 4.8, tube: 0.035, color: C_PURPLE, y: 1,  tilt: Math.PI / 6 },
    { radius: 6.2, tube: 0.028, color: C_GREEN,  y: 1,  tilt: -Math.PI / 4 },
    { radius: 7.8, tube: 0.022, color: C_PINK,   y: 1,  tilt: Math.PI / 3 },
  ]
  ringDefs.forEach(({ radius, tube, color, y, tilt }) => {
    const geo = new THREE.TorusGeometry(radius, tube, 8, 128)
    const mat = new THREE.MeshPhongMaterial({
      color,
      emissive: new THREE.Color(color),
      emissiveIntensity: 1.4,
      transparent: true,
      opacity: 0.75,
      blending: THREE.AdditiveBlending,
    })
    const ring = new THREE.Mesh(geo, mat)
    ring.position.y = y
    ring.rotation.x = Math.PI / 2 + tilt
    rings.push(ring)
    scene.add(ring)
  })

  // ── Orbiting energy spheres ───────────────────────────────
  const orbitAnchor = new THREE.Group()
  orbitAnchor.position.set(0, 1, 0)
  scene.add(orbitAnchor)

  const orbiters: { mesh: THREE.Mesh; speed: number; radius: number; phase: number; yOff: number }[] = []
  const orbiterDefs = [
    { radius: 4.5, speed: 0.45, color: C_CYAN,   size: 0.22, phase: 0,              yOff: 0.5 },
    { radius: 4.5, speed: 0.45, color: C_PURPLE,  size: 0.22, phase: Math.PI,        yOff: -0.5 },
    { radius: 6.5, speed: 0.28, color: C_GREEN,   size: 0.18, phase: Math.PI / 2,    yOff: 1.2 },
    { radius: 6.5, speed: 0.28, color: C_PINK,    size: 0.18, phase: 3 * Math.PI/2,  yOff: -1.2 },
    { radius: 8.5, speed: 0.17, color: C_GOLD,    size: 0.15, phase: 1.0,            yOff: 0 },
  ]
  orbiterDefs.forEach(({ radius, speed, color, size, phase, yOff }) => {
    const geo  = new THREE.SphereGeometry(size, 14, 14)
    const mat  = new THREE.MeshPhongMaterial({
      color,
      emissive: new THREE.Color(color),
      emissiveIntensity: 1.8,
      transparent: true,
      opacity: 0.9,
      blending: THREE.AdditiveBlending,
    })
    const mesh = new THREE.Mesh(geo, mat)
    orbitAnchor.add(mesh)
    orbiters.push({ mesh, speed, radius, phase, yOff })
  })

  // ── Floating wireframe icosahedra ─────────────────────────
  const floaterGroup = new THREE.Group()
  const floaterDefs = [
    { x: -7, y: 2,   z: -5 }, { x:  7, y: -1, z: -6 },
    { x: -5, y: -3,  z:  3 }, { x:  8, y:  4, z: -3 },
    { x: -9, y:  0,  z: -2 }, { x:  5, y:  3, z:  3 },
    { x: -3, y:  5,  z: -7 }, { x:  9, y:  2, z:  2 },
    { x:  0, y: -5,  z: -5 },
  ]
  floaterDefs.forEach(({ x, y, z }) => {
    const size = 0.5 + Math.random() * 0.9
    const col  = neonPalette[Math.floor(Math.random() * neonPalette.length)]!
    const geo  = new THREE.IcosahedronGeometry(size, 0)
    const mat  = new THREE.MeshPhongMaterial({
      color: col, emissive: col, emissiveIntensity: 0.9,
      wireframe: true, transparent: true, opacity: 0.55,
    })
    const mesh = new THREE.Mesh(geo, mat)
    mesh.position.set(x, y, z)
    mesh.userData['speed'] = 0.004 + Math.random() * 0.006
    mesh.userData['phase'] = Math.random() * Math.PI * 2
    floaterGroup.add(mesh)
  })
  scene.add(floaterGroup)

  // ── Orbiting octahedra ring ───────────────────────────────
  const octaGroup = new THREE.Group()
  for (let i = 0; i < 10; i++) {
    const col = neonPalette[i % neonPalette.length]!
    const geo = new THREE.OctahedronGeometry(0.3 + Math.random() * 0.35, 0)
    const mat = new THREE.MeshPhongMaterial({
      color: col, emissive: col, emissiveIntensity: 0.9,
      transparent: true, opacity: 0.7,
    })
    const mesh = new THREE.Mesh(geo, mat)
    const angle = (i / 10) * Math.PI * 2
    mesh.position.set(Math.cos(angle) * 12, (Math.random() - 0.5) * 6, Math.sin(angle) * 12)
    mesh.userData['speed'] = 0.005 + Math.random() * 0.008
    octaGroup.add(mesh)
  }
  scene.add(octaGroup)

  // ── Synthwave scrolling grid ──────────────────────────────
  const grid1 = new THREE.GridHelper(120, 70, C_CYAN, 0x0a0a2e)
  grid1.position.y = -5
  ;(grid1.material as THREE.LineBasicMaterial).transparent = true
  ;(grid1.material as THREE.LineBasicMaterial).opacity = 0.3
  scene.add(grid1)

  const grid2 = new THREE.GridHelper(120, 140, C_PURPLE, 0x080818)
  grid2.position.y = -5
  ;(grid2.material as THREE.LineBasicMaterial).transparent = true
  ;(grid2.material as THREE.LineBasicMaterial).opacity = 0.12
  scene.add(grid2)

  // Horizon vertical plane
  const horizonGeo = new THREE.PlaneGeometry(120, 35, 28, 8)
  const horizonMat = new THREE.MeshBasicMaterial({
    color: C_PURPLE, wireframe: true,
    transparent: true, opacity: 0.07, side: THREE.DoubleSide,
  })
  const horizon = new THREE.Mesh(horizonGeo, horizonMat)
  horizon.position.set(0, 8, -40)
  scene.add(horizon)

  // ── Laser beam lines from floor to center ─────────────────
  const laserGroup = new THREE.Group()
  const laserColors = [C_CYAN, C_PURPLE, C_GREEN, C_PINK]
  for (let i = 0; i < 14; i++) {
    const angle  = (i / 14) * Math.PI * 2
    const r = 10
    const pts = [
      new THREE.Vector3(Math.cos(angle) * r, -5, Math.sin(angle) * r),
      new THREE.Vector3(0, 1, 0),
    ]
    const geo = new THREE.BufferGeometry().setFromPoints(pts)
    const mat = new THREE.LineBasicMaterial({
      color: laserColors[i % laserColors.length],
      transparent: true, opacity: 0.16,
      blending: THREE.AdditiveBlending,
    })
    laserGroup.add(new THREE.Line(geo, mat))
  }
  scene.add(laserGroup)

  // ── Hexagonal pillar columns ──────────────────────────────
  const pillarGroup = new THREE.Group()
  const pillarDefs = [[-8, -12], [8, -12], [-11, 3], [11, 3], [0, -14], [-5, -8], [5, -8]]
  pillarDefs.forEach(([px, pz]) => {
    const geo = new THREE.CylinderGeometry(0.28, 0.28, 12, 6, 1, true)
    const mat = new THREE.MeshPhongMaterial({
      color: C_CYAN, emissive: new THREE.Color(C_CYAN), emissiveIntensity: 0.6,
      transparent: true, opacity: 0.2, wireframe: true,
    })
    const mesh = new THREE.Mesh(geo, mat)
    mesh.position.set(px!, -1, pz!)
    pillarGroup.add(mesh)
  })
  scene.add(pillarGroup)

  // ── Mouse parallax ────────────────────────────────────────
  let mouseX = 0
  let mouseY = 0
  const onMouseMove = (e: MouseEvent) => {
    mouseX = (e.clientX / window.innerWidth  - 0.5) * 2
    mouseY = (e.clientY / window.innerHeight - 0.5) * 2
  }
  window.addEventListener('mousemove', onMouseMove)

  // ── Resize ────────────────────────────────────────────────
  const onResize = () => {
    w = window.innerWidth
    h = window.innerHeight
    camera.aspect = w / h
    camera.updateProjectionMatrix()
    renderer.setSize(w, h)
  }
  window.addEventListener('resize', onResize)

  // ── Animation loop ────────────────────────────────────────
  let frameId: number
  const clock = new THREE.Clock()

  const animate = () => {
    frameId = requestAnimationFrame(animate)
    const t = clock.getElapsedTime()

    // Star particle slow drift
    particles.rotation.y = t * 0.018
    particles.rotation.x = t * 0.007

    // Torus knot spin with color cycle
    torusKnot.rotation.x = t * 0.28
    torusKnot.rotation.y = t * 0.48
    innerKnot.rotation.x = t * 0.38
    innerKnot.rotation.y = t * 0.62
    torusKnotMat.emissive.setHSL((t * 0.05) % 1, 1, 0.5)

    // Pulsing rings rotate & breathe
    rings.forEach((ring, i) => {
      ring.rotation.z = t * (0.12 + i * 0.04)
      const pulse = 0.7 + 0.3 * Math.sin(t * 1.8 + i * 1.2)
      ;(ring.material as THREE.MeshPhongMaterial).opacity = pulse * 0.75
      ;(ring.material as THREE.MeshPhongMaterial).emissiveIntensity = 0.9 + 0.7 * pulse
    })

    // Orbiting spheres
    orbiters.forEach(({ mesh, speed, radius, phase, yOff }) => {
      const angle = t * speed + phase
      mesh.position.set(
        Math.cos(angle) * radius,
        yOff + Math.sin(t * 0.4 + phase) * 0.6,
        Math.sin(angle) * radius,
      )
    })

    // Floaters bob & spin
    floaterGroup.children.forEach((child, i) => {
      const m = child as THREE.Mesh
      m.rotation.x += m.userData['speed'] as number
      m.rotation.y += (m.userData['speed'] as number) * 1.4
      m.position.y += Math.sin(t * 0.55 + (m.userData['phase'] as number) + i) * 0.003
    })

    // Octahedra spin & orbit vertically
    octaGroup.children.forEach((child, i) => {
      const m = child as THREE.Mesh
      m.rotation.x += m.userData['speed'] as number
      m.rotation.z += (m.userData['speed'] as number) * 0.8
      m.position.y = Math.sin(t * 0.4 + i) * 2.5
    })

    // Laser beams rotate
    laserGroup.rotation.y = t * 0.10

    // Streaking particles rain down
    const sp = streakGeo.attributes['position'] as THREE.BufferAttribute
    const spArr = sp.array as Float32Array
    for (let i = 0; i < STREAK_COUNT; i++) {
      const vel = streakVelocities[i] ?? -0.1
      spArr[i * 3 + 1]! += vel
      if ((spArr[i * 3 + 1] ?? 0) < -10) {
        spArr[i * 3 + 1] = 30 + Math.random() * 20
        spArr[i * 3]     = (Math.random() - 0.5) * 70
        spArr[i * 3 + 2] = (Math.random() - 0.5) * 70
      }
    }
    sp.needsUpdate = true

    // Animate lights color cycle
    cyanLight.color.setHSL((t * 0.07) % 1, 1, 0.5)
    purpleLight.color.setHSL((t * 0.055 + 0.5) % 1, 1, 0.5)
    pinkLight.intensity = 40 + 20 * Math.sin(t * 2.0)

    // Synthwave scrolling grid
    grid1.position.z = (t * 2.0) % (120 / 70)
    grid2.position.z = (t * 2.8) % (120 / 140)

    // Mouse parallax camera with smooth lerp
    camera.position.x += (mouseX * 3.0 - camera.position.x) * 0.032
    camera.position.y += (-mouseY * 2.0 + 4 - camera.position.y) * 0.032
    camera.lookAt(0, 0, 0)

    renderer.render(scene, camera)
  }
  animate()

  // ── Cleanup ───────────────────────────────────────────────
  onBeforeUnmount(() => {
    cancelAnimationFrame(frameId)
    window.removeEventListener('mousemove', onMouseMove)
    window.removeEventListener('resize', onResize)
    renderer.dispose()
  })
})
</script>

<template>
  <canvas
    ref="canvasRef"
    class="fixed inset-0 w-full h-full pointer-events-none"
    style="z-index: 0;"
  />
</template>
