<template>
  <div id="page" @click="setFocus()">
    <!-- <nav>
    <router-link to="/">Home</router-link> |
    <router-link to="/about">About</router-link>
  </nav>
  <router-view /> -->
    <h1>Purrdle</h1>
    <h2>Like wordle, but it purrs</h2>
    <div class="guess-holder">
      <p class="letters">{{ guess[0] }}</p>
      <p class="letters">{{ guess[1] }}</p>
      <p class="letters">{{ guess[2] }}</p>
      <p class="letters">{{ guess[3] }}</p>
      <p class="letters">{{ guess[4] }}</p>
    </div>
    <!-- {{ submissions }} -->
    <div
      class="guess-holder"
      v-for="submission in submissions"
      :key="submission"
    >
      <div v-for="(letter, index) in submission[0]" :key="index">
        <p class="letters" :class="setClass(submission[1][index])">
          {{ letter }}
        </p>
      </div>
      <!-- <p class="letters submitted">{{ letter[index] }}</p> -->
    </div>
    <input
      type="text"
      id="guessInput"
      v-model="guess"
      maxlength="5"
      v-on:keyup.enter="handleGuess(guess)"
    />
    <div v-if="winning">
      <div>Congrats, you win!</div>
      <div>Play again?</div>
    </div>
    <div v-else-if="losing">Sorry! The word was {{ target }}</div>
    <div v-else-if="!inDictionary">Not in dictionary</div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  data() {
    return {
      target: "TABLE",
      guess: "",
      letters: [],
      submissions: [],
      winning: false,
      losing: false,
      inDictionary: true,
    };
  },
  methods: {
    handleWin() {
      let score = this.submissions
        .slice(-1)
        .pop()[1]
        .reduce((a, b) => a + b);
      if (score == 10) {
        console.log("win");
        this.winning = true;
      }
      if (this.submissions.length == 6) {
        console.log("lose");
        this.losing = true;
      }
    },
    checkInDictionary(word) {
      console.log("checking if in dictionary");
      let test = axios
        .get("https://api.dictionaryapi.dev/api/v2/entries/en/" + word)
        .then((response) => {
          // console.log(response.status);
          if (response.status == 200) {
            return true;
          }
          return false;
        })
        .catch(() => {
          // console.log("not in dictionary");
          return false;
        });
      return test;
    },
    async handleGuess(guess) {
      // Guard gates for win condition, length, in dictionary
      if (this.winning) return;
      if (this.losing) return;
      if (guess.length != 5) return;
      this.inDictionary = await this.checkInDictionary(guess);
      if (!this.inDictionary) {
        this.guess = "";
        return;
      }

      guess = guess.toUpperCase();
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

      // Clear out the v-model guess
      this.guess = "";

      // Handle win condition
      this.handleWin();
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

#page {
  height: 100vh;
  width: 100vw;
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
  width: 30px;
  height: 30px;
  text-transform: capitalize;
}

.correct-spot {
  background-color: rgb(107, 217, 107);
}
.in-word {
  background-color: yellow;
}
</style>
