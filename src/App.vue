<template>
  <div id="app">
    <!-- <img alt="Vue logo" src="./assets/logo.png" /> -->
    <!-- <HelloWorld msg="Welcome to my very first Vue.js App" /> -->
    <button @click="running = !running">{{running ? "PAUSE" : "START"}}</button>
    <div>time = {{time.toFixed(2)}} s, numCol = {{numCol}}</div>
    <div class="container" height="800px" v-bind:style="{width: width + 'px'}">
      <div v-for="(x,i) of xs" :key="i" class="dot" v-bind:style="{
        left:x + 'px',
        height: diameter + 'px',
        width: diameter + 'px',
        transitionDuration: dtNext / 1000 + 's'
      }"></div>
    </div>
  </div>
</template>

<script>
// import HelloWorld from "./components/HelloWorld.vue";

export default {
  name: "App",
  // components: {
  //   HelloWorld,
  // },
  data() {
    return {
      timeout: null,
      dtNext: 0,
      time: 0,
      dtLast: null,
      xs: [100, 300, 500, 700, 900, 1100],
      vs: [200, 400, 1000, -200, 300, -70],
      diameter: 100,
      numCol: 0,
      width: 1200,
      running: true
    }
  },
  beforeDestroy() {
    // prevent memory leak
    clearTimeout(this.timeout)
  },
  methods: {
    increment: function () {
      // "numCol" means the number of the next collision, w/1-based counting
      if (this.numCol > 0) {
        this.dtLast = this.dtNext;
        let dtMin = Infinity;
        let dt, iMin, jMin, dv, dx;
        for (let i = 0; i < this.xs.length; i++) {
          for (let j = i; j < this.xs.length; j++) {
            if (j !== i) {
              dv = this.vs[i] - this.vs[j];
              dx = this.xs[j] - this.xs[i];
            } else {
              dv = 2 * this.vs[i];
              dx = 2 * ((dv > 0 ? this.width : 0) - this.xs[i]);
            }
            if (Math.sign(dv) === Math.sign(dx)) {
              dt = (Math.abs(dx) - this.diameter) / Math.abs(dv);
              if (dt < dtMin)[iMin, jMin, dtMin] = [i, j, dt];
            }
          }
        }
        this.xs = this.xs.map((x, i) => x + this.vs[i] * dtMin);
        this.dtNext = dtMin * 1000;
        if (iMin !== jMin) {
          [this.vs[iMin], this.vs[jMin]] = [this.vs[jMin], this.vs[iMin]];
        } else {
          this.vs[iMin] *= -1;
        }
        this.time += this.running ? this.dtNext / 1000 : 0;
      }
      this.numCol++;
      this.timeout = setTimeout(this.increment, this.dtNext);
    }
  },
  created() {this.increment()}
    // this.interval = setInterval(this.increment, this.dt);
    // this.interval = setInterval(() => this.time += this.running ? this.dt / 1000 : 0, this.dt);
    // if (this.running) {
    //   this.interval = setInterval(() => this.time += this.dt / 1000, this.dt);
    // } else {
    //   if (this.time) clearInterval(this.interval);
    // }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.container { position: relative; }
.dot {
  position: absolute;
  box-sizing: border-box;
  border-width: 1px;
  border-style: solid;
  border-color: black;
  border-radius: 50%;
  top: 100px;
  transition-timing-function: linear;
  transition-property: all;
}

</style>
