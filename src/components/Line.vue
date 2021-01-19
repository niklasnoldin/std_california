<template>
  <div>
    <div class="info" @mouseout="current = null">
      <h2>Total Number of Cases per Year</h2>
      <p>divided in STDs</p>
      <svg id="line" viewBox="0 0 500 500" width="500" height="500">
        <g transform="translate(50, 20)">
          <g>
            <path
              @mousemove="handleMouseOver"
              stroke="#0f0"
              :fill="
                highlightedDisease && highlightedDisease === 'Chlamydia'
                  ? '#0f03'
                  : 'none'
              "
              stroke-width="2"
              :d="
                highlightedDisease && highlightedDisease === 'Chlamydia'
                  ? area(clamData)
                  : line(clamData)
              "
              data-disease="Chlamydia"
            ></path>
            <path
              @mousemove="handleMouseOver"
              stroke="#f00"
              :fill="
                highlightedDisease && highlightedDisease === 'Early Syphilis'
                  ? '#f003'
                  : 'none'
              "
              stroke-width="2"
              :d="
                highlightedDisease && highlightedDisease === 'Early Syphilis'
                  ? area(syphData)
                  : line(syphData)
              "
              data-disease="Early Syphilis"
            ></path>
            <path
              @mousemove="handleMouseOver"
              stroke="#00f"
              :fill="
                highlightedDisease && highlightedDisease === 'Gonorrhea'
                  ? '#00f3'
                  : 'none'
              "
              stroke-width="2"
              :d="
                highlightedDisease && highlightedDisease === 'Gonorrhea'
                  ? area(gonData)
                  : line(gonData)
              "
              data-disease="Gonorrhea"
            ></path>
            <path
              @mousemove="handleMouseOver"
              data-disease="Total"
              :stroke="
                highlightedDisease && highlightedDisease !== ''
                  ? 'lightgray'
                  : 'black'
              "
              fill="none"
              stroke-width="2"
              :d="line(totalData)"
            ></path>
          </g>
        </g>
        <g id="bottom-axis" transform="translate(50, 420)"></g>
        <g id="left-axis" transform="translate(50,20)"></g>
      </svg>
    </div>
    <div
      id="map_tooltip"
      class="tooltip"
      v-if="current"
      :style="{ top: current.top, left: current.left }"
    >
      <span>{{ current.name }}</span>
    </div>
  </div>
</template>

<script>
import * as d3 from "d3";

export default {
  props: {
    data: { type: Array, required: true },
    highlightedDisease: String,
  },
  data() {
    return {
      scaleY: null,
      scaleX: null,
      line: null,
      area: null,
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
      d3.select("#bottom-axis").call(d3.axisBottom(this.scaleX));
      d3.select("#left-axis").call(d3.axisLeft(this.scaleY));
    },
    updateLine() {
      this.line = d3
        .line()
        .x((d) => {
          return this.scaleX(d3.timeParse("%Y")(d.Year));
        })
        .y((d) => {
          return this.scaleY(d.Cases);
        });
      this.area = d3
        .area()
        .x((d) => {
          return this.scaleX(d3.timeParse("%Y")(d.Year));
        })
        .y0(() => this.scaleY(0))
        .y1((d) => {
          return this.scaleY(d.Cases);
        });
    },
    updateScales() {
      this.scaleY = d3
        .scaleLinear()
        .domain([
          0,
          d3.max(this.totalData, (d) => {
            return d.Cases;
          }),
        ])
        .range([400, 0]);

      this.scaleX = d3
        .scaleTime()
        .domain(d3.extent(this.totalData, (d) => d3.timeParse("%Y")(d.Year)))
        .range([0, 400]);
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
        (row) => row.County === "California" && row.Sex === "Total"
      );
      return [...new Set(filteredData.map((row) => row.Year))].map((year) => {
        return {
          Year: year,
          Cases: filteredData
            .filter((row) => row.Year === year)
            .reduce((total, row) => total + row.Cases, 0),
        };
      });
    },
    clamData() {
      return this.data.filter(
        (row) =>
          row.County === "California" &&
          row.Sex === "Total" &&
          row.Disease === "Chlamydia"
      );
    },
    syphData() {
      return this.data.filter(
        (row) =>
          row.County === "California" &&
          row.Sex === "Total" &&
          row.Disease === "Early Syphilis"
      );
    },
    gonData() {
      return this.data.filter(
        (row) =>
          row.County === "California" &&
          row.Sex === "Total" &&
          row.Disease === "Gonorrhea"
      );
    },
  },
};
</script>

<style lang="scss" scoped>
</style>
<style lang="scss" scoped>
</style>