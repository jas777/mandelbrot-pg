<template>
  <header class="m-8">
    <h1>Zbiór Mandelbrota</h1>
    <h3 class="text-red-500">PROJEKT WYKONANY DO ZADANIA "PIKTOGRAM 1" DNIA 21.10.2023</h3>
    <p class="text-2xl font-light my-3 w-2/3 max-w-2/3">
      Zbiór Mandelbrota (zwany także Żukiem Mandelbrota) -
      <span class="text-violet-400">podzbiór</span> płaszczyzny zespolonej, którego brzeg jest
      jednym z <i>najbardziej</i> znanych fraktali.
    </p>
    <details class="text-2xl font-light my-3">
      <summary class="mb-2">Wytłumaczenie generowania</summary>
      Aby uzyskać taki obraz, dla każdego punktu <i>p</i> oblicza się pewną liczbę początkowych
      wyrazów ciągu <i>z<sub>n</sub></i
      >. Decyduje się, że punkt należy do zbioru, jeżeli dla wszystkich wyrazów tego podciągu
      spełniony jest warunek <i>|z<sub>n</sub>|</i> &lt; 2. Jest to tym samym obraz przybliżony.
      Zbiór Mandelbrota zawiera się w (jest podzbiorem) każdym przybliżeniu.
    </details>
  </header>
  <main class="m-8">
    <h2>Wygeneruj Mandelbrota</h2>
    <div class="my-4 w-fit">
      <form @submit.prevent="() => draw()">
        <label for="iterations">Ilość iteracji: </label>
        <input
          type="number"
          name="iterations"
          class="rounded-lg bg-gray-800 border-2 border-violet-600/30 select-none px-2 py-1 text-violet-200 focus:outline-none"
          v-model="iterations"
        />
        <button
          type="submit"
          class="bg-violet-600 px-2 py-1 border-2 border-violet-600 rounded-lg text-violet-200 ml-2 hover:bg-violet-700"
        >
          Wygeneruj!
        </button>
        <span class="text-red-500 ml-2">
          Uwaga! Generowanie może zająć od kilku do nawet kilkudziesięciu sekund - w zależności od
          ilości iteracji oraz specyfikacji komputera!
        </span>
      </form>
      <p class="mt-4" v-if="working">Generuję...</p>
    </div>
    <div class="h-full">
      <canvas class="w-full h-2/3 max-h-2/3" id="result" v-if="!working"></canvas>
    </div>
  </main>
  <footer class="my-2 text-center">
    <p class="text-gray-600">Made with ❤ by Jan Czeszejko-Sochacki (EiT rok 1, gr. 1, 198038)</p>
  </footer>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue'
import Panzoom from '@panzoom/panzoom'

interface Point {
  x: number
  y: number
}

const canvas = ref<HTMLCanvasElement>()
const ctx = ref<CanvasRenderingContext2D>()

onMounted(() => {
  canvas.value = document.getElementById('result') as HTMLCanvasElement
  ctx.value = canvas.value?.getContext('2d') as CanvasRenderingContext2D
  ctx.value.canvas.width = window.innerWidth
  ctx.value.canvas.height = window.innerHeight

  const panzoom = Panzoom(canvas.value, {
    maxScale: 10,
    canvas: true
  })

  canvas.value.addEventListener('wheel', panzoom.zoomWithWheel)
})

const iterations = ref<number>(50)
const working = ref<boolean>(false)
const WIDTH = window.innerWidth
const HEIGHT = window.innerHeight

const REAL_SET = { start: -2, end: 1 }
const IMAGINARY_SET = { start: -1, end: 1 }

const colors = [
  '#100238',
  '#1a0c44',
  '#261351',
  '#33195e',
  '#41206b',
  '#4e2778',
  '#5d2e85',
  '#6b3592',
  '#7b3c9f',
  '#8a43ad',
  '#9a4aba',
  '#ab51c7',
  '#bc58d4',
  '#ce60e1',
  '#e067ed',
  '#f26efa'
]

const mandelbrot = (c: Point): [number, boolean] => {
  let z = { x: 0, y: 0 }
  let n = 0
  let p, d
  do {
    p = {
      x: Math.pow(z.x, 2) - Math.pow(z.y, 2),
      y: 2 * z.x * z.y
    }
    z = {
      x: p.x + c.x,
      y: p.y + c.y
    }
    // |z|
    d = Math.sqrt(Math.pow(z.x, 2) + Math.pow(z.y, 2))
    n += 1
  } while (d <= 2 && n < iterations.value)
  return [n, d <= 2]
}

const draw = async () => {
  working.value = true
  if (!ctx.value) return
  for (let i = 0; i < WIDTH; i++) {
    for (let j = 0; j < HEIGHT; j++) {
      const complex: Point = {
        x: REAL_SET.start + (i / WIDTH) * (REAL_SET.end - REAL_SET.start),
        y: IMAGINARY_SET.start + (j / HEIGHT) * (IMAGINARY_SET.end - IMAGINARY_SET.start)
      }

      const [m, isMandelbrotSet] = mandelbrot(complex)
      ctx.value.fillStyle = colors[isMandelbrotSet ? 0 : (m % colors.length) - 1 + 1]
      ctx.value.fillRect(i, j, 1, 1)
    }
  }
  working.value = false
}
</script>

<style scoped>
p,
label,
details,
summary {
  @apply text-violet-200 font-light;
}

summary {
  @apply font-normal text-violet-500 cursor-pointer hover:text-violet-400 transition-colors duration-200;
}

label {
  @apply text-xl;
}

h1 {
  @apply text-5xl font-light text-violet-500;
}

h2 {
  @apply text-4xl font-light text-violet-500;
}
</style>
