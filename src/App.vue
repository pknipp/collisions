<template>
  <div id="app">
    <!-- <img alt="Vue logo" src="./assets/logo.png" /> -->
    <!-- <HelloWorld msg="Welcome to my very first Vue.js App" /> -->
    <button @click="running = !running">{{running ? "PAUSE" : "START"}}</button>
    <div>time = {{time.toFixed(2)}} s, numCol = {{numCol}}</div>
    <div class="container" v-bind:style="{width: width + 'px', height: height + 'px'}">
      <div v-for="dot of dots" :key="dot.id" class="dot" v-bind:style="{
        left:dot.x + 'px',
        top:dot.y + 'px',
        height: diameter + 'px',
        width: diameter + 'px',
        transitionDuration: dtNext + 's'
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
      // units are px and px/s
      dots: [
        {id: 1, x: 200, y: 300, vx: 40, vy: 4},
        {id: 2, x: 800, y: 300, vx: 0, vy:0},
      ],
      diameter: 100,
      numCol: -1,
      width: 1200,
      height: 800,
      running: true
    }
  },
  beforeDestroy() {
    // prevent memory leak
    clearTimeout(this.timeout)
  },
  methods: {
    willCollide: function (radius, dr, dv) {
      const crossProd = dr[0] * dv[1] - dv[0] * dr[1];
      return crossProd * crossProd < (dv[0] * dv[0] + dv[1] * dv[1]) * radius * radius;
    },
    timeToCollide: function (radius, dr, dv) {
      const crossProd = dr[0] * dv[1] - dv[0] * dr[1];
      const dr2 = dr.reduce((dr2, comp) => dr2 + comp * comp, 0);
      const dv2 = dv.reduce((dv2, comp) => dv2 + comp * comp, 0);
      const theta = Math.asin(Math.abs(crossProd / Math.sqrt(dr2 * dv2)));
      const theta2 = Math.PI - Math.asin(Math.sqrt(dr2) * Math.sin(theta) / radius);
      const theta3 = Math.PI - theta - theta2;
      const ds = Math.sin(theta3) * radius / Math.sin(theta);
      console.log(crossProd, dr2, dv2, theta, theta2, theta3, ds);
      return ds / Math.sqrt(dv2);
    },
    increment: function () {
      // "numCol" means the number of the next collision, w/1-based counting
      if (this.numCol > 0) {
        this.dtLast = this.dtNext;
        let dtMin = Infinity;
        let dv, dr, dt; //, iMin, jMin, dot;
        for (let i = 0; i < this.dots.length; i++) {
          for (let j = i; j < this.dots.length; j++) {
            if (j !== i) {
              dv = [this.dots[i].vx- this.dots[j].vx, this.dots[i].vy- this.dots[j].vy];
              dr = [this.dots[i].x - this.dots[j].x , this.dots[i].y - this.dots[j].y];
              // dot = dv.reduce((dot, vcomp, i) => dot + vcomp * dr[i]);
            // } else {
            //   dv = 2 * this.vs[i];
            //   dx = 2 * ((dv > 0 ? this.width : 0) - this.xs[i]);
              if (this.willCollide(this.diameter, dr, dv)) {
                // console.log(i, " and " , j, "will collide.")
                dt = this.timeToCollide(this.diameter, dr, dv);
                if (dt < dtMin) dtMin = dt;
              }
            }
            // if (Math.sign(dot) === -1) {
            //   // dt = (Math.abs(dx) - this.diameter) / Math.abs(dv);
            //   // if (dt < dtMin)[iMin, jMin, dtMin] = [i, j, dt];
            // }
          }
        }
        this.dots.forEach((dot, i) => {
          this.dots[i].x = this.dots[i].x + this.dots[i].vx * dtMin;
          this.dots[i].y = this.dots[i].y + this.dots[i].vy * dtMin;
        })
        // this.xs = this.xs.map((x, i) => x + this.vs[i] * dtMin);
        this.dtNext = dtMin;
        // if (iMin !== jMin) {
        //   [this.vs[iMin], this.vs[jMin]] = [this.vs[jMin], this.vs[iMin]];
        // } else {
        //   this.vs[iMin] *= -1;
        // }
        // this.time += this.running ? this.dtNext : 0;
      }
      this.numCol++;
      this.timeout = setTimeout(this.increment, this.dtNext * 1000);
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
