<script setup lang="ts">
import * as THREE from 'three'
import { OrbitControls } from 'three/addons/controls/OrbitControls.js'

const canvasRef = ref<HTMLCanvasElement | null>(null)

onMounted(() => {
  if (!canvasRef.value) return

  const canvas = canvasRef.value
  const container = canvas.parentElement!
  const W = container.clientWidth || 320
  const H = container.clientHeight || 320

  // ── Renderer ─────────────────────────────────────
  const renderer = new THREE.WebGLRenderer({ canvas, antialias: true, alpha: true })
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  renderer.setSize(W, H)
  renderer.setClearColor(0x000000, 0)          // transparent background
  renderer.shadowMap.enabled = true
  renderer.shadowMap.type = THREE.PCFSoftShadowMap
  renderer.toneMapping = THREE.ACESFilmicToneMapping
  renderer.toneMappingExposure = 1.2

  // ── Scene & Camera ────────────────────────────────
  const scene = new THREE.Scene()

  const camera = new THREE.PerspectiveCamera(45, W / H, 0.1, 100)
  camera.position.set(0, 1.4, 5.5)

  // ── Lights ────────────────────────────────────────
  scene.add(new THREE.AmbientLight(0xffffff, 0.6))

  const sun = new THREE.DirectionalLight(0xffffff, 2.5)
  sun.position.set(4, 8, 5)
  sun.castShadow = true
  sun.shadow.mapSize.set(512, 512)
  scene.add(sun)

  const fillLight = new THREE.DirectionalLight(0xa0c8ff, 0.8)
  fillLight.position.set(-4, 2, -3)
  scene.add(fillLight)

  const rimLight = new THREE.DirectionalLight(0xff88cc, 0.5)
  rimLight.position.set(0, -2, -5)
  scene.add(rimLight)

  // ── Materials ─────────────────────────────────────
  const bodyMat  = new THREE.MeshStandardMaterial({ color: 0xf5c97a, roughness: 0.7, metalness: 0 })
  const darkMat  = new THREE.MeshStandardMaterial({ color: 0x3a2010, roughness: 0.8 })
  const whiteMat = new THREE.MeshStandardMaterial({ color: 0xfdf0d5, roughness: 0.7 })
  const pinkMat  = new THREE.MeshStandardMaterial({ color: 0xffaacc, roughness: 0.6 })
  const eyeWhite = new THREE.MeshStandardMaterial({ color: 0xffffff, roughness: 0.3 })
  const pupilMat = new THREE.MeshStandardMaterial({ color: 0x111111, roughness: 0.5 })
  const noseMat  = new THREE.MeshStandardMaterial({ color: 0xff6688, roughness: 0.5 })
  const whiskerMat = new THREE.MeshStandardMaterial({ color: 0xcccccc, roughness: 0.6 })
  const pawMat   = new THREE.MeshStandardMaterial({ color: 0xeead65, roughness: 0.8 })
  const shadowMat = new THREE.MeshStandardMaterial({ color: 0x000000, transparent: true, opacity: 0.18, roughness: 1 })

  const cat = new THREE.Group()
  scene.add(cat)

  // helper
  const mesh = (geo: THREE.BufferGeometry, mat: THREE.Material) => {
    const m = new THREE.Mesh(geo, mat)
    m.castShadow = true
    return m
  }

  // ── BODY ──────────────────────────────────────────
  const bodySphere = mesh(new THREE.SphereGeometry(0.75, 24, 18), bodyMat)
  bodySphere.scale.set(1, 0.88, 0.9)
  bodySphere.position.set(0, 0, 0)
  cat.add(bodySphere)

  // belly patch
  const belly = mesh(new THREE.SphereGeometry(0.5, 16, 12), whiteMat)
  belly.scale.set(0.68, 0.74, 0.3)
  belly.position.set(0, -0.05, 0.68)
  cat.add(belly)

  // ── HEAD ──────────────────────────────────────────
  const head = new THREE.Group()
  head.position.set(0, 0.98, 0.1)
  cat.add(head)

  const headSphere = mesh(new THREE.SphereGeometry(0.52, 24, 18), bodyMat)
  headSphere.scale.set(1, 0.96, 0.96)
  head.add(headSphere)

  // muzzle
  const muzzle = mesh(new THREE.SphereGeometry(0.22, 16, 12), whiteMat)
  muzzle.scale.set(1, 0.6, 0.7)
  muzzle.position.set(0, -0.1, 0.45)
  head.add(muzzle)

  // nose
  const nose = mesh(new THREE.SphereGeometry(0.055, 12, 8), noseMat)
  nose.position.set(0, -0.02, 0.68)
  head.add(nose)

  // ── EYES ──────────────────────────────────────────
  for (const side of [-1, 1]) {
    const eyeGroup = new THREE.Group()
    eyeGroup.position.set(side * 0.19, 0.12, 0.44)
    head.add(eyeGroup)

    const eyeW = mesh(new THREE.SphereGeometry(0.1, 16, 12), eyeWhite)
    eyeGroup.add(eyeW)

    const pupil = mesh(new THREE.SphereGeometry(0.062, 12, 8), pupilMat)
    pupil.position.set(0, 0, 0.065)
    eyeGroup.add(pupil)

    // shine dot
    const shine = mesh(new THREE.SphereGeometry(0.022, 8, 6), eyeWhite)
    shine.position.set(0.02, 0.03, 0.12)
    eyeGroup.add(shine)
  }

  // ── EARS ──────────────────────────────────────────
  for (const side of [-1, 1]) {
    const ear = new THREE.Group()
    ear.position.set(side * 0.3, 0.44, 0.04)
    ear.rotation.z = side * 0.18
    ear.rotation.x = -0.1
    head.add(ear)

    const outerEar = mesh(new THREE.ConeGeometry(0.22, 0.38, 16), bodyMat)
    ear.add(outerEar)

    const innerEar = mesh(new THREE.ConeGeometry(0.11, 0.24, 16), pinkMat)
    innerEar.position.set(0, 0.04, 0.04)
    innerEar.scale.z = 0.4
    ear.add(innerEar)
  }

  // ── WHISKERS ──────────────────────────────────────
  for (const side of [-1, 1]) {
    for (let i = 0; i < 3; i++) {
      const w = mesh(new THREE.CylinderGeometry(0.007, 0.003, 0.5, 4), whiskerMat)
      w.rotation.z = Math.PI / 2
      w.rotation.x = (i - 1) * 0.2
      w.position.set(side * 0.34, -0.1 + (i - 1) * 0.055, 0.58)
      head.add(w)
    }
  }

  // ── TAIL ──────────────────────────────────────────
  const tailCurve = new THREE.CatmullRomCurve3([
    new THREE.Vector3(0, -0.3, -0.68),
    new THREE.Vector3(-0.5, -0.1, -0.9),
    new THREE.Vector3(-0.85, 0.3, -0.7),
    new THREE.Vector3(-0.7, 0.75, -0.3),
    new THREE.Vector3(-0.35, 0.9, -0.1),
  ])
  const tailGeo = new THREE.TubeGeometry(tailCurve, 20, 0.095, 10, false)
  const tail = mesh(tailGeo, bodyMat)
  cat.add(tail)

  // tail tip
  const tailTip = mesh(new THREE.SphereGeometry(0.14, 12, 8), whiteMat)
  tailTip.position.set(-0.35, 0.9, -0.1)
  cat.add(tailTip)

  // ── LEGS ──────────────────────────────────────────
  const legPositions = [
    [-0.38, -0.55, 0.3],
    [ 0.38, -0.55, 0.3],
    [-0.35, -0.55, -0.3],
    [ 0.35, -0.55, -0.3],
  ] as const

  for (const [lx, ly, lz] of legPositions) {
    const leg = mesh(new THREE.CapsuleGeometry(0.14, 0.28, 6, 8), bodyMat)
    leg.position.set(lx, ly, lz)
    cat.add(leg)

    // paw
    const paw = mesh(new THREE.SphereGeometry(0.16, 12, 8), pawMat)
    paw.scale.set(1, 0.55, 1.1)
    paw.position.set(lx, ly - 0.27, lz + 0.05)
    cat.add(paw)
  }

  // ── SHADOW BLOB ───────────────────────────────────
  const shadowBlob = mesh(new THREE.CircleGeometry(0.75, 32), shadowMat)
  shadowBlob.rotation.x = -Math.PI / 2
  shadowBlob.position.set(0, -0.88, 0)
  cat.add(shadowBlob)

  // ── OrbitControls ─────────────────────────────────
  const controls = new OrbitControls(camera, canvas)
  controls.enablePan = false
  controls.enableZoom = false
  controls.autoRotate = true
  controls.autoRotateSpeed = 1.8
  controls.minPolarAngle = Math.PI / 4
  controls.maxPolarAngle = Math.PI / 1.7
  controls.target.set(0, 0.3, 0)
  controls.update()

  // ── Idle bobbing animation state ──────────────────
  let raf: number
  const clock = new THREE.Clock()

  const animate = () => {
    raf = requestAnimationFrame(animate)
    const t = clock.getElapsedTime()

    // gentle float
    cat.position.y = Math.sin(t * 1.4) * 0.06
    // slight tilt
    cat.rotation.x = Math.sin(t * 0.9) * 0.03

    // tail wag
    tail.rotation.y = Math.sin(t * 2.2) * 0.25
    tailTip.rotation.y = tail.rotation.y

    // ear flick
    if (head.children.length) {
      head.rotation.y = Math.sin(t * 0.7) * 0.04
    }

    controls.update()
    renderer.render(scene, camera)
  }
  animate()

  // ── Resize ────────────────────────────────────────
  const ro = new ResizeObserver(() => {
    const nW = container.clientWidth
    const nH = container.clientHeight
    renderer.setSize(nW, nH)
    camera.aspect = nW / nH
    camera.updateProjectionMatrix()
  })
  ro.observe(container)

  onUnmounted(() => {
    cancelAnimationFrame(raf)
    ro.disconnect()
    renderer.dispose()
    controls.dispose()
  })
})
</script>

<template>
  <div class="relative w-full h-full select-none">
    <!-- hint label -->
    <div
      class="pointer-events-none absolute bottom-2 left-1/2 -translate-x-1/2 z-10
             text-xs font-mono tracking-widest opacity-50 whitespace-nowrap"
      style="color: rgba(0,255,255,0.7);"
    >
      drag to rotate · auto-spin
    </div>
    <canvas
      ref="canvasRef"
      class="w-full h-full block cursor-grab active:cursor-grabbing"
    />
  </div>
</template>
