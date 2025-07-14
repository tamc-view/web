<template>
    <div class="mainBody">
        <!-- Meteorology セクション -->
        <v-container class="mx-auto overflow-visible">
            <div class="category-title text-h5 pa-3" style="font-weight: bold; border-bottom: 2px solid #000;">
                気象 (Meteorology)
            </div>
            <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
                <v-row justify="space-between" align="stretch" style="width: 100%;">
                    <v-col v-for="(media, index) in meteorologyData.medias" :key="`met-${index}`" cols="12" sm="6" md="6" lg="6">
                        <v-card class="black-card" height="100%">
                            <v-img
                            :src="media"
                            cover
                            :aspect-ratio="3/2"
                            ></v-img>
                            <v-card-title class="flex-column align-start white-text" style="height: auto; display: block;">
                                <div class="text-h4 mb-2 d-flex align-center" style="white-space: pre-line;">
                                    <v-icon class="mr-2" size="90">
                                        {{ meteorologyData.icons[index] }}
                                    </v-icon>
                                    {{ meteorologyData.types[index] }}: {{ meteorologyData.results[index] }}
                                </div>
                                <div class="text-h6 font-weight-regular text-grey" style="white-space: pre-line;">
                                    {{ meteorologyData.times[index] }}
                                </div>
                                <div class="text-h6 font-weight-regular text-grey" style="white-space: pre-line;">
                                    {{ meteorologyData.explanations[index] }}
                                </div>
                            </v-card-title>
                        </v-card>
                    </v-col>
                </v-row>
            </v-container>
        </v-container>
        
        <!-- Astronomy セクション -->
        <v-container class="mx-auto overflow-visible">
            <div class="category-title text-h5 pa-3" style="font-weight: bold; border-bottom: 2px solid #000;">
                天文 (Astronomy)
            </div>
            <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
                <v-row justify="space-between" align="stretch" style="width: 100%;">
                    <v-col v-for="(media, index) in astronomyData.medias" :key="`astro-${index}`" cols="12" sm="6" md="6" lg="6">
                        <v-card class="black-card" height="100%">
                            <v-img
                            :src="media"
                            cover
                            :aspect-ratio="3/2"
                            ></v-img>
                            <v-card-title class="flex-column align-start white-text" style="height: auto; display: block;">
                                <!-- 流星の場合 -->
                                <template v-if="astronomyData.types[index] === '流星'">
                                    <div class="text-h4 mb-2 d-flex align-center" style="white-space: pre-line;">
                                        <v-icon class="mr-2" size="90">mdi-meteor</v-icon>
                                        流星： {{ getDirection(astronomyData.results[index]) }}
                                    </div>
                                    <div class="text-h5 font-weight-regular white-text mb-2" style="white-space: pre-line;">
                                        {{ getTime(astronomyData.results[index]) }}
                                    </div>
                                </template>
                                <!-- 黒点の場合 -->
                                <template v-else-if="astronomyData.types[index] === '黒点'">
                                    <div class="text-h4 mb-2 d-flex align-center" style="white-space: pre-line;">
                                        <v-icon class="mr-2" size="90">mdi-white-balance-sunny</v-icon>
                                        黒点面積： {{ astronomyData.results[index] }} pixel
                                    </div>
                                    <div class="text-h6 font-weight-regular text-grey" style="white-space: pre-line;">
                                        観測日: {{ astronomyData.times[index] }}
                                    </div>
                                </template>
                                <div class="text-h6 font-weight-regular text-grey" style="white-space: pre-line;">
                                    {{ astronomyData.explanations[index] }}
                                </div>
                            </v-card-title>
                        </v-card>
                    </v-col>
                </v-row>
            </v-container>
        </v-container>
        <v-divider class="border-opacity-100"></v-divider>
    </div>
</template>

<script>
import axios from 'axios';

