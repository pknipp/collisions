<template>
  <div id="app">
    <!-- <img alt="Vue logo" src="./assets/logo.png" /> -->
    <!-- <HelloWorld msg="Welcome to my very first Vue.js App" /> -->
    <!-- <button @click="running = !running">{{running ? "PAUSE" : "START"}}</button> -->
    <div>
      Coefficient of restitution: <input v-model.number="e">
    </div>
    <button @click="() => {if (running = !running) increment()}">
      {{running ? "PAUSE" : "START"}}
    </button>
    <div>time: {{time.toFixed(3)}} s, number of collisions: {{numCol}}</div>
    <div class="container">
      <div class="sphere-container" v-bind:style="{
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
      <table class="inputs">
        <!-- <simple-vue-table :items="items" :columns="columns"></simple-vue-table> -->
        <thead>
          <tr>
            <td colspan='6'><button @click="addOne">Add another particle</button> whose parameters are randomly chosen from the max/min values below.</td></tr>
          <tr v-if="dots.length > 1"><td colspan='6'><button @click="subtractOne">Remove last particle</button></td></tr>
          <tr><th></th>
            <!-- <th>#</th> -->
            <th v-for="col of cols" :key="'heading' + col.id">{{col.name}}</th></tr>
            <tr>
            <td>min<br/>max</td>
            <td v-for="col of cols" :key="'max/min' + col.id">
              <span v-if="running">{{col.min}}</span><input v-else v-model.number="col.min">
              <br/>
              <span v-if="running">{{col.max}}</span><input v-else v-model.number="col.max">
            </td>
          </tr>
        </thead>
        <tbody>
          <tr><td colspan='6'><b>
            Parameters for {{this.dots.length}} particle{{this.dots.length > 1 ? 's' : ''}}
          </b></td></tr>
          <tr v-for="dot of dots" :key="dot.id"><td></td>
            <td v-for="col of cols" :key="'data' + col.id">
              <span v-if="running">{{dot[col.name].toFixed(0)}}</span>
              <input v-else v-model.number="dot[col.name]">
            </td>
          </tr>
        </tbody>
      </table>

    </div>
  </div>
</template>

<script>
// import HelloWorld from "./components/HelloWorld.vue";
// import SimpleVueTable from './simple-vue-table';

export default {
  name: "App",
  // components: {
  //   HelloWorld,
  // },
  // components: {
  //   SimpleVueTable,
  // },
  data() {
    return {
      timeout: null,
      time: 0,
      dt: 0,
      // units are px, px/s, and degrees
      dots: [
        {id: 0, diameter: 100, rxy: [100, 300], v: 100, vxy: [], theta: 30},
      ],
      numCol: 0,
      dims: [800, 800],
      e: 1,
      running: false
    }
  },
  beforeDestroy() {
    // prevent memory leak
    clearTimeout(this.timeout)
  },
  methods: {
    addOne: function () {
      let newDot = {id: this.dots.length};
      this.cols.forEach(col => newDot[col.name] = Math.floor(col.min + (col.max - col.min) * Math.random()));
      newDot.rxy = [newDot.x, newDot.y];
      let theta = newDot.theta * Math.PI / 180;
      newDot.vxy = [Math.cos(theta), Math.sin(theta)];
      newDot.vxy = newDot.vxy.map(vcomp => vcomp * newDot.v);
      this.dots.push(newDot);
    },
    subtractOne: function () {
      this.dots.pop();
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
      if (!this.numCol) {
        this.dots.forEach(dot => {
          let theta = dot.theta * Math.PI / 180;
          dot.vxy = [Math.cos(theta), Math.sin(theta)].map(comp => dot.v * comp);
          // dot.vxy = dot.vxy.map(vcomp => vcomp * dot.v);
        });
        let cols = ["diameter", "x", "y", "v", "theta"];
        let mins =[10, 100, 100, 0, -180];
        let maxs =[2 * this.dots[0].diameter, 700, 700, 2 * this.dots[0].v, 180];
        this.cols = cols.map((col, i) => ({
          id: i,
          name: col,
          min: mins[i],
          max: maxs[i],
        }));
      } else {
        let dtMin = Infinity;
        let dv, dr, wallIndexMin, iMin, jMin; //wallIndex;
        let wallIndex = [];
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
              // set up for use of quadratic formula
              let a = dv.reduce((dv2, comp) => dv2 + comp ** 2, 0);
              let c = dr.reduce((dr2, comp) => dr2 + comp ** 2, 0) - totRadius * totRadius;
              let b = 2 * dr.reduce((dot, comp, k) => dot + comp * dv[k], 0);
              let disc = b * b - 4 * a * c;
              // 1st test if particles are "closing the gap", and then whether collision'll occur
              if (b <= 0 && disc >= 0) dt = 2 * c / (-b + Math.sqrt(disc));
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
          this.dots[iMin].vxy[wallIndexMin[0]] *= -this.e;
        } else {
          // If the collision was between two dots, the procedure is more complicated.
          let [doti, dotj] = [this.dots[iMin], this.dots[jMin]];
          // First, shift into the center-of-mass frame of the two colliding objects.
          // Assume that each dot's mass is proportional to the cube of its diameter
          let [massi, massj] = [doti.diameter ** 3, dotj.diameter ** 3];
          let v_cm = doti.vxy.map((vcomp, k) => (massi * vcomp + massj * dotj.vxy[k]) / (massi + massj));
          let vi = doti.vxy.map((vcomp, k) => vcomp - v_cm[k]);
          let vj = dotj.vxy.map((vcomp, k) => vcomp - v_cm[k]);

          // Next, find the vector (and its squared magnitude) which points in direction of normal force
          dr = doti.rxy.map((rcomp, k) => rcomp - dotj.rxy[k]);
          let dr2 = dr.reduce((dr2, comp) => dr2 + comp * comp, 0);

          // (2x) projection of one momentum along the normal force yields the momentum transfer
          let dot = dr.reduce((dot, rcomp, k) => dot + rcomp * massi * vi[k], 0);
          let dp = dr.map(comp => (1 + this.e) * comp * dot / dr2);

          // transfer momentum from one object to the other:
          vi = vi.map((comp, k) => comp - dp[k] / massi);
          vj = vj.map((comp, k) => comp + dp[k] / massj);

          // Shift back to lab frame.
          doti.vxy = vi.map((vcomp, k) => vcomp + v_cm[k]);
          dotj.vxy = vj.map((vcomp, k) => vcomp + v_cm[k]);
        }
        // Advance the clock.
        this.time += this.dt; //this.running ? this.dt : 0;
      }
      this.numCol++;
      this.dots.forEach(dot => {
        [dot.x, dot.y] = dot.rxy;
        dot.v = Math.sqrt((dot.vxy[0] ** 2) + (dot.vxy[1] ** 2));
        dot.theta = 180 * Math.atan2(dot.vxy[1], dot.vxy[0]) / Math.PI;
      })
      if (this.running) this.timeout = setTimeout(this.increment, this.dt * 1000);
    }
  },
  created() {
    this.increment()
    // this.interval = setInterval(this.increment, this.dt);
    // this.interval = setInterval(() => this.time += this.running ? this.dt / 1000 : 0, this.dt);
    // if (this.running) {
    //   this.interval = setInterval(() => this.time += this.dt / 1000, this.dt);
    // } else {
    //   if (this.time) clearInterval(this.interval);
    // }
  }
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

.container { display: flex; }
.sphere-container { position: relative; }
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
