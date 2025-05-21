<template>
    <div class="mainBody">
        <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
            <div class="text-h5 pa-3" style="font-weight: bold; border-bottom: 2px solid #000;">
              富士山方面の直近撮影画像
            </div>
        </v-container>
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
                            <div class="text-h4 mb-2" style="white-space: pre-line;">富士山: {{ flags[2] }}</div>
                            <div class="text-h6 font-weight-regular text-grey" style="white-space: pre-line;">
                                {{ observedTimes[index] }}
                            </div>
                            <div class="text-h6 font-weight-regular text-grey" style="white-space: pre-line;">
                                {{ obsEx[2] }}
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
            flags: {0: "False", 1: "True", 2: "不明"},
            latestResult: [],
            observedTimes: [],
            interval: -1,
            obsEx: {
                0: "富士山の確認不可",
                1: "富士山の確認可能",
                2: "不明",
            },
        };
    },
    methods: {
        async fetchlatestMedias() {
            try {
                const visFuji1 = await axios.get("http://100.119.204.18:5000/Fuji_latest_image", { responseType: 'blob' });
                const visFuji2 = await axios.get("http://100.119.204.18:5000/Fuji_30minago_image", { responseType: 'blob' });
                const visFuji3 = await axios.get("http://100.119.204.18:5000/Fuji_60minago_image", { responseType: 'blob' });
                // 古い Blob URL を解放
                this.latestMedias.forEach(url => URL.revokeObjectURL(url));

                const visFuji1Blob = new Blob([visFuji1.data]);
                const visFuji2Blob = new Blob([visFuji2.data]);
                const visFuji3Blob = new Blob([visFuji3.data]);
              
              this.latestMedias = [
                  URL.createObjectURL(visFuji1Blob),
                  URL.createObjectURL(visFuji2Blob),
                  URL.createObjectURL(visFuji3Blob)
              ];
            } catch (error) {
                console.log("Error fetching latestMedias:", error);
            }
        },
        async fetchlatestResults() {
            try {
                const Fuji_class1 = await axios.get("http://100.119.204.18:5000/Fuji_latest_class");
                const Fuji_class2 = await axios.get("http://100.119.204.18:5000/Fuji_30minago_class");
                const Fuji_class3 = await axios.get("http://100.119.204.18:5000/Fuji_60minago_class");
                const visFujiRes1 = Fuji_class1.data[0];
                const visFujiRes2 = Fuji_class2.data[0];
                const visFujiRes3 = Fuji_class3.data[0];
                this.latestResult = [visFujiRes1, visFujiRes2, visFujiRes3];
            } catch (error) {
                console.log("Error fetching latestResults: ", error);
            }
        },
        async fetchObservedTime() {
            try {
                const Fujilatest_time = await axios.get("http://100.119.204.18:5000/Fuji_latest_info");
                const Fuji30min_time = await axios.get("http://100.119.204.18:5000/Fuji_30minago_info");
                const Fuji60min_time = await axios.get("http://100.119.204.18:5000/Fuji_60minago_info");

                const visFujiTime1 = Fujilatest_time.data.time;
                const visFujiTime2 = Fuji30min_time.data.time;
                const visFujiTime3 = Fuji60min_time.data.time;

                this.observedTimes = [visFujiTime1, visFujiTime2, visFujiTime3];
            } catch(error) {
                console.log("Error fetching observedTimes:", error);
            }
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
</style>
