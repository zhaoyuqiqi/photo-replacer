<script setup lang="ts">
import { computed, onMounted, ref, useTemplateRef, watch } from "vue";

const canvasRef = useTemplateRef("canvas");
const ctx = ref<CanvasRenderingContext2D>();

onMounted(() => {
  ctx.value = canvasRef.value.getContext("2d");
});
const onDrop = (e: DragEvent) => {
  const file = e.dataTransfer.files[0];
  loadImage(file);
};
const currentImage = ref<HTMLImageElement>();
const hasSelectedImage = computed(() => !!currentImage.value);
const originImageSrc = ref("");

const loadImage = (file: File) => {
  const reader = new FileReader();
  reader.onload = (e) => {
    const image = new Image();
    image.onload = () => {
      canvasRef.value.width = image.width;
      canvasRef.value.height = image.height;
      currentImage.value = image;
      ctx.value.drawImage(image, 0, 0);
    };
    image.onerror = () => {
      currentImage.value = undefined;
      originImageSrc.value = "";
    };
    originImageSrc.value = e.target.result.toString();
    image.src = e.target.result.toString();
  };
  reader.onerror = () => {
    window.alert("图片加载失败");
  };
  reader.readAsDataURL(file);
};
const fileInput = useTemplateRef("uploadFile");
const onUploadFile = () => fileInput.value.click();
const onFileChange = (e: Event) => {
  const file = (e.target as HTMLInputElement).files[0];
  currentImage.value = undefined;
  originImageSrc.value = "";
  loadImage(file);
};
const clear = () => {
  currentImage.value = undefined;
  originImageSrc.value = "";
  fileInput.value.value = "";
};
const currentColor = ref("#ff0000");
const distance = ref(10);
const targetColor = ref("#ff0000");

const onDistanceChange = (e: InputEvent) => {
  distance.value = Math.min(
    Number((e.target as HTMLInputElement).value.replace(/\D/g, "")),
    255
  );
};

const hexToRgb = (hex: string) => {
  const bigint = parseInt(hex.slice(1), 16);
  return {
    r: (bigint >> 16) & 255,
    g: (bigint >> 8) & 255,
    b: bigint & 255,
  };
};

const isBackgroundColor = (
  r: number,
  g: number,
  b: number,
  targetRGB: { r: number; g: number; b: number },
  tolerance: number
) => {
  // 判断颜色是否在目标颜色的容差范围内
  return (
    Math.abs(r - targetRGB.r) < tolerance &&
    Math.abs(g - targetRGB.g) < tolerance &&
    Math.abs(b - targetRGB.b) < tolerance
  );
};
const replaceColor = () => {
  if (ctx.value && canvasRef.value) {
    ctx.value.drawImage(currentImage.value, 0, 0);
    const currentRGB = hexToRgb(currentColor.value);
    const targetRGB = hexToRgb(targetColor.value);
    const imageData = ctx.value.getImageData(
      0,
      0,
      canvasRef.value.width,
      canvasRef.value.height
    );
    const data = imageData.data;

    for (let i = 0; i < data.length; i += 4) {
      const r = data[i];
      const g = data[i + 1];
      const b = data[i + 2];

      // 这里的容差可以调整以匹配更复杂的颜色识别
      if (isBackgroundColor(r, g, b, currentRGB, distance.value)) {
        data[i] = targetRGB.r; // 替换R通道
        data[i + 1] = targetRGB.g; // 替换G通道
        data[i + 2] = targetRGB.b; // 替换B通道
      }
    }

    ctx.value.putImageData(imageData, 0, 0);
    // resultImage.value = canvasRef.value.toDataURL(); // 更新结果图像
  }
};

watch([distance, currentColor, targetColor], replaceColor);

const download = () => {
  const url = canvasRef.value?.toDataURL?.();
  if (url) {
    window.services.writeImageFile(url);
    window.alert('保存成功！')
  }
};
</script>

<template>
  <div class="container">
    <h1>证件照替换工具</h1>
    <div
      v-show="!hasSelectedImage"
      @dragover.stop.prevent
      @drop.stop.prevent="onDrop"
      class="uploader"
    >
      <input
        ref="uploadFile"
        type="file"
        id="fileInput"
        accept="image/*"
        style="display: none"
        @change="onFileChange"
      />
      <button id="uploadButton" @click="onUploadFile">上传图片</button>
      <p>或将图片拖放到这里</p>
    </div>
    <div class="image-editor" v-show="hasSelectedImage">
      <div class="content">
        <div class="wrapper">
          原始图像
          <img class="img" :src="originImageSrc" />
        </div>
        <div class="line" />
        <div class="wrapper">
          替换后图像
          <canvas ref="canvas" class="canvas"></canvas>
        </div>
      </div>
      <div class="controls">
        <div class="color-container">
          <label>
            待替换颜色
            <input class="color" type="color" v-model="currentColor" />
          </label>
          <label>
            容差
            <input
              type="number"
              class="no-spin"
              step="1"
              max="255"
              min="0"
              @input="onDistanceChange"
              :value="distance"
            />
          </label>
          <label>
            目标颜色
            <input class="color" type="color" v-model="targetColor" />
          </label>
        </div>
        <button @click="replaceColor">替换</button>
        <button @click="clear">清空图片</button>
        <button @click="download">下载图片</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
label {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.color-container {
  width: 150px;
  display: flex;
  flex-direction: column;
  gap: 4px;
}
.color {
  border: 1px solid #4caf50;
  font-size: 16px;
  width: 72px;
  height: 40px;
  border-radius: 4px;
}
.no-spin {
  -moz-appearance: textfield; /* Firefox */
  -webkit-appearance: none; /* Safari 和 Chrome */
  appearance: none; /* 现代浏览器 */
  border: 1px solid #4caf50;
  font-size: 16px;
  padding: 10px 20px;
  width: 30px;
  border-radius: 4px;
  outline: none;
}

.no-spin::-webkit-inner-spin-button,
.no-spin::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0; /* Safari 和 Chrome */
}
body {
  font-family: Arial, sans-serif;
  line-height: 1.6;
  margin: 0;
  padding: 20px;
  background-color: #f4f4f4;
}

.container {
  max-width: 800px;
  margin: 0 auto;
  background-color: #fff;
  padding: 20px;
  border-radius: 5px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

h1 {
  text-align: center;
  color: #333;
}

.uploader {
  border: 2px dashed #ccc;
  padding: 20px;
  text-align: center;
  margin-bottom: 20px;
  cursor: pointer;
}

.uploader:hover {
  border-color: #aaa;
}
.image-editor {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.content {
  flex: 1;
  width: 100%;
  display: flex;
  margin-bottom: 20px;
  border: 1px solid #4caf50;
  box-sizing: border-box;
}
.wrapper {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.line {
  width: 1px;
  align-self: stretch;
  background-color: #4caf50;
}
.img {
  max-width: 48%;
  height: auto;
  margin: 0 auto;
  position: relative;
}

.canvas {
  max-width: 48%;
  height: auto;
  margin: 0 auto;
}

.controls {
  align-self: stretch;
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
}

button {
  background-color: #4caf50;
  border: none;
  color: white;
  padding: 10px 20px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin: 4px 2px;
  cursor: pointer;
  border-radius: 4px;
}

button:hover {
  background-color: #45a049;
}

#colorPicker {
  width: 50px;
  height: 50px;
  padding: 0;
  border: none;
  cursor: pointer;
}
</style>
