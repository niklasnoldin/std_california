<template>
  <div>
    <div id="bar"></div>
    <div class="info">
      <h2>Total Number of Cases per STD</h2>
      <p>over a period of 18 years (2001 - 2018)</p>
    </div>
  </div>
</template>

<script>
import * as d3 from "d3";
export default {
  props: { data: { type: Array, required: true } },
  data() {
    return {
      svg: null,
      width: 500,
      height: 500,
      xScale: null,
      yScale: null,
    };
  },
  computed: {
    uniqueYears() {
      return [...new Set(this.data.flatMap(({ Year }) => Year))];
    },
    totalPerYear() {
      return [...this.uniqueYears]
        .map((year) => ({
          year,
          total: this.data
            .filter((row) => row.Year === year && row.Sex === "Total")
            .reduce((total, current) => total + current.Cases, 0),
        }))
        .sort((a, b) => a.year - b.year);
    },
  },
  watch: {
    data() {
      this.xScale = d3.scaleLinear().domain([2001, 2018]).range([0, 400]);
      this.yScale = d3
        .scaleLinear()
        .domain([0, d3.max(this.totalPerYear, (d) => d.total)])
        .range([0, 500]);

      const xAxis = d3.axisBottom().scale(this.xScale);

      const yAxis = d3.axisLeft().scale(this.yScale).ticks(10);

      this.svg
        .append("g")
        .attr("class", "x axis")
        .attr("transform", `translate(20, ${this.height - 20})`)
        .call(xAxis)
        .selectAll("text")
        .style("text-anchor", "end")
        .attr("dx", "-.8em")
        .attr("dy", "-.55em")
        .attr("transform", "rotate(-90)");

      this.svg
        .append("g")
        .attr("class", "y axis")
        .call(yAxis)
        .append("text")
        .attr("transform", "rotate(-90)")
        .attr("y", 6)
        .attr("dy", ".71em")
        .style("text-anchor", "end")
        .text("Value ($)");
      this.svg
        .append("g")
        .selectAll("bar")
        .data(this.totalPerYear)
        .enter()
        .append("rect")
        .attr("x", (d) => this.xScale(d.year))
        .attr("y", (d) => 480 - this.yScale(d.total))
        .attr("height", (d) => this.yScale(d.total))
        .attr("width", 7);
    },
  },
  methods: {
    init() {
      this.svg = d3
        .select("#bar")
        .append("svg")
        .attr("viewport", `0 0 ${this.width} ${this.height}`)
        .attr("width", this.width)
        .attr("height", this.height);
    },
  },
  mounted() {
    this.init();
  },
};
</script>

<style lang="scss" scoped>
</style>