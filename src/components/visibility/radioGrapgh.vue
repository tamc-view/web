<template>
  <div>
    <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
      <div class="text-h5 pa-3" style="font-weight: bold; border-bottom: 2px solid #000;">
        検出した流星（ラジオ）
      </div>
    </v-container>
    <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
      <v-row class="d-flex" style="width: 100%;">
        <v-col cols="12">
          <v-card flat class="graph-card">
            <canvas id="barChart" class="bar-chart"></canvas>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import axios from 'axios';
import Chart from 'chart.js/auto';

export default {
  name: 'MeteorRadioGraph',
  props: {
    sidebarOpen: { type: Boolean, default: false },
    endpoints: {
      type: Array,
      default: () => [
        "https://toms-server.tail2925.ts.net/Meteor_toyokawa_count",
        "https://toms-server.tail2925.ts.net/Meteor_otsu_count",
      ],
    },
  },
  data() {
    return {
      meteor_count: [[], []], // Toyokawa と Otsu のデータを格納
      meteor_barColors: ['#FFC800', '#6495ED'], // Toyokawa: 黄色, Otsu: 青
      meteor_labels: ["1日当たりの流星数 (豊川)", "1日当たりの流星数 (大津)"],
      chart: null,
      interval: null,
    };
  },
  methods: {
    async fetchData() {
      try {
        const [toyokawaResponse, otsuResponse] = await Promise.all([
          axios.get(this.endpoints[0]),
          axios.get(this.endpoints[1]),
        ]);
        this.meteor_count[0] = toyokawaResponse.data;
        this.meteor_count[1] = otsuResponse.data;
        this.drawBarChart();
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    },
    drawBarChart() {
      this.$nextTick(() => {
        const canvas = document.getElementById('barChart');
        if (!canvas) {
          console.error("Canvas element not found");
          return;
        }

        if (this.chart) {
          this.chart.destroy();
        }

        const labels = this.meteor_count[0].map(entry => entry[0]); 
        const toyokawaValues = this.meteor_count[0].map(entry => entry[1]);
        const otsuValues = this.meteor_count[1].map(entry => entry[1]);
        
        const mobileScreen = window.innerWidth <= 450;

        const ctx = canvas.getContext('2d');
        this.chart = new Chart(ctx, {
          type: 'bar',
          data: {
            labels: labels,
            datasets: [
              {
                label: this.meteor_labels[0],
                data: toyokawaValues,
                backgroundColor: this.meteor_barColors[0],
                borderWidth: 1,
              },
              {
                label: this.meteor_labels[1],
                data: otsuValues,
                backgroundColor: this.meteor_barColors[1],
                borderWidth: 1,
              },
            ],
          },
          options: {
            responsive: true,
            maintainAspectRatio: true,
            aspectRatio: mobileScreen ? 5/3 : 6/1,
            plugins: {
              legend: {
                display: !mobileScreen,
              },
            },
            scales: {
              y: {
                min: 0,
                max: 100,
                beginAtZero: false,
                font: {
                  size: mobileScreen ? 10 : 12,
                },
              },
              x: {
                ticks: {
                  display: true,
                  maxTicksLimit: mobileScreen ? 10 : 30,
                  font: {
                    size: mobileScreen ? 10 : 12,
                  },
                },
              },
            },
          },
        });
      });
    },
    initializeChart() {
      this.fetchData();
    },
  },
  mounted() {
    this.interval = setInterval(() => {
      this.initializeChart();
    }, 600000); 

    this.initializeChart();
  },
  beforeUnmount() {
    clearInterval(this.interval);
    if (this.chart) {
      this.chart.destroy();
    }
  },
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