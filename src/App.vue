<script setup lang="ts">
import { onMounted, reactive, ref } from 'vue'
import L from 'leaflet'
import markerIconUrl from 'leaflet/dist/images/marker-icon.png'
import markerShadowUrl from 'leaflet/dist/images/marker-shadow.png'
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
const DefaultIcon = L.icon({
  iconUrl: markerIconUrl,
  shadowUrl: markerShadowUrl,
  iconSize: [25, 41],
  iconAnchor: [12, 41],
  popupAnchor: [1, -34],
  shadowSize: [41, 41],
});

L.Marker.prototype.options.icon = DefaultIcon;

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
  <main class="app-container">
    
    <aside class="sidebar" :class="{ 'collapsed': !isPanelOpen }">
      <button type="button" class="panel-toggle" @click="isPanelOpen = !isPanelOpen">
        {{ isPanelOpen ? 'Hide' : 'Add Asset' }}
      </button>

      <div v-show="isPanelOpen" class="form-content">
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
            <input v-model.number="form.latitude" type="number" step="any" required />
          </label>

          <label>
            Longitude
            <input v-model.number="form.longitude" type="number" step="any" required />
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
      </div>
    </aside>

    <section class="map-area">
      <div v-if="isLoading" class="status status--loading">Loading assets&hellip;</div>
      <div v-if="error" class="status status--error">{{ error }}</div>
      <div ref="mapContainer" class="map"></div>
    </section>
  </main>
</template>

<style scoped>
.app-container {
  display: flex;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background-color: #ffffff;
}

.sidebar {
  width: 320px;
  background-color: #ffffff;
  border-right: 1px solid #e5e7eb;
  padding: 1.5rem;
  display: flex;
  flex-direction: column;
  transition: width 0.3s ease, padding 0.3s ease;
  z-index: 1000;
  overflow: auto;
}

.sidebar.collapsed {
  width: 80px;
  padding: 1.5rem 0.5rem;
}

.sidebar h2,
.sidebar p,
.sidebar label,
.sidebar input {
  color: #111827;
}

.panel-toggle {
  margin-bottom: 1.5rem;
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  background: #1d4ed8;
  color: white;
  cursor: pointer;
  align-self: flex-start;
}

.form-content h2 {
  margin: 0 0 0.25rem;
  font-size: 1.25rem;
}

.panel-hint {
  margin: 0 0 1.5rem;
  color: #6b7280;
  font-size: 0.85rem;
}

.panel-form {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.panel-form label {
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
  font-size: 0.9rem;
  font-weight: 500;
}

.panel-form input {
  padding: 0.5rem;
  border: 1px solid #d1d5db;
  border-radius: 4px;
  font-size: 0.95rem;
  background-color: #ffffff;
}

.panel-form button[type='submit'] {
  margin-top: 0.5rem;
  padding: 0.6rem;
  border: none;
  border-radius: 4px;
  background: #16a34a;
  color: white;
  font-weight: 600;
  cursor: pointer;
}

.panel-form button[type='submit']:disabled {
  background: #9ca3af;
  cursor: not-allowed;
}

.map-area {
  flex-grow: 1;
  position: relative;
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
  color: #111827;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.3);
  font-size: 0.9rem;
}

.status--error, .form-message--error {
  color: #dc2656;
}

.form-message {
  margin: 0;
  font-size: 0.85rem;
}

.form-message--success {
  color: #16a34a;
}
</style>

