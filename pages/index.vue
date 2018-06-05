<template>
	<section>
    <div v-if="!socket">
      <input v-model="port">
      <button @click="start">포트연결</button>
    </div>
		<div v-else>
      녹음중...
    </div>
	</section>
</template>

<script>
export default {
	data() {
		return {
      port: null,
      socket: null,
			recognition: null,
      interm: '',
			transcript: []
		}
  },
  methods: {
    start() {
      if (!this.port) return
      this.socket = require('socket.io-client')('http://localhost:' + this.port)
      this.socket.on('connect', () => {
        console.log('Socket connected')
      })
      this.socket.on('disconnect', () => {
        this.socket.close()
        this.socket = null
      })
    }
  },
  mounted() {
    if (this.socket) this.socket.close()
    if (
      !('webkitSpeechRecognition' in window) &&
      !('SpeechRecognition' in window)
    ) {
      console.log('이 브라우저는 speech to text를 지원하지 않음')
    }
    const SpeechRecognition = window.webkitSpeechRecognition || window.SpeechRecognition
    this.recognition = new SpeechRecognition()
    this.recognition.continuous = true
    this.recognition.interimResults = false
    this.recognition.lang = 'ko-KR'
    this.recognition.start()
    const self = this
    this.recognition.onresult = function(event) {
      let interm = ''
      for (var i = event.resultIndex; i < event.results.length; ++i) {
        if (event.results[i].isFinal) {
          if (self.socket) self.socket.emit('record', event.results[i][0].transcript)
          self.transcript.push(event.results[i][0].transcript)
        } else {
          interm += event.results[i][0].transcript
          recognition.stop()
        }
      }
      self.interm = interm
    }
      this.recognition.onend = () => {
      this.recognition.start()
    }
  }
}
</script>