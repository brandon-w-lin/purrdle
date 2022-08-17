<template>
  <div :class="keyboardClass"></div>
</template>

<script>
import Keyboard from "simple-keyboard";
import "simple-keyboard/build/css/index.css";

export default {
  name: "SimpleKeyboard",
  props: {
    keyboardClass: {
      default: "simple-keyboard",
      type: String,
    },
    input: {
      type: String,
    },
  },
  data: () => ({
    keyboard: null,
    layout: {
      default: [
        "Q W E R T Y U I O P",
        "A S D F G H J K L",
        "{submit} Z X C V B N M {backspace}",
      ],
    },
    display: {
      "{submit}": "in",
      "{backspace}": "⌫",
    },
    theme: "hg-theme-default hg-theme-ios",
  }),
  mounted() {
    this.keyboard = new Keyboard(this.keyboardClass, {
      onChange: this.onChange,
      onKeyPress: this.onKeyPress,
      layout: this.layout,
      display: this.display,
      theme: this.theme,
    });
  },
  methods: {
    onChange(input) {
      this.$emit("onChange", input);
    },
    onKeyPress(button) {
      this.$emit("onKeyPress", button);

      /**
       * If you want to handle the shift and caps lock buttons
       */
      if (button === "{shift}" || button === "{lock}") this.handleShift();
    },
    handleShift() {
      let currentLayout = this.keyboard.options.layoutName;
      let shiftToggle = currentLayout === "default" ? "shift" : "default";

      this.keyboard.setOptions({
        layoutName: shiftToggle,
      });
    },
  },
  watch: {
    input(input) {
      this.keyboard.setInput(input);
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
/**
 * hg-theme-default theme
 */

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
.simple-keyboard.hg-theme-ios .hg-row:last-child .hg-button:last-child {
  /* border: solid; */
  padding: 0% 4%;
}
.simple-keyboard.hg-theme-ios .hg-row:last-child .hg-button:first-child {
  /* border: solid; */
  padding: 0% 4%;
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
.simple-keyboard.hg-theme-ios.hg-theme-default.hg-layout-custom {
  background-color: #e5e5e5;
  padding: 5px;
}
.simple-keyboard.hg-theme-ios.hg-theme-default .hg-button {
  border-radius: 5px;
  box-sizing: border-box;
  padding: 0;
  background: rgb(249, 249, 249);
  border-bottom: 1px solid #b5b5b5;
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
.simple-keyboard.hg-theme-ios.hg-theme-default .hg-button:active,
.simple-keyboard.hg-theme-ios.hg-theme-default .hg-button:focus {
  background: #e4e4e4;
}
.simple-keyboard.hg-theme-ios.hg-theme-default .hg-button.hg-functionBtn {
  background-color: #adb5bb;
}
.simple-keyboard.hg-theme-ios.hg-theme-default .hg-button.hg-button-space,
.simple-keyboard.hg-theme-ios.hg-theme-default .hg-button.hg-button-shift,
.simple-keyboard.hg-theme-ios.hg-theme-default
  .hg-button.hg-button-shiftactivated {
  background-color: #ffffff;
}
.simple-keyboard.hg-theme-ios.hg-theme-default .hg-button-space {
  max-width: 448px;
  min-width: 448px;
}
.simple-keyboard.hg-theme-ios.hg-theme-default .hg-button-enter {
  max-width: 110px;
  min-width: 110px;
}
.simple-keyboard.hg-theme-ios.hg-theme-default .hg-button-altright,
.simple-keyboard.hg-theme-ios.hg-theme-default .hg-button-back {
  min-width: 80px;
  max-width: 80px;
}
</style>
