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
      vs: [0, -20, -25, -10],
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
        if (this.numCol === 1) {
          this.dtNext = 1000 * (this.xs[1] - this.xs[0] - this.diameter) / (this.vs[0] - this.vs[1]);
        }
        if (this.numCol === 2) {
          this.dtNext = 1000 * (this.xs[2] - this.xs[1] - this.diameter) / (this.vs[1] - this.vs[2]);
        }
        if (this.numCol === 3) {
          this.dtNext = 1000 * (this.xs[3] - this.xs[2] - this.diameter) / (this.vs[2] - this.vs[3]);
        }
        this.xs[0] += this.vs[0] * this.dtNext / 1000;
        this.xs[1] += this.vs[1] * this.dtNext / 1000;
        this.xs[2] += this.vs[2] * this.dtNext / 1000;
        this.xs[3] += this.vs[3] * this.dtNext / 1000;
        if (this.numCol === 1) {
          [this.vs[0], this.vs[1]] = [this.vs[1], this.vs[0]];
        }
        if (this.numCol === 2) {
          [this.vs[1], this.vs[2]] = [this.vs[2], this.vs[1]];
        }
        this.time += this.running ? this.dtNext / 1000 : 0;
      }
      this.numCol++;
      console.log(this.numCol, this.dtLast, this.dtNext, this.xs, this.vs);
      if (this.numCol < 4) this.timeout = setTimeout(this.increment, this.dtNext);
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
