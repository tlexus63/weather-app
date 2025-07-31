<template>
  <div>
    <input v-model="query" @keyup.enter="searchLocation" placeholder="Enter location" />
    <ul>
      <li v-for="loc in locations" :key="loc.id" @click="selectLocation(loc)">
        {{ loc.name }}, {{ loc.country }}
      </li>
    </ul>
    <div v-if="forecast.length">
      <h2>Forecast for {{ selected?.name }}</h2>
      <ul>
        <li v-for="(temp, i) in forecast" :key="i">
          {{ localTimes[i] }} — {{ temp }}°C
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const query = ref('');
const locations = ref<any[]>([]);
const selected = ref<any>(null);
const forecast = ref<number[]>([]);
const localTimes = ref<string[]>([]);

const searchLocation = async () => {
  if (!query.value) {
    locations.value = [];
    return;
  }
  const res = await fetch(`https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(query.value)}&count=5&language=en&format=json`);
  const data = await res.json();
  locations.value = data.results || [];
};

const selectLocation = async (loc: any) => {
  selected.value = loc;
  const today = new Date().toISOString().split('T')[0];
  const tomorrow = new Date(Date.now() + 86400000).toISOString().split('T')[0];
  const res = await fetch(`https://api.open-meteo.com/v1/forecast?latitude=${loc.latitude}&longitude=${loc.longitude}&hourly=temperature_2m&start_date=${today}&end_date=${tomorrow}`);
  const data = await res.json();
  forecast.value = data.hourly.temperature_2m;
  localTimes.value = data.hourly.time.map((t: string) => new Date(t).toLocaleTimeString());
};
</script>
