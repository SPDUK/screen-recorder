<template>
  <div>
    <div class="draggable">
      <h4>Drag Window</h4>
    </div>
    <!-- <button @click="startRecording">Start Recording</button> -->
    <!-- <button @click="stopRecording">Stop Recording</button> -->
    Current recording state: {{recording}}
 

    <video></video>

    <button id="startBtn" @click="startRecording" class="button is-primary">Start</button>
    <button id="stopBtn" @click="stopRecording" class="button is-warning">Stop</button>

    <hr />


  </div>
</template>
<script>
const { writeFile } = require('fs')
const { desktopCapturer, remote } = require('electron')
const { dialog } = remote

export default {
  name: 'landing-page',
  data () {
    return {
      recording: false,
      mediaRecorder: {},
      recordedChunks: []
    }
  },

  mounted () {
    this.selectSource()
  },

  methods: {

    startRecording () {
      const startBtn = document.getElementById('startBtn')

      this.mediaRecorder.start()
      startBtn.innerText = 'Recording'
    },

    stopRecording () {
      const startBtn = document.getElementById('startBtn')

      startBtn.innerText = 'Not Recording'

      this.mediaRecorder.stop()
    },

    // Saves the video file on stop
    async  handleStop (e) {
      const blob = new Blob(this.recordedChunks, {
        type: 'video/webm; codecs=vp9'
      })

      const buffer = Buffer.from(await blob.arrayBuffer())

      const { filePath } = await dialog.showSaveDialog({

        buttonLabel: 'Save video',
        defaultPath: `vid-${Date.now()}.webm`
      })

      console.log(filePath)

      writeFile(filePath, buffer, () => console.log('video saved successfully!'))
    },

    handleDataAvailable (e) {
      console.log('video data available')
      this.recordedChunks.push(e.data)
    },
    // Change the videoSource window to record
    // Get the available video sources

    async selectSource () {
      const sources = await desktopCapturer.getSources({ types: ['screen'] })

      for (const source of sources) {
        try {
          const stream = await navigator.mediaDevices.getUserMedia({
            audio: false,
            video: {
              mandatory: {
                chromeMediaSource: 'desktop',
                chromeMediaSourceId: source.id,
                minWidth: 400,
                maxWidth: 400,
                minHeight: 400,
                maxHeight: 400
              }
            }
          })

          console.log(stream)
          this.handleStream(stream)

          const videoElement = document.querySelector('video')
          // Preview the source in a video element
          videoElement.srcObject = stream
          videoElement.play()

          // Create the Media Recorder
          const options = { mimeType: 'video/webm; codecs=vp9' }
          this.mediaRecorder = new MediaRecorder(stream, options)

          console.log('____')
          console.log(this.mediaRecorder)

          console.log('____')

          // Register Event Handlers
          this.mediaRecorder.ondataavailable = this.handleDataAvailable
          this.mediaRecorder.onstop = this.handleStop
        } catch (e) {
          console.log(e)
        }
      }
    },

    handleStream (stream) {
      this.mediaRecorder.ondataavailable = event => {
        this.blobs.push(event.data)
      }
    }

  }
}
</script>


<style lang="scss">
@import url("https://fonts.googleapis.com/css?family=Source+Sans+Pro");

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: "Source Sans Pro", sans-serif;
  background: rgba(0, 0, 0, 0);
  border: 2px solid rgb(77, 139, 255);
  height: 100vh;
  width: 100vw;
}

.draggable {
  -webkit-app-region: drag;
  height: 30px;
  color: #fff;
  background: rgba(0, 0, 0, 0.3);
  display: flex;
  justify-content: center;
  line-height: 30px;

  span {
    margin-left: auto;
  }
}

.titlebar {
  -webkit-user-select: none;
  -webkit-app-region: drag;
}
</style>
