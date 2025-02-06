<script setup lang="ts">
/**
 * @author zhaoyuqi
 * @Date 2025-02-06 20:39:26"
 * @description
 */
const onDrop = (e: DragEvent) => {
  const file = e.dataTransfer.files[0];
  console.log(file);
};
</script>

<template>
  <div class="container">
    <h1>证件照替换工具</h1>
    <div @dragover.stop.prevent @drop.stop.prevent="onDrop" class="uploader">
      <input
        type="file"
        id="fileInput"
        accept="image/*"
        style="display: none"
      />
      <button id="uploadButton">上传图片</button>
      <p>或将图片拖放到这里</p>
    </div>
    <div id="imageEditor" style="display: none">
      <canvas id="canvas"></canvas>
      <div class="controls">
        <input type="color" id="colorPicker" value="#ff0000" />
        <button id="replaceButton">替换选中区域</button>
        <button id="downloadButton">下载图片</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
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

canvas {
  max-width: 100%;
  margin-bottom: 20px;
}

.controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
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
