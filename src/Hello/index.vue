<script lang="ts" setup>
import { ref } from "vue";

defineProps({
  enterAction: {
    type: Object,
    required: true,
  },
});
const targetColor = ref("#ffffff"); // 默认目标颜色
const resultImage = ref<string | null>(null);
const canvasRef = ref<HTMLCanvasElement | null>(null);
const ctx = ref<CanvasRenderingContext2D | null>(null);

const onFileChange = (event: Event) => {
  const file = (event.target as HTMLInputElement).files?.[0];
  if (file) {
    const reader = new FileReader();
    reader.onload = (e) => {
      const image = new Image();
      image.src = e.target?.result as string;
      image.onload = () => {
        if (canvasRef.value) {
          canvasRef.value.width = image.width;
          canvasRef.value.height = image.height;
          ctx.value = canvasRef.value.getContext("2d");
          ctx.value?.drawImage(image, 0, 0);
          resultImage.value = canvasRef.value.toDataURL(); // 初始化结果图片
        }
      };
    };
    reader.readAsDataURL(file);
  }
};
const retImg = ref(null)
const replaceColor = () => {
  if (ctx.value && canvasRef.value) {
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
      const tolerance = 50;
      if (isBackgroundColor(r, g, b, targetRGB, tolerance)) {
        data[i] = 255; // 替换R通道
        data[i + 1] = 0; // 替换G通道
        data[i + 2] = 0; // 替换B通道
      }
    }

    ctx.value.putImageData(imageData, 0, 0);
    // resultImage.value = canvasRef.value.toDataURL(); // 更新结果图像
    retImg.value = canvasRef.value.toDataURL(); // 更新结果图像
  }
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
</script>
<!-- 
<template>
  <div class="hello">
    <h1>你好，Hello</h1>
    <h2>插件应用进入参数</h2>
    <pre>
        {{ JSON.stringify(enterAction, undefined, 2) }}
      </pre>
  </div>
</template> -->
<template>
  <div>
    <h1>证件照底色替换</h1>
    <input type="file" @change="onFileChange" accept="image/*" />
    <input type="color" v-model="targetColor" />
    <button @click="replaceColor">替换底色</button>
    <canvas ref="canvasRef" style="display: none"></canvas>
    <img :src="resultImage" alt="处理后的证件照" />
    <img :src="retImg" alt="处理后的证件照" />
  </div>
</template>

<style>
.hello {
  padding: 10px 28px;
}
</style>
