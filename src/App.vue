<template>
  <div id="app">
    <BarChart v-if="!loading" :data="barData" />
    <LineChart v-if="!loading" :data="lineData" />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import BarChart from "./components/BarChart";
import LineChart from "./components/LineChart";

interface ChartDataset {
  label: string;
  backgroundColor: string;
  fill?: boolean;
  borderColor?: string[];
  data: number[];
}

interface ChartData {
  labels: string[];
  datasets: ChartDataset[];
}

interface Data {
  date: string;
  newCases: number;
}

@Component({
  components: { BarChart, LineChart },
})
export default class App extends Vue {
  barData: ChartData = {
    labels: [],
    datasets: [
      {
        label: "London",
        backgroundColor: "#f87979",
        data: [],
      },
    ],
  };
  lineData: ChartData = {} as ChartData;
  loading = true;
  mounted() {
    const url = "https://api.coronavirus.data.gov.uk/v1/data?";
    const filters = "filters=areaName=london;areaType=region&";
    const structure =
      "structure={'date':'date','newCases':'newCasesByPublishDate'}";
    const fullurl = url + filters + structure;

    this.$http
      .get(
        'https://api.coronavirus.data.gov.uk/v1/data?filters=areaName=london;areaType=region&structure={"date":"date","newCases":"newCasesByPublishDate"}'
      )
      .then((res) => {
        console.log("res:", res.data.data);
        res.data.data.forEach((d: Data) => {
          if (d.newCases && d.date !== "2020-07-01") {
            this.barData.labels.push(d.date);
            this.barData.datasets[0].data.push(d.newCases);
          }
          if (d.newCases > 6000) {
            console.log("weird one:", d);
          }
        });
        this.barData.labels = this.barData.labels.reverse();
        this.barData.datasets[0].data = this.barData.datasets[0].data.reverse();
        this.lineData = this.calcRollingAverage(res.data.data);
        this.loading = false;
      });
  }

  calcRollingAverage(data: Data[]): ChartData {
    let rollingData: number[] = [];
    let labels: string[] = [];
    for (let i = 6; i < data.length - 6; i++) {
      let total = 0;
      for (let x = i; x >= i - 6; x--) {
        if (data[x].date !== "2020-07-01") {
          total += data[x].newCases;
        }
      }
      if (total !== 0) {
        rollingData.push(Math.round(total / 7));
        labels.push(data[i].date);
      }
    }
    rollingData = rollingData.reverse();
    labels = labels.reverse();

    console.log("rolling average:", rollingData);
    return {
      labels: labels,
      datasets: [
        {
          label: "London",
          backgroundColor: "#f87979",
          fill: false,
          borderColor: ["red"],
          data: rollingData,
        },
      ],
    };
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
