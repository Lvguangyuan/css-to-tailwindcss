<!-- src/App.vue -->
<template>
  <div class="min-h-screen flex flex-col bg-gradient-to-b from-slate-50 to-slate-100">
    <!-- Header -->
    <header
        class="sticky top-0 z-10 bg-gradient-to-r from-indigo-600 via-violet-600 to-purple-600 shadow-lg"
    >
      <div class="mx-auto flex max-w-7xl items-center justify-between px-6 py-4">
        <h1 class="text-lg font-semibold tracking-wide text-white">
          1234&nbsp;CSS&nbsp;→&nbsp;TailwindCSS&nbsp;Converter
        </h1>
        <span class="text-xs font-medium text-white/80">
          Built with&nbsp;Vue&nbsp;3&nbsp;+&nbsp;TailwindCSS
        </span>
      </div>
    </header>

    <!-- Workspace -->
    <main
        class="mx-auto grid w-full max-w-7xl flex-1 gap-6 p-6 lg:grid-cols-2"
    >
      <!-- Card – Input -->
      <article
          class="flex flex-col rounded-2xl bg-white/60 shadow backdrop-blur-md ring-1 ring-black/5"
      >
        <header class="flex items-center justify-between border-b px-6 py-4">
          <h2 class="font-medium">Paste CSS</h2>
          <button
              :disabled="!cssInput.trim()"
              @click="convert"
              class="flex items-center gap-1 rounded-lg bg-indigo-600 px-3 py-1.5 text-sm font-medium text-white hover:bg-indigo-700 disabled:opacity-50"
          >
            <svg
                v-if="!converting"
                xmlns="http://www.w3.org/2000/svg"
                class="h-4 w-4"
                fill="none"
                viewBox="0 0 24 24"
                stroke="currentColor"
            >
              <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M14 5l7 7m0 0l-7 7m7-7H3"
              />
            </svg>
            <svg
                v-else
                xmlns="http://www.w3.org/2000/svg"
                class="h-4 w-4 animate-spin"
                fill="none"
                viewBox="0 0 24 24"
                stroke="currentColor"
            >
              <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M12 4v4m0 8v4m8-8h4M4 12H0m16.24 5.66l2.83 2.83M4.93 4.93l2.83 2.83m0 8.48l-2.83 2.83m14.14-14.14l-2.83 2.83"
              />
            </svg>
            Convert
          </button>
        </header>

        <textarea
            v-model="cssInput"
            placeholder="e.g. .btn { background-color: #1d4ed8; padding: 0.5rem 1rem; }"
            class="min-h-[16rem] flex-1 resize-none rounded-b-2xl bg-transparent p-6 font-mono focus:outline-none focus:ring-0"
        />
      </article>

      <!-- Card – Output -->
      <article
          class="flex flex-col rounded-2xl bg-white/60 shadow backdrop-blur-md ring-1 ring-black/5"
      >
        <header class="flex items-center justify-between border-b px-6 py-4">
          <h2 class="font-medium">Generated Tailwind classes</h2>
          <button
              :disabled="!tailwindOutput.trim()"
              @click="copyToClipboard"
              class="flex items-center gap-1 rounded-lg bg-emerald-600 px-3 py-1.5 text-sm font-medium text-white hover:bg-emerald-700 disabled:opacity-50"
          >
            <svg
                xmlns="http://www.w3.org/2000/svg"
                class="h-4 w-4"
                fill="none"
                viewBox="0 0 24 24"
                stroke="currentColor"
            >
              <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M8 16h8m-4-4h4m-9 8h9a2 2 0 002-2V7m-4 0H7a2 2 0 00-2 2v11a2 2 0 002 2h9a2 2 0 002-2V7a2 2 0 00-2-2h-3l-2-2H9l-2 2H4"
              />
            </svg>
            Copy
          </button>
        </header>

        <textarea
            readonly
            :value="tailwindOutput"
            class="min-h-[16rem] flex-1 resize-none rounded-b-2xl bg-gray-50 p-6 font-mono text-gray-800 focus:outline-none"
        />
      </article>
    </main>

    <!-- Footer -->
    <footer class="pb-6">
      <p class="text-center text-xs text-gray-500/80">
        © 2025 — Made&nbsp;with ❤️ by Your Name
      </p>
    </footer>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { TailwindConverter } from 'css-to-tailwindcss'

/* --- reactive state --- */
const cssInput = ref('')
const tailwindOutput = ref('')
const converting = ref(false)

const converter = new TailwindConverter({
  remInPx: 16,
  postCSSPlugins: [],
  tailwindConfig: {
    content: [],
    theme: {
      extend: {
        colors: {
          'custom-color': {
            100: '#123456',
            200: 'hsla(210,100%,51%,0.016)',
            300: '#654321',
            gold: 'hsl(41,28.3%,79.8%)',
            marine: 'rgb(4,55,242,0.75)',
          },
        },
        screens: { 'custom-screen': { min: '768px', max: '1024px' } },
      },
      supports: {
        grid: 'display: grid',
        flex: 'display: flex',
      },
    },
  },
})

async function convert() {
  if (!cssInput.value.trim()) return
  converting.value = true
  const { convertedRoot } = await converter.convertCSS(cssInput.value)
  tailwindOutput.value = extractApply(convertedRoot.toString())
  converting.value = false
}

/**
 * Extracts the part after `@apply` and converts px → rem.
 */
function extractApply(css: string) {
  const raw = (css.match(/@apply\s+([^;]+);/) || [, ''])[1].trim()
  return raw.replace(/(\d*\.?\d+)px/g, (_, num) => {
    const rem = parseFloat(num) / 16
    return `${rem.toFixed(4).replace(/\.?0+$/, '')}rem`
  })
}

async function copyToClipboard() {
  if (!tailwindOutput.value) return
  await navigator.clipboard.writeText(tailwindOutput.value)
}
</script>

<style>
/* No additional global styles needed – all handled by Tailwind */
</style>
