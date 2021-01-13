<template>
  <div>
    <div id="map"></div>
    <div
      id="map_tooltip"
      class="tooltip"
      v-if="current"
      :style="{ top: current.top, left: current.left }"
    >
      <p>{{ current.name }}</p>
      <p>{{ current.perMil }} cases per million and year</p>
      <p>{{ current.total }} Absolute Cases</p>
      <p>{{ current.syph }} Early Syphillis Cases</p>
      <p>{{ current.gon }} Gonorrhea Cases</p>
      <p>{{ current.clam }} Chlamydia Cases</p>
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
      caliGeo: null,
      width: 500,
      height: 500,
      tooltip: null,
      current: null,
    };
  },
  computed: {
    viewport() {
      return `0 0 ${this.width} ${this.height}`;
    },
    counties() {
      const uniqueCounties = [
        ...new Set(this.data.flatMap(({ County }) => County)),
      ];
      return uniqueCounties;
    },
    perCounty() {
      const perCounty = {};
      for (const county of this.counties) {
        const filteredData = this.data.filter(
          (row) => row.County === county && row.Sex === "Total"
        );
        perCounty[county] = {
          total: filteredData.reduce((total, row) => total + row.Cases, 0),
          clam: filteredData
            .filter((row) => row.Disease === "Chlamydia")
            .reduce((total, row) => total + row.Cases, 0),
          syph: filteredData
            .filter((row) => row.Disease === "Early Syphilis")
            .reduce((total, row) => total + row.Cases, 0),
          gon: filteredData
            .filter((row) => row.Disease === "Gonorrhea")
            .reduce((total, row) => total + row.Cases, 0),
        };
      }
      return perCounty;
    },
    casesPer1000() {
      const perCounty = {};
      for (const county of this.counties) {
        const filteredData = this.data.filter(
          (row) => row.County === county && row.Sex === "Total"
        );
        perCounty[county] = filteredData.reduce(
          (total, row) => total + (row.Cases / row.Population / 18) * 1000,
          0
        );
      }
      return perCounty;
    },
  },
  watch: {
    data() {
      this.svg
        .append("g")
        .selectAll("path")
        .data(this.caliGeo.features)
        .enter()
        .append("path")
        .attr("d", this.path())
        .attr("fill", (d) => {
          const casesPer1000 = this.casesPer1000[d.properties.name];
          return d3.interpolateBlues(
            d3
              .scaleLinear()
              .domain(d3.extent(Object.values(this.casesPer1000)))
              .range([0, 1])(casesPer1000)
          );
        })
        .attr("stroke", "white")
        .attr("stroke-width", 0.5)
        .on("mouseover mousemove", this.handleMouseOver);
      d3.select("g").on("mouseout", this.handleMouseOut);
    },
  },
  methods: {
    handleMouseOver(event, d) {
      this.current = {
        name: d.properties.name,
        perMil: this.casesPer1000[d.properties.name] * 1000,
        total: this.perCounty[d.properties.name].total,
        syph: this.perCounty[d.properties.name].syph,
        gon: this.perCounty[d.properties.name].gon,
        clam: this.perCounty[d.properties.name].clam,
        left: event.pageX + "px",
        top: event.pageY + "px",
      };
    },
    handleMouseOut() {
      this.current = null;
    },
    path() {
      return d3.geoPath().projection(
        d3
          .geoMercator()
          .center([-120, 37])
          .translate([this.width / 2, this.height / 2])
          .scale([this.width * 4.3])
      );
    },
    async init() {
      this.caliGeo = await d3.json("./cali.json");
      this.tooltip = d3.select("#map_tooltip");

      this.svg = d3
        .select("#map")
        .append("svg")
        .attr("viewport", this.viewport)
        .attr("width", this.width)
        .attr("height", this.height);
    },
  },
  mounted() {
    this.init();
  },
};
</script>

<style>
main {
  margin: 2rem;
}

.tooltip {
  position: absolute;
  background: white;
  box-shadow: 2px 0 5px #0004;
  transition: all 100ms;
  padding: 0.5em 1em;
  border-radius: 5px;
  border-top-left-radius: 0;
  pointer-events: none;
}
p {
  margin: 0;
}
</style>