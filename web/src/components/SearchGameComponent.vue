<template>
  <div v-if="show" class="mini-game dark-theme" @mousemove="handleMouseMove">
    <div class="grid">
      <div
        class="grid-element"
        v-for="(element, index) in grid"
        :key="index"
        @click="handleGridClick(index)"
      >
        <i v-if="element.showIcon" :class="element.icon"></i>
      </div>
      <div class="grid-overlay"></div>
    </div>

    <div class="timer">{{ formattedTime }}</div>
  </div>
</template>

<script>
import axios from "axios";
import threeBeepSound from "../../public/audio/threetonebeep.wav";
import successBeep from "../../public/audio/successbeep.wav";
import failBeep from "../../public/audio/failbeep.wav";
export default {
  data() {
    return {
      grid: [],
      timer: null,
      cursorPosition: { x: 0, y: 0 },
      currentIndex: null,
      clickCount: 0, // Current number of clicks
      internalTime: 0, // Internal property to store the time value
    };
  },
  props: {
    show: {
      type: Boolean,
      required: true,
      default: false,
    },
    time: {
      type: Number,
      required: true,
      default: 10,
    },
    additionalTime: {
      type: Number,
      required: true,
      default: 5,
    },
    targetClicks: {
      type: Number,
      required: true,
      default: 5,
    },
  },
  computed: {
    formattedTime() {
      const minutes = Math.floor(this.internalTime / 60);
      const seconds = this.internalTime % 60;
      return `${minutes.toString().padStart(2, "0")}:${seconds.toString().padStart(2, "0")}`;
    },
  },
  watch: {
    show(value) {
      if (value) {
        this.initializeGame();
      } else {
        this.stopTimer();
      }
    },
  },
  methods: {
    playBeepSound() {
      const audio = new Audio(threeBeepSound);
      audio.play();
    },
    playSuccessSound() {
      const audio = new Audio(successBeep);
      audio.play();
    },
    playFailSound() {
      const audio = new Audio(failBeep);
      audio.play();
    },
    initializeGame() {
      this.grid = Array.from({ length: 9 * 6 }, () => ({
        showIcon: false,
        clicked: false,
        icon: "fas fa-gem",
      }));

      const randomIndex = Math.floor(Math.random() * this.grid.length);
      this.grid[randomIndex].showIcon = true;
      this.currentIndex = randomIndex;

      this.startTimer();
    },
    startTimer() {
      this.internalTime = this.time; // Set the initial internal time value

      this.timer = setInterval(() => {
        this.internalTime--; // Decrement the internal time

        if (this.internalTime <= 0) {
          // Handle fail status
          this.playFailSound()
          clearInterval(this.timer);
          axios
            .post("https://gems-ui/completeSearchGame", {
              didPass: false,
            })
            .then((response) => {
              console.log(response.data);
            })
            .catch((error) => {
              console.error(error);
            });
          this.$emit("complete-search-game");
          this.clickCount = 0
        }
      }, 1000);
    },
    stopTimer() {
      clearInterval(this.timer);
    },
    handleGridClick(index) {
      if (this.grid[index].showIcon && !this.grid[index].clicked) {
        this.playBeepSound();
        this.grid[index].clicked = true;
        this.grid[index].showIcon = false;
        this.clickCount++;
        if (this.clickCount === this.targetClicks) {
          // Handle success
          this.playSuccessSound();
          axios
            .post("https://gems-ui/completeSearchGame", {
              didPass: true,
            })
            .then((response) => {
              console.log(response.data);
            })
            .catch((error) => {
              console.error(error);
            });
          this.$emit("complete-search-game");
          this.clickCount = 0
          clearInterval(this.timer);
        } else {
          let randomIndex;
          do {
            randomIndex = Math.floor(Math.random() * this.grid.length);
          } while (this.grid[randomIndex].clicked);

          this.grid[randomIndex].showIcon = true;
          this.currentIndex = randomIndex;

          this.internalTime = Math.min(this.internalTime + this.additionalTime, 60);
        }
      }
    },
    handleMouseMove(event) {
      const overlay = document.querySelector(".grid-overlay");
      const rect = overlay.getBoundingClientRect();
      const offsetX = event.clientX - rect.left;
      const offsetY = event.clientY - rect.top;
      const holeRadius = 50;

      overlay.style.backgroundImage = `
        radial-gradient(circle at ${offsetX}px ${offsetY}px, transparent ${holeRadius}px, rgba(0, 0, 0, 1.0) ${
        holeRadius + 1
      }px)
      `;
    },
  },
};
</script>

<style scoped>
.mini-game {
  display: flex;
  flex-direction: column;
  justify-content: center;
  width: 50%;
  height: 60%;
  border-style: solid;
  border-width: 2px;
  border-color: black;
  padding: 15px;
}

.dark-theme {
  background-color: #333;
  border-color: #666;
}

.grid {
  position: relative;
  display: grid;
  grid-template-columns: repeat(9, 1fr);
  grid-template-rows: repeat(6, 1fr);
  width: 100%;
  padding: 5px;
  border-style: solid;
  border-width: 2px;
  border-color: black;
}

.grid-element {
  position: relative;
  background-color: #555;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: none;
  min-height: 60px;
}

.grid-element i {
  color: #5b09b4;
  font-size: 24px;
}

.timer {
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 24px;
  color: #fff;
  margin-top: 20px;
  font-family: monospace;
  text-align: center;
}

.grid-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 2;
  pointer-events: none;
  background-image: radial-gradient(
    circle at 0px 0px,
    transparent 1px,
    rgba(0, 0, 0, 1) 1px
  );
}
</style>
