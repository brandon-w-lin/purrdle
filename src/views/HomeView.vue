<template>
  <!-- WELCOME -->
  <div v-if="!isPlaying">
    <h1>Purrdle</h1>
    <h2>Like wordle, but it purrs</h2>
  </div>

  <!-- HOLD SUBMITTED GUESSES -->
  <div class="guess-holder" v-for="submission in submissions" :key="submission">
    <p
      class="letters submitted"
      :class="setClass(submission[1][index])"
      v-for="(letter, index) in submission[0]"
      :key="index"
    >
      {{ letter }}
    </p>
  </div>

  <!-- WIN/LOSE CONDITIONS -->
  <div v-if="isWinning">
    <div>
      <button @click="reload()" class="play-again">Play again?</button>
    </div>
  </div>
  <div v-else-if="isLosing">
    <h2>Sorry! The word was {{ target }}</h2>
    <div>
      <button @click="reload()" class="play-again">Play again?</button>
    </div>
  </div>

  <!-- INPUTS AND WARNINGS -->
  <div v-else>
    <div class="guess-holder">
      <p v-for="letter in 5" :key="letter" class="letters">
        {{ guess[letter - 1] || "" }}
      </p>
    </div>
    <div v-if="!inDictionary">Not in dictionary</div>
    <div v-else-if="isAlreadySubmitted">Word already submitted</div>
  </div>

  <!-- PICTURE -->
  <CatPicture :isWinning="isWinning" :isLosing="isLosing" />

  <!-- KEYBOARD -->
  <SimpleKeyboard
    v-if="!isWinning && !isLosing"
    @onKeyPress="onKeyPress"
    :charScores="charScores"
    id="keyboard"
  />
</template>

<script>
import axios from "axios";
import SimpleKeyboard from "../components/SimpleKeyboard.vue";
import CatPicture from "../components/CatPicture.vue";
export default {
  components: {
    SimpleKeyboard,
    CatPicture,
  },
  data() {
    return {
      target: "",
      guess: "",
      submissions: [],
      inDictionary: true,
      moew: new Audio(
        "https://cdn.freesound.org/previews/412/412016_3652520-lq.mp3"
      ),
      input: "",
      acceptedChars: {
        A: true,
        B: true,
        C: true,
        D: true,
        E: true,
        F: true,
        G: true,
        H: true,
        I: true,
        J: true,
        K: true,
        L: true,
        M: true,
        N: true,
        O: true,
        P: true,
        Q: true,
        R: true,
        S: true,
        T: true,
        U: true,
        V: true,
        W: true,
        X: true,
        Y: true,
        Z: true,
        BACKSPACE: true,
        ENTER: true,
      },
    };
  },
  mounted() {
    this.getWord();
    document.addEventListener("keyup", (event) => {
      this.onKeyPress(event.key);
    });
  },
  watch: {
    isWinning() {
      this.moew.play();
      this.winImage =
        this.winImages[Math.floor(Math.random() * this.winImages.length)];
      this.matchColor(this.winImage);
    },
    isLosing() {
      this.loseImage =
        this.loseImages[Math.floor(Math.random() * this.loseImages.length)];
      this.matchColor(this.loseImage);
    },
  },
  computed: {
    charScores() {
      // creates k,v pairs for letter, score
      // i.e. [c,a,t][0,1,2] => {c: 0, a: 1, t: 2}
      let obj = {};
      const zipLetterWithScore = (letters, scores) =>
        letters.forEach((char, i) => {
          obj[char] = obj[char] > scores[i] ? obj[char] : scores[i]; // only overwrite if higher score
        });
      this.submissions.forEach((submission) =>
        zipLetterWithScore(submission[0], submission[1])
      );
      return obj;
    },
    currentScore() {
      if (Object.keys(this.charScores).length === 0) {
        return 0;
      } else {
        return Object.values(this.charScores).reduce((a, b) => a + b);
      }
    },
    isWinning() {
      return this.currentScore === 10 ? true : false;
    },
    isLosing() {
      return !this.isWinning && this.submissions.length == 6;
    },
    submittedWords() {
      return this.submissions.map((submission) => submission[0].join(""));
    },
    isAlreadySubmitted() {
      return this.submittedWords.includes(this.guess.toUpperCase());
    },
    isPlaying() {
      return this.submittedWords.length > 0;
    },
  },
  methods: {
    getWord() {
      axios
        .get("https://random-word-api.herokuapp.com/word?length=5")
        .then((response) => {
          this.target = response.data[0].toUpperCase();
        })
        .catch((error) => {
          console.log(error);
        });
    },
    onKeyPress(button) {
      button = button.replace("{", "");
      button = button.replace("}", "");
      button = button.toUpperCase();
      if (this.acceptedChars[button]) {
        if (button == "BACKSPACE") {
          this.guess = this.guess.slice(0, -1);
          this.inDictionary = true;
        } else if (button == "ENTER") {
          this.handleGuess(this.guess);
        } else {
          if (this.guess.length == 5) return;
          this.guess += button;
        }
      }
    },
    reload() {
      location.reload();
    },
    matchColor(imgSrc) {
      // Ensures that win/loss cat images match background
      const myImg = new Image();
      myImg.crossOrigin = "Anonymous";
      myImg.src = imgSrc;
      myImg.onload = () => {
        const context = document.createElement("canvas").getContext("2d");
        context.drawImage(myImg, 0, 0);
        const { data } = context.getImageData(10, 10, 1, 1);
        document.body.style.backgroundColor = `rgb(${data[0]}, ${data[1]}, ${data[2]})`;
      };
    },
    checkInDictionary(word) {
      if (word == this.target) return true; // needed to reconcile when target word is not in dictionary API

      let test = axios
        .get("https://api.dictionaryapi.dev/api/v2/entries/en/" + word)
        .then((response) => {
          if (response.status == 200) {
            return true;
          }
          return false;
        })
        .catch(() => {
          return false;
        });
      return test;
    },
    async isGuessAllowed(guess) {
      if (
        this.isWinning ||
        this.isLosing ||
        guess?.length != 5 ||
        this.isAlreadySubmitted
      ) {
        return false;
      }
      this.inDictionary = await this.checkInDictionary(guess);
      if (!this.inDictionary) return false;

      return true;
    },
    calculateScoreArray(word) {
      return word
        .toUpperCase()
        .split("")
        .map((letter, index) => {
          if (this.target[index] == letter) {
            return 2;
          } else if (this.target.includes(letter)) {
            return 1;
          } else {
            return 0;
          }
        });
    },
    async handleGuess(guess) {
      if (!(await this.isGuessAllowed(guess))) return;
      const score = this.calculateScoreArray(guess);
      const submission = [this.guess.split(""), score];
      this.submissions.push(submission);
      this.guess = "";
    },
    setClass(score) {
      if (score == 2) {
        return "correct-spot";
      } else if (score == 1) {
        return "in-word";
      }
    },
  },
};
</script>

