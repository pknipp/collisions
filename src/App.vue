<template>
  <div id="app">
    <!-- <img alt="Vue logo" src="./assets/logo.png" /> -->
    <!-- <HelloWorld msg="Welcome to my very first Vue.js App" /> -->
    <button @click="running = !running">{{running ? "PAUSE" : "START"}}</button>
    <div>time = {{time.toFixed(2)}} s</div>
    <div>dtLast = {{dtLast / 1000}} s</div>
    <div>vs = {{vs}} px/s</div>
    <div>xs = {{xs}} px</div>
    <div>numCol = {{numCol}}</div>
    <div class="container" height="800px" width="1200px">
      <div class="dot" v-bind:style="{
        left:xs[0] + 'px',
        height: diameter + 'px',
        width: diameter + 'px',
        transitionDuration: 1 * dtNext / 1000 + 's'
      }"></div>
      <div class="dot" v-bind:style="{
        left:xs[1] + 'px',
        height: diameter + 'px',
        width: diameter + 'px',
        transitionDuration: 1 * dtNext / 1000 + 's'
      }"></div>
      <div class="dot" v-bind:style="{
        left:xs[2] + 'px',
        height: diameter + 'px',
        width: diameter + 'px',
        transitionDuration: 1 * dtNext / 1000 + 's'
      }"></div>
      <div class="dot" v-bind:style="{
        left:xs[3] + 'px',
        height: diameter + 'px',
        width: diameter + 'px',
        transitionDuration: 1 * dtNext / 1000 + 's'
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
      xs: [300, 500, 700, 900],
      vs: [10, -20, -25, -10],
      diameter: 100,
      numCol: 0,
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
        let dt, iMin, jMin;
        for (let i = 0; i < this.xs.length; i++) {
          for (let j = i + 1; j < this.xs.length; j++) {
            dt = (this.xs[j] - this.xs[i] - this.diameter) / (this.vs[i] - this.vs[j]);
            if (dt > 0 && dt < dtMin) [iMin, jMin, dtMin] = [i, j, dt];
          }
        }
        this.dtNext = dtMin * 1000;
        this.xs = this.xs.map((x, i) => x + this.vs[i] * this.dtNext / 1000);
        [this.vs[iMin], this.vs[jMin]] = [this.vs[jMin], this.vs[iMin]];
        this.time += this.running ? this.dtNext / 1000 : 0;
      }
      this.numCol++;
      if (this.dtNext >= 0) this.timeout = setTimeout(this.increment, this.dtNext);
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
