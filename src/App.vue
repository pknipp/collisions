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
        <tr><th>diam</th><th>x</th><th>y</th><th>speed</th><th>dir</th></tr>
        </thead>
        <tbody>
          <tr v-for="dot of dots" :key="dot.id">
            <td><input v-if="!running" v-model.number="dot.diameter"><span v-if="running">{{dot.diameter}}</span></td>
            <td><input v-if="!running" v-model.number="dot.rxy[0]"><span v-if="running">{{dot.rxy[0].toFixed(0)}}</span></td>
            <td><input v-if="!running" v-model.number="dot.rxy[1]"><span v-if="running">{{dot.rxy[1].toFixed(0)}}</span></td>
            <td><input v-if="!running" v-model.number="dot.v"><span v-if="running">{{dot.v.toFixed(1)}}</span></td>
            <td><input v-if="!running" v-model.number="dot.theta"><span v-if="running">{{dot.theta.toFixed(1)}}</span></td>
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
      // units are px and px/s
      dots: [
        {id: 1, diameter: 200, rxy: [100, 300], v: 100, vxy: [], theta: 30},
        {id: 2, diameter: 300, rxy: [400, 600], v: 300, vxy: [], theta: 70}
        // {id: 2, diameter: 40, rxy:  [400, 300], vxy: [-100,46]},
        // {id: 3, diameter: 60, rxy:  [600, 200], vxy: [-100, -90]},
        // {id: 4, diameter: 80, rxy:  [700, 500], vxy: [50, 50]},
        // {id: 5, diameter: 100, rxy:[100, 100], vxy: [-40, -80]},
        // {id: 6, diameter: 120, rxy:[200, 300], vxy: [80, -300]},
        // {id: 7, diameter: 140, rxy: [500, 700], vxy: [-100, 200]},
        // {id: 8, diameter: 160, rxy: [700, 700], vxy: [100, 90]},
        // {id: 9, diameter: 180, rxy:[100, 500], vxy: [-50, -50]},
        // {id: 10, diameter: 100, rxy:[690, 100], vxy: [40, -80]},
      ],
      // items: [
      //   { id: 1, username: "Curran Wong", contact: "16690110 3116", email: "Mauris.eu@volutpat.net" },
      //   { id: 2, username: "Donovan Lambert", contact: "16670921 6862", email: "ante.dictum.cursus@natoque.ca" },
      //   { id: 3, username: "Austin Lindsay", contact: "16591004 4485", email: "non.bibendum.sed@dictummi.com" },
      //   { id: 4, username: "Jerome Velazquez", contact: "16060105 2525", email: "magna@Etiamimperdiet.com" },
      //   { id: 5, username: "Alden Hudson", contact: "16880710 2754", email: "torquent@enim.org" },
      //   { id: 6, username: "Gregory Britt", contact: "16260904 9933", email: "nostra.per@velconvallisin.net" },
      //   { id: 7, username: "Jarrod Mcconnell", contact: "16930717 4434", email: "metus@acarcu.org" },
      //   { id: 8, username: "Leonard Pitts", contact: "16690125 6773", email: "a.dui.Cras@utaliquam.com" },
      //   { id: 9, username: "Jamal Sanders", contact: "16070716 6989", email: "Aliquam.erat@odio.edu" },
      //   { id: 10, username: "Armand Barry", contact: "16170519 4759", email: "non.hendrerit@ipsum.edu" }
      // ],
      // columns: [
      //   { name: 'id', text: 'User ID' },
      //   { name: 'username', text: 'Name' },
      //   { name: 'contact', text: 'Contact No.' },
      //   { name: 'email', text: 'Email Address' }
      // ],
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
      this.dots.forEach(dot => {
        let theta = dot.theta * Math.PI / 180;
        dot.vxy[0] = dot.v * Math.cos(theta);
        dot.vxy[1] = dot.v * Math.sin(theta);
      });
      // console.log(this.dots[0].vx, this.dots[0].vy);
      if (this.numCol) {
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
