<template>
  <div>
    <div class="draggable"> <h4>Drag Window</h4> </div>
    <button @click="startRecording">Start Recording</button>
    <button @click="stopRecording">Stop Recording</button>
    <button id="screen-shot" @click="setRecordingArea">Select Recording Area</button>
    <div id="screenshot-path"></div>


    Current recording state: {{recording}}
  </div>
</template>
<script>

export default {
  name: 'landing-page',
  data () {
    return {
      recording: false
    }
  },
  methods: {
    setRecordingArea (e) {
      // console.log(fs)
    },
    startRecording (e) {
      this.recording = true
    },
    stopRecording (e) {
      this.recording = false
    }

  },
  mounted () {

  }
}
</script>


<style>
@import url("https://fonts.googleapis.com/css?family=Source+Sans+Pro");

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: "Source Sans Pro", sans-serif;
  background: rgba(0,0,0,0);
  border: 2px solid rgb(77, 139, 255);
  height: 100vh;
  width: 100vw;
}

.draggable {
  -webkit-app-region: drag;
  height: 30px;
  color: #fff;
  background: rgba(0,0,0, 0.3);

}
.titlebar {
  -webkit-user-select: none;
  -webkit-app-region: drag;
}

</style>
