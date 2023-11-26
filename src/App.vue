<script setup>
import { ref, onMounted } from 'vue'

const transcript = ref('')
const isRecording = ref(false)
let sr = null

// Check for Speech Recognition support
if ('SpeechRecognition' in window || 'webkitSpeechRecognition' in window) {
  const Recognition = window.SpeechRecognition || window.webkitSpeechRecognition
  sr = new Recognition()
}

onMounted(() => {
  if (sr) {
    sr.continuous = true
    sr.interimResults = true

    sr.onstart = () => {
      console.log('SR Started')
      isRecording.value = true
    }

    sr.onend = () => {
      console.log('SR Stopped')
      isRecording.value = false
    }

    sr.onresult = (evt) => {
  let finalTranscript = '';
  for (let i = 0; i < evt.results.length; i++) {
    const result = evt.results[i];
    const alternative = result[0];

    if (result.isFinal) {
      finalTranscript += alternative.transcript + ' ';
    }
  }

  transcript.value = finalTranscript.trim();
};
  }
})

const CheckForCommand = (result) => {
  const t = result[0].transcript;
  if (t.includes('stop recording')) {
    sr.stop()
  } else if (
    t.includes('what is the time') ||
    t.includes('what\'s the time')
  ) {
    sr.stop()
    alert(new Date().toLocaleTimeString())
    setTimeout(() => sr.start(), 100)
  }
}

const ToggleMic = () => {
  if (sr && isRecording.value) {
    sr.stop()
  } else if (sr) {
    sr.start()
  }
}
</script>

<template>
	<div class="app">
		<button :class="`mic`" @click="ToggleMic">Start</button>
		<div class="transcript" v-text="transcript"></div>
	</div>
</template>

<style>
* {
	margin: 0;
	padding: 0;
	box-sizing: border-box;
	font-family: 'Fira sans', sans-serif;
}

body {
	background: #281936;
	color: #FFF;
}
</style>