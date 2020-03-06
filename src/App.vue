<template>
  <div id="app">
    <b-jumbotron
      style="padding-top:30px; padding-bottom:30px"
      fluid
      header-level="4"
    >
      <template v-slot:header>
        <div style="font-family: 'Roboto', sans-serif; font-size: 45px">
          COVID-19 Map
        </div>
      </template>
      <hr class="my-4" />
      <template v-slot:lead>
        <span style="font-family: 'Crimson Text', serif; font-size:18px">
          Time-based mapping of the 2019 Wuhan Coronavirus (2019-nCoV) ongoing
          viral epidemic
        </span>
      </template>
    </b-jumbotron>
    <div id="coronamap" ref="chartdiv"></div>
    <!--<b-container style="background-color: white; height:100%" fluid>
      <p>wow</p>
    </b-container>-->
  </div>
</template>

<script>
import axios from "axios";
import _ from "lodash";
import * as csv from "csvtojson";
//import * as am4core from "@amcharts/amcharts4/core";
//import * as am4charts from "@amcharts/amcharts4/charts";
//import * as am4maps from "@amcharts/amcharts4/maps";
//import am4themes_animated from "@amcharts/amcharts4/themes/animated";
//import am4themes_dark from "@amcharts/amcharts4/themes/amchartsdark";
//import am4geodata_worldHigh from "@amcharts/amcharts4-geodata/worldHigh";
export default {
  name: "app",
  components: {},
  data() {
    return {
      dataModel: [],
      confirmed: [],
      deaths: [],
      recovered: [],
      sources: [
        {
          URL:
            "https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_19-covid-Deaths.csv",
          Content: "deaths"
        },
        {
          URL:
            "https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_19-covid-Recovered.csv",
          Content: "recovered"
        },
        {
          URL:
            "https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_19-covid-Confirmed.csv",
          Content: "confirmed"
        }
      ]
    };
  },
  methods: {
    getData: function() {
      let vm = this;
      return new Promise(resolve => {
        var done = _.after(3, function() {
          resolve("Data Retrieved");
        });
        this.sources.forEach(function(t) {
          axios
            .get(t.URL)
            .then(response => {
              csv({
                colParser: {
                  "Province/State": "omit",
                  Lat: "omit",
                  Long: "omit"
                },
                checkType: true
              })
                .fromString(response.data)
                .then(response => {
                  // Consolidates duplicate countries but no longer needed...
                  //let countries = [];
                  //response.map(function(confirmed) {
                  //  if (
                  //    countries.some(
                  //      country => country.Title == confirmed["Country/Region"]
                  //    ) == false
                  //  ) {
                  //    countries.push({
                  //      Title: confirmed["Country/Region"]
                  //    });
                  //  }
                  //  let country = confirmed["Country/Region"];
                  //  delete confirmed["Country/Region"];
                  //  for (var key of Object.keys(confirmed)) {
                  //    if (!countries.find(c => c.Title == country)[key]) {
                  //      countries.find(c => c.Title == country)[key] = 0;
                  //    }
                  //    countries.find(c => c.Title == country)[key] +=
                  //      confirmed[key];
                  //  }
                  //});
                  vm[t.Content] = response;
                  done();
                });
            })
            .catch(error => {
              console.log(error);
            });
        });
      });
    },
    generateModel: function() {
      return new Promise(resolve => {
        const done = resolve("Model Generated");
        let countries = [];
        this.confirmed.forEach(function(x) {
          countries.push(x["Country/Region"]);
        });
        let dates = _.drop(Object.keys(this.confirmed[0]), 1);
        let data = [];
        dates.forEach(function(x) {
          data.push({
            date: x
          });
        });
        data.forEach(function(z) {
          countries.forEach(function(x) {
            z[x] = {
              confirmed: 0,
              recovered: 0,
              deaths: 0
            };
          });
        });
        this.dataModel = _.chain(data)
          .keyBy("date")
          .mapValues(v => _.omit(v, "date"))
          .value();
        done();
      });
    },
    populateModel: function() {
      let vm = this;
      let modelUpdate = this.dataModel;
      this.sources.forEach(function(p) {
        vm[p.Content].forEach(function(n) {
          let country = n["Country/Region"];
          delete n["Country/Region"];
          for (const [key, value] of Object.entries(n)) {
            modelUpdate[key][country][p.Content] =
              modelUpdate[key][country][p.Content] + value;
          }
        });
      });
      this.dataModel = modelUpdate;
      console.log(this.dataModel);
    }
  },
  mounted() {
    let vm = this;
    this.getData().then(function() {
      vm.generateModel().then(function() {
        vm.populateModel();
      });
    });
  },
  beforeDestroy() {
    if (this.chart) {
      this.chart.dispose();
    }
  }
};
</script>

<style lang="scss">
@import url("https://fonts.googleapis.com/css?family=Crimson+Text|Roboto&display=swap");
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
}
#coronamap {
  width: 100%;
  height: 900px;
  background: #222;
}
</style>
