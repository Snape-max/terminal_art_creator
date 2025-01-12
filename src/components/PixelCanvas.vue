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
            :style="{ backgroundColor: cell }"
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
      isPainting: false, // 是否正在绘制
      isErasing: false, // 是否正在擦除
      isCtrlPressed: false // Ctrl 键是否按下
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
    // 初始化画布
    initializeGrid(width, height) {
      this.grid = Array.from({ length: height }, () =>
        Array.from({ length: width }, () => '#ffffff00')
      );
    },
    // 调整画布大小
    resizeGrid(newWidth, newHeight, oldWidth, oldHeight) {
      const newGrid = Array.from({ length: newHeight }, () =>
        Array.from({ length: newWidth }, () => '#ffffff00')
      );

      // 将旧画布内容复制到新画布的中心位置
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
    },
    // 开始绘制
    startPainting(rowIndex, cellIndex) {
      if (event.button === 0) { // 左键
        this.isPainting = true;
        this.isErasing = false;
        this.paintCell(rowIndex, cellIndex);
      } else if (event.button === 2) { // 右键
        this.isErasing = true;
        this.isPainting = false;
        this.eraseCell(rowIndex, cellIndex);
      }
    },
    // 如果鼠标按下或 Ctrl 键按下，则绘制或擦除
    paintIfMouseDown(rowIndex, cellIndex) {
      if (this.isPainting || this.isCtrlPressed) {
        this.paintCell(rowIndex, cellIndex);
      } else if (this.isErasing) {
        this.eraseCell(rowIndex, cellIndex);
      }
    },
    // 绘制单元格
    paintCell(rowIndex, cellIndex) {
      this.grid[rowIndex][cellIndex] = this.selectedColor;
      this.$emit('change');
    },
    // 擦除单元格（设置为白色）
    eraseCell(rowIndex, cellIndex) {
      this.grid[rowIndex][cellIndex] = '#FFFFFF00';
      this.$emit('change');
    },
    // 开始擦除
    startErasing(rowIndex, cellIndex) {
      this.isErasing = true;
      this.isPainting = false;
      this.eraseCell(rowIndex, cellIndex);
    },
    // 清除画布内容
    clearGrid() {
      this.initializeGrid(this.width, this.height);
      this.$emit('change'); // 通知父组件有更改
    },
    // 监听 Ctrl 键按下
    onKeyDown(event) {
      if (event.key === 'Control') {
        this.isCtrlPressed = true;
      }
    },
    // 监听 Ctrl 键松开
    onKeyUp(event) {
      if (event.key === 'Control') {
        this.isCtrlPressed = false;
      }
    }
  },
  mounted() {
    // 初始化画布
    this.initializeGrid(this.width, this.height);

    // 监听鼠标松开事件
    document.addEventListener('mouseup', () => {
      this.isPainting = false;
      this.isErasing = false;
    });

    // 监听 Ctrl 键按下和松开
    document.addEventListener('keydown', this.onKeyDown);
    document.addEventListener('keyup', this.onKeyUp);
  },
  beforeDestroy() {
    // 移除事件监听
    document.removeEventListener('keydown', this.onKeyDown);
    document.removeEventListener('keyup', this.onKeyUp);
  }
};
  </script>
  
  <style scoped>
  .pixel-canvas {
    display: flex;
    justify-content: center;
    align-items: center;
    user-select: none; /* 禁用文字选中 */
    -webkit-user-select: none; /* Safari */
    -moz-user-select: none; /* Firefox */
    -ms-user-select: none; /* IE/Edge */
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
    width: 13px;
    height: 13px;
    border: 1px solid #3f3f3f;
    cursor: pointer;    
  }
  </style>