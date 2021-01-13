<template>
  <main>
    <h1>Sexually transmitted deseases in California</h1>
    <p>
      These data contain case counts and rates for sexually transmitted diseases
      (chlamydia, gonorrhea, and early syphilis which includes primary,
      secondary, and early latent syphilis) reported for California residents,
      by disease, county, year, and sex.
    </p>
    <v-map :data="data" />
    <v-line :data="data" />
  </main>
</template>

<script>
import VMap from "./components/Map";
import VLine from "./components/Line";
import * as d3 from "d3";
export default {
  components: { VMap, VLine },
  methods: {
    async loadData() {
      this.data = await d3.csv("./dataset.csv");
      this.data.forEach(function (d) {
        d.Cases = +d.Cases;
        d.Year = +d.Year;
        d.Population = +d.Population;
      });
    },
  },
  mounted() {
    this.loadData();
  },
  data() {
    return {
      data: [],
    };
  },
};
</script>