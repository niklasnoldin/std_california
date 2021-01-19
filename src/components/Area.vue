<template>
  <div>
    <div class="info" @mouseout="current = null">
      <h2>Cases over the Years</h2>
      <p>divided in female and male</p>
      <svg id="line" viewBox="0 0 500 500" width="500" height="500">
        <g transform="translate(50, 20)">
          <g>
            <path
              @mousemove="handleMouseOver"
              fill="aqua"
              data-disease="Male"
              :d="areaMale(totalData)"
            ></path>
            <path
              @mousemove="handleMouseOver"
              fill="tomato"
              data-disease="Female"
              :d="areaFemale(totalData)"
            ></path>
          </g>
        </g>
        <g id="bottom-axis-area" transform="translate(50, 420)"></g>
        <g id="left-axis-area" transform="translate(50,20)"></g>
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
    <div>
      <label
        ><input v-model="disease" type="radio" name="disease" value="" /> All
      </label>
      <label
        ><input
          v-model="disease"
          type="radio"
          name="disease"
          value="Early Syphilis"
        />
        Early Syphilis
      </label>
      <label
        ><input
          v-model="disease"
          type="radio"
          name="disease"
          value="Chlamydia"
        />
        Chlamydia
      </label>
      <label
        ><input
          v-model="disease"
          type="radio"
          name="disease"
          value="Gonorrhea"
        />
        Gonorrhea
      </label>
    </div>
  </div>
</template>

<script>
import * as d3 from "d3";

export default {
  props: {
    data: { type: Array, required: true },
  },
  data() {
    return {
      scaleY: null,
      scaleX: null,
      areaFemale: null,
      areaMale: null,
      current: null,
      disease: "",
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
      this.scaleY = d3.scaleLinear().domain([0, 1]).range([400, 0]);

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
    disease(val) {
      this.$emit("setHighlightDisease", val);
    },
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
          (!this.disease || this.disease === row.Disease)
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