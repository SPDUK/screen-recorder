<template>
  <div>
    <div class="draggable">
      <h4>Drag Window</h4>
    </div>
    <button @click="startRecording">Start Recording</button>
    <button @click="stopRecording">Stop Recording</button>
    <button id="screen-shot" @click="setRecordingArea">Select Recording Area</button>
    <div id="screenshot-path"></div>
    Current recording state: {{recording}}
  </div>
</template>
<script>
const fs = require('fs')
const { desktopCapturer } = require('electron')

var blobs = []

export default {
  name: 'landing-page',
  data () {
    return {
      recording: false,
      recorder: {}
    }
  },
  methods: {
    setRecordingArea (e) {
      // console.log(fs)
    },

    startRecording () {
      this.recording = true
      var title = document.title

      desktopCapturer.getSources({ types: ['screen'] }, (error, sources) => {
        console.log(sources)
        if (error) throw error
        for (let i = 0; i < sources.length; i++) {
          let src = sources[i]
          document.title = title

          navigator.webkitGetUserMedia({
            audio: false,
            video: {
              mandatory: {
                chromeMediaSource: 'desktop',
                chromeMediaSourceId: src.id,
                minWidth: 800,
                maxWidth: 1280,
                minHeight: 600,
                maxHeight: 720
              }
            }
          }, this.handleStream, this.handleUserMediaError)
        }
      })
    },

    handleStream (stream) {
      console.log('handling stream')
      this.recorder = new MediaRecorder(stream)
      console.log(this.recorder)
      blobs = []
      this.recorder.ondataavailable = (event) => {
        blobs.push(event.data)
      }
      this.recorder.start()
    },

    stopRecording () {
      console.log('stopping...')
      this.recording = false
      this.recorder.stop()
      this.toArrayBuffer(new Blob(blobs, {type: 'video/webm'}), (ab) => {
        var buffer = this.toBuffer(ab)
        var file = `./videos/example.webm`
        fs.writeFile(file, buffer, (err) => {
          if (err) {
            console.error('Failed to save video ' + err)
          } else {
            console.log('Saved video: ' + file)
          }
        })
      })
    },

    handleUserMediaError (e) {
      console.error('handleUserMediaError', e)
    },
    toArrayBuffer (blob, cb) {
      let fileReader = new FileReader()
      fileReader.onload = function () {
        let arrayBuffer = this.result
        cb(arrayBuffer)
      }
      fileReader.readAsArrayBuffer(blob)
    },

    toBuffer (ab) {
      let buffer = Buffer.alloc(ab.byteLength)
      let arr = new Uint8Array(ab)
      for (let i = 0; i < arr.byteLength; i++) {
        buffer[i] = arr[i]
      }
      return buffer
    }

  },
  mounted () {
    this.startRecording()
    setTimeout(() => { this.stopRecording() }, 2000)
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
