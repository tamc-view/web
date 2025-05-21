<template>
    <div class="equipments">
        <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
            <div class="text-h5 pa-3" style="font-weight: bold; border-bottom: 2px solid #000;">
              視程観測装置(都心方面)の使用状況
            </div>
        </v-container>

        <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
            <v-row justify="space-between" align="stretch" style="width: 100%;">
                <v-col cols="12" md="6" class="d-flex">
                    <v-container fluid class="d-flex flex-column" style="height: 100%;">
                        <v-row dense style="flex-grow: 1;">
                            <v-col cols="12" md="6" class="d-flex">
                                <v-card class="info-card d-flex flex-column flex-grow-1">
                                    <v-card-item>
                                        <div class="text-h6 mb-1">{{ basicInfos.title }}</div>
                                        <div class="text-h6 font-weight-regular text-grey">
                                            <div>疎通: True</div>
                                            <div>更新: {{ basicInfos.time }}</div>
                                            <div>温度: {{ basicInfos.temp }} 度</div>
                                            <div>湿度: {{ basicInfos.humis }} %</div>
                                            <div>CPU温度: {{ basicInfos.cpuTemp }} 度</div>
                                            <div>稼働時間: {{ basicInfos.operating }}</div>
                                        </div>
                                    </v-card-item>
                                </v-card>
                            </v-col>
                            <v-col cols="12" md="6" class="d-flex">
                                <v-card class="d-flex flex-column align-center justify-center info-card flex-grow-1" style="padding: 20px;">
                                    <!-- <v-card-item class="d-flex">
                                        <div class="text-h6 mb-1">{{ usages.title }}</div>
                                    </v-card-item> -->
                                    <v-row justify="center" align="center">
                                        <v-col cols="6" class="text-center">
                                            <v-progress-circular :model-value="basicInfos.sdUsage" :size="120" color="primary" class="mb-2" :width="10">
                                                {{ basicInfos.sdUsage }} %
                                            </v-progress-circular>
                                            <div class="text-subtitle1">本体使用率</div>
                                        </v-col>
                                        <v-col cols="6" class="text-center">
                                            <v-progress-circular :model-value="basicInfos.usbUsage" :size="120" color="primary" class="mb-2" :width="10">
                                                {{ basicInfos.usbUsage }} %
                                            </v-progress-circular>
                                            <div class="text-subtitle1">USB使用率</div>
                                        </v-col>
                                    </v-row>
                                </v-card>
                            </v-col>
                        </v-row>
                        <v-row dense>
                            <v-col cols="12" class="d-flex">
                                <v-card class="d-flex align-center justify-center info-card flex-grow-1">
                                    <!-- <v-card-item>
                                        <div class="text-h6 mb-1">{{ usaging.title }}</div>
                                    </v-card-item> -->
                                    <canvas id="lineChart" class="line-chart"></canvas>
                                </v-card>
                            </v-col>
                        </v-row>
                    </v-container>
                </v-col>
        
                <v-col cols="12" md="6" class="d-flex">
                    <v-container fluid class="d-flex flex-column" style="height: 100%;">
                        <v-card class="log-card flex-grow-1">
                            <v-list lines="three">
                                <v-list-subheader>撮影ログ等</v-list-subheader>
                                <v-list-item>
                                    <v-list-item-title>notify-error.log</v-list-item-title>
                                    <v-list-item-subtitle>{{ notifyErrorLog }}</v-list-item-subtitle>
                                </v-list-item>
                                <v-divider></v-divider>
                                <v-list-item>
                                    <v-list-item-title>send_info.log</v-list-item-title>
                                    <v-list-item-subtitle>{{ sendInfoErrorLog }}</v-list-item-subtitle>
                                </v-list-item>
                                <v-divider></v-divider>
                                <v-list-item>
                                    <v-list-item-title>obs-once-error.log</v-list-item-title>
                                    <v-list-item-subtitle>{{ obsOnceErrorLog }}</v-list-item-subtitle>
                                </v-list-item>
                                <v-divider></v-divider>
                                <v-list-item>
                                    <v-list-item-title>obs-once.log</v-list-item-title>
                                    <v-list-item-subtitle>{{ obsOnceLog }}</v-list-item-subtitle>
                                </v-list-item>
                            </v-list>
                        </v-card>
                    </v-container>
                </v-col>
            </v-row>
        </v-container>
        <v-divider class="border-opacity-100"></v-divider>
    </div>
</template>
  
  
<script>
import axios from 'axios';
import { Chart, LinearScale, CategoryScale, Title, Tooltip, Legend, LineElement, LineController, PointElement } from 'chart.js';
Chart.register(LinearScale, CategoryScale, Title, Tooltip, Legend, LineElement, LineController, PointElement);



