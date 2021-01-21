<template>
  <div>
    <h3>{{ highlightedDisease || "All STDs combined"}} </h3>
    <div id="area"  @mouseout="current = null">
      <svg viewBox="20 60 650 160" width="900" height="480">
        <g transform="translate(50, 20)">
          <g>
            <path
              @mousemove="handleMouseOver"
              fill="#084d71"
              data-disease="Male"
              :d="areaMale(totalData)"
            ></path>
            <path
              @mousemove="handleMouseOver"
              fill="#bfd04d"
              data-disease="Female"
              :d="areaFemale(totalData)"
            ></path>
          </g>
        </g>
        <g id="bottom-axis-area" transform="translate(50, 270)"></g>
        <g id="left-axis-area" transform="translate(50,20)"></g>
      </svg>
    </div>
    <div
      id="map_tooltip"
      class="tooltip"
      v-if="current"
      :style="{ top: current.top, left: current.left }"
    >
      <h3>{{ current.name }}</h3>
    </div>
    <div class="info">
      <h2>Ratio between sexes</h2>
      <p>divided in female and male</p>
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
      areaFemale: null,
      areaMale: null,
      current: null,
    };
  },
  methods: {
    handleMouseOver(event) {
      this.current = {
        left: event.pageX + "px",
        top: event.pageY + "px",
        name: event.target.dataset.disease,
      };
    },
    renderAxis() {
      d3.select("#bottom-axis-area").call(d3.axisBottom(this.scaleX));
      d3.select("#left-axis-area").call(d3.axisLeft(this.scaleY));
    },
    updateLine() {
      this.areaFemale = d3
        .area()
        .x((d) => {
          return this.scaleX(d3.timeParse("%Y")(d.Year));
        })
        .y0(this.scaleY(0))
        .y1((d) => {
          return this.scaleY(d.Percent);
        });
      this.areaMale = d3
        .area()
        .x((d) => {
          return this.scaleX(d3.timeParse("%Y")(d.Year));
        })
        .y0(this.scaleY(1))
        .y1((d) => {
          return this.scaleY(d.Percent);
        });
    },
    updateScales() {
      this.scaleY = d3.scaleLinear().domain([0, 1]).range([250, 0]);

      this.scaleX = d3
        .scaleTime()
        .domain(d3.extent(this.totalData, (d) => d3.timeParse("%Y")(d.Year)))
        .range([0, 600]);
    },

    render() {
      this.updateScales();
      this.updateLine();
      this.renderAxis();
    },
  },
  watch: {
    totalData: {
      handler() {
        this.render();
      },
      immediate: true,
    },
  },
  computed: {
    totalData() {
      const filteredData = this.data.filter(
        (row) =>
          row.County === "California" &&
          (row.Sex === "Male" || row.Sex === "Female") &&
          (!this.highlightedDisease || this.highlightedDisease === row.Disease)
      );
      return [...new Set(filteredData.map((row) => row.Year))].map((year) => {
        const male = filteredData
          .filter((row) => row.Sex === "Male" && row.Year === year)
          .reduce((total, row) => total + row.Cases, 0);
        const female = filteredData
          .filter((row) => row.Sex === "Female" && row.Year === year)
          .reduce((total, row) => total + row.Cases, 0);
        return {
          Year: year,
          Percent: female / (female + male),
        };
      });
    },
  },
};
</script>