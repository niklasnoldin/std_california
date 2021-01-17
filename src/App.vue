<template>
  <main>
    <div class="header">
      <img src="/condom.png">
      <h1>STDs in <br>California</h1>
    </div>
  
  <section>
    <v-map :data="data" class="map-container"/>
    <v-bar :data="data" class="bar-container"/>
    <v-line :data="data" class="line-container"/>
    <v-area :data="data" class="area-container"/>
  </section>
  </main>
</template>

<script>
import VMap from "./components/Map";
import VBar from "./components/Bar";
import VLine from "./components/Line";
import VArea from "./components/Area";
import * as d3 from "d3";
export default {
  components: { VMap, VBar, VLine, VArea },
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

<style lang="scss">
  @font-face {
      font-family: 'silkabold';
      font-display: swap;
      src: url('/fonts/Silka-Roman-Webfont/silka-bold-webfont.eot');
      src: url('/fonts/Silka-Roman-Webfont/silka-bold-webfont.eot?#iefix') format('embedded-opentype'),
          url('/fonts/Silka-Roman-Webfont/silka-bold-webfont.woff2') format('woff2'),
          url('/fonts/Silka-Roman-Webfont/silka-bold-webfont.woff') format('woff'),
          url('/fonts/Silka-Roman-Webfont/silka-bold-webfont.ttf') format('truetype');
      font-weight: normal;
      font-style: normal;
  }
  @font-face {
    font-family: 'silkamedium';
    font-display: swap;
    src: url('/fonts/Silka-Roman-Webfont/silka-medium-webfont.eot');
    src: url('/fonts/Silka-Roman-Webfont/silka-medium-webfont.eot?#iefix') format('embedded-opentype'),
         url('/fonts/Silka-Roman-Webfont/silka-medium-webfont.woff2') format('woff2'),
         url('/fonts/Silka-Roman-Webfont/silka-medium-webfont.woff') format('woff'),
         url('/fonts/Silka-Roman-Webfont/silka-medium-webfont.ttf') format('truetype');
    font-weight: normal;
    font-style: normal;
  }
  @font-face {
      font-family: 'silkaregular';
      font-display: swap;
      src: url('/fonts/Silka-Roman-Webfont/silka-regular-webfont.eot');
      src: url('/fonts/Silka-Roman-Webfont/silka-regular-webfont.eot?#iefix') format('embedded-opentype'),
          url('/fonts/Silka-Roman-Webfont/silka-regular-webfont.woff2') format('woff2'),
          url('/fonts/Silka-Roman-Webfont/silka-regular-webfont.woff') format('woff'),
          url('/fonts/Silka-Roman-Webfont/silka-regular-webfont.ttf') format('truetype');
      font-weight: normal;
      font-style: normal;
  }
//-----------------------
  body, main {
    margin: 0;
    font-family: 'silkabold', sans-serif;
  }
  p, a, ul, li, span {
    font-family: 'silkaregular', sans-serif;
    margin: 0;
  }
  h1 {
    font-size: 36px;
  }
  h2, h3, h4, h5, h6 {
    font-family: 'silkamedium', sans-serif;
    margin: 0 0 15px 0;
  }
  $lightgrey: #ededed;
//-----------------------
  .header {
    display: flex;
    align-items: center;
    margin-bottom: 10px;
    padding: 10px 50px;

    img {
      width: 100px;
      margin-right: 30px;
    }
  }

  section {
    display: flex;
    align-items: stretch;
    flex-wrap: wrap;
    margin: 0 40px;

    & > div {
      margin: 15px;
      box-shadow: 0px 0px 60px $lightgrey;
      border-radius: 40px;
      padding: 35px;
      flex-grow: 1;
      display: flex;
      flex-direction: column-reverse;
      justify-content: space-between;
    }

    .info {
      h2 {
        margin-bottom: 5px;
      }
      p {
        margin-bottom: 25px;
      }
    }

    .map-container {
      padding: 35px 30px 0 35px;
    }
    .line-container {
    }
    .bar-container {
    }
    .area-container {
    }
  }
</style>