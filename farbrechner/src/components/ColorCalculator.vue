<template>
  <div class="color-calculator-container">
    <div class="canvas-container">
      <canvas id="shade-picker" :width="shadePicker.width" :height="shadePicker.height" @mousedown="mousedown" @mouseup="mouseup" @mousemove="mousemove"></canvas>
      <canvas id="color-picker" :width="colorPicker.width" :height="colorPicker.height" @click="click"></canvas>
    </div>
    <div class="color-blocks-container">
      <div class="color-block">
        <div :style="{ backgroundColor: firstPickedColor }" @click="getActiveBlock(1)"></div>
        <span> {{ rgbToHex(firstPickedColor) }}</span>
      </div>
      <div class="color-block">
        <div :style="{ backgroundColor: secondPickedColor }" @click="getActiveBlock(2)"></div>
        <span>{{ rgbToHex(secondPickedColor) }}</span>
      </div>
    </div>
    <div class="mixed-colors-container">
      <div :style="{ backgroundColor: mixColors(firstPickedColor, secondPickedColor) }"></div>
      <p>{{ rgbToHex(mixColors(firstPickedColor, secondPickedColor)) }}</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      shadePicker: {
        ctx: null,
        width: 200,
        height: 200,
        rgbaColor: "rgba(255,0,0,1)",
      },
      colorPicker: {
        ctx: null,
        width: 40,
        height: 200,
        rgbaColor: "rgba(255,0,0,1)",
      },
      firstPickedColor: "rgba(255,0,0,1)",
      secondPickedColor: "rgba(255,0,0,1)",
      rgbaColor: "rgba(255,0,0,1)",
      x: 0,
      y: 0,
      drag: false,
      chooseColor: 1,
    };
  },
  methods: {
    initContext() {
      this.shadePicker.ctx = document.getElementById("shade-picker").getContext("2d");

      this.colorPicker.ctx = document.getElementById("color-picker").getContext("2d");
    },

    createRect(picker) {
      picker.ctx.rect(0, 0, picker.width, picker.height);
    },

    fillGradient() {
      this.shadePicker.ctx.fillStyle = this.rgbaColor;
      this.shadePicker.ctx.fillRect(0, 0, this.shadePicker.width, this.shadePicker.height);

      const grdWhite = this.colorPicker.ctx.createLinearGradient(0, 0, this.shadePicker.width, 0);
      grdWhite.addColorStop(0, "rgba(255,255,255,1)");
      grdWhite.addColorStop(1, "rgba(255,255,255,0)");
      this.shadePicker.ctx.fillStyle = grdWhite;
      this.shadePicker.ctx.fillRect(0, 0, this.shadePicker.width, this.shadePicker.height);

      const grdBlack = this.colorPicker.ctx.createLinearGradient(0, 0, 0, this.shadePicker.height);
      grdBlack.addColorStop(0, "rgba(0,0,0,0)");
      grdBlack.addColorStop(1, "rgba(0,0,0,1)");
      this.shadePicker.ctx.fillStyle = grdBlack;
      this.shadePicker.ctx.fillRect(0, 0, this.shadePicker.width, this.shadePicker.height);
    },

    createColorGradient() {
      const gradient = this.colorPicker.ctx.createLinearGradient(0, 0, 0, this.shadePicker.height);
      gradient.addColorStop(0, "rgba(255, 0, 0, 1)");
      gradient.addColorStop(0.17, "rgba(255, 255, 0, 1)");
      gradient.addColorStop(0.34, "rgba(0, 255, 0, 1)");
      gradient.addColorStop(0.51, "rgba(0, 255, 255, 1)");
      gradient.addColorStop(0.68, "rgba(0, 0, 255, 1)");
      gradient.addColorStop(0.85, "rgba(255, 0, 255, 1)");
      gradient.addColorStop(1, "rgba(255, 0, 0, 1)");
      this.colorPicker.ctx.fillStyle = gradient;
      this.colorPicker.ctx.fill();
    },

    getColorPicker() {
      this.createRect(this.shadePicker);
      this.createRect(this.colorPicker);
      this.fillGradient();
      this.createColorGradient();
    },

    click(e) {
      this.x = e.offsetX;
      this.y = e.offsetY;

      const imageData = this.colorPicker.ctx.getImageData(this.x, this.y, 1, 1).data;
      this.rgbaColor = `rgba(${imageData[0]},${imageData[1]},${imageData[2]},1)`;
      this.fillGradient();
    },

    mouseup() {
      this.drag = false;
    },

    mousedown(e) {
      this.drag = true;
      this.changeColor(e);
    },

    mousemove(e) {
      if (this.drag) {
        this.changeColor(e);
      }
    },

    changeColor(e) {
      this.x = e.offsetX;
      this.y = e.offsetY;
      const imageData = this.shadePicker.ctx.getImageData(this.x, this.y, 1, 1).data;
      this.rgbaColor = `rgba(${imageData[0]},${imageData[1]},${imageData[2]},1)`;
      
      if (this.chooseColor == 1) {
        this.firstPickedColor = this.rgbaColor;
      } else {
        this.secondPickedColor = this.rgbaColor;
      }
      console.log(this.rgbaColor);
      this.rgbToHex(this.rgbaColor)
    },

    getActiveBlock(blockNum) {
      this.chooseColor = blockNum;
    },

    rgbToHex(color) {
      const rgba = color.replace(/^rgba?\(|\s+|\)$/g, "").split(",");
      const hex = `#${((1 << 24) + (parseInt(rgba[0]) << 16) + (parseInt(rgba[1]) << 8) + parseInt(rgba[2])).toString(16).slice(1)}`;
      return hex;
    },

    mixColors(firstColor, secondColor) {
      const firstRgba = firstColor.replace(/^rgba?\(|\s+|\)$/g, "").split(",").map(Number);
      const secondRgba = secondColor.replace(/^rgba?\(|\s+|\)$/g, "").split(",").map(Number);

      const r = (firstRgba[0] + secondRgba[0])/2
      const g = (firstRgba[1] + secondRgba[1])/2
      const b = (firstRgba[2] + secondRgba[2])/2

      return `rgba(${r},${g},${b},1)`;
    }
  },

  mounted() {
    this.initContext();
    this.getColorPicker();
  },
};
</script>
