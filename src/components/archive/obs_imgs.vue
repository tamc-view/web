<template>
  <!-- <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
    <div class="text-h5 pa-3" style="font-weight: bold; border-bottom: 2px solid #000;">
      自作自動観測装置の観測画像
    </div>
  </v-container> -->
  <v-container fluid clas="content-conrainer">
    <v-card :loading="isUpdating" class="mx-auto content-card">

      <template v-slot:loader="{ isActive }">
        <v-progress-linear
          :active="isActive"
          color="green-lighten-3"
          height="4"
          interminate
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
            <h3 class="text-h5">{{ title }}</h3>
            <span class="text-grey-lighten-1">{{ default_type }} / {{ default_year }} / {{ default_month}} / {{ default_day}} / {{ default_hour}}</span>
          </v-col>
        </v-row>
      </v-row>

      <v-form>
        <v-container>
          <v-row dense>
            <v-col cols="12" md="12">
              <v-select
                v-model="default_type"
                :items="types"
                :disabled="isUpdating"
                color="blue-grey-lighten-2"
                label="data type"
              ></v-select>
            </v-col>

            <v-col cols="12" md="6">
              <v-select
                v-model="default_year"
                :items="search_year"
                :disabled="isUpdating"
                color="blue-grey-lighten-2"
                label="data year"
              ></v-select>
            </v-col>

            <v-col cols="12" md="6">
              <v-select
                v-model="default_month"
                :items="search_month"
                :disabled="isUpdating"
                color="blue-grey-lighten-2"
                label="data month"
              ></v-select>
            </v-col>

            <v-col cols="12" md="6">
              <v-select
                v-model="default_day"
                :items="search_day"
                :disabled="isUpdating"
                color="blue-grey-lighten-2"
                label="data day"
              ></v-select>
            </v-col>

            <v-col cols="12" md="6">
              <v-select
                v-model="default_hour"
                :items="search_hour"
                :disabled="isUpdating"
                color="blue-grey-lighten-2"
                label="data hour"
              ></v-select>
            </v-col>
          </v-row>
        </v-container>
      </v-form>

      <v-divider />

      <v-card-actions>
        <v-btn
          block
          :loading="isUpdating"
          prepend-icon="mdi-update"
          @click="searchImages"
        >SEARCH</v-btn>
      </v-card-actions>
    </v-card>


    <v-container fluid v-if="count" class="mx-auto" max-height="1200">
      <v-row dense>
        <v-col v-for="info in infos" :key="info.name" cols="4" md="3">
          <v-card>
            <iframe
              :src="baseUrl + info.src"
              width="100%"
              height="480"
              allow="autodisplay"
            ></iframe>

            <v-card-actions>
              <v-spacer></v-spacer>

              <!-- <v-btn
                size="small"
                color="surface-variant"
                variant="text"
                icon="mdi-bookmark"
              ></v-btn> -->
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </v-container>
</template>

<script>
  import axios from 'axios';

  export default {
    data() {
      return {
        autoUpdate: true,
        isUpdating: false,
        title: '自作自動観測装置の観測画像',
        types: ['壱号機(スカイツリー方面)', '参号機(富士山方面)', '並べたまとめ画像'],
        init_year: 2018,
        c_year: new Date().getFullYear(),
        search_year: [],   // c_yearを使うがdata()では他を参照できないのでcreated()で。
        // search_year: Array.from({length: c_year - init_year + 1}, (_, i) => String(i + init_year)),
        search_month: Array.from({length: 12}, (_, i) => String(i + 1).padStart(2, '0')),
        search_day: Array.from({length: 31}, (_, i) => String(i + 1).padStart(2, '0')),
        search_hour: Array.from({length: 16}, (_, i) => String(i + 4).padStart(2, '0')),
        default_type: '画像種類',
        default_year: '選択年',
        default_month: '選択月',
        default_day: '選択日',
        default_hour: '選択時間',
        infos: [],
        timeout: null,
        count: null
      };
    },

    created() {
      this.search_year = Array.from({ length: this.c_year -this.init_year + 1}, (_, i) => String(this.init_year + i))
    },

    methods: {
      async searchImages() {
        this.isUpdating = true;
        this.count = null;
        this.infos = [];

        try {
          const params = {
            type: this.default_type,
            Y: this.default_year,
            m: this.default_month,
            d: this.default_day,
            H: this.default_hour
          };

          const response = await axios.get('https://toms-server.tail2925.ts.net/searchImages', {params});
          const data = response.data;

          this.infos = (data.urls || []).map((url, i) => ({
            src: url,
            name: data.name?.[i] || '',
            description: data.description?.[i] || '',
          }));

          this.count = this.infos.urls.length > 0;
        } catch (error) {
          console.error('データの取得に失敗しました:', error);
        } finally {
          this.isUpdating = false;
        }
      }
    }
  }
</script>