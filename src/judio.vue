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
        @mousedown="clickOnTrack" 
        @mouseout="mouseOverElem = false"
        @mouseenter="mouseOverElem = true"
      ><!--где будет опущена клавиша мыши, от туда и будет проигрываться видос-->
        <div class="time-line" :class="{ ':after': mouseOverElem }" :style="{ width: widthTimeLine }"></div>
      </div>
      <span class="dial">00:00/00:00</span>
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
    mouseOverElem: false,
    posX: null,
    time: null,
  }),
  props: {
    url: {
      type: String,
      default:
        "https://firebasestorage.googleapis.com/v0/b/judio-10aa1.appspot.com/o/videos%2F16201271506042.webm?alt=media&token=60672d27-282e-49b6-b4b6-466b33d52934",
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
      this.condition = false;
      this.videoPlay = setInterval(() => {
        let videoTime = this.videoElement.currentTime;
        let videoLenght = Math.round(this.videoElement.duration);
        this.widthTimeLine = (videoTime * 100) / videoLenght + "%";
      });
      let time = new Date(this.videoElement.duration);
      this.durationOfVideo = [
        time.getHours(),
        time.getMinutes(),
        time.getSeconds(),
      ]
        .map((x) => {
          return x < 10 ? "0" + x : x;
        })
        .join(":");
      this.currentTimeOfVideo = this.videoElement.currentTime;
    },
    clickOnTrack(e) {
      this.posX = e.offsetX; //берем координату по X, над которой была нажата ЛКМ... // НИКОГДА! НИ-КО-ГДА!!! НЕ ИСПОЛЬЗУЙ clientX
      this.time = (this.posX * 100) / this.$refs.videoTrack.offsetWidth; // процент нашего Х
      this.widthTimeLine = this.time + "%"; // Выравниваем стили ориентируясь на наш процент
      this.videoElement.currentTime = (this.time * this.videoElement.duration) / 100; // перематываем видос в место, где мы нажали
    },
    mouseUp() {
      this.time = (this.posX * 100) / this.$refs.videoTrack.offsetWidth; // процент нашего Х
      this.widthTimeLine = this.time + "%"; // Выравниваем стили ориентируясь на наш процент
      // this.videoElement.currentTime = (time * this.videoElement.duration) / 100; // перематываем видос в место, где мы нажали
      this.returnPosX
    },
    pause() {
      this.videoElement.pause();
      this.condition = true;
      clearInterval(this.videoPlay);
    },
    hover() {
      this.active = !this.active;
    },
  },
  computed: {
    playing() {
      return !this.paused;
    },
    returnPosX() {
      // this.videoElement.currentTime = (this.time * this.videoElement.duration) / 100;
    }
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
  height: 10%;
  bottom: 0;
  display: flex;
  justify-content: center;
  align-items: center;
}
.video-track {
  position: relative;
  background-color: rgba(255, 255, 255, 0.3);
  width: 100%;
  height: 5px;
  cursor: pointer;
}
.video-track:hover {
  padding-top: 5px;
}
.time-line {
  position: absolute;
  height: 100%;
  width: 0;
  bottom: 0;
  background-color: blueviolet;
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
