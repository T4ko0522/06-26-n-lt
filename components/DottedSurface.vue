<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'
import * as THREE from 'three'

const props = withDefaults(defineProps<{
  speed?: number
  amplitude?: number
  frequencyX?: number
  frequencyY?: number
  color?: [number, number, number]
}>(), {
  speed: 0.005,
  amplitude: 50,
  frequencyX: 0.3,
  frequencyY: 0.5,
  color: () => [200, 200, 200],
})

const containerRef = ref<HTMLDivElement>()

let animationId: number | null = null
let renderer: THREE.WebGLRenderer | null = null

onMounted(() => {
  if (!containerRef.value) return

  const SEPARATION = 100
  const AMOUNTX = 60
  const AMOUNTY = 90

  const scene = new THREE.Scene()

  const camera = new THREE.PerspectiveCamera(
    60,
    containerRef.value.clientWidth / containerRef.value.clientHeight,
    1,
    10000,
  )
  camera.position.set(0, 355, 1220)

  renderer = new THREE.WebGLRenderer({
    alpha: true,
    antialias: true,
  })
  renderer.setPixelRatio(window.devicePixelRatio)
  renderer.setSize(containerRef.value.clientWidth, containerRef.value.clientHeight)
  renderer.setClearColor(0x000000, 0)

  containerRef.value.appendChild(renderer.domElement)

  const positions: number[] = []
  const colors: number[] = []
  const geometry = new THREE.BufferGeometry()

  for (let ix = 0; ix < AMOUNTX; ix++) {
    for (let iy = 0; iy < AMOUNTY; iy++) {
      const x = ix * SEPARATION - (AMOUNTX * SEPARATION) / 2
      const z = iy * SEPARATION - (AMOUNTY * SEPARATION) / 2
      positions.push(x, 0, z)
      colors.push(props.color[0], props.color[1], props.color[2])
    }
  }

  geometry.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3))
  geometry.setAttribute('color', new THREE.Float32BufferAttribute(colors, 3))

  const material = new THREE.PointsMaterial({
    size: 8,
    vertexColors: true,
    transparent: true,
    opacity: 0.8,
    sizeAttenuation: true,
  })

  const points = new THREE.Points(geometry, material)
  scene.add(points)

  let count = 0

  const animate = () => {
    animationId = requestAnimationFrame(animate)

    const positionAttribute = geometry.attributes.position
    const posArray = positionAttribute.array as Float32Array

    let i = 0
    for (let ix = 0; ix < AMOUNTX; ix++) {
      for (let iy = 0; iy < AMOUNTY; iy++) {
        const index = i * 3
        posArray[index + 1] =
          Math.sin((ix + count) * props.frequencyX) * props.amplitude +
          Math.sin((iy + count) * props.frequencyY) * props.amplitude
        i++
      }
    }

    positionAttribute.needsUpdate = true
    renderer!.render(scene, camera)
    count += props.speed
  }

  const handleResize = () => {
    if (!containerRef.value || !renderer) return
    camera.aspect = containerRef.value.clientWidth / containerRef.value.clientHeight
    camera.updateProjectionMatrix()
    renderer.setSize(containerRef.value.clientWidth, containerRef.value.clientHeight)
  }

  window.addEventListener('resize', handleResize)
  animate()

  onUnmounted(() => {
    window.removeEventListener('resize', handleResize)
    if (animationId !== null) cancelAnimationFrame(animationId)
    scene.traverse((object) => {
      if (object instanceof THREE.Points) {
        object.geometry.dispose()
        if (Array.isArray(object.material))
          object.material.forEach(m => m.dispose())
        else
          object.material.dispose()
      }
    })
    renderer?.dispose()
    if (containerRef.value && renderer?.domElement && containerRef.value.contains(renderer.domElement))
      containerRef.value.removeChild(renderer.domElement)
  })
})
</script>

<template>
  <div ref="containerRef" class="dotted-surface" />
</template>

<style>
.dotted-surface {
  position: fixed;
  inset: 0;
  z-index: 0;
  pointer-events: none;
  filter: blur(2px);
}
</style>
