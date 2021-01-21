<template>
  <div>
    <div id="bar" @mouseout="current = null">
      <svg width="500" height="500" viewBox="-20 0 500 500" id="bar">
        <g transform="translate(60, 20)">
          <rect
            v-for="(disease, idx) in uniqueDiseases"
            :key="'female_' + disease"
            data-name="Female"
            v-bind="{
              height: scaleY(0) - femaleHeights[idx],
              width: 100,
              y: femaleHeights[idx],
              x: scaleX(disease) - 50,
              fill: highlightedDisease && highlightedDisease !== disease
                  ? 'lightgray'
                  : '#BFD04D',
              'data-text': `${disease}: ${femalePerDisease[idx].total.toLocaleString()}`,
            }"
            @mousemove="handleMouseOver"
            @click="$emit('diseaseSelected', disease)"
          ></rect>
          <rect
            v-for="(disease, idx) in uniqueDiseases"
            :key="'male_' + disease"
            :data-text="`${disease}: ${malePerDisease[idx].total.toLocaleString()}`"
            data-name="Male"
            v-bind="{
              height: scaleY(0) - maleHeights[idx],
              width: 100,
              y: maleHeights[idx] - (scaleY(0) - femaleHeights[idx]),
              x: scaleX(disease) - 50,
            }"
            @mousemove="handleMouseOver"
            :fill="
                highlightedDisease && highlightedDisease !== disease
                  ? 'darkgray'
                  : '#084D71'
            "
            @click="$emit('diseaseSelected', disease)"
          ></rect>
        </g>
        <g id="left-axis-bars" transform="translate(60, 20)"></g>
        <g id="bottom-axis-bars" transform="translate(60, 420)"></g>
      </svg>
    </div>
        <div
      id="map_tooltip"
      class="tooltip"
      v-if="current"
      :style="{ top: current.top, left: current.left }"
    >
      <h3>{{ current.name }}</h3>
      <p>{{ current.text }}</p>
    </div>
    <div class="info">
      <h2>Total Number of Cases per STD</h2>
      <p>over a period of 18 years (2001 - 2018)</p>
      <p>and divided in female and male</p>
    </div>
  </div>
</template>

<script>
import * as d3 from "d3";
export default {
  props: { 
    data: { type: Array, required: true },
    highlightedDisease: { type: String }  
  },
  data() {
    return {
      scaleY: null,
      scaleX: null,
      current: null,
    };
  },
  computed: {
    femaleHeights() {
      return this.femalePerDisease.map(({ total }) => this.scaleY(total));
    },
    maleHeights() {
      return this.malePerDisease.map(({ total }) => this.scaleY(total));
    },
    uniqueYears() {
      return [...new Set(this.data.map(({ Year }) => Year))];
    },
    uniqueDiseases() {
      return [...new Set(this.data.map(({ Disease }) => Disease))];
    },
    totalPerDisease() {
      return [...this.uniqueDiseases].map((disease) => ({
        disease,
        total: this.data
          .filter(
            (row) =>
              row.Disease === disease &&
              row.County === "California" &&
              row.Sex === "Total"
          )
          .reduce((total, current) => total + parseInt(current.Cases), 0),
      }));
    },
    malePerDisease() {
      return [...this.uniqueDiseases].map((disease) => ({
        disease,
        total: this.data
          .filter(
            (row) =>
              row.Disease === disease &&
              row.County === "California" &&
              row.Sex === "Male"
          )
          .reduce((total, current) => total + parseInt(current.Cases), 0),
      }));
    },
    femalePerDisease() {
      return [...this.uniqueDiseases].map((disease) => ({
        disease,
        total: this.data
          .filter(
            (row) =>
              row.Disease === disease &&
              row.County === "California" &&
              row.Sex === "Female"
          )
          .reduce((total, current) => total + parseInt(current.Cases), 0),
      }));
    },
  },
  watch: {
    totalPerDisease: {
      handler() {
        this.initScales();
        this.renderAxis();
      },
      immediate: true,
    },
  },
  methods: {
    handleMouseOver(event) {
      this.current = {
        left: event.pageX + "px",
        top: event.pageY + "px",
        name: event.target.dataset.name,
        text: event.target.dataset.text
      };
    },
    initScales() {
      this.scaleX = d3
        .scaleOrdinal()
        .domain(["", ...this.uniqueDiseases, " "])
        .range([0, 75, 200, 325, 400]);
      this.scaleY = d3
        .scaleLinear()
        .domain([0, d3.max(this.totalPerDisease, (d) => d.total)])
        .range([400, 0]);
    },
    renderAxis() {
      let axisX = d3.axisBottom(this.scaleX);
      let axisY = d3.axisLeft(this.scaleY);
      d3.select("#left-axis-bars").call(axisY);
      d3.select("#bottom-axis-bars").call(axisX);
    },
  },
};
</script>

<style lang="scss" scoped>
</style>