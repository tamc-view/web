<template>
  <v-card :loading="isUpdating" class="mx-auto" style="margin-bottom: 15px;">
    <template v-slot:loader="{ isActive }">
      <v-progress-linear
        :active="isActive"
        color="green-lighten-3"
        height="4"
        indeterminate
      ></v-progress-linear>
    </template>
    <v-row class="pa-3">
      <v-col cols="12">
        <v-menu location="bottom start" origin="overlap" transition="slide-y-transition">
          <v-list :lines="false">
            <v-list-item title="Update" @click="isUpdating = true"></v-list-item>
          </v-list>
        </v-menu>
      </v-col>
      <v-row>
        <v-col class="text-center">
          <h3 class="text-h5">{{ name }}</h3>
          <span class="text-grey-lighten-1">{{ sub_name[0] }} / {{ sub_name[1] }} / {{ sub_name[2] }} / {{ sub_name[3] }}</span>
        </v-col>
      </v-row>
    </v-row>
    <v-form>
      <v-container>
        <v-row dense>
          <v-col cols="12" md="6">
            <v-select
              v-model="sub_name[0]"
              :items="types"
              :disabled="isUpdating"
              color="blue-grey-lighten-2"
              label="data type"
            ></v-select>
          </v-col>
          <v-col cols="12" md="6">
            <v-select
              v-model="sub_name[1]"
              :items="yy"
              :disabled="isUpdating"
              color="blue-grey-lighten-2"
              label="year"
            ></v-select>
          </v-col>
  
          <v-col cols="12" md="6">
            <v-select
              v-model="sub_name[2]"
              :items="months"
              :disabled="isUpdating"
              color="blue-grey-lighten-2"
              label="month"
            ></v-select>
          </v-col>
  
          <v-col cols="12" md="6">
            <v-select
              v-model="sub_name[3]"
              :items="weeks_num"
              :disabled="isUpdating"
              color="blue-grey-lighten-2"
              label="week"
            ></v-select>
          </v-col>

          <v-col cols="12">
            <v-autocomplete
              v-model="conditions"
              :disabled="isUpdating"
              :items="condition"
              chips
              closable-chips
              color="blue-grey-lighten-2"
              item-title="name"
              item-value="name"
              label="others"
              multiple
            >
              <template v-slot:chip="{ props, item }">
                <v-chip
                  v-bind="props"
                  :prepend-icon="item.raw.icon"
                  :text="item.raw.name"
                ></v-chip>
              </template>
  
              <template v-slot:item="{ props, item }">
                <v-list-item
                  v-bind="props"
                  :prepend-icon="item?.raw?.icon"
                  :title="item?.raw?.name"
                  :subtitle="item?.raw?.group"
                ></v-list-item>
              </template>
            </v-autocomplete>
          </v-col>
        </v-row>
      </v-container>
    </v-form>

    <v-divider></v-divider>

    <v-card-actions>
      <v-btn
        block
        :loading="isUpdating"
        prepend-icon="mdi-update"
        @click="searchClicked"
      >SEARCH</v-btn>
    </v-card-actions>

  </v-card>

  <!-- <div class="text-h5 pa-3" style="font-weight: bold; border-bottom: 2px solid #000;">
    該当画像数
  </div> -->

  <v-container v-if="showResult" class="mx-auto" max-width="1200">
    <v-row>
        <v-col v-for="index in searchedImages" :key="index" cols="4">
            <v-card height="200px" class="black-card">
                <v-img :src="searchedImages[index]" :aspect-ratio="3/2"></v-img>
                <v-card-title class="flex-column align-start white-text" style="height: auto; display: block;">
                    <div class="text-h6 font-weight-regular text-grey" style="white-space: pre-line;">
                        {{ imgTimes[index] }}
                    </div>
                </v-card-title>
            </v-card>
        </v-col>
    </v-row>
  </v-container>
</template>
  
<script>
import axios from 'axios';

export default {
  data() {
    return {
      autoUpdate: true,
      conditions: [],
      name: '自動撮影装置の画像検索',
      condition: [
          { name: 'スカイツリー_◯', group: 'visibility', icon: 'mdi-eiffel-tower'},
          { name: 'ビル群_◯', group: 'visibility', icon: 'mdi-domain'},
          { name: '晴れ', group: 'weather', icon: 'mdi-weather-sunny'},
          { name: '曇り', group: 'weather', icon: 'mdi-weather-cloudy'},
          { name: '午前', group: 'time', icon: 'mdi-sun-clock'},
      ],
      types: ['スカイツリー方面', '富士山方面', '武蔵小杉方面'],
      yy: ["2019", "2020", "2021", "2022", "2023", "2024", "2025"],
      weeks_num: ['---','1~7', '8~14', '15~21', '22~28', '29~31'],
      sub_name: ['観測装置' ,'年', '月', '週', 'その他'],
      months: ['01', '02', '03', '04', '05', '06', '07', '08', '09', '10', '11', '12'],
      timeout: null,
      showResult: false,
      searchedImages: [],
      imgTimes: [],
      timeout: null
    };
  },
  watch: {
      isUpdating(val) {
          clearTimeout(this.timeout);

          if (val) {
              this.timeout = setTimeout(() => (this.isUpdating = false), 3000);
          }
      },
  },
  methods: {
    remove(item) {
      const index = this.conditions.indexOf(item.name);
      if (index >= 0) this.conditions.splice(index, 1);
    },

    async searchClicked() {
      this.searchedImages = [];
      this.showResult = false;
      this.isUpdating = true;

      try {
        const response = await axios.get(`https://toms-server.tail2925.ts.net/folderID/${this.name}/${this.title}/${this.month}`);
         
        const filesArray = response.data.files;

        if (Array.isArray(filesArray)) {
          const numberOfFiles = filesArray.length;
          console.log('取得したデータのfilesの数:', numberOfFiles);

          filesArray.forEach(file => {
            this.cards.push({
              title: file.name,
              src: `https://drive.google.com/file/d/${file.id}/preview`,
              flex: 6,
            });
          });
        } else {
          console.clear();
          console.log('取得したデータにfilesが含まれていないか、配列ではありません。');
        }

        setTimeout(() => {
          this.isUpdating = false;
          this.showResult = true;
        }, 3000);
      } catch (error) {
        console.error('データの取得に失敗しました:', error);
        this.isUpdating = false;
      }
    },
  },
}
</script>