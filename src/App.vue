<template>
  <div id="app">
    <!-- <img alt="Vue logo" src="./assets/logo.png" /> -->
    <!-- <HelloWorld msg="Welcome to my very first Vue.js App" /> -->
    <button @click="running = !running">{{running ? "PAUSE" : "START"}}</button>
    <div>time = {{time.toFixed(2)}} s, numCol = {{numCol}}</div>
    <div class="container" v-bind:style="{width: width + 'px', height: height + 'px'}">
      <div v-for="dot of dots" :key="dot.id" class="dot" v-bind:style="{
        left:dot.rxy[0] + 'px',
        top:dot.rxy[1] + 'px',
        height: diameter + 'px',
        width: diameter + 'px',
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
        {id: 1, rxy: [200, 300], vxy: [800, .1]},
        {id: 2, rxy: [400, 301], vxy: [0,0]},
        {id: 3, rxy: [600, 200], vxy: [0, 900]},
        {id: 4, rxy: [800, 500], vxy: [500, 500]},
        {id: 5, rxy: [1000, 100], vxy: [400, 800]},
      ],
      diameter: 200,
      numCol: 0,
      width: 1200,
      height: 700,
      running: true,
      wallIndex: 0
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
      // console.log(crossProd, dr2, dv2, theta, theta2, theta3, ds);
      return ds / Math.sqrt(dv2);
    },
    whichWall: function (dot) {
      // walls are numbered 1 (top), 2 (right), 3 (bottom), 4 (left)
      if (dot.vxy[1]>0 && dot.vxy[0]>0) {
        return (dot.vxy[1]*(this.width-this.diameter/2-dot.rxy[0])>dot.vxy[0]*(this.height-this.diameter/2-dot.rxy[1]))?3:2;
      }
      if (dot.vxy[1]>0 && dot.vxy[0]<0) {
        return (dot.vxy[1] * (dot.rxy[0]-this.diameter/2)>-dot.vxy[0]*(this.height-this.diameter/2-dot.rxy[1]))? 3 : 4;
      }
      if (dot.vxy[1]<0 && dot.vxy[0]>0) {
        return (-dot.vxy[1]*(this.width - this.diameter/2 - dot.rxy[0])> dot.vxy[0] * (dot.rxy[1] - this.diameter/2)) ? 1 : 2;
      }
      if (dot.vxy[1]<0 && dot.vxy[0]<0) {
        return (-dot.vxy[1]*(dot.rxy[0]-this.diameter/2)>-dot.vxy[0] * (dot.rxy[1]-this.diameter/2)) ? 1 : 4;
      }
    },
    increment: function () {
      // "numCol" means the number of the next collision, w/1-based counting
      if (this.numCol) {
        // console.log("time = ", this.time, "vxy = ", this.dots[0].vxy);
        // this.dtLast = this.dtNext;
        let dtMin = Infinity;
        let dv, dr, wallIndex, wallIndexMin, iMin, jMin;//, dot;
        for (let i = 0; i < this.dots.length; i++) {
          for (let j = i; j < this.dots.length; j++) {
            let dt = Infinity;
            console.log("(i, j) = ", i, j);
            if (j !== i) {
              dv = this.dots[i].vxy.map((vcomp, k) => vcomp - this.dots[j].vxy[k]);
              dr = this.dots[i].rxy.map((rcomp, k) => rcomp - this.dots[j].rxy[k]);
              console.log("dr = ", dr, " and dv = ", dv);
              if (dv.reduce((dot, vcomp, k) => dot + vcomp * dr[k], 0) < 0) {
                if (this.willCollide(this.diameter, dr, dv)) {
                  dt = this.timeToCollide(this.diameter, dr, dv);
                  console.log("dt = ", dt, "dr = ", dr, "dv = ", dv);
                }
              }
              wallIndex = 0;
            } else {
              wallIndex = this.whichWall(this.dots[i]);
              if (wallIndex === 1) dt = -(this.dots[i].rxy[1] - this.diameter / 2) / this.dots[i].vxy[1];
              if (wallIndex === 3) dt = (this.height - this.diameter / 2 - this.dots[i].rxy[1]) / this.dots[i].vxy[1];
              if (wallIndex === 4) dt = -(this.dots[i].rxy[0] - this.diameter / 2) / this.dots[i].vxy[0];
              if (wallIndex === 2) dt = (this.width - this.diameter / 2 - this.dots[i].rxy[0]) / this.dots[i].vxy[0];
            // dv = 2 * this.vs[i];
            //   dx = 2 * ((dv > 0 ? this.width : 0) - this.xs[i]);
            }
            console.log("(dt, dtMin) are ", dt, dtMin);
            if (dt < dtMin) {
              console.log("That was a shorter time.");
              [dtMin, iMin, jMin, wallIndexMin] = [dt, i, j, wallIndex];
            }
            // if (Math.sign(dot) === -1) {
            //   // dt = (Math.abs(dx) - this.diameter) / Math.abs(dv);
            //   // if (dt < dtMin)[iMin, jMin, dtMin] = [i, j, dt];
            // }
          }
        }
        // Change each dot's coordinates until the moment of the next collision.
        this.dots.forEach((dot, i) => dot.rxy.forEach((comp, j) => this.dots[i].rxy[j] += dot.vxy[j] * dtMin));
        this.dt = dtMin;
        let v_cm;
        if (iMin === jMin) {
          console.log("this was a wall collision");
          // If the collision was with a wall, negate the appropriate component of the relevant dot's velocity.
          this.dots[iMin].vxy[wallIndexMin % 2] *= -1;
        } else {
          // If the collision was between two dots, the procedure is more complicated.
          // First, shift into center-of-mass frame of the two colliding objects.
          v_cm = this.dots[iMin].vxy.map((vcomp, k) => (vcomp + this.dots[jMin].vxy[k])/2);
          let vi = this.dots[iMin].vxy.map((vcomp, k) => vcomp - v_cm[k]);
          let vj = this.dots[jMin].vxy.map((vcomp, k) => vcomp - v_cm[k]);

          // vector (and squared magnitude) which points in direction of normal force
          dr = this.dots[iMin].rxy.map((rcomp, k) => rcomp - this.dots[jMin].rxy[k]);
          let dr2 = dr.reduce((dr2, comp) => dr2 + comp * comp, 0);

          // projection of one velocity along the normal force
          dv = this.dots[iMin].vxy.map((vcomp, k) => vcomp - this.dots[jMin].vxy[k]);
          let dot = dr.reduce((dot, rcomp, k) => dot + rcomp * dv[k], 0);
          dv = dr.map(comp => comp * dot / dr2);
          console.log("dot = ", dot, " dr2 = ", dr2, " dv = ", dv)

          // transfer velocity change from one object to the other:
          vi = vi.map((comp, k) => comp - 1 * dv[k]);
          vj = vj.map((comp, k) => comp + 1 * dv[k]);

          // Shift back to lab frame.
          this.dots[iMin].vxy = vi.map((vcomp, k) => vcomp + v_cm[k]);
          this.dots[jMin].vxy = vj.map((vcomp, k) => vcomp + v_cm[k]);
        }
        // if (wallIndex) {
        //   this.dots[iMin].
        // }
        // if (iMin !== jMin) {
        //   [this.vs[iMin], this.vs[jMin]] = [this.vs[jMin], this.vs[iMin]];
        // } else {
        //   this.vs[iMin] *= -1;
        // }
        this.time += this.running ? this.dt : 0;
      }
      console.log("postcollision velocities are ", this.dots[0].vxy, " and ", this.dots[1].vxy);
      this.numCol++;
      // console.log("this.dt = ", this.dt);
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
