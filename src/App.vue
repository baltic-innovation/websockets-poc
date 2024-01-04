<script setup lang="ts">
import { ref } from 'vue'

const userInput = ref()

const resultRaw = ref()
const resultParsed = ref()

let ws = ref<WebSocket>()

const start = () => {
  if (ws.value) {
    ws.value.close()
  }
  ws.value = new WebSocket(import.meta.env.VITE_WS_ENDPOINT as string)
  ws.value.onopen = () => {
    ws.value?.send(userInput.value)
  }
  ws.value.onclose = (e) => {
    // Everything above 1001 indicates an error
    if (e.code > 1001) {
      console.error(e)
    } else {
      console.log(e)
    }
  }
  ws.value.onmessage = (e) => {
    if (e.data === 'ping') {
      return
    }

    resultRaw.value = e.data
    try {
      resultParsed.value = JSON.parse(resultRaw.value)
    } catch {
      /* This can remain empty, we catch if you want to check value that can't be parsed */
    }
  }
}

const stop = () => {
  resultParsed.value = undefined
  resultRaw.value = undefined
  // Close with 1000 code (normal closure)
  ws.value?.close(1000)
}
</script>

<template>
  <div>
    <div v-if="!ws || ws.readyState === ws.CLOSED || ws.readyState === ws.CLOSING">
      <input v-model="userInput" placeholder="Message content" />

      <button @click="start">Send message</button>
    </div>
    <button v-if="ws && ws.readyState === ws.OPEN" @click="stop">Close connection</button>
    <div>
      <div v-if="resultRaw">
        <h2>WebSocket result:</h2>
        <div v-if="resultParsed">
          <p>Result as JSON: {{ resultParsed }}</p>
        </div>
        <p v-else>Raw value: {{ resultRaw }}</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
