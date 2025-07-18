<template>
  <div>
    <v-container class="mx-auto d-flex align-center justify-start overflow-visible">
      <div class="text-h4 pa-3" style="font-weight: bold; border-bottom: 2px solid #000;">
        流星電波観測ライブ
      </div>
    </v-container>
    <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
      <v-row>
        <v-col v-for="(item, index) in 2" :key="index" cols="12" md="6">
          <v-card class="sharp-card" outlined>
            <v-img
              :src="LiveImages[index]"
              cover
              :aspect-ratio="16/9"
            ></v-img>
            <v-card-title class="flex-column align-start">
              <div class="text-h4 mb-2">
                {{ ['豊川HRO', '大津HRO'][index] }}
              </div>
              <div class="text-h6 font-weight-regular text-grey" v-if="index === 0">
                {{ToyokawalatestTime}}
              </div>
              <div class="text-h6 font-weight-regular text-grey" v-if="index === 1">
                {{OtsulatestTime}}
              </div>
            </v-card-title>
            <v-divider class="mx-4"></v-divider>
          </v-card>
        </v-col>
      </v-row>
    </v-container>

    <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
      <div class="text-h4 pa-3" style="font-weight: bold; border-bottom: 2px solid #000;">
        最新の検出流星
      </div>
    </v-container>
    <v-card :class="{ 'parent-card': true, 'sidebar-open': isSidebarOpen }">
      <v-row>
        <v-col v-for="(item, index) in 3" :key="index" cols="12" md="4">
          <v-card class="sharp-card" outlined>
            <v-img
              :src="ToyokawalatestMeteorImages[index]"
              cover
              :aspect-ratio="16/9"
            ></v-img>
            <v-card-title class="flex-column align-start">
              <div class="text-h4 mb-2">豊川HRO</div>
              <div class="text-h6 font-weight-regular text-grey">{{ Toyokawa_meteor_logs[index] }}</div>
            </v-card-title>
            <v-divider class="mx-4"></v-divider>
          </v-card>
        </v-col>
      </v-row>
      <v-row>
        <v-col v-for="(item, index) in 3" :key="index" cols="12" md="4">
          <v-card class="sharp-card" outlined>
            <v-img
              :src="OtsulatestMeteorImages[index]"
              cover
              :aspect-ratio="16/9"
            ></v-img>
            <v-card-title class="flex-column align-start">
              <div class="text-h4 mb-2">大津HRO</div>
              <div class="text-h6 font-weight-regular text-grey">{{ Otsu_meteor_logs[index] }}</div>
            </v-card-title>
            <v-divider class="mx-4"></v-divider>
          </v-card>
        </v-col>
      </v-row>
    </v-card>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'MeteorRadio',
  data() {
    return {
      ToyokawalatestImage: '',
      OtsulatestImage: '',
      ToyokawalatestTime: '',
      OtsulatestTime: '',
      LiveImages: [],
      ToyokawalatestMeteorImages: [],
      OtsulatestMeteorImages: [],
      Toyokawa_meteor_logs: [],
      Otsu_meteor_logs: [],
      interval: -1,
    };
  },
  computed: {
    isSidebarOpen() {
      return this.$vuetify && this.$vuetify.sidebar && this.$vuetify.sidebar.model;
    },
  },
  methods: {
    async fetchMeteorLatestImages() {
      try {
        const ToyokawaimageUrl1 = "https://toms-server.tail2925.ts.net/Toyokawa_latest_img";
        const OtsuimageUrl1 = "https://toms-server.tail2925.ts.net/Otsu_latest_img";
        const ToyokawaMeteorimageUrl1 = "https://toms-server.tail2925.ts.net/Toyokawa_latest_meteor_img";
        const ToyokawaMeteorimageUrl2 = "https://toms-server.tail2925.ts.net/Toyokawa_latest2_meteor_img";
        const ToyokawaMeteorimageUrl3 = "https://toms-server.tail2925.ts.net/Toyokawa_latest3_meteor_img";
        const OtsuMeteorimageUrl1 = "https://toms-server.tail2925.ts.net/Otsu_latest_meteor_img";
        const OtsuMeteorimageUrl2 = "https://toms-server.tail2925.ts.net/Otsu_latest2_meteor_img";
        const OtsuMeteorimageUrl3 = "https://toms-server.tail2925.ts.net/Otsu_latest3_meteor_img";

        this.ToyokawalatestImage = ToyokawaimageUrl1;
        this.OtsulatestImage = OtsuimageUrl1;
        this.LiveImages = [
          ToyokawaimageUrl1,
          OtsuimageUrl1
        ]
        this.ToyokawalatestMeteorImages = [
          ToyokawaMeteorimageUrl1,
          ToyokawaMeteorimageUrl2,
          ToyokawaMeteorimageUrl3,
        ];
        this.OtsulatestMeteorImages = [
          OtsuMeteorimageUrl1,
          OtsuMeteorimageUrl2,
          OtsuMeteorimageUrl3,
        ];
      } catch (error) {
        console.error("Error fetching latest images:", error);
      }
    },
    async fetchMeteorInfo() {
      try {
        const response_toyokawa = await axios.get("https://toms-server.tail2925.ts.net/Toyokawa_info");
        const response_otsu = await axios.get("https://toms-server.tail2925.ts.net/Otsu_info");
        const response_meteor_toyokawa = await axios.get("https://toms-server.tail2925.ts.net/Toyokawa_meteor_info");
        const response_meteor_otsu = await axios.get("https://toms-server.tail2925.ts.net/Otsu_meteor_info");

        this.ToyokawalatestTime = response_toyokawa.data;
        this.OtsulatestTime = response_otsu.data;
        this.Toyokawa_meteor_logs = response_meteor_toyokawa.data;
        this.Otsu_meteor_logs = response_meteor_otsu.data;
      } catch (error) {
        console.error("Error fetching meteor info:", error);
      }
    },
  },
  mounted() {
    this.interval = setInterval(() => {
      this.fetchMeteorLatestImages();
      this.fetchMeteorInfo();
    }, 600000); 


    this.fetchMeteorLatestImages();
    this.fetchMeteorInfo();
  },
  beforeUnmount() {
    clearInterval(this.interval);
  },
};
</script>

<style scoped>
.parent-card {
  margin: 2%;
  padding: 2%;
  padding-top: 0px;
}

.sidebar-open.parent-card {
  margin-left: 250px;
}

.sharp-card {
  border-radius: 0 !important; 
  border: 1px solid #000 !important; 
}
</style>