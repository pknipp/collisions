<template>
  <div id="app">
    <!-- <img alt="Vue logo" src="./assets/logo.png" /> -->
    <!-- <HelloWorld msg="Welcome to my very first Vue.js App" /> -->
    <button @click="running = !running">{{running ? "PAUSE" : "START"}}</button>
    <div>time = {{time.toFixed(2)}} s, numCol = {{numCol}}</div>
    <div class="container" v-bind:style="{
      width: dims[0] + 'px',
      height: dims[1] + 'px',
      borderWidth: '1px',
      borderStyle: 'solid',
      borderColor: 'black',
    }">
      <div v-for="dot of dots" :key="dot.id" class="dot" v-bind:style="{
        left:(dot.rxy[0] - dot.diameter/2) + 'px',
        top:(dot.rxy[1] - dot.diameter/2) + 'px',
        zIndex:dot.rxy[2],
        height: dot.diameter + 'px',
        width: dot.diameter + 'px',
        backgroundColor: 'white',
        transitionDuration: dt + 's'
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
      time: 0,
      dt: 0,
      // units are px and px/s
      dots: [
        {id: 1, diameter: 300, rxy: [300, 400, 500], vxy: [200, 150, 0]},
        // {id: 2, diameter: 300, rxy: [500, 500, 400], vxy: [3, 4, 5]},
        // {id: 3, diameter: 100, rxy: [600, 200, 150], vxy: [-100, -90, 20]},
        // {id: 4, diameter: 100, rxy: [800, 500, 200], vxy: [50, 50, -30]},
        // {id: 5, diameter: 100, rxy: [1100, 100, 100], vxy: [-40, -80, 50]},
        // {id: 6, diameter: 120, rxy: [1200, 300, 600], vxy: [80, -300, 30]},
        // {id: 7, diameter: 140, rxy: [500, 700, 100], vxy: [-100, 200, 300]},
        // {id: 8, diameter: 160, rxy: [700, 700, 300], vxy: [100, 90, 10]},
        // {id: 9, diameter: 180, rxy: [1100, 500, 500], vxy: [-50, -50, 100]},
        // {id: 10, diameter: 200, rxy: [900, 100, 600], vxy: [40, -80, -200]},
      ],
      numCol: 0,
      dims: [800, 800, 800],
      running: true
    }
  },
  beforeDestroy() {
    // prevent memory leak
    clearTimeout(this.timeout)
  },
  methods: {
    whichWall: function (dot) {
      // 0th index: whether wall is vertical or horizontal
      // 1st index: whether wall represents min or max value of coordinate
      // DRY up following code?
      if (dot.vxy[1]>=0 && dot.vxy[0]>=0) {
        return (dot.vxy[1]*(this.dims[0]-dot.diameter/2-dot.rxy[0])>dot.vxy[0]*(this.dims[1]-dot.diameter/2-dot.rxy[1]))? [ 1, 1] : [0, 1];
      }
      if (dot.vxy[1]>=0 && dot.vxy[0]<0) {
        return (dot.vxy[1] * (dot.rxy[0]-dot.diameter/2)>-dot.vxy[0]*(this.dims[1]-dot.diameter/2-dot.rxy[1]))? [1, 1]: [0, 0];
      }
      if (dot.vxy[1]<0 && dot.vxy[0]>=0) {
        return (-dot.vxy[1]*(this.dims[0] - dot.diameter/2 - dot.rxy[0])> dot.vxy[0] * (dot.rxy[1] - dot.diameter/2)) ? [1, 0] : [0, 1];
      }
      if (dot.vxy[1]<0 && dot.vxy[0]<0) {
        return (-dot.vxy[1]*(dot.rxy[0]-dot.diameter/2)>-dot.vxy[0] * (dot.rxy[1]-dot.diameter/2)) ? [1, 0] : [0, 0];
      }
    },
    walls: function(dims, dot) {
      let radius = dot.diameter / 2;
      let [rxy, vxy] = [[...dot.rxy], [...dot.vxy]];
      let x, y, z, t;
      vxy.forEach((vcomp, k) => {
        if (vcomp > 0) {
          vxy[k] *= -1;
          rxy[k] = dims[k] - rxy[k];
        }
      })
      // 1st: look at x = 0 plane.
      if (vxy[0]) {
        t = -(rxy[0] - radius) / vxy[0];
        y = rxy[1] + vxy[1] * t;
        z = rxy[2] + vxy[2] * t;
        console.log("y = ", y)
        if (y >= radius && z >= radius) return {index: 0, t};
      }
      if (vxy[1]) {
        // 2nd: look at y = 0 plane.
        t = -(rxy[1] - radius) / vxy[1];
        x = rxy[0] + vxy[0] * t;
        z = rxy[2] + vxy[2] * t;
        if (x >= radius && z >= radius) return {index: 1, t};
      }
      // By process of elimination, it must be z = 0 plane.
      if (vxy[2]) return {index: 2, t: -(rxy[2] - radius) / vxy[2]};
    },
    increment: function () {
      // "numCol" means the number of the next collision, w/1-based counting
      if (this.numCol) {
        let dtMin = Infinity;
        let dv, dr, result, iMin, jMin, indexMin;
        for (let i = 0; i < this.dots.length; i++) {
          for (let j = i; j < this.dots.length; j++) {
            let dt = Infinity;
            let doti = this.dots[i];
            if (j !== i) {
              let dotj = this.dots[j];
              // relative velocity and position (vectors) of two dots
              dv = doti.vxy.map((vcomp, k) => vcomp - dotj.vxy[k]);
              dr = doti.rxy.map((rcomp, k) => rcomp - dotj.rxy[k]);
              let totRadius = (doti.diameter + dotj.diameter) / 2;
              // set up for use of quadratic formula
              let a = dv.reduce((dv2, comp) => dv2 + comp ** 2, 0);
              let c = dr.reduce((dr2, comp) => dr2 + comp ** 2, 0) - totRadius * totRadius;
              let b = 2 * dr.reduce((dot, comp, k) => dot + comp * dv[k], 0);
              let disc = b * b - 4 * a * c;
              // 1st test if 2 particles are "closing the gap", and then whether collision'll occur
              if (b <= 0 && disc >= 0) dt = 2 * c / (-b + Math.sqrt(disc));
            } else {
              result = this.walls(this.dims, doti);
              dt = result.t;
              // wallIndex = this.whichWall(this.dots[i]);
              // if (wallIndex[1]) {
              //   // reflects from top & bottom walls
              //   dt = (this.dims[wallIndex[0]] - this.dots[i].diameter / 2 - this.dots[i].rxy[wallIndex[0]]) / this.dots[i].vxy[wallIndex[0]];
              // } else {
              //   // reflects from right & left walls
              //   dt = -(this.dots[i].rxy[wallIndex[0]] - this.dots[i].diameter / 2) / this.dots[i].vxy[wallIndex[0]];
              // }
            }
            // update the details for the collision which'll occur next
            if (dt < dtMin) [dtMin, iMin, jMin, indexMin] = [dt, i, j, result.index];
          }
        }
        // Change each dot's coordinates until the moment of the next collision.
        this.dots.forEach((dot, i) => dot.rxy.forEach((comp, j) => this.dots[i].rxy[j] += dot.vxy[j] * dtMin));
        this.dots.forEach((dot, i) => console.log("i/z/indexMin = ", i, dot.rxy[2], indexMin));
        this.dt = dtMin;
        // The collision changes one or more components of one or two dots.
        if (iMin === jMin) {
          // If the collision was with a wall, negate the appropriate component of the relevant dot's velocity.
          // this.dots[iMin].vxy[wallIndexMin[0]] *= -1;
          this.dots[iMin].vxy[indexMin] *= -1;
        } else {
          // If the collision was between two dots, the procedure is more complicated.
          let [doti, dotj] = [this.dots[iMin], this.dots[jMin]];
          // First, shift into the center-of-mass frame of the two colliding objects.
          // Assume that each dot's mass is proportional to the square of its diameter ("colliding rods")
          let [massi, massj] = [doti.diameter * doti.diameter, dotj.diameter * dotj.diameter];
          let v_cm = doti.vxy.map((vcomp, k) => (massi * vcomp + massj * dotj.vxy[k]) / (massi + massj));
          let vi = doti.vxy.map((vcomp, k) => vcomp - v_cm[k]);
          let vj = dotj.vxy.map((vcomp, k) => vcomp - v_cm[k]);

          // Next, find the vector (and its squared magnitude) which points in direction of normal force
          dr = doti.rxy.map((rcomp, k) => rcomp - dotj.rxy[k]);
          let dr2 = dr.reduce((dr2, comp) => dr2 + comp * comp, 0);

          // (2x) projection of one momentum along the normal force yields the momentum transfer
          let dot = dr.reduce((dot, rcomp, k) => dot + rcomp * massi * vi[k], 0);
          let dp = dr.map(comp => 2 * comp * dot / dr2);

          // transfer momentum from one object to the other:
          vi = vi.map((comp, k) => comp - dp[k] / massi);
          vj = vj.map((comp, k) => comp + dp[k] / massj);

          // Shift back to lab frame.
          doti.vxy = vi.map((vcomp, k) => vcomp + v_cm[k]);
          dotj.vxy = vj.map((vcomp, k) => vcomp + v_cm[k]);
        }
        // Advance the clock.
        this.time += this.running ? this.dt : 0;
      }
      this.numCol++;
      this.timeout = setTimeout(this.increment, this.dt * 1000);
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
