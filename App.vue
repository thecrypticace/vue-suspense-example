<template>
  <div>
    <link href="https://unpkg.com/tailwindcss@^1.0/dist/tailwind.min.css" rel="stylesheet">
    <div class="p-4 flex items-baseline">
      <h1 class="text-3xl">Async / Suspense example with {{ componentCount }} component(s)</h1>
      <button @click="again" class="bg-green-200 text-green-800 px-3 py-1 rounded font-bold ml-auto">
        Reload
      </button>
    </div>

    <hr>

    <div class="p-4">
    <h1 class="text-2xl">Loading suspense tree</h1>
    <Suspense :key="componentCount">
      <template #default>
        <div>
          All {{ componentCount }} components are now ready and this will take only
          as long as it takes the longest component to load.
        </div>

        <template v-for="n in componentCount">
          <AsyncExample :key="n" :name="`Async${n}`"></AsyncExample>
        </template>
      </template>
      <template #fallback>
        Loading {{ componentCount }} components…
        <br>
        Time elapsed: {{ elapsed }}ms
      </template>
    </Suspense>
    </div>

    <div class="bg-gray-200 text-gray-900 px-4 py-2 border-t border-b">
      <h2 class="text-xl font-medium">Console Logs <small class="font-normal">(check your console too!)</small></h2>
      <hr class="my-2">
      <template v-for="(log, idx) in logs">
        <pre :key="idx" class="py-1">{{ log }}</pre>
      </template>
    </div>
  </div>
</template>

<script>
import { ref, h, computed } from "vue";
export default {
  setup() {
    const start = ref(+new Date)
    const now = ref(start.value)
    const elapsed = computed(() => now.value - start.value)
    const logs = ref([])

    const origConsoleLog = console.log
    console.log = (...args) => {
      logs.value.push(args[0])
      origConsoleLog(...args)
    }

    setInterval(() => now.value = +new Date, 1)

    const componentCount = ref()
    const again = () => {
      start.value = +new Date
      componentCount.value = Math.max(2, Math.floor(Math.random() * 10))
      logs.value = []

      console.log(`Loading ${componentCount.value} components`)
    }

    again()

    return {
      logs,
      elapsed,
      again,
      componentCount,
    };
  },

  components: {
    AsyncExample: {
      props: { name: { type: String }},
      async setup (props) {
        console.log(`${props.name} started loading`)

        const start = +new Date
        await new Promise(resolve => setTimeout(resolve, Math.random() * 1000 * 10))
        const end = +new Date

        console.log(`${props.name} ready after ${end-start}ms (but not rendered until everything is ready)`)

        return () => h("div", `${props.name} ready after ${end-start}ms`)
      }
    },
  }
};
</script>
