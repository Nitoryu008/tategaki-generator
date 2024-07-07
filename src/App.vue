<script setup>
import domtoimage from "dom-to-image";
import { ref, computed, nextTick } from "vue";
import "@fontsource-variable/noto-serif-jp";

const text = ref(
  `この
ジェネレータは
縦書きの文章と
画像を
合成できる
縦書きで
あなたを
驚かせたい`
);

const image = ref(null);
const image_url = ref("");
const text_renderer = ref(null);
const padding = ref(10);

const line_height = computed(() => {
  return text_renderer.value.offsetWidth / text.value.split(/\n/).length - 1 + "px";
});

const font_size = computed(() => {
  const lines = text.value.split(/\n/);
  let max_length = 0;
  for (let i = 0; i < lines.length; i++) {
    let line = lines[i];
    if (line.length > max_length) max_length = line.length;
  }
  return text_renderer.value.offsetHeight / max_length - 4 + "px";
});

function setImage() {
  let image_src = image.value.files[0];
  image_url.value = URL.createObjectURL(image_src);
}

function click() {
  image.value.click();
}

function generate() {
  const link = document.createElement("a");
  const result = document.getElementById("result");
  const scale = 1600 / result.clientWidth;
  console.log(result.clientWidth, result.clientHeight);
  domtoimage
    .toPng(result, {
      width: result.clientWidth * scale,
      height: result.clientHeight * scale,
      style: {
        transform: "scale(" + scale + ")",
        transformOrigin: "top left",
      },
    })
    .then(function (dataUrl) {
      link.href = dataUrl;
      link.download = `result.png`;
      link.click();
    });
}
</script>

<template>
  <main class="mt-6 has-text-centered mx-auto">
    <div class="mx-3">
      <h1 class="title">縦書き画像ジェネレータ</h1>
      <br />
      <input
        class="is-hidden"
        type="file"
        name="example"
        accept="image/jpeg, image/png, image/gif, image/webp"
        ref="image"
        @change="setImage"
      />
      <button class="button is-primary mt-5 mb-6" @click="click">
        画像を追加
      </button>
      <textarea
        v-model="text"
        class="textarea is-block mx-auto"
        :rows="text.split(/\n/).length"
      ></textarea>
      <input class="input is-hidden" type="number" v-model="padding">
      <div class="wrapper image mx-auto my-6 is-4by3" ref="wrapper">
        <div class="result has-background-black" id="result">
          <div class="text-wrapper">
            <p
              class="has-text-left has-text-weight-bold has-text-white"
              ref="text_renderer"
            >
              {{ text }}
            </p>
          </div>
          <img :src="image_url" v-if="image_url" />
        </div>
      </div>
      <button class="button is-primary mb-6" @click="generate">
        画像をダウンロード
      </button>
      <p>Developed by Nito(<a href="https://x.com/nito_008">@nito_008</a>)</p>
      <p class="mb-4">Source code on <a href="">GitHub</a></p>
    </div>
  </main>
</template>

<style lang="scss">
textarea {
  width: 100%;
  resize: none;
}

main {
  max-width: 800px;
}

.wrapper {
  max-width: 500px;
  position: relative;
  overflow: hidden;

  p {
    font-family: "Noto Serif JP Variable", serif;
    font-size: v-bind(font_size);
    line-height: v-bind(line_height);
    -ms-writing-mode: tb-rl;
    writing-mode: vertical-rl;
    white-space: pre-wrap;
    position: relative;
    z-index: 1;
    width: 100%;
    height: 100%;
  }

  img {
    position: relative;
    z-index: 0;
    object-fit: cover;
  }
}

.text-wrapper {
  width: 100%;
  height: 100%;
  padding: 20px;
}

.result {
  width: 100%;
  height: 100%;
}

@media screen and (max-width: calc(500px + 12px * 2 - 1px)) {
  .wrapper {
    p {
      font-size: 7vw;
      line-height: 15vw;
    }
  }
}
</style>
