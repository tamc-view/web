<template>
  <div>
    <v-container class="mx-auto d-flex align-center justify-start overflow-visible">
      <div class="text-h4 pa-3" style="font-weight: bold; border-bottom: 2px solid #000;">
        黒点画像(Seestar)
      </div>
    </v-container>
    <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
      <v-row>
        <v-col v-for="(item, index) in 2" :key="index" cols="12" md="6">
          <v-card class="sharp-card" outlined>
            <v-img
              :src="SunspotImages[index]"
              cover
            ></v-img>            
            <v-card-title class="flex-column align-start">
              <div class="text-h4 pa-3" style="font-weight: bold;" v-if="index === 0">
                最新の観測日: {{ latestSunInfo.date }}
              </div>
              <div class="text-h4 pa-3" style="font-weight: bold;" v-if="index === 1">
                黒点面積: {{ latestSunInfo.total_area }} pixel
              </div>
            </v-card-title>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'Sunspots',
  data() {
    return {
      latestSunImage: "",      // 元画像のURL
      contourSunImage: "",     // 検出画像のURL
      latestSunInfo: {},       // APIからのレスポンス全体を保持
      SunspotImages: [],
    };
  },
  methods: {
    async fetchSunInfos() {
      try {
        const response = await axios.get("https://toms-server.tail2925.ts.net/manualReport/sunspot/info");
        // 画像URLを直接使用
        this.latestSunImage = `https://toms-server.tail2925.ts.net${response.data.image_url}`;
        this.contourSunImage = `https://toms-server.tail2925.ts.net${response.data.contour_image_url}`;
        this.latestSunInfo = response.data;
        this.SunspotImages = [
          this.latestSunImage,
          this.contourSunImage,
        ];
      } catch (error) {
        console.error("Error fetching latest sunspot data:", error);
      }
    },
  },
  mounted() {
    this.interval = setInterval(() => {
      this.fetchSunInfos();
    }, 600000); // 10分ごとに更新
    this.fetchSunInfos();
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
