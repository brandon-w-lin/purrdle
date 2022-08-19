<template>
  <div v-if="isWinning || isLosing">
    <img :src="this.image" alt="cat picture" class="win-loss-image" />
  </div>
</template>

<script>
export default {
  name: "CatPicture",
  props: {
    isWinning: {
      type: Boolean,
    },
    isLosing: {
      type: Boolean,
    },
  },
  data: () => ({
    winImages: [
      "https://i.giphy.com/media/IcJ6n6VJNjRNS/giphy.webp",
      "https://i.giphy.com/media/jEl8zlvatJBVS/giphy.webp",
      "https://cutewallpaper.org/24/cat-transparent-gif/cat-gif-transparent.gif",
      "https://acegif.com/wp-content/uploads/gifs/cat-birthday-31.gif",
      "https://i.pinimg.com/originals/e7/4b/38/e74b38cbb1cf9d5ddd4edf15557fedd1.gif",
      "https://acegif.com/wp-content/uploads/gifs/cat-birthday-30.gif",
      "https://cutewallpaper.org/24/cat-transparent-gif/nyan-cat-gif-transparent-nyan-cat-cat-gif-cats.gif",
      "https://cutewallpaper.org/24/cat-transparent-gif/transparent-951c3-cat-e1725-transparent-970c0-gif-c3c48-wifflegif.gif",
      "https://cutewallpaper.org/24/cat-transparent-gif/dancing-cats-gifs-65-funny-animated-images-for-free.gif",
    ],
    loseImages: [
      "https://cliply.co/wp-content/uploads/2021/09/142109670_SAD_CAT_400.gif",
      "https://cutewallpaper.org/24/cat-transparent-gif/transparent-gif-sticker-find-amp-share-on-giphy-grumpy-cat-cats-cat-gif.gif",
      "https://cutewallpaper.org/24/cat-transparent-gif/transparent-transparente-grumpy-cat-gif-on-gifer-by-gavinis.gif",
      "https://media3.giphy.com/media/11FMB3s2TTlPwc/giphy.gif",
    ],
    image: "",
  }),
  // created() {
  //   () => {
  //     if (this.isWinning) {
  //       this.image =
  //         this.winImages[Math.floor(Math.random() * this.winImages.length)];
  //     } else if (this.isLosing) {
  //       this.image =
  //         this.loseImages[Math.floor(Math.random() * this.loseImages.length)];
  //     }
  //   };
  // },
  watch: {
    isWinning() {
      if (!this.isWinning) return;
      this.image =
        this.winImages[Math.floor(Math.random() * this.winImages.length)];
    },
    isLosing() {
      if (!this.isLosing) return;
      this.image =
        this.loseImages[Math.floor(Math.random() * this.loseImages.length)];
    },
  },
};
</script>

<style>
.win-loss-image {
  max-width: 275px;
}
</style>
