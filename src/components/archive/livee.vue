<template>
  <v-container class="mx-auto d-flex align-center justify-center overflow-visible">
    <div class="text-h5 pa-3" style="font-weight: bold; border-bottom: 2px solid #000;">
      気象データの比較(自動気象観測装置)
    </div>
  </v-container>  
  <v-container fluid class="content-container">
    <v-card :loading="isUpdating" class="mx-auto content-card">

      <template v-slot:loader="{ isActive }">
        <v-progress-linear
          :active="isActive"
          color="green-lighten-3"
          height="4"
          indeterminate
        ></v-progress-linear>
      </template>

      <v-row dense>
        <v-col cols="8" class="left-section">
          <v-col>
            <v-select v-model="selectedDataType1" :items="dataTypes" label="kind of data A"></v-select>
          </v-col>
          <v-col>
            <v-select v-model="selectedDataType2" :items="dataTypes" label="kind of data B"></v-select>
          </v-col>
          <!-- <v-col>
            <v-select v-model="selectedWeather" :items="weatherTypes" label="weather"></v-select>
          </v-col> -->
          <v-col>
            <v-text-field
              :model-value="formattedRange"
              label="range of dates"
              hint="like ~~~"
              persistent-hint
              variant="outlined"
            ></v-text-field>
          </v-col>
          <v-col>
            <v-btn block append-icon="mdi-download" variant="outlined" color="grey-darken-1" @click="downloadCSV"> Download </v-btn>
          </v-col>
        </v-col>

        <v-col cols="4" class="d-flex justify-center">
          <Datepicker class="datepicker"
            v-model="date"
            :format="format"
            cancel-text="back"
            select-text="OK"
            range
            multi-calendars
            auto-apply
            inline
            time-picker-inline
          >
            <template #year="{ value }">{{ value }}年</template>
          </Datepicker>
        </v-col>
      </v-row>

      <!-- <v-divider></v-divider> -->

      <v-card-actions class="v-card-actions">
        <v-btn
          block
          prepend-icon="mdi-update"
          @click="search"
          class="search-btn"
        >SEARCH</v-btn>
      </v-card-actions>
    </v-card>

    <!-- Line chart container outside v-card
    <div style="height: 300px; width: 100%;">
      <canvas id="lineChart"></canvas>
    </div> -->

    <v-card :loading="isUpdating" class="mx-auto graph-card">
      <canvas id="lineChart"></canvas>
    </v-card>

    <v-card :loading="isUpdating" class="mx-auto table-card">
      <v-data-table :headers="tableHeaders" :items="tableData"></v-data-table>
    </v-card>
  </v-container>
</template>

<script>
import { ref, defineComponent, onMounted } from 'vue';
import Datepicker from '@vuepic/vue-datepicker';
import '@vuepic/vue-datepicker/dist/main.css';
// import FileCompo from '@/components/FileInput.vue';
import { Chart } from 'chart.js/auto';
import axios from 'axios';

