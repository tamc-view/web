<template>
  <div>
    <v-container class="mx-auto d-flex align-center justify-start overflow-visible">
      <div class="text-h5 pa-3" style="font-weight: bold; border-bottom: 2px solid #000;">
        黒点画像（Seestar）
      </div>
    </v-container>
    <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
      <v-card :class="{ 'parent-card': true, 'sidebar-open': isSidebarOpen }" class="d-flex flex-column">
        <v-card-text class="pa-10">
          <!-- 元画像と検出画像を横に並べる -->
          <div class="d-flex justify-space-around">
            <!-- 元画像 -->
            <div class="image-container">
              <div class="text-h6 text-center mb-2">元画像</div>
              <div class="crop-container">
                <img
                  :src="latestSunImage"
                  class="cropped-image"
                  alt="Original Sun Image"
                >
              </div>
            </div>
            <!-- 検出画像 -->
            <div class="image-container">
              <div class="text-h6 text-center mb-2">検出画像</div>
              <div class="crop-container">
                <img
                  :src="contourSunImage"
                  class="cropped-image"
                  alt="Contour Sun Image"
                >
              </div>
            </div>
          </div>
          <!-- 最新観測日と黒点面積 -->
          <v-card-title class="flex-column align-start mt-6">
            <div class="text-h4 mb-2">
              最新の観測日: {{ latestSunInfo.date }}
              <span class="ml-4">黒点面積: {{ latestSunInfo.total_area }} pixel </span>
            </div>
          </v-card-title>
        </v-card-text>
      </v-card>
    </v-container>
  </div>
</template>

<script>
import axios from 'axios';
import { Chart } from 'chart.js/auto';

export default {
  name: 'NightSky',
  data() {
    return {
      latestSunImage: "",      // 元画像のURL
      contourSunImage: "",     // 検出画像のURL
      latestSunInfo: {},       // APIからのレスポンス全体を保持
    };
  },
  methods: {
    async fetchSunInfos() {
      try {
        const response = await axios.get("https://toms-server.tail2925.ts.net/manualReport/sunspot/info");
        this.latestSunInfo = response.data;
        this.latestSunImage = `https://toms-server.tail2925.ts.net${response.data.image_url}`;
        this.contourSunImage = `https://toms-server.tail2925.ts.net${response.data.contour_image_url}`;
      } catch (error) {
        console.error("Error fetching latest sunspot data:", error);
      }
    },
    drawSunspotChart(sunspotNum, avgTemp, label) {
      this.$nextTick(() => {
        const canvas = document.getElementById('barChart');
        if (!canvas) {
          console.error('Canvas element not found');
          return;
        }
        if (this.chart) {
          this.chart.destroy();
        }
        const ctx = canvas.getContext('2d');
        this.chart = new Chart(ctx, {
          type: 'bar',
          data: {
            labels: label,
            datasets: [
              {
                type: 'bar',
                label: 'Relative SunSpot Number',
                data: sunspotNum,
                backgroundColor: this.meteor_barColor,
                borderWidth: 1,
                yAxisID: 'y'
              },
              {
                type: 'line',
                label: 'Average Temperature',
                data: avgTemp,
                borderColor: 'rgba(255, 99, 132, 1)',
                backgroundColor: 'rgba(255, 99, 132, 0.2)',
                borderWidth: 2,
                fill: false,
                yAxisID: 'y1'
              }]
          },
          options: {
            responsive: true,
            maintainAspectRatio: false,
            scales: {
              y: {
                type: 'linear',
                position: 'left',
                min: 0,
                max: 200,
                beginAtZero: false
              },
              y1: {
                type: 'linear',
                position: 'right',
                min: this.y1MinValues_sunspot,
                max: this.y1MaxValues_sunspot,
                beginAtZero: false,
                grid: {
                  drawOnChartArea: false
                }
              }
            }
          }
        });
      });
    },
    async sunspotChart() {
      try {
        const sunspots = await axios.get("https://toms-server.tail2925.ts.net/manualReport/sunspot/list");
        const dateList = await axios.get("https://toms-server.tail2925.ts.net/manualReport/sunspot/datelist");
        const temps = await axios.get("https://toms-server.tail2925.ts.net/thesedays/avgTemp");
        this.sunspot_count = sunspots.data;
        this.sunspotDates = dateList.data;
        this.temp_datas = temps.data;

        const aryMax = function (a, b) { return Math.max(a, b); };
        const aryMin = function (a, b) { return Math.min(a, b); };
        this.yMaxValues_sunspot = this.sunspot_count.reduce(aryMax);
        this.yMinValues_sunspot = this.sunspot_count.reduce(aryMin);
        this.y1MaxValues_sunspot = this.temp_datas.reduce(aryMax);
        this.y1MinValues_sunspot = this.temp_datas.reduce(aryMin);
        this.drawSunspotChart(this.sunspot_count, this.temp_datas, this.sunspotDates);
      } catch (error) {
        console.error("Error fetching latest images:", error);
      }
    }
  },
  mounted() {
    this.interval = setInterval(() => {
      this.fetchSunInfos();
      this.sunspotChart();
      this.drawSunspotChart();
    }, 600000);
    this.fetchSunInfos();
    this.sunspotChart();
    this.drawSunspotChart();
  },
  beforeUnmount() {
    clearInterval(this.interval);
  },
  computed: {
    isSidebarOpen() {
      return this.$vuetify && this.$vuetify.sidebar && this.$vuetify.sidebar.model;
    }
  }
};
</script>

<style scoped>
.parent-card {
  width: 100%;
  max-width: 1200px; /* 画像が大きくなった分、カードの最大幅を調整 */
  margin: auto;
}
.image-container {
  width: 45%; /* 画像を横に並べるために幅を調整 */
  position: relative; /* キャプションの位置調整用 */
}
.text-h6 {
  font-size: 1.5rem; /* 文字サイズを大きく */
  font-weight: bold;
}
.crop-container {
  width: 100%;
  height: 500px; /* 元の高さ500pxの60%（上下1/5カット） */
  overflow: hidden !important; /* 確実にオーバーフローを隠す */
  position: relative;
}
.cropped-image {
  display: block; /* インラインマージンを回避 */
  width: 100% !important; /* コンテナ幅にフィット */
  height: 500px !important; /* 元画像の高さを維持 */
  object-fit: contain !important; /* コンテナにフィット */
  object-position: center !important; /* 画像の中心をコンテナの中心に */
  position: absolute !important;
  top: 50% !important; /* コンテナの垂直中央 */
  transform: translateY(-50%) !important; /* 画像の中心をコンテナの中心に揃える */
}
</style>
