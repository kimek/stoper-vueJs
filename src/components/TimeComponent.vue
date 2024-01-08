<template>
    <div class="display">
        <span>{{ display }}</span><span>.{{ displayMs }}</span>
    </div>

    <button type='button' @click='start' v-if='!isStarted'>
      Start
    </button>
    <button type='button' @click='pause' v-else>
      Stop
    </button>

    <button type='button' @click='stop' v-if='!isStopped'>
      Resetuj
    </button>
</template>

<script setup>
import { ref } from 'vue' //pomocnik do DOMa

const emit = defineEmits(['send-message'])
// const message = ref('A message from the child component');

let display = ref('00:00:00'),
    displayMs = ref('000'),
    defaultSeconds = 0,
    isStarted = false,
    isStopped = true,
    targetTimestamp = 0,
    pausedDifference = 0,
    frameInterval = () => {}

function start() {
  let nowTimestamp = Date.now()
  let targetTime = defaultSeconds
  if (pausedDifference) {
    targetTime = pausedDifference
    pausedDifference = 0
  }
  targetTimestamp = nowTimestamp - targetTime
  frameInterval = () => {
    updateDisplay()
    requestAnimationFrame(frameInterval)
  }
  frameInterval()
  isStarted = true
  isStopped = false
}

function pause() {
  let nowTimestamp = Date.now()
  pausedDifference = Math.abs(targetTimestamp - nowTimestamp)
  frameInterval = () => {}
  updateDisplay()
  isStarted = false
}

function stop() {
  if (isStarted) pause()
  reset()
  updateDisplay()
  isStopped = true

  emit('send-message', 'removeRounds')
}

function reset() {
  targetTimestamp = Date.now() + defaultSeconds
  pausedDifference = 0
}

// Wyswietlanie czasu
function updateDisplay(targetTime) {
  if (isStopped) return
  let nowTimestamp = Date.now()
  if (targetTime) {
    targetTimestamp = nowTimestamp + targetTime
  }
  const time = Math.abs(nowTimestamp - targetTimestamp) / 1000
  const numMs = Math.floor((time % 1) * 1000)
  const numS = Math.floor(time % 60)
  const numM = Math.floor(time/60) % 60
  const numH = Math.floor(time/3600) % 24
  const numD = Math.floor(time/86400)
  let displayTemp = numS.toString().padStart(2, '0')

  displayTemp = numM.toString().padStart(2, '0') + ':' + displayTemp
  displayTemp = numH.toString().padStart(2, '0') + ':' + displayTemp
  if (numD) displayTemp = numD.toString().padStart(2, '0') + ':' + displayTemp
  display.value = displayTemp
  displayMs.value = ("000" + numMs).slice(-3)


  const x = Math.abs(targetTimestamp) / 1000
  const xs = Math.floor(x / 60)% 60

  emit('send-message', 'time', displayTemp, xs )
}
</script>

<style>
.display {
  font-family: 'digital';
  font-size: 3em;
}
</style>