export default defineComponent({
  name: "App",
  components: {
    Datepicker,
    // FileCompo,
  },
  setup() {
    const selectedDataType1 = ref(null);
    const selectedDataType2 = ref(null);
    const selectedWeather = ref(null);
    const date = ref([]);
    const formattedRange = ref('');
    const isUpdating = ref(false);
    const showChart = ref(false);
    const dataTypes = ['temperature', 'humidity', 'avgwinddir', 'maxwinddir', 'minwinddir', 'avgwindvel', 'maxwindvel', 'minwindvel', 'pressure', 'raininten', 'rainfall', 'hailinten', 'hailfall'];
    const weatherTypes = ['Sunny', 'Cloudy', 'Rainy'];
    const tableHeaders = [
      { text: 'Date', value: 'date'},
      { text: 'selectedData1', value: 'selectedData1'},
      { text: 'selectedData2', value: 'selectedData2'}
    ];
    const tableData = ref([]);

    // const demodatas = [
    //   { date: 'xxxx', fdata: '23'}, { date: 'xxxx', fdata: '23'}, { date: 'xxxx', fdata: '23'}, { date: 'xxxx', fdata: '23'}, { date: 'xxxx', fdata: '23'}
    // ];

    const format = (dateArray) => {
      if (dateArray.length === 2) {
        const startDate = dateArray[0];
        const endDate = dateArray[1];

        const formatDate = (date) => {
          const year = date.getFullYear();
          const month = String(date.getMonth() + 1).padStart(2, '0');
          const day = String(date.getDate()).padStart(2, '0');
          const hours = String(date.getHours()).padStart(2, '0');
          const minutes = String(date.getMinutes()).padStart(2, '0');
          // const seconds = String(date.getSeconds()).padStart(2, '0');

          return `${year}-${month}-${day} ${hours}:${minutes}`;
        };

        const formattedStartDate = formatDate(startDate);
        const formattedEndDate = formatDate(endDate);
        formattedRange.value = `${formattedStartDate} ~ ${formattedEndDate}`;
        //const apiUrl = `http://100.119.204.18:5000/pastdata/${selectedDataType.value}/${formattedStartDate}/${formattedEndDate}`

        // return `${formattedStartDate} ~ ${formattedEndDate}`;
        return {
          formattedStartDate,
          formattedEndDate
        };
      }
      return '';
    };

    const drawLineChart = () => {
      const ctx = document.getElementById('lineChart').getContext('2d');
      new Chart(ctx, {
        type: 'line',
        data: {
          labels: ['25', '26', '27', '28', '29', '30', '1', '2', '3', '4', '5', '6'],
          datasets: [],
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          scales: {
            x: {
              grid: {
                color: 'rgba(255, 255, 255, 0.1)',
              },
            },
            y: {
              grid: {
                color: 'rgba(255, 255, 255, 0.1)',
              },
            },
          },
          plugins: {
            legend: {
              labels: {
                color: 'black',
              },
            },
          },
          animation: {
            onComplete: () => {
              showChart.value = true;
            },
          },
        },
      });
    };
    
    onMounted(() => {
      drawLineChart();
    });

    const search = async () => {
      isUpdating.value = true;

      const { formattedStartDate, formattedEndDate } = format(date.value);

      const apiUrl = `http://100.119.204.18:5000/pastdata/${selectedDataType1.value}/${formattedStartDate}/${formattedEndDate}`;
      const apiUrl2 = `http://100.119.204.18:5000/pastdata/${selectedDataType2.value}/${formattedStartDate}/${formattedEndDate}`;

      try {
        const response = await axios.get(apiUrl);
        const response2 = await axios.get(apiUrl2);
        const newData = response.data.data;
        const newData2 = response2.data.data;
        const lineChart = Chart.getChart('lineChart');

        const dates = newData.map(entry => entry.date)
        const selectedData1 = newData.map(entry => entry[selectedDataType1.value]);
        const selectedData2 = newData2.map(entry => entry[selectedDataType2.value]);

        lineChart.data.labels = dates;
        lineChart.data.datasets = [
          {
            label: selectedDataType1.value,
            data: selectedData1,
            borderColor: 'red',
            borderWidth: 0.5,
            fill: false
          },
          {
            label: selectedDataType2.value,
            data: selectedData2,
            borderColor: 'blue',
            borderWidth: 0.5,
            fill: false,
          }
        ];

        lineChart.update();

        tableData.value = newData.map((entry, index) => ({
          date: entry.date,
          selectedData1: entry[selectedDataType1.value],
          selectedData2: newData2[index][selectedDataType2.value]
        }));
        
        isUpdating.value = false;
      } catch (error) {
        console.error('Error fetching data:', error);
        isUpdating.value = false;
      }
    };

    const downloadCSV = async () => {
      const { formattedStartDate, formattedEndDate } = format(date.value);
      const url = `http://100.119.204.18:5000/pastdata/${selectedDataType1.value}/${selectedDataType2.value}/${formattedStartDate}/${formattedEndDate}`;
  
      try {
        const response = await axios.get(url, { responseType: 'blob' });
        const csvUrl = window.URL.createObjectURL(new Blob([response.data]));
        const link = document.createElement('a');
        link.href = csvUrl;
        const file_name = `${selectedDataType1.value}${selectedDataType2.value}_${formattedStartDate}-${formattedEndDate}.csv`;
        link.setAttribute('download', file_name);
        document.body.appendChild(link);
        link.click();
      } catch (error) {
        console.error('Error downloading csv:', error);
      }
    };

    return {
      selectedDataType1,
      selectedDataType2,
      selectedWeather,
      date,
      isUpdating,
      showChart, // グラフの表示・非表示を管理する変数を返す
      dataTypes,
      weatherTypes,
      format,
      search,
      tableData,
      tableHeaders,
      formattedRange,
      downloadCSV
    };
  },
});
</script>



<style>
.v-card-actions {
  margin: 0 !important;
  padding: 0 !important;
}

.search-btn {
  border-radius: 0px !important;
  border-top: 1px solid #212121 !important;
  min-width: 100% !important;
  padding: 0 !important;
}

.datepicker {
  display: flex;
  justify-content: center;
  width: 100%;
  max-width: 700px;
  min-width: 700px;
  margin-left: auto;
  margin-right: auto;
}

.graph-card, .v-card, .table-card {
  border-radius: 0px !important;
  border: 1px solid #212121 !important;
  margin: 1vw;
}

.content-card {
  border-radius: 0px !important;
  border: 1px solid #212121 !important;
  margin: 1vw;
}

</style>