<template>
  <div
    id="judio"
    :class="{ judio: video.active }"
    @mousemove="userIsNotActive"
    @mouseleave="video.active = false"
    ref="widthParent"
  >
    <video
      class="video"
      @canplay="updatePaused"
      @playing="updatePaused"
      @pause="updatePaused"
    >
      <source :src="url" type="video/mp4" />
      <source :src="url" type="video/webm" />
    </video>
    <div :class="{ dimming: video.active }" @click="play_pause"></div>
    <div class="btn-play-paused" v-show="video.active">
      <span v-show="video.paused" class="material-icons btn-play" @click="play"
        >play_arrow
      </span>
      <span v-show="playing" class="material-icons btn-pause" @click="pause"
        >pause
      </span>
    </div>
    <div :class="{ 'control-panel': video.active }" v-show="video.active">
      <div
        class="video-track"
        ref="videoTrack"
        @click="moveToHerePC"
        @mousemove="moveToHerePC"
        @touch="moveToHereMobile"
        @touchmove="moveToHereMobile"
      >
        <!-------------------- где будет опущена клавиша мыши, от туда и будет проигрываться видос -------------------->
        <div class="time-line" :style="{ width: video.widthTimeLine }">
          <span class="pull"></span>
        </div>
        <!-------------------- где будет опущена клавиша мыши, от туда и будет проигрываться видос -------------------->
      </div>
      <div class="btn-panel">
        <div class="settings">
          <div class="fullscreen">
            <span class="material-icons" @click="fullSizeWindow"
              >fullscreen</span
            >
          </div>
          <div class="dial">
            {{ video.currentMinutes }}:{{ video.currentSecond }}/
            {{ video.minutes }}:{{ video.seconds }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
//import video from "./video for tests/1.webm"; // длинный webm
//import video from "./video for tests/2.webm"; // webm широкий
import video from "./video for tests/3.mp4"; // видео 36 секунд
//import video from "./video for tests/4.mp4"; // видео 4к

export default {
  data: () => ({
    video: {
      videoElement: null, // сам элемент с видео
      paused: null, // состояние паузы (true/false)
      active: false, // переменная для добавления н-х стилей
      videoPlay: null, // нужна для правильной работы функций play() / pause()
      widthTimeLine: "", // нужна для стилей
      posX: null, // нужна для расчета позиции нажатия мыши
      time: null, // вспомогательная переменная для расчета
      skip: 5, // переменная для перемотки видео
      currentSecond: "00", // секунда в данный момент
      currentMinutes: "0", // минута в данный момент (высчитывается из секунд)
      seconds: 0, // общее количество секунд
      minutes: 0, // общее количество минут (высчитывается из секунд)
      timer: null,
    },
    x: null,
    y: null,
  }),
  props: {
    url: {
      type: String,
      default: video,
    },
  },
  methods: {
    //-------------------- Метод для инициализации некоторых значений --------------------//
    updatePaused(e) {
      this.video.videoElement = e.target;
      this.video.seconds =
        this.video.videoElement.duration % 60 < 10
          ? "0" + (Math.round(this.video.videoElement.duration) % 60)
          : Math.round(this.video.videoElement.duration % 60);
      this.video.minutes = Math.round(
        (this.video.videoElement.duration - this.video.seconds) / 60
      );
      this.video.paused = e.target.paused;
    },
    //-------------------- Метод для инициализации некоторых значений --------------------//

    //-------------------- Этот метод не будет жить, наверное --------------------//
    play_pause() {
      if (this.video.paused) {
        this.play();
      } else this.pause();
    },
    //-------------------- Этот метод не будет жить, наверное --------------------//

    //-------------------- Методы для проигрывания и паузы  --------------------//
    play() {
      this.video.videoElement.play();
      this.video.videoPlay = setInterval(() => {
        // пришел сОдОмИт и оставил здесь этот код
        this.video.currentSecond =
          this.video.currentSecond < 10
            ? "0" + (Math.round(this.video.videoElement.currentTime) % 60)
            : Math.round(this.video.videoElement.currentTime) % 60;
        this.video.currentMinutes = Math.round(
          (this.video.videoElement.currentTime - this.video.currentSecond) / 60
        );
        let videoTime = this.video.videoElement.currentTime;
        let videoLenght = Math.round(this.video.videoElement.duration);
        this.video.widthTimeLine = (videoTime * 100) / videoLenght + "%";
      });
    },
    pause() {
      this.video.statePause = this.video.paused;
      this.video.videoElement.pause();
      clearInterval(this.videoPlay);
      this.video.active = true;
    },
    //-------------------- Методы для проигрывания и паузы --------------------//

    //-------------------- Скрывает управление через некоторое время --------------------//
    userIsNotActive(e) {
      if (this.video.active == false) {
        this.video.active = true;
      } // здесь появляется панель управления при движении мыши
      // ниже я хочу сравнить координаты мыши. Если они равны в течении 3х секунд, что панель управления нужно скрыть.
      let X = e.clientX;
      let Y = e.clientY;
      if (this.x == null && this.y == null) {
        this.x = e.clientX;
        this.y = e.clientY;
      }
      this.$refs.widthParent.style.cursor = "default";
      // эта функция в целом работает, но не так, как хотелось бы... её нужно переписать.
      // условие ниже будет срабатывать всегда, т.к. сравниваются значения из первого ивента,
      // а мне нужно следить за переменными, то есть нужно следить за их содержимым: если содержимое не меняется в течении
      // некотрого времени, то this.video.active = false (убираю панель управления). В общем, опять переписывать...
      this.video.timer = setTimeout(() => {
        if (this.x == X && this.y == Y) {
          this.video.active = false;
          this.x = null;
          this.y = null;
          this.$refs.widthParent.style.cursor = "none";
        }
        this.video.timer = null;
      }, 3000);
    },
    //-------------------- Скрывает управление через некоторое время --------------------//

    //-------------------- Переход к некоторому времени по клику --------------------//
    // Метод для пк
    moveToHerePC(e) {
      if (e.which != 1) return null;
      else {
        if (window.innerWidth <= 420) {
          this.video.posX = e.clientX;
        } else
          this.video.posX =
            e.clientX - (this.$refs.widthParent.clientWidth / 100) * 10;
        this.video.time =
          (this.video.posX * 100) / this.$refs.videoTrack.offsetWidth;
        this.video.widthTimeLine = this.video.time + "%";
        this.video.videoElement.currentTime =
          (this.video.time * this.video.videoElement.duration) / 100;
      }
    },
    // Метод для мобилки
    moveToHereMobile(e) {
      if (window.innerWidth <= 420)
        this.video.posX = e.changedTouches[0].clientX;
      else
        this.video.posX =
          e.changedTouches[0].clientX -
          (this.$refs.widthParent.clientWidth / 100) * 10;
      this.video.time =
        (this.video.posX * 100) / this.$refs.videoTrack.offsetWidth;
      this.video.widthTimeLine = this.video.time + "%";
      this.video.videoElement.currentTime =
        (this.video.time * this.video.videoElement.duration) / 100;
    },
    //-------------------- Переход к некоторому времени по клику --------------------//

    //-------------------- Методы для скипа видео на 5 секунд --------------------//
    skip() {
      this.video.videoElement.currentTime += this.video.skip;
    },
    back() {
      this.video.videoElement.currentTime -= this.video.skip;
    },
    //-------------------- Методы для скипа видео на 5 секунд --------------------//

    //-------------------- Полноэкранный режим --------------------//
    fullSizeWindow() {
      //console.log(screen.orientation) нужно реализовать поворот экрана на мобильных устройствах
      let box = this.$refs.widthParent;
      if (document.fullscreenElement !== null) document.exitFullscreen();
      else box.requestFullscreen();
    },
    //-------------------- Полноэкранный режим --------------------//
  },
  computed: {
    playing() {
      return !this.video.paused; // меняем состояние паузы
    },
  },
  mounted() {
    // назначаю горячие клавиши
    document.addEventListener("keydown", (e) => {
      switch (e.code) {
        // пауза на пробел
        case "Space": {
          this.play_pause();
          break;
        }
        // скип на пять секунд вперед, по нажатию на правую стрелку
        case "ArrowRight": {
          this.skip();
          break;
        }
        // скип на пять секунд назад, по нажатию на левую стрелку
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
  color: #ffffff;
  cursor: pointer;
}
#judio {
  font-family: "Segoe UI";
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
  background-image: linear-gradient(0deg, rgba(0, 0, 0, 0.5), rgba(0, 0, 0, 0));
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
  position: absolute;
  color: rgb(255, 255, 255);
  height: 50%;
  padding: 0.5em 0 0 0;
}
.fullscreen {
  right: 0;
  bottom: 0.5em;
  position: absolute;
}
.fullscreen > .material-icons {
  font-size: 2em;
  color: rgb(255, 255, 255);
}
.settings {
  height: 50%;
  position: relative;
}
</style>
