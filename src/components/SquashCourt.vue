<template>
  <div id="line-drawing">
    <div v-if="countdown > 0" class="countdown">{{ countdown }}</div>
    <svg width="480" height="669" viewBox="0 0 160 223" xmlns="http://www.w3.org/2000/svg">
      <rect x="1" y="1" width="158" height="221" fill="none" stroke="red"/>
      <line x1="1" y1="125" x2="159" y2="125" stroke="red"/>
      <line x1="80" y1="125" x2="80" y2="222" stroke="red"/>
      <line x1="120" y1="125" x2="120" y2="160" stroke="red"/>
      <line x1="40" y1="125" x2="40" y2="160" stroke="red"/>
      <line x1="1" y1="160" x2="40.5" y2="160" stroke="red"/>
      <line x1="119.5" y1="160" x2="159" y2="160" stroke="red"/>
      <rect v-for="(color, index) in gridColors" :key="index"
            :x="1 + (index % 2) * 79" :y="1 + Math.floor(index / 2) * 73.67"
            width="78" height="73.67" :fill="color"/>
    </svg>
  </div>
  <button class="start" v-if="!intervalId && countdown === 0" @click="startColorCycle">
    Start Ghosting
  </button>
  <button class="stop" v-else @click="stopColorCycle">
    Stop Ghosting
  </button>
  <br>
  <select v-model="difficulty">
    <option value="easy">Easy</option>
    <option value="medium">Medium</option>
    <option value="hard">Hard</option>
  </select>
</template>

<script>
export default {
  name: 'SquashCourt',
  data() {
    return {
      gridColors: Array(6).fill('transparent'),
      intervalId: null,
      countdown: 0,
      difficulty: 'easy'
    };
  },
  computed: {
    colorChangeInterval() {
      switch (this.difficulty) {
        case 'medium':
          return 2500;
        case 'hard':
          return 1500;
        default:
          return 3000;
      }
    }
  },
  methods: {
    changeColor(index) {
      this.gridColors[index] = 'rgba(0, 255, 0, 0.6)';
      setTimeout(() => {
        this.gridColors[index] = 'transparent';
      }, 500);
    },
    startColorCycle() {
      if (this.intervalId || this.countdown > 0) {
        return;
      }
      this.countdown = 3;
      const countdownInterval = setInterval(() => {
        this.countdown -= 1;
        if (this.countdown === 0) {
          clearInterval(countdownInterval);
          this.intervalId = setInterval(() => {
            const index = Math.floor(Math.random() * this.gridColors.length);
            this.changeColor(index);
          }, this.colorChangeInterval);
        }
      }, 1000);
    },
    stopColorCycle() {
      clearInterval(this.intervalId);
      this.intervalId = null;
      this.countdown = 0;
      this.gridColors.fill('transparent');
    }
  }
}
</script>

<style scoped>
#line-drawing {
  display: flex;
  justify-content: center;
  align-items: center;
}

button, select {
  margin-top: 10px;
}

.countdown {
  position: absolute;
  top: 3em;
  font-size: 7em;
  color: #161c25;
}

.start {
  background-color: green;
}

.stop {
  background-color: #57010e;
}
</style>
