<script setup>
import domtoimage from "dom-to-image";
import { ref, onMounted, watch } from "vue";
import "@fontsource-variable/noto-serif-jp";

const text = ref(
  `この
ジェネレータは
縦書きの文章と
画像を
合成できる。
縦書きで
あなたを
驚かせたい。`
);

const image = ref(null);
const image_url = ref("");
const text_renderer = ref(null);
const line_height = ref("20px");
const font_size = ref("20px");
const generated_url = ref("");
const brightness = ref(100);
const text_color = ref("#FFFFFF");
const bg_color = ref("#000000");
const show_detail_setting = ref(false);

function setImage() {
  let image_src = image.value.files[0];
  image_url.value = URL.createObjectURL(image_src);
}

function click() {
  image.value.click();
}

function generate(callback) {
  const result = document.getElementById("result");
  const scale = 1600 / result.clientWidth;
  domtoimage
    .toPng(result, {
      width: result.clientWidth * scale,
      height: result.clientHeight * scale,
      style: {
        transform: "scale(" + scale + ")",
        transformOrigin: "top left",
      },
    })
    .then((dataUrl) => {
      callback(dataUrl);
    });
}

function download() {
  generate((dataUrl) => {
    const link = document.createElement("a");
    link.href = dataUrl;
    link.download = `tategaki.png`;
    link.click();
  });
}

function showImage() {
  generate((dataUrl) => {
    generated_url.value = dataUrl;
  });
}

function render() {
  line_height.value =
    text_renderer.value.offsetWidth / text.value.split(/\n/).length - 1 + "px";

  const lines = text.value.split(/\n/);
  let max_length = 0;
  for (let i = 0; i < lines.length; i++) {
    let line = lines[i];
    if (line.length > max_length) max_length = line.length;
  }

  font_size.value = text_renderer.value.offsetHeight / max_length + "px";
}

function toggleDetailSetting() {
  show_detail_setting.value = !show_detail_setting.value;
}

onMounted(() => {
  render();

  const resizeObserver = new ResizeObserver(() => {
    render();
  });
  if (text_renderer.value) {
    resizeObserver.observe(text_renderer.value);
  }
});

watch(text, () => {
  render();
});
</script>

<template>
  <main class="mt-6 has-text-centered mx-auto">
    <div class="mx-3 is-centered">
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
        @change="render"
        class="textarea is-block mx-auto"
        :rows="text.split(/\n/).length"
      ></textarea>
      <button class="button is-info my-5" @click="toggleDetailSetting">
        詳細設定
      </button>
      <form v-if="show_detail_setting" class="mb-5">
        <div class="field">
          <label class="label">テキストの色</label>
          <div class="control">
            <input class="input" type="color" v-model="text_color" />
          </div>
        </div>
        <div class="field">
          <label class="label">背景の色</label>
          <div class="control">
            <input class="input" type="color" v-model="bg_color" />
          </div>
        </div>
        <div class="field">
          <label class="label">背景画像の明るさ</label>
          <div class="control">
            <input
              class="input"
              type="range"
              min="0"
              max="100"
              step="1"
              v-model="brightness"
            />
          </div>
        </div>
      </form>
      <div class="wrapper image mx-auto mb-6 is-4by3" ref="wrapper">
        <div class="result" id="result">
          <div class="text-wrapper">
            <p class="has-text-left has-text-weight-bold" ref="text_renderer">
              {{ text }}
            </p>
          </div>
          <img :src="image_url" v-if="image_url" />
        </div>
      </div>
      <button class="button is-primary mb-5 is-block mx-auto" @click="download">
        画像をダウンロード
      </button>
      <button
        class="button is-primary mb-6 is-block mx-auto"
        @click="showImage"
      >
        画像を表示
      </button>
      <img :src="generated_url" class="generated" v-if="generated_url" />
      <p class="mb-6">iOSは画像を表示→出てくる画像を長押し→写真に保存</p>
      <p>Developed by Nito(<a href="https://x.com/nito_008">@nito_008</a>)</p>
      <p class="mb-4">
        Source code on
        <a href="https://github.com/Nitoryu008/tategaki-generator">GitHub</a>
      </p>
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
  overflow: hidden;
  position: relative;

  p {
    font-family: "Noto Serif JP Variable", serif;
    font-size: v-bind(font_size);
    line-height: v-bind(line_height);
    -ms-writing-mode: tb-rl;
    writing-mode: vertical-rl;
    text-orientation: upright;
    white-space: pre-wrap;
    position: relative;
    z-index: 1;
    width: 100%;
    height: 100%;
    color: v-bind(text_color);
  }

  img {
    position: relative;
    z-index: 0;
    object-fit: cover;
    filter: brightness(v-bind('brightness + "%"'));
  }
}

.generated {
  max-width: 500px;
  width: 100%;
  object-fit: contain;
}

.text-wrapper {
  width: 100%;
  height: 100%;
  padding: 20px;
}

.result {
  width: 100%;
  height: 100%;
  background-color: v-bind(bg_color);
}
</style>
