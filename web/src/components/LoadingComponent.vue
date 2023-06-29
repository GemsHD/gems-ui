<template>
  <div class="loading-component dark-theme" v-if="show">
    <div class="loading-icon">
      <i class="fas fa-spinner fa-spin"></i>
    </div>
    <div class="loading-text">Bypass Tool Loading...</div>
    <div class="loading-progress" :style="{ width: progressBarWidth }"></div>
  </div>
</template>

<script>
import threeBeepSound from "../../public/audio/threetonebeep.wav";
export default {
  data() {
    return {
      progress: 0,
      intervalId: null,
      duration: 5000, // Progress bar duration in milliseconds
    };
  },
  props: {
    show: {
      type: Boolean,
      required: true,
      default: false,
    },
    data: {
        type: null,
        required: true,
        default: null
    }
  },
  computed: {
    progressBarWidth() {
      return `${this.progress}%`;
    },
  },
  methods: {
    playBeepSound() {
      const audio = new Audio(threeBeepSound);
      audio.play();
    },
    startCountdown() {
      this.intervalId = setInterval(() => {
      this.progress += 1;
      if (this.progress >= 100) {
        clearInterval(this.intervalId);
        // Emit an event when progress is complete
        this.playBeepSound();
        this.$emit("loading-complete", this.data);
        this.progress = 0
      }
    }, this.duration / 100);
    },
  },
  watch: {
    show(value) {
      if (value) {
        this.startCountdown();
      }
    },
  },
};
</script>

<style scoped>
.loading-component {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 500px;
  height: 300px;
  border: 2px solid #666;
  border-radius: 5px;
  padding: 10px;
}

.dark-theme {
  background-color: #333;
  color: #fff;
}

.loading-icon {
  font-size: 24px;
}

.loading-text {
  margin-top: 10px;
  font-weight: bold;
}

.loading-progress {
  height: 10px;
  background-color: #666;
  margin-top: 20px;
  transition: width 0.2s;
}
</style>
