<template>
  <main>
    <div class="header">
      <div class="logo">
        <img src="/condom.png" />
        <h1>STDs in <br />California</h1>
      </div>

      <div class="std-filter">
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


    <section>
      <v-map 
        :data="data" 
        class="map-container"
        :highlightedDisease="disease"
      />
      <v-bar 
        :data="data" 
        class="bar-container"
        :highlightedDisease="disease"
        @diseaseSelected="disease=$event"
      />
      <v-line
        :data="data"
        class="line-container"
        :highlightedDisease="disease"
        @diseaseSelected="disease=$event"
      />
      <v-area
        :data="data"
        class="area-container"
        :highlightedDisease="disease"
      />
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
  async mounted() {
    this.data = await d3.csv("./dataset.csv");
    this.data.forEach(function (d) {
      d.Cases = +d.Cases;
      d.Year = +d.Year;
      d.Population = +d.Population;
    });
  },
  data() {
    return {
      data: [],
      disease: "",
    };
  },
};
</script>

<style lang="scss">
@font-face {
  font-family: "silkabold";
  font-display: swap;
  src: url("/fonts/Silka-Roman-Webfont/silka-bold-webfont.eot");
  src: url("/fonts/Silka-Roman-Webfont/silka-bold-webfont.eot?#iefix")
      format("embedded-opentype"),
    url("/fonts/Silka-Roman-Webfont/silka-bold-webfont.woff2") format("woff2"),
    url("/fonts/Silka-Roman-Webfont/silka-bold-webfont.woff") format("woff"),
    url("/fonts/Silka-Roman-Webfont/silka-bold-webfont.ttf") format("truetype");
  font-weight: normal;
  font-style: normal;
}
@font-face {
  font-family: "silkamedium";
  font-display: swap;
  src: url("/fonts/Silka-Roman-Webfont/silka-medium-webfont.eot");
  src: url("/fonts/Silka-Roman-Webfont/silka-medium-webfont.eot?#iefix")
      format("embedded-opentype"),
    url("/fonts/Silka-Roman-Webfont/silka-medium-webfont.woff2") format("woff2"),
    url("/fonts/Silka-Roman-Webfont/silka-medium-webfont.woff") format("woff"),
    url("/fonts/Silka-Roman-Webfont/silka-medium-webfont.ttf")
      format("truetype");
  font-weight: normal;
  font-style: normal;
}
@font-face {
  font-family: "silkaregular";
  font-display: swap;
  src: url("/fonts/Silka-Roman-Webfont/silka-regular-webfont.eot");
  src: url("/fonts/Silka-Roman-Webfont/silka-regular-webfont.eot?#iefix")
      format("embedded-opentype"),
    url("/fonts/Silka-Roman-Webfont/silka-regular-webfont.woff2")
      format("woff2"),
    url("/fonts/Silka-Roman-Webfont/silka-regular-webfont.woff") format("woff"),
    url("/fonts/Silka-Roman-Webfont/silka-regular-webfont.ttf")
      format("truetype");
  font-weight: normal;
  font-style: normal;
}
//-----------------------
body,
main {
  margin: 0;
  font-family: "silkabold", sans-serif;
}
p,
a,
ul,
li,
span {
  font-family: "silkaregular", sans-serif;
  margin: 0;
}
h1 {
  font-size: 30px;
}
h2,
h3,
h4,
h5,
h6 {
  font-family: "silkamedium", sans-serif;
  margin: 0 0 15px 0;
}
.text, 
text {
  font-family: "silkaregular", sans-serif;
  margin: 0;
  font-size: 12px;
}
$lightgrey: #ededed;
$women: #bfd04d;
$men: #084d71;
$std1: #f09d51;
$std2: #de74a5;
$std3: #4dcdd0;
//-----------------------
.header {
  position: fixed;
  background-color: #fff;
  width: 95%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
  padding: 0 50px;
  box-shadow: 0px 0px 20px $lightgrey;

.logo {
    display: flex;
    align-items: center;

    img {
      width: 80px;
      margin-right: 30px;
    }
  }

  .std-filter {
    display: flex;

    label {
      display: flex;
      align-items: center;
      margin: 0 15px;
      font-family: "silkamedium", sans-serif;
      font-size: 18px;

      &:hover {
        cursor: pointer;
      }

      input[type="radio"] {
        width: 20px;
        height: 20px;
        margin-right: 10px;
        margin-top: 0;

        &:hover {
          cursor: pointer;
        }
      }
    }
  }
}
section {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  grid-template-rows: 1fr 1fr;
  justify-items: stretch;
  margin: 0 20px;
  padding-top: 8rem;

  & > div {
    margin: 10px;
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
      margin-bottom: 5px;
    }
  }

  .map-container {
    padding: 35px 30px 0 35px;
    grid-column: 3/4;
    grid-row: 1/2;

    #map {
      display: flex;
      justify-content: center;
    }
  }

  .line-container {
    padding: 35px 30px;
    grid-row: 1/2;
    grid-column: 1/3;

    .tooltip h3 {
      margin: 0;
    }

    #line {
      display: flex;
      justify-content: center;

      path {
        &:hover {
          cursor: pointer;
        }
      }
    }
  }

  .bar-container {
    padding: 35px 30px;
    grid-column: 1/2;
    grid-row: 2/3;

    #bar {
      display: flex;
      justify-content: center;

      rect {
        &:hover {
          cursor: pointer;
        }
      }
    }
  }

  .area-container {
    padding: 35px 30px;
    grid-row: 2/3;
    grid-column: 2/4;

    > h3 {
      text-align: center;
      margin: 15px 0;
    }

    .tooltip h3 {
      margin: 0;
    }

    #area {
      display: flex;
      justify-content: center;
    }
  }
}
</style>