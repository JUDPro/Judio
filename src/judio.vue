<template>
  <div
    id="judio"
    :class="{ judio: video.active }"
    @mouseenter="video.active = true"
    @mouseleave="video.active = false"
    ref="widthParent"
  >
    <video
      class="video"
      :src="url"
      @canplay="updatePaused"
      @playing="updatePaused"
      @pause="updatePaused"
    ></video>
    <div :class="{ dimming: video.active }" @click="play_pause"></div>
    <div class="btn-play-paused" v-show="video.active">
      <span v-show="video.paused" class="material-icons btn-play" @click="play"
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
        @mousemove="relocate"
      >
        <!--------------------где будет опущена клавиша мыши, от туда и будет проигрываться видос-------------------->
        <div class="time-line" :style="{ width: video.widthTimeLine }">
          <span class="pull"></span>
        </div>
        <!--------------------где будет опущена клавиша мыши, от туда и будет проигрываться видос-------------------->
      </div>
      <div class="btn-panel">
        <div class="dial">{{ video.seconds }}/{{ video.fullTime }}</div>
        <div class="settings"></div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data: () => ({
    video: {
      videoElement: null,
      paused: null,
      active: false,
      videoPlay: null,
      widthTimeLine: "",
      posX: null,
      time: null,
      skip: 5,
      seconds: 0,
      fullTime: 0,
    },
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
      this.video.videoElement = e.target;
      this.video.paused = e.target.paused;
      this.video.fullTime =  Math.round(this.video.videoElement.duration);
    },
    play_pause() {
      if (this.video.paused) {
        this.play();
      } else this.pause();
    },
    play() {
      this.video.videoElement.play();
      this.video.videoPlay = setInterval(() => {
        this.video.seconds = Math.round(this.video.videoElement.currentTime);
        let videoTime = this.video.videoElement.currentTime;
        let videoLenght = Math.round(this.video.videoElement.duration);
        this.video.widthTimeLine = (videoTime * 100) / videoLenght + "%";
      });
    },
    pause() {
      this.video.statePause = this.video.paused;
      this.video.videoElement.pause();
      clearInterval(this.videoPlay);
    },
    //-------------------- Переход к некоторому времени по клику --------------------//
    moveToHere(e) {
      if (e.which != 1) return null;
      else {
        if (window.innerWidth > 420) {
          this.video.posX =
            e.clientX - (this.$refs.widthParent.clientWidth / 100) * 10;
        } else this.video.posX = e.clientX;
        this.video.time =
          (this.video.posX * 100) / this.$refs.videoTrack.offsetWidth;
        this.video.widthTimeLine = this.video.time + "%";
        this.video.videoElement.currentTime =
          (this.video.time * this.video.videoElement.duration) / 100;
      }
    },
    relocate(e) {
      this.moveToHere(e);
    },
    //-------------------- Переход к некоторому времени по клику --------------------//
    skip() {
      this.video.videoElement.currentTime += this.video.skip;
    },
    back() {
      this.video.videoElement.currentTime -= this.video.skip;
    },
  },
  computed: {
    playing() {
      return !this.video.paused;
    },
  },
  mounted() {
    document.addEventListener("keydown", (e) => {
      switch (e.code) {
        case "Space": {
          this.play_pause();
          break;
        }
        case "ArrowRight": {
          this.skip();
          break;
        }
        case "ArrowLeft": {
          this.back();
          break;
        }
        default:
          return null;
      }
    });
  },
};
</script>

<style scoped>
.test {
  width: 600px;
  height: 350px;
}
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
  font-family:'Segoe UI';
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
  overflow: hidden;
}
.pull {
  position: absolute;
  background: aquamarine;
  right: 0;
  height: 100%;
  width: 0.3em;
}
.btn-panel {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: start;
  flex-direction: column;
}
.dial {
  color: aquamarine;
  height: 50%;
}
.settings {
  height: 50%;
}
</style>
