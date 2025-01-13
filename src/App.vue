<template>
  <a-layout>
    <a-layout-header>
    </a-layout-header>
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
            <a-button @click="saveCanvas" type="dashed">保存</a-button>
            <a-button @click="exportCanvas" type="dashed">导出为图片</a-button>
            <a-button @click="exportAsTerminalArt" type="dashed">导出为终端字符画</a-button>
            <a-popconfirm content="确定要清空画布吗？" okText="Yes" cancelText="No" @ok="clearCanvas">
              <a-button status="danger">
                清除
              </a-button>
            </a-popconfirm>
          </div>
          <br>
          <a-button type="primary" @click="handleClick">说明</a-button>
          <a-drawer :width="400" :visible="drewVisible" @ok="handleOk" @cancel="handleCancel" unmountOnClose>
            <template #title>
              <h1>说明</h1>
            </template>
            <h2>原理</h2>
            <p>
              在Unicode字符中 \u2584 为 ▄ ，\u2580 为 ▀ ,配合终端控制字符便可显示出像素画。
            </p>
            <p>
              本项目可快速用于快速制作终端程序logo，也可用于绘制像素画。
            </p>
            <h2>使用</h2>
            <li>按住鼠标左键开始绘制</li>
            <li>按住Ctrl鼠标经过的地方也可绘制</li>
            <li>按住鼠标右键擦除</li>
            <li>导出为终端字符画会将像素画转义的字符串写入剪贴板</li>
            <li>导出为图片如字面意</li>
            <li>擦除即清空画板</li>
            <li>保存会将当前像素画保存到Localstorage(只有一张)</li>
            <h2>Github</h2>
            <a-link href="https://github.com/Snape-max/terminal_art_creator" icon>Snape-max/terminal_art_creator</a-link>
          </a-drawer>
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
      hasUnsavedChanges: false, // 是否有未保存的更改
      drewVisible: false,
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
      // if (canvasData.length == 0) return;
      localStorage.setItem('pixelCanvas', canvasData);
      this.hasUnsavedChanges = false;
      this.$message.success('画布已保存！');
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
    },
    // 导出为终端艺术字符串
    exportAsTerminalArt() {
      if (this.$refs.PixelCanvas) {
        const terminalArt = this.$refs.PixelCanvas.exportAsTerminalArt();

        // 将结果复制到剪贴板
        navigator.clipboard.writeText(terminalArt)
          .then(() => {
            this.$message.success("终端字符画已复制到剪贴板！");
          })
          .catch((err) => {
            console.error("无法复制到剪贴板：", err);
            this.$message.error("复制失败，请手动复制以下内容：\n\n" + terminalArt);
          });
      } else {
        console.error("PixelCanvas 组件未正确加载");
      }
    },
    // 抽屉控件
    handleClick() {
      this.drewVisible = true
    },
    handleOk() {
      this.drewVisible = false
    },
    handleCancel() {
      this.drewVisible = false
    }
  },
  mounted() {
    // 从 localStorage 加载画布内容
    this.$nextTick(() => {
      if (this.$refs.PixelCanvas) {
        const savedCanvas = localStorage.getItem('pixelCanvas');
        if (savedCanvas) {
          const grid = JSON.parse(savedCanvas);
          this.$refs.PixelCanvas.grid = grid;

          // 手动更新 canvasWidth 和 canvasHeight
          this.canvasWidth = grid.length;
          this.canvasHeight = grid[0].length;

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
  background-image: url("./assets/herringbone.png");
  
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
  background: rgba(119, 119, 119, 0.111); /* 半透明背景 */
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