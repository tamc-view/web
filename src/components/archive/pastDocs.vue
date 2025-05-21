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
          <v-menu
            location="bottom start"
            origin="overlap"
            transition="slide-y-transition"
          >
            <v-list :lines="false">
              <v-list-item title="Update" @click="isUpdating = true"></v-list-item>
            </v-list>
          </v-menu>
        </v-col>
  
        <v-row>
          <v-col class="text-center">
            <h3 class="text-h5">{{ name }}</h3>
            <span class="text-grey-lighten-1">{{ title }} / {{ month }} / {{ week }}</span>
          </v-col>
        </v-row>
      </v-row>
  
      <v-form>
        <v-container>
          <v-row dense>
            <v-col cols="12" md="6">
              <v-select
                v-model="name"
                :items="types"
                :disabled="isUpdating"
                color="blue-grey-lighten-2"
                label="data type"
              ></v-select>
            </v-col>
  
            <v-col cols="12" md="6">
              <v-select
                v-model="title"
                :items="yy"
                :disabled="isUpdating"
                color="blue-grey-lighten-2"
                label="year"
              ></v-select>
            </v-col>
  
            <v-col cols="12" md="6">
              <v-select
                v-model="month"
                :items="months"
                :disabled="isUpdating"
                color="blue-grey-lighten-2"
                label="month"
              ></v-select>
            </v-col>
  
            <v-col cols="12" md="6">
              <v-select
                v-model="week"
                :items="weeks_num"
                :disabled="isUpdating"
                color="blue-grey-lighten-2"
                label="week"
              ></v-select>
            </v-col>
  
            <v-col cols="12">
              <v-autocomplete
                v-model="friends"
                :disabled="isUpdating"
                :items="people"
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
  
    <v-container fluid v-if="showResult" class="mx-auto" max-width="1200">
      <v-row dense>
        <v-col v-for="card in cards" :key="card.title" cols="4" md="3">
          <v-card>
            <iframe
              :src="card.src"
              width="100%"
              height="480"
              allow="autoplay"
            ></iframe>
  
            <v-card-actions>
              <v-spacer></v-spacer>
  
              <v-btn
                size="small"
                color="surface-variant"
                variant="text"
                icon="mdi-bookmark"
              ></v-btn>
  
              <v-btn
                size="small"
                color="surface-variant"
                variant="text"
                icon="mdi-share-variant"
              ></v-btn>
            </v-card-actions>
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
        friends: ['sunny', 'cloudy'],
        isUpdating: false,
        name: 'WEATHER DATA',
        people: [
          { name: 'sunny', group: 'Group 1', icon: 'weather-sunny' },
          { name: 'cloudy', group: 'Group 2', icon: 'weather-cloudy' },
        ],
        types: ['AirPressure_room', 'weather data(未実装)', 'sunspot data(未実装)',],
        yy: ["1968(データ欠け)", "1969", 1970, 1971,1972,1973,1974,1975,1976,1977,1978,1979,1980,1981,1982,1983,1984,1985,1986,1987,1988,1989,1990,1991,1992,1993],
        weeks_num: ['---','1~7', '8~14', '15~21', '22~28', '29~31'],
        months: ['01', '02', '03', '04', '05', '06', '07', '08', '09', '10', '11', '12'],
        title: 'detail',
        month: 'month',
        week: 'week',
        timeout: null,
        cards: [
          {
            title: 'test pic',
            src: 'https://drive.google.com/file/d/1LT8YSCnd2IMijnhm4UA66YrB_gtbP7jU/preview',
            flex: 6,
          },
          {
            title: 'test pic',
            src: 'https://drive.google.com/file/d/1k9yzKgWeBLjdeXXZq64jon-pJrnttfgr/preview',
            flex: 6,
          },
          {
            title: 'test pic',
            src: 'https://drive.google.com/file/d/1DR7URhGXeRT68aOnpFZp5K-aY4C4JyZG/preview',
            flex: 6,
          },
        ],
        showResult: false,
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
        const index = this.friends.indexOf(item.name);
        if (index >= 0) this.friends.splice(index, 1);
      },
  
      async searchClicked() {
        this.cards = [];
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

  