<!-- src/App.vue -->
<template>
  <div class="min-h-screen flex flex-col bg-gray-100">
    <!-- Header -->
    <header class="bg-indigo-600 text-white px-6 py-4 shadow-md">
      <h1 class="text-xl font-semibold">1234 CSS → TailwindCSS Converter</h1>
    </header>

    <!-- Editor area -->
    <main class="flex-1 grid lg:grid-cols-2 gap-6 p-6">
      <!-- Left pane – raw CSS -->
      <section class="flex flex-col">
        <label class="mb-2 font-medium">Paste CSS</label>
        <textarea
            v-model="cssInput"
            class="flex-1 rounded-lg border p-4 font-mono resize-none focus:ring-2 focus:ring-indigo-400"
            placeholder="e.g. .btn { background-color: #1d4ed8; padding: 0.5rem 1rem; }"
        />
        <button
            class="mt-4 self-start px-4 py-2 rounded bg-indigo-600 text-red hover:bg-indigo-700 disabled:opacity-50"
            :disabled="!cssInput.trim()"
            @click="convert"
        >
          Convert
        </button>
      </section>

      <!-- Right pane – Tailwind classes -->
      <section class="flex flex-col">
        <label class="mb-2 font-medium">Generated Tailwind classes</label>
        <textarea
            readonly
            :value="tailwindOutput"
            class="flex-1 rounded-lg border p-4 font-mono bg-gray-50 resize-none"
        />
        <button
            v-if="tailwindOutput"
            class="mt-4 self-start px-4 py-2 rounded bg-emerald-600 text-white hover:bg-emerald-700"
            @click="copyToClipboard"
        >
          Copy
        </button>
      </section>
    </main>

    <!-- Footer -->
    <footer class="text-center text-sm text-gray-500 py-4">
      Built with&nbsp;Vue 3 + TailwindCSS
    </footer>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { TailwindConverter } from 'css-to-tailwindcss';

/** --- reactive state --- */
const cssInput = ref('')
const tailwindOutput = ref('')

const converter = new TailwindConverter({
  remInPx: 16, // set null if you don't want to convert rem to pixels
  postCSSPlugins: [], // add any postcss plugins to this array
  tailwindConfig: {
    // your tailwind config here
    content: [],
    theme: {
      extend: {
        colors: {
          'custom-color': {
            100: '#123456',
            200: 'hsla(210, 100%, 51.0%, 0.016)',
            300: '#654321',
            gold: 'hsl(41, 28.3%, 79.8%)',
            marine: 'rgb(4, 55, 242, 0.75)',
          },
        },
        screens: {
          'custom-screen': { min: '768px', max: '1024px' },
        },
      },
      supports: {
        grid: 'display: grid',
        flex: 'display: flex',
      },
    },
  },
});

const inputCSS = `
.bar {
  padding-top: 72px;
}`;


function convert() {
  alert("fdsf");
  converter.convertCSS(inputCSS).then(({ convertedRoot, nodes }) => {
    console.log(convertedRoot.toString());
    console.log(nodes);
  });
}

async function copyToClipboard() {
  await navigator.clipboard.writeText(tailwindOutput.value)
  alert('Tailwind classes copied!')
}
</script>

<!-- Hook Tailwind only once in index.html (default Vite template already has it) -->
<style>
/* Optional: custom global styles */
</style>
