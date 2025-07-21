<!-- src/App.vue -->
<template>
  <div class="min-h-screen flex flex-col bg-gradient-to-b from-slate-50 to-slate-100">
    <!-- ───── Header ───── -->
    <header class="sticky top-0 z-10 bg-gradient-to-r from-indigo-600 via-violet-600 to-purple-600 shadow-lg">
      <div class="mx-auto flex max-w-7xl items-center justify-between px-6 py-4">
        <h1 class="text-lg font-semibold tracking-wide text-white">
          CSS&nbsp;→&nbsp;TailwindCSS&nbsp;Converter
        </h1>
        <span class="text-xs font-medium text-white/80">
          Built with&nbsp;Vue&nbsp;3&nbsp;+&nbsp;TailwindCSS
        </span>
      </div>
    </header>

    <!-- ───── Workspace ───── -->
    <main class="mx-auto grid w-full max-w-7xl flex-1 gap-6 p-6 lg:grid-cols-2">
      <!-- Card – Input -->
      <article class="flex flex-col rounded-2xl bg-white/60 shadow backdrop-blur-md ring-1 ring-black/5">
        <header class="flex items-center justify-between border-b px-6 py-4">
          <h2 class="font-medium">Paste CSS (auto‑convert)</h2>
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
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                    d="M14 5l7 7m0 0l-7 7m7-7H3" />
            </svg>
            <svg
                v-else
                xmlns="http://www.w3.org/2000/svg"
                class="h-4 w-4 animate-spin"
                fill="none"
                viewBox="0 0 24 24"
                stroke="currentColor"
            >
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                    d="M12 4v4m0 8v4m8-8h4M4 12H0m16.24 5.66l2.83 2.83M4.93 4.93l2.83 2.83m0 8.48l-2.83 2.83m14.14-14.14l-2.83 2.83" />
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
      <article class="flex flex-col rounded-2xl bg-white/60 shadow backdrop-blur-md ring-1 ring-black/5">
        <header class="flex items-center justify-between border-b px-6 py-4">
          <h2 class="font-medium">Generated Tailwind classes</h2>
        </header>

        <!-- NEW: list one row per selector -->
        <ul class="flex-1 space-y-6 overflow-y-auto p-6">
          <li
              v-for="item in tailwindList"
              :key="item.selector"
              class="space-y-1"
          >
            <p class="text-sm font-medium text-gray-700">{{ item.selector }}</p>

            <div class="flex items-center gap-2">
              <code
                  class="inline-block rounded bg-gray-100 px-2 py-1 text-xs font-mono text-gray-800"
              >
                {{ item.classes }}
              </code>

              <button
                  class="rounded p-1 hover:bg-gray-200"
                  @click="copy(item)"
                  title="Copy"
              >
                <svg
                    xmlns="http://www.w3.org/2000/svg"
                    class="h-4 w-4"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke="currentColor"
                >
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                        d="M8 16h8m-4-4h4m-9 8h9a2 2 0 002-2V7m-4 0H7a2 2 0 00-2 2v11a2 2 0 002 2h9a2 2 0 002-2V7a2 2 0 00-2-2h-3l-2-2H9l-2 2H4" />
                </svg>
              </button>

              <span
                  v-if="copiedSelector === item.selector"
                  class="animate-fade text-xs font-medium text-emerald-600"
              >
                Copied!
              </span>
            </div>
          </li>
        </ul>
      </article>
    </main>

    <!-- Footer -->
    <footer class="pb-6">
      <p class="text-center text-xs text-gray-500/80">
        © 2025 — Made with ❤️ by Gary Lyu
      </p>
    </footer>
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'
import { TailwindConverter } from 'css-to-tailwindcss'

/* ─── reactive state ─── */
const cssInput       = ref('')
const tailwindList   = ref<{ selector: string; classes: string }[]>([])
const converting     = ref(false)
const copiedSelector = ref<string | null>(null)

/* ─── Tailwind converter instance ─── */
const converter = new TailwindConverter({
  remInPx: 16,
  postCSSPlugins: [],
  tailwindConfig: {
    content: [],
    theme: {
      extend: {
        colors: {
          'custom-color': {
            100  : '#123456',
            200  : 'hsla(210,100%,51%,0.016)',
            300  : '#654321',
            gold : 'hsl(41,28.3%,79.8%)',
            marine: 'rgb(4,55,242,0.75)',
          },
        },
        screens: { 'custom-screen': { min: '768px', max: '1024px' } },
      },
      supports: { grid: 'display: grid', flex: 'display: flex' },
    },
  },
})

/* ─── auto‑convert (400 ms debounce) ─── */
let debounceId: ReturnType<typeof setTimeout>
watch(cssInput, () => {
  clearTimeout(debounceId)
  debounceId = setTimeout(() => {
    cssInput.value.trim() ? convert() : (tailwindList.value = [])
  }, 400)
})

/* ─── explicit convert ─── */
async function convert() {
  if (!cssInput.value.trim()) return
  converting.value = true
  const { convertedRoot } = await converter.convertCSS(cssInput.value)
  tailwindList.value = extractAll(convertedRoot.toString())
  converting.value  = false
}

/* ─── helper: pull out every “selector { @apply …; }” ─── */
function extractAll(css: string) {
  const entries: { selector: string; classes: string }[] = []
  const re = /([^{\s]+)\s*\{[^@]*@apply\s+([^;]+);/g
  let m: RegExpExecArray | null
  while ((m = re.exec(css))) {
    const selector = m[1].trim()
    let classes    = m[2].trim()
    /* px → rem conversion */
    classes = classes.replace(/(\d*\.?\d+)px/g, (_, num) => {
      const rem = parseFloat(num) / 16
      return `${rem.toFixed(4).replace(/\.?0+$/, '')}rem`
    })
    entries.push({ selector, classes })
  }
  return entries
}

/* ─── copy one line & show badge ─── */
async function copy(item: { selector: string; classes: string }) {
  await navigator.clipboard.writeText(item.classes)
  copiedSelector.value = item.selector
  setTimeout(() => (copiedSelector.value = null), 1500)
}
</script>

<style>
/* tiny badge fade‑in/out */
@keyframes fade {
  0%, 100% { opacity: 0; transform: translateY(-2px); }
  10%, 90% { opacity: 1; transform: translateY(0); }
}
.animate-fade { animation: fade 1.5s ease-in-out forwards; }
</style>
