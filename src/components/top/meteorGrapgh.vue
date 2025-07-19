<template>
  <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
    <div class="text-h4 pa-3" style="font-weight: bold; border-bottom: 2px solid #000;">
      1日に検出した流星数
    </div>
  </v-container>
  <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
    <v-row class="d-flex" style="width: 100%;">
      <v-col v-for="(endpoint, index) in endpoints" :key="index" cols="12" md="4">
        <v-card flat class="graph-card">
          <v-card-title class="text-h6 font-weight-bold pb-0 d-flex align-center justify-center">
            <v-icon class="mr-2" size="large">
              {{ ['mdi-meteor', 'mdi-radio-tower', 'mdi-radio-tower'][index] }}
            </v-icon>
            {{ ['ビデオ', '電波(豊川)', '電波(大津)'][index] }}
          </v-card-title>
          <canvas :id="'barChart' + index" class="bar-chart"></canvas>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from 'axios';
import Chart from 'chart.js/auto';

export default {
  name: 'MeteorGraph',
  props: {
    sidebarOpen: { type: Boolean, default: false },
    endpoints: {
      type: Array,
      default: () => [
        "https://toms-server.tail2925.ts.net/Meteor_count",
        "https://toms-server.tail2925.ts.net/Meteor_toyokawa_count",
        "https://toms-server.tail2925.ts.net/Meteor_otsu_count",
      ],
    },
  },
  data() {
    return {
      lineColors: ['rgba(255, 99, 132, 1)', '#FFC800', '#6495ED'],
      yMinValues: [0, 0, 0],
      yMaxValues: [100, 100, 100],
      interval: null,
    };
  },
  methods: {
    async fetchData(endpoint, index) {
      try {
        const response = await axios.get(endpoint);
        const data = response.data;
        console.log(`Data for ${endpoint}:`, data);
        const values = data.map(entry => entry[1]);
        const minValue = Math.min(...values);
        const maxValue = Math.max(...values);

        this.yMinValues[index] = minValue;
        this.yMaxValues[index] = maxValue;

        const label = ['1晩あたりの流星数', '1日当たりの流星数 (豊川)', '1日当たりの流星数 (大津)'][index];
        this.drawBarChart(index, data, label);
      } catch (error) {
        console.error(`Error fetching data from ${endpoint}:`, error);
      }
    },
    drawBarChart(index, data, label) {
      this.$nextTick(() => {
        const canvas = document.getElementById('barChart' + index);
        if (!canvas) {
          console.error(`Canvas not found for index ${index}`);
          return;
        }

        console.log(`Drawing chart for index ${index}`);
        Chart.getChart(canvas)?.destroy();

        const labels = data.map(entry => entry[0]);
        const values = data.map(entry => entry[1]);

        const mobileScreen = window.innerWidth <= 450;

        const chartData = {
          labels,
          datasets: [{
            label,
            data: values,
            borderColor: this.lineColors[index],
            backgroundColor: this.lineColors[index],
            fill: false
          }]
        };

        const options = {
          maintainAspectRatio: true,
          aspectRatio: mobileScreen ? 5 / 2 : 2 / 1,
          responsive: true,
          plugins: {
            legend: {
              display: !mobileScreen
            }
          },
          scales: {
            x: {
              ticks: {
                display: true,
                maxTicksLimit: mobileScreen ? 5 : 8,
                rotation: 0,  // corrected spelling
                font: {
                  size: mobileScreen ? 10 : 12
                }
              }
            },
            y: {
              min: this.yMinValues[index],
              max: this.yMaxValues[index],
              beginAtZero: false,
              ticks: {
                font: {
                  size: mobileScreen ? 10 : 12
                }
              }
            }
          }
        };

        new Chart(canvas.getContext('2d'), {
          type: 'bar',
          data: chartData,
          options
        });
      });
    },
    initializeCharts() {
      for (let i = 0; i < this.endpoints.length; i++) {
        this.fetchData(this.endpoints[i], i);
      }
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.initializeCharts();
      this.interval = setInterval(() => this.initializeCharts(), 600000); // 10 minutes
    });
  },
  beforeUnmount() {
    if (this.interval) clearInterval(this.interval);
  }
};
</script>

<style scoped>
.graph-card {
  border-radius: 0 !important;
  background-color: white;
  border: 1px solid #212121;
}

.bar-chart {
  width: 100%;
}
</style>