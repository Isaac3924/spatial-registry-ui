<script setup lang="ts">
import { onMounted, ref } from 'vue'
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'

interface SpatialAsset {
  id: number
  assetId: string
  sensorType: string
  latitude: number
  longitude: number
  cloudCoverPercentage: number
  capturedAt: string
}

const API_URL = 'http://localhost:8080/api/v1/assets'
const US_CENTER: L.LatLngTuple = [39.0, -98.0]
const DEFAULT_ZOOM = 4

const mapContainer = ref<HTMLDivElement | null>(null)
const isLoading = ref(true)
const error = ref<string | null>(null)

let map: L.Map | null = null

async function fetchAssets(): Promise<SpatialAsset[]> {
  const response = await fetch(API_URL)
  if (!response.ok) {
    throw new Error(`Failed to fetch assets: ${response.status} ${response.statusText}`)
  }
  return response.json()
}

function plotAssets(assets: SpatialAsset[]) {
  if (!map) return

  for (const asset of assets) {
    L.marker([asset.latitude, asset.longitude])
      .addTo(map)
      .bindPopup(`<strong>${asset.assetId}</strong><br />${asset.sensorType}`)
  }
}

onMounted(async () => {
  if (!mapContainer.value) return

  map = L.map(mapContainer.value).setView(US_CENTER, DEFAULT_ZOOM)

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
  }).addTo(map)

  try {
    const assets = await fetchAssets()
    plotAssets(assets)
  } catch (err) {
    error.value = err instanceof Error ? err.message : 'Unknown error fetching assets'
  } finally {
    isLoading.value = false
  }
})
</script>

<template>
  <main class="app">
    <div v-if="isLoading" class="status status--loading">Loading assets&hellip;</div>
    <div v-else-if="error" class="status status--error">{{ error }}</div>
    <div ref="mapContainer" class="map"></div>
  </main>
</template>

<style scoped>
.app {
  position: relative;
  width: 100vw;
  height: 100vh;
}

.map {
  width: 100%;
  height: 100%;
}

.status {
  position: absolute;
  top: 1rem;
  left: 50%;
  transform: translateX(-50%);
  z-index: 1000;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  background: white;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.3);
  font-size: 0.9rem;
}

.status--error {
  color: #b91c1c;
}
</style>
