<!-- This is a Vue 3 single-file component -->
<template>
  <!-- Here we're rendering the HelloWorld component -->
  <KeyPadComponent class="main" :show="showKeyPad" @confirm-keypad="confirmKeypad"/>
</template>

<script setup>
import KeyPadComponent from "./components/KeyPadComponent.vue";
import { onMounted, ref } from 'vue';
import axios from 'axios';

const showKeyPad = ref(false)

onMounted(() => {
  window.addEventListener('message', handleNUIEvent);
  window.addEventListener('keydown', handleKeyDown);
});

function handleNUIEvent(event) {
  if (event.data.action === "openKeyPad") {
    showKeyPad.value = event.data.show
  }
  if (event.data.action === "closeKeyPad") {
    showKeyPad.value = event.data.show
  }
}

function handleKeyDown(event) {
  if (event.key === 'Escape') {
    axios.post('https://gems-ui/closeKeyPad', {})
      .then(response => {
          console.log(response.data);
      })
      .catch(error => {
          console.error(error);
      }
    )
  }
}

</script>
<script>
  export default {
  methods: {
    confirmKeypad() {
      this.showKeyPad = false
    }
  }
}
</script>
<style scoped>
.main {
  user-select: none;
  margin: auto;
  margin-top: 15rem;
}
</style>