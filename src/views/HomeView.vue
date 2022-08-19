<template>
  <!-- WELCOME -->
  <div v-if="!isPlaying">
    <h1>Purrdle</h1>
    <h2>Like wordle, but it purrs</h2>
  </div>

  <div>
    Computed Properties:
    <p>currentScore: {{ currentScore }}</p>
    <p>isWinning: {{ isWinning }}</p>
    <p>isLosing: {{ isLosing }}</p>
    <p>submittedWords: {{ submittedWords }}</p>
    <p>isAlreadySubmitted: {{ isAlreadySubmitted }}</p>
  </div>
  <!-- HOLD SUBMITTED GUESSES -->
  <div class="guess-holder" v-for="submission in submissions" :key="submission">
    <p
      class="letters"
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
      <button @click="reload()">Play again?</button>
    </div>
    <div>
      <img :src="winImage" alt="happy cat" />
    </div>
  </div>
  <div v-else-if="isLosing">
    <h2>Sorry! The word was {{ target }}</h2>
    <div>
      <button @click="reload()">Play again?</button>
    </div>
    <img :src="loseImage" alt="sad cat" />
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
export default {
  components: {
    SimpleKeyboard,
  },
  data() {
    return {
      // Needed at top level
      target: "",
      guess: "",
      letters: [],
      submissions: [],
      inDictionary: true,
      winImages: [
        "https://i.giphy.com/media/LRHBzifRrWQky9zAO7/giphy.webp",
        "https://i.giphy.com/media/T2zhJop5K1joHXd3DJ/giphy.webp",
        "https://i.giphy.com/media/IcJ6n6VJNjRNS/giphy.webp",
        "https://i.giphy.com/media/Jmlv4CUO1qWlGWJf3J/giphy.webp",
        "https://i.giphy.com/media/H3NF3JvE1mOsOXb8l3/giphy.webp",
        "https://i.giphy.com/media/jEl8zlvatJBVS/giphy.webp",
        "https://i.giphy.com/media/AwroDzTSZ2SDRjbGll/giphy.webp",
        "https://i.giphy.com/media/VP62gZkQXtTgfFUa1Z/giphy.webp",
        "https://i.giphy.com/media/jU2IAEtFVGBDMu2vWg/giphy.webp",
        "https://i.giphy.com/media/hGFiFeUFnvUPjW8lEZ/giphy.webp",
        "https://i.giphy.com/media/2YgW9PZJpIhjL7b6Zt/giphy.webp",
        "https://i.giphy.com/media/117IVXpuqIITx6/giphy.webp",
      ],
      winImage: "",
      loseImages: [
        "https://i.giphy.com/media/BCI6CWVkNUefm/giphy.webp",
        "https://i.giphy.com/media/IkwD0hpaqza8MM5e8J/giphy.webp",
        "https://i.giphy.com/media/4L7Q2eAKjd2wM/giphy.webp",
      ],
      loseImage: "",
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
  methods: {
    onKeyPress(button) {
      button = button.replace("{", "");
      button = button.replace("}", "");
      button = button.toUpperCase();
      if (this.acceptedChars[button]) {
        console.log("allowed");
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
    async handleGuess(guess) {
      // Guard gates for win condition, length, in dictionary

      if (
        this.isWinning ||
        this.isLosing ||
        guess?.length != 5 ||
        this.isAlreadySubmitted
      ) {
        return;
      }
      this.inDictionary = await this.checkInDictionary(guess);
      if (!this.inDictionary) return;

      // if (this.isLosing) return;
      // if (guess?.length != 5) return;
      // if (!this.inDictionary) return;
      // if (this.isAlreadySubmitted) return;
      guess = guess?.toUpperCase();
      this.letters[0] = guess[0] || "";
      this.letters[1] = guess[1] || "";
      this.letters[2] = guess[2] || "";
      this.letters[3] = guess[3] || "";
      this.letters[4] = guess[4] || "";

      // Check for matches
      let score = this.letters.map((letter, index) => {
        if (this.target[index] == letter) {
          return 2;
        } else if (this.target.includes(letter)) {
          return 1;
        } else {
          return 0;
        }
      });

      // Create formatted submission with word and score
      let submission = [this.letters.map((letter) => letter), score];
      this.submissions.push(submission);

      // Handle win condition
      // this.handleWinLoss();
      // Clear out the guess
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
  mounted() {
    this.getWord();
    document.addEventListener("keyup", (event) => {
      console.log(event.key);
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
      // converts format from (word)(wordscore) format,
      // to {letter: score} format, i.e.
      // target = act
      // [c,a,t][0,1,2] => {c: 0, a: 1, t: 2}
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
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

@media screen and (max-width: 767px) {
  input {
    font-size: 16px; /* this is the min size to avoid "zoom in " behavior on focus */
  }
}

.guess-holder {
  justify-content: center;
  flex-direction: row;
  display: flex;
}

.letters {
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 2px;
  border: solid;
  width: 50px;
  height: auto;
  aspect-ratio: 1;
  text-transform: capitalize;
  background-color: whitesmoke;
  font-size: 30px;
}

.correct-spot {
  background-color: rgb(107, 217, 107);
}
.in-word {
  background-color: yellow;
}

#keyboard {
  position: fixed;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
}
</style>
