<template>
  <div class="canvas-viewer">
    <canvas ref="canvas" :width="canvasSize" :height="canvasSize"></canvas>
  </div>
</template>

<script>
export default {
  props: ['grid', 'size'],
  computed: {
    canvasSize() {
      return this.size * 30; 
    }
  },
  mounted() {
    this.drawGrid();
  },
  methods: {
    drawGrid() {
      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext('2d');
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      for (let row = 0; row < this.grid.length; row++) {
        for (let col = 0; col < this.grid[row].length; col++) {
          const x = col * 30 + 15;
          const y = row * 30 + 15;
          ctx.strokeRect(col * 30, row * 30, 30, 30);
          ctx.fillText(this.grid[row][col], x, y);
        }
      }
    }
  },
  watch: {
    grid() {
      this.drawGrid();
    }
  }
};
</script>

  