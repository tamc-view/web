<template>
    <div class="equipments">
        <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
            <div class="text-h5 pa-3" style="font-weight: bold; border-bottom: 2px solid #000;">
              観測装置使用状況
            </div>
        </v-container>

        <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
            <v-row justify="space-between" align="stretch" style="width: 100%;">
                <v-col cols="12" md="6" class="d-flex">
                    <v-card class="flex-grow-1 status-card">
                        <v-card-item title="各撮影装置">
                            <template v-slot:subtitle>
                              <v-icon
                                class="me-1 pb-1"
                                icon="mdi-camera-wireless-outline"
                                size="18"
                              ></v-icon>
              
                              使用状況
                            </template>
                          </v-card-item>

                          <v-card-text class="py-0">
                            <div class="text-center">
                              <v-progress-circular
                                :model-value="usbUsages[0]"
                                :rotate="360"
                                :size="100"
                                :width="15"
                                color="primary"
                              >
                                1go: {{ usbUsages[0] }}%
                              </v-progress-circular>
              
                              <v-progress-circular
                                :model-value="usbUsages[1]"
                                :rotate="360"
                                :size="100"
                                :width="15"
                                color="green"
                              >
                                2go: {{ usbUsages[1] }}%
                              </v-progress-circular>
              
                              <v-progress-circular
                                :model-value="usbUsages[2]"
                                :rotate="360"
                                :size="100"
                                :width="15"
                                color="yellow"
                              >
                                3go: {{ usbUsages[2] }}%
                              </v-progress-circular>
                            </div>
                          </v-card-text>

                          <v-expand-transition>
                            <div>
                              <v-list class="bg-transparent">
                                <v-list-item
                                  v-for="item, index in forecast"
                                  :key="item.day"
                                  :append-icon="item.icon"
                                  :subtitle="`Time: ${Dates[index]}, CPUtemperature: ${cpuTempers[index]}, usbUsage: ${usbUsages[index]}, sdCardUsage: ${sdCardUsages[index]}, Temperature: ${Tempers[index]}, Humidity: ${Humis[index]}`"
                                  :title="item.day"
                                >
                                </v-list-item>
                              </v-list>
                            </div>
                          </v-expand-transition>
                    </v-card>
                </v-col>
                <v-col cols="12" md="6" class="d-flex">
                    <v-card class="flex-grow-1 log-card">
                      <v-list lines="three">
                        <v-list-subheader>撮影ログ等</v-list-subheader>
          
                        <v-list-item>
                          <v-list-item-title>notify-error.log</v-list-item-title>
          
                          <v-list-item-subtitle>
                            {{ notifyErrorLog }}
                          </v-list-item-subtitle>
                        </v-list-item>
          
                        <v-divider></v-divider>
          
                        <v-list-item>
                          <v-list-item-title>send_info.log</v-list-item-title>
          
                          <v-list-item-subtitle>
                            {{  sendInfoErrorLog }}
                          </v-list-item-subtitle>
                        </v-list-item>
          
                        <v-divider></v-divider>
          
                        <v-list-item>
                          <v-list-item-title>obs-once-error.log</v-list-item-title>
          
                          <v-list-item-subtitle>
                            {{ obsOnceErrorLog }}
                          </v-list-item-subtitle>
                        </v-list-item>
          
                        <v-divider></v-divider>
          
                        <v-list-item>
                          <v-list-item-title>obs-once.log</v-list-item-title>
          
                          <v-list-item-subtitle>
                            {{ obsOnceLog }}
                          </v-list-item-subtitle>
                        </v-list-item>
                      </v-list>
                    </v-card>
                </v-col>
            </v-row>
        </v-container>
        <v-divider class="border-opacity-100"></v-divider>
    </div>
</template>
  
  
<script>
import axios from 'axios';

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
      forecast: [
        {
          day: '都心方面(壱号機)',
          icon: 'mdi-eiffel-tower',
          temp: '24\xB0/12\xB0',
        },
        {
          day: '武蔵小杉方面(弐号機)',
          icon: 'mdi-office-building',
          temp: '22\xB0/14\xB0',
        },
        { day: '富士山方面(参号機)', icon: 'mdi-image-filter-hdr', temp: '25\xB0/15\xB0' },
      ],
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
        const firstInfo = await axios.get("http://100.119.204.18:5000/1go/info");
        const secondInfo = await axios.get("http://100.119.204.18:5000/2go/info");
        const thirdInfo = await axios.get("http://100.119.204.18:5000/3go/info");

        const firstusbUsage = firstInfo.data.usbUsage;
        const secondusbUage = secondInfo.data.usbUsage;
        const thirdusbUsage = thirdInfo.data.usbUsage;
        this.usbUsages = [firstusbUsage, secondusbUage, thirdusbUsage]
        const firstDate = firstInfo.data.date;
        const secondDate = secondInfo.data.date;
        const thirdDate = thirdInfo.data.date;
        this.Dates = [firstDate, secondDate, thirdDate]
        const firstCpuTemper = firstInfo.data.cpuTemperature;
        const secondCpuTemper = secondInfo.data.cpuTemperature;
        const thirdCpuTemper = thirdInfo.data.cpuTemperature;
        this.cpuTempers = [firstCpuTemper, secondCpuTemper, thirdCpuTemper];
        const firstTemper = firstInfo.data.temperature;
        const secondTemper = secondInfo.data.temperature;
        const thirdTemper = thirdInfo.data.temperature;
        this.Tempers = [firstTemper, secondTemper, thirdTemper];
        const firstHum = firstInfo.data.humidity;
        const secondHum = secondInfo.data.humidity;
        const thirdHum = thirdInfo.data.humidity;
        this.Humis = [firstHum, secondHum, thirdHum];
        const firstsdUsage = firstInfo.data.sdCardUsage;
        const secondsdUsage = secondInfo.data.sdCardUsage;
        const thirdsdUsage = thirdInfo.data.sdCardUsage;
        this.sdCardUsages = [firstsdUsage, secondsdUsage, thirdsdUsage];
        const firstInfos = firstInfo.data;
        const secondInfos = secondInfo.data;
        const thirdInfos = thirdInfo.data;
        this.Fgo = firstInfos;
        this.Sgo = secondInfos;
        this.Tgo = thirdInfos;
      } catch (error) {
        console.error("Error fetching logs:", error);
      }
    },
  },
  mounted() {
    this.interval = setInterval(() => {
      this.fetchLogs();
      this.fetchStatus();
    }, 600000);
    this.fetchLogs();
    this.fetchStatus();
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

.status-card {
    border: 1px solid #212121;
    border-radius: 0;
}

.log-card {
    border: 1px solid #212121;
    border-radius: 0;
}
</style>
  
  