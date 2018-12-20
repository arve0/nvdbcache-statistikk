<template>
  <div id="app">
    <div id="performance">
      <h2>Timings</h2>
      <table>
        <tr>
          <th>Minimum</th>
          <td>{{perf.min}}</td>
        </tr>
        <tr>
          <th>Maximum</th>
          <td>{{perf.max}}</td>
        </tr>
        <tr>
          <th>Average</th>
          <td>{{perf.average}}</td>
        </tr>
        <tr>
          <th>Total transfer time</th>
          <td>{{perf.total}}</td>
        </tr>
      </table>
    </div>

    <div id="selector">
      <label for="spread">Spre over mx</label>
      <input name="spread" type="checkbox" v-model="spread">
      <label for="zoom">Zoom</label>
      <select name="zoom" id="zoom" v-model="zoom">
        <option v-for="n in 16" :key="n" :value="n">{{n}}</option>
      </select>
      <label for="x-from">X fra</label>
      <select name="x-from" id="x-from" v-model="x_from">
        <option v-for="n in 1000" :key="n" :value="n">{{n}}</option>
      </select>
      <label for="x-to">X til</label>
      <select name="x-to" id="x-to" v-model="x_to">
        <option v-for="n in 1000" :key="n" :value="n">{{n}}</option>
      </select>
      <label for="y-from">Y fra</label>
      <select name="y-from" id="y-from" v-model="y_from">
        <option v-for="n in 1000" :key="n" :value="n">{{n}}</option>
      </select>
      <label for="y-to">Y til</label>
      <select name="y-to" id="y-to" v-model="y_to">
        <option v-for="n in 1000" :key="n" :value="n">{{n}}</option>
      </select>
    </div>

    <div v-for="y in ys" :key="y" class="row">
      <LoadingImage v-for="x in xs" :key="x" :src="imgURL(x, y)" :style="max_width"/>
    </div>
  </div>
</template>

<script>
import LoadingImage from "./LoadingImage.vue";

export default {
  data: function() {
    return {
      zoom: 10,
      x_from: 505,
      x_to: 525,
      y_from: 368,
      y_to: 383,
      statistics: {
        min: 10000,
        max: 0,
        average: 0,
        sum: 0
      },
      spread: true,
    };
  },
  mounted: function() {
    setInterval(() => {
      this.statistics = performance
        .getEntriesByType("resource")
        .filter(p => p.initiatorType === "img")
        .map(({ duration, fetchStart, responseEnd }) => ({
          duration,
          fetchStart,
          responseEnd
        }))
        .map(p => {
          if (!p.duration) {
            p.duration = p.responseEnd - p.fetchStart;
          }
          return p;
        })
        .reduce(
          (acc, value, i) => {
            acc.sum += value.duration;
            acc.average = acc.sum / (i + 1);
            acc.min = Math.min(value.duration, acc.min);
            acc.max = Math.max(value.duration, acc.max);
            return acc;
          },
          {
            min: 10000,
            max: 0,
            average: 0,
            sum: 0
          }
        );
    }, 100);
  },
  computed: {
    ys: function() {
      return Math.abs(this.y_to - this.y_from);
    },
    xs: function() {
      return Math.abs(this.x_to - this.x_from);
    },
    max_width: function() {
      return `max-width:${100 / this.xs}%`;
    },
    perf: function() {
      return {
        min: this.prettyTime(this.statistics.min),
        max: this.prettyTime(this.statistics.max),
        average: this.prettyTime(this.statistics.average),
        total: this.prettyTime(this.statistics.sum)
      };
    }
  },
  methods: {
    imgURL: function(x, y) {
      var i = this.x_from + x;
      var j = this.y_from + y;

      var prefix = this.spread ? `m${(x % 10) + 1}-` : "";

      return `https://${prefix}nvdbcache.geodataonline.no/arcgis/rest/services/Trafikkportalen/GeocacheTrafikkJPG/MapServer/tile/${
        this.zoom
      }/${j}/${i}`;
    },
    prettyTime: t =>
      t < 1000 ? Math.floor(t) + " ms" : Math.floor(t / 10) / 100 + " s"
  },
  components: {
    LoadingImage
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 20px;
}
.row {
  padding: 0;
  max-width: 90%;
  margin-left: 5%;
  margin-right: 5%;
}
.row img {
  vertical-align: bottom;
}
table {
  margin-left: auto;
  margin-right: auto;
  margin-bottom: 20px;
}
tr:nth-child(2n) {
  background-color: lightblue;
}
#selector {
  margin-bottom: 20px;
}
label {
  margin-left: 8px;
  margin-right: 8px;
  font-weight: bold;
}
</style>
