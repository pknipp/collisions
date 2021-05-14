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
        height: dot.diameter + 'px',
        width: dot.diameter + 'px',
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
        {id: 1, diameter: 50, rxy: [200, 300], vxy: [80, 3]},
        {id: 2, diameter: 100, rxy: [500, 300], vxy: [100,2]},
        {id: 3, diameter: 150, rxy: [600, 200], vxy: [100, 90]},
        {id: 4, diameter: 50, rxy: [800, 500], vxy: [500, 50]},
        {id: 5, diameter: 100, rxy: [1000, 100], vxy: [40, 800]},
        {id: 6, diameter: 50, rxy: [1200, 300], vxy: [80, 3]},
        {id: 7, diameter: 100, rxy: [500, 700], vxy: [100,2]},
        {id: 8, diameter: 150, rxy: [700, 700], vxy: [100, 90]},
        {id: 9, diameter: 50, rxy: [1100, 500], vxy: [500, 50]},
        {id: 10, diameter: 100, rxy: [1000, 100], vxy: [40, 800]},
      ],
      numCol: 0,
      dims: [1700, 800],
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
      if (!dv2) return Infinity;
      const theta = Math.asin(Math.abs(crossProd / Math.sqrt(dr2 * dv2)));
      const theta2 = Math.PI - Math.asin(Math.sqrt(dr2) * Math.sin(theta) / radius);
      const theta3 = Math.PI - theta - theta2;
      // Might sin(theta) ever vanish?
      const ds = Math.sin(theta3) * radius / Math.sin(theta);
      return ds / Math.sqrt(dv2);
    },
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
    increment: function () {
      // "numCol" means the number of the next collision, w/1-based counting
      if (this.numCol) {
        let dtMin = Infinity;
        let dv, dr, wallIndex, wallIndexMin, iMin, jMin;
        for (let i = 0; i < this.dots.length; i++) {
          for (let j = i; j < this.dots.length; j++) {
            let dt = Infinity;
            if (j !== i) {
              let doti = this.dots[i];
              let dotj = this.dots[j];
              // relative velocity and position (vectors) of two dots
              dv = doti.vxy.map((vcomp, k) => vcomp - dotj.vxy[k]);
              dr = doti.rxy.map((rcomp, k) => rcomp - dotj.rxy[k]);
              let totRadius = (doti.diameter + dotj.diameter) / 2;
              // coarse requirement which must be met, if inter-dot collision is to occur.
              if (dv.reduce((dot, vcomp, k) => dot + vcomp * dr[k], 0) < 0) {
                // precise requirement for inter-dot collision
                if (this.willCollide(totRadius, dr, dv)) {
                  dt = this.timeToCollide(totRadius, dr, dv);
                  console.log(dt, totRadius, dr, dv);
                }
              }
            } else {
              wallIndex = this.whichWall(this.dots[i]);
              if (wallIndex[1]) {
                // reflects from top & bottom walls
                dt = (this.dims[wallIndex[0]] - this.dots[i].diameter / 2 - this.dots[i].rxy[wallIndex[0]]) / this.dots[i].vxy[wallIndex[0]];
              } else {
                // reflects from right & left walls
                dt = -(this.dots[i].rxy[wallIndex[0]] - this.dots[i].diameter / 2) / this.dots[i].vxy[wallIndex[0]];
              }
            }
            // update the details for the collision which'll occur next
            if (dt < dtMin) [dtMin, iMin, jMin, wallIndexMin] = [dt, i, j, [...wallIndex]];
          }
        }
        // Change each dot's coordinates until the moment of the next collision.
        this.dots.forEach((dot, i) => dot.rxy.forEach((comp, j) => this.dots[i].rxy[j] += dot.vxy[j] * dtMin));
        this.dt = dtMin;
        if (iMin === jMin) {
          // If the collision was with a wall, negate the appropriate component of the relevant dot's velocity.
          this.dots[iMin].vxy[wallIndexMin[0]] *= -1;
        } else {
          // If the collision was between two dots, the procedure is more complicated.
          // First, shift into the center-of-mass frame of the two colliding objects.
          let [doti, dotj] = [this.dots[iMin], this.dots[jMin]];
          // Assume that each dot's mass is proportional to the square of its diameter ("colliding rods")
          let [massi, massj] = [doti.diameter * doti.diameter, dotj.diameter * dotj.diameter];
          let v_cm = doti.vxy.map((vcomp, k) => (massi * vcomp + massj * dotj.vxy[k]) / (massi + massj));
          let vi = this.dots[iMin].vxy.map((vcomp, k) => vcomp - v_cm[k]);
          let vj = this.dots[jMin].vxy.map((vcomp, k) => vcomp - v_cm[k]);

          // Next, find the vector (and its squared magnitude) which points in direction of normal force
          dr = this.dots[iMin].rxy.map((rcomp, k) => rcomp - this.dots[jMin].rxy[k]);
          let dr2 = dr.reduce((dr2, comp) => dr2 + comp * comp, 0);

          // projection of one momentum along the normal force yields the momentum transfer
          let dot = dr.reduce((dot, rcomp, k) => dot + rcomp * massi * vi[k], 0);
          let dp = dr.map(comp => 2 * comp * dot / dr2);

          // transfer momentum from one object to the other:
          vi = vi.map((comp, k) => comp - dp[k] / massi);
          vj = vj.map((comp, k) => comp + dp[k] / massj);

          // Shift back to lab frame.
          this.dots[iMin].vxy = vi.map((vcomp, k) => vcomp + v_cm[k]);
          this.dots[jMin].vxy = vj.map((vcomp, k) => vcomp + v_cm[k]);
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
