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
      borderColor: 'black',
    }">
      <div v-for="dot of dots" :key="dot.id" class="dot" v-bind:style="{
        left:(dot.Rxyz[0] - dot.Diameter/2) + 'px',
        top:(dot.Rxyz[1] - dot.Diameter/2) + 'px',
        zIndex:dot.rxyz[2],
        height: dot.Diameter + 'px',
        width: dot.Diameter + 'px',
        backgroundColor: 'grey',
        transitionDuration: dt + 's'
      }"></div>
      <div v-for="dot of dots" :key="dot.id" class="dot" v-bind:style="{
        left:(dot.Rxyz[0] - dot.Diameter/2) + 'px',
        top:(dot.Rxyz[1] - dot.Diameter/2) + 'px',
        zIndex:4000,
        height: dot.Diameter + 'px',
        width: dot.Diameter + 'px',
        transitionDuration: dt + 's',
        borderStyle: 'dashed',
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
        {id: 1, diameter: 100, Rxyz: [], rxyz: [800, 400, 0], vxyz: [200, 300, 1000]},
        {id: 2, diameter: 100, Rxyz: [], rxyz: [1100, 100, 700],vxyz: [-200, 0, -200]},
        {id: 3, diameter: 100, Rxyz: [], rxyz: [600, 200, 150], vxyz: [-100, -90, 20]},
        {id: 4, diameter: 100, Rxyz: [], rxyz: [800, 500, 200], vxyz: [50, 50, -30]},
        {id: 5, diameter: 100, Rxyz: [], rxyz: [1100, 100, 100],vxyz: [-40, -80, 50]},
        {id: 6, diameter: 100, Rxyz: [], rxyz: [1200, 300, 600],vxyz: [80, -300, 30]},
        {id: 7, diameter: 100, Rxyz: [], rxyz: [500, 700, 100], vxyz: [-100, 200, 300]},
        {id: 8, diameter: 100, Rxyz: [], rxyz: [700, 700, 300], vxyz: [100, 90, 10]},
        {id: 9, diameter: 100, Rxyz: [], rxyz: [1100, 500, 500],vxyz: [-50, -50, 100]},
        {id: 10,diameter: 100, Rxyz: [], rxyz: [900, 100, 600], vxyz: [40, -80, -200]},
      ],
      numCol: 0,
      dims: [1600, 800, 800],
      running: true
    }
  },
  beforeDestroy() {
    // prevent memory leak
    clearTimeout(this.timeout)
  },
  methods: {
    walls: function(dims, dot) {
      let radius = dot.diameter / 2;
      let [rxyz, vxyz] = [[...dot.rxyz], [...dot.vxyz]];
      let x, y, z, t;
      vxyz.forEach((vcomp, k) => {
        if (vcomp > 0) {
          vxyz[k] *= -1;
          rxyz[k] = dims[k] - rxyz[k];
        }
      })
      // 1st: look at x = 0 plane.
      if (vxyz[0]) {
        t = -(rxyz[0] - radius) / vxyz[0];
        y = rxyz[1] + vxyz[1] * t;
        z = rxyz[2] + vxyz[2] * t;
        if (y >= radius && z >= radius) return {index: 0, t};
      }
      if (vxyz[1]) {
        // 2nd: look at y = 0 plane.
        t = -(rxyz[1] - radius) / vxyz[1];
        x = rxyz[0] + vxyz[0] * t;
        z = rxyz[2] + vxyz[2] * t;
        if (x >= radius && z >= radius) return {index: 1, t};
      }
      // By process of elimination, it must be z = 0 plane.
      if (vxyz[2]) return {index: 2, t: -(rxyz[2] - radius) / vxyz[2]};
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
              dv = doti.vxyz.map((vcomp, k) => vcomp - dotj.vxyz[k]);
              dr = doti.rxyz.map((rcomp, k) => rcomp - dotj.rxyz[k]);
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
            }
            // update the details for the collision which'll occur next
            if (dt < dtMin) [dtMin, iMin, jMin, indexMin] = [dt, i, j, result.index];
          }
        }
        // Change each dot's coordinates until the moment of the next collision.
        this.dots.forEach(dot => {
          let rxyz = dot.rxyz;
          rxyz.forEach((comp, j) => rxyz[j] += dot.vxyz[j] * dtMin);
          let magFactor = 1;
          let mag = magFactor * rxyz[2] / this.dims[2] + 1;
          dot.Diameter = dot.diameter * mag;
          for (let k = 0; k < 2; k++) {
            dot.Rxyz[k] = (rxyz[k] - this.dims[k] / 2) * mag + this.dims[k] / 2;
          }
        });
        this.dots.forEach((dot, i) => console.log("i/z/indexMin = ", i, dot.rxyz[2], indexMin));
        this.dt = dtMin;
        // The collision changes one or more components of one or two dots.
        if (iMin === jMin) {
          // If the collision was with a wall, negate the appropriate component of the relevant dot's velocity.
          this.dots[iMin].vxyz[indexMin] *= -1;
        } else {
          // If the collision is between two dots, the procedure is more complicated.
          let [doti, dotj] = [this.dots[iMin], this.dots[jMin]];
          // First, shift into the center-of-mass frame of the two colliding objects.
          // Assume that each dot's mass is proportional to the square of its diameter ("colliding rods")
          let [massi, massj] = [doti.diameter * doti.diameter, dotj.diameter * dotj.diameter];
          let v_cm = doti.vxyz.map((vcomp, k) => (massi * vcomp + massj * dotj.vxyz[k]) / (massi + massj));
          let vi = doti.vxyz.map((vcomp, k) => vcomp - v_cm[k]);
          let vj = dotj.vxyz.map((vcomp, k) => vcomp - v_cm[k]);

          // Next, find the vector (and its squared magnitude) which points in direction of normal force
          dr = doti.rxyz.map((rcomp, k) => rcomp - dotj.rxyz[k]);
          let dr2 = dr.reduce((dr2, comp) => dr2 + comp * comp, 0);

          // (2x) projection of one momentum along the normal force yields the momentum transfer
          let dot = dr.reduce((dot, rcomp, k) => dot + rcomp * massi * vi[k], 0);
          let dp = dr.map(comp => 2 * comp * dot / dr2);

          // transfer momentum from one object to the other:
          vi = vi.map((comp, k) => comp - dp[k] / massi);
          vj = vj.map((comp, k) => comp + dp[k] / massj);

          // Shift back to lab frame.
          doti.vxyz = vi.map((vcomp, k) => vcomp + v_cm[k]);
          dotj.vxyz = vj.map((vcomp, k) => vcomp + v_cm[k]);
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
  transition-timing-function: linear;
  transition-property: all;
}

</style>
