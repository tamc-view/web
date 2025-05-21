<template>
    <div class="mainBody">
        <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
            <v-row justify="space-between" align="stretch" style="width: 100%;">
                <v-col v-for="(media, index) in latestMedias" :key="index" cols="12" sm="6" md="4" lg="4">
                    <v-card class="black-card" height="100%">
                        <v-img
                          :src="latestMedias[index]"
                          cover
                          :aspect-ratio="3/2"
                        ></v-img>
                        <v-card-title class="flex-column align-start white-text" style="height: auto; display: block;">
                           <!-- 流星の場合の特別なフォーマット -->
                           <template v-if="realtimeDataType[index] === '流星'">
                                <div class="text-h4 mb-2 d-flex align-center" style="white-space: pre-line;">
                                    <v-icon class="mr-2" size="90">mdi-meteor</v-icon>
                                    流星： {{ getDirection(latestResult[index]) }}
                                </div>
                                <div class="text-h5 font-weight-regular white-text mb-2" style="white-space: pre-line;">
                                    {{ getTime(latestResult[index]) }}
                                </div>
                            </template>
                            <!-- 流星以外の場合 -->
                            <template v-else>
                                <div class="text-h4 mb-2 d-flex align-center" style="white-space: pre-line;">
                                    <v-icon class="mr-2" size="90">
                                        {{ ['mdi-eye-outline', 'mdi-image-filter-hdr-outline', 'mdi-meteor'][index] }}
                                    </v-icon>
                                    {{ realtimeDataType[index] }}: {{ latestResult[index] }}
                                </div>
                            </template>
                            <!-- 観測時間と説明（流星以外は通常表示、流星は時間のみ上記で処理） -->
                            <div v-if="realtimeDataType[index] !== '流星'" class="text-h6 font-weight-regular text-grey" style="white-space: pre-line;">
                                {{ observedTimes[index] }}
                            </div>
                            <div class="text-h6 font-weight-regular text-grey" style="white-space: pre-line;">
                                {{ obsEx[index] }}
                            </div>
                        </v-card-title>
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
            latestMedias: [],
            realtimeDataType: ["視程距離", "富士山", "流星"],
            visDis: {0:"0~0.1km", 1: "0.45km", 2: "0.75km", 3: "1.0km", 4: "3.0km", 5: "4.3km", 6: "4.9km", 7:"13km", 8: "25km", 9: "36km~"},
            Fujiobs: {0: "富士山の確認不可", 1: "富士山の確認可能", 2: "不明"},
            latestResult: [],
            observedTimes: [],
            interval: -1,
            obsEx: ["視程とは観測場所から識別することのできる距離の程度を表し、大気汚染等の指標となる。", "本校から富士山までの直線距離は70kmである。富士山付近では笠雲やつるし雲がみられることがある。", "流星とは流れ星のことであり、流星の観測は宇宙空間の彗星・小惑星の姿や高層の地球大気を解明することに繋がる。",],
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
                const visTree = await axios.get("https://toms-server.tail2925.ts.net/latest_image", { responseType: 'blob' });
                const visFuji = await axios.get("https://toms-server.tail2925.ts.net/Fuji_latest_image", { responseType: 'blob' });
                const metVideo = "https://toms-server.tail2925.ts.net/Meteor_latest_video";
                // 古い Blob URL を解放
                this.latestMedias.forEach(url => URL.revokeObjectURL(url));

                const visTreeBlob = new Blob([visTree.data]);
                const visFujiBlob = new Blob([visFuji.data]);
                
                this.latestMedias = [
                    URL.createObjectURL(visTreeBlob),
                    URL.createObjectURL(visFujiBlob),
                    metVideo
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
                const visTreeRes = latest_class.data[0];
                // const visFujiRes = Fujilatest_class.data[0];
                const meteorRes = meteor_dir.data[0];
                this.latestResult = [this.visDis[visTreeRes], this.Fujiobs[2], meteorRes];
            } catch (error) {
                console.log("Error fetching latestResults: ", error);
            }
        },
        async fetchObservedTime() {
            try {
                const latest_time = await axios.get("https://toms-server.tail2925.ts.net/latest_info");
                const Fujilatest_time = await axios.get("https://toms-server.tail2925.ts.net/Fuji_latest_info");
                const meteor_time = await axios.get("https://toms-server.tail2925.ts.net/Meteor_info");

                const visTreeTime = latest_time.data.time;
                const visFujiTime = Fujilatest_time.data.time;
                this.meteor_logs = meteor_time.data;

                this.observedTimes = [visTreeTime, visFujiTime, meteor_logs[0]];
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
