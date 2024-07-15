<template>
  <hgroup>
    <h2 class="heading">Squash Ghoster (Beta)</h2>
    <h2 class="heading">How to use</h2>
  </hgroup>
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
  <select v-model="difficulty" :class="difficultyClass">
    <option class="difficulty-class" value="easy">Easy</option>
    <option class="difficulty-class" value="medium">Medium</option>
    <option class="difficulty-class" value="hard">Hard</option>
  </select>
  <select class="ghosting-time" v-model="cycleDuration">
    <option v-for="seconds in durationOptions" :value="seconds" :key="seconds">
      {{ formatDuration(seconds) }}
    </option>
  </select>
  <br>
  <select class="ghosting-time" v-model="rallyType">
    <option value="full">Full Court</option>
    <option value="backhand">Backhand rally</option>
    <option value="forehand">Forehand rally</option>
    <option value="volleys">Volleys</option>
    <option value="short">Short game</option>
  </select>
</template>

<script>
// noinspection JSUnusedGlobalSymbols
export default {
  name: 'SquashCourt',
  data() {
    return {
      gridColors: Array(6).fill('transparent'),
      intervalId: null,
      countdown: 0,
      difficulty: 'easy',
      cycleDuration: 15,
      durationOptions: [15, 30, 45, 60, 75, 90, 105, 120, 135, 150],
      rallyType: 'full',
      sounds: [
        new Audio('/sounds/front-left.mp3'),
        new Audio('/sounds/front-right.mp3'),
        new Audio('/sounds/middle-left.mp3'),
        new Audio('/sounds/middle-right.mp3'),
        new Audio('/sounds/back-left.mp3'),
        new Audio('/sounds/back-right.mp3')
      ]
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
    },
    difficultyClass() {
      switch (this.difficulty) {
        case 'easy':
          return 'easy';
        case 'medium':
          return 'medium';
        case 'hard':
          return 'hard';
        default:
          return '';
      }
    }
  },
  methods: {
    playSound(index) {
      if (this.sounds[index]) {
        this.sounds[index].currentTime = 0;
        this.sounds[index].play();
      }
    },
    changeColor() {
      let index;
      switch (this.rallyType) {
        case 'backhand':
          index = Math.random() < 0.7 ? (Math.random() < 0.5 ? 0 : 2) : Math.floor(Math.random() * 6);
          break;
        case 'forehand':
          index = Math.random() < 0.7 ? (Math.random() < 0.5 ? 1 : 3) : Math.floor(Math.random() * 6);
          break;
        case 'volleys':
          index = [2, 3][Math.floor(Math.random() * 2)];
          break;
        case 'short':
          index = [0, 1][Math.floor(Math.random() * 2)];
          break;
        case 'full':
        default:
          index = Math.floor(Math.random() * 6);
          break;
      }
      this.gridColors[index] = 'rgba(0, 255, 0, 0.6)';
      this.playSound(index);
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
          this.intervalId = setInterval(this.changeColor, this.colorChangeInterval);
          setTimeout(() => {
            this.stopColorCycle();
          }, this.cycleDuration * 1000);
        }
      }, 1000);
    },
    stopColorCycle() {
      clearInterval(this.intervalId);
      this.intervalId = null;
      this.countdown = 0;
      this.gridColors.fill('transparent');
    },
    formatDuration(seconds) {
      const minutes = Math.floor(seconds / 60);
      const remainingSeconds = seconds % 60;
      return `${minutes}m ${remainingSeconds}s`;
    }
  }
}
</script>

<!--suppress CssUnusedSymbol -->
<style scoped>
#line-drawing {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  max-width: 480px;
  height: auto;
}

button {
  font-size: 1.5em;
}

button, select {
  margin-top: 10px;
}

.heading {
  display: flex;
  justify-content: center;
  align-items: center;
  color: #1a1a1a;
  margin-top: -1em;
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

select {
  margin-top: 10px;
  margin-left: 0.5em;
  margin-right: 0.5em;
  padding: 1em 2em;
  border: none;
  border-radius: 30px;
  background-color: #f4f4f4;
  font-size: 1em;
  font-weight: bold;
  color: #333;
  outline: none;
  cursor: pointer;
}

hgroup {
  margin-bottom: -1.3em;
}

.ghosting-time {
  color: black;
  border: 3px solid #3f3d3d;
}

.easy {
  color: green;
  border: 3px solid green;
}

.medium {
  color: #d46e00;
  border: 3px solid #d46e00;
}

.hard {
  color: red;
  border: 3px solid red;
}

.difficulty-class {
  color: #1a1a1a;
}
</style>
