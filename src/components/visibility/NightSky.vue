<template>
  <div>
    <v-container class="mx-auto d-flex align-center justify-start overflow-visible">
      <div class="text-h5 pa-3" style="font-weight: bold; border-bottom: 2px solid #000;">
        全天カメラライブ（HQカメラ）
      </div>
    </v-container>
    <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
      <v-row>
        <v-col>
          <v-row>
            <v-card class="sharp-card" outlined>
              <v-img
                :width="740"
                :src="HQlatestImage"
                cover
              ></v-img>
              <v-card-title class="flex-column align-start">
                <!-- <div class="text-h4 mb-2">Cloud Cover: Φ</div> -->
                <div class="text-h4 mb-2">{{ HQlatestTime }}</div>
              </v-card-title>
              <v-divider class="mx-4"></v-divider>
            </v-card>
          </v-row>
        </v-col>
        <v-col>
          <v-row>
            <v-card class="sharp-card" outlined>
              <v-img
                :width="555"
                :src="HosizoralatestImage"
                cover
              ></v-img>
              <v-card-title class="flex-column align-start">
                <div class="text-h4 mb-2">今夜の星空</div>
                <div class="text-h6 font-weight-regular text-grey">{{ HosizoraTime }}</div>
              </v-card-title>
              <v-divider class="mx-4"></v-divider>
            </v-card>
          </v-row>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'NightSky',
  data() {
    return {
      HQlatestImage: '',
      HQlatestTime: '',
      HosizoralatestImage: '',
      HosizoraTime: '',
      interval: -1,
    };
  },
  methods: {
    async fetchHQLatestImages() {
      try {
        const HQimageUrl = await axios.get("https://toms-server.tail2925.ts.net/HQ_latest_img", { responseType: 'blob' });
        const response_Hosizora = await axios.get("https://toms-server.tail2925.ts.net/Hosizora_img", { responseType: 'blob' });
        
        this.HQlatestImage = URL.createObjectURL(HQimageUrl.data);
        this.HosizoralatestImage = URL.createObjectURL(response_Hosizora.data);
      } catch (error) {
        console.error("Error fetching latest images:", error);
      }
    },
    async fetchHQInfo() {
      try {
        const response_HQ = await axios.get("https://toms-server.tail2925.ts.net/HQ_info");
        const response_Hosizora_time = await axios.get("https://toms-server.tail2925.ts.net/Hosizora_info");

        this.HQlatestTime = response_HQ.data;
        this.HosizoraTime = response_Hosizora_time.data;
      } catch (error) {
        console.error("Error fetching info:", error);
      }
    },
  },
  mounted() {
    this.interval = setInterval(() => {
      this.fetchHQLatestImages();
      this.fetchHQInfo();
    }, 600000); // Update every 10 minutes

    // Initial fetch
    this.fetchHQLatestImages();
    this.fetchHQInfo();
  },
  beforeUnmount() {
    clearInterval(this.interval);
  },
};
</script>

<style scoped>
.sharp-card {
  border-radius: 0 !important; /* 角の丸みをなくす */
  border: 1px solid #000 !important; /* 黒い枠線を追加 */
}
</style>