<template>
  <div id="app">
    <!-- <img alt="Vue logo" src="./assets/logo.png" /> -->
    <!-- <HelloWorld msg="Welcome to my very first Vue.js App" /> -->
    <!-- <button @click="running = !running">{{running ? "PAUSE" : "START"}}</button> -->
    <div>coefficient of restitution: <span v-if="running">{{e}}</span><input v-else v-model.number="e" size="5"></div>
    <button @click="() => {if (running = !running) {
      increment();
    } else {
      message = `PAUSED as numCollision = ${numCollision}`;
      clearTimeout(this.timeout);
    }}">
      {{running ? "PAUSE" : "START"}}
    </button>
    <div>time: {{time.toFixed(3)}} s, number of collisions: {{numCollision}}</div>
    <br/>creator: <a
      href="https://pknipp.github.io/"
      target="_blank"
      rel="noreferrer"
    >
      Peter Knipp
    </a>
    <br/>repo: <a
      href="https://github.com/pknipp/collisions"
      target="_blank"
      rel="noreferrer"
    >
      github.com/pknipp/collisions
    </a>
    <div v-bind:style="{
      width: '100px',
      height: '20px',
      borderTop: '5px', solid, black,
    }"></div>
    <div class="container">
      <div class="sphere-container" v-bind:style="{
        width: dims[0] + 'px',
        height: dims[1] + 'px',
        borderWidth: '1px',
        borderStyle: 'solid',
        borderColor: 'black',
      }">
        <div v-for="dot of dots" :key="dot.id" class="dot" v-bind:style="{
          left:(dot.x - dot.diameter/2) + 'px',
          // External and internal y-coordinates are opposites.
          top:(dims[1] - (dot.y + dot.diameter/2) ) + 'px',
          height: dot.diameter + 'px',
          width: dot.diameter + 'px',
          background: '#' + (256 - dot.density).toString(16).repeat(3),
          transitionDuration: dt + 's'
        }"></div>
      </div>
      <table class="inputs">
        <thead>
          <tr>
            <td colspan='6'>
              <button @click="addOne">Add another particle</button>
              whose parameters are randomly chosen from the max/min values below.
            </td>
          </tr>
          <tr v-if="message"><td colspan='6'>{{message}}</td></tr>
          <tr v-if="dots.length > 1">
            <td colspan='6'>
              <button @click="subtractOne">Remove last particle</button>
            </td>
          </tr>
          <tr><th></th>
            <th v-for="[name, column] of Object.entries(columns)" :key="'heading' + column.id">{{ name}}</th>
          </tr>
          <tr>
            <td>min<br/>max</td>
            <td v-for="column of Object.values(columns)" :key="'max/min' + column.id">
              <span v-if="running">{{column.min}}</span><input v-else v-model.number="column.min" size="5">
              <br/>
              <span v-if="running">{{column.max}}</span><input v-else v-model.number="column.max" size="5">
            </td>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td colspan='6'>
              <b>Parameters for {{this.dots.length}} particle{{this.dots.length > 1 ? 's' : ''}}</b>
            </td>
          </tr>
          <tr v-for="(dot, index) in dots" :key="dot.id">
            <td>
              <button v-if="dots.length > 1" @click="() => subtractSpecificOne(index)">Remove particle</button>
            </td>
            <td v-for="[name, column] of Object.entries(columns)" :key="name + column.id">
              <span v-if="running">{{dot[name].toFixed(0)}}</span>
              <input v-else v-model.number="dot[name]" size="5">
            </td>
          </tr>
        </tbody>
      </table>
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
      // units are px, px/s, and degrees
      dots: [
        {id: 0, density: 128, diameter: 100, x: 100, y: 300, v: 400, theta: 30},
        {id: 1, density: 100, diameter: 60, x: 400, y: 500, v: 200, theta: 120},
        {id: 2, density: 80, diameter: 120, x: 400, y: 100, v: 600, theta: 180},
      ],
      cols: [
        ["density", 1, 256],
        ["diameter", 10, 100],
        ["x", null, null],
        ["y", null, null],
        ["v", 0, 100],
        ["theta", -180, 180],
      ].reduce((columns, col, i) => ({...columns, [col[0]]: {id: i, min: col[1], max: col[2]}}), {}),
      numCollision: 0,
      dims: [900, 800],
      e: 1,
      running: false,
      maxCount: 100,
      message:''
    }
  },
  computed: {
      columns: function () {
        return {...this.cols,
        x: {...this.cols.x,
          min: this.cols.diameter.max / 2,
          max: this.dims[0] - this.cols.diameter.max / 2},
        y: {...this.cols.y,
          min: this.cols.diameter.max / 2,
          max: this.dims[1] - this.cols.diameter.max / 2}
        }
    }
  },
  beforeDestroy() {
    // prevent memory leak
    clearTimeout(this.timeout)
  },
  methods: {
    addOne: function () {
      let newDot = {id: this.dots.length};
      Object.entries(this.columns).filter(([name]) => !['x', 'y', 'diameter'].includes(name)).forEach(([name, col]) => {
        newDot[name] = Math.floor(col.min + (col.max - col.min) * Math.random());
      });
      let count = 0;
      while (count < this.maxCount) {
        newDot.x = Math.floor(this.columns.diameter.max / 2 + (this.dims[0] - this.columns.diameter.max) * Math.random());
        newDot.y = Math.floor(this.columns.diameter.max / 2 + (this.dims[1] - this.columns.diameter.max) * Math.random());
        count++;
        let diameterMax = this.columns.diameter.max;
        this.dots.forEach(dot => {
          let dr2 = [dot.x - newDot.x, dot.y - newDot.y].reduce((dr2, comp) => dr2 + comp * comp, 0);
          diameterMax = Math.min(diameterMax, 2 * Math.sqrt(dr2) - dot.diameter);
        });
        if (diameterMax >= this.columns.diameter.min) {
          newDot.diameter = Math.floor(this.columns.diameter.min + (diameterMax - this.columns.diameter.min) * Math.random());
          let theta = newDot.theta * Math.PI / 180;
          newDot.vxy = [Math.cos(theta), -Math.sin(theta)].map(comp => newDot.v * comp);
          newDot.rxy = [newDot.x, this.dims[1] - newDot.y];
          this.message = '';
          return this.dots.push(newDot);
        }
      }
      this.message = `App was unable to fit an additional particle after ${this.maxCount} attempts.  Try again.`;
    },
    subtractOne: function () {this.dots.pop()},
    subtractSpecificOne: function (index) {this.dots.splice(index, 1)},
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
      if (!this.numCollision) {
        this.dots.forEach(dot => {
          dot.rxy = [dot.x, this.dims[1] - dot.y];
          let theta = dot.theta * Math.PI / 180;
          dot.vxy = [Math.cos(theta), -Math.sin(theta)].map(comp => dot.v * comp);
        });
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
              let c = dr.reduce((dr2, comp) => dr2 + comp ** 2, 0) - totRadius ** 2;
              let b = 2 * dr.reduce((dot, comp, k) => dot + comp * dv[k], 0);
              let disc = b * b - 4 * a * c;
              // 1st test whether particles already overlap
              // 2nd confirm that relative velocity is nonzero
              // 3rd test if particles are approaching (ie, not receding)
              // 4th test whether two particles'll ever collide
              if (c > 0 && a > 0 && b < 0 && disc > 0) dt = 2 * c / (-b + Math.sqrt(disc));
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
        this.dots.forEach(dot => dot.rxy.forEach((blah, j) => dot.rxy[j] += dot.vxy[j] * dtMin));
        this.dt = dtMin;
        // A collision w/a wall means that i = j.
        if (iMin === jMin) {
          // If the collision was with a wall, negate the appropriate component of the relevant dot's velocity.
          this.dots[iMin].vxy[wallIndexMin[0]] *= -this.e;
        } else {
          // If the collision was between two dots, the procedure is more complicated.
          let [doti, dotj] = [this.dots[iMin], this.dots[jMin]];
          // First, shift into the center-of-mass frame of the two colliding objects.
          // Assume that each dot's mass is proportional to the product of its density and the cube of its diameter
          let [massi, massj] = [doti.density * doti.diameter ** 3, dotj.density * dotj.diameter ** 3];
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
      this.numCollision++;
      this.dots.forEach(dot => {
        [dot.x, dot.y] = dot.rxy;
        dot.y = this.dims[1] - dot.y;
        dot.v = Math.sqrt((dot.vxy[0] ** 2) + (dot.vxy[1] ** 2));
        dot.theta = 180 * Math.atan2(dot.vxy[1], -dot.vxy[0]) / Math.PI;
      })
      if (this.running) {
        this.timeout = setTimeout(this.increment, this.dt * 1000);
      } else {
        clearTimeout(this.timeout);
      }
    }
  },
  created() {
    // this.increment()
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
  background-color: #787878;
  border-radius: 50%;
  top: 100px;
  transition-timing-function: linear;
  transition-property: all;
}

</style>
