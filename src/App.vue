<template @click="setFocus()">
  <h1>Purrdle</h1>
  <h2>Like wordle, but it purrs</h2>
  <div class="guess-holder" v-for="submission in submissions" :key="submission">
    <div v-for="(letter, index) in submission[0]" :key="index">
      <p class="letters" :class="setClass(submission[1][index])">
        {{ letter }}
      </p>
    </div>
  </div>
  <div v-if="winning">
    <div>
      <button @click="reload()">Play again?</button>
    </div>
    <div>
      <img :src="winImage" alt="happy cat" />
    </div>
  </div>
  <div v-else-if="losing">
    <h2>Sorry! The word was {{ target }}</h2>
    <div>
      <button @click="location.reload()">Play again?</button>
    </div>
    <img :src="loseImage" alt="sad cat" />
  </div>
  <div v-else>
    <div class="guess-holder">
      <p class="letters">{{ guess[0] || "" }}</p>
      <p class="letters">{{ guess[1] || "" }}</p>
      <p class="letters">{{ guess[2] || "" }}</p>
      <p class="letters">{{ guess[3] || "" }}</p>
      <p class="letters">{{ guess[4] || "" }}</p>
    </div>
    <div v-if="!inDictionary">Not in dictionary</div>
    <div v-else-if="alreadySubmitted">Word already submitted</div>
  </div>
  <input
    type="text"
    id="guessInput"
    v-model="guess"
    maxlength="5"
    v-on:keyup.enter="handleGuess(guess)"
    @input="
      if (guess?.length < 5) {
        inDictionary = true;
        alreadySubmitted = false;
      }
    "
  />
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      target: "",
      guess: "",
      letters: [],
      submissions: [],
      winning: false,
      losing: false,
      inDictionary: true,
      alreadySubmitted: false,
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
    };
  },
  methods: {
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
    handleWinLoss() {
      let score = this.submissions
        .slice(-1)
        .pop()[1]
        .reduce((a, b) => a + b);
      if (score == 10) {
        this.winning = true;
        this.winImage =
          this.winImages[Math.floor(Math.random() * this.winImages.length)];
        this.matchColor(this.winImage);
      } else if (this.submissions.length == 6) {
        this.losing = true;
        this.loseImage =
          this.loseImages[Math.floor(Math.random() * this.loseImages.length)];
        this.matchColor(this.loseImage);
      }
    },
    checkInDictionary(word) {
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
    isAlreadySubmitted(word) {
      const words = this.submissions.map((submission) =>
        submission[0].join("")
      );
      this.alreadySubmitted = true;
      return words.includes(word.toUpperCase());
    },
    async handleGuess(guess) {
      // Guard gates for win condition, length, in dictionary
      if (this.winning) return;
      if (this.losing) return;
      if (guess?.length != 5) return;
      this.inDictionary = await this.checkInDictionary(guess);
      if (!this.inDictionary) return;
      if (this.isAlreadySubmitted(guess)) return;
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
      this.handleWinLoss();
      // Clear out the v-model guess
      this.guess = "";
      this.alreadySubmitted = false;
    },
    setClass(score) {
      if (score == 2) {
        return "correct-spot";
      } else if (score == 1) {
        return "in-word";
      }
    },
    setFocus() {
      {
        document.getElementById("guessInput").focus();
      }
    },
  },
  mounted() {
    this.setFocus();
    this.getWord();
    document.addEventListener("click", () => {
      this.setFocus();
    });
    document.addEventListener("touchstart", () => {
      this.setFocus();
    });
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

nav {
  padding: 30px;
}

nav a {
  font-weight: bold;
  color: #2c3e50;
}

nav a.router-link-exact-active {
  color: #42b983;
}

@media screen and (max-width: 767px) {
  input {
    font-size: 16px;
  }
}

#guessInput {
  position: absolute;
  top: -50px;
  left: -50px;
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
  height: 50px;
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
</style>
