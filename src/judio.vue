<template>
  <div
    id="judio"
    :class="{ judio: active }"
    @mouseenter="hover"
    @mouseleave="hover"
  >
    <video
      class="video"
      :src="url"
      @canplay="updatePaused"
      @playing="updatePaused"
      @pause="updatePaused"
    ></video>
    <div :class="{ dimming: active }" @click="play_pause"></div>
    <div class="btn-play-paused" v-show="active">
      <span v-show="paused" class="material-icons btn-play" @click="play"
        >play_arrow
      </span>
      <span v-show="playing" class="material-icons btn-pause" @click="pause"
        >pause
      </span>
    </div>
    <div class="control-panel">
      <div
        class="video-track"
        ref="videoTrack"
        @click="moveToHere"
      >
        <!--------------------где будет опущена клавиша мыши, от туда и будет проигрываться видос-------------------->
        <div class="time-line" :style="{ width: widthTimeLine }"></div>
        <!--------------------где будет опущена клавиша мыши, от туда и будет проигрываться видос-------------------->
      </div>
      <div class="btn-panel">
        <span class="dial">00:00/00:00</span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data: () => ({
    videoElement: null,
    paused: null,
    active: false,
    videoPlay: null,
    widthTimeLine: "",
    posX: null,
    time: null,
  }),
  props: {
    url: {
      type: String,
      default:
        "https://firebasestorage.googleapis.com/v0/b/judio-10aa1.appspot.com/o/videos%2F16222774131560.webm?alt=media&token=488385bc-e4e4-447e-bd31-e1f789d045f8",
    },
  },
  methods: {
    updatePaused(e) {
      this.videoElement = e.target;

      this.paused = e.target.paused;
      console.log();
    },
    play_pause() {
      if (this.paused) {
        this.play();
      } else this.pause();
    },
    play() {
      this.videoElement.play();
    },
    //-------------------- Переход к некоторому времени по клику -------------------- //
    moveToHere(e) {
      if (e.which != 1) return null;
      if (window.innerWidth > 420) {
        this.posX = e.clientX - (window.innerWidth / 100) * 10;
      } else this.posX = e.clientX;
      this.time = (this.posX * 100) / this.$refs.videoTrack.offsetWidth;
      this.widthTimeLine = this.time + "%";
      this.videoElement.currentTime = (this.time * this.videoElement.duration) / 100;
    },
    //-------------------- Переход к некоторому времени по клику -------------------- //
    dragAndDropHere(e) {
      this.moveToHere(e);
    },
    pause() {
      this.videoElement.pause();
      clearInterval(this.videoPlay);
    },
    //-------------------- Метод, который умрет -------------------- //
    hover() {
      this.active = !this.active;
    },
    //-------------------- Метод, который умрет -------------------- //
  },
  computed: {
    playing() {
      return !this.paused;
    },
  },
  mounted() {},
};
</script>

<style scoped>
@media screen and (max-width: 420px) {
  .control-panel {
    width: 100%;
  }
  .material-icons {
    font-size: 4em;
  }
}
@media screen and (min-width: 420px) and (max-width: 960px) {
  .control-panel {
    width: 80%;
  }
  .material-icons {
    font-size: 5em;
  }
}
@media screen and (min-width: 960px) {
  .control-panel {
    width: 80%;
  }
  .material-icons {
    font-size: 6em;
  }
}
.material-icons {
  position: relative;
  color: #4c92e7;
  cursor: pointer;
}
#judio {
  position: relative;
  background-color: #000;
  display: flex;
  overflow: hidden;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  -khtml-user-select: none;
}
.judio::before {
  z-index: 1;
  top: 0;
  height: 5%;
  background-image: linear-gradient(
    180deg,
    rgba(0, 0, 0, 0.4),
    rgba(0, 0, 0, 0)
  );
}
.judio::after {
  bottom: 0;
  height: 15%;
  background-image: linear-gradient(0deg, rgba(0, 0, 0, 0.4), rgba(0, 0, 0, 0));
}
.judio::after,
.judio::before {
  content: "";
  position: absolute;
  width: 100%;
  filter: blur(8px);
}
.video {
  position: absolute;
  width: 100%;
  height: 100%;
}
.dimming {
  position: absolute;
  width: 100%;
  height: 100%;
}
.control-panel {
  position: absolute;
  z-index: 20;
  height: 5em;
  bottom: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
}
.active-zone {
  position: absolute;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
.video-track {
  position: absolute;
  background-color: rgba(255, 255, 255, 0.3);
  width: 100%;
  height: 0.3em;
  cursor: pointer;
  transition-duration: 0.15s;
  padding: 0.5em 0 0.5em 0;
  background-clip: content-box;
}
.video-track:hover {
  height: 0.725em;
  padding: 0.8em 0 0.8em 0;
  background-clip: content-box;
}
.time-line {
  position: relative;
  height: 100%;
  width: 0;
  top: 0;
  background-color: blueviolet;
}
.btn-panel {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: start;
}
/* .time-line:after {
  content: "";
  position: absolute;
  width: 5px;
  height: 5px;
  right: -3px;
  border-radius: 50%;
  background-color: hotpink;
} */
.dial {
  color: aquamarine;
}
</style>
