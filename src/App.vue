<template>
  <a-layout>
    <a-layout-content>
      <div class="container">
        <!-- 左侧设置区域 -->
        <div class="settings-panel">
          <!-- 调色板 -->
          <Palette @color-selected="handleColorSelected" />
          <!-- 画布大小调整 -->
          <div class="canvas-size-controls">
            <div class="canvas-size-control">
              <label for="canvas-width">画布宽度：</label>
              <a-input-number
                id="canvas-width"
                v-model:modelValue="canvasWidth"
                :min="1"
                :max="50"
                :step="2"
              />
            </div>
            <div class="canvas-size-control">
              <label for="canvas-height">画布高度：</label>
              <a-input-number
                id="canvas-height"
                v-model:modelValue="canvasHeight"
                :min="1"
                :max="50"
                :step="2"
              />
            </div>
          </div>
          <!-- 像素宽度设置 -->
          <div class="pixel-size-control">
            <label for="pixel-size">像素宽度（px）：</label>
            <a-input-number
              id="pixel-size"
              v-model:modelValue="pixelSize"
              :min="1"
              :max="50"
              :step="1"
            />
          </div>
          <!-- 保存和导出按钮 -->
          <div class="action-buttons">
            <a-button @click="saveCanvas">保存</a-button>
            <a-button @click="exportCanvas">导出</a-button>
            <a-button @click="clearCanvas" status="danger">清除</a-button>
          </div>
        </div>
        <!-- 右侧画布区域 -->
        <div class="canvas-container">
          <PixelCanvas
            ref="PixelCanvas"
            :width="canvasWidth"
            :height="canvasHeight"
            :selected-color="selectedColor"
            :pixel-size="pixelSize"
            @change="handleChange"
          />
        </div>
      </div>
    </a-layout-content>
  </a-layout>
</template>

<script>
import Palette from './components/Palette.vue';
import PixelCanvas from './components/PixelCanvas.vue';

export default {
  components: {
    Palette,
    PixelCanvas
  },
  data() {
    return {
      canvasWidth: 16, // 默认画布宽度
      canvasHeight: 16, // 默认画布高度
      selectedColor: '#000000', // 默认颜色
      pixelSize: 20, // 默认像素宽度
      hasUnsavedChanges: false // 是否有未保存的更改
    };
  },
  methods: {
    handleColorSelected(color) {
      this.selectedColor = color;
    },
    handleChange() {
      this.hasUnsavedChanges = true;
    },
    // 保存画布内容到 localStorage
    saveCanvas() {
      const canvasData = JSON.stringify(this.$refs.PixelCanvas.grid);
      localStorage.setItem('pixelCanvas', canvasData);
      this.hasUnsavedChanges = false;
      alert('画布已保存！');
    },
    // 导出画布内容为图片
    exportCanvas() {
      const canvas = document.createElement('canvas');
      const ctx = canvas.getContext('2d');
      const pixelSize = this.pixelSize;
      const width = this.canvasWidth * pixelSize;
      const height = this.canvasHeight * pixelSize;

      canvas.width = width;
      canvas.height = height;

      // 绘制每个像素
      this.$refs.PixelCanvas.grid.forEach((row, rowIndex) => {
        row.forEach((cell, cellIndex) => {
          ctx.fillStyle = cell;
          ctx.fillRect(
            cellIndex * pixelSize,
            rowIndex * pixelSize,
            pixelSize,
            pixelSize
          );
        });
      });

      // 导出为图片
      const link = document.createElement('a');
      link.href = canvas.toDataURL('image/png');
      link.download = 'pixel-art.png';
      link.click();
    },
    // 清除画布内容
    clearCanvas() {
      if (this.$refs.PixelCanvas) {
        this.$refs.PixelCanvas.clearGrid();
        this.hasUnsavedChanges = true; // 标记为有未保存的更改
      } else {
        console.error('PixelCanvas 组件未正确加载');
      }
    }
  },
  mounted() {
    // 从 localStorage 加载画布内容
    this.$nextTick(() => {
      if (this.$refs.PixelCanvas) {
        const savedCanvas = localStorage.getItem('pixelCanvas');
        if (savedCanvas) {
          this.$refs.PixelCanvas.grid = JSON.parse(savedCanvas);
        }
      }
    });

    // 监听 beforeunload 事件
    window.addEventListener('beforeunload', (event) => {
      if (this.hasUnsavedChanges) {
        event.preventDefault();
        event.returnValue = '';
        const confirmExport = confirm('您有未保存的更改，是否导出当前画布？');
        if (confirmExport) {
          this.exportCanvas();
        }
      }
    });
  },
  beforeDestroy() {
    // 移除事件监听
    window.removeEventListener('beforeunload', this.promptBeforeUnload);
  }
};
</script>

<style>

body {
  background-color: #DFDBE5;
  background-image: url("data:image/svg+xml,%3Csvg width='6' height='6' viewBox='0 0 6 6' xmlns='http://www.w3.org/2000/svg'%3E%3Cg fill='%239C92AC' fill-opacity='1' fill-rule='evenodd'%3E%3Cpath d='M5 0h1L0 6V5zM6 5v1H5z'/%3E%3C/g%3E%3C/svg%3E");
  
  user-select: none; /* 禁用文字选中 */
  -webkit-user-select: none; /* Safari */
  -moz-user-select: none; /* Firefox */
  -ms-user-select: none; /* IE/Edge */
}

/* 毛玻璃效果 */
body::before {
  content: '';
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(255, 255, 255, 0); /* 半透明背景 */
  backdrop-filter: blur(1px); /* 毛玻璃效果 */
  z-index: -1; /* 置于底层 */
}
.container {
  display: flex;
  align-items: center;
  gap: 20px;
  padding: 20px;
}

.settings-panel {
  width: 200px;
  padding: 16px;
}

.canvas-size-controls {
  display: flex;
  gap: 16px; /* 控件之间的间距 */
  margin-top: 20px;
}

.canvas-size-control {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.canvas-size-control label {
  font-weight: bold;
}

.pixel-size-control {
  margin-top: 10px;
}

.pixel-size-control label {
  font-weight: bold;
  margin-bottom: 1px;
}


.action-buttons {
  margin-top: 20px;
  display: flex;
  gap: 8px;
}

.canvas-container {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 80vh;
  /* border-left: 1px solid #e5e5e5; */
  padding-left: 20px;
}
</style>