<template>
  <v-container class="mx-auto d-flex align-center justify-start overflow-visible">
    <div class="text-h5 pa-3" style="font-weight: bold; border-bottom: 2px solid #000;">
      流星ビデオ観測ライブ（昼間は観測停止）
    </div>
  </v-container>
  <v-row class="cross-layout">
    <v-col>
      <v-col cols="12" md="4" class="cross-item center">
        <div class="iframe-wrapper">
          <iframe
            :src="'https://www.youtube.com/embed/oDbjlIRcCJE?autoplay=1'"
            frameborder="0"
            scrolling="no"
            allowfullscreen
            class="camera-frame"
          ></iframe>
        </div>
      </v-col>
    </v-col>
  </v-row>
  <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
    <div class="text-h5 pa-3" style="font-weight: bold; border-bottom: 2px solid #000;">
      最新の検出流星
    </div>
  </v-container>
  <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
    <v-row>
      <v-col v-for="(item, index) in 3" :key="index" cols="12" md="4">
        <v-card class="sharp-card" outlined>
          <v-img
            :src="MeteorlatestImages[index]"
            cover
            :aspect-ratio="16/9"
          ></v-img>
          <v-card-title class="flex-column align-start">
            <div class="text-h5 mb-2">{{ formatMeteorLog(meteor_logs[index]) }}</div>
          </v-card-title>
          <v-divider class="mx-4"></v-divider>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from 'axios';

export default {
  name: 'MeteorVideo',
  data() {
    return {
      MeteorlatestImages: [],
      meteor_logs: [],
      interval: -1,
      directionMap: {
        'Zenith': '天頂',
        'North': '北',
        'East': '東',
        'South': '南',
        'West': '西'
      }
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
        const MeteorimageUrl1 = "https://toms-server.tail2925.ts.net/Meteor_latest_video";
        const MeteorimageUrl2 = "https://toms-server.tail2925.ts.net/Meteor_latest2_video";
        const MeteorimageUrl3 = "https://toms-server.tail2925.ts.net/Meteor_latest3_video";
        this.MeteorlatestImages = [MeteorimageUrl1, MeteorimageUrl2, MeteorimageUrl3];
      } catch (error) {
        console.error("Error fetching meteor images:", error);
      }
    },
    async fetchMeteorinfo() {
      try {
        const response_video = await axios.get("https://toms-server.tail2925.ts.net/Meteor_info");
        this.meteor_logs = response_video.data;
      } catch (error) {
        console.error("Error fetching meteor info:", error);
      }
    },
    formatMeteorLog(log) {
      if (!log) return '';
      const parts = log.split(' :');
      if (parts.length < 2) return log; // フォーマットが異なる場合はそのまま返す
      const time = parts[0].trim();
      const direction = parts[1].trim();
      const japaneseDirection = this.directionMap[direction] || direction; // マッピングにない場合はそのまま
      return `${time} : ${japaneseDirection}`;
    }
  },
  mounted() {
    this.interval = setInterval(() => {
      this.fetchMeteorLatestImages();
      this.fetchMeteorinfo();
    }, 600000);


    this.fetchMeteorLatestImages();
    this.fetchMeteorinfo();
  },
  beforeUnmount() {
    clearInterval(this.interval);
  },
};
</script>

<style scoped>
.sharp-card {
  border-radius: 0 !important;
  background-color: white;
  border: 1px solid #212121;
}

.parent-card {
  margin: 2%;
  padding: 2%;
  padding-top: 0px;
}

.sidebar-open.parent-card {
  margin-left: 250px;
}

.cross-layout {
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
  height: 70vh;
  overflow: visible;
}

.cross-item {
  position: absolute;
  width: 80vw;
  max-width: 1200px;
  aspect-ratio: 16 / 9;
}

.cross-item.center {
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.cross-item.top {
  top: 3%;
  left: 50%;
  transform: translateX(-50%);
}

.cross-item.bottom {
  top: 65%;
  left: 50%;
  transform: translateX(-50%);
}

.cross-item.left {
  left: 5%;
  top: 50%;
  transform: translateY(-50%);
}

.cross-item.right {
  right: 5%;
  top: 50%;
  transform: translateY(-50%);
}

.iframe-wrapper {
  overflow: hidden;
  width: 100%;
  height: 100%;
  position: relative;
}

.camera-frame {
  width: 100%;
  height: 100%;
  object-fit: cover;
  position: absolute;
}
</style>