export default {
  data() {
    return {
      usbUsages: [],
      Dates: [],
      cpuTempers: [],
      Tempers: [],
      Humis: [],
      sdCardUsages: [],
      notifyErrorLog: "",
      sendInfoErrorLog: "",
      obsOnceErrorLog: "",
      obsOnceLog: "",
      Fgo: "",
      Sgo: "",
      Tgo: "",
      basicInfos: {title: "基本情報", time: "", cpuTemp: "", usbUsage: "", sdUsage: "", temp: "", humis: "", operating: ""},
      usages: {title: "使用率", path: "/", path: "/mnt/usb1", sdUsage: "", usbUsage: ""},
      usaging: {title: "使用状況"},
      endpoint: "http://100.119.204.18:5000/1go/usages",
      lineColors: {
        temperature: 'rgba(255, 99, 132, 1)',
        cpuTemperature: 'rgba(255, 165, 38, 1)'
      },
      yMinValue: -5,
      yMaxValue: 30,
      interval: -1,
    };
  },
  methods: {
    async fetchLogs() {
      try {
        const response = await axios.get("http://100.119.204.18:5000/1go/logs");
        const logs = response.data;

        this.notifyErrorLog = logs["notify-error.log"];
        this.sendInfoErrorLog = logs["send_info-error.log"];
        this.obsOnceErrorLog = logs["obs-once-error.log"];
        this.obsOnceLog = logs["obs-once.log"];
      } catch (error) {
        console.error("Error fetching logs:", error);
      }
    },
    async fetchStatus(){
      try {
        const infos = await axios.get("http://100.119.204.18:5000/1go/info");

        this.basicInfos.time = infos.data.date;
        this.basicInfos.temp = infos.data.temperature;
        this.basicInfos.humis = infos.data.humidity;
        this.basicInfos.operating = infos.data.uptime;
        this.basicInfos.cpuTemp = infos.data.cpuTemperature;
        this.basicInfos.usbUsage = infos.data.usbUsage;
        this.basicInfos.sdUsage = infos.data.sdCardUsage;
      } catch (error) {
        console.error("Error fetching logs:", error);
      }
    },
    async fetchData() {
        try {
            const response = await axios.get(this.endpoint);
            const data = response.data;

            console.log("Fetched data:", data);

            if (!data || !data.temperature || !data.cpuTemperature || !data.date) {
                console.error("API response is missing required fields");
                return;
            }

            const times = data.date;
            const temperatures = data.temperature.filter(temp => temp !== -128); // -128の値を無視
            const cpuTemperatures = data.cpuTemperature.filter(cpuTemp => cpuTemp !== -128); // -128無視

            const minTemperature = Math.min(...temperatures);
            const maxTemperature = Math.max(...temperatures);
            const minCpuTemperature = Math.min(...cpuTemperatures);
            const maxCpuTemperature = Math.max(...cpuTemperatures);
            const minValues = [minTemperature, minCpuTemperature];
            const maxValues = [maxTemperature, maxCpuTemperature];
            const minValue = Math.min(...minValues);
            const maxValue = Math.max(...maxValues);

            this.yMinValue = minValue;
            this.yMaxValue = maxValue;



            this.drawChart(times, temperatures, cpuTemperatures);
        } catch (error) {
            console.error("エラーーー:", error);
        }
    },
    drawChart(labels, temperatures, cpuTemperatures) {
        const canvas = document.getElementById('lineChart');
        if (!canvas) {
            console.error("Canvas not found");
            return;
        }
        Chart.getChart(canvas)?.destroy();

        const chartData = {
            labels: labels,
            datasets: [
                {
                  label: "Temperature",
                  data: temperatures,
                  borderColor: this.lineColors.temperature,
                  fill: false
                },
                {
                  label: "CPU Temperature",
                  data: cpuTemperatures,
                  borderColor: this.lineColors.cpuTemperature,
                  fill: false
                }
            ]
        };

        const options = {
            maintainAspectRatio: false,
            responsive: true,
            scales: {
                y: {
                    min: this.yMinValue,
                    max: this.yMaxValue
                }
            }
        };

        const ctx = canvas.getContext('2d');
        new Chart(ctx, {
            type: 'line',
            data: chartData,
            options: options
        });
    }
  },
  mounted() {
    this.interval = setInterval(() => {
      this.fetchLogs();
      this.fetchStatus();
      this.fetchData();
    }, 600000);
    this.fetchLogs();
    this.fetchStatus();
    this.fetchData();
  },
  beforeUnmount() {
    clearInterval(this.interval);
  },
}
</script>

  
<style scoped>
.equipments {
    width: 100vw;
    background-color: white;
}

.info-card {
    border: 1px solid #212121;
    border-radius: 0;
}

.log-card {
    border: 1px solid #212121;
    border-radius: 0;
}

.line-chart {
    width: 100%;
    height: 15vw;
    max-height: 15vw;
  }
</style>
  
