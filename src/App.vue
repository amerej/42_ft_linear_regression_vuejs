<template>
  <div id="app">
    <input type="file" @change="processFile($event)">
    <div id="tester"></div>
    <div id="errorsCost"></div>
  </div>
</template>

<script>
import Papa from 'papaparse'

export default {
  name: 'app',
  data () {
    return {
      x: [],
      y: [],
      xCost: [],
      yCost: [],
      learningRate: 0.01,
      initialTheta0: 0.0,
      initialTheta1: 0.0,
      nbIterations: 2000,
      m: 0,
      errorsCost: []
    }
  },
  methods: {
    drawPlot (res) {
      var trace1 = {
        x: this.x,
        y: this.y,
        mode: 'markers',
        name: 'Sell price',
        marker: { size: 12, color: '#FF6B6B' }
      }
      var trace2 = {
        x: [18000, 248000],
        y: [res.theta0 + ((res.theta1 * 0) / 10000), res.theta0 + ((res.theta1 * 250000) / 10000)],
        mode: 'lines',
        name: 'Linear (Price)',
        line: {
          dash: 'dashdot',
          color: '#4ECDC4',
          width: 4
        }
      }
      var layout = {
        title:'Mileage vs. Price',
        xaxis: {
          title: 'Mileage',
          titlefont: { family: 'Arial, sans-serif', size: 18, color: 'lightgrey'
          }
        },
        yaxis: {
          title: 'Price',
          titlefont: { family: 'Arial, sans-serif', size: 18, color: 'lightgrey' }
        },
        margin: { l: 200, r: 200, b: 100, t: 100 }
      }
      Plotly.newPlot(tester, [ trace1, trace2 ], layout)
    },
    drawCost () {
      for (let i = 0; i < this.errorsCost.length; ++i) {
        this.xCost.push(i)
        this.yCost.push(this.errorsCost[i] / 10000)
      }
      var trace = {
        x: this.xCost,
        y: this.yCost,
        mode: 'dot',
        name: 'test',
        line: {
          dash: 'dashdot',
          color: '#4ECDC4',
          width: 2
        }
      }
      var layout = {
        title:'Precision',
        xaxis: {
          title: 'Iterations',
          titlefont: { family: 'Arial, sans-serif', size: 18, color: 'lightgrey'
          }
        },
        yaxis: {
          title: 'Global error cost',
          titlefont: { family: 'Arial, sans-serif', size: 18, color: 'lightgrey' }
        },
        margin: { l: 200, r: 200, b: 100, t: 100 }
      }
      Plotly.newPlot(errorsCost, [ trace ], layout)
    },
    processFile (event) {
      var self = this
      Papa.parse(event.target.files[0], {
        dynamicTyping: true,
        header: true,
        complete: function(results) {
          for (let entry of results.data) {
            if (Number.isInteger(entry.km) && Number.isInteger(entry.price)) {
              self.y.push(entry.price)
              self.x.push(entry.km)
            }
          }
          self.m = self.x.length
          let res = self.gradientDescent()
          self.drawPlot(res)
          self.drawCost()
        }
      })
    },
    calcDerivates (theta0, theta1) {
      let derivateTheta0 = 0.0
      let derivateTheta1 = 0.0
      for (let i = 0; i < this.m; ++i) {
        derivateTheta0 += theta0 + (theta1 * (this.x[i] / 10000)) - this.y[i]
        derivateTheta1 += (theta0 + (theta1 * (this.x[i] / 10000)) - this.y[i]) * (this.x[i] / 10000)
      }
      derivateTheta0 = (1 / this.m) * derivateTheta0
      derivateTheta1 = (1 / this.m) * derivateTheta1
      return {
        theta0: derivateTheta0,
        theta1: derivateTheta1
      }
    },
    updateTheta (theta0, theta1) {
      let derivate = this.calcDerivates(theta0, theta1)
      let updateTheta0 = theta0 - (this.learningRate * derivate.theta0)
      let updateTheta1 = theta1 - (this.learningRate * derivate.theta1)
      this.errorsCost.push(this.calcCost(updateTheta0, updateTheta1))
      return {
        theta0: updateTheta0,
        theta1: updateTheta1
      }
    },
    gradientDescent () {
      let tmpTheta0 = this.initialTheta0
      let tmpTheta1 = this.initialTheta1
      for (let i = 0; i < this.nbIterations; ++i) {
        let updated = this.updateTheta(tmpTheta0, tmpTheta1)
        tmpTheta0 = updated.theta0
        tmpTheta1 = updated.theta1
      }
      return {
        theta0: tmpTheta0,
        theta1: tmpTheta1
      }
    },
    calcCost (theta0, theta1) {
      let globalCost = 0
      for (let i = 0; i < this.m; ++i) {
        globalCost += ((theta0 + (theta1 * (this.x[i] / 10000))) - this.y[i]) * ((theta0 + (theta1 * (this.x[i] / 10000))) - this.y[i])
      }
      return (1 / (this.m)) * globalCost
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
