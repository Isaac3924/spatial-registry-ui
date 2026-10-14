<script setup lang="ts">
import { onMounted, reactive, ref } from 'vue'
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'

interface SpatialAsset {
  id?: number
  assetId: string
  sensorType: string
  latitude: number
  longitude: number
  cloudCoverPercentage?: number
  capturedAt: string
  imageUrl?: string
}

interface AssetFormState {
  assetId: string
  sensorType: string
  latitude: number | null
  longitude: number | null
  cloudCoverPercentage: number | null
  imageUrl: string
}

const API_URL = `${import.meta.env.VITE_API_BASE_URL}/api/v1/assets`
const US_CENTER: L.LatLngTuple = [39.0, -98.0]
const DEFAULT_ZOOM = 4

function createEmptyForm(): AssetFormState {
  return {
    assetId: '',
    sensorType: '',
    latitude: null,
    longitude: null,
    cloudCoverPercentage: null,
    imageUrl: '',
  }
}

const mapContainer = ref<HTMLDivElement | null>(null)
const isLoading = ref(true)
const error = ref<string | null>(null)

const isPanelOpen = ref(true)
const form = reactive<AssetFormState>(createEmptyForm())
const isSubmitting = ref(false)
const submitError = ref<string | null>(null)
const submitSuccess = ref(false)

let map: L.Map | null = null

async function fetchAssets(): Promise<SpatialAsset[]> {
  const response = await fetch(API_URL)
  if (!response.ok) {
    throw new Error(`Failed to fetch assets: ${response.status} ${response.statusText}`)
  }
  return response.json()
}

function plotAsset(asset: SpatialAsset) {
  if (!map) return

  L.marker([asset.latitude, asset.longitude])
    .addTo(map)
    .bindPopup(`<strong>${asset.assetId}</strong><br />${asset.sensorType}`)
}

async function createAsset(asset: SpatialAsset): Promise<SpatialAsset> {
  const response = await fetch(API_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(asset),
  })
  if (!response.ok) {
    throw new Error(`Failed to create asset: ${response.status} ${response.statusText}`)
  }
  return response.json()
}

async function handleSubmit() {
  if (form.latitude === null || form.longitude === null) {
    submitError.value = 'Click the map to set a latitude and longitude first.'
    return
  }

  isSubmitting.value = true
  submitError.value = null
  submitSuccess.value = false

  const payload: SpatialAsset = {
    assetId: form.assetId,
    sensorType: form.sensorType,
    latitude: form.latitude,
    longitude: form.longitude,
    cloudCoverPercentage: form.cloudCoverPercentage ?? undefined,
    imageUrl: form.imageUrl || undefined,
    capturedAt: new Date().toISOString(),
  }

  try {
    const created = await createAsset(payload)
    plotAsset(created)
    Object.assign(form, createEmptyForm())
    submitSuccess.value = true
  } catch (err) {
    submitError.value = err instanceof Error ? err.message : 'Unknown error creating asset'
  } finally {
    isSubmitting.value = false
  }
}

onMounted(async () => {
  if (!mapContainer.value) return

  map = L.map(mapContainer.value).setView(US_CENTER, DEFAULT_ZOOM)

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
  }).addTo(map)

  map.on('click', (event: L.LeafletMouseEvent) => {
    form.latitude = Number(event.latlng.lat.toFixed(6))
    form.longitude = Number(event.latlng.lng.toFixed(6))
    submitSuccess.value = false
  })

  try {
    const assets = await fetchAssets()
    assets.forEach(plotAsset)
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

    <button type="button" class="panel-toggle" @click="isPanelOpen = !isPanelOpen">
      {{ isPanelOpen ? 'Hide' : 'Add Asset' }}
    </button>

    <aside v-if="isPanelOpen" class="panel">
      <h2>New Spatial Asset</h2>
      <p class="panel-hint">Click the map to set coordinates.</p>

      <form class="panel-form" @submit.prevent="handleSubmit">
        <label>
          Asset ID
          <input v-model="form.assetId" type="text" required />
        </label>

        <label>
          Sensor Type
          <input v-model="form.sensorType" type="text" required />
        </label>

        <label>
          Latitude
          <input v-model.number="form.latitude" type="number" step="any" required required />
        </label>

        <label>
          Longitude
          <input v-model.number="form.longitude" type="number" step="any" required required />
        </label>

        <label>
          Cloud Cover %
          <input v-model.number="form.cloudCoverPercentage" type="number" min="0" max="100" />
        </label>

        <label>
          Image URL
          <input v-model="form.imageUrl" type="text" />
        </label>

        <button type="submit" :disabled="isSubmitting">
          {{ isSubmitting ? 'Saving…' : 'Create Asset' }}
        </button>

        <p v-if="submitError" class="form-message form-message--error">{{ submitError }}</p>
        <p v-if="submitSuccess" class="form-message form-message--success">Asset created!</p>
      </form>
    </aside>
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

.panel-toggle {
  position: absolute;
  top: 1rem;
  right: 1rem;
  z-index: 1001;
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  background: #1d4ed8;
  color: white;
  cursor: pointer;
}

.panel {
  position: absolute;
  top: 4rem;
  right: 1rem;
  z-index: 1000;
  width: 280px;
  max-height: calc(100% - 6rem);
  overflow-y: auto;
  padding: 1rem;
  border-radius: 8px;
  background: white;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.25);
}

.panel h2 {
  margin: 0 0 0.25rem;
  font-size: 1.1rem;
}

.panel-hint {
  margin: 0 0 1rem;
  color: #6b7280;
  font-size: 0.8rem;
}

.panel-form {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.panel-form label {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  font-size: 0.85rem;
}

.panel-form input {
  padding: 0.4rem 0.5rem;
  border: 1px solid #d1d5db;
  border-radius: 4px;
  font-size: 0.9rem;
}

.panel-form button[type='submit'] {
  padding: 0.5rem;
  border: none;
  border-radius: 4px;
  background: #16a34a;
  color: white;
  cursor: pointer;
}

.panel-form button[type='submit']:disabled {
  background: #9ca3af;
  cursor: not-allowed;
}

.form-message {
  margin: 0;
  font-size: 0.85rem;
}

.form-message--error {
  color: #b91c1c;
}

.form-message--success {
  color: #16a34a;
}
</style>

