<template>
  <ion-page>
    <ion-header>
      <ion-toolbar color="primary">
        <ion-title>Prakiraan Cuaca Jakarta</ion-title>
        <ion-buttons slot="end">
          <ion-button @click="loadWeather">
            <ion-icon slot="icon-only" :icon="refreshOutline"></ion-icon>
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <div v-if="loading" class="center">
        <ion-spinner name="crescent"></ion-spinner>
        <p>Memuat data...</p>
      </div>

      <ion-text color="danger" v-else-if="error">
        <p>{{ error }}</p>
      </ion-text>

      <ion-list v-else>
        <ion-list-header>
          <ion-label>Waktu &amp; Suhu per Jam (°C)</ion-label>
        </ion-list-header>

        <ion-item v-for="(item, index) in weatherList" :key="index">
          <ion-label>
            <h3>{{ formatTime(item.time) }}</h3>
            <p>{{ item.time }}</p>
          </ion-label>
          <ion-badge slot="end" :color="badgeColor(item.temperature_2m)">
            {{ item.temperature_2m }} °C
          </ion-badge>
        </ion-item>
      </ion-list>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
  IonList, IonListHeader, IonItem, IonLabel, IonBadge,
  IonSpinner, IonText, IonButtons, IonButton, IonIcon,
} from '@ionic/vue';
import { refreshOutline } from 'ionicons/icons';

interface WeatherItem {
  time: string;
  temperature_2m: number;
}

const API_URL =
  'https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&hourly=temperature_2m';

const weatherList = ref<WeatherItem[]>([]);
const loading = ref<boolean>(true);
const error = ref<string>('');

const loadWeather = async () => {
  loading.value = true;
  error.value = '';
  try {
    const res = await fetch(API_URL);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    const data = await res.json();

    // API mengembalikan array paralel: hourly.time[] dan hourly.temperature_2m[]
    const times: string[] = data.hourly.time;
    const temps: number[] = data.hourly.temperature_2m;

    // Gabungkan jadi array objek, ambil 10 data pertama
    weatherList.value = times
      .map((t, i) => ({ time: t, temperature_2m: temps[i] }))
      // .slice(0, 10);
  } catch (e) {
    error.value = 'Gagal memuat data cuaca. Periksa koneksi internet Anda.';
    console.error(e);
  } finally {
    loading.value = false;
  }
};

const formatTime = (iso: string): string =>
  new Date(iso).toLocaleString('id-ID', {
    weekday: 'short', day: 'numeric', month: 'short',
    hour: '2-digit', minute: '2-digit',
  });

const badgeColor = (t: number): string =>
  t >= 32 ? 'danger' : t >= 28 ? 'warning' : 'success';

onMounted(loadWeather);
</script>

<style scoped>
.center {
  text-align: center;
  margin-top: 40%;
}
</style>