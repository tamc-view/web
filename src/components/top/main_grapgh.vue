<template>
  <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
    <div class="text-h5 pa-3" style="font-weight: bold; border-bottom: 2px solid #000;">
      現在の気象データ
    </div>
  </v-container>
  <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
    <v-row class="d-flex" style="width: 100%;">
      <v-col v-for="(endpoint, index) in endpoints" :key="index" cols="12" md="4">
        <v-card flat class="graph-card">
          <v-card-title class="text-h6 font-weight-bold pb-0 d-flex align-center justify-center">
            <v-icon class="mr-2" size="large">
              {{ ['mdi-thermometer', 'mdi-water-percent', 'mdi-weather-windy'][index] }}
            </v-icon>
            {{ ['気温', '湿度', '気圧'][index] }}
          </v-card-title>
          <canvas :id="'lineChart' + index" class="line-chart"></canvas>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
  <!-- <v-divider class="border-opacity-100"></v-divider> -->
</template>


<script>
import axios from 'axios';
import Chart from 'chart.js/auto';

export default {
  name: 'MainGraph',
  props: {
    sidebarOpen: { type: Boolean, default: false },
    endpoints: {
      type: Array,
      default: () => [
        "https://toms-server.tail2925.ts.net/latest/temper",
        "https://toms-server.tail2925.ts.net/latest/humidity",
        "https://toms-server.tail2925.ts.net/latest/pressure"
      ]
    }
  },
  data() {
    return {
      lineColors: ['rgba(255, 99, 132, 1)', 'rgba(54, 162, 235, 1)', 'rgba(255, 206, 86, 1)'],
      yMinValues: [-5, 0, 950],
      yMaxValues: [30, 100, 1100],
      interval: null
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

        let label = ['気温', '湿度', '気圧'][index];
        this.drawChart(index, data, label);
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    },
    drawChart(index, data, label) {
      const canvas = document.getElementById('lineChart' + index);
      if (!canvas) {
        console.error(`Canvas not found for index ${index}`);
        return;
      }
      console.log(`Drawing chart for index ${index}`);
      Chart.getChart(canvas)?.destroy();
      const labels = data.map(entry => entry[0]);
      const values = data.map(entry => entry[1]);

      const chartData = {
        labels: labels,
        datasets: [{
          label: label,
          data: values,
          borderColor: this.lineColors[index],
          fill: false
        }]
      };

      const options = {
        maintainAspectRatio: false,
        responsive: true,
        scales: {
          y: {
            min: this.yMinValues[index],
            max: this.yMaxValues[index],
            beginAtZero: false
          }
        }
      };

      const ctx = canvas.getContext('2d');
      new Chart(ctx, {
        type: 'line',
        data: chartData,
        options: options
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
      this.interval = setInterval(() => this.initializeCharts(), 600000);
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

.line-chart {
  width: 100%;
  height: 15vw;
  max-height: 15vw;
}
</style>

