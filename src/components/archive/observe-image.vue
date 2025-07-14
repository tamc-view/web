<template>
  <v-container fluid class="content-container">
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
            <span class="text-grey-lighten-1">{{ selectData.type }} / {{ selectData.year }} / {{ selectData.month}} / {{ selectData.day}} / {{ selectData.hour}}</span>
          </v-col>
        </v-row>
      </v-row>

      <v-form>
        <v-container>
          <v-row dense>
            <v-col cols="12" md="12">
              <v-select
                v-model="selectData.type"
                :items="types"
                :disabled="isUpdating"
                :error-messages="v$.type.$error ? ['データタイプを選択してください'] : []"
                color="blue-grey-lighten-2"
                label="data type"
                placeholder="画像種類を選択"
                required
              ></v-select>
            </v-col>

            <v-col cols="12" md="6">
              <v-select
                v-model="selectData.year"
                :items="search_year"
                :disabled="isUpdating"
                :error-messages="v$.year.$error ? ['年を選択してください'] : []"
                color="blue-grey-lighten-2"
                label="data year"
                placeholder="年を選択"
                required
              ></v-select>
            </v-col>

            <v-col cols="12" md="6">
              <v-select
                v-model="selectData.month"
                :items="search_month"
                :disabled="isUpdating"
                color="blue-grey-lighten-2"
                label="data month"
              ></v-select>
            </v-col>

            <v-col cols="12" md="6">
              <v-select
                v-model="selectData.day"
                :items="search_day"
                :disabled="isUpdating"
                color="blue-grey-lighten-2"
                label="data day"
              ></v-select>
            </v-col>

            <v-col cols="12" md="6">
              <v-select
                v-model="selectData.hour"
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
        <v-col v-for="info in infos" :key="info.name" cols="4" md="4">
          <v-card>
            <v-img
              :src='baseUrl + info.src'
              aspect-ratio="1.5"
              cover
              class="image-name"
            >
              <div class="imageName">
                <span>{{ info.name }}</span>
              </div>
            </v-img>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </v-container>
</template>

<script>
import { reactive, ref, computed, onMounted } from 'vue';
import { useVuelidate } from '@vuelidate/core';
import { required } from '@vuelidate/validators';
import axios from 'axios';

export default {
  setup() {
    const isUpdating = ref(false);
    const title = ref('自作自動観測装置の観測画像');
    const types = ref(['壱号機(スカイツリー方面)', '参号機(富士山方面)', 'まとめ画像(スカイツリー方面)']);
    const baseUrl = ref('https://toms-server.tail2925.ts.net');
    const infos = ref([]);
    const count = ref(0);
    
    const init_year = 2018;
    const c_year = new Date().getFullYear();
    
    const selectData = reactive({
      type: '画像種類',
      year: '選択年',
      month: '選択月',
      day: '選択日',
      hour: '選択時間'
    });

    const rules = {
      type: { 
        required: (value) => value !== null && value !== '' && value !== '画像種類'
      },
      year: { 
        required: (value) => value !== null && value !== '' && value !== '選択年'
      }
    };

    const v$ = useVuelidate(rules, selectData);

    const search_year = computed(() => 
      Array.from({ length: c_year - init_year + 1}, (_, i) => String(init_year + i))
    );
    const search_month = computed(() => 
      Array.from({length: 12}, (_, i) => String(i + 1).padStart(2, '0'))
    );
    const search_day = computed(() => 
      Array.from({length: 31}, (_, i) => String(i + 1).padStart(2, '0'))
    );
    const search_hour = computed(() => 
      Array.from({length: 16}, (_, i) => String(i + 4).padStart(2, '0'))
    );

    const searchImages = async () => {
      await v$.value.$validate();    // バリデーションチェック
      
      if (v$.value.$error) {
        console.log('バリデーションエラーがあります');
        return; //やり直し
      }

      isUpdating.value = true;
      count.value = 0;
      infos.value = [];

      try {
        const params = {
          type: selectData.type,
          Y: selectData.year,
        };

        if (selectData.month !== '選択月') {
          params.m = selectData.month;
        }
        if (selectData.day !== '選択日') {
          params.d = selectData.day;
        }
        if (selectData.hour !== '選択時間') {
          params.H = selectData.hour;
        }

        const response = await axios.get('https://toms-server.tail2925.ts.net/searchImages/', { params });
        const data = response.data;

        infos.value = (data.urls || []).map((url, i) => ({
          src: url,
          name: data.name?.[i] || '',
          description: data.description?.[i] || '',
        }));

        count.value = infos.value.length > 0;
      } catch (error) {
        console.error('データの取得に失敗しました:', error);
      } finally {
        isUpdating.value = false;
      }
    };

    // const resetSelect = () => {
    //   v$.value.$reset();
    //   selectData.type = null;
    //   selectData.year = null;
    //   selectData.month = '選択月';
    //   selectData.day = '選択日';
    //   selectData.hour = '選択時間';
    // };

    return {
      isUpdating,
      title,
      types,
      baseUrl,
      infos,
      count,
      selectData,
      v$,
      search_year,
      search_month,
      search_day,
      search_hour,
      searchImages,
      // resetSelect
    };
  }
};
</script>