<script setup lang="ts">
import { e } from 'unocss'

const el = $ref<HTMLCanvasElement>()
const ctx = $computed(() => el!.getContext('2d')!)
const WIDTH = 600,
  HEIGHT = 600
const angel = $ref(0.4)
const length = $ref(20)
const depth = $ref(8)
const peddingTask = [] as Array<Function>
interface Point {
  x: number
  y: number
}
interface Branch {
  start: Point
  angle: number
  length: number
}
function getEndPoint({ start, angle, length }: Branch): Point {
  const x = start.x + Math.cos(angle) * length
  const y = start.y + Math.sin(angle) * length
  return { x, y }
}
function lineTo(a: Point, b: Point) {
  ctx.beginPath()
  ctx.moveTo(a.x, a.y)
  ctx.lineTo(b.x, b.y)
  ctx.stroke()
}
function drawBranch(b: Branch) {
  lineTo(b.start, getEndPoint(b))
}
function step(b: Branch, dep: number = 0) {
  const end = getEndPoint(b)
  drawBranch(b)

  if (Math.random() < 0.5 || dep < depth) {
    peddingTask.push(() =>
      step(
        {
          start: end,
          angle: b.angle - angel * Math.random(),
          length: b.length + (Math.random() * 10 - 5),
        },
        dep + 1
      )
    )
  }
  if (Math.random() < 0.5 || depth < depth) {
    peddingTask.push(() =>
      step(
        {
          start: end,
          angle: b.angle + angel * Math.random(),
          length: b.length + (Math.random() * 10 - 5),
        },
        dep + 1
      )
    )
  }
}
function init() {
  // reset canvas w h
  el!.width = window.innerWidth
  el!.height = window.innerHeight
  //randow color
  ctx.strokeStyle = `rgba(255,0,0,0.4)`
  const b = {
    start: { x: 0, y: el!.height / 2 },
    angle: 0,
    length: length,
  }
  const r = {
    start: { x: el!.width, y: el!.height / 2 },
    angle: Math.PI,
    length: length,
  }
  step(b)
  step(r)
}
function frame() {
  const tasks = [...peddingTask]
  peddingTask.length = 0
  tasks.forEach((task) => task())
}
let frameCount = 0
function startFrame() {
  requestAnimationFrame(() => {
    frameCount++
    if (frameCount % 3 === 0) {
      frame()
    }
    startFrame()
  })
}
startFrame()
onMounted(() => {
  init()
})
</script>

<template>
  <main font-sans text="center gray-700 dark:gray-200">
    <canvas ref="el" class="w-screen h-screen"></canvas>
  </main>
</template>