export default {
    data() {
        return {
            meteorologyData: {
                medias: [],
                types: ["視程距離", "富士山"],
                icons: ["mdi-eye-outline", "mdi-image-filter-hdr-outline"],
                results: [],
                times: [],
                explanations: [
                    "視程とは観測場所から識別することのできる距離の程度を表し、大気汚染等の指標となる。",
                    "本校から富士山までの直線距離は70kmである。富士山付近では笠雲やつるし雲がみられることがある。",
                ]
            },
            astronomyData: {
                medias: [],
                types: ["流星", "黒点"],
                icons: ["mdi-meteor", "mdi-white-balance-sunny"],
                results: [],
                times: [],
                explanations: [
                    "流星とは流れ星のことであり、流星の観測は宇宙空間の彗星・小惑星の姿や高層の地球大気を解明することに繋がる。",
                    "黒点は太陽表面の暗い部分で、太陽活動の指標となる。",
                ]
            },
            visDis: {0:"0~0.1km", 1: "0.45km", 2: "0.75km", 3: "1.0km", 4: "3.0km", 5: "4.3km", 6: "4.9km", 7:"13km", 8: "25km", 9: "36km~"},
            Fujiobs: {0: "富士山の確認不可", 1: "富士山の確認可能", 2: "不明"},
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
    methods: {
        async fetchlatestMedias() {
            try {
                // Meteorology データ
                const visTree = await axios.get("https://toms-server.tail2925.ts.net/latest_image", { responseType: 'blob' });
                const visFuji = await axios.get("https://toms-server.tail2925.ts.net/Fuji_latest_image", { responseType: 'blob' });
                // Astronomy データ
                const metVideo = "https://toms-server.tail2925.ts.net/Meteor_latest_video";
                const sunspotImage = await axios.get("https://toms-server.tail2925.ts.net/manualReport/sunspot/info");
                
                // 古い Blob URL を解放
                this.meteorologyData.medias.forEach(url => {
                    if (url.startsWith('blob:')) URL.revokeObjectURL(url);
                });
                this.astronomyData.medias.forEach(url => {
                    if (url.startsWith('blob:')) URL.revokeObjectURL(url);
                });

                const visTreeBlob = new Blob([visTree.data]);
                const visFujiBlob = new Blob([visFuji.data]);
                const sunspotImageUrl = `https://toms-server.tail2925.ts.net${sunspotImage.data.image_url}`;
                
                this.meteorologyData.medias = [
                    URL.createObjectURL(visTreeBlob),
                    URL.createObjectURL(visFujiBlob)
                ];
                this.astronomyData.medias = [
                    metVideo,
                    sunspotImageUrl
                ];
            } catch (error) {
                console.log("Error fetching latestMedias:", error);
            }
        },
        async fetchlatestResults() {
            try {
                const latest_class = await axios.get("https://toms-server.tail2925.ts.net/latest_class");
                const Fujilatest_class = await axios.get("https://toms-server.tail2925.ts.net/Fuji_latest_class");
                const meteor_dir = await axios.get("https://toms-server.tail2925.ts.net/Meteor_info");
                const sunspotInfo = await axios.get("https://toms-server.tail2925.ts.net/manualReport/sunspot/info");
                
                const visTreeRes = latest_class.data[0];
                const meteorRes = meteor_dir.data[0];
                const sunspotArea = sunspotInfo.data.total_area;
                
                this.meteorologyData.results = [
                    this.visDis[visTreeRes],
                    this.Fujiobs[2]
                ];
                this.astronomyData.results = [
                    meteorRes,
                    sunspotArea
                ];
            } catch (error) {
                console.log("Error fetching latestResults: ", error);
            }
        },
        async fetchObservedTime() {
            try {
                const latest_time = await axios.get("https://toms-server.tail2925.ts.net/latest_info");
                const Fujilatest_time = await axios.get("https://toms-server.tail2925.ts.net/Fuji_latest_info");
                const meteor_time = await axios.get("https://toms-server.tail2925.ts.net/Meteor_info");
                const sunspotInfo = await axios.get("https://toms-server.tail2925.ts.net/manualReport/sunspot/info");

                const visTreeTime = latest_time.data.time;
                const visFujiTime = Fujilatest_time.data.time;
                const meteor_logs = meteor_time.data;
                const sunspotTime = sunspotInfo.data.date;

                this.meteorologyData.times = [
                    visTreeTime,
                    visFujiTime
                ];
                this.astronomyData.times = [
                    meteor_logs[0],
                    sunspotTime
                ];
            } catch(error) {
                console.log("Error fetching observedTimes:", error);
            }
        },
        getTime(result) {
            if (!result) return '';
            const [time] = result.split(' :');
            return time.trim();
        },
        getDirection(result) {
            if (!result) return '';
            const parts = result.split(' :');
            const direction = parts.length > 1 ? parts[1].trim() : '';
            return this.directionMap[direction] || direction; 
        }
    },
    
    mounted() {
        this.interval = setInterval(() => {
            this.fetchlatestMedias();
            this.fetchlatestResults();
            this.fetchObservedTime();
        }, 600000);

        this.fetchlatestMedias();
        this.fetchlatestResults();
        this.fetchObservedTime();
    },
    beforeUnmount() {
        clearInterval(this.interval);
    },
};
</script>

<style scoped>
.mainBody {
    width: 100vw;
    background-color: white;
}

.full-width-row {
    width: 100vw;
    max-width: 100vw;
    margin: 0;
}

.black-card {
  background-color: black;
  border: 5px solid #212121;
  border-radius: 0;
}

.white-text {
  color: white;
}

.mr-2 {
    margin-right: 8px; 
}
</style>
