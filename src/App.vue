<template>
  <div
    :class="[
      darkMode ? 'bg-gray-900 text-gray-100' : 'bg-gray-100 text-gray-800',
      'min-h-screen gap-5 p-10 flex md:flex-row flex-col items-center justify-center',
    ]"
  >
    <div class="w-full md:w-1/2">
      <img
        ref="image"
        class="rounded-lg w-full shadow-lg"
        :src="imageFile"
        alt="image"
      />
    </div>
    <div
      :class="[
        darkMode ? 'bg-gray-800' : 'bg-white',
        'p-6 rounded-lg shadow-lg w-full md:w-1/2 max-w-lg',
      ]"
    >
      <div class="flex justify-between items-center mb-4">
        <h1 class="text-2xl font-bold">Forecast Data</h1>
        <button
          @click="toggleDarkMode"
          class="text-sm px-3 py-1 rounded-lg border"
        >
          {{ darkMode ? "Light Mode" : "Dark Mode" }}
        </button>
      </div>
      <form @submit.prevent="onSubmit">
        <div class="mb-5">
          <label
            for="Masukan Target Tahun Prediksi"
            class="block mb-2 text-sm font-medium"
            >Masukan Target Tahun Prediksi</label
          >
          <input
            type="number"
            id="Masukan Target Tahun Prediksi"
            v-model="year"
            :class="[
              darkMode
                ? 'bg-gray-700 text-gray-100 border-gray-600 placeholder-gray-400'
                : 'bg-gray-50 text-gray-900 border-gray-300',
              'text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5',
            ]"
            placeholder="Masukan Target Tahun Prediksi, contoh: 2025"
            required
          />
        </div>
        <div class="flex justify-end mb-5 w-full">
          <button
            type="submit"
            :class="[
              darkMode
                ? 'bg-blue-600 hover:bg-blue-700 focus:ring-blue-800'
                : 'bg-blue-700 hover:bg-blue-800 focus:ring-blue-300',
              'text-white w-full font-medium rounded-lg text-sm px-5 py-2.5',
            ]"
          >
            Prediksi
          </button>
        </div>
      </form>

      <table
        :class="[
          darkMode ? 'bg-gray-700 text-gray-100' : 'bg-gray-200 text-gray-800',
          'table-auto w-full',
        ]"
      >
        <thead>
          <tr>
            <th class="px-4 py-2 text-left">Tahun</th>
            <th class="px-4 py-2 text-left">Hasil Panen</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="item in forecasts"
            :key="item.year"
            :class="[darkMode ? 'hover:bg-gray-600' : 'hover:bg-gray-100', '']"
          >
            <td class="px-4 py-2">{{ item.year }}</td>
            <td class="px-4 py-2">{{ item.forecast.toLocaleString() }} ton</td>
          </tr>
        </tbody>
      </table>
      <div
        v-if="loading"
        class="text-center mt-4"
        :class="darkMode ? 'text-gray-400' : 'text-gray-500'"
      >
        Prossesing...
      </div>
      <div v-if="error" class="text-center mt-4 text-red-500">
        Failed to load data: {{ error }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import axios from "axios";
const baseURL = import.meta.env.VITE_BASE_URL;
const forecasts = ref([]);
const loading = ref(false);
const error = ref(null);
const year = ref(null);
const darkMode = ref(false);
let imageFile = ref(baseURL + "/static/forecast_plot.png");
const toggleDarkMode = () => {
  darkMode.value = !darkMode.value;
};

const getSteps = () => {
  return year.value - 2020;
};

let image = ref(null);
const onSubmit = async () => {
  try {
    error.value = null;
    if (year.value < 2021) {
      error.value = "Tahun harus lebih besar dari 2020.";
      return;
    }
    loading.value = true;
    const response = await axios.post(baseURL + "/forecast", {
      steps: getSteps(),
    });
    forecasts.value = response.data.forecast;
    // Tambahkan timestamp ke URL untuk memaksa refresh
    if (image.value) {
      image.value.src = `${baseURL}/static/forecast_plot.png?timestamp=${Date.now()}`;
    }
  } catch (err) {
    error.value = err.message || "An error occurred";
  } finally {
    loading.value = false;
  }
};
</script>

<style>
body {
  transition: background-color 0.3s, color 0.3s;
}
</style>
