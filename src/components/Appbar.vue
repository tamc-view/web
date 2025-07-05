<template>
  <v-app-bar flat class="custom-app-bar" height="110"> 
    <v-container class="mx-auto d-flex align-center justify-start">
      <v-img
        class="appbar-logo"
        height="85"
        max-width="500"
        src="@/assets/tamos_app_ogo.png"
        @click="navigateToTop" 
        style="cursor: pointer;" 
      />
    </v-container>
      <!-- <v-spacer></v-spacer> -->
    <v-container class="mx-auto d-flex align-center justify-end">
      <v-responsive max-width="230">
        <v-text-field
          density="compact"
          readonly
          :value="jstTime"
          rounded="lg"
          variant="solo-filled"
          flat
          hide-details
          single-line
        ></v-text-field>
      </v-responsive>
      <div class="d-flex flex-row justify-end" color="white">
        <v-time-picker v-model="picker" use-seconds></v-time-picker>
      </div>
    </v-container>
  </v-app-bar>

  <v-app-bar color="grey-darken-3" height="50">
    <v-spacer></v-spacer>
    <v-tabs align-tabs="center" grow hide-slider >
      <v-menu v-for="data in head_datas" :key="data.title" open-on-hover>
        <template v-slot:activator="{ props }">
          <v-tab v-bind="props" class="font-weight-bold"  @click="navigateToIndex(data)">
            {{ data.title }}
          </v-tab>
        </template>
        <v-list v-if="data.subs && data.subs.length" class="app-submenu">
          <v-list-item
            v-for="(sub, i) in data.subs"
            :key="i"
            :href="sub.href"
            :to="sub.path"
            @click="appDatasClick(sub)">
            <v-list-item-title class="d-flex align-center">
              <v-icon class="v-list-item__icon">{{ sub.icon }}</v-icon>
              <span>{{ sub.title }}</span>
            </v-list-item-title>

          </v-list-item>  
        </v-list>
      </v-menu>
    </v-tabs>
    <v-spacer></v-spacer>
  </v-app-bar>
</template>

<script>
export default {
data() {
return {
  picker: null,
  head_datas: [
    { title: 'トップ', subs: []},
    { 
      title: 'リアルタイムデータ', 
      subs: [
        { title: "都心方面データ", path: "/toshin", icon: "mdi-city" },
        { title: "富士山方面データ", path: "/fuji", icon: "mdi-image-filter-hdr" },
        { title: "流星ビデオデータ", path: "/meteor-video", icon: "mdi-meteor" },
        { title: "流星電波データ", path: "/meteor-radio", icon: "mdi-radio-tower" },
        { title: "夜空全天データ", path: "/night-sky", icon: "mdi-weather-night" },
        { title: "黒点データ", path: "/sunspots", icon: "mdi-white-balance-sunny" }
      ]
    },
    { 
      title: 'アーカイブデータ', 
      subs: [
        { title: "自動観測画像", path: "/observe-image"},
        { title: "Live E!", path: "/liveE" },
        { title: "気象観測記録", path: "/pastDocs" },
        { title: "黒点観測記録(未実装)", path: "#" },
        { title: "過去撮影画像", path: "/pastImages" },
      ]
    },
    { 
      title: 'インフォメーション', 
      subs: [
        { title: " (旧Twitter)", href: "https://x.com/tamc_net", icon: "mdi-alpha-x-box" },
        { title: "Instagram", href: "https://www.instagram.com/tamc_net", icon: "mdi-instagram" },
        { title: "立川高校公式", href: "https://www.metro.ed.jp/tachikawa-h", icon: "mdi-town-hall" }
      ]
    }
  ],
  jstTime: ''
};
},
mounted() {
this.updateTime();
this.timer = setInterval(this.updateTime, 1000);
},
beforeUnmount() {
clearInterval(this.timer);
},
methods: {
appDatasClick(selected_menu) {
  console.log("閲覧ページ:", selected_menu);
},
updateTime() {
  const now = new Date();
  const formattedTime = now.toLocaleString('ja-JP', {
    timeZone: 'Asia/Tokyo',
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit',
    hour12: false
  });
  this.jstTime = `JST   ${formattedTime}`;
},
navigateToIndex(data) {
    if (!data.subs || data.subs.length === 0) {
      this.$router.push('/');
    }
},
navigateToTop() {
      this.$router.push('/');
}
}
};
</script>

<style scoped>
.custom-app-bar {
background-image: url('@/assets/earth.png') !important;;
background-size: cover !important;;     
background-position: top right !important;;  
background-repeat: no-repeat !important;; 
}
.app-submenu {
background-color: rgb(255, 255, 255) !important;
backdrop-filter: blur(10px);
}
.v-list-item__icon{
  margin-right: 12 !important; 
}
.appbar-logo {
  display: inline-block; 
  flex-shrink: 0; 
}
</style>