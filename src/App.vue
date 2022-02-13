<template>
  <div id="app">
    <div class="scrollable">
       <Chart :styles="myStyles" :chart-data="datacollection" :options="options"></Chart>
    </div>
    <div class="form-wrapper"><form><input type="text" v-model="value"><button type="button" @click="fillData()">更新</button></form></div>
  </div>
</template>

<script>
import Chart from './components/Chart.vue'

const dataColors = [
  'rgba(27, 210, 55)',
  'rgba(93, 43, 201)',
  'rgba(213, 157, 71, 0.8)',
  'rgba(155, 148, 221, 0.8)',
  'rgba(186, 42, 147, 0.8)'
]

const resultColors = [
  'rgba(255, 0, 0)',
  'rgba(0, 255, 0, 0.8)',
  'rgba(0, 0, 255, 0.8)'
]

export default {
  name: 'App',
  components: {
    Chart
  },
  data () {
    return {
      val: 0,
      val2: 0,
      index: 0,
      datacollection: {},
      options: {
        responsive: true,
        maintainAspectRatio: false,
        // height: 100,
        scales: {
          yAxes: [{
            ticks: {
                min: -80,
                max: -30,
            }
          }],
          xAxes: [{
            ticks: {
              stepSize: 1000000,
              callback: function(label, index, labels) {
                return (label - labels[0]) / 1000000;
              }
            }
          }]
        },
        onClick: (e, el) => {
          if (el[0]._datasetIndex === 0) {
            this.val += parseInt(this.datacollection.datasets[el[0]._datasetIndex].data[el[0]._index].x)
          } else {
            this.val2 += parseInt(this.datacollection.datasets[el[0]._datasetIndex].data[el[0]._index].x)
            this.index++
          }
          console.log(this.val,this.val2, (this.val2-this.val)/this.index)
        }
      },
      chartWidth: window.innerWidth-50,
      chartHeight: window.innerHeight-100,
      value: ''
    }
  },
  mounted () {
  },
  methods: {
    extend() {
      this.chartWidth += 100;
    },
    async fillData () {
      const [gid, db] = this.value.split('@')
      const [log, result] = await Promise.all([
        fetch(`/log/${gid}?db=${db}`).then(res => res.json()),
        fetch(`/result/${gid}?db=${db}`).then(res => res.json())
      ])
      // console.log(log)
      let maxWidth = 0;
      result.forEach((d, i) => {
        d['radius'] = 6
        d['backgroundColor'] = resultColors[i%resultColors.length]
        d['borderWidth'] = 0
        d['order'] = 99
      })
      log.forEach((d, i) => {
        d['radius'] = 2
        d['backgroundColor'] = dataColors[i%dataColors.length]
        d['borderWidth'] = 0
      })
      this.datacollection = {
        datasets: [...result, ...log]
      }
      const right = (() => {
        const d0 = Math.floor(log[0]['data'][log[0]['data']['length']-1]['x']/1000000)+1
        // const d1 = Math.floor(log[1]['data'][log[1]['data']['length']-1]['x']/1000000)+1
        // return d0 > d1 ? d0 : d1
        return d0
      })()
      const left = (() => {
        const d0 = Math.floor(log[0]['data'][0]['x']/1000000)
        // const d1 = Math.floor(log[1]['data'][0]['x']/1000000)
        // return d0 > d1 ? d1 : d0
        return d0
      })()

      this.chartWidth = (right - left) * 30
      this.chartHeight = 700
    }
  },
  computed: {
    myStyles () {
      return {
        width: `${this.chartWidth}px`,
        height: `${this.chartHeight}px`,
        position: 'relative'
      }
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
.form-wrapper {
  display: inline-block;
  margin-top: 20px;
}
form {
  display: flex;
  background: lavender;
  border-radius: 5px;
  height: 1.8rem;
  width: 500px
}
form input {
  border: none;
  outline: none;
  background: transparent;
  flex: 1;
  padding: 0 5px;
}
form button {
  border: none;
  background: transparent;
  cursor: pointer;
  font-weight: bold;
  color: white;
  background: midnightblue;
  border-radius: 0 5px 5px 0;
  padding: 0 10px;
}
.scrollable {
  overflow: scroll;
}
</style>
