<template>
  <div class="container">
    <div style="flex-direction: column; margin-right: 50px">
      <vs-input
        style="margin-bottom: 30px;"
        label-placeholder="Playfair Cipher"
        v-model="playfairCipher"
        autocomplete="off"
        @input="onPlayfairCipherInput"
      />
      <vs-select
        label-placeholder="Playfair Alphabet"
        v-model="playfairAlphabet"
        color="warn"
        autocomplete="off"
        @input="onPlayfairCipherInput(playfairCipher)"
      >
        <vs-option label="With Q without J" value="abcdefghiklmnopqrstuvwxyz">
          With Q without J
        </vs-option>
        <vs-option label="With J without Q" value="abcdefghijklmnoprstuvwxyz">
          With J without Q
        </vs-option>
      </vs-select>

      <vs-button relief @click="resetGrids">
        Clear
      </vs-button>
      <div class="fiveByFive" style="margin-bottom: 50px">
        5x5 on small mushroom
        <div class="row" v-for="row in 5" :key="row">
          <input
            v-for="col in 5"
            :key="'i' + col"
            type="text"
            :style="`background:${findInputBgColor(row-1, col-1, 0)}`"
            v-model="largeGrid[row-1][col-1]"
          >
        </div>
      </div>
      <div class="fourByFive">
        4x5 on large mushroom
        <div class="row" v-for="row in 5" :key="row">
          <input
            v-for="col in 4"
            :key="'i' + col"
            type="text"
            :style="`background:${findInputBgColor(row-1, col-1, 1)}`"
            v-model="smallGrid[row-1][col-1]"
          >
        </div>
      </div>
    </div>
    <div height="600px" style="flex-direction: column">
      <img height="600px" src="~assets/ElizaRaetselPilz.png">
      <p>Result: {{result}}</p>
    </div>
  </div>
</template>

<script>
const find = require('lodash.find');

export default {
  components: {
  },
  data: () => ({
    largeGrid: [],
    smallGrid: [],
    gridMapping: [
      { 'shroomIndex': 0, col: 0, row: 4, color: '#1fffff' },
      { 'shroomIndex': 0, col: 1, row: 0, color: '#b57edc' },
      { 'shroomIndex': 0, col: 3, row: 3, color: '#5f7edc' },
      { 'shroomIndex': 0, col: 0, row: 1, color: '#dcce5f' },
      { 'shroomIndex': 0, col: 1, row: 1, color: '#86dc5f' },
      { 'shroomIndex': 0, col: 1, row: 0, color: '#b57edc' },
      { 'shroomIndex': 0, col: 1, row: 2, color: '#333743' },
      { 'shroomIndex': 1, col: 0, row: 0, color: '#9d0b0b' },
      { 'shroomIndex': 1, col: 0, row: 3, color: '#34744a' },
      { 'shroomIndex': 0, col: 2, row: 3, color: '#d6744a' },
      { 'shroomIndex': 0, col: 2, row: 4, color: '#b6a119' },
      { 'shroomIndex': 1, col: 3, row: 3, color: '#ff00e9' },
    ],
    colorMapping: {},
    playfairCipher: '',
    playfairAlphabet: 'abcdefghiklmnopqrstuvwxyz',
    result: ''
  }),
  watch: {
    largeGrid: {
      deep: true,
      handler() {
        this.recalculateResult();
        this.focusNextInput()
      }
    },
    smallGrid: {
      deep: true,
      handler() {
        this.recalculateResult();
        this.focusNextInput()
      }
    }
  },
  methods: {
    recalculateResult() {
      const shrooms = [this.largeGrid, this.smallGrid];
      this.result = this.gridMapping.reduce((prev, cur) => {
        const val = shrooms[cur.shroomIndex][cur.row][cur.col];
        return prev + (val !== '' ? val : '_ ')
      }, '') + '.gif';
    },
    findInputBgColor(row, col, shroomIndex) {
      return (find(this.gridMapping, { col, row, shroomIndex }) || { color: '#403a3a' }).color
    },
    resetGrids() {
      this.largeGrid = [...Array(5)].map(e => Array(5).fill(''));
      this.smallGrid = [...Array(5)].map(e => Array(4).fill(''));
    },
    onPlayfairCipherInput(val) {
      const usedLetters = [];
      const cipherKey = val.split('');
      const alphabet = this.playfairAlphabet.split('');
      for (let row = 0; row < this.largeGrid.length; row++) {
        for (let col = 0; col < this.largeGrid[0].length; col++) {
          let nextLetter;
          do {
            nextLetter = cipherKey.length > 0
              ? cipherKey.shift()
              : alphabet.shift();
          } while (usedLetters.includes(nextLetter));

          this.$set(this.largeGrid[row], col, nextLetter);
          this.largeGrid[row][col] = nextLetter;
          usedLetters.push(nextLetter)
        }
      }
    },
    focusNextInput() {
      //add all elements we want to include in our selection
      // debugger;
      const focusableElements = 'a:not([disabled]), button:not([disabled]), input[type=text]:not([disabled]), [tabindex]:not([disabled]):not([tabindex="-1"])';
      if (document.activeElement) {
        const focusable = Array.prototype.filter.call(document.querySelectorAll(focusableElements),
          function (element) {
            //check for visibility while always include the current activeElement
            return element.offsetWidth > 0 || element.offsetHeight > 0 || element === document.activeElement
          });
        const index = focusable.indexOf(document.activeElement);
        if(index > -1) {
          const nextElement = focusable[index + 1] || focusable[0];
          nextElement.focus();
        }
      }
    }
  },
  created() {
    this.resetGrids()
  }
}
</script>

<style>
html {
  background-color: #2a2626;
  color: white;
  --vs-text: 255, 255, 255 !important;
  --vs-gray-2: 64, 58, 58 !important;
  --vs-gray-3: 64, 58, 58 !important;
  --vs-background: 64, 58, 58 !important;
}

.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.subtitle a {
  font-weight: 500;
  color: inherit;
}
input {
  padding: 5px;
  width: 30px;
  text-align: center;
  border-color: #2a2626;
  text-transform:uppercase;
  border-style: solid;
  color: white !important;
}

</style>
