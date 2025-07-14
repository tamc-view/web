<template>
    <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
      <div class="text-h5 pa-3" style="font-weight: bold; border-bottom: 2px solid #000;">
        検出した流星
      </div>
    </v-container>
    <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
      <v-row class="d-flex" style="width: 100%;">
        <v-col v-for="(endpoint, index) in endpoints" :key="index" cols="12">
          <v-card flat class="graph-card">
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
  name: 'MainGraph',
  props: {
    sidebarOpen: { type: Boolean, default: false },
    endpoints: {
      type: Array,
      default: () => [
        "https://toms-server.tail2925.ts.net/Meteor_count",
      ]
    }
  },
  data() {
    return {
      meteor_count: [],
      meteor_barColor: '#FFC800', 
      meteor_label: "1晩あたりの流星数",
      charts: [],
      interval: null,
    };
  },
  methods: {
    async fetchData(endpoint, index) {
      try {
        const response = await axios.get(endpoint);
        this.meteor_count[index] = response.data;
        this.drawBarChart(this.meteor_count[index], this.meteor_label, index);
      } catch (error) {
        console.error(`Error fetching data from ${endpoint}:`, error);
      }
    },
    drawBarChart(data, label, index) {
      this.$nextTick(() => {
        const canvas = document.getElementById(`barChart${index}`);
        if (!canvas) {
          console.error(`Canvas element not found for index ${index}`);
          return;
        }

        if (this.charts[index]) {
          this.charts[index].destroy();
        }

        const labels = data.map(entry => entry[0]);
        const values = data.map(entry => entry[1]);

        const mobileScreen = window.innerWidth <= 450;

        const ctx = canvas.getContext('2d');
        this.charts[index] = new Chart(ctx, {
          type: 'bar',
          data: {
            labels: labels,
            datasets: [{
              label: label,
              data: values,
              backgroundColor: this.meteor_barColor,
              borderWidth: 1
            }]
          },
          options: {
            responsive: true,
            maintainAspectRatio: true,
            aspectRatio: mobileScreen ? 5/2 : 6/1,
            plugins: {
              legend: {
                display: !mobileScreen,
              }
            },
            scales: {
              y: {
                min: 0,
                max: 100,
                beginAtZero: false,
                // ticks: {
                //   display: !mobileScreen,
                // }
                font: {
                  size: mobileScreen ? 10 : 12
                }
              },
              x: {
                ticks: {
                  display: true,
                  maxTicksLimit: mobileScreen ? 10 : 30,
                  Rotation: 0,
                  font : {
                    size: mobileScreen ? 10 : 12
                  }
                }
              }
            }
          }
        });
      });
    },
    initializeCharts() {
      this.endpoints.forEach((endpoint, index) => {
        this.fetchData(endpoint, index);
      });
    }
  },
  mounted() {
    this.interval = setInterval(() => {
      this.initializeCharts();
    }, 600000);

    this.initializeCharts();
  },
  beforeUnmount() {
    clearInterval(this.interval);
    this.charts.forEach(chart => {
      if (chart) chart.destroy();
    });
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
  