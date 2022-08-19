<template>
  <div :class="keyboardClass"></div>
</template>

<script>
import Keyboard from "simple-keyboard";
import "simple-keyboard/build/css/index.css";

export default {
  name: "SimpleKeyboard",
  emits: ["onChange", "onKeyPress"],
  props: {
    keyboardClass: {
      default: "simple-keyboard",
      type: String,
    },
    charScores: {
      type: Object,
    },
  },
  data: () => ({
    keyboard: null,
    layout: {
      default: [
        "Q W E R T Y U I O P",
        "A S D F G H J K L",
        "{enter} Z X C V B N M {backspace}",
      ],
    },
    display: {
      "{enter}": "enter",
      "{backspace}": "⌫",
    },
    theme: "hg-theme-default hg-theme-ios",
  }),
  mounted() {
    this.keyboard = new Keyboard(this.keyboardClass, {
      onKeyPress: this.onKeyPress,
      layout: this.layout,
      display: this.display,
      theme: this.theme,
    });
    document.addEventListener("keydown", (e) => this.handleKeyDown(e));
    document.addEventListener("keyup", (e) => this.handleKeyUp(e));
  },
  watch: {
    charScores() {
      this.handleScoreChange();
    },
  },
  methods: {
    onKeyPress(button) {
      this.$emit("onKeyPress", button);
    },
    handleKeyDown(event) {
      // To handle the physical keyboard press -> format on screen display
      const key =
        event.key.length == 1 ? event.key.toUpperCase() : `{${event.key}}`;
      this.keyboard.addButtonTheme(key, "active");
    },
    handleKeyUp(event) {
      // To handle the physical keyboard press -> format on screen display
      const key =
        event.key.length == 1 ? event.key.toUpperCase() : `{${event.key}}`;
      this.keyboard.removeButtonTheme(key, "active");
    },
    handleScoreChange() {
      // To handle formatting of keys after checking vs. target word
      for (const key in this.charScores) {
        if (this.charScores[key] == 2) {
          this.keyboard.removeButtonTheme(key, "in-word");
          this.keyboard.addButtonTheme(key, "correct-spot");
        } else if (this.charScores[key] == 1) {
          this.keyboard.addButtonTheme(key, "in-word");
        } else {
          this.keyboard.addButtonTheme(key, "submitted");
        }
      }
    },
  },
};
</script>

<style>
.hg-button {
  width: 10px;
}
.simple-keyboard.hg-theme-ios {
  max-width: 750px;
  margin: 0 auto;
}
.simple-keyboard.hg-theme-ios.hg-theme-default .hg-row .hg-button {
  flex-grow: 1;
  cursor: pointer;
  max-width: initial;
}
.simple-keyboard.hg-theme-ios .hg-row {
  display: flex;
}
.simple-keyboard.hg-theme-ios .hg-row:not(:last-child) {
  margin-bottom: 5px;
}
.simple-keyboard.hg-theme-ios .hg-row .hg-button:not(:last-child) {
  margin-right: 5px;
}

.simple-keyboard.hg-theme-ios .hg-row:nth-child(2) {
  margin-left: 5%;
  margin-right: 5%;
}
.simple-keyboard.hg-theme-ios.hg-theme-default {
  background-color: rgba(0, 0, 0, 0.1);
  padding: 5px;
  border-radius: 5px;
}

.simple-keyboard.hg-theme-ios.hg-theme-default .hg-button {
  border-radius: 5px;
  box-sizing: border-box;
  padding: 0;
  color: var(--font-color);
  background: var(--primary-color);
  border-bottom: none;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: none;
  font-weight: 400;
  font-size: 20px;
  max-width: 60px;
  min-width: 20px;
  height: 60px;
  min-height: 60px;
}

.simple-keyboard.hg-theme-ios.hg-theme-default .hg-button.correct-spot {
  background-color: var(--success-bg-color);
  /* background-color: rgb(107, 217, 107); */
}
.simple-keyboard.hg-theme-ios.hg-theme-default .hg-button.in-word {
  background-color: var(--almost-bg-color);
}

.simple-keyboard.hg-theme-ios.hg-theme-default .hg-button:active,
.simple-keyboard.hg-theme-ios.hg-theme-default .hg-button.active,
.simple-keyboard.hg-theme-ios.hg-theme-default .hg-button:focus {
  background: var(--secondary-color);
}
.simple-keyboard.hg-theme-ios.hg-theme-default .hg-button.hg-functionBtn {
  background-color: var(--secondary-color);
}

.simple-keyboard.hg-theme-ios.hg-theme-default .hg-button-backspace {
  min-width: 50px;
  padding: 0% 4%;
}
.simple-keyboard.hg-theme-ios.hg-theme-default .hg-button-enter {
  min-width: 50px;
  padding: 0% 4%;
}
.simple-keyboard.hg-theme-ios.hg-theme-default .hg-button-altright,
.simple-keyboard.hg-theme-ios.hg-theme-default .hg-button-back {
  min-width: 80px;
  max-width: 80px;
}
.simple-keyboard.hg-theme-ios.hg-theme-default .hg-button.submitted {
  background: var(--secondary-color);
}
</style>
