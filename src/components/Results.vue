<template>
  <div class="results-container">
    <div class="svg-container">
      <svg ref="svg" class="w-full h-96"></svg>
      <div class="absolute top-0 left-0 p-1">
        <div class="blue-green-test-result-color">
          <p class="result-text bg-white bg-opacity-70 p-1 rounded"><i>Your</i> green</p>
        </div>
      </div>
      <div class="absolute top-0 right-0 p-1">
        <div class="blue-green-test-result-color">
          <p class="result-text bg-white bg-opacity-70 p-1 rounded"><i>Your</i> blue</p>
        </div>
      </div>
    </div>
    <div class="blue-green-test-result-text w-full mt-0 bg-white">
      <p class="result-text">
        <i>Your</i> boundary is at hue {{ Math.round(userThreshold) }},
        <span v-if="greenInclusive > 0.55">
        </span>
      </p>
    </div>
  </div>
</template>

<script>
import * as d3 from 'd3'

export default {
  props: {
    binPosition: Array,
    count: Array,
    xCdf: Array,
    yCdf: Array,
    userThreshold: Number
  },
  computed: {
    currentColor() {
      return `hsl(${this.userThreshold}, 100%, 50%)`
    },
    greenInclusive() {
      const index = this.xCdf.findIndex((value) => value > this.userThreshold)
      const greenInclusive = index !== -1 ? this.yCdf[index] : 1
      return greenInclusive
    }
  },
  mounted() {
    this.createPlot()
  },
  methods: {
    handleResize() {
      this.createPlot()
    },
    createPlot() {
      const svg = d3.select(this.$refs.svg)
      svg.selectAll('*').remove()

      const width = this.$refs.svg.clientWidth
      const height = this.$refs.svg.clientHeight
      const margin = { top: 0, right: 0, bottom: 0, left: 0 }
      const innerWidth = width - margin.left - margin.right
      const innerHeight = height - margin.top - margin.bottom

      const range_l = 155
      const range_r = 205
      const x = d3.scaleLinear().domain([range_l, range_r]).range([0, innerWidth])
      const y = d3.scaleLinear().domain([0, 1]).range([innerHeight, 0])

      const g = svg.append('g').attr('transform', `translate(${margin.left},${margin.top})`)

      const defs = svg.append('defs')
      const gradient = defs.append('linearGradient').attr('id', 'hue-gradient').attr('x1', '0%').attr('y1', '0%').attr('x2', '100%').attr('y2', '0%')

      for (let i = range_l; i <= range_r; i++) {
        gradient.append('stop').attr('offset', `${((i - range_l) / (range_r - range_l)) * 100}%`).attr('stop-color', `hsl(${i}, 100%, 50%)`)
      }

      g.append('rect').attr('width', innerWidth).attr('height', innerHeight).attr('fill', 'url(#hue-gradient)')

      const line = d3.line().x((d) => x(d[0])).y((d) => y(d[1]))
      g.append('path').datum(d3.zip(this.xCdf, this.yCdf)).attr('fill', 'none').attr('stroke', 'black').attr('stroke-width', 1).attr('d', line)

      const userThresholdX = x(this.userThreshold)
      g.append('line')
        .attr('x1', userThresholdX)
        .attr('x2', userThresholdX)
        .attr('y1', 0)
        .attr('y2', innerHeight)
        .attr('stroke', 'black')
        .attr('stroke-width', 3)
        .attr('stroke-dasharray', '5,5')

      g.append('g').attr('transform', `translate(0,${innerHeight})`).call(d3.axisBottom(x).ticks(5))
      g.append('g').call(d3.axisLeft(y).ticks(5).tickFormat(d3.format('.0%')))
    }
  },
  beforeUnmount() {
    window.removeEventListener('resize', this.handleResize)
  }
}
</script>

<style src="./BlueGreenTest.css" scoped />
<style scoped>
.results-container {
  display: flex;
  flex-direction: column;
  height: 100%;
  width: 100%;
  padding: 0;
  margin: 0;
}

.svg-container {
  flex-grow: 1;
  position: relative;
  width: 100%;
}

svg {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.color-chip {
  display: inline-block;
  width: 1em;
  height: 1em;
  background-color: turquoise;
  border: 2px solid black;
  border-radius: 0.2em;
  margin-bottom: -0.2em;
}
</style>
