<template>
  <div class="pixel-canvas">
    <div class="canvas-grid">
      <div
        v-for="(row, rowIndex) in grid"
        :key="rowIndex"
        class="canvas-row"
      >
        <div
          v-for="(cell, cellIndex) in row"
          :key="cellIndex"
          :style="{ 
            backgroundColor: cell,
            outline: cell === '#ffffff00' ? '1px solid #3f3f3f' : 'none', 
            width: pixelSize + 'px',
            height: pixelSize + 'px'
          }"
          class="canvas-cell"
          @mousedown="startPainting(rowIndex, cellIndex)"
          @mouseenter="paintIfMouseDown(rowIndex, cellIndex)"
          @contextmenu.prevent="startErasing(rowIndex, cellIndex)"
        ></div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "PixelCanvas",
  props: {
    width: {
      type: Number,
      required: true
    },
    height: {
      type: Number,
      required: true
    },
    selectedColor: {
      type: String,
      default: '#FFFFFF'
    }
  },
  data() {
    return {
      grid: [],
      isPainting: false,
      isErasing: false,
      isCtrlPressed: false,
      pixelSize: 10 // 默认值
    };
  },
  watch: {
    width: {
      immediate: true,
      handler(newWidth, oldWidth) {
        this.resizeGrid(newWidth, this.height, oldWidth, this.height);
      }
    },
    height: {
      immediate: true,
      handler(newHeight, oldHeight) {
        this.resizeGrid(this.width, newHeight, this.width, oldHeight);
      }
    }
  },
  methods: {
    initializeGrid(width, height) {
      this.grid = Array.from({ length: height }, () =>
        Array.from({ length: width }, () => '#ffffff00')
      );
    },
    resizeGrid(newWidth, newHeight, oldWidth, oldHeight) {
      oldHeight = this.grid.length;
      oldWidth = oldHeight > 0 ? this.grid[0].length : 0;
      const newGrid = Array.from({ length: newHeight }, () =>
        Array.from({ length: newWidth }, () => '#ffffff00')
      );

      const offsetX = Math.floor((newWidth - oldWidth) / 2);
      const offsetY = Math.floor((newHeight - oldHeight) / 2);

      for (let i = 0; i < oldHeight; i++) {
        for (let j = 0; j < oldWidth; j++) {
          const newRow = i + offsetY;
          const newCol = j + offsetX;

          if (
            newRow >= 0 &&
            newCol >= 0 &&
            newRow < newHeight &&
            newCol < newWidth
          ) {
            newGrid[newRow][newCol] = this.grid[i][j];
          }
        }
      }

      this.grid = newGrid;
      
      // 确保 DOM 更新完成后再计算 pixelSize
      this.$nextTick(() => {
        this.calculatePixelSize();
      });
    },
    calculatePixelSize() {
      if (!this.$el || !this.$el.parentElement) {
        console.warn('Canvas container is not available.');
        return;
      }

      const canvasContainer = this.$el.parentElement;
      const containerWidth = canvasContainer.clientWidth;
      const containerHeight = canvasContainer.clientHeight;

      const maxWidth = containerWidth / this.width;
      const maxHeight = containerHeight / this.height;
      this.pixelSize = Math.min(maxWidth, maxHeight);

      this.pixelSize = Math.floor(this.pixelSize);
    },
    startPainting(rowIndex, cellIndex) {
      if (event.button === 0) {
        this.isPainting = true;
        this.isErasing = false;
        this.paintCell(rowIndex, cellIndex);
      } else if (event.button === 2) {
        this.isErasing = true;
        this.isPainting = false;
        this.eraseCell(rowIndex, cellIndex);
      }
    },
    paintIfMouseDown(rowIndex, cellIndex) {
      if (this.isPainting || this.isCtrlPressed) {
        this.paintCell(rowIndex, cellIndex);
        
      } else if (this.isErasing) {
        this.eraseCell(rowIndex, cellIndex);
      }
    },
    paintCell(rowIndex, cellIndex) {
      this.grid[rowIndex][cellIndex] = this.selectedColor;
      this.$emit('change');
    },
    eraseCell(rowIndex, cellIndex) {
      this.grid[rowIndex][cellIndex] = '#ffffff00';
      this.$emit('change');
    },
    startErasing(rowIndex, cellIndex) {
      this.isErasing = true;
      this.isPainting = false;
      this.eraseCell(rowIndex, cellIndex);
    },
    clearGrid() {
      this.initializeGrid(this.width, this.height);
      this.$emit('change');
    },
    onKeyDown(event) {
      if (event.key === 'Control') {
        this.isCtrlPressed = true;
        
      }
    },
    onKeyUp(event) {
      if (event.key === 'Control') {
        this.isCtrlPressed = false;
      }
    },
    // 将十六进制颜色转换为 RGB
    hexToRgb(hex) {
      if (hex.length === 7) {
        // 包含透明度的十六进制颜色
        const r = parseInt(hex.slice(1, 3), 16);
        const g = parseInt(hex.slice(3, 5), 16);
        const b = parseInt(hex.slice(5, 7), 16);
        return [r, g, b];
      }
      return [0, 0, 0]; // 默认黑色
    },

    // 将十六进制颜色转换为透明度
    hexToAlpha(hex) {
      if (hex.length === 9) {
        // 包含透明度的十六进制颜色
        return parseInt(hex.slice(7, 9), 16) / 255;
      }
      return 1; // 默认不透明
    },
    // 导出为终端艺术字符串
    exportAsTerminalArt() {
      let output = "";

      // 遍历每一对行
      for (let y = 0; y < this.height; y += 2) {
        for (let x = 0; x < this.width; x++) {
          // 获取上下两个像素
          const upperPixel = this.grid[y][x];
          const lowerPixel = y + 1 < this.height ? this.grid[y + 1][x] : "#ffffff00";

          // 解析颜色和透明度
          const upperColor = this.hexToRgb(upperPixel);
          const upperAlpha = this.hexToAlpha(upperPixel);
          const lowerColor = this.hexToRgb(lowerPixel);
          const lowerAlpha = this.hexToAlpha(lowerPixel);

          // 根据透明度决定如何显示
          if (upperAlpha > 0 && lowerAlpha > 0) {
            // 上下像素均不透明
            output += `\\x1b[48;2;${upperColor[0]};${upperColor[1]};${upperColor[2]}m\\x1b[38;2;${lowerColor[0]};${lowerColor[1]};${lowerColor[2]}m▄\\x1b[0m`;
          } else if (upperAlpha === 0 && lowerAlpha === 0) {
            // 上下像素均透明
            output += " ";
          } else if (upperAlpha === 0) {
            // 上半像素透明
            output += `\\x1b[38;2;${lowerColor[0]};${lowerColor[1]};${lowerColor[2]}m▄\\x1b[0m`;
          } else if (lowerAlpha === 0) {
            // 下半像素透明
            output += `\\x1b[38;2;${upperColor[0]};${upperColor[1]};${upperColor[2]}m▀\\x1b[0m`;
          }
        }
        output += "\\n"; // 换行
      }

      // 删除末尾的换行符
      output = output.slice(0, -2);

      return output;
    },
  },
  mounted() {
    this.initializeGrid(this.width, this.height);
    this.calculatePixelSize();

    document.addEventListener('mouseup', () => {
      this.isPainting = false;
      this.isErasing = false;
    });

    document.addEventListener('keydown', this.onKeyDown);
    document.addEventListener('keyup', this.onKeyUp);

    this.observer = new ResizeObserver(() => {
      this.calculatePixelSize();
    });

    if (this.$el.parentElement) {
      this.observer.observe(this.$el.parentElement);
    }
  },
  beforeDestroy() {
    document.removeEventListener('keydown', this.onKeyDown);
    document.removeEventListener('keyup', this.onKeyUp);

    if (this.observer) {
      this.observer.disconnect();
    }
  }
};
</script>

<style scoped>
.pixel-canvas {
  display: flex;
  justify-content: center;
  align-items: center;
  user-select: none;
  background-color: #b3b3b300;
}

.canvas-grid {
  display: flex;
  flex-direction: column;
  gap: 0px;
}

.canvas-row {
  display: flex;
  gap: 0px;
}

.canvas-cell {
  cursor: pointer;
  box-sizing: border-box;
}
</style>