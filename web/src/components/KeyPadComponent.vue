<template class="template" v-if="show">
    <div class="keypad dark-mode main-container" v-if="show">
      <div class="display">{{ displayText }}</div>
      <div class="keypad-buttons">
        <button v-for="(number, index) in keypadNumbers" :key="index" @click="updateDisplay(number)">{{ number }}</button>
        <div class="button-group">
          <button @click="cancel" class="cancel"><i class="fas fa-times"></i></button>
          <button class="zero" @click="updateDisplay(0)">0</button>
          <button @click="confirm" class="confirm"><i class="fas fa-check"></i></button>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import axios from 'axios';
  import keypadSound from '../../public/audio/keypadpress.wav'
  export default {
    data() {
      return {
        displayText: "",
        keypadNumbers: [1, 2, 3, 4, 5, 6, 7, 8, 9]
      };
    },
    props: {
        show: {
            type: Boolean,
            required: true,
            default: false,
        },
    },
    methods: {
      updateDisplay(number) {
        this.playSound()
        this.displayText += number;
      },
      confirm() {
        const returnValue = this.displayText
        axios.post('https://gems-ui/confirmKeyPad', {
            returnValue
        })
        .then(response => {
            console.log(response.data);
        })
        .catch(error => {
            console.error(error);
        })
        this.displayText = "";
        this.$emit('confirm-keypad')
      },
      cancel() {
        this.displayText = "";
      },
      playSound() {
        const audio = new Audio(keypadSound);
        audio.play();
      }
    }
  };
  </script>
  
  <style scoped>
  .keypad {
    width: 400px;
    background-color: #222;
    padding: 10px;
    display: flex;
    flex-direction: column;
    color: #fff;
    margin: auto;
    margin-top: 10rem;
  }
  
  .display {
    height: 90px;
    border: 1px solid #555;
    margin-bottom: 10px;
    padding: 10px;
    background-color: #333;
    font-size: 32px;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  
  .keypad-buttons {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 5px;
  }
  
  button {
    width: 120px;
    height: 120px;
    border: none;
    border-radius: 5px;
    background-color: #555;
    font-size: 36px;
    color: #fff;
    cursor: pointer;
  }
  
  button i {
    font-size: 32px;
  }
  
  button i:before {
    font-family: "Font Awesome 5 Free";
    font-weight: 900;
  }
  
  .button-group {
    display: flex;
    grid-column: 1 / span 3;
    grid-row: 4;
    justify-content: space-between;
  }
  
  .cancel {
    flex: 1;
    margin-right: 5px;
  }
  
  .zero {
    flex: 1;
    margin: 0 5px;
  }
  
  .confirm {
    flex: 1;
    margin-left: 5px;
    margin-right: 5px;
  }
  
  .dark-mode {
    color: #fff;
    background-color: #222;
  }

  .template {
      background: transparent;
    }

    .main-container {
        max-width: 50%;
        max-height: 75vh;
    }
  </style>
  