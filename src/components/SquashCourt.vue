<template>
  <div class="mobile-wrapper">

    <div class="centered scroll-container">
      <div id="line-drawing">
        <div v-if="countdown > 0" class="countdown">{{ countdown }}</div>
        <h1 v-else-if="setCountdown > 0" class="countdown" style="color: #cb0202">{{ setCountdown }}s</h1>
        <h1 v-else-if="cooldownCountdown > 0" class="countdown" style="color: #646cff">{{ cooldownCountdown }}s</h1>

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
    </div>

    <button class="start" v-if="!intervalId && countdown === 0 && setCountdown === 0 && cooldownCountdown === 0"
            @click="startColorCycle">
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

    <select class="ghosting-time" v-model="sets">
      <option v-for="n in 20" :key="n" :value="n">
        {{ n }} Set{{ n > 1 ? 's' : '' }}
      </option>
    </select>

    <select class="ghosting-time" v-model="cooldown">
      <option v-for="cool in cooldownOptions" :value="cool" :key="cool">
        {{ formatDuration(cool) }} cooldown
      </option>
    </select>

    <br>
  </div>
</template>

<script>
export default {
  name: 'SquashCourt',
  data() {
    const base = import.meta.env.BASE_URL;
    return {
      gridColors: Array(6).fill('transparent'),
      intervalId: null,
      countdown: 0,
      setCountdown: 0,
      cooldownCountdown: 0,
      difficulty: 'easy',
      cycleDuration: 15,
      cooldown: 10,
      sets: 1,
      rallyType: 'full',
      durationOptions: [15, 30, 45, 60, 75, 90, 105, 120, 135, 150],
      cooldownOptions: [10, 20, 30, 40, 50, 60],
      sounds: [
        new Audio(`${base}/sounds/front-left.mp3`),
        new Audio(`${base}/sounds/front-right.mp3`),
        new Audio(`${base}/sounds/middle-left.mp3`),
        new Audio(`${base}/sounds/middle-right.mp3`),
        new Audio(`${base}/sounds/back-left.mp3`),
        new Audio(`${base}/sounds/back-right.mp3`)
      ],
      timeoutIds: [],
      setTimer: null,
      cooldownTimer: null
    };
  },
  computed: {
    colorChangeInterval() {
      switch (this.difficulty) {
        case 'medium':
          return 2800;
        case 'hard':
          return 1900;
        default:
          return 3500;
      }
    },
    difficultyClass() {
      return this.difficulty;
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
      if (this.intervalId || this.countdown > 0) return;

      this.countdown = 3;
      let currentSet = 0;

      const preStart = setInterval(() => {
        this.countdown--;
        if (this.countdown === 0) {
          clearInterval(preStart);
          this.runSet(currentSet);
        }
      }, 1000);
      this.timeoutIds.push(preStart);
    },
    runSet(currentSet) {
      this.setCountdown = this.cycleDuration;

      this.setTimer = setInterval(() => {
        this.setCountdown--;
        if (this.setCountdown === 0) clearInterval(this.setTimer);
      }, 1000);

      this.intervalId = setInterval(this.changeColor, this.colorChangeInterval);

      const endOfSet = setTimeout(() => {
        clearInterval(this.intervalId);
        this.intervalId = null;
        this.gridColors.fill('transparent');

        currentSet++;

        if (currentSet < this.sets) {
          this.cooldownCountdown = this.cooldown;

          this.cooldownTimer = setInterval(() => {
            this.cooldownCountdown--;
            if (this.cooldownCountdown === 0) clearInterval(this.cooldownTimer);
          }, 1000);

          const nextSetTimeout = setTimeout(() => {
            this.runSet(currentSet);
          }, this.cooldown * 1000);

          this.timeoutIds.push(this.cooldownTimer, nextSetTimeout);
        }
      }, this.cycleDuration * 1000);

      this.timeoutIds.push(this.setTimer, this.intervalId, endOfSet);
    },
    stopColorCycle() {
      clearInterval(this.intervalId);
      clearInterval(this.setTimer);
      clearInterval(this.cooldownTimer);

      this.timeoutIds.forEach(id => clearTimeout(id));
      this.timeoutIds = [];

      this.intervalId = null;
      this.setTimer = null;
      this.cooldownTimer = null;

      this.countdown = 0;
      this.setCountdown = 0;
      this.cooldownCountdown = 0;
      this.gridColors.fill('transparent');
    },
    formatDuration(seconds) {
      const minutes = Math.floor(seconds / 60);
      const remaining = seconds % 60;
      return `${minutes ? minutes + 'm ' : ''}${remaining}s`;
    }
  }
};
</script>

<style scoped>
.scroll-container {
  width: 100%;
  overflow-x: hidden;
}

#line-drawing {
  width: 100%;
  max-width: 480px;
  height: auto;
}

svg {
  width: 100%;
  height: auto;
}

.centered {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 100%;
}

button {
  font-size: 1.5em;
}

button, select {
  margin-top: 10px;
  width: 100%;
  max-width: 400px;
  box-sizing: border-box;
  font-size: 1.2em;
}

.heading {
  display: flex;
  justify-content: center;
  align-items: center;
  color: #1a1a1a;
  margin-top: -2em;
  margin-bottom: 0.3em;
}

.countdown {
  position: absolute;
  top: 1em;
  font-size: 9em;
  color: #161c25;
  font-weight: bold;
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

.mobile-wrapper {
  width: 100vw;
  max-width: 100%;
  overflow-x: hidden;
  padding: 1em;
  box-sizing: border-box;
}
</style>
