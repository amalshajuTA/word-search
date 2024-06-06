<template>
  <div class="form">
    <h1 class="title">Word Search Generator</h1>
    <form @submit.prevent="generate">
      <div class="size-input">
        <label for="size">Grid Size : </label>
        <input type="number" v-model="size" min="5" max="50" required />
      </div>
      <div class="words-input">
        <label for="words">Words (comma separated) : </label>
        <input type="text" v-model="words" required />
      </div>
      <div class="backward-input">
        <label for="backward">Allow Backward : </label>
        <input type="checkbox" v-model="allowBackward" />
      </div>
      <div class="diagonal-input">
        <label for="diagonal">Allow Diagonal : </label>
        <input type="checkbox" v-model="allowDiagonal" />
      </div>
      <button class="submit-button" type="submit">Generate</button>
    </form>
    <Canvas v-if="grid.length" :grid="grid" :size="size" ref="canvasComponent" />
    <button class="downloadBtn" @click="downloadImage" v-if="grid.length">Download</button>
  </div>
</template>

<script>
import Canvas from './components/CanvasViewer.vue';

export default {
  data() {
    return {
      size: 10,
      words: '',
      allowBackward: false,
      allowDiagonal: false,
      grid: []
    };
  },
  methods: {
    getDirections(allowReverse, allowDiagonal) {
      const directions = [
        { dx: 1, dy: 0 }, 
        { dx: 0, dy: 1 },   
      ];

      if (allowDiagonal) {
        directions.push({ dx: 1, dy: 1 }, { dx: 1, dy: -1 });
      }

      if (allowReverse) {
        directions.push({ dx: -1, dy: 0 }, { dx: 0, dy: -1 });
        if (allowDiagonal) {
          directions.push({ dx: -1, dy: -1 }, { dx: -1, dy: 1 });
        }
      }

      return directions;
    },

    generate() {
      const wordsArray = this.words.split(',').map(word => word.trim().toUpperCase());
      try {
        const grid = this.placeWordsInGrid(wordsArray, this.size, this.allowBackward, this.allowDiagonal);
        this.grid = this.fillEmptySpaces(grid);
      } catch (error) {
        alert(error.message);
        this.grid = [];
      }
    },

    placeWordsInGrid(words, size, allowBackward, allowDiagonal) {
      const grid = Array.from({ length: size }, () => Array(size).fill(''));

      words.forEach(word => {
        const placed = this.tryPlacing(word, grid, size, allowBackward, allowDiagonal);
        if (!placed) {
          throw new Error(`Failed to place word: ${word}`);
        }
      });
      return grid;
    },

    tryPlacing(word, grid, size, allowBackward, allowDiagonal) {
      const directions = this.getDirections(allowBackward, allowDiagonal);

      for (let i = directions.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [directions[i], directions[j]] = [directions[j], directions[i]];
      }

      for (const { dx, dy } of directions) {
        for (let attempt = 0; attempt < 100; attempt++) {
          const x = Math.floor(Math.random() * size);
          const y = Math.floor(Math.random() * size);
          if (this.canPlaceWord(word, grid, x, y, dx, dy)) {
            this.placeWord(word, grid, x, y, dx, dy);
            return true;
          }
        }
      }

      return false;
    },

    canPlaceWord(word, grid, x, y, dx, dy) {
      for (let i = 0; i < word.length; i++) {
        const newX = x + i * dx;
        const newY = y + i * dy;

        if (
          newX < 0 || newX >= this.size || newY < 0 || newY >= this.size ||
          (grid[newY][newX] !== '' && grid[newY][newX] !== word[i])
        ) {
          return false;
        }
      }

      return true;
    },

    placeWord(word, grid, x, y, dx, dy) {
      for (let i = 0; i < word.length; i++) {
        const newX = x + i * dx;
        const newY = y + i * dy;
        grid[newY][newX] = word[i];
      }
    },

    fillEmptySpaces(grid) {
      for (let row = 0; row < this.size; row++) {
        for (let col = 0; col < this.size; col++) {
          if (!grid[row][col]) {
            grid[row][col] = String.fromCharCode(65 + Math.floor(Math.random() * 26));
          }
        }
      }
      return grid;
    },

    downloadImage() {
      const canvas = this.$refs.canvasComponent.$refs.canvas;
      if (canvas) {
        const link = document.createElement('a');
        link.href = canvas.toDataURL('image/png');
        link.download = 'word-search.png';
        link.click();
      } else {
        console.error('Canvas element not found');
      }
    }
  },
  components: {
    Canvas
  }
};
</script>
<style lang="scss">
@import './styles.scss';
</style>