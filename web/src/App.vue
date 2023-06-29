<!-- This is a Vue 3 single-file component -->
<template>
  <!-- Here we're rendering the HelloWorld component -->
  <KeyPadComponent
    class="main"
    :show="showKeyPad"
    @confirm-keypad="confirmKeypad"
  />
  <SearchGameComponent
    class="main"
    :show="showSearchGame"
    :time="sgTime"
    :additionalTime="sgAdditionalTime"
    :targetClicks="sgTargetClicks"
    @complete-search-game="completeSearchGame"
  />
  <LoadingComponent
    class="main"
    :show="showLoading"
    :data="loadingData"
    @loading-complete="loadingComplete"
  />
</template>

<script setup>
import KeyPadComponent from "./components/KeyPadComponent.vue";
import SearchGameComponent from "./components/SearchGameComponent.vue";
import LoadingComponent from "./components/LoadingComponent.vue";
import { onMounted, ref } from "vue";
import axios from "axios";

// Show Components
const showKeyPad = ref(false);

onMounted(() => {
  window.addEventListener("message", handleNUIEvent);
  window.addEventListener("keydown", handleKeyDown);
});

function handleNUIEvent(event) {
  if (event.data.action === "openKeyPad") {
    showKeyPad.value = event.data.show;
  }
  if (event.data.action === "closeKeyPad") {
    showKeyPad.value = event.data.show;
    showSearchGame.value = event.data.show;
  }
  if (event.data.action === "openSearchGame") {
    showLoading.value = event.data.show;
    loadingData.value = event.data;
  }
}

function handleKeyDown(event) {
  if (event.key === "Escape") {
    axios
      .post("https://gems-ui/closeKeyPad", {})
      .then((response) => {
        console.log(response.data);
      })
      .catch((error) => {
        console.error(error);
      });
  }
}
</script>
<script>
//Search Game Props
const sgTime = ref(10);
const sgAdditionalTime = ref(1);
const sgTargetClicks = ref(5);
const showSearchGame = ref(false);
const showLoading = ref(false);
const loadingData = ref(null);

export default {
  methods: {
    confirmKeypad() {
      this.showKeyPad = false;
    },
    completeSearchGame() {
      this.showSearchGame = false;
    },
    loadingComplete(event) {
      showLoading.value = false;
      sgTime.value = event.time;
      sgAdditionalTime.value = event.additionalTime;
      sgTargetClicks.value = event.targetClicks;
      showSearchGame.value = event.show;
    },
  },
};
</script>
<style scoped>
.main {
  user-select: none;
  margin: auto;
  margin-top: 15rem;
}
</style>