<style>
:root {
  --bg-color: #000000;
  --primary-color: #000000;
  --secondary-color: #2f2f2f;
  --font-color-negative: grayscale;
  --font-color-positive: whitesmoke;
  --header-color: whitesmoke;
  --success-bg-color: rgba(107, 217, 107, 0.808);
  --success-font-color: rgb(255, 255, 255);
  --almost-bg-color: rgb(194, 184, 0);
  --almost-font-color: rgb(255, 255, 255);
}

[data-theme="dark"] {
  --primary-color: #000000;
  --secondary-color: #010101;
  --font-color: whitesmoke;
  --bg-color: #000000;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: var(--font-color);
}

@media screen and (max-width: 767px) {
  input {
    font-size: 16px; /* this is the min size to avoid "zoom in " behavior on focus */
  }
}

h1,
h2 {
  color: var(--header-color);
}

.guess-holder {
  justify-content: center;
  flex-direction: row;
  display: flex;
}

body {
  background-color: var(--bg-color);
}

.letters {
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 2px;
  border: solid;
  border-width: 2px;
  border-color: var(--secondary-color);
  color: var(--font-color-positive);
  font-weight: bold;
  width: 50px;
  height: auto;
  aspect-ratio: 1;
  text-transform: capitalize;
  background-color: transparent;
  font-size: 30px;
}

.submitted {
  background-color: var(--secondary-color);
  font-weight: bold;
  border: none;
  padding: 2px;
}

.correct-spot {
  background-color: var(--success-bg-color);
}
.correct-spot .letter {
  color: var(--success-font-color);
}
.in-word {
  background-color: var(--almost-bg-color);
}
.in-word .letter {
  color: var(--almost-font-color);
}

.play-again {
  border: solid;
  border-radius: 5px;
  color: black;
  background-color: whitesmoke;
  font-size: 30px;
  padding: 5px;
  margin: 2px;
  font-family: inherit;
}
#keyboard {
  position: fixed;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
}
</style